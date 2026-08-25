# Linux Users, Groups and Permissions

Linux is a multi-user operating system.

Even if only one person administrates a server, the system usually contains many different users. Some represent real people, while others exist only so applications and services can run with limited privileges.

This guide explains how Linux users, groups, authentication, ownership, and permissions work by building a small practical lab on an Ubuntu server.

The lab starts with the default `ubuntu` administrative user and gradually creates a new user named `alice`, a new group named `developers`, SSH access for Alice, and a group-controlled project directory.

---

# 1. What Linux Actually Sees as a User

Humans normally identify users by names:

```text
ubuntu
alice
root
www-data
```

Linux internally relies primarily on numeric identifiers.

For users, that identifier is called a:

```text
UID = User ID
```

For groups:

```text
GID = Group ID
```

For example:

```text
ubuntu -> UID 1000
root   -> UID 0
```

The username is the human-readable name associated with a UID.

This matters because files and processes are ultimately associated with numeric user and group IDs.

For example, when you see:

```text
ubuntu ubuntu
```

in `ls -l`, Linux is mapping stored numeric UID and GID values back to readable names.

The `root` user is special:

```text
UID 0 = root
```

UID `0` represents the superuser account with unrestricted administrative privileges.

---

# 2. Check Your Current Identity

Start by checking which user you are currently using:

```bash
whoami
```

On a typical Ubuntu AWS server, the result may be:

```text
ubuntu
```

Now run:

```bash
id
```

Example:

```text
uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),4(adm),27(sudo)
```

The output contains several important pieces of information.

```text
uid=1000(ubuntu)
```

means:

```text
username = ubuntu
UID      = 1000
```

This part:

```text
gid=1000(ubuntu)
```

shows the user's **primary group**.

And:

```text
groups=1000(ubuntu),4(adm),27(sudo)
```

shows all supplementary groups the current user belongs to.

A user has:

- one primary group
- zero or more supplementary groups

For example:

```text
ubuntu
├── primary group: ubuntu
├── supplementary group: adm
└── supplementary group: sudo
```

The `sudo` group is particularly important on Ubuntu because its members are normally allowed to execute administrative commands through `sudo`.

---

# 3. `/etc/passwd`

Linux keeps local account information in:

```text
/etc/passwd
```

Display it with:

```bash
cat /etc/passwd
```

You will see many lines similar to:

```text
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
```

Each line represents one account.

The format is:

```text
username:password-field:UID:GID:comment:home-directory:login-shell
```

For example:

```text
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
```

can be read as:

```text
ubuntu          username
x               password information is stored elsewhere
1000            UID
1000            primary GID
Ubuntu          comment / GECOS field
/home/ubuntu    home directory
/bin/bash       login shell
```

The `x` is **not the user's password**.

It indicates that password authentication information is stored in another protected file:

```text
/etc/shadow
```

## Why is `/etc/passwd` readable by normal users?

Check its permissions:

```bash
ls -l /etc/passwd
```

You will normally see permissions similar to:

```text
-rw-r--r-- 1 root root ...
```

Normal users can read this file.

That is intentional.

Many programs need to translate numeric UIDs and GIDs into readable usernames and group names.

For example, if a file is internally owned by UID `1000`, the system needs a way to determine that UID `1000` corresponds to the user `ubuntu`.

Because `/etc/passwd` must be readable, sensitive password hashes should not be stored there.

That is the purpose of `/etc/shadow`.

---

# 4. `/etc/shadow`

Password authentication information for local accounts is stored in:

```text
/etc/shadow
```

Try to read it as a normal user:

```bash
cat /etc/shadow
```

You should receive:

```text
Permission denied
```

Now use administrative privileges:

```bash
sudo cat /etc/shadow
```

This works because `sudo` executes the command with elevated privileges.

Check the file permissions:

```bash
ls -l /etc/shadow
```

Unlike `/etc/passwd`, `/etc/shadow` is protected from ordinary users.

A simplified entry may look like:

```text
ubuntu:$y$...:...
```

The password itself is not stored in plaintext.

Linux stores a **password hash**.

Conceptually:

```text
password
   |
   v
password hashing algorithm
   |
   v
stored password hash
```

When a user later enters a password, Linux does not decrypt the stored hash.

Instead, the entered password is processed using the appropriate password hashing scheme and compared with the stored authentication data.

A password hash is therefore not the same thing as encrypted text.

There is no normal operation like:

```text
hash -> decode -> original password
```

However, weak passwords can sometimes be discovered by repeatedly guessing possible passwords and checking whether a candidate matches the stored hash.

