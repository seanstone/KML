# Introduction
## KML
Kernel-Mode Linux is a technology which enables us to execute user programs in kernel mode. In Kernel Mode Linux, user programs can be executed as user processes that have the privilege level of kernel mode. The benefit of executing user programs in kernel mode is that the user programs can access kernel address space directly. For example, user programs can invoke system calls very fast because it is unnecessary to switch between a kernel mode and user-mode by using costly software interruptions or context switches. In addition, user programs are executed as ordinary processes (except for their privilege level, of course), so scheduling and paging are performed as usual, unlike kernel modules. With this technology, we can implement the real time task in user space with advantages of flexibility and low latency.

KML supports generic x86_64 and ARM architecture family. At present, we evaluate KML on ARM platform along with real time enhancements, such as PREEMPT_RT and tickless kernel. The major differential of KML on ARM is that user processes specified by KML are created within SYSTEM_MODE by setting the CPSR register as SYSTEM mode and authorize this program the access of the kernel address by set_fs(KERNEL_DS). However, on ARM platform, not all system calls can be invoked directly. If the schedule related functions like schedule used in KML, the whole system will freeze or go into segfault. This reason of this restriction is not clarified and needs further investigation.
Limitation of Resource Access

With KML, though the user programs can invoke the kernel functions directly, they can't understand the symbols of these functions. One solution is that the user program invokes the kernel functions by addresses directly. Here is an example. If we want to invoke the kernel function, sys_clock_gettime, in KML user program, we can first find out the address from kernel's ELF image, named after vmlinx, by the nm command and invoke this address by casting it to a function. e.g.

Declaration:
```
long sys_clock_gettime(clockid_t which_clock, struct timespec __user *tp);
```

Invoke sys_clock_gettime in KML by address:
```
((long (*)(clockid_t which_clock, struct timespec *tp))0x8007fa20)(CLOCK_MONOTONIC, &time2);

```

It is very inconvenient for this solution because the user program needs to modify this address whenever the kernel is updated.

## vDSO
The vDSO, virtual dynamic shared object, is a small shared library that the kernel automatically maps into the address space of all user-space applications. User programs usually do not need to concern themselves with these details as the vDSO is most commonly called by the C library. This technique provides a good interface for the KML user program to access the kernel resource through symbols. When kernel is updated, what we need to do is rebuild the user program by relinking it with the new vDSO library.
Known Issues:
 - Scheduling specific operations provided by Linux kernel such as schedule can not be invoked in KML user program, otherwise system will freeze or get into segmentation faults.
 - On AM572x, powered by Cortex A15 core, LPAE (i.e. CONFIG_ARM_LPAE) should be disabled.

## Instructions

KML now is evaluated on AM572x and i.MX6Q SDB with Linux 4.4.12. To enable it, get the kernel source with 4.4.12 version and patch it by the following patch files:
 - 0001-Integrate-KML.patch
 - 0002-Use-VDSO-as-the-wrapper-for-KML-resource.patch

Then say Y in Kernel Mode Linux and VDSO field of kernel configuration, and N for ARM LPAE.
 - CONFIG_KERNEL_MODE_LINUX=y
 - CONFIG_KML_CHECK_CHROOT=y
 - CONFIG_VDSO=y
 - CONFIG_ARM_LPAE=N

In addition to the Linux kernel, the KML user program needs to be linked with vdso.so produced in kernel building (located in arch/arm/vdso/vdso.so). Simply supply vdso.so together with other object files while linking. If the program is linked with vdso correctly, the source of the symbols provided by vDSO should be LINUX not GLIBC. Here is an example for clock_gettime in vDSO.
```
$ arm-linux-gnueabihf-nm  build/user/drivers/mctest | grep clock_gettime
         U clock_gettime@@LINUX_2.6
```

The KML user program should be put under the "/trusted" directory.

## Current status

#### Linux 4.18.16 working for x86_64

#### Linux 4.14.71 broken for ARM
```
[  432.445868] Internal error: Oops - undefined instruction: 0 [#3] ARM
[  432.452528] Modules linked in: usb_f_mass_storage usb_f_acm u_serial usb_f_rndis u_ether libcomposite
[  432.462188] CPU: 0 PID: 1033 Comm: busybox Tainted: G      D         4.14.71-kml #1
[  432.470183] Hardware name: Generic AM33XX (Flattened Device Tree)
[  432.476547] task: df70c000 task.stack: df34c000
[  432.481294] PC is at ret_fast_syscall+0x50/0x54
[  432.486024] LR is at 0xb6eeba00
[  432.489300] pc : [<c01077b0>]    lr : [<b6eeba00>]    psr: 00080093
[  432.495843] sp : df34dfa8  ip : 00000000  fp : 000ca3b8
[  432.501296] r10: 00000000  r9 : df34c000  r8 : c0107944
[  432.506751] r7 : 0000000b  r6 : 000a5fbe  r5 : 000ca3b8  r4 : 000ca394
[  432.513565] r3 : c0a75e04  r2 : df34dfec  r1 : 0000001f  r0 : 00000000
[  432.520383] Flags: nzcv  IRQs off  FIQs on  Mode SVC_32  ISA ARM  Segment kernel
[  432.528106] Control: 10c5387d  Table: 9e084019  DAC: 00000055
[  432.534105] Process busybox (pid: 1033, stack limit = 0xdf34c210)
[  432.540468] Stack: (0xdf34dfa8 to 0xdf34e000)
[  432.545019] dfa0:                   000ca394 000ca3b8 00000000 00000000 00000000 00000000
[  432.553564] dfc0: 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
[  432.562108] dfe0: 00000000 beebfd10 00000000 b6eeba00 0000001f 00000000 00000000 00000000
[  432.570660] Code: e9527ffe e1a00000 e28dd050 e1b0f00e (e7f001f2) 
[  432.577026] ---[ end trace bcbbd71a3e1e3453 ]---
```

```
