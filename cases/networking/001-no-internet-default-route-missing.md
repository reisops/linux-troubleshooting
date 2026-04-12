# Case: No internet — interface is up but default route is missing

## Symptom

```
$ ping 8.8.8.8
connect: Network is unreachable

$ ip route
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.XX metric 100
192.168.56.0/24 dev enp0s8 proto kernel scope link src 192.168.56.XX metric 101
# no default route — confirmed at this point
```

## Diagnosis

```
# State 1 — immediately after the failure was observed

# Confirmed default route was missing
ip route
# 10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.XX metric 100
# 192.168.56.0/24 dev enp0s8 proto kernel scope link src 192.168.56.XX metric 101
# no default line present

# Checked journal for recent events
journalctl -e
# sudo[1333]: COMMAND=/sbin/ip route del default
# the command path /sbin/ip confirms this was a system-level routing operation,
# not an interface or service failure — narrowed the issue to routing layer
# note: this is evidence of the action, not direct proof of causality

# Checked interface status and IP assignment
ifconfig
# enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> inet 10.0.2.XX
# enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> inet 192.168.56.XX
# both interfaces up and with IPs assigned — issue likely isolated to routing

# ---
# State 2 — after some time, during continued investigation

# Checked route resolution for the target host
ip route get 8.8.8.8
# 8.8.8.8 via 10.0.2.XX dev enp0s3 src 10.0.2.XX
# a route was now present — observed behavior suggests DHCP had restored it
# note: a valid route does not guarantee full connectivity — firewall rules or ARP
# failures can still block traffic after routing is confirmed

# Confirmed with ip route
ip route
# default via 10.0.2.XX dev enp0s3 proto dhcp src 10.0.2.XX metric 100
# default route now present
```

## Root Cause

The journal showed that `ip route del default` was executed shortly before the failure was observed. The `/sbin/ip` path in the log confirmed this was a direct routing operation — not an interface failure or service crash — which helped narrow the investigation quickly.

The absence of a default route left the kernel with no path to forward packets to external hosts, producing `Network is unreachable`.

The default route was restored before any manual intervention, likely by the DHCP client on `enp0s3` during its renewal cycle.

Note: the journal entry confirms the deletion occurred, but does not rule out other network events between the deletion and the observed failure.

## Fix

No manual fix was required — the default route was restored automatically, likely by the DHCP client:

```
$ ip route
default via 10.0.2.XX dev enp0s3 proto dhcp src 10.0.2.XX metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.XX metric 100
192.168.56.0/24 dev enp0s8 proto kernel scope link src 192.168.56.XX metric 101
```

If the route had not been restored automatically, the manual fix would be:

```
sudo ip route add default via 10.0.2.XX dev enp0s3
```

## Validation

```
$ ping 8.8.8.8
64 bytes from 8.8.8.8: icmp_seq=1 ttl=255 time=16.7 ms
# external IP reachable
# note: TTL 255 is expected in this VirtualBox NAT environment

$ ping google.com
64 bytes from rio07s01-in-f14.1e100.net: icmp_seq=1 ttl=255 time=14.7 ms
# hostname resolution working, external host reachable

$ nslookup google.com
Server:		10.0.2.XX
Address:	10.0.2.XX#53
Name:	google.com
Address: 142.250.78.XXX
# DNS resolving correctly via DHCP-assigned nameserver
```

## Lesson

In this case, `Network is unreachable` at the IP level indicated a missing default route. This error can also appear in other scenarios — interface down, policy routing misconfiguration, or network namespace issues — so routing should be verified first, but not assumed to be the only possible cause.

The journal entry showing `/sbin/ip route del default` was a useful early signal: seeing a system-level routing command in the log immediately pointed to routing as the likely cause, rather than an interface or DNS problem.

`ip route get <destination>` shows which route the kernel would use for a specific host. A successful result confirms a route exists at the routing layer, but does not guarantee end-to-end connectivity — firewall rules or ARP failures can still block traffic after routing is confirmed.

In DHCP environments, deleted routes may be restored automatically on the next renewal cycle. If connectivity returns without intervention, check the journal for DHCP events to understand what likely happened.