That is one reason `/etc/shadow` is sensitive even though it does not contain plaintext passwords.

If an administrator needs to replace a forgotten password, the normal solution is to set a new one:

```bash
sudo passwd <username>
```

---

# 5. Service Users

Not every Linux user represents a human.

Many applications and services run under dedicated accounts.

A common example on Ubuntu systems running Nginx is:

```text
www-data
```

You can check whether the account exists with:

```bash
getent passwd www-data
```

A typical result looks similar to:

```text
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

Nginx worker processes commonly run as `www-data`.

You can inspect running processes with:

```bash
ps aux
```

or specifically search for Nginx:

```bash
ps aux | grep nginx
```

Why not run every application as `root`?

Because a process inherits the privileges of the user running it.

If a vulnerable web application runs as `root` and becomes compromised, the attacker may gain extremely powerful access to the system.

If the service runs under a restricted account such as `www-data`, the process has only the permissions assigned to that user.

This follows the:

```text
Principle of Least Privilege
```

The idea is simple:

> A user or service should receive only the privileges it actually needs.

---

# 6. Why Do Some Users Have `/usr/sbin/nologin`?

Look again at a service account:

```bash
getent passwd www-data
```

You may see:

```text
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

The final field is the login shell:

```text
/usr/sbin/nologin
```

This means the account exists as a Linux identity but is not intended for normal interactive login.

The account can still:

- own files
- own directories
- run processes
- belong to groups
- receive filesystem permissions
- have a UID and GID

But it is not meant to log in and receive an interactive shell like:

```text
/bin/bash
```

This is common for service users.

---

# 7. Build a Small User Lab

Now create a real test user.

We will use only one user:

```text
alice
```

The goal is to gradually give Alice:

- a Linux account
- a home directory
- a normal Bash shell
- optional password authentication
- her own SSH key
- membership in a new group
- access to a group-controlled directory

Do not modify your main `ubuntu` account while experimenting.

---

## Step 1 — Create Alice

Create the user:

```bash
sudo useradd -m alice
```

The `-m` option tells `useradd` to create a home directory.

Verify:

```bash
id alice
```

Example:

```text
uid=1002(alice) gid=1002(alice) groups=1002(alice)
```

Alice now exists as a Linux identity.

Check the account database:

```bash
getent passwd alice
```

You may see something similar to:

```text
alice:x:1002:1002::/home/alice:/bin/sh
```

Alice now has:

```text
username: alice
UID:      1002
GID:      1002
home:     /home/alice
shell:    /bin/sh
```

The exact UID and GID numbers may be different on your system.

---

## Step 2 — Give Alice Bash

For an interactive human account, Bash is often more convenient than `/bin/sh`.

Set Alice's login shell:

```bash
sudo usermod -s /bin/bash alice
```

Verify:

```bash
getent passwd alice
```

You should now see:

```text
alice:x:1002:1002::/home/alice:/bin/bash
```

Alice now exists as a normal Linux user with a home directory and interactive shell.

However, we still have not configured how she will authenticate.

---

# 8. Create a Group

Now create a new group:

```bash
sudo groupadd developers
```

Verify it:

```bash
getent group developers
```

Example:

```text
developers:x:1005:
```

The format of `/etc/group` entries is:

```text
group-name:password-field:GID:members
```

At this point the group exists, but Alice is not yet a member.

Add Alice:

```bash
sudo usermod -aG developers alice
```

Verify:

```bash
id alice
```

Example:

```text
uid=1002(alice) gid=1002(alice) groups=1002(alice),1005(developers)
```

Alice now has:

```text
primary group:
alice

supplementary group:
developers
```

You can also inspect the group directly:

```bash
getent group developers
```

Now you should see something similar to:

```text
developers:x:1005:alice
```

## Why `-aG`?

The command:

```bash
sudo usermod -aG developers alice
```

contains two important options:

```text
-G = set supplementary groups
-a = append
```

Together:

```text
-aG
```

means:

> Add this group to Alice's existing supplementary groups.

Be careful with:

```bash
sudo usermod -G developers alice
```

Without `-a`, existing supplementary group memberships can be replaced.

---

# 9. Understand the Relationship Between the User Databases

Now inspect Alice in several different ways.

```bash
id alice
```

```bash
getent passwd alice
```

```bash
getent group developers
```

These commands answer different questions.

`id alice` shows the identity Linux currently knows for Alice:

```text
UID
primary GID
group memberships
```

`getent passwd alice` asks the configured system identity databases for Alice's account information.

For a local account, it may return information that comes from `/etc/passwd`.

For example:

```text
alice:x:1002:1002::/home/alice:/bin/bash
```

