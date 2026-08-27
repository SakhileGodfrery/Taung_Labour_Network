# Network Design — Physical & Logical Topology

## 1. Physical topology

**Primary ISP** and **Backup ISP** ─▶ **Edge router (R-EDGE)** ─▶ **Core switch (SW-CORE)** ─▶ 5 access switches (one per department: Management, Employment Services, Finance, IT/Server, Reception) ─▶ end devices (PCs, printers, servers, one AP for reception if needed).

- R-EDGE holds the two WAN interfaces (Primary ISP, Backup ISP) and one LAN interface trunked to SW-CORE (router-on-a-stick for inter-VLAN routing).
- SW-CORE trunks VLANs down to each department's access switch.
- The DHCP/DNS server sits in VLAN 40 (IT/Servers), reachable from all VLANs via inter-VLAN routing.

See `diagrams/physical-topology.png`.

## 2. Logical topology

- R-EDGE receives a **default route via Primary ISP** (administrative distance 1) and a **floating static default route via Backup ISP** (administrative distance 5, e.g. `ip route 0.0.0.0 0.0.0.0 <backup-next-hop> 5`). The router only installs the backup route in its routing table if the primary route is withdrawn.
- Five VLAN subnets hang off R-EDGE's sub-interfaces (router-on-a-stick), each mapped to a department.
- Finance (VLAN 30) reaches the internet through R-EDGE like every other VLAN, so when the floating route activates on primary-link failure, Finance (and everyone else) keeps internet connectivity — satisfying the "backup path for Finance" constraint through the edge/ISP design.

See `diagrams/logical-topology.png`.

## 3. Design decisions & justification

| Decision | Justification |
|---|---|
| Router-on-a-stick for inter-VLAN routing | Single edge router keeps the design appropriately scoped for an "Intermediate" difficulty challenge; avoids introducing a separate L3 core switch that isn't required by the brief. |
| Floating static default route for backup ISP | Directly implements the assigned "Default Routing (edge/ISP path design)" challenge, and satisfies the design constraint (backup path) without extra routing protocols. |
| VLAN segmentation by department | Contains broadcast traffic, supports department-level access control (e.g. restricting Finance/IT from public Reception access). |
| DHCP/DNS server placed in its own VLAN (40) | Centralises core services, reachable from every VLAN via inter-VLAN routing, easy to secure separately from end-user VLANs. |
| Growth absorbed via subnet headroom, not new subnets | Directly satisfies CR5's "without renumbering" requirement — every device keeps its existing subnet as headcount grows. |

## 4. Verification plan (for Milestone 2 / final submission)

- End-to-end connectivity test between every VLAN and the internet.
- Simulated primary ISP link failure (shut the primary WAN interface) → confirm the floating default route is installed and traffic (including Finance's) continues to flow via the backup ISP.
- `show ip route` before/after failure to demonstrate the administrative-distance behaviour.
- DHCP lease verification on each VLAN.
