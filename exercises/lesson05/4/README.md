 
 
 # changes
 
 ## user_process
 
 ```
 void user_process(){
        char buf[30] = {0};
        tfp_sprintf(buf, "User process started. \r\n");
        //tfp_sprintf(buf, "User process started. EL %d\r\n", get_el());
        //call_sys_write(buf);
        //unsigned long stack = call_sys_malloc();
        //if (stack < 0) {
        //      printf("Error while allocating stack for process 1\n\r");
        //      return;
        //}
        //int err = call_sys_clone((unsigned long)&user_process1, (unsigned long)"12345", stack);
        //if (err < 0){
        //      printf("Error while clonning process 1\n\r");
        //      return;
        //} 
        //stack = call_sys_malloc();
        //if (stack < 0) {
        //      printf("Error while allocating stack for process 1\n\r");
        //      return;
        //}
        //err = call_sys_clone((unsigned long)&user_process1, (unsigned long)"abcd", stack);
        //if (err < 0){
        //      printf("Error while clonning process 2\n\r");
        //      return;
        //} 
        //call_sys_exit();
}

 ```
 
 # how to run user process
 
							 						 
## gdb

```
qemu-system-aarch64 -machine raspi3 -serial null -serial mon:stdio -nographic -kernel kernel8.img -s -S
```


![image](https://github.com/magnate3/raspberry-pi-os/blob/master/exercises/lesson05/4/pic/flow.png)

![image](https://github.com/magnate3/raspberry-pi-os/blob/master/exercises/lesson05/4/pic/fork.png)

![image](https://github.com/magnate3/raspberry-pi-os/blob/master/exercises/lesson05/4/pic/move.png)

![image](https://github.com/magnate3/raspberry-pi-os/blob/master/exercises/lesson05/4/pic/to_user.png)

 ```
root@ubuntu:~/arm/raspberry-pi-os/exercises/lesson05/3/bl4ckout31# gdb-multiarch build/kernel8.elf 
GNU gdb (Ubuntu 8.1.1-0ubuntu1) 8.1.1
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
Reading symbols from build/kernel8.elf...done.
(gdb) target remote:1234
Remote debugging using :1234
0x0000000000000000 in ?? ()
(gdb) b ret_from_fork
Breakpoint 1 at 0x82fe0
(gdb) c
Continuing.

Thread 1 hit Breakpoint 1, 0x0000000000082fe0 in ret_from_fork ()
(gdb) bt
#0  0x0000000000082fe0 in ret_from_fork ()
#1  0x0000000000082fe0 in ret_from_syscall ()
Backtrace stopped: previous frame identical to this frame (corrupt stack?)
(gdb) b kernel_process
Breakpoint 2 at 0x80898: file src/kernel.c, line 51.
(gdb) c
Continuing.

Thread 1 hit Breakpoint 2, kernel_process () at src/kernel.c:51
51              printf("Kernel process started. EL %d\r\n", get_el());
(gdb) b move_to_user_mode
Breakpoint 3 at 0x8124c: file src/fork.c, line 49.
(gdb) c
Continuing.

Thread 1 hit Breakpoint 3, move_to_user_mode (pc=526428) at src/fork.c:49
49              struct pt_regs *regs = task_pt_regs(current);
(gdb) bt
#0  move_to_user_mode (pc=526428) at src/fork.c:49
#1  0x00000000000808b8 in kernel_process () at src/kernel.c:52
#2  0x0000000000082ff0 in ret_from_fork ()
Backtrace stopped: previous frame identical to this frame (corrupt stack?)
(gdb) b ret_to_user
Breakpoint 4 at 0x82ff0
(gdb) c
Continuing.

Thread 1 hit Breakpoint 4, 0x0000000000082ff0 in ret_to_user ()
(gdb) bt
#0  0x0000000000082ff0 in ret_to_user ()
#1  0x0000000000082ff0 in ret_from_fork ()
Backtrace stopped: previous frame identical to this frame (corrupt stack?)
(gdb) b   user_process
Breakpoint 5 at 0x80864: file src/kernel.c, line 23.
(gdb) c
Continuing.

Thread 1 hit Breakpoint 5, user_process () at src/kernel.c:23
23              char buf[30] = {0};
(gdb) bt
#0  user_process () at src/kernel.c:23
#1  0x0000000000000000 in ?? ()
Backtrace stopped: not enough registers or memory available to unwind further
(gdb) 
 ```