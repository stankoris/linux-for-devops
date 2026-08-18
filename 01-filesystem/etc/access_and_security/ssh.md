#### `/etc/ssh`

Three different things with similar names are easy to confuse here:

1. **SSH** = protocol, i.e. communication rules
2. **ssh** = the program you use as a client
3. **sshd** = the program that runs on the server and receives SSH connections

**What is SSH (Secure Shell)?**

It's a protocol that defines how two computers securely communicate over a network.

```
your laptop
     |
     | SSH protocol
     |
     v
Linux server
```

SSH roughly says:

- How will we establish a connection?
- How will we encrypt the data?
- How will the server prove who it is?
- How will the user prove who they are?
- How will we send commands afterward?

So, SSH is not one file nor one program. It's a protocol → a set of rules.

**What is ssh?**

When on your laptop you type:

```
ssh ubuntu@123.456.78.99
```

you're running a program called "ssh".

That's the SSH client. Its job is to connect you to the SSH server.

On Linux you can find it with the command:

```
which ssh -> (probably: /usr/bin/ssh)
```

Mentally:

```
ssh
↓
CLIENT
↓
initiates the connection
```

**What is sshd?**

On the server there's another program, "sshd".

The "d" at the end means daemon, i.e. a program that runs in the background for a long time and waits for connections.

```
sshd
↓
SSH daemon
↓
SSH server
```

`/usr/sbin/sshd` is the executable program and it works roughly like this:

```
I've been started.
↓
I'm listening on port 22.
↓
Waiting.
↓
Waiting.
↓
A connection arrived.
↓
Checking the SSH protocol.
↓
Checking authentication.
↓
If everything is OK → allow the session.
```

That's why when you do:

```
ss -tulpn
```

you can see that some process is listening on the SSH port.

**Good analogy**

Imagine a restaurant.

SSH is the rule for how ordering works in the restaurant.

Let's say:

1. a guest arrives
2. the waiter takes the order
3. the guest confirms what they want
4. the kitchen processes the request

That's the protocol.

The ssh client is the guest.

`ssh client → "I want to connect."`

sshd is the waiter standing in the restaurant waiting for guests.

`sshd → "I'm waiting for connections."`

**Now ssh_config vs sshd_config**

`/etc/ssh/ssh_config` is the system-wide configuration file for the SSH client.

```
ssh_config
↓
How should I connect to OTHER machines?
```

`/etc/ssh/sshd_config` is the configuration of the SSH server. I.e., `sshd`.

```
sshd_config
↓
How are OTHERS allowed to connect TO ME?
```

**The most important difference**

Remember one letter:

`d` = daemon.

```
ssh → client program
ssh_config → client

sshd → server daemon
sshd_config → daemon/server
```

**The whole picture**

When from your laptop you do:

```
ssh ubuntu@server
```

**YOUR LAPTOP**
```
--------------------------------

ssh
SSH client program

reads:
~/.ssh/config
/etc/ssh/ssh_config

        |
        | SSH protocol
        | encrypted network connection
        v
```

**SERVER**
```
--------------------------------

sshd
SSH server program

reads:
/etc/ssh/sshd_config
```

And now we arrive at a very important pattern:

```
PROTOCOL → defines communication rules
CLIENT   → initiates communication
SERVER   → receives communication
```

For example:

```
HTTP protocol
curl/browser → HTTP client
Nginx        → HTTP server
```

or:

```
SSH protocol
ssh  → SSH client
sshd → SSH server
```