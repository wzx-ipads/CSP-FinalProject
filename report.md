# CSP Final Project

## Question 2: Memory Management in seL4

**Principle**

* Single protection mechanism: capabilities

**The key to kernel objects - Capabilities**

The term "capability" originated in a 1966 paper by Dennis and Van Horn . Capabilities are used in capability-based access control system which aims to do authorization. A capability (also known as a key in some systems) a communicable, **unforgeable** token of authority. A capability can be thought of as a pair (x, r) where x is the name of an object and r is a set of privileges or rights. The capability should be totally transferable. It doesn't matter who holds the capability. The capability itself is the key to access the corresponding object. In seL4, all kernel resources including thread control blocks, page tables, frames exposed to applications are managed by capability mechanism. The capability in seL4 is a Prima-facie evidence of privilege which is composed of two components: 1) a reference to an object, which is equivalent to the name of an object as described above. 2) a set of access rights (eg. read, write, send, execute). It provide fine-grained access control and reasoning about information flow.

**Kernel objects related to memory management**

* Untyped
* Threads (thread control block)
* Address Space (page table)
* Endpoints (IPC)
* Frames

**Untyped**

In seL4, apart from a small, static amount of kernel memory, all physical memory is managed by user-level in an seL4 system. Excluding the objects used to create the root task, capabilities to all available physical memory are passed to the root task as capabilities to ***untyped*** memory which is a block of contiguous physical memory with a specific size. We can invoke the untyped capabilities with `seL4_Untyped_Retype` . This function can help us to create a new capability from an untyped. The new capability provides the access right to a subset of the memory from the original untyped. Note that the new capability can also be untyped but with a smaller size.

**Mapping**

It is very interesting that , unlike most of the other operating systems, seL4 kernel does not provide virtual memory management except for some basic manipulation on hardware paging structures. It means that user-level must provide services for creating intermediate paging structures, mapping and unmapping pages.  User processes are free to manage their own virtual address spaces with only one restriction: the high part of the virtual address space have already been occupied by the kernel.

**Threads**

As we can learn in any operating systems courses, threads are the basic units for processor scheduling. seL4 provides threads to represent an execution context and manage processor time. Threads in seL4 are realised by thread control block objects (TCBs), one for each kernel thread.

**IPC**

The principle of microkernel is keeping the most basic part of origin kernel in kernel space and moving as many kernel modules as possible to user space as  system servers,  which can reduce the TCB (trusted computing base) and provide strong isolation between these kernel modules because they reside in different address spaces. But doing so is actually trading one problem for another: how to achieve the high performance as before? So the IPC overhead has become the Achilles' Heel of microkernels. In seL4, IPC is facilitated by small kernel objects known as ***endpoints***, which act as general communication ports. Invocations on endpoint objects are used to send and receive IPC messages. Endpoints consist of a queue of threads waiting to send, or waiting to receive messages.

**Faults**

In seL4, faults are modeled as separately programmer-designated “fault handler” threads. In such case, faults are delivered to a user space handler. Each thread has only one fault handler endpoint.  When a thread generates a thread fault, the kernel will block the faulting thread’s execution and attempt to deliver an IPC message across this fault handler endpoint.  The kernel expects that the fault handler will correct the anomaly that ails the faulting thread and then tell the kernel when it is safe to try executing the faulting thread once again by invoking  a reply operation (with `seL4_Reply()`) or `seL4_TCB_Resume()`. A fault triggered by incoherent page table state or incorrect memory accesses by a thread is called VM fault which is like page fault.

**A detailed analysis of memory management in seL4**

TODO: give a detailed analysis using conceptions mentioned above.



## Question 3

### 问题

* 请结合代码和相关文档列出seL4、Fiasco.OC，Zircon三种微内核的提供的用户态系统服务，支持哪些Linux上的系统调用，进而分析其对于Linux上常见应用的支持能力

### 微内核如何提供系统服务？

