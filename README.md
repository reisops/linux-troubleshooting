# 🔧 linux-troubleshooting

Hands-on Linux troubleshooting lab. Real failures, real diagnosis, real fixes — documented.

![Linux](https://img.shields.io/badge/Linux-AlmaLinux_9-0068B5?style=flat-square&logo=almalinux&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-Scripting-4EAA25?style=flat-square&logo=gnu-bash&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)
![Focus](https://img.shields.io/badge/Focus-Sysadmin%20%7C%20Troubleshooting-blue?style=flat-square)
![Goal](https://img.shields.io/badge/Goal-RHCSA-EE0000?style=flat-square&logo=redhat&logoColor=white)
![Environment](https://img.shields.io/badge/Environment-Isolated%20VM-lightgrey?style=flat-square)

---

## 🖥️ Environment

| Component | Details |
|-----------|---------|
| OS | AlmaLinux 9 (RHEL-compatible) |
| Platform | VirtualBox on Fedora Linux (host) |
| Access | SSH from host terminal |
| Goal | RHCSA preparation + practical sysadmin skills |

---

## 📁 Structure

```
cases/
  ssh/
  networking/
  systemd/
  filesystem/
  logs/
scripts/
notes/
```

Each case follows the same format: **symptom → root cause → fix → lesson**.

---

## 🧠 Methodology

```
OBSERVE → DIAGNOSE → FIX → DOCUMENT
```

A problem you can explain is a problem you actually understand.

---

## 🧰 Skills in Practice

`systemctl` `journalctl` `firewalld` `SELinux` `ss` `ip` `dig` `sshd` `chmod` `awk` `bash`

---

## 🔗 Related

[`projeto-linux`](https://github.com/reisops/projeto-linux) — multi-VM infrastructure lab (DNS, DHCP, Samba, Apache, Squid)

---

*Luis Reis — [LinkedIn](https://www.linkedin.com/in/luis-reis-ops/)*