`getent group developers` returns group information:

```text
developers:x:1005:alice
```

There is an important conceptual difference between:

```bash
grep alice /etc/passwd
```

and:

```bash
getent passwd alice
```

`grep` searches one specific file.

`getent` asks the system's configured identity sources.

This becomes important in larger environments where accounts may come from services such as LDAP or Active Directory integrations rather than only local files.

At this point Alice exists as a Linux identity:

```text
alice
├── UID
├── primary GID
├── supplementary group: developers
├── home: /home/alice
└── shell: /bin/bash
```

Now we can configure authentication.

---

# 10. Give Alice a Password

Alice currently exists, but if we created her with:

```bash
sudo useradd -m alice
```

we did not interactively configure a password.

Set one with:

```bash
sudo passwd alice
```

You will be asked to enter the new password twice.

Example:

```text
New password:
Retype new password:
passwd: password updated successfully
```

Alice now has password authentication data stored in `/etc/shadow`.

You can confirm that an entry exists:

```bash
sudo grep '^alice:' /etc/shadow
```

You will see a password hash, not the plaintext password.

---

## When Does Alice Actually Need a Password?

This is important.

A Linux account does **not always need a password**.

For example, if Alice logs in exclusively with an SSH key and never uses password-based authentication, a password may not be necessary.

A password becomes useful when the system needs to authenticate Alice using her password.

Examples can include:

- local password login
- password-based SSH login, if the SSH server allows it
- `sudo`, if Alice is authorized to use `sudo`

Setting a password does **not** automatically enable password-based SSH authentication.

The SSH server may have password authentication disabled in its configuration.

Also:

```text
having a password != having sudo privileges
```

Authentication answers:

> Who are you?

Authorization answers:

> What are you allowed to do?

Alice can have a valid password but still have no administrative privileges.

Check her groups:

```bash
id alice
```

If `sudo` is not listed, Alice is not receiving sudo privileges through the Ubuntu `sudo` group.

If she runs:

```bash
sudo whoami
```

having the correct Alice password alone is not enough if she is not authorized to use `sudo`.

For this lab, keep Alice as a normal non-administrative user.

Use the `ubuntu` account for administration.

---

# 11. Configure a New SSH Key for Alice

Now give Alice her own SSH key.

SSH key authentication uses a key pair:

```text
private key
public key
```

The private key stays on the client computer.

The public key is copied to the server.

Conceptually:

```text
Your computer
├── private key
└── public key
        |
        v
Server
└── /home/alice/.ssh/authorized_keys
```

Never upload the private key to the server.

---

## Step 1 — Generate Alice's SSH Key on Your Own Computer

Run this on your **local computer**, not on the VPS.

### Linux / macOS

```bash
ssh-keygen -t ed25519 -f ~/.ssh/alice-aws
```

This creates:

```text
~/.ssh/alice-aws
~/.ssh/alice-aws.pub
```

The file:

```text
alice-aws
```

is the private key.

The file:

```text
alice-aws.pub
```

is the public key.

Display the public key:

```bash
cat ~/.ssh/alice-aws.pub
```

Copy the complete line.

---

### Windows PowerShell

Generate the key:

```powershell
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\alice-aws"
```

This creates:

```text
C:\Users\<username>\.ssh\alice-aws
C:\Users\<username>\.ssh\alice-aws.pub
```

Display the public key:

```powershell
Get-Content "$env:USERPROFILE\.ssh\alice-aws.pub"
```

Copy the complete public key line.

---

## Step 2 — Create Alice's `.ssh` Directory on the Server

Back on the Ubuntu server, while logged in as your administrative `ubuntu` user:

```bash
sudo mkdir -p /home/alice/.ssh
```

Because the command was executed through `sudo`, verify ownership:

```bash
ls -ld /home/alice/.ssh
```

> You may need to run "sudo ls -ld /home/alice/.ssh"

We want Alice to own her own SSH directory.

Set the ownership:

```bash
sudo chown alice:alice /home/alice/.ssh
```

Set secure permissions:

```bash
sudo chmod 700 /home/alice/.ssh
```

`700` means:

```text
owner:  rwx
group:  ---
others: ---
```

Only Alice should have access to this directory.

---

## Step 3 — Add Alice's Public Key

Open Alice's `authorized_keys` file:

```bash
sudo nano /home/alice/.ssh/authorized_keys
```

Paste the **public** key generated on your local computer.

Save and exit.

Now set ownership:

```bash
sudo chown alice:alice /home/alice/.ssh/authorized_keys
```

Set permissions:

