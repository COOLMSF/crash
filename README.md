# crash - linux kernel coredump analysis

## 相关站点

* <https://crash-utility.github.io/>
* GitHub仓库：<https://github.com/crash-utility/crash/releases>
* Fedora koji 构建：<https://koji.fedoraproject.org/koji/packageinfo?packageID=307>
* 帮助信息：<https://crash-utility.github.io/help.html>
* anolis bug追踪平台： <https://bugs.openanolis.cn/view_all_bug_page.php>

## 目录

* [基础知识](docs/基础知识.md)
    * [kdump](docs/基础知识/kdump.md)
    * [crash](docs/基础知识/crash.md)
    * [crashkernel启动参数](docs/基础知识/crashkernel启动参数.md)
    * [栈回溯机制](docs/基础知识/栈回溯机制.md)
    * [X86堆栈](docs/基础知识/X86堆栈.md)
    * [/proc/sysrq-trigger功能](docs/基础知识/sysrq-trigger功能.md)
    * [no-omit-frame-pointer编译标识](docs/基础知识/no-omit-frame-pointer.md)
    * [ELF符号](docs/基础知识/ELF符号.md)
    * [内核ELF中的percpu变量](docs/基础知识/内核ELF中的percpu变量.md)
    * [进程内存空间](docs/基础知识/进程内存空间.md)
* [crash命令](docs/crash命令.md)
    * [指针(*)](docs/crash命令/指针.md)
    * [extend](docs/crash命令/extend.md)
    * [log](docs/crash命令/log.md)
    * [rd](docs/crash命令/rd.md)
    * [task](docs/crash命令/task.md)
    * [alias](docs/crash命令/alias.md)
    * [files](docs/crash命令/files.md)
    * [mach](docs/crash命令/mach.md)
    * [repeat](docs/crash命令/repeat.md)
    * [timer](docs/crash命令/timer.md)
    * [ascii](docs/crash命令/ascii.md)
    * [foreach](docs/crash命令/foreach.md)
    * [mod](docs/crash命令/mod.md)
    * [runq](docs/crash命令/runq.md)
    * [tree](docs/crash命令/tree.md)
    * [bpf](docs/crash命令/bpf.md)
    * [fuser](docs/crash命令/fuser.md)
    * [mount](docs/crash命令/mount.md)
    * [search](docs/crash命令/search.md)
    * [union](docs/crash命令/union.md)
    * [bt](docs/crash命令/bt.md)
    * [gdb](docs/crash命令/gdb.md)
    * [net](docs/crash命令/net.md)
    * [set](docs/crash命令/set.md)
    * [vm](docs/crash命令/vm.md)
    * [btop](docs/crash命令/btop.md)
    * [help](docs/crash命令/help.md)
    * [p](docs/crash命令/p.md)
    * [sig](docs/crash命令/sig.md)
    * [vtop](docs/crash命令/vtop.md)
    * [dev](docs/crash命令/dev.md)
    * [ipcs](docs/crash命令/ipcs.md)
    * [ps](docs/crash命令/ps.md)
    * [struct](docs/crash命令/struct.md)
    * [waitq](docs/crash命令/waitq.md)
    * [dis](docs/crash命令/dis.md)
    * [irq](docs/crash命令/irq.md)
    * [pte](docs/crash命令/pte.md)
    * [swap](docs/crash命令/swap.md)
    * [whatis](docs/crash命令/whatis.md)
    * [eval](docs/crash命令/eval.md)
    * [kmem](docs/crash命令/kmem.md)
    * [ptob](docs/crash命令/ptob.md)
    * [sym](docs/crash命令/sym.md)
    * [wr](docs/crash命令/wr.md)
    * [exit](docs/crash命令/exit.md)
    * [list](docs/crash命令/list.md)
    * [ptov](docs/crash命令/ptov.md)
    * [sys](docs/crash命令/sys.md)
    * [q](docs/crash命令/q.md)
* [源码分析](docs/源码分析.md)
    * [kexec系统调用](docs/源码分析/kexec系统调用.md)
    * [kexec用户态程序](docs/源码分析/kexec用户态程序.md)
    * [kdump服务](docs/源码分析/kdump服务.md)  
* [crash基本用法](docs/crash基本用法.md)
    * [x86_64虚拟地址空间布局](docs/crash基本用法/x86_64虚拟地址空间布局.md)
    * [获取pfn获取pfn、page和mem_map](docs/crash基本用法/获取pfn_page_mem_map.md)
    * [获取进程CR3寄存器值](docs/crash基本用法/获取进程CR3寄存器值.md)
    * [获取percpu变量](docs/crash基本用法/获取percpu变量.md)
    * [获取所有task元数据](docs/crash基本用法/获取所有task元数据.md)
    * [获取当前系统支持的文件系统](docs/crash基本用法/获取当前系统支持的文件系统.md)
    * [获取进程vm_area_struct链表](docs/crash基本用法/获取进程vm_area_struct链表.md)
* [问题分类](docs/问题分类.md)
    * [Oops](docs/问题分类/Oops.md)
    * [panic](docs/问题分类/panic.md)
    * [soft-lockup](docs/问题分类/soft-lockup.md)
    * [hard-locakup](docs/问题分类/hard-locakup.md)
* [案例](docs/案例.md)
    * [手动触发panic](docs/案例/手动触发panic.md)
    * [内核模块触发空指针异常](docs/案例/内核模块触发空指针异常.md)
    * [NULLPOINTER-空指针参数](docs/案例/NULLPOINTER-空指针参数.md)
    * [内核链表踩踏-前序节点](docs/案例/内核链表踩踏-前序节点.md)
    * [内核链表踩踏-后继节点](docs/案例/内核链表踩踏-后继节点.md)
    * [softlockup-等待状态寄存器](docs/案例/softlockup-等待状态寄存器.md)
    * [softlockup-CPU无限循环](docs/案例/softlockup-CPU无限循环.md)



## 图示

![20220315_140057_50](image/20220315_140057_50.png)

![20220402_140850_41](image/20220402_140850_41.png)

![20220403_204433_64](image/20220403_204433_64.png)

![20220405_094715_95](image/20220405_094715_95.png)

![20220406_160055_99](image/20220406_160055_99.png) 

---
