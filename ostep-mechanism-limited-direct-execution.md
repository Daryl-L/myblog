#《Operating Systen: Three Easy Pieces》读书笔记 —— 第五章

这一章讨论的问题是操作系统如何运行程序，并且掌握 CPU 的控制权。

如上一章所讲，操作系统将程序代码加载到内存中，设置寄存器的值，然后跳转到 `main` 函数执行。这样，进程是直接在 CPU 运行的，这样对于一个进程来说是最高效的。

**但是，对于操作系统来说，需要解决两个问题：如何控制进程以防止其做不该做的事情，如何能够切换正在运行的进程。**

## 限制级的操作

这是解决第一个问题，如何控制进程以防止其做不该做的事情（限制级的操作）。

用户进程是运行在用户空间的，用户空间的运行环境是有限制的，例如其不能进行 I/O 的调用。相对的，这些限制级的操作是需要在内核空间来完成的。一旦用户空间的代码执行了这些操作，就会触发异常。

用于解决这个问题，操作系统引入了系统调用的概念。用户进程在需要限制级的操作时，执行诸如 `write()` 的系统调用函数，来进行内核空间的操作。

系统调用是操作系统在启动阶段就在 CPU 中注册好的一系列程序，这些程序执行了内核空间的操作，每个系统调用都有相应的数字标识。用户程序需要执行限制级的操作时，只需要执行 `trap` 指令将自身陷入陷阱，同时传递系统调用标识来代表需要执行什么操作，之后的一切就由操作系统接管了。这样，既保留了进程直接运行在 CPU 的高效率，又实现了对某些操作的限制。

不过，为了避免应用开发者记忆系统调用的标识，操作系统会定义一系列的系统调用函数，来执行 `trap` 指令，并传递系统调用标识。

一个完整的用户程序进行系统调用的步骤如下：

1. 操作系统在启动的时候初始化一个陷阱列表。
2. 操作系统初始化程序的数据，然后跳转到 `main` 函数的地方，执行 `return-from-trap` 指令切换到用户态。
3. 这时，进程需要进行 I/O 请求，于是执行了一个 I/O 相关的系统调用函数（本质上是执行了 `trap` 指令，传递了 I/O 相关的标识），这时，进程不再控制 CPU，取而代之的是操作系统。
4. 操作系统开始执行这一部分的操作，在处理结束后，操作系统执行 `return-from-trap` 指令切换到用户态，进程继续执行。
5. 进程结束后从 `main` 函数返回，这时也会做一次系统调用 `exit`，操作系统将这个进程的数据清除，整个过程结束。

## 进程的切换

这是解决第二个问题，操作系统如何保持对 CPU 的控制。

当进程正在运行的时候，操作系统一定是没有占据 CPU 的，这时，操作系统无法干涉进程的运行。

### 协作式

如前面所讲，系统调用会陷入内核态。在陷入内核态后，操作系统就可以接管 CPU 了。这时，操作系统便可以干预进程的运行了。协作式就是利用了系统调用或者异常来使操作系统获得 CPU 的控制权来进行进程切换的。

但是，协作式有一个问题：它对进程是完全信任的，也就是它信任程序一定不会长时间占用 CPU（每隔一段时间进行一次系统调用）。但是，并不是所有的程序员都会这么做。那么问题来了，如果程序没有系统调用，那么进程将会一直占据 CPU。这样操作系统就没有机会获得 CPU 的控制权。

### 非协作式。

要解决协作式的问题，需要硬件的支持。

在 CPU 中加入时钟中断。加入时钟中断后，操作系统只需要在启动时注册中断处理程序来处理中断，这样，在每次中断时，操作系统都能重新控制 CPU，无论进程是不是进行了系统调用。

### 进程切换

进程切换主要是对上下文的操作，上下文即进程运行过程中的寄存器状态。

1. A 进程运行的过程中，发生了时钟中断（或者是其它中断、系统调用），CPU 将寄存器压入内核栈，regs(A) -> k-stack(A)。
2. 内核接管了 CPU 的控制权，然后决定接下来运行的是哪个进程。
3. 如果内核决定继续运行 A 进程，执行 `return-from-trap` 指令，CPU 将内核栈弹出到寄存器，regs(A) <- k-stack(A)。
4. 如果内核决定运行 B 进程，内核会将 A 进程的上下文弹出内核栈，proc_t(A) <- k-stack(A)，B 的上下文压入内核栈，k-stack(B) <- proc_t(B)。然后执行 `return-from-trap` 指令，CPU 将内核栈弹出到寄存器，regs(B) <- k-stack(B)。