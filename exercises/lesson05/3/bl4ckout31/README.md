# out user mode elX

```
void user_process(){
        char buf[30] = {0};
        tfp_sprintf(buf, "User process started. EL %d\r\n", get_el());
		..............
}
```


## run

```
 qemu-system-aarch64 -machine raspi3 -serial null -serial mon:stdio -nographic -kernel kernel8.img
Kernel process started. EL 1
0      , ESR: 2000000, address: 81e0c
QEMU: Terminated
```
** run with bugs**

# gdb
```
qemu-system-aarch64 -machine raspi3 -serial null -serial mon:stdio -nographic -kernel kernel8.img -s -S
```

```
root@ubuntu:~/arm/raspberry-pi-os/exercises/lesson05/3/bl4ckout31# gdb build/kernel8.elf 
GNU gdb (Ubuntu 8.1-0ubuntu3.2) 8.1.0.20180409-git
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "aarch64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from build/kernel8.elf...(no debugging symbols found)...done.
(gdb) target remote:1234
Remote debugging using :1234
0x0000000000000000 in ?? ()
(gdb) b el1_irq
Breakpoint 1 at 0x82d10
(gdb) b el0_irq
Breakpoint 2 at 0x82db8
(gdb) b el0_sync
Breakpoint 3 at 0x82e68
(gdb) c
Continuing.

Thread 1 hit Breakpoint 3, 0x0000000000082e68 in el0_sync ()
(gdb) bt
#0  0x0000000000082e68 in el0_sync ()
#1  0x000000000008088c in user_process ()
#2  0x0000000000000000 in ?? ()
Backtrace stopped: not enough registers or memory available to unwind further
(gdb) c
Continuing.

Thread 1 hit Breakpoint 3, 0x0000000000082e68 in el0_sync ()
(gdb) bt
#0  0x0000000000082e68 in el0_sync ()
#1  0x0000000000080890 in user_process ()
#2  0x0000000000000000 in ?? ()
Backtrace stopped: not enough registers or memory available to unwind further
```


```
Thread 1 hit Breakpoint 2, 0x0000000000082db8 in el0_irq ()
(gdb) bt
#0  0x0000000000082db8 in el0_irq ()
#1  0x0000000000080840 in user_process1 ()
Backtrace stopped: previous frame inner to this frame (corrupt stack?)
(gdb) 
```

