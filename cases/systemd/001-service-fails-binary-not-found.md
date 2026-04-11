# Case: Service fails to start — binary not found (status=203/EXEC)

## Symptom

```
$ sudo systemctl start myapp
$ systemctl status myapp
× myapp.service - My Application
     Loaded: loaded (/etc/systemd/system/myapp.service; enabled; preset: disabled)
     Active: failed (Result: exit-code) since Sat 2026-04-11 18:42:42 -03; 26s ago
   Main PID: 4641 (code=exited, status=203/EXEC)
```

## Diagnosis

```
# Inspected the unit file to confirm the ExecStart path
cat /etc/systemd/system/myapp.service
# ExecStart=/usr/bin/myapp

# Checked the journal for the exact error
journalctl -u myapp --no-pager
# systemd: Failed to locate executable /usr/bin/myapp: No such file or directory
# systemd: Failed at step EXEC spawning /usr/bin/myapp: No such file or directory

# Confirmed the binary didn't exist
ls /usr/bin/myapp
# No such file or directory

# Checked if the service was masked
systemctl status myapp
# Loaded: loaded (/etc/systemd/system/myapp.service; enabled; preset: disabled)
# Active: failed — service was also masked, blocking enable attempts
```

## Root Cause

Two separate issues:

1. The unit file pointed to `/usr/bin/myapp`, but the binary was never created — systemd had valid instructions but nothing to execute
2. The service was masked (`systemctl mask`), which blocked any attempt to enable or restart it until explicitly unmasked

`status=203/EXEC` always means the same thing: systemd found the unit file, attempted to spawn the process, but the executable did not exist or was not accessible.

## Fix

Unmasked the service first:

```
sudo systemctl unmask myapp
```

Created the missing binary:

```
sudo bash -c 'echo "#!/bin/bash
while true; do sleep 60; done" > /usr/bin/myapp'
```

Made it executable:

```
sudo chmod +x /usr/bin/myapp
```

Started and enabled the service:

```
sudo systemctl start myapp
sudo systemctl enable myapp
```

## Validation

```
$ systemctl status myapp
● myapp.service - My Application
     Loaded: loaded (/etc/systemd/system/myapp.service; enabled; preset: disabled)
     Active: active (running) since Sat 2026-04-11 18:57:59 -03
   Main PID: 4765 (myapp)
     CGroup: /system.slice/myapp.service
             ├─4765 /bin/bash /usr/bin/myapp
             └─4766 sleep 60
```

## Lesson

`status=203/EXEC` means systemd reached the execution step but could not spawn the process — the binary was missing, not executable, or the path was wrong. Always verify the `ExecStart` path exists and has execute permissions before investigating anything else.

A masked service is completely blocked by systemd — `enable`, `start`, and `restart` will all fail silently or with access denied. Always check for masking when a service refuses to respond to normal commands: `systemctl status` will show `Loaded: masked` if that's the case.

In production, this error typically appears after a failed deployment, a package removal that left the unit file behind, or a binary path change without updating the unit file.
