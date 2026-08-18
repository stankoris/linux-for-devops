# `/etc/sudoers`

`/etc/sudoers` is one of the most important files in Linux access control.

It tells the system:

- who is allowed to use `sudo`
- which user they are allowed to run commands as
- which commands they are allowed to run with elevated privileges

## A Simple Analogy

Imagine a company.

You are a regular employee called `ubuntu`. You are not allowed to change everything on the server.

But the company says:

> "`ubuntu` is allowed to temporarily enter the administrator's room when necessary."

That temporary administrator pass is:

```bash
sudo
```

The rules that define who has that pass and what they are allowed to do with it are stored in:

```text
/etc/sudoers
```

---

## What Does `sudo` Actually Do?

Suppose you try to run:

```bash
apt update
```

as a regular user.

You will likely get a permission-related error.

Why?

Because modifying package state requires elevated privileges.

But when you run:

```bash
sudo apt update
```

it roughly means:

> "Run `apt update` with the privileges allowed by the sudo policy."

Most commonly, this means running the command as:

```text
root
```

The flow looks like this:

```text
ubuntu
  ↓
sudo
  ↓
policy check
  ↓
allowed?
  ↓
run the command as root
```

---

## Where Does `/etc/sudoers` Fit In?

When you run:

```bash
sudo apt update
```

`sudo` first has to check:

> "Is the `ubuntu` user actually allowed to do this?"

The main sudo policy is usually configured in:

```text
/etc/sudoers
```

A common Ubuntu sudoers rule looks like this:

```text
%sudo ALL=(ALL:ALL) ALL
```

---

## Understanding `%sudo ALL=(ALL:ALL) ALL`

At first, this line looks complicated:

```text
%sudo ALL=(ALL:ALL) ALL
```

Let's break it down.

### `%sudo`

The `%` symbol means:

```text
group
```

So:

```text
%sudo
```

means:

> all members of the `sudo` group

---

### First `ALL`

The first:

```text
ALL
```

refers to hosts.

---

### `(ALL:ALL)`

This means that matching users are allowed to run commands as any permitted user and group.

Most of the time, you use:

```bash
sudo command
```

which usually means running the command as `root`.

But `sudo` can also run a command as another user.

For example:

```bash
sudo -u mysql command
```

means:

> run the command as the `mysql` user

So `sudo` does not simply mean:

```text
become root
```

A more accurate mental model is:

```text
run a command as another user, if the sudo policy allows it
```

That distinction is important.

---

### Final `ALL`

The final:

```text
ALL
```

means:

```text
all commands
```

So, for now, you can read:

```text
%sudo ALL=(ALL:ALL) ALL
```

as:

> Members of the `sudo` group are allowed to run any command with sudo privileges.

---

## Why Not Work as `root` All the Time?

It is safer to use a regular user such as:

```text
ubuntu
```

for normal daily work.

If you accidentally run:

```bash
rm something
```

the potential damage is more limited by the Linux permission model.

Only when you actually need administrative privileges do you use:

```bash
sudo ...
```

This makes privilege escalation intentional and visible.

This follows an important security principle:

## Least Privilege

Give a user or process only the permissions it actually needs.

---

# `/etc/sudoers.d/`

Instead of putting every sudo rule directly inside:

```text
/etc/sudoers
```

Linux systems can also use:

```text
/etc/sudoers.d/
```

This directory can contain separate sudo policy files.

For example:

```text
/etc/sudoers.d/deploy
```

could contain rules for a deployment user.

This keeps the configuration cleaner and can make automation easier.

---

## Example: Limited Deployment Permissions

Imagine that one day you create a user called:

```text
deploy
```

You want your CI/CD system to be able to restart your application, but you do not want the `deploy` user to have full root access.

Instead of:

```text
deploy → can do everything
```

you could conceptually define a rule like:

```text
deploy ALL=(root) /usr/bin/systemctl restart firewallmindset.service
```

This means:

> `deploy` can use `sudo` to run this specific administrative command.

It would not be allowed to freely run commands such as:

```bash
sudo rm -rf /
```

It would also not be allowed to arbitrarily:

- modify the firewall
- create new root-level users

This is another practical example of the **least privilege** principle.