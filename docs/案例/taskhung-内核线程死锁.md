<!-- TOC -->

- [内核模块触发空指针异常](#内核模块死锁异常)
	- [代码](#代码)
	- [堆栈](#堆栈)

<!-- /TOC -->
# 内核模块死锁异常

## 开启内核参数

``` bash
# **`sysctl -w kernel.hung_task_panic=1`**
kernel.hung_task_panic = 1
kernel.hung_task_timeout_secs=60
```

``` bash
sysctl -p
```
## 代码

``` c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/mutex.h>
#include <linux/sched.h>
#include <linux/delay.h>
#include <linux/smp.h>
#include <linux/cpumask.h>

static DEFINE_MUTEX(mutex1);
static DEFINE_MUTEX(mutex2);

// 定义一个函数，在每个 CPU 上执行
static void deadlock_on_cpu(void *data)
{
    int cpu = smp_processor_id();
    printk(KERN_INFO "Deadlock on CPU %d\n", cpu);

    if (cpu % 2 == 0) {
        mutex_lock(&mutex1);
        msleep(100);  // 等待一段时间，确保另一个 CPU 获取 mutex2
        mutex_lock(&mutex2);
    } else {
        mutex_lock(&mutex2);
        msleep(100);  // 等待一段时间，确保另一个 CPU 获取 mutex1
        mutex_lock(&mutex1);
    }

    // 永远不会到达这里
    mutex_unlock(&mutex1);
    mutex_unlock(&mutex2);
}

// 模块初始化函数
static int __init deadlock_init(void)
{
    printk(KERN_INFO "Loading Deadlock module...\n");

    // 在所有 CPU 上执行 deadlock_on_cpu 函数
    smp_call_function_many(cpu_online_mask, deadlock_on_cpu, NULL, true);

    return 0;
}

// 模块退出函数
static void __exit deadlock_exit(void)
{
    printk(KERN_INFO "Unloading Deadlock module...\n");
}

module_init(deadlock_init);
module_exit(deadlock_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("A simple kernel module to simulate a deadlock");
```

makefile
``` c
obj-m += lkm_hello_world.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

test:
	sudo dmesg -C
	sudo insmod lkm_hello_world.ko
	sudo rmmod lkm_hello_world.ko
	dmesg
```

``` bash
[root@centos7 hello-world-linux-module]# ls
Makefile  README.md  lkm_hello_world.c  lkm_hello_world.c.bak  screenshots
[root@centos7 hello-world-linux-module]# make
make -C /lib/modules/3.10.0-862.el7.x86_64/build M=/root/hello-world-linux-module modules
make[1]: Entering directory `/usr/src/kernels/3.10.0-862.el7.centos.x86_64'
  CC [M]  /root/hello-world-linux-module/lkm_hello_world.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /root/hello-world-linux-module/lkm_hello_world.mod.o
  LD [M]  /root/hello-world-linux-module/lkm_hello_world.ko
make[1]: Leaving directory `/usr/src/kernels/3.10.0-862.el7.centos.x86_64'
```

安装完ko之后，系统会hung死，并出发crash
``` bash
insmod lkm_hello_world.ko
```

## 堆栈

![1](image/WeChatWorkScreenshot_13889554-7cfa-4536-b945-efc6e3feddb2.png)





---
