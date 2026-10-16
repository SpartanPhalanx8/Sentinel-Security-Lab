# Sentinel — Verification Record

**Final stage:** 7.20.12  
**Project:** Complete; saved `.pkt` confirmed  
**Evidence source:** Interactive Packet Tracer CLI output and user-confirmed endpoint tests during the project.

## Final regression sequence

| # | Test | Expected | Observed | Result |
|---|---|---|---|---|
| 1 | Client1 `ping 10.10.20.1` | Gateway reachable | 4/4 replies | PASS |
| 2 | Client1 `ping 10.10.30.10` | Blocked | Destination host unreachable from gateway | PASS |
| 3 | Client1 `ping 10.10.40.10` | Blocked | Destination host unreachable from gateway | PASS |
| 4 | Client1 `nslookup www.sentinel.local` | Resolve WEB-SRV | Successful | PASS |
| 5 | Client1 browser `http://10.10.40.10` | HTTP allowed | Page loaded | PASS |
| 6 | Client1 `ftp 10.10.110.10`, login, `dir` | Allowed | Login and directory listing successful | PASS |
| 7 | Client2 `ftp 10.10.110.10` | Denied | Connection failed | PASS |
| 8 | Client1 `tracert 203.0.113.1` | CORE → ASA → ISP | Route confirmed | PASS |

## Final CORE-SW CLI checks

`show ip interface brief`:
- Active VLAN SVIs 10–90, 110, 250: up/up.
- VLAN 1 and VLAN 100: administratively down.
- Physical uplinks Fa0/2–Fa0/5 and Gi0/1–Gi0/2: up/up.
- Unused ports Fa0/1 and Fa0/6–Fa0/24: administratively down.

`show ip route`:
```text
C 10.10.10.0/24 through 10.10.90.0/24 (nine operational VLANs)
C 10.10.110.0/24 directly connected, Vlan110
C 10.10.250.0/30 directly connected, Vlan250
S* 0.0.0.0/0 [1/0] via 10.10.250.1
```

## Final ASA CLI checks

`show interface ip brief`:
```text
Vlan2    203.0.113.2    up up
Vlan250  10.10.250.1   up up
Vlan1    unassigned    administratively down down
```

`show nat`:
```text
Auto NAT Policies (Section 2)
1 (inside) to (outside) source dynamic SENTINEL-INTERNAL interface
    translate_hits = 3, untranslate_hits = 2
```

`show route` key lines:
```text
S  10.10.0.0 255.255.0.0 [1/0] via 10.10.250.2, inside
C  10.10.250.0 255.255.255.252 is directly connected, inside, Vlan250
C  203.0.113.0 255.255.255.252 is directly connected, outside, Vlan2
S* 0.0.0.0/0 [1/0] via 203.0.113.1
```

Additional anomalous route also displayed:
```text
C 10.0.0.0 255.255.255.252 is directly connected, inside, Vlan250
```
See `SECURITY-NOTES.md` for treatment.

`show run interface vlan 250`:
```text
interface Vlan250
 nameif inside
 security-level 100
 ip address 10.10.250.1 255.255.255.252
```

`show run interface vlan 2`:
```text
interface Vlan2
 nameif outside
 security-level 0
 ip address 203.0.113.2 255.255.255.252
```

ASA `write memory` and final Packet Tracer File → Save: confirmed successful.

## Prior-stage tests

- Authorised SSH from Analyst-PC1 succeeded; unauthorised SSH from Analyst-PC2 denied.
- Rogue MAC caused eight port-security violations on SW-CLIENT Fa0/1; legitimate client connectivity restored.
- Client1 TCP/23 to AD-DNS and TCP/445 to FILE-SRV blocked; ACL counters increased.
- FTP server unable to initiate unauthorised connection to AD-DNS.
- CORE-SW root priority configured to 4096 for VLANs 10–90; root verified.
- CDP disabled on all six switches.
- PortFast/BPDU Guard enabled on designated endpoint ports across access switches; FTP SPAN preserved.
- Syslog received at SIEM endpoint; NTP synchronisation tested on six switches.

**Evidence note:** This document reports outcomes observed or confirmed during interactive testing. It does not claim that packet captures, screenshots or full configuration exports are included in this repository.