```bash
sudo chmod 600 /home/alice/.ssh/authorized_keys
```

`600` means:

```text
owner:  rw-
group:  ---
others: ---
```

Verify the complete setup:

```bash
ls -la /home/alice/.ssh
```

You want something similar to:

```text
drwx------ alice alice .ssh
-rw------- alice alice authorized_keys
```

---

## Step 4 — Test Alice's SSH Login

From your local computer:

### Linux / macOS

```bash
ssh -i ~/.ssh/alice-aws alice@<server-public-ip>
```

### Windows PowerShell

```powershell
ssh -i "$env:USERPROFILE\.ssh\alice-aws" alice@<server-public-ip>
```

After logging in, verify:

```bash
whoami
```

Expected:

```text
alice
```

Then:

```bash
id
```

You should see Alice's UID, primary group, and the `developers` supplementary group.

For example:

```text
uid=1002(alice) gid=1002(alice) groups=1002(alice),1005(developers)
```

Alice now has her own independent SSH access.

---

# 12. Understand Ownership and Permissions

Before creating the project directory, understand how Linux represents filesystem access.

Every file and directory has:

```text
object
├── owner
├── group
└── permissions
    ├── owner permissions
    ├── group permissions
    └── others permissions
```

For example:

```text
drwxrwx--- root developers dev-project
```

contains two different concepts:

```text
ownership:
owner = root
group = developers
```

and:

```text
permissions:
owner  = rwx
group  = rwx
others = ---
```

Group ownership alone does not automatically give group members access.

The group permission bits must also allow the required operation.

---

# 13. `r`, `w`, and `x`

Linux uses three basic permission bits:

```text
r = read
w = write
x = execute
```

They are shown for:

```text
owner
group
others
```

Example:

```text
rwxr-x---
```

Break it into three sets:

```text
rwx | r-x | ---
owner group others
```

Meaning:

```text
owner  -> read, write, execute
group  -> read, execute
others -> no permissions
```

---

## Permissions on Files

For regular files:

```text
r = read the file
w = modify the file
x = execute the file as a program/script when applicable
```

---

## Permissions on Directories

For directories, the meaning is slightly different.

```text
r = list directory entries
w = create/delete/rename entries in the directory
x = traverse or enter the directory
```

A useful mental model for directory `x` is:

> Permission to pass through or enter the directory.

For example:

```bash
cd /home/dev-project
```

requires execute/traverse permission on the directories in the path.

---

# 14. Numeric Permissions

Linux permissions are commonly represented using numbers.

```text
r = 4
w = 2
x = 1
```

Add the values:

```text
7 = 4 + 2 + 1 = rwx
6 = 4 + 2     = rw-
5 = 4 + 1     = r-x
4             = r--
0             = ---
```

For example:

```bash
chmod 770 directory
```

means:

```text
7 -> owner  -> rwx
7 -> group  -> rwx
0 -> others -> ---
```

Result:

```text
rwxrwx---
```

---

# 15. Create a Group-Controlled Project Directory

Now use everything together.

Create a project directory:

```bash
sudo mkdir /home/dev-project
```

Check it:

```bash
ls -ld /home/dev-project
```

Because `ubuntu` created it through `sudo`, it will normally be owned by:

```text
root root
```

Now assign the directory to the `developers` group:

```bash
sudo chown root:developers /home/dev-project
```

Verify:

```bash
ls -ld /home/dev-project
```

You should now see:

```text
root developers
```

But this alone does not guarantee that Alice can work inside it.

We also need appropriate group permissions.

Set:

```bash
sudo chmod 770 /home/dev-project
```

Check again:

```bash
ls -ld /home/dev-project
```

Example:

```text
drwxrwx--- 2 root developers ... /home/dev-project
```

Break it down:

```text
d
```

means this object is a directory.

Then:

```text
rwx
```

belongs to the owner:

```text
root
```

The next:

```text
rwx
```

belongs to the group:

```text
developers
```

And:

```text
---
```

belongs to everyone else.

The complete model is:

```text
/home/dev-project
├── owner: root
├── group: developers
└── permissions
    ├── owner:  rwx
    ├── group:  rwx
    └── others: ---
```

Alice is not the owner.

But Alice belongs to:

```text
developers
```

Therefore Linux evaluates the group permission set for her:

```text
rwx
```

Alice can enter the directory and create files.

---

# 16. Test the Directory as Alice

Log in through Alice's SSH key:

```bash
ssh -i <alice-private-key> alice@<server-public-ip>
```

Enter the project directory:

```bash
cd /home/dev-project
```

Create a file:

```bash
touch alice-file.txt
```

