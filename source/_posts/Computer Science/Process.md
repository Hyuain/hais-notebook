---
title: Process
date: 2023-01-18 21:37:30
categories:
  - - 计算机
tags:
  - CX
---

**进程（Process）** 是操作系统运行程序时的主要抽象，**程序** 在CPU上执行的活动叫做 **进程**。在任何时间，进程都可以用其 **机器状态（Machine State）** 来描述。他的机器状态包括：

- 进程 **地址空间（Address Space）** 的内存中的内容
- **CPU 寄存器（CPU Registers）** 中的内容，其中有两个寄存器比较特殊：
	- **程序计数器（Program Counter, PC）**，有时也叫 **指令指针（Instruction Pointer, IP）**，告诉我们程序下一步需要执行的指令
	- **栈指针（Stack Pointer）** 和 对应的 **帧指针（Frame Pointer）** 用来管理存放程序参数、本地变量、返回地址的栈
- **I/O 信息**

# 进程状态

进程有以下三个状态：

- **运行（Running）**：进程在处理器上运行、执行指令
- **就绪（Ready）**：进程准备好运行，但是因为某些原因操作系统在此时不运行它。他可以被操作系统的 **调度器（Scheduler）** **调度（Schedule）** 从而转变到运行态
- **阻塞（Block）**：进程执行了某些操作，导致他现在没有准备好运行，直到某些事件发生。比如，当一个进程向磁盘发起 I/O 请求时，他会进入阻塞，此时别的进程可以使用处理器

# 数据结构

操作系统也是一个程序，他使用一些关键的数据结构来跟踪进程信息，比如说 **进程列表（Process List）**，或者说叫 **任务列表（Task List）**。有时，人们会将存储的单个进程的结构称为 **进程控制块（Process Controller Block, PCB）** 或 **进程描述符（Process Descriptor）**。

下面是 [xv6](https://pdos.csail.mit.edu/6.828/2012/xv6.html) 内核的中采用的[描述进程的数据结构](https://github.com/mit-pdos/xv6-public/blob/HEAD/proc.h#L27-L52)，在其他的操作系统，诸如 Mac、Windows 中也有类似的数据结构：

```c
// the registers xv6 will save and restore
// to stop and subsequently restart a process
struct context {
  uint edi;
  uint esi;
  uint ebx;
  uint ebp;
  uint eip;
};

// the different states a process can be in
enum procstate { UNUSED, EMBRYO, SLEEPING, RUNNABLE, RUNNING, ZOMBIE };

// the information xv6 tracks about each process
// including its register context and state
struct proc {
  uint sz;                     // Size of process memory (bytes)
  pde_t* pgdir;                // Page table
  char *kstack;                // Bottom of kernel stack for this process
  enum procstate state;        // Process state
  int pid;                     // Process ID
  struct proc *parent;         // Parent process
  struct trapframe *tf;        // Trap frame for current syscall
  struct context *context;     // swtch() here to run process
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)
};
```

可以看到 **寄存器上下文（Register Context）** 会为停止的进程保存他寄存器的内容。当进程停止时，寄存器的内容会保存到内存中；当恢复运行该进程时，这些内容会被重新放回寄存器中。这被称为 **上下文切换（Context Switch）**。

# Process API

请参考 [GitHub 仓库](https://github.com/Hyuain/os-practices/tree/main/process) 中的代码。