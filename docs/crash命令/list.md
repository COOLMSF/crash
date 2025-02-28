# list(linked list)

## 概述

crash工具中，list命令是用于遍历和显示内核中的链表结构的命令。list命令有以下作用：

- 可以根据链表头的地址，遍历链表中的所有节点，并显示节点的地址。
- 可以根据链表节点中嵌入的list_head结构体的偏移量，定位到节点所属的结构体，并显示结构体的地址或者成员。
- 可以根据链表头或者节点所属的结构体类型，自动识别链表节点中嵌入的list_head结构体的偏移量，简化输入参数。
- 可以指定遍历链表的方向，正向或者反向。
- 可以指定遍历链表的终止条件，比如遍历到某个地址或者某个值。
- 可以指定显示链表节点的格式，比如十六进制或者十进制。

## 举例子

- 解析文件系统链表信息

```shell
crash> list file_system_type.next -s file_system_type.name,fs_flags 0xffffffff91f66b80
ffffffff91f66b80
  name = 0xffffffff91cdb6cd "sysfs"
  fs_flags = 8
ffffffff91e12700
  name = 0xffffffff91cae469 "rootfs"
  fs_flags = 0
ffffffff91f67280
  name = 0xffffffff91c6f9df "ramfs"
  fs_flags = 8
ffffffff91f53720
  name = 0xffffffff91cae9b3 "bdev"
  fs_flags = 0
ffffffff91f66800
  name = 0xffffffff91c9c8d6 "proc"
  fs_flags = 8
ffffffff91f0dd40
  name = 0xffffffff91ca735e "cpuset"
  fs_flags = 0
ffffffff91f09460
  name = 0xffffffff91ccd15b "cgroup"
  fs_flags = 8
ffffffff91f09760
  name = 0xffffffff91c960b4 "cgroup2"
  fs_flags = 8
ffffffff91f29ee0
  name = 0xffffffff91cf824f "tmpfs"
  fs_flags = 8
```

- `list -H head`：显示以head为头节点的链表中所有节点的地址，不包括头节点自身。head可以是一个list_head结构体的地址，也可以是一个包含list_head结构体成员的结构体类型或者地址。
- `list -h node`：显示以node为头节点的链表中所有节点的地址，包括头节点自身。node可以是一个list_head结构体的地址，也可以是一个包含list_head结构体成员的结构体类型或者地址。
- `list -o offset [-s struct[.member[,member]]] start`：显示以start为起始节点的链表中所有节点所属的结构体的地址或者成员。offset是一个数字或者一个形如struct.member的字符串，表示list_head结构体在结构体中的偏移量。struct是一个结构体类型，member是一个或多个结构体成员，用逗号分隔。如果省略struct和member，则只显示结构体地址。
- `list -r [-e end] start`：反向遍历以start为起始节点的链表，显示所有节点的地址。end是一个可选参数，表示遍历到该地址时停止。
- `list -x [-e value] start`：遍历以start为起始节点的链表，显示所有节点中存储的值。value是一个可选参数，表示遍历到该值时停止。



## 帮助信息

* <https://crash-utility.github.io/help_pages/list.html>