* A limited number of service primitives are provided by the microkernel; more complex services may be implemented as applications on top of these primitives. In this way, the functionality of the system can be extended without increasing the code and complexity in privileged mode, while still supporting a potentially wide number of services for varied application domains

### Linux

* Linux系统调用表https://filippo.io/linux-syscall-table/
* Linux上的常见应用（后面将分析三种微内核对以下列出的应用的支持能力）
  * 文本编辑类：Vim
  * 文件传输：Rsync
  * 虚拟化：qemu、kvm
  * 其它：Nginx、Apache、memcached、mysql

### SEL4

SEL4提供的内核态的系统服务包括Capability based access control、system calls (主要是send、recv以及yield)、kernel objects以及kernel memory allocation. SEL4提供的system call大部分都是用于ipc的，如果单靠这些内核提供的system call，那么Linux上的系统调用大多数都无法完成，且绝大多数的Linux常见应用都无法得到支持。所以必须要借助于用户态的系统服务

* 提供的用户态系统服务
  * 目前还未发现SEL4的源代码中有提供运行在用户态的服务器
  * 但是有Available userlevel components，包含lwip等

* 支持的Linux系统调用

  * ipc类的系统调用，如signal, send, recv等
  * yield

  * poll

* 对Linux上常见应用的支持能力

  * 支持vim，rsync，Nginx，Apache、mysql，memcached

### Fiasco.OC

Fiasco.OC提供了一个L4 Runtime Environment. The L4Re Runtime Environment is an operating system framework for building systems with real-time, security, safety and virtualization requirements. It consists of the L4Re hypervisor/microkernel and a user-level infrastructure that includes basic services such as program loading and memory management up to virtual machine management. L4Re also provides the environment for applications, including libraries and process-local functionality.

* 提供的用户态系统服务
  * C library with pthreads and shared libraries
  * libstdc++, fully features STL
  * Virtual file-system infrastructure
  * C and C++ environment
  * Client/Server Framework
  * Scriptable program and system management
  * Input/Output drivers
  * Virtual machines / Hypervisor
  * Platform and device management, including ACPI and PCI
  * Device Driver Environment
  * Input/output multiplexing, including graphics

* 支持的Linux系统调用
  * ipc类的系统调用，如signal, send, recv等
* 对Linux上常见应用的支持能力
  * 由于Fiasco.OC提供的用户态系统服务包括Virtual machines / Hypervisor，所以它应该可以支持qemu等虚拟化应用
  * 支持vim，rsync，Nginx，Apache、mysql，memcached

### Zircon

Zircon is the core platform that powers the Fuchsia. Zircon is composed of a microkernel (source in [/zircon/kernel](https://fuchsia.googlesource.com/fuchsia/+/master/zircon/kernel)) as well as a small set of userspace services, drivers, and libraries (source in [/zircon/system/](https://fuchsia.googlesource.com/fuchsia/+/master/zircon/system)) necessary for the system to boot, talk to hardware, load userspace processes and run them, etc. Fuchsia builds a much larger OS on top of this foundation.

The Zircon Kernel provides syscalls to manage processes, threads, virtual memory, inter-process communication, waiting on object state changes, and locking (via futexes).

* 提供的用户态系统服务

  Zircon的用户态系统服务包含在zircon/system/文件夹中，主要包括以下服务：

  * clock
  * audio
  * cmdline
  * disk
  * fs
  * drivers如uart、usb等

* 支持的Linux系统调用

  * ipc类的系统调用，如signal, send, recv等

* 对Linux上常见应用的支持能力

  * 虚拟机应用如qemu等
  * 支持vim，rsync，Nginx，Apache、mysql，memcached



**Reference:**

[1] http://www.cs.cornell.edu/courses/cs513/2005fa/L08.html

[2] https://www.cse.unsw.edu.au/~cs9242/19/lectures/01b-introx6.pdf

[3] https://en.wikipedia.org/wiki/Capability-based_security

[4] https://docs.sel4.systems/

[5] https://ipads.se.sjtu.edu.cn/_media/publications/guatc20.pdf