# Linux Filesystem Hierarchy

Linux uses a single directory tree that starts at:

```text
/
```

This directory is called the **root directory**.

Everything on the system — configuration files, applications, user files, devices, logs, and runtime information — is organized somewhere below `/`.

The goal is not to memorize every directory and file. The important skill is to understand the **purpose of the main directories** and know where to start looking when you need configuration, logs, application data, user files, or system information.

## Main Directories

```text
/
├── etc     → system-wide configuration
├── var     → variable data that changes during system operation
├── proc    → process and kernel information
├── sys     → kernel and device information
├── dev     → device files
├── home    → users' home directories
├── root    → root user's home directory
├── opt     → optional/additional application software
├── usr     → user-space programs, libraries, and shared data
├── tmp     → temporary files
├── run     → runtime state
└── boot    → files required during the boot process
```

---

## `/etc` — System-wide Configuration

`/etc` contains most system-wide configuration files.

Programs and services usually keep their configuration here, while the executable program itself normally lives somewhere else.

Examples:

```text
/etc/ssh/        → SSH configuration
/etc/nginx/      → Nginx configuration
/etc/systemd/    → systemd configuration
/etc/apt/        → APT package manager configuration
```

A useful mental model is:

```text
program
↓
knows how to perform a job

configuration in /etc
↓
defines how that program should behave on this system
```

User-specific configuration is often stored in the user's home directory instead, for example:

```text
~/.ssh/config
~/.bashrc
~/.gitconfig
```

---

## `/var` — Variable Data

`/var` contains data that changes while the system and its services are running.

Typical examples include:

```text
/var/log/     → logs
/var/lib/     → persistent application state
/var/cache/   → cached data
/var/spool/   → queued data waiting to be processed
/var/tmp/     → temporary files that may live longer than files in /tmp
```

For example, MySQL commonly stores its database state under:

```text
/var/lib/mysql/
```

while many service logs can be found somewhere under:

```text
/var/log/
```

---

## `/proc` — Processes and Kernel Information

`/proc` is a **virtual filesystem**.

Its contents are not normal files stored permanently on disk. The kernel exposes information through it while the system is running.

Examples:

```text
/proc/cpuinfo
/proc/meminfo
/proc/<PID>/
```

This allows tools and administrators to inspect processes and parts of the current kernel state.

---

## `/sys` — Kernel and Device Information

`/sys` is another virtual filesystem provided by the kernel.

It exposes information about:

- devices
- drivers
- kernel subsystems
- hardware relationships

`/proc` and `/sys` both expose kernel information, but `/sys` is especially focused on the kernel's device model and system objects.

---

## `/dev` — Devices

Linux represents many devices as files.

These device files are located under:

```text
/dev
```

Examples include disks, terminals, and special devices.

Common examples:

```text
/dev/null
/dev/zero
/dev/sda
/dev/tty
```

This reflects an important Unix/Linux idea:

> Many system resources can be accessed through file-like interfaces.

---

## `/home` — User Home Directories

Normal users usually have their personal directory under:

```text
/home
```

For example:

```text
/home/ubuntu
/home/stanko
```

A user's home directory commonly contains personal files and user-specific configuration.

---

## `/root` — Root User's Home Directory

The `root` user does not normally use:

```text
/home/root
```

Its home directory is:

```text
/root
```

`root` is the administrative superuser, so its home directory is kept separately from normal users.

---

## `/opt` — Optional Application Software

`/opt` is commonly used for additional or self-contained applications that are not part of the normal distribution-managed filesystem layout.

For example, a custom application could live under:

```text
/opt/myapplication/
```

This is useful for keeping manually deployed application files separate from operating-system files.

---

## `/usr` — Programs, Libraries, and Shared Data

`/usr` contains a large part of the user-space software installed on the system.

Important locations include:

```text
/usr/bin/     → common user commands
/usr/sbin/    → system administration programs
/usr/lib/     → libraries
/usr/local/   → locally installed software
```

Despite its historical name, `/usr` should not be thought of as a directory for individual users' personal files.

---

## `/tmp` — Temporary Files

`/tmp` is used for temporary files created by programs and users.

Files stored here should not be treated as permanent data.

The system may automatically remove them according to its cleanup policy.

---

## `/run` — Runtime State

`/run` contains information that is relevant to the **currently running system**.

Examples can include:

- PID files
- sockets
- service runtime information
- temporary state used by system services

Its contents are normally recreated after boot.

---

## `/boot` — Boot Files

`/boot` contains files required to start the operating system.

Depending on the system, this can include:

- Linux kernel images
- initramfs files
- bootloader-related files

These files participate in the process that eventually starts the Linux kernel and the rest of the operating system.

---

# Mental Model

A useful simplified map is:

```text
/etc
→ How should the system and services behave?

/var
→ What data changes while services operate?

/usr
→ Where is much of the installed user-space software?

/opt
→ Where can additional/custom applications live?

/home
→ Where are normal users' files?

/proc
→ What does the kernel currently know about processes/system state?

/sys
→ What does the kernel expose about devices and system objects?

/dev
→ How are devices exposed to user space?

/run
→ What temporary state belongs to the current boot?

/tmp
→ Where can temporary files be stored?

/boot
→ What files are needed to start Linux?
```

# What I Should Know

After studying this topic, I should be able to:

- explain the purpose of the main Linux directories without memorizing every subdirectory
- know where system-wide configuration is normally stored
- know where to look for logs and persistent service state
- understand that `/proc` and `/sys` are virtual filesystems
- understand why devices appear under `/dev`
- distinguish system-wide configuration from user-specific configuration
- know the general difference between `/usr`, `/opt`, `/var`, and `/run`
- use the filesystem hierarchy as a map when troubleshooting an unfamiliar Linux server
