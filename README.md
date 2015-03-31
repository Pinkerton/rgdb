# rgdb
:star: Seamless remote debugging with gdb for CS 225

##Description##
Sometimes the verison of gdb shipped with OSX isn't very helpful, and manually committing and pushing code to school workstations is slow. So, this script does it for you.

There's a couple ways to run this script. The easiest is probably to copy `rgdb.py` to your `cs225` or specific MP/lab directory, [make it executable, and add it to your path](http://stackoverflow.com/questions/15587877/run-a-python-script-in-terminal-without-the-python-command). 

Then, run gdb remotely with some variation of `python rgdb.py mp5` or `rgdb.py mp5`. It'll commit your code and launch an ssh session that pulls the most recent version, makes it, and starts gdb.

You'll probably only find this useful if you like developing locally, but it's been helpful for debugging my segfault-ridden code.

**Known issue:** exiting out of the remote gdb session is a bit of a pain. This could probably be solved in code, but **CTRL+D** twice works well enough.

##Configuration##

This project relies on having a proper ssh alias for logging into EWS. To make this work, your `~/.ssh/config` should look something like this:

```
Host ews
	User your_netID
	Hostname remlnx.ews.illinois.edu
```
[This guide](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/) explains how to set up password-less ssh authentication on OSX.

Also, change `EWS_PATH` and in `rgdb.py` to reflect where your `cs225` directory lives on EWS. By default, it's `~/Documents/cs225`. If you have a pre-existing SSH alias, also be sure to update `SSH_ALIAS`.

##Details##

If you've tried debugging your CS 225 MP on a Mac, you might have run across something like this before:

```
[localhost mp5]$ gdb mp5                                                                                
(gdb) r
[...]
Starting program: /Users/pinkerton/Documents/cs225/mp5/mp5 
Program received signal SIGSEGV, Segmentation fault.
0x00000001000039c6 in PNG::_clamp_xy (this=0x7fff5fbff4f0, x=@0x7fff5f400260: 0, y=@0x7fff5f400258: 0)
    at png.cpp:101
101		if (x >= _width)
(gdb) bt
#0  0x00000001000039c6 in PNG::_clamp_xy (this=0x7fff5fbff4f0, x=@0x7fff5f400260: 0, y=@0x7fff5f400258: 0)
    at png.cpp:101
#1  0x0000000100005532 in PNG::operator() (this=0x7fff5fbff4f0, x=0, y=0) at png.cpp:189
#2  0x0000000100002d08 in ?? ()
#3  0x0000000000000000 in ?? ()
```

:anguished: Not super helpful, right? So you `svn commit` and log into EWS.  `cd ~/Documents/cs225/mp5` and `svn up; make; gdb mp5` and you're finally up and running on a real Linux system. On there, gdb looks something like this:

```
[ssh@ews mp5]$ gdb mp5
[...]
(gdb) r
Starting program: /home/pinkerton/Documents/cs225/mp5/mp5 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
made it here

Program received signal SIGSEGV, Segmentation fault.
0x000000000040bde6 in PNG::_clamp_xy (this=0x7fffffffd050, 
    x=@0x7fffff3ff2c0: 0, y=@0x7fffff3ff2b8: 0) at png.cpp:101
101		if (x >= _width)
(gdb) bt
#0  0x000000000040bde6 in PNG::_clamp_xy (this=0x7fffffffd050, 
    x=@0x7fffff3ff2c0: 0, y=@0x7fffff3ff2b8: 0) at png.cpp:101
#1  0x000000000040d722 in PNG::operator() (this=0x7fffffffd050, x=0, y=0)
    at png.cpp:189
#2  0x000000000040b198 in Quadtree::buildTree (this=0x7fffffffd000, 
    source=..., x=0, y=0, nx=0, ny=0) at quadtree.cpp:100
#3  0x000000000040b234 in Quadtree::buildTree (this=0x7fffffffd000, 
[...]
```

:bulb: Yay! Turns out something in Quadtree::buildTree is causing a segfault. Back to work...

## Licence, etc.##
The license is some variation of GPL (check the LICENSE file). It's because of the code I stole from the paramiko project for interactive SSH sessions. 

I wrote this code in ~30 minutes, so you probably shouldn't trust your life with it. That said, it works pretty well for what it tries to do.
