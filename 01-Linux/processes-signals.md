# Processes & Signals

Processes are one of the core concepts in Linux.

Almost everything that is running on a Linux system exists as a process: a shell session, a web server, a database, a monitoring tool, or an application.

Understanding processes makes it easier to troubleshoot systems, work with services, understand `systemd`, and later work with containers and orchestration platforms.

---

## 1. Program vs Process

A **program** is an executable file stored on disk.

A **process** is a running instance of that program.

For example:

```text
/usr/bin/node
```

is a program.

When it is executed:

```bash
node server.js
```

the kernel creates a process for it.

A process has information associated with it, such as:

- PID
- parent process
- user and group
- CPU usage
- memory usage
- process state
- open files
- environment variables
- current working directory
- scheduling priority

The Linux kernel keeps track of all of this information.

---

## 2. PID — Process ID

Every process receives a unique number called a **PID**.

PID stands for:

```text
Process ID
```

Example:

```bash
ps
```

Output:

```text
PID     TTY          TIME CMD
66098   pts/0    00:00:00 bash
66325   pts/0    00:00:00 ps
```

Here:

```text
bash -> PID 66098
ps   -> PID 66325
```

A PID is how the system identifies a specific process.

For example:

```bash
kill 66098
```

sends a signal to process `66098`.

---

## 3. The Current Shell PID

In Bash:

```bash
echo $$
```

prints the PID of the current shell.

Example:

```text
66098
```

This means the current Bash shell is running as process:

```text
PID 66098
```

It can be inspected with:

```bash
ps -p $$
```

or:

```bash
ps -p $$ -f
```

---

## 4. Parent and Child Processes

Linux processes form a hierarchy.

A process can start another process.

The process that starts another process is the **parent**, while the new process is the **child**.

Example:

```text
sshd
└── bash
    └── htop
```

If an SSH session starts a Bash shell, and Bash starts `htop`, the relationship may look like:

```text
sshd -> bash -> htop
```

Each process has:

```text
PID  = its own Process ID
PPID = Parent Process ID
```

The relationship can be inspected with:

```bash
ps -o pid,ppid,user,state,cmd
```

---

## 5. PID 1

The first userspace process on most modern Linux systems is:

```text
systemd
```

It normally runs as:

```text
PID 1
```

Check it with:

```bash
ps -p 1
```

Example:

```text
PID TTY          TIME CMD
1   ?        00:00:48 systemd
```

A simplified process hierarchy may look like:

```text
kernel
└── systemd (PID 1)
    ├── sshd
    ├── nginx
    ├── mysqld
    └── application services
```

`systemd` is responsible for starting and supervising many services on the system.

---

## 6. `/proc` and Processes

Linux exposes information about running processes through the virtual filesystem:

```text
/proc
```

For every running process, a directory usually exists:

```text
/proc/<PID>
```

For example, if a process has:

```text
PID 4200
```

then:

```text
/proc/4200
```

contains information about that process.

Useful entries include:

```text
/proc/<PID>/status
/proc/<PID>/cmdline
/proc/<PID>/cwd
/proc/<PID>/environ
/proc/<PID>/exe
/proc/<PID>/fd
```

Example:

```bash
cat /proc/4200/status
```

This can show information such as:

```text
Name
State
Pid
PPid
Uid
Gid
VmSize
VmRSS
Threads
```

---

## 7. Why `/proc` Is a Virtual Filesystem

Files under `/proc` are not normal files permanently stored on disk.

The kernel generates their contents dynamically.

For example:

```bash
cat /proc/4200/status
```

is effectively asking:

> Kernel, show me the current information you have about process 4200.

When the process exits:

```text
/proc/4200
```

disappears.

Example:

```bash
sleep 300 &
```

Output:

```text
[1] 4200
```

Now:

```bash
ls /proc | grep 4200
```

may show:

```text
4200
```

After the process exits, that directory is gone.

This is why `/proc` is called a **virtual filesystem**.

---

## 8. Background Processes and Jobs

Normally, a command runs in the foreground:

```bash
sleep 300
```

The shell waits for it to finish.

Adding:

```text
&
```

runs it in the background:

```bash
sleep 300 &
```

Example output:

```text
[1] 4200
```

Here:

```text
1     = shell job number
4200 = PID
```

Shell jobs can be displayed with:

```bash
jobs
```

Example:

```text
[1]+ Running sleep 300 &
```

The job number and PID are not the same thing.

---

## 9. Exit Status

If a command cannot be found:

```bash
sleep300 &
```

the shell tries to execute a program literally named:

```text
sleep300
```

If it does not exist:

```text
sleep300: command not found
```

The job may finish with:

