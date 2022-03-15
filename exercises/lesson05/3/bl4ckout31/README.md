# access elX in user mode 

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

**run with bugs**

# irq_vector_init  vbar_el1

```
.globl irq_vector_init
irq_vector_init:
        adr     x0, vectors             // load VBAR_EL1 with virtual
        msr     vbar_el1, x0            // vector table address
        ret
```
**irq_vector_init  vbar_el1 not vbar_el0**

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



#  kernel_exit



```
.macro	kernel_exit, el
	ldp	x22, x23, [sp, #16 * 16]
	ldp	x30, x21, [sp, #16 * 15] 

	.if	\el == 0
	msr	sp_el0, x21
	.endif /* \el == 0 */

	msr	elr_el1, x22			
	msr	spsr_el1, x23


	ldp	x0, x1, [sp, #16 * 0]
	ldp	x2, x3, [sp, #16 * 1]
	ldp	x4, x5, [sp, #16 * 2]
	ldp	x6, x7, [sp, #16 * 3]
	ldp	x8, x9, [sp, #16 * 4]
	ldp	x10, x11, [sp, #16 * 5]
	ldp	x12, x13, [sp, #16 * 6]
	ldp	x14, x15, [sp, #16 * 7]
	ldp	x16, x17, [sp, #16 * 8]
	ldp	x18, x19, [sp, #16 * 9]
	ldp	x20, x21, [sp, #16 * 10]
	ldp	x22, x23, [sp, #16 * 11]
	ldp	x24, x25, [sp, #16 * 12]
	ldp	x26, x27, [sp, #16 * 13]
	ldp	x28, x29, [sp, #16 * 14]
	add	sp, sp, #S_FRAME_SIZE		
	eret
	.endm
```
**call eret**


#  kernel_process

***successfully moving process to user mode***

```
void kernel_process(){
        printf("Kernel process started. EL %d\r\n", get_el());
        int err = move_to_user_mode((unsigned long)&user_process);
        if (err < 0){
                printf("Error while moving process to user mode\n\r");
        }
        else
        {
                printf(" successfully moving process to user mode\n\r");
        }
}
```

```
qemu-system-aarch64 -machine raspi3 -serial null -serial mon:stdio -nographic -kernel kernel8.img 
Kernel process started. EL 1
 successfully moving process to user mode
User process started. 
```
