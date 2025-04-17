binder是android提供的一种实现跨进程通信的机制。它是android framework核心的组件之一。

linux常见的跨进程通信方式:
1. 管道和命名管道: 管道是半双工的，命名管道是半双工的，但可以通过FIFO实现全双工。
2. 共享内存: 共享内存是进程间通信的最高效方式，因为它避免了数据的复制，但需要进程同步。
3. Socket: Socket是一种用于网络通信的机制，它可以在不同主机之间进行通信。
4. 文件IO: 文件IO是一种用于进程间通信的机制，它可以在不同进程之间共享文件。

binder的优势:
1. 安全性: binder机制提供了进程间通信的安全性，它可以通过UID和PID来验证进程的身份。
2. 效率: binder机制是一种高效的进程间通信机制，它避免了数据的复制，并且可以通过内存映射来实现高效的数据传输。
3. 灵活性: binder机制是一种灵活的进程间通信机制，它可以通过Binder驱动程序来实现进程间通信，并且可以通过Binder对象来实现进程间通信。
4. 稳定性: 共享内存会产生多线程的竞争问题。Binder基于C/S架构，客户端（Client）通过transact方法向服务端（Server）发送请求，服务端通过onTransact方法接收请求并返回结果，整个过程是同步的，因此不会存在并发问题。

binder的作用:
远程调用服务的方法，实现进程间通信。因为android系统中的服务都是运行在system_server进程中的，所以需要通过binder机制来调用这些服务的方法。context.getService()方法就是通过binder机制来获取服务的。例如WindowManagerService、PackageManagerService、ActivityManagerService、ActivityTaskManagerService等。

<img src="https://img-blog.csdnimg.cn/20190927101151644.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nvbmd6aTEyMjg=,size_16,color_FFFFFF,t_70" style="width:100%">

<img src="https://img-blog.csdnimg.cn/img_convert/94dcca4747ea62bfd955b82033951ada.png" style="width:100%">

### binder一次可以传输多大的数据
4M或者1M~8KB
[参考](https://cloud.tencent.com/developer/article/1639706)
> frameworks/native/libs/binder/ProcessState.cpp
```c++
#define BINDER_VM_SIZE ((1 * 1024 * 1024) - sysconf(_SC_PAGE_SIZE) * 2)//这里的限制是1MB-4KB*2
```
>  /drivers/staging/android/binder.c
```c
static int binder_mmap(struct file *filp, struct vm_area_struct *vma)
{
    int ret;
    struct vm_struct *area;
    struct binder_proc *proc = filp->private_data;
    const char *failure_string;
    struct binder_buffer *buffer;

    if (proc->tsk != current)
        return -EINVAL;

    if ((vma->vm_end - vma->vm_start) > SZ_4M)
        vma->vm_end = vma->vm_start + SZ_4M;//如果申请的size大于4MB了，会在驱动中被修改成4MB

    binder_debug(BINDER_DEBUG_OPEN_CLOSE,
             "binder_mmap: %d %lx-%lx (%ld K) vma %lx pagep %lx\n",
             proc->pid, vma->vm_start, vma->vm_end,
             (vma->vm_end - vma->vm_start) / SZ_1K, vma->vm_flags,
             (unsigned long)pgprot_val(vma->vm_page_prot));
```
如果是非`oneway`方式还需要除以2，因为`oneway`方式不需要返回值，所以可以多使用一半的内存。