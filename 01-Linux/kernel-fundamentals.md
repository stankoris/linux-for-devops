# What Is a Kernel?

## The Castle Analogy 🏰

Imagine your computer (or phone) is a **big castle** with many rooms.
Living in that castle are various "residents" --- the programs you use:
games, YouTube, Word, your browser, everything.

Inside that castle lives a **head steward** who: - Decides who gets
access to which room - Hands out food (electricity/resources) fairly to
all residents - Stops two residents from fighting over the same room -
Knows what's happening everywhere in the castle

That steward is the **kernel** --- the most important part of an
operating system (Windows, Android, iOS, Linux...) that manages
everything "behind the scenes."

------------------------------------------------------------------------

## Why Does the Kernel Exist?

A computer has limited resources: - **CPU** (the brain that does the
calculating) - **RAM/Memory** (the desk it works on) - **Hard disk**
(the cabinet for permanent storage)

If every program could do whatever it wanted with no supervision, chaos
would follow: two programs fighting over the same memory, one program
crashing the entire machine, or one program spying on another. The
kernel prevents this by controlling and coordinating access to hardware
and system resources. Normal user-space programs generally request
privileged operations through the kernel rather than controlling
hardware directly.

------------------------------------------------------------------------

## How Does the Kernel Work?

1.  When you open a game, it does **not** talk to the CPU directly.
2.  The game asks the kernel: "I need some memory and some CPU time!"
3.  The kernel decides: "OK, you get this slice, and Chrome gets that
    slice."
4.  The kernel keeps dividing resources between **all** running
    programs, dozens of times per second --- which is why everything
    *feels* like it's happening at once.

------------------------------------------------------------------------

## Where Is the Kernel Used?

Every device you use has a kernel: - **Windows** has its own kernel
(called the **NT kernel**) - **Android** uses the **Linux kernel** -
**iOS and macOS** use the **XNU kernel**

When your computer freezes or shows a crash screen, it's often a sign
the kernel ran into a problem it couldn't recover from.

------------------------------------------------------------------------

## Is the Kernel "Installed Separately" from the OS?

**No.** The kernel is not something added on top of the operating system
--- it **is** a built-in component of it, baked in from the start.

Think of it like a car: you don't say "the engine gets installed with
the car" --- the engine **is part of** the car, built in from the
factory. In the same way, there's no "Windows without a kernel" that
later gets one added --- the kernel exists from the moment the OS is
built.

> A more precise way to say it: **the kernel is a distinct unit *within*
> the operating system, not a separate product that gets bolted on.**

------------------------------------------------------------------------

## User Space vs. Kernel Space

Linux separates normal applications from the privileged kernel.

``` text
User Space
────────────────────
shell
systemd
nginx
databases
applications

        ↓ system calls

Kernel Space
────────────────────
Linux kernel

        ↓

Hardware
```

Most programs run in **user space**. They cannot freely access arbitrary
memory or directly perform privileged hardware operations.

The kernel runs in **kernel space** and manages low-level resources such
as:

-   Processes and CPU scheduling
-   Memory
-   Filesystems
-   Devices and drivers
-   Networking
-   Access control

This separation is fundamental to system stability and security.

------------------------------------------------------------------------

## System Calls

When a user-space program needs the kernel to perform an operation, it
requests it through a **system call**.

Examples include operations such as:

-   Opening and reading files
-   Creating processes
-   Allocating memory
-   Using network sockets

Conceptually:

``` text
Application
    ↓
System call
    ↓
Kernel
    ↓
Resource / hardware
```

------------------------------------------------------------------------

## Drivers and Kernel Modules

The kernel communicates with hardware through **device drivers**.

Some kernel functionality can be loaded dynamically as **kernel
modules**.

List currently loaded modules:

``` bash
lsmod
```

Inspect a module:

``` bash
modinfo <module>
```

------------------------------------------------------------------------

## `/proc` and `/sys`