```text
Exit 127
```

Exit status `127` commonly means:

```text
command not found
```

The correct command is:

```bash
sleep 300 &
```

where:

```text
sleep = command
300   = argument
```

---

## 10. `ps`

`ps` displays information about processes.

The basic command:

```bash
ps
```

normally shows processes belonging to the current user that are associated with the current terminal.

Example:

```text
PID     TTY          TIME CMD
66098   pts/0    00:00:00 bash
66325   pts/0    00:00:00 ps
```

This shows:

```text
bash
└── ps
```

inside the current terminal session.

---

## 11. `ps aux`

A common command for inspecting processes across the whole system is:

```bash
ps aux
```

It displays processes belonging to multiple users, including processes without a controlling terminal.

Example:

```text
USER      PID   %CPU %MEM   VSZ      RSS   TTY STAT START TIME COMMAND
mysql     1200    0.5 11.6 1790396 456456   ?   Ssl  ...   ... /usr/sbin/mysqld
ubuntu    2450    0.0  3.2 1764464 127104   ?   Ssl  ...   ... /usr/bin/node server.js
root      3100    0.0  0.3   66252  12016   ?   S    ...   ... nginx: master process
www-data  3101    0.0  0.4   71648  16752   ?   S    ...   ... nginx: worker process
```

Important columns:

```text
USER    user running the process
PID     Process ID
%CPU    CPU usage
%MEM    percentage of physical RAM used
RSS     resident physical memory
STAT    process state
COMMAND command that started the process
```

---

## 12. Reading a Real Process

Example:

```text
ubuntu 2450 0.0 3.2 1764464 127104 ? Ssl 10:15 0:12 /usr/bin/node server.js
```

This can be interpreted as:

```text
USER    ubuntu
PID     2450
CPU     0.0%
MEM     3.2%
RSS     about 127 MB
STATE   Ssl
COMMAND /usr/bin/node server.js
```

The important distinction is:

```text
ubuntu         = Linux user
node server.js = running process
```

The application is not required to have the same name as the service or user that runs it.

---

## 13. `ps` Is a Snapshot

`ps` shows process information at the moment the command is executed.

Example:

```bash
ps aux
```

might show:

```text
mysql 0.5% CPU
node  0.0% CPU
```

Five seconds later, another:

```bash
ps aux
```

could show:

```text
mysql 20% CPU
node   5% CPU
```

The command does not continuously refresh.

This is called a **snapshot**.

Think of `ps` as taking a photograph of the process table.

---

## 14. `top`

`top` continuously refreshes process information.

Run:

```bash
top
```

Unlike `ps`, it keeps updating the screen.

This makes it useful for answering questions such as:

```text
Which process is currently using the most CPU?
Which process is consuming memory?
Is resource usage increasing?
What is happening on the server right now?
```

A useful mental model is:

```text
ps  = photograph
top = live video
```

---

## 15. Sorting in `top`

By default, `top` commonly sorts processes by CPU usage.

This means that a process can exist but not appear near the top of the visible list if it is currently using little CPU.

Inside `top`:

```text
P = sort by CPU
M = sort by memory
```

For example, a mostly idle Node.js application may be much lower in the process list than MySQL even though both are running.

To monitor a specific PID:

```bash
top -p 2450
```

---

## 16. `htop`

`htop` provides similar information to `top`, but with a more interactive interface.

It typically provides:

- CPU meters
- memory meters
- process list
- scrolling
- filtering
- sorting
- process tree view
- signal sending
- nice adjustment

Run:

```bash
htop
```

`htop` is often easier for interactive troubleshooting.

However, `top` is important to know because it is commonly available even on minimal Linux systems where `htop` may not be installed.

A useful comparison is:

```text
ps    -> process snapshot
top   -> live process monitoring
htop  -> interactive live process monitoring
```

---

## 17. System Memory vs Process Memory

There are two different things to distinguish.

### System memory

This describes RAM usage across the entire machine.

`top` shows a memory summary near the top of the interface.

`htop` commonly displays the same idea using a visual memory bar.

### Process memory

Each process also has its own memory usage.

For example:

```text
mysqld     11.7%
node        3.2%
nginx       0.4%
```

The `%MEM` column describes the percentage of physical RAM used by that individual process.

---

## 18. `VIRT`, `RES`, and `SHR`

Process monitoring tools often show:

```text
VIRT
RES
SHR
```

For a basic process investigation, the most useful of these is usually:

```text
RES
```

`RES` means **resident memory**.

It approximately represents how much physical RAM is currently resident for the process.

Example:

```text
VIRT = 1764464 KB
RES  = 127104 KB
MEM  = 3.2%
```

