# FIXME

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
