# Enterprise-Multi-Protocol-Routing-Core-Infrastructure-Challenge
A GNS3 enterprise network simulation featuring multi-area OSPF, RIPv2 route redistribution, and core path optimization.

# Enterprise Multi-Protocol Routing Challenge

An enterprise-grade network architecture designed, deployed, and validated within GNS3 using Cisco IOS (v15.x). This project demonstrates multi-area OSPFv2 implementation, RIPv2 branch integration, and mutual route redistribution with precise metric translations on an Autonomous System Boundary Router (ASBR) to maintain loop-free dynamic routing.

---

## 🗺️ Topology Architecture

The network infrastructure is partitioned into distinct routing domains to balance scalability, administrative control, and isolation:

1. **Core Backbone (OSPF Area 0):** The central transit area connecting core routing resources.
2. **Internal Transit (OSPF Area 1):** Configured with the `10.1.2.0/24` subnet. 
   * **Area1R2** acts as the internal OSPF transit router (`FastEthernet0/0` - `10.1.2.0/24`).
   * **BoundaryR3 (ASBR)** acts as the OSPF boundary interface (`GigabitEthernet1/0` - `10.1.2.3/24`).
3. **Branch Network (RIPv2):** Configured on remote segments (e.g., `192.168.34.0/24`) to handle branch operations via distance-vector routing.

---

## ⚙️ Key Technical Implementations

### 1. Multi-Area OSPF Configuration
* Dynamic neighbor adjacencies established and verified across Area 0 and Area 1.
* Interface network types, hello/dead timers, and area boundaries tuned for fast convergence.

### 2. Mutual Route Redistribution
To bridge the OSPF and RIPv2 domains, **BoundaryR3 (ASBR)** performs bidirectional redistribution:
* **OSPF into RIPv2:** Routes learned via OSPF are injected into RIPv2 with a translated metric of `5` to ensure consistent path vector propagation across the branch.
* **RIPv2 into OSPF:** Branch subnets are redistributed into OSPF as Type 2 External (E2) routes, ensuring full reachability throughout the backbone.

### 3. Subnet Integration & Validation
* Core and transit segments are strictly mapped out to eliminate IP overlapping.
* Verification of correct transit routing via the dedicated `10.1.2.0/24` OSPF Area 1 network path.

---