List the contents:

```bash
ls -l
```

If the command works, Alice received access through the `developers` group.

This is the important chain:

```text
alice
   |
   v
member of developers
   |
   v
/home/dev-project group = developers
   |
   v
group permissions = rwx
   |
   v
Alice can access and modify the directory
```

A user who is neither:

- the owner
- nor a member of `developers`

would receive the:

```text
others
```

permission set:

```text
---
```

and would not be able to access the directory.

---

# 17. One Important Detail About Newly Created Files

If Alice creates:

```bash
touch /home/dev-project/alice-file.txt
```

the new file may still use Alice's **primary group**:

```text
alice
```

rather than automatically inheriting:

```text
developers
```

For a true multi-user shared project directory, Linux provides additional mechanisms such as the SGID directory bit and appropriate `umask` settings.

Those are separate concepts and should be learned after the basic user/group/permission model is clear.

For this lab, the important point is that Alice gained access to the directory because:

```text
Alice belongs to developers
+
the directory belongs to developers
+
developers has rwx permissions
```

---

# 18. Useful Commands From This Lab

Check the current user:

```bash
whoami
```

Inspect identity:

```bash
id
```

Inspect Alice:

```bash
id alice
getent passwd alice
```

Inspect a group:

```bash
getent group developers
```

Create Alice:

```bash
sudo useradd -m alice
```

Set Bash as Alice's shell:

```bash
sudo usermod -s /bin/bash alice
```

Set or replace Alice's password:

```bash
sudo passwd alice
```

Create a group:

```bash
sudo groupadd developers
```

Add Alice to the group:

```bash
sudo usermod -aG developers alice
```

Create Alice's SSH directory:

```bash
sudo mkdir -p /home/alice/.ssh
sudo chown alice:alice /home/alice/.ssh
sudo chmod 700 /home/alice/.ssh
```

Protect `authorized_keys`:

```bash
sudo chown alice:alice /home/alice/.ssh/authorized_keys
sudo chmod 600 /home/alice/.ssh/authorized_keys
```

Create the project directory:

```bash
sudo mkdir /home/dev-project
```

Assign group ownership:

```bash
sudo chown root:developers /home/dev-project
```

Set permissions:

```bash
sudo chmod 770 /home/dev-project
```

Inspect ownership and permissions:

```bash
ls -ld /home/dev-project
```

---

# What I Should Know

After completing this guide, I should be able to:

- Explain why Linux internally identifies users with UIDs and groups with GIDs.
- Explain why the `root` user is special and why UID `0` matters.
- Use `whoami` and `id` to inspect the current identity.
- Explain the difference between a primary group and supplementary groups.
- Explain the purpose and basic structure of `/etc/passwd`.
- Explain why `/etc/passwd` is readable by normal users.
- Explain the purpose of `/etc/shadow`.
- Explain why Linux stores password hashes instead of plaintext passwords.
- Explain why a password hash cannot simply be decoded back into the original password.
- Explain what service users are and why services should not normally run as `root`.
- Explain why service accounts commonly use `/usr/sbin/nologin`.
- Create a normal Linux user with `useradd`.
- Create a home directory for a new user.
- Change a user's login shell with `usermod`.
- Create a group with `groupadd`.
- Add a user to a supplementary group using `usermod -aG`.
- Explain why omitting `-a` from `usermod -G` can be dangerous.
- Use `getent passwd` and `getent group` to inspect account information.
- Explain the difference between searching `/etc/passwd` directly and querying system identity databases with `getent`.
- Set or reset a user's password with `passwd`.
- Explain when a Linux account needs a password and when SSH key-only authentication may be enough.
- Explain the difference between authentication and authorization.
- Explain why having a password does not automatically give a user `sudo` privileges.
- Generate a new Ed25519 SSH key pair for a specific user.
- Explain the difference between an SSH private key and public key.
- Configure `/home/<user>/.ssh/authorized_keys`.
- Apply secure ownership and permissions to `.ssh` and `authorized_keys`.
- Log in to the server using a user's own SSH private key.
- Explain file and directory ownership using owner and group.
- Explain the owner, group, and others permission classes.
- Explain what `r`, `w`, and `x` mean.
- Explain the special meaning of `r`, `w`, and `x` on directories.
- Convert basic numeric permissions such as `770` into `rwx` notation.
- Change ownership with `chown`.
- Change permissions with `chmod`.
- Create a directory owned by a specific group.
- Give members of that group access to the directory.
- Explain why group ownership alone is not enough without matching permission bits.
- Diagnose a basic `Permission denied` problem by checking identity, group membership, ownership, and permissions.