```
NAME
  list - linked list

SYNOPSIS
  list [[-o] offset][-e end][-[s|S] struct[.member[,member] [-l offset]] -[x|d]]
       [-r|-B] [-h [-O head_offset]|-H] start

DESCRIPTION

  This command dumps the contents of a linked list.  The entries in a linked
  list are typically data structures that are tied together in one of two
  formats:

  1. A starting address points to a data structure; that structure contains
     a member that is a pointer to the next structure, and so on.  This type
     of a singly-linked list typically ends when a "next" pointer value
     contains one of the following:

       (a) a NULL pointer.
       (b) a pointer to the start address.
       (c) a pointer to the first item pointed to by the start address.
       (d) a pointer to its containing structure.

  2. Most Linux lists of data structures are doubly-linked using "list_head"
     structures that are embedded members of the data structures in the list:

       struct list_head {
           struct list_head *next, *prev;
       };

     The linked list is typically headed by an external, standalone list_head,
     which is simply initialized to point to itself, signifying that the list
     is empty:

       #define LIST_HEAD_INIT(name) { &(name), &(name) }
       #define LIST_HEAD(name) struct list_head name = LIST_HEAD_INIT(name)

     In the case of list_head-linked lists, the "list_head.next" pointer is
     the address of a list_head structure that is embedded in the next data
     structure in the list, and not the address of the next data structure
     itself.  The starting point of the list may be:

       (a) an external, standalone, LIST_HEAD().
       (b) a list_head that is embedded within a data structure of the same
           type as the whole linked list.
       (c) a list_head that is embedded within a data structure that is
           different than the type of structures in the the linked list.

     The list typically ends when the embedded "list_head.next" pointer of
     a data structure in the linked list points back to the LIST_HEAD()
     address.  However, some list_head-linked lists have no defined starting
     point, but just link back onto themselves in a circular manner.

  This command can handle both types of linked list; in both cases the list
  of addresses that are dumped are the addresses of the data structures
  themselves.

  Alternatively, the address of a list_head, or other similar list linkage
  structure whose first member points to the next linkage structure, may be
  used as the starting address.  The caveat with this type of usage is that
  the list may pass through, and display the address of, an external standalone
  list head which is not an address of a list linkage structure that is embedded
  within the data structure of interest.

  The arguments are as follows:

  [-o] offset  The offset within the structure to the "next" pointer
               (default is 0).  If non-zero, the offset may be entered
               in either of two manners:

               1. In "structure.member" format; the "-o" is not necessary.
               2. A number of bytes; the "-o" is only necessary on processors
                  where the offset value could be misconstrued as a kernel
                  virtual address.

       -e end  If the list ends in a manner unlike the typical manners that
               are described above, an explicit ending address value may be
               entered.
    -s struct  For each address in list, format and print as this type of
               structure; use the "struct.member" format in order to display
               a particular member of the structure.  To display multiple
               members of a structure, use a comma-separated list of members.
               If any structure member contains an embedded structure or is an
               array, the output may be restricted to the embedded structure
               or an array element by expressing the struct argument as
               "struct.member.member" or "struct.member[index]"; embedded
               member specifications may extend beyond one level deep by
               expressing the argument as "struct.member.member.member...".
    -S struct  Similar to -s, but instead of parsing gdb output, member values
               are read directly from memory, so the command works much faster
               for 1-, 2-, 4-, and 8-byte members.
    -O offset  Only used in conjunction with -h; it specifies the offset of
               head node list_head embedded within a data structure which is
               different than the offset of list_head of other nodes embedded
               within a data structure.
               The offset may be entered in either of the following manners:

                 1. in "structure.member" format.
                 2. a number of bytes.

    -l offset  Only used in conjunction with -s, if the start address argument
               is a pointer to an embedded list head (or any other similar list
               linkage structure whose first member points to the next linkage
               structure), the offset to the embedded member may be entered
               in either of the following manners:

                 1. in "structure.member" format.
                 2. a number of bytes.

           -x  Override the default output format with hexadecimal format.
           -d  Override the default output format with decimal format.
           -r  For a list linked with list_head structures, traverse the list
               in the reverse order by using the "prev" pointer instead
               of "next".
           -B  Use the algorithm from R. P. Brent to detect loops instead of
               using a hash table.  This algorithm uses a tiny fixed amount of
               memory and so is especially helpful for longer lists.  The output
               is slightly different than the normal list output as it will
               print the length of the loop, the start of the loop, and the
               first duplicate in the list.

  The meaning of the "start" argument, which can be expressed symbolically,
  in hexadecimal format, or an expression evaluating to an address, depends
  upon whether the -h or -H option is pre-pended:

      start  The address of the first data structure in the list.
      start  When both the -s and -l options are used, the address of an
             embedded list_head or similar linkage structure whose first
             member points to the next linkage structure.
   -H start  The address of a list_head structure, typically that of an
             external, standalone LIST_HEAD().  The list typically ends
             when the embedded "list_head.next" of a data structure in
             the linked list points back to this "start" address.
   -h start  The address of a data structure which contains an embedded
             list_head.  The list typically ends when the embedded
             "list_head.next" of a data structure in the linked list
             points back to the embedded list_head contained in the data
             structure whose address is this "start" argument.

WARNING
  When the "-h start" option is used, it is possible that the list_head-linked
  list will:

    1. pass through an external standalone LIST_HEAD(), or
    2. pass through a list_head that is the actual starting list_head, but is
       contained within a data structure that is not the same type as all of
       the other data structures in the list.

  When that occurs, the data structure address displayed for that list_head
  will be incorrect, because the "-h start" option presumes that all
  list_head structures in the list are contained within the same type of
  data structure.  Furthermore, if the "-s struct[.member[,member]" option
  is used, it will display bogus data for that particular list_head.

  A similar issue may be encountered when the "start" address is an embedded
  list_head or similar linkage structure whose first member points to the next
  linkage structure.  When that occurs, the address of any external list head
  will not be distinguishable from the addresses that are embedded in the data
  structure of interest.  Furthermore, if the "-s" and "-l" options are used,
  it will display bogus structure data when passing through any external list
  head structure that is not embedded in the specified data structure type.

EXAMPLES
  Note that each task_struct is linked to its parent's task_struct via the
  p_pptr member:

    crash> struct task_struct.p_pptr
    struct task_struct {
       [136] struct task_struct *p_pptr;
    }

  That being the case, given a task_struct pointer of c169a000, show its
  parental hierarchy back to the "init_task" (the "swapper" task):

    crash> list task_struct.p_pptr c169a000
    c169a000
    c0440000
    c50d0000
    c0562000
    c0d28000
    c7894000
    c6a98000
    c009a000
    c0252000

  Given that the "task_struct.p_pptr" offset is 136 bytes, the same
  result could be accomplished like so:

    crash> list 136 c169a000
    c169a000
    c0440000
    c50d0000
    c0562000
    c0d28000
    c7894000
    c6a98000
    c009a000
    c0252000

  The list of currently-registered file system types are headed up by a
  struct file_system_type pointer named "file_systems", and linked by
  the "next" field in each file_system_type structure.  The following
  sequence displays the structure address followed by the name and
  fs_flags members of each registered file system type:

    crash> p file_systems
    file_systems = $1 = (struct file_system_type *) 0xc03adc90
    crash> list file_system_type.next -s file_system_type.name,fs_flags c03adc90
    c03adc90
      name = 0xc02c05c8 "rootfs",
      fs_flags = 0x30,
    c03abf94
      name = 0xc02c0319 "bdev",
      fs_flags = 0x10,
    c03acb40
      name = 0xc02c07c4 "proc",
      fs_flags = 0x8,
    c03e9834
      name = 0xc02cfc83 "sockfs",
      fs_flags = 0x10,
    c03ab8e4
      name = 0xc02bf512 "tmpfs",
      fs_flags = 0x20,
    c03ab8c8
      name = 0xc02c3d6b "shm",
      fs_flags = 0x20,
    c03ac394
      name = 0xc02c03cf "pipefs",
      fs_flags = 0x10,
    c03ada74
      name = 0xc02c0e6b "ext2",
      fs_flags = 0x1,
    c03adc74
      name = 0xc02c0e70 "ramfs",
      fs_flags = 0x20,
    c03ade74
      name = 0xc02c0e76 "hugetlbfs",
      fs_flags = 0x20,
    c03adf8c
      name = 0xc02c0f84 "iso9660",
      fs_flags = 0x1,
    c03aec14
      name = 0xc02c0ffd "devpts",
      fs_flags = 0x8,
    c03e93f4
      name = 0xc02cf1b9 "pcihpfs",
      fs_flags = 0x28,
    e0831a14
      name = 0xe082f89f "ext3",
      fs_flags = 0x1,
    e0846af4
      name = 0xe0841ac6 "usbdevfs",
      fs_flags = 0x8,
    e0846b10
      name = 0xe0841acf "usbfs",
      fs_flags = 0x8,
    e0992370
      name = 0xe099176c "autofs",
      fs_flags = 0x0,
    e2dcc030
      name = 0xe2dc8849 "nfs",
      fs_flags = 0x48000,

  In some kernels, the system run queue is a linked list headed up by the
  "runqueue_head", which is defined like so:

    static LIST_HEAD(runqueue_head);

  The run queue linking is done with the "run_list" member of the task_struct:

    crash> struct task_struct.run_list
    struct task_struct {
        [60] struct list_head run_list;
    }

  Therefore, to view the list of task_struct addresses in the run queue,
  either of the following commands will work:

    crash> list task_struct.run_list -H runqueue_head
    f79ac000
    f7254000
    f7004000
    crash> list 60 -H runqueue_head
    f79ac000
    f7254000
    f7004000

  In some kernel versions, the vfsmount structures of the mounted
  filesystems are linked by the LIST_HEAD "vfsmntlist", which uses the
  mnt_list list_head of each vfsmount structure in the list.  To dump each
  vfsmount structure in the list, append the -s option:

    crash> list -H vfsmntlist vfsmount.mnt_list -s vfsmount
    c3fc9e60
    struct vfsmount {
      mnt_hash = {
        next = 0xc3fc9e60,
        prev = 0xc3fc9e60
      },
      mnt_parent = 0xc3fc9e60,
      mnt_mountpoint = 0xc3fc5dc0,
      mnt_root = 0xc3fc5dc0,
      mnt_instances = {
        next = 0xc3f60a74,
        prev = 0xc3f60a74
      },
      mnt_sb = 0xc3f60a00,
      mnt_mounts = {
        next = 0xf7445e08,
        prev = 0xf7445f88
      },
      mnt_child = {
        next = 0xc3fc9e88,
        prev = 0xc3fc9e88
      },
      mnt_count = {
        counter = 209
      },
      mnt_flags = 0,
      mnt_devname = 0xc8465b20 "/dev/root",
      mnt_list = {
        next = 0xf7445f9c,
        prev = 0xc02eb828
      },
      mnt_owner = 0
    }
    f7445f60
    struct vfsmount {
    ...

  The task_struct of every task in the system is linked into a circular list
  by its embedded "tasks" list_head.  Show the task_struct addresses and the
  pids of all tasks in the system using "-h" option, starting with the
  task_struct at ffff88012b98e040:

    crash> list task_struct.tasks -s task_struct.pid -h ffff88012b98e040
    ffff88012b98e040
      pid = 14187
    ffff8801277be0c0
      pid = 14248
    ffffffff81a2d020
      pid = 0
    ffff88012d7dd4c0
      pid = 1
    ffff88012d7dca80
      pid = 2
    ffff88012d7dc040
      pid = 3
    ffff88012d7e9500
      pid = 4
    ...
    ffff88012961a100
      pid = 14101
    ffff880129017580
      pid = 14134
    ffff8801269ed540
      pid = 14135
    ffff880128256080
      pid = 14138
    ffff88012b8f4100
      pid = 14183

  Similar to the above, display the embedded sched_entity structure's on_rq
  member from each task_struct in the system:

    crash> list task_struct.tasks -s task_struct.se.on_rq -h ffff8800b66a0000
    ffff8800b66a0000
      se.on_rq = 1,
    ffff8800b66a0ad0
      se.on_rq = 0,
    ffff8800b66a15a0
      se.on_rq = 0,
    ffff8800b66a2070
      se.on_rq = 0,
    ffff8800b66a2b40
      se.on_rq = 0,
    ffff8800b67315a0
      se.on_rq = 0,
    ffff8800b6732b40
      se.on_rq = 0,
    ...

  The task_struct.tasks example above requires that the -h option be given
  the address of a task_struct.  Alternatively, the -l option can be given
  the address of a list_head or similar linkage structure whose first member
  points to the next linkage structure.  Again using the task_struct.tasks
  embedded list_head, dump the "comm" member of all tasks by using -l in
  conjunction with -s option:

    crash> task -R tasks.next
    PID: 7044   TASK: ffff88005ac10000  CPU: 2   COMMAND: "crash"
      tasks.next = 0xffff880109b8e3d0,
    crash> list 0xffff880109b8e3d0 -l task_struct.tasks -s task_struct.comm
    ffff880109b8e3d0
      comm = "kworker/1:2"
    ffff880109b8be00
      comm = "bash"
    ffff88019d26c590
      comm = "cscope"
    ffff880109b8b670
      comm = "kworker/0:1"
    ffff880109b8cd20
      comm = "kworker/1:0"
    ffff88005ac15c40
      comm = "vi"
    ffff88005ac11fc0
      comm = "sleep"
    ffffffff81c135c0
      comm = "swapper/0"
    ffff880212828180
      comm = "systemd"
    ...
    ffff8801288d1830
      comm = "chrome"
    ffff8801534dd4b0
      comm = "kworker/0:0"
    ffff8801534d8180
      comm = "kworker/1:1"
    ffff88010902b670
      comm = "kworker/2:2"
    ffff880109b8a750
      comm = "sudo"
    ffff88005ac10180
      comm = "crash"

  To display a liked list whose head node and other nodes are embedded within
  either same or different data structures resulting in different offsets for
  head node and other nodes, e.g. dentry.d_subdirs and dentry.d_child, the
  -O option can be used:

    crash> list -o dentry.d_child -s dentry.d_name.name -O dentry.d_subdirs -h ffff9c585b81a180
    ffff9c585b9cb140
      d_name.name = 0xffff9c585b9cb178 ccc.txt
    ffff9c585b9cb980
      d_name.name = 0xffff9c585b9cb9b8 bbb.txt
    ffff9c585b9cb740
      d_name.name = 0xffff9c585b9cb778 aaa.txt

  The dentry.d_subdirs example above is equal to the following sequence:

    crash> struct -o dentry.d_subdirs ffff9c585b81a180
    struct dentry {
      [ffff9c585b81a220] struct list_head d_subdirs;
    }
    crash> list -o dentry.d_child -s dentry.d_name.name -H ffff9c585b81a220
```

---
