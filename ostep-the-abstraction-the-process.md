# 《Operating Systen: Three Easy Pieces》读书笔记 —— 第四章

## 进程

这一章，主要介绍了操作系统提供给用户的最基本的元素，进程。

进程主要解决了一个问题：如何给用户营造出一个计算机拥有许多个 CPU 的假象，以同时运行多个程序。

这其实是一种对 CPU 的虚拟化，使用有限的 CPU 虚拟出无限的 CPU 来同时运行不同的程序。

但是，一个 CPU 在同一时刻只能被一个进程所占用。这就意味着操作系统每时每刻都在切换不同的进程，操作系统需要决定下一时刻运行哪个进程，以及保存进程的寄存器数据、堆栈数据等来等待下一次运行。

## 进程的接口

操作系统会抽象出这样一些接口：

- Create: 创建一个进程，也就是运行一个程序。
- Destory: 退出一个进程，销毁进程的上下文。
- Wait: 进程暂停执行，等待某件事情发生后继续执行。
- Miscellaneous Control: 挂起一个进程，以及恢复执行。
- Status: 进程的状态。

## 进程的创建

进程创建时，操作系统会将硬盘上的代码复制到内存中，同时在内存中初始化数据。

早期的操作系统，会在进程创建时，将所有的代码以及数据复制到内存中。现代操作系统，会进行懒加载，也就是只复制需要用到的数据到内存中。

在这之后，操作系统会找到程序的执行点，也就是 C 语言中的 `main` 函数，同时将 `argc` 和 `argv` 存放到栈中，并开辟一块空间叫做堆（`heap`），来存储动态的内容。

同时，操作系统会初始化如 I/O 以及文件描述符等。在 UNIX 系统中，会默认创建三个文件描述符，分别是输入、输出、错误。

最后，在上面的准备工作完成后，操作系统会跳转到 main 函数的地方，开始执行。

## 进程的状态

进程由三个状态，分别是运行、就绪和阻塞。

- 运行 (Running): 进程在这个状态会占据 CPU。
- 就绪 (Ready): 进程在这个状态，说明已经准备好执行，正在等待操作系统的调度。
- 阻塞 (Block): 进程在做诸如 I/O 的阻塞的操作时候，会进入这个状态，这时，操作系统会调度其它进程。

## 进程的数据结构

```c
// the registers xv6 will save and restore
// to stop and subsequently restart a process
struct context {
    int eip;
    int esp;
    int ebx;
    int ecx;
    int edx;
    int esi;
    int edi;
    int ebp;
};

// the different states a process can be in
enum proc_state { UNUSED, EMBRYO, SLEEPING,
                  RUNNABLE, RUNNING, ZOMBIE };
// the information xv6 tracks about each process
// including its register context and state
struct proc {
    char *mem;                  // Start of process memory
    uint sz;                    // Size of process memory
    char *kstack;               // Bottom of kernel stack
                                // for this process
    enum proc_state state;      // Process state
    int pid;                    // Process ID
    struct proc *parent;        // Parent process
    void *chan;                 // If !zero, sleeping on chan
    int killed;                 // If !zero, has been killed
    struct file *ofile[NOFILE]; // Open files
    struct inode *cwd;          // Current directory
    struct context context;     // Switch here to run process
    struct trapframe *tf;       // Trap frame for the
                                // current interrupt
};
```