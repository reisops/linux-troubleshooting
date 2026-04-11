# Case: SSH alias not resolving — could not connect to VM from host

## Symptom

```bash
$ ssh almalinux-lab
ssh: Could not resolve hostname almalinux-lab: Name or service not known
```

## Diagnosis

```bash
# Checked if ~/.ssh/ existed on the guest
ls -la ~/.ssh/
# No such file or directory — directory didn't exist

# Checked network reachability from host
ping 192.168.56.X
# OK — packets reaching the VM

# Tested SSH connection with verbose output
ssh -v luis@192.168.56.X
# Connected successfully — issue was in the config, not the network
```

## Root Cause

Two separate issues:

1. `~/.ssh/` directory didn't exist on the guest — Vim couldn't write `~/.ssh/config`, throwing `E212: Can't open file for writing`
2. The SSH alias was never added to `~/.ssh/config` on the **host** — only the guest config was edited

## Fix

On the guest — created the directory with correct permissions:

```bash
mkdir ~/.ssh
chmod 700 ~/.ssh
```

On the host — added the alias to `~/.ssh/config`:

```
Host almalinux-lab
    HostName 192.168.56.X
    User luis
    Port 22
```

## Lesson

`~/.ssh/` must exist with `700` permissions before any SSH config can be written — SSH enforces strict permission rules and will refuse to operate if they're wrong. When an alias fails to resolve, always check both ends: the config on the host **and** whether the directory structure exists on the guest.
