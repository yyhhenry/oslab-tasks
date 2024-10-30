# proc文件系统的实现

## 实验目的

此次实验的基本目的：掌握虚拟文件系统的实现原理，深入实践文件、目录、索引节点、目录解析等概念，通过实践加深对文件系统的深入认识。

## 实验内容

本实验要求实现一个`/proc`文件——`/proc/psinfo`，要求在执行`cat /proc/psinfo`时能在屏幕上打印出当前系统中的进程信息。包含的字段有 `pid, exec, state, uid, father-id, start_time`。

## 提示

`kernel/printk.c`中可以补充一个 `sprintf` 函数，之后方便使用。

注意 `include/sys/stat.h` 中的文件类型宏定义，添加 `S_IFPROC` 和 `S_ISPROC` 宏定义，仿写 `S_IFCHR` 和 `S_ISCHR` 即可。

```c
#define S_IFPROC 0110000
#define S_ISPROC(m) (((m) & S_IFMT) == S_IFPROC)
```

在 `fs/namei.c` 中修改 `sys_mknod` 函数`if(S_ISBLK(mode) || S_ISCHR(mode))`的部分，增加我们自定义的 `S_ISPROC`。

在 `fs/read_write.c` 中修改 `sys_read` 函数，同样添加 `S_ISPROC(inode->i_mode)` 的判断，内部在 `inode->i_zone[0] == 0` 的情况下，调用 `psinfo_read` 函数。

为 `init/main.c` 添加关于 `mkdir` 和 `mknod` 的系统调用声明，其中的 `init` 部分需要添加如下代码以创建 `/proc` 目录和 `/proc/psinfo` 文件。

```c
mkdir("/proc", 0755);
mknod("/proc/psinfo", S_IFPROC | 0444, 0);
```

新建 `kernel/proc.c` 实现 `psinfo_read` 函数，仿照 `block_read` 进行定义，然后内部实现直接对参数中的`buf`使用`put_fs_byte`进行写入，具体的写入内容可以使用`sprintf`函数进行格式化。
