<!-- TOC -->

- [内核模块触发softlockCPU无限循环异常](#内核模块触发softlockCPU无限循环异常)
	- [代码](#代码)
	- [堆栈](#堆栈)

<!-- /TOC -->
# 内核模块触发softlockCPU无限循环异常

## 开启内核参数

``` bash
kernel.hardlockup_all_cpu_backtrace = 0
kernel.hardlockup_panic = 0

kernel.softlockup_all_cpu_backtrace = 1
kernel.softlockup_panic = 1
```

## 代码

``` c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/smp.h>
#include <linux/cpumask.h>

// 定义一个函数，在每个 CPU 上执行
static void hello_from_cpu(void *data)
{
    int cpu = smp_processor_id();
    printk(KERN_INFO "Hello from CPU %d\n", cpu);
    while (1){};
}

// 模块初始化函数
static int __init hello_init(void)
{
    printk(KERN_INFO "Loading Hello module...\n");

    // 在所有 CPU 上执行 hello_from_cpu 函数
    smp_call_function_many(cpu_online_mask, hello_from_cpu, NULL, true);

    return 0;
}

// 模块退出函数
static void __exit hello_exit(void)
{
    printk(KERN_INFO "Unloading Hello module...\n");
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("A simple kernel module to print 'Hello from CPU X' on all CPUs");
```

## 堆栈


``` bash
GNU gdb (GDB) 7.6
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-unknown-linux-gnu"...

WARNING: kernel relocated [52MB]: patching 82532 gdb minimal_symbol values

      KERNEL: /lib/debug/lib/modules/3.10.0-862.el7.x86_64/vmlinux
    DUMPFILE: vmcore  [PARTIAL DUMP]
        CPUS: 4
        DATE: Mon Oct 14 16:11:42 2024
      UPTIME: 00:02:08
LOAD AVERAGE: 1.98, 0.50, 0.17
       TASKS: 317
    NODENAME: centos7
     RELEASE: 3.10.0-862.el7.x86_64
     VERSION: #1 SMP Fri Apr 20 16:44:24 UTC 2018
     MACHINE: x86_64  (2593 Mhz)
      MEMORY: 16 GB
       PANIC: "Kernel panic - not syncing: softlockup: hung tasks"
         PID: 1899
     COMMAND: "insmod"
        TASK: ffff95545a28bf40  [THREAD_INFO: ffff9550f5420000]
         CPU: 0
       STATE: TASK_RUNNING (PANIC)

crash> bt
PID: 1899   TASK: ffff95545a28bf40  CPU: 0   COMMAND: "insmod"
 #0 [ffff95546ce03d38] machine_kexec at ffffffff84460b2a
 #1 [ffff95546ce03d98] __crash_kexec at ffffffff84513402
 #2 [ffff95546ce03e68] panic at ffffffff84b07a75
 #3 [ffff95546ce03ee8] watchdog_timer_fn at ffffffff8453f5f1
 #4 [ffff95546ce03f20] __hrtimer_run_queues at ffffffff844beff6
 #5 [ffff95546ce03f78] hrtimer_interrupt at ffffffff844bf58f
 #6 [ffff95546ce03fc0] local_apic_timer_interrupt at ffffffff8445782b
 #7 [ffff95546ce03fd8] smp_apic_timer_interrupt at ffffffff84b240a3
 #8 [ffff95546ce03ff0] apic_timer_interrupt at ffffffff84b207f2
--- <IRQ stack> ---
 #9 [ffff9550f5423c88] apic_timer_interrupt at ffffffff84b207f2
    [exception RIP: init_module+4]
    RIP: ffffffffc00b8004  RSP: ffff9550f5423d30  RFLAGS: 00000246
    RAX: ffff9550f616ac01  RBX: ffffffffc03ca018  RCX: 0000000000019bb2
    RDX: 0000000000019bb1  RSI: 0000000000000000  RDI: ffff9551bfc07b00
    RBP: ffff9550f5423d30   R8: 000000000001ba40   R9: ffffffff844020f8
    R10: ffff95546ce1ba40  R11: ffffee3b02d85a80  R12: ffff9550f5423d00
    R13: ffff9550f5423d40  R14: 0000000000000018  R15: ffffffff8475c7fa
    ORIG_RAX: ffffffffffffff10  CS: 0010  SS: 0018
#10 [ffff9550f5423d38] do_one_initcall at ffffffff8440210a
#11 [ffff9550f5423d68] load_module at ffffffff8450f5ac
#12 [ffff9550f5423eb8] sys_finit_module at ffffffff8450fbf6
#13 [ffff9550f5423f50] system_call_fastpath at ffffffff84b1f7d5
    RIP: 00007f73bfb5ee29  RSP: 00007ffe98c20e68  RFLAGS: 00010207
    RAX: 0000000000000139  RBX: 0000000001dcc210  RCX: 00007f73bfbd0f90
    RDX: 0000000000000000  RSI: 000000000041aa0e  RDI: 0000000000000003
    RBP: 000000000041aa0e   R8: 0000000000000000   R9: 00007ffe98c21078
    R10: 0000000000000003  R11: 0000000000000206  R12: 0000000000000000
    R13: 0000000001dcc1d0  R14: 0000000000000000  R15: 0000000000000000
    ORIG_RAX: 0000000000000139  CS: 0033  SS: 002b
crash>
```





---
