# Sentinel — Security Design and Residual Risk Notes

## Core traffic-policy intent

**CLIENTS-IN** permits client-to-gateway ICMP, DNS to AD-DNS-SRV, HTTP to WEB-SRV, and Client1-only FTP control/data to FTP-LAB-SRV. Other internal destinations are denied; permitted external traffic is forwarded toward ASA subject to perimeter handling.

**FTP-LAB-IN / FTP-LAB-OUT** permit narrowly identified FTP responses and infrastructure management/monitoring flows, with default-deny restrictions for the FTP segment. The FTP-LAB switch uplink is access VLAN 110; Fa0/2 is the SPAN destination and must not be repurposed.

**ASA perimeter** uses inside VLAN 250 at security level 100, outside VLAN 2 at security level 0, dynamic PAT for internal clients, and explicit routes to CORE-SW and ISP-RTR. Security levels alone are not a substitute for verifying the full effective ASA access policy.

## Security controls and caveats

| Control | Implementation | Caveat |
|---|---|---|
| SSH administration | SSH v2, local authentication, source restriction | Packet Tracer simulated feature set |
| Switch port security | Sticky MAC, max 1 on tested client port | Rogue MAC violation test passed; not an enterprise NAC replacement |
| STP protection | CORE root priority, PortFast and BPDU Guard | Edge ports only; do not apply PortFast indiscriminately to uplinks |
| DHCP snooping | VLAN 20 enabled, 10 pps rate limit on client ports | Static-addressed lab; no DHCP bindings to support DAI |
| Dynamic ARP Inspection | Not enabled | ARP ACL syntax unsupported in simulator; residual ARP spoofing risk |
| Native VLAN | 998 on designated trunks | FTP-LAB uplink intentionally access VLAN 110 |
| Unused ports | Disabled, parked in VLAN 999 | FTP SPAN monitor Fa0/2 is in use, not unused |
| FTP service | Client1 allow; Client2 deny | Wide TCP data-port range required by simulated FTP; plaintext FTP is not confidential |
| Logging | Syslog to SIEM host | Log receipt verified; not proof of full SIEM correlation |
| Network monitoring | SPAN and designated PCAP/IDS servers | Topology does not itself prove real IDS/PCAP analytical capability |

## Simulator anomalies and limits

- **DAI:** ARP ACL configuration unsupported on the simulated switch. DAI deliberately left off rather than risk disrupting static hosts.
- **HTTP service inspection:** Several IOS commands for checking or disabling the HTTP server were unsupported. Runtime management exposure remains unverified.
- **ASA route output:** An unexpected `10.0.0.0/30` connected route appeared in addition to the correct `10.10.250.0/30`. The cause was not established; no corrective changes were made solely on this basis.
- **ASA inspection/policy commands:** Some expected global policy commands were unsupported by the Packet Tracer ASA model.
- **FTP:** Plaintext FTP and broad passive/data-port allowance are lab choices, not recommended production defaults.

## Suggested future extension (not part of completed scope)

A production-equivalent implementation could use a supported IOS/ASA platform or virtual network environment, enforce DAI with appropriate trust/bindings, narrow passive FTP ports or migrate to SFTP, add authenticated central log collection, validate IDS signatures and PCAP retrieval, and run a documented adversarial test plan. These are **future enhancements**, not unfinished tasks in the Packet Tracer project.
