CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
project(youbot-driver)

set(CMAKE_CXX_FLAGS "-Wall -Werror -fvisibility=hidden")
set( CMAKE_CXX_COMPILER clang++ )
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake") ## temporary resources, for UBX

# Global
set(INSTALL_LIB_TYPES_DIR lib/microblx/types CACHE PATH "Installation directory for types libraries")
set(INSTALL_LIB_BLOCKS_DIR lib/microblx/blocks CACHE PATH "Installation directory for blocks libraries")
set(INSTALL_INCLUDE_DIR include/microblx CACHE PATH "Installation directory for header files (types)")
set(INSTALL_CMAKE_DIR  share/microblx/cmake CACHE PATH  "Installation directory for CMake files") # default ${DEF_INSTALL_CMAKE_DIR}

# Local
set(LIBRARY_OUTPUT_PATH ${CMAKE_HOME_DIRECTORY}/lib/ CACHE PATH "Configure the library output path.")
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_HOME_DIRECTORY}/bin/ CACHE PATH "Configure the executable output path.")

# check environment and UBX
set (UBX_ROOT $ENV{UBX_ROOT} )
if (NOT UBX_ROOT)
 message( FATAL_ERROR "UBX_ROOT is not defined. CMake install will abort." )
endif()

find_package(Microblx REQUIRED)

# check soem
set(SOEM_ROOT $ENV{SOEM_ROOT})
if (NOT SOEM_ROOT)
 message( FATAL_ERROR "SOEM_ROOT is not defined. CMake install will abort." )
endif()
set(SOEM_INC ${SOEM_ROOT}/src)
set(SOEM_LIB ${SOEM_ROOT}/src)

##
# Add uninstall target.
##
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)
    
add_custom_target(uninstall
    "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake/cmake_uninstall.cmake"
)

##
# Generate config package TODO
##
# set(CONF_INCLUDE_DIRS "${INSTALL_INCLUDE_DIR}")
# set(CONF_CMAKE_DIR "${INSTALL_CMAKE_DIR}")
# configure_file(
#     "${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT}-config.cmake.in"
#     "${CMAKE_CURRENT_BINARY_DIR}/cmake/${PROJECT}-config.cmake"
#      @ONLY
# )

##
# Generate types
##
set(GEN_HEXARR ${UBX_ROOT}/tools/file2carr.lua)

file(GLOB types
  ${CMAKE_CURRENT_SOURCE_DIR}/types/*.h
)


set(outfiles "")
foreach( _type ${types} )
  string(REPLACE ".h" ".h.hexarr" _outfile ${_type})
  add_custom_command(
  OUTPUT ${_outfile}
  COMMAND ${GEN_HEXARR}
  ARGS ${_type}  > ${_outfile}
  )
  list(APPEND outfiles ${_outfile})
endforeach(_type)
add_custom_target( expanded_defs ALL DEPENDS ${outfiles} )


find_package(kdl-types REQUIRED)

include_directories(${SOEM_INC}
                    ${UBX_INCLUDE_DIR}
                    ${KDL_TYPES_INCLUDE_DIRS}
)

# NOTE: In the original code, only one library is generated, both for type and block. We keep in that way.
add_library(youbot_driver SHARED youbot_driver.c)
set_target_properties(youbot_driver PROPERTIES PREFIX ""
                                               PUBLIC_HEADER "${outfiles};${types}" # Install headers
) 
set_property(TARGET youbot_driver PROPERTY INSTALL_RPATH_USE_LINK_PATH TRUE)
target_link_libraries(youbot_driver ${UBX_LIBRARIES} ${SOEM_LIB})

install(TARGETS youbot_driver 
  EXPORT ${PROJECT}-targets
  LIBRARY DESTINATION "${INSTALL_LIB_BLOCKS_DIR}" COMPONENT youbot_driver
  PUBLIC_HEADER DESTINATION "${INSTALL_INCLUDE_DIR}/types" COMPONENT dev
)

# export(PACKAGE ${PROJECT}) TODO
# 
# # Install the youbt-driver-config.cmake 
# install(FILES
#   "${PROJECT_BINARY_DIR}/cmake/${PROJECT}-config.cmake"
#   DESTINATION "${INSTALL_CMAKE_DIR}" COMPONENT dev)
#  
# # # Install the export set for use with the install-tree
# install(EXPORT ${PROJECT}-targets DESTINATION
#   "${INSTALL_CMAKE_DIR}" COMPONENT dev)