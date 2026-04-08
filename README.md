# 🔧 linux-troubleshooting

> A practical Linux sysadmin study lab focused on real-world troubleshooting, diagnostics, log analysis, and technical documentation.

![Linux](https://img.shields.io/badge/Linux-Ubuntu_Server-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-Scripting-4EAA25?style=flat-square&logo=gnu-bash&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)
![Focus](https://img.shields.io/badge/Focus-Sysadmin%20%7C%20Troubleshooting-blue?style=flat-square)
![Environment](https://img.shields.io/badge/Environment-Isolated%20VM-lightgrey?style=flat-square)

---

## 📋 About

This repository documents my hands-on Linux sysadmin practice in an isolated Ubuntu Server VM. It is not a finished product — it is a structured learning environment where I break things intentionally, diagnose what went wrong, fix it, and document the process.

Every entry here represents a real problem encountered, investigated, and resolved. The goal is not just to fix issues, but to understand *why* they happen.

---

## 🎯 Objectives

Skills being actively developed in this lab:

- **Troubleshooting** — systematic diagnosis of service failures, boot issues, and misconfigurations
- **Log analysis** — reading and interpreting `journalctl`, `syslog`, `auth.log`, and service-specific logs
- **Networking** — diagnosing connectivity, routing, DNS resolution, and firewall rules
- **systemd** — service lifecycle management, unit file analysis, dependency resolution
- **SSH hardening** — key-based auth, config hardening, connection debugging
- **Bash scripting** — automation of diagnostic and maintenance routines
- **File system** — permissions, ownership, mount points, disk usage analysis

---

## 🖥️ Study Environment

| Component | Details |
|---|---|
| **OS** | Ubuntu Server 24.04 LTS |
| **Platform** | VirtualBox 7.2 on Fedora Linux (host) |
| **Network** | Isolated — not part of the main infrastructure lab |
| **Access** | SSH from host terminal |
| **Purpose** | Dedicated troubleshooting sandbox |

> This VM is intentionally kept separate from the [main lab](https://github.com/reisops/projeto-linux) to avoid interference with production-like services.

---

## 📁 Repository Structure

```
linux-troubleshooting/
│
├── cases/                        # Documented troubleshooting cases
│   ├── ssh/                      # SSH connection and config issues
│   ├── networking/               # Routing, DNS, interface problems
│   ├── systemd/                  # Service failures and unit debugging
│   ├── filesystem/               # Permissions, mounts, disk issues
│   └── logs/                     # Log analysis walkthroughs
│
├── scripts/                      # Diagnostic and utility bash scripts
│   ├── check-services.sh         # Quickly verify service status
│   ├── net-diag.sh               # Network diagnostics snapshot
│   └── log-parser.sh             # Filter and summarize common log patterns
│
├── notes/                        # Reference notes and command cheatsheets
│   ├── systemd-cheatsheet.md
│   ├── network-commands.md
│   └── log-locations.md
│
└── README.md
```

Each entry inside `cases/` follows the same structure:

```
cases/ssh/001-connection-refused.md
cases/ssh/002-publickey-denied.md
cases/networking/001-default-route-missing.md
...
```

---

## 🧠 Learning Methodology

Every case in this repo follows the same four-step process:

```
OBSERVE → DIAGNOSE → FIX → DOCUMENT
```

| Step | Description |
|---|---|
| **Observe** | Reproduce or identify the symptom — error message, behavior, log output |
| **Diagnose** | Use system tools to isolate the root cause |
| **Fix** | Apply the correct solution — not just a workaround |
| **Document** | Write it down: what happened, why, how it was fixed, what was learned |

The principle here is simple: **a problem you can explain is a problem you actually understand.**

---

## 🔧 Troubleshooting Example

### Case: SSH — `Connection refused` on port 2201

**Symptom**
```bash
$ ssh user@127.0.0.1 -p 2201
ssh: connect to host 127.0.0.1 port 2201: Connection refused
```

**Diagnosis**
```bash
# Check if sshd is running
sudo systemctl status ssh

# Output:
# Unit ssh.service could not be found
```

Root cause: OpenSSH server was not installed on the VM. The NAT port forward rule existed on the host, but there was no service listening on the guest.

**Fix**
```bash
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

**Verification**
```bash
$ ssh user@127.0.0.1 -p 2201
# Connected successfully
```

**What I learned**

A `Connection refused` at the TCP level means the port is reachable but nothing is listening — which is different from a timeout (host unreachable) or a firewall drop. Distinguishing these three failure modes is essential for efficient troubleshooting.

---

## 🚀 How to Use

This is a **study repository**, not a deployable application.

```bash
git clone https://github.com/reisops/linux-troubleshooting.git
cd linux-troubleshooting
```

Browse `cases/` to read documented issues and solutions.  
Run scripts from `scripts/` on your own Ubuntu environment to replicate diagnostics.

> Scripts are written for Ubuntu Server 24.04. Running them on other distros may require adaptation.

---

## 🧰 Skills Developed

| Area | Tools & Concepts |
|---|---|
| Service management | `systemctl`, `journalctl`, unit files, targets |
| Network diagnostics | `ip`, `ss`, `ping`, `traceroute`, `dig`, `nslookup` |
| Log analysis | `journalctl`, `tail -f`, `grep`, `awk`, `/var/log/*` |
| SSH | `sshd_config`, key-based auth, `ssh -v`, port forwarding |
| File system | `chmod`, `chown`, `df`, `du`, `mount`, `lsblk` |
| Bash scripting | Functions, conditionals, loops, exit codes, `trap` |
| Firewall | `iptables`, chain policies, rule inspection |

---

## 🔗 Related Projects

| Repository | Description |
|---|---|
| [projeto-linux](https://github.com/reisops/projeto-linux) | Full infrastructure lab: DNS, DHCP, Samba, Apache, Squid, iptables across 3 Ubuntu Server VMs |

This repository complements `projeto-linux` by focusing on the diagnostic layer — what to do when services in that environment (or any Linux environment) stop working as expected.

---

## 👤 Author

**Luis Reis**  
[GitHub](https://github.com/reisops) · [LinkedIn](https://linkedin.com/in/luis-reis-ops)

---

## ⚠️ Disclaimer

All activity in this repository is performed in an isolated virtual machine with no connection to production systems or external networks. Scripts are provided as-is for educational purposes.
