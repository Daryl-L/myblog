#《Operating Systen: Three Easy Pieces》读书笔记 —— 第七章

这一章主要的依然是进程调度的问题。由于前面的调度方法 SJF、RR 等等都有一定的缺点。

SJF 优化了周转时间，但是对应答时间是一场灾难。RR 优化了应答时间，但是对周转时间是一场灾难。

这里提出了一个调度方法 Multi-level Feedback Queue (MLFQ)。可以同时解决两个问题。

这个方法的本质就是通过进程过去的运行规律，来对其将来的运行加以干涉。

## 优先级

建立一些 queue，并且赋予优先级，然后根据进程的运行状态，放到不同的 queue 里。然后根据不同的等级进行调度。

- **规则 1：如果 A 进程优先级高于 B，那么就运行 A。**
- **规则 2：如果 A 进程和 B 进程优先级相同，则使用 RR 来执行 A 和 B 进程。**

我们根据进程占用 CPU 的时间来决定优先级。进程越长，优先级就越低。也就是如果进程是 CPU 密集的，那么它的优先级低，如果进程的 I/O 比较多，那么它的优先级高。可以参考之前的 STCF，一旦 I/O 就会让出 CPU，则就可以将其看成由 I/O 分成的若干个短进程。

## 决定优先级

但是，当一个进程到达 CPU 的时候，我们没有办法知道它的长短。所以一开始并不能决定它的优先级是高还是低，那么开始就直接将它假定是高优先级的进程。同时操作系统会一直观察。

经过一个时间片后，如果当前进程没有让出 CPU，那么就会将这个进程降低一个优先级。如此循环。如果发现在时间片内进程让出 CPU 了，那么就维持进程在当前的优先级不动。

- **规则 3：如果一个任务开始执行，那么久吧就把它放在最高优先级中。**

但是这样做会有一些问题：
1. 如果高优先级的进程一直在运行，那么低优先级的进程就得不到调度，就会变成饥饿状态。
2. 如果开发者通过一些无关紧要的 I/O 让出 CPU 来欺骗调度，那么它（长进程）也会顺利被认为是高优先级进程。
3. 如果长进程的性质发生变更，需要交互而成为短进程，那么这样没有办法修改它的状态。

## 提升优先级

这个方法可以用来解决问题 1 和问题 3。

设置一个时间间隔，操作系统在时间间隔结束后，将所有低优先级的进程重新放置到最高优先级。这样，可以保证低优先级进程不会饥饿，因为它在最高优先级的时候得到了调度。也可以保证，如果低优先级的进程状态改变了，需要 I/O 交互了，它可以运行并让出 CPU 以证明自己。

- **规则 4：在一个时间间隔结束后，将所有的任务都放到最高优先级的 queue 中。**

## 解决问题 2

这个我实在想不出怎么用标题描述，这里解决的是进程通过作弊获取高优先级的问题。

这个问题的关键在于，在一个时间片内让出 CPU 都会被认为是短进程。那么，我们可以给任务分配一个时间片，如果这个任务用完了分配给自己的时间片后，就给它降级，无论它是长任务还是短任务，这样就避免了通过欺骗来得到高优先级的任务。而短任务由于时间很短，会在较长时间之后才会被降级。

- **规则 5：如果一个任务用完了分配给自己的时间，它就会被降级，不管它让出了 CPU 多少次。**