# Enterprise Multi-Protocol Routing Challenge

**Design, Implementation, and Failover Engineering in Mixed OSPF/RIPv2 Environments**

[![Protocol](https://img.shields.io/badge/OSPF-Multi--Area-blue)]()
[![Protocol](https://img.shields.io/badge/RIPv2-Classless-orange)]()
[![Platform](https://img.shields.io/badge/Cisco%20IOS-7200%20Series-lightgrey)]()
[![Status](https://img.shields.io/badge/Status-Verified%20%26%20Operational-brightgreen)]()

> Internship Project — Network Administration Track
> **ITSimplera Institute** | Submitted by **Ghulam Abbas** (Reg. No. NETB01-8085)
> Supervisor: Sir Jawad Qayum | Week 3 Deliverable | July 17, 2026

---

## 📖 Overview

This project simulates a real-world **corporate merger scenario**: a Corporate Headquarters running a modern **OSPF** link-state core is integrated with a newly acquired **RIPv2** branch office running legacy distance-vector routing.

The goal is complete, seamless, **bidirectional IP reachability** across both routing domains, achieved through:

- **Multi-Area OSPF** (Area 0 backbone + Area 1 transit) to control LSA flooding and SPF overhead
- **Mutual route redistribution** at the ASBR to translate between OSPF cost and RIPv2 hop-count metrics
- A **floating static route** (AD 50) over a secondary WAN link for deterministic, automatic failover

---

## 🗺️ Network Topology

```
   Corporate_HQ                                              Acquired_Branch
   (10.1.10.0/24)                                             (192.168.44.0/24)
        |                                                            |
     CoreR1 ─────Gi1/0──── OSPF Area 0 ────Gi1/0───── Area1R2
        |  (10.1.1.0/24 backbone)                        |
        |                                             Fa0/0 (OSPF Area 1)
   Fa0/0 (Backup)                                          |
        |                                              BoundaryR3 (ASBR)
        |                                                  |
        |                                         Gi1/0 (RIPv2, 192.168.34.0/24)
        |                                                  |
        └──────────── Floating Static (192.168.14.0/24) ──BranchR4
                          Backup Link — idle unless primary fails
```

**Three logically isolated routing domains:**

| Domain | Routers | Protocol | Purpose |
|---|---|---|---|
| OSPF Area 0 (Backbone) | CoreR1 ↔ Area1R2 | OSPFv2 | High-speed core transit |
| OSPF Area 1 (Transit) | Area1R2 ↔ BoundaryR3 | OSPFv2 | Isolates local LSA flooding |
| RIPv2 Domain | BoundaryR3 ↔ BranchR4 | RIPv2 | Legacy acquired branch |

---

## 🧩 Router Roles

| Router | Role | Function |
|---|---|---|
| **CoreR1** | Core Router | Anchors OSPF Area 0; hosts HQ LAN gateway and backup-link interface |
| **Area1R2** | ABR (Area Border Router) | Splits link-state boundary between Area 0 and Area 1 |
| **BoundaryR3** | ASBR | Performs mutual redistribution between OSPF and RIPv2 |
| **BranchR4** | Branch Edge Router | Runs RIPv2; gateway for acquired branch LAN and backup link termination |

---

## 🌐 IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Segment / Purpose |
|---|---|---|---|---|
| CoreR1 | Gi1/0 | 10.1.1.1 | /24 | OSPF Area 0 Backbone |
| CoreR1 | Gi2/0 | 10.1.10.1 | /24 | Corporate HQ LAN Gateway |
| CoreR1 | Fa0/0 | 192.168.14.1 | /24 | Resilient Backup Path |
| Area1R2 | Gi1/0 | 10.1.1.2 | /24 | Area 0 / Area 1 Transition |
| Area1R2 | Fa0/0 | 10.1.2.1 | /24 | OSPF Area 1 Transit |
| BoundaryR3 | Fa0/0 | 10.1.2.3 | /24 | OSPF Area 1 Boundary |
| BoundaryR3 | Gi1/0 | 192.168.34.3 | /24 | RIPv2 Redistribution Boundary |
| BranchR4 | Gi1/0 | 192.168.34.4 | /24 | RIPv2 Segment Link |
| BranchR4 | Gi2/0 | 192.168.44.1 | /24 | Acquired Branch LAN Gateway |
| BranchR4 | Fa0/0 | 192.168.14.4 | /24 | Backup Link Termination |

### Subnet Zones

- **Core Enterprise (10.1.0.0/16):** `10.1.1.0/24` OSPF Area 0 · `10.1.10.0/24` HQ LAN · `10.1.2.0/24` OSPF Area 1 transit
- **Acquired Branch (192.168.0.0/16):** `192.168.34.0/24` translation link · `192.168.44.0/24` branch LAN · `192.168.14.0/24` backup link (idle under normal conditions)

---

## ⚙️ Configuration Summary

### CoreR1 — OSPF Area 0 Core
```
interface GigabitEthernet1/0
 ip address 10.1.1.1 255.255.255.0
 no shutdown
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
router ospf 1
 router-id 1.1.1.1
 network 10.1.1.0 0.0.0.255 area 0
 network 1.1.1.1 0.0.0.0 area 0
```

### Area1R2 — ABR (Area 0 ↔ Area 1)
```
interface GigabitEthernet1/0
 ip address 10.1.1.2 255.255.255.0
interface FastEthernet0/0
 ip address 10.1.2.1 255.255.255.0
router ospf 1
 router-id 2.2.2.2
 network 10.1.1.0 0.0.0.255 area 0
 network 10.1.2.0 0.0.0.255 area 1
```

### BoundaryR3 — ASBR (Mutual Redistribution)
```
interface GigabitEthernet1/0
 ip address 10.1.2.3 255.255.255.0
interface FastEthernet0/0
 ip address 192.168.34.3 255.255.255.0
router ospf 1
 router-id 192.168.34.3
 network 10.1.2.0 0.0.0.255 area 1
 redistribute rip subnets
router rip
 version 2
 network 192.168.34.0
 redistribute ospf 1 metric 5
 no auto-summary
```

### BranchR4 — RIPv2 Branch Edge
```
interface GigabitEthernet1/0
 ip address 192.168.34.4 255.255.255.0
interface GigabitEthernet2/0
 ip address 192.168.44.1 255.255.255.0
router rip
 version 2
 network 192.168.34.0
 network 192.168.44.0
 no auto-summary
```

### Floating Static Route (Failover Backup)
```
ip route 10.1.10.0 255.255.255.0 FastEthernet0/0 50
```
Configured with **AD 50** — higher than OSPF (110 for the primary learned path is not applicable here since this is a static override on the backup segment) but still lower than default static, ensuring the route stays dormant until the primary path is unavailable.

---

## 📊 Administrative Distance Reference

| Source | Administrative Distance |
|---|---|
| Directly Connected | 0 |
| Static Route | 1 |
| OSPF | 110 |
| RIPv2 | 120 |
| Floating Static (this project) | 50 |

---

## ✅ Verification & Testing

Validation performed using standard Cisco IOS `show` and reachability commands:

- `show ip ospf neighbor` — confirms FULL/BDR adjacency between CoreR1 ↔ Area1R2 and Area1R2 ↔ BoundaryR3
- `show ip protocols` — confirms Area1R2 as ABR and BoundaryR3 as ASBR with active redistribution
- `show ip route` — confirms OSPF-learned routes (`O`, `O IA`), RIP-learned routes (`R`), and redistributed external routes (`O E2`) across all four routers
- `ping 10.1.10.1` from BranchR4 — **100% success**, RTT ~136–152 ms
- `traceroute 10.1.10.1` from BranchR4 — confirms correct 3-hop path: BoundaryR3 → Area1R2 → CoreR1

**Result:** Full end-to-end reachability between Corporate HQ and the Acquired Branch across both routing domains, with the backup path verified idle under normal operating conditions.

---

## 🏁 Conclusion

The deployment demonstrates a fully converged, resilient multi-protocol architecture:

- OSPF Area 0/Area 1 segmentation minimizes control-plane overhead and SPF recalculation scope
- Mutual OSPF ↔ RIPv2 redistribution at the ASBR (BoundaryR3) achieves loop-free, controlled route leakage
- A floating static route provides deterministic failover without impacting normal traffic flow

---
## 📁 Project Directory Structure

```text
├── Config/                  # Cisco device startup configuration files (.cfg)
├── images/                  # Network topology screenshots and diagrams
├── Enterprise Multi-Protocol Challenge.pdf  # Detailed project documentation
├── Enterprise Multi-Protocol Routing.gns3   # Main GNS3 environment file
└── README.md                # Project overview and technical protocols list

---


## 👤 Author

**Ghulam Abbas**
Registration No: NETB01-8085
Network Administration — ITSimplera Institute