Do not assume that:

```text
VIRT = actual RAM usage
```

Virtual memory and physical resident memory are different concepts.

Memory management should be studied separately in more detail.

---

## 19. Finding Processes

A common approach is:

```bash
ps aux | grep nginx
```

However, the output may also include the `grep` command itself:

```text
grep --color=auto nginx
```

This happens because the command line of `grep` itself contains the word being searched.

One workaround is:

```bash
ps aux | grep '[n]ginx'
```

A cleaner tool for process lookup is:

```bash
pgrep nginx
```

To also show the command line:

```bash
pgrep -a nginx
```

Example:

```text
133921 nginx: master process /usr/sbin/nginx
133924 nginx: worker process
133925 nginx: worker process
```

---

## 20. Process Trees

Processes form parent-child relationships.

A useful command is:

```bash
pstree
```

To include PIDs:

```bash
pstree -p
```

Example:

```text
systemd(1)
└── sshd(700)
    └── sshd(66090)
        └── bash(66098)
            └── htop(66268)
```

This makes it easier to understand where processes came from and how they relate to one another.

---

# Signals

A **signal** is a mechanism used by the kernel or another process to notify or control a process.

Signals can mean things such as:

```text
interrupt
terminate
stop
continue
reload
```

---

## 21. `kill` Does Not Always Mean "Kill"

The command:

```bash
kill PID
```

does not directly mean:

> immediately destroy this process

It means:

> send a signal to this process

By default:

```bash
kill PID
```

sends:

```text
SIGTERM
```

which is signal:

```text
15
```

These are equivalent:

```bash
kill 2450
```

```bash
kill -15 2450
```

```bash
kill -TERM 2450
```

---

## 22. SIGTERM

`SIGTERM` requests that a process terminate cleanly.

Conceptually:

```text
Please shut down.
```

A process can use the opportunity to:

- finish current work
- close files
- close network connections
- close database connections
- flush buffered data
- release resources

This is called a **graceful shutdown**.

For normal process termination, `SIGTERM` should usually be attempted before `SIGKILL`.

---

## 23. SIGKILL

`SIGKILL` is signal:

```text
9
```

Example:

```bash
kill -9 PID
```

The kernel immediately terminates the process.

The process cannot catch, ignore, or handle `SIGKILL`.

This means it cannot perform normal cleanup.

A useful operational rule is:

```text
SIGTERM
   ↓
wait
   ↓
if the process does not respond
   ↓
SIGKILL
```

`kill -9` should not be the default way to stop processes.

---

## 24. SIGINT

`SIGINT` is signal:

```text
2
```

It is commonly generated when pressing:

```text
Ctrl+C
```

For example:

```bash
sleep 300
```

then:

```text
Ctrl+C
```

causes the foreground process to receive `SIGINT`.

---

## 25. SIGSTOP and SIGCONT

`SIGSTOP` pauses a process.

With the standard Bash/Linux `kill` implementations, signals can be specified by name:

```bash
kill -STOP 4200
```

The process may then appear in state:

```text
T
```

Check:

```bash
ps -o pid,state,cmd -p 4200
```

Resume it with:

```bash
kill -CONT 4200
```

This sends:

```text
SIGCONT
```

and the process continues execution.

---

## 26. Common Signals

The signal numbers below are the conventional values used on common Linux architectures such as x86 and ARM. Using signal names in commands is generally clearer than relying on numeric values.

| Signal | Number | Purpose |
|---|---:|---|
| `SIGHUP` | 1 | Hangup; commonly used by some daemons to reload configuration |
| `SIGINT` | 2 | Interrupt; commonly generated by `Ctrl+C` |
| `SIGKILL` | 9 | Immediate forced termination |
| `SIGTERM` | 15 | Request graceful termination |
| `SIGCONT` | 18 | Continue a stopped process |
| `SIGSTOP` | 19 | Stop/pause a process |

List available signals with:

```bash
kill -l
```

---

## 27. Process States

Processes can exist in different states.

Common states include:

```text
R
S
D
T
Z
I
```

### `R` — Running / Runnable

The process is executing or ready to receive CPU time.

### `S` — Interruptible Sleep

The process is waiting for an event.

This is extremely common for server applications.

For example, a web server may sleep while waiting for a request.

### `D` — Uninterruptible Sleep

The process is usually waiting for I/O.

Persistent processes in this state can indicate storage or I/O problems.

### `T` — Stopped

The process has been paused.

For example:

```bash
kill -STOP PID
```

### `Z` — Zombie

The process has finished execution, but its parent has not yet collected its exit status.

The process is no longer doing useful work, but an entry remains in the process table until the parent handles it.

