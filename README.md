# CMPG 325 — Individual Semester Project
## Computer Networks | Project ID: CMPG325-2026-095

**Student:** Sakhile Mthimunye
**Client:** Labour Department Taung Office (Taung) — Government sector
**Assigned addressing block:** `192.168.41.0/24`
**Assigned technical challenge:** Default Routing (edge/ISP path design) — Intermediate
**Project period:** 14 August 2026 – 16 October 2026

---

## Project overview

This repository documents the design, implementation, testing, and evidence for a Cisco Packet Tracer network built for the Taung Labour Department office. The office is organised into five functional groups — Management, Employment Services, Finance, IT/Servers, and Reception — each isolated on its own VLAN.

The network is built around two requirements from the client brief:

1. **Design constraint** — a backup internet path is required for the Finance function.
2. **Change request CR5** — user numbers will grow by 25%; the addressing plan must absorb this without renumbering.

These are addressed together through the assigned technical challenge: the edge router holds a **primary default route** to the main ISP and a **floating static default route** to a backup ISP, so internet access (including Finance's) survives a primary-link failure. Dual NAT overload rules (one per WAN interface) keep internet access working regardless of which path is active. The IP addressing plan builds 25%+ headroom into every VLAN subnet from the start, so growth never requires re-addressing a device.

As an additional access-control measure, an extended ACL restricts the public-facing Reception VLAN from reaching Finance or IT, while preserving DNS resolution — see `docs/network-design.md` for details.

## Repository structure

```
├── README.md                     ← you are here
├── docs/
│   ├── client-requirements.md    Client needs, functional & non-functional requirements
│   ├── network-design.md         Physical and logical topology, design decisions, ACL
│   └── ip-addressing-plan.md     VLSM subnetting, addressing table
├── diagrams/
│   ├── physical-topology.png
│   └── logical-topology.png
├── configs/                      Full running-config for every device
│   ├── R-EDGE.txt
│   ├── SW-CORE.txt
│   ├── SW-MGMT.txt
│   ├── SW-EMP.txt
│   ├── SW-FIN.txt
│   ├── SW-IT.txt
│   └── SW-RECEP.txt
├── packet-tracer/
│   └── taung-labour-network.pkt  Final working Packet Tracer file
├── screenshots/                  Configuration and testing evidence
└── reflection.md                 Build notes and troubleshooting log
```

## Milestones

| Milestone | Date | Status |
|---|---|---|
| Milestone 1 — Client design review | 28 Aug 2026 | ✅ Requirements, topology, addressing plan, repo scaffold |
| Milestone 2 — Client implementation review | 02 Oct 2026 | ✅ device configs and Default Routing challenge implemented; testing evidence and final `.pkt` being finalised |
| Final submission | 16 Oct 2026 | ⬜ .pkt, GitHub portfolio, technical report, video demonstration |

## How to review this project

Start with `docs/client-requirements.md` for the brief interpretation, then `docs/network-design.md` for the topology, routing, and access-control decisions, then `docs/ip-addressing-plan.md` for the full VLSM breakdown. Device configurations are in `configs/`, and `reflection.md` documents the troubleshooting process behind the final working setup (hardware module limitations, cabling fixes, and NAT-over-failover behaviour).
