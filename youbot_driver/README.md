The KUKA YouBot microblx driver
===============================

Compiling
---------

1. Clone the SOEM EtherCAT library somewhere from here:

```Shell
$ git clone https://github.com/kmarkus/soem1.2.5.git
```

and compile it:

```Shell
$ cd soem1.2.5/src/
$ make
```


  2. Define the enviromental variable SOEM_ROOT

```Shell
$ export SOEM_ROOT=<my_soem_path>
```


3. Compile the youbot driver block

```Shell
$ cd <devel_location>/microblx_youbot/youbot_driver
$ mkdir build && cd build
$ cmake -DCMAKE_INSTALL_PREFIX=<my_microblx_install_path> ..
$ make
$ make install
```

Running
-------

Run the `youbot_test.lua` script.

NOTE: Always calibrate the arm before use! Run help() to get
information about possible options.

The youbot driver requires permission to open raw sockets (capability
`cap_net_raw`) for EtherCAT communication. Furthermore, it is
advisable to run with real-time priorities (capability:
`cap_sys_nice`). To add these, run the following `setcap` command on
the luajit binary:

```Shell
$ sudo setcap cap_sys_nice,cap_net_raw+ep /usr/local/bin/luajit-2.0.2
```


License
-------

LGPL or Modified BSD


History
-------

This driver derived from the Orocos RTT driver code, with substantial
cleanup and rewriting. Nevertheless, a large amount of code has been
copied, hence the original licensing conditions apply.


Acknowledgement
---------------

The research leading to these results has received funding from the
European Community's Seventh Framework Programme (FP7/2007-2013) under
grant agreement no. FP7-ICT-231940-BRICS (Best Practice in Robotics)

