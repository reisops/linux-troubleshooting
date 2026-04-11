Case: SSH alias not resolving — could not connect to VM from host

## Symptom

```bash
$ ssh almalinux-lab
ssh: Could not resolve hostname almalinux-lab: Name or service not known
```

## Diagnosis

```bash
# Check connectivity
ping 192.168.56.X
# OK — packets reaching the VM

# Attempted direct connection by IP to isolate the issue
ssh luis@192.168.56.X
# Connected successfully — network was not the problem

# Tested with verbose output to inspect the handshake
ssh -v luis@192.168.56.X
# Connected — confirmed issue was in the alias config, not SSH itself

# Inspected the host SSH config
cat ~/.ssh/config
# alias for almalinux-lab was missing
```

## Root Cause

Two separate issues in two different contexts:

1. On the VM — `~/.ssh/` directory didn't exist; Vim threw `E212: Can't open file for writing` when attempting to create `~/.ssh/config`
2. On the host — `~/.ssh/config` existed (with other VM aliases) but the `almalinux-lab` entry was never added

## Fix

On the VM — created the directory with correct permissions:

```bash
mkdir ~/.ssh
chmod 700 ~/.ssh
```

On the host — added the missing alias to `~/.ssh/config` using nano:

```
Host almalinux-lab
    HostName 192.168.56.X
    User luis
    Port 22
```

## Validation

```bash
ssh almalinux-lab
# Connected successfully using alias
```

## Lesson

When an SSH alias fails to resolve, the issue is always on the client side. Always verify:

- `~/.ssh/config` on the host — the alias must be present and correctly formatted
- DNS or `/etc/hosts` — if hostname resolution is involved

Guest-side issues may block configuration attempts, but they do not affect alias resolution itself. SSH alias resolution does not involve DNS unless explicitly configured — it is a local mapping defined in `~/.ssh/config`.

`~/.ssh/` must exist with `700` permissions before any SSH config can be written. SSH enforces strict permission rules and will refuse to operate if they're wrong. Use `ssh -v` to confirm whether the issue is network, config, or authentication.