Linux exposes useful kernel information through virtual filesystems.

`/proc` contains runtime information about processes and the system:

``` bash
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/<PID>/status
```

`/sys` exposes information about devices, drivers, and kernel objects:

``` bash
ls /sys/class/net
```

These are not ordinary collections of permanent files stored on disk.
They are interfaces exposed by the running system.

------------------------------------------------------------------------

## Useful Kernel Commands

Check the running kernel version:

``` bash
uname -r
```

Show more system and kernel information:

``` bash
uname -a
```

Inspect recent kernel messages:

``` bash
dmesg | tail -20
```

On systems that restrict access:

``` bash
sudo dmesg | tail -20
```

`dmesg` is particularly useful when troubleshooting hardware, drivers,
storage, networking, and boot-related problems.

------------------------------------------------------------------------

## Kernel and the Linux Boot Process

A simplified boot sequence looks like this:

``` text
Power on
    ↓
BIOS / UEFI
    ↓
Bootloader
    ↓
Linux kernel
    ↓
systemd (commonly PID 1)
    ↓
Services and other userspace processes
```

The kernel initializes the system and starts the first userspace
process.

On modern Ubuntu systems, that process is normally **systemd**.

This gives us an important distinction:

**The kernel manages low-level system resources. systemd runs in user
space and manages services and other system units.**

------------------------------------------------------------------------

## Linux Kernel vs. Windows Kernel

  -----------------------------------------------------------------------
                          **Windows Kernel (NT)** **Linux Kernel**
  ----------------------- ----------------------- -----------------------
  **Made by**             Only Microsoft          Thousands of developers
                                                  worldwide

  **Source code**         Closed --- you can't    Open --- anyone can
                          see inside              read every line

  **Who can modify it**   No one outside          Anyone, and they can
                          Microsoft               build their own OS from
                                                  it

  **Used in**             Windows PCs, Xbox       Android, servers, Steam
                                                  Deck, smart TVs, even
                                                  satellites
  -----------------------------------------------------------------------

### The Recipe Analogy 🍰

-   **Windows kernel** = a secret KFC recipe. Only they have it, locked
    away --- no one else is allowed to cook with it.
-   **Linux kernel** = your grandma's cake recipe posted online. Anyone
    can take it, tweak it, add chocolate instead of vanilla, and share
    their version.

That's why Ubuntu, Android, Fedora, and Chrome OS can all use the
**same** Linux kernel underneath, yet look and behave completely
differently --- because everything built *on top of* the kernel is
different in each case.

------------------------------------------------------------------------

## Can You Run Just the Kernel by Itself?

The kernel is essential, but the kernel alone is not a complete,
practical operating-system environment. A usable Linux system also needs
**userspace** components such as system libraries, utilities, an init
system, shells, and applications.

Projects such as **Linux From Scratch** show how a complete Linux system
can be built from source components. This includes building the
toolchain and userspace software as well as configuring and building the
Linux kernel.

This is why Linux **distributions** such as Ubuntu, Fedora, and Debian
exist: they combine the Linux kernel with userspace software and package
everything into a usable operating system.

------------------------------------------------------------------------

# What I Should Know

After this topic, I should understand:

-   What the kernel is and why an operating system needs it
-   That the kernel is part of the operating system, not the entire
    operating system
-   The difference between **user space** and **kernel space**
-   What a **system call** is conceptually
-   That the kernel manages processes, CPU scheduling, memory,
    filesystems, devices, and networking
-   What device drivers and kernel modules are
-   How `/proc` exposes runtime process and system information
-   How `/sys` exposes information about devices and kernel objects
-   How to check the running kernel with `uname`
-   How to inspect kernel messages with `dmesg`
-   The basic boot relationship: **bootloader → kernel → systemd →
    services**
-   Why `systemd` can be PID 1 while still being a userspace process
-   Why the Linux kernel alone is not a complete Linux operating system
