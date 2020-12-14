## Question 3

请结合代码和相关文档列出seL4、Fiasco.OC，Zircon三种微内核的提供的用户态系统服务，支持哪些Linux上的系统调用，进而分析其对于Linux上常见应用的支持能力。

### 微内核如何提供系统服务？

* A limited number of service primitives are provided by the microkernel; more complexservices may be implemented as applications on top of these primitives. In this way, thefunctionality of the system can be extended without increasing the code and complexityin privileged mode, while still supporting a potentially wide number of services forvaried application domains.

### Linux

* Linux系统调用表https://filippo.io/linux-syscall-table/
* Linux上的常见应用（后面将分析三种微内核对以下列出的应用的支持能力）
  * 文本编辑类：VSCode、Vim、Libreoffice
  * 文件传输：Rsync
  * 浏览器：Chrome、Firefox
  * 电子邮件：Thunder bird
  * 桌面环境：Gnome、KDE
  * 虚拟化：VirtualBox、VMware

### SEL4

SEL4提供的内核态的系统服务包括Capability based access control、system calls (主要是send、recv以及yield)、kernel objects以及kernel memory allocation. SEL4提供的system call大部分都是用于ipc的，如果单靠这些内核提供的system call，那么Linux上的系统调用大多数都无法完成，且绝大多数的Linux常见应用都无法得到支持。所以必须要借助于用户态的系统服务。

* 提供的用户态系统服务
  * 目前还未发现SEL4的源代码中有提供运行在用户态的服务器。
  * 但是有Available userlevel components. 其中有许多库，如lwip等

* 支持的Linux系统调用

  * ipc类的系统调用，如signal, send, recv等
  * yield

  * Poll, poll

* 对Linux上常见应用的支持能力

  * 

### Fiasco.OC

Fiasco.OC提供了一个L4 Runtime Environment. The L4Re Runtime Environment is an operating system framework for building systems with real-time, security, safety and virtualization requirements. It consists of the [L4Re hypervisor/microkernel](https://l4re.org/fiasco/) and a user-level infrastructure that includes basic services such as program loading and memory management up to virtual machine management. L4Re also provides the environment for applications, including libraries and process-local functionality.

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
  * 由于Fiasco.OC提供的用户态系统服务包括Virtual machines / Hypervisor，所以它应该可以支持VMware、VirtualBox等虚拟机软件。

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
  * drivers如uart、usb、virtue等

* 支持的Linux系统调用

  * ipc类的系统调用，如signal, send, recv等

* 对Linux上常见应用的支持能力

  * 虚拟机应用如VMware、VirtualBox等