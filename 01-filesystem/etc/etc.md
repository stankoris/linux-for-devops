### ├── etc

**Why is it called /etc?**

The historical name remains from very early Unix systems. Today it is best understood as:

**system-wide configuration directory**

`/etc` is where Linux and programs on a server mostly store their rules and settings.

Imagine the server is a big building.
In that building you have:
- doors
- security
- elevators
- electricity
- employees
- different offices

And `/etc` is like a cabinet with rulebooks and instructions for the whole building.

For example:
- `/etc/ssh/` tells the SSH server who is allowed to connect, by which method, on which port, whether root is allowed to log in, etc.
- `/etc/nginx/` tells Nginx which sites to host, which ports to listen on, and where to forward requests.
- `/etc/systemd/` contains configurations related to systemd and services.
- `/etc/apt/` tells the package manager where to fetch packages from.

**Important thing!**

`/etc` mostly does not contain the programs themselves.

For example:
`/usr/bin/nginx` would be the program/binary.

While:
`/etc/nginx/nginx.conf` tells that program how it should behave.

Imagine:

```
nginx program
    ↓
"I know how to be a web server."

/etc/nginx/nginx.conf
    ↓
"Here's EXACTLY how you should work on this server."
```

It's not true that all configurations on Linux are located in `/etc`. Rather, most system-wide configurations are located in `/etc`.

A user can have their own configurations in `/home/ubuntu/`. For example:

```
~/.bashrc
~/.ssh/config
~/.gitconfig
```

Those are user-specific settings.

So you have:

```
/etc
↓
rules for the whole system

/home/ubuntu
↓
rules/settings only for the ubuntu user
```