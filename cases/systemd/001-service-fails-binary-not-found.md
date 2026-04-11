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
test -f /usr/bin/myapp || echo "binary not found"
# binary not found
```

## Root Cause

The unit file pointed to `/usr/bin/myapp`, but the binary was never created — systemd had valid instructions but nothing to execute.

`status=203/EXEC` indicates systemd reached the execution step but could not spawn the process. This can happen when the binary does not exist, when it exists but lacks execute permissions, or when the shebang points to a non-existent interpreter.

In this case, the journal confirmed the specific cause: `No such file or directory`.

## Fix

Created the missing binary:

```
sudo bash -c 'printf "#!/bin/bash\nwhile true; do sleep 60; done\n" > /usr/bin/myapp'
```

Verified the file was created correctly:

```
cat /usr/bin/myapp
# #!/bin/bash
# while true; do sleep 60; done
```

Made it executable:

```
sudo chmod +x /usr/bin/myapp
```

Started the service:

```
sudo systemctl start myapp
```

Enabled it to start on boot:

```
sudo systemctl enable myapp
```

Verified the enable took effect:

```
systemctl is-enabled myapp
# enabled
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

`status=203/EXEC` means systemd reached the execution step but could not spawn the process. The cause can be a missing binary, missing execute permission, or an invalid interpreter in the shebang. The journal always contains the specific reason — check it before assuming the cause.

Always verify the `ExecStart` path exists, is a regular file, and has execute permissions:

```
test -f /path/to/binary && test -x /path/to/binary && echo "ok" || echo "problem"
```

In production, this error typically appears after a failed deployment, a package removal that left the unit file behind, or a binary path change without updating the unit file.