### `I` — Idle Kernel Thread

Commonly seen for idle kernel worker threads.

---

## 28. `nice` and Process Priority

The Linux scheduler decides which runnable processes receive CPU time.

A process can have a **nice value**.

Typical range:

```text
-20 ... 19
```

Conceptually:

```text
-20 = higher scheduling priority
  0 = normal
 19 = lower scheduling priority
```

Start a process with a specific nice value:

```bash
nice -n 10 command
```

Change an existing process:

```bash
renice -n 10 -p PID
```

A higher nice value means the process is being "nicer" to other processes by accepting a lower CPU scheduling priority.

---

# Practical Troubleshooting Workflow

The purpose of process tools is not to memorize commands.

They are used to answer operational questions.

For example:

```text
The server is slow.
```

A useful workflow could be:

```text
1. top / htop
   ↓
What is happening right now?

2. Identify the PID
   ↓
Which process is consuming resources?

3. ps
   ↓
What is this process and which user runs it?

4. pstree
   ↓
Who started it and where does it belong in the process hierarchy?

5. /proc/<PID>
   ↓
What does the kernel know about the process?

6. logs
   ↓
Why is the application behaving this way?

7. signals / service management
   ↓
Can it be stopped or restarted safely?
```

Example:

```bash
top
```

A process is identified:

```text
PID 1200 mysqld 95% CPU
```

Inspect it:

```bash
ps -p 1200 -f
```

Check its status:

```bash
cat /proc/1200/status
```

Inspect its place in the process tree:

```bash
pstree -p
```

The important question is not:

> Which command should I memorize?

The important question is:

> What is this process, who started it, what resources is it using, and why?

---

# Practical Lab

## 1. Inspect the Current Shell

```bash
echo $$
```

Then:

```bash
ps -p $$ -f
```

Inspect the kernel's process information:

```bash
cat /proc/$$/status
```

---

## 2. Create a Background Process

```bash
sleep 300 &
```

Example:

```text
[1] 4200
```

Check shell jobs:

```bash
jobs
```

Find the process:

```bash
pgrep -a sleep
```

Check `/proc`:

```bash
ls /proc | grep 4200
```

---

## 3. Inspect the Process

```bash
ps -o pid,ppid,user,state,ni,%cpu,%mem,cmd -p 4200
```

Inspect its kernel status:

```bash
cat /proc/4200/status
```

---

## 4. Pause the Process

```bash
kill -STOP 4200
```

Check its state:

```bash
ps -o pid,state,cmd -p 4200
```

Expected state:

```text
T
```

---

## 5. Resume the Process

```bash
kill -CONT 4200
```

Check again:

```bash
ps -o pid,state,cmd -p 4200
```

---

## 6. Terminate the Process Gracefully

```bash
kill 4200
```

This sends:

```text
SIGTERM
```

Verify that it disappeared:

```bash
ps -p 4200
```

and:

```bash
ls /proc | grep 4200
```

---

# What I Should Know

After this topic, I should be able to explain and demonstrate:

- The difference between a program and a process.
- What PID and PPID represent.
- How parent and child processes form a process hierarchy.
- Why PID 1 is special on a modern Linux system.
- What `echo $$` returns.
- Why `/proc/<PID>` exists.
- Why `/proc` is called a virtual filesystem.
- The difference between a shell job number and a PID.
- The difference between foreground and background processes.
- Why `ps` normally shows only processes from the current terminal.
- Why `ps aux` shows a much broader system-wide process view.
- The meaning of the most useful `ps aux` columns: `USER`, `PID`, `%CPU`, `%MEM`, `RSS`, `STAT`, and `COMMAND`.
- Why `ps` is considered a snapshot.
- Why `top` and `htop` are useful for live troubleshooting.
- The practical difference between `ps`, `top`, and `htop`.
- How to sort `top` by CPU and memory.
- The difference between system memory and per-process memory.
- Why `VIRT` should not be interpreted as actual physical RAM usage.
- How to find processes using `pgrep`.
- Why `ps aux | grep process` can show the `grep` process itself.
- How to inspect process relationships using `pstree`.
- What a signal is.
- Why `kill` is really a signal-sending command.
- The difference between `SIGTERM` and `SIGKILL`.
- Why `SIGTERM` should normally be attempted before `SIGKILL`.
- What `SIGINT`, `SIGSTOP`, and `SIGCONT` do.
- The basic meaning of common process states such as `R`, `S`, `D`, `T`, `Z`, and `I`.
- What `nice` and `renice` are used for.
- How to investigate a process using `ps`, `top`, `pstree`, and `/proc`.
- How process inspection helps troubleshoot real Linux servers.
