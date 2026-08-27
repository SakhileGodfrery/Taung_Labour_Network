# Client Requirements Analysis

**Client:** Labour Department Taung Office (Taung) | **Industry:** Government
**Client ID:** CLI-095 | **Project ID:** CMPG325-2026-095

## 1. Assigned parameters

| Item | Detail |
|---|---|
| Addressing block | `192.168.41.0/24` |
| Assigned technical challenge | Default Routing (edge/ISP path design) — Intermediate |
| Design constraint | A backup internet path is required for the Finance function |
| Change request (CR5) | User numbers will grow by 25% — the addressing plan must absorb this **without renumbering** |

## 2. Interpreting the client's needs

The Taung Labour Department office is a small government branch office. Based on the functions a Labour Department branch typically performs, the network must support the following departmental groups:

- **Management/Administration** — office management, HR, general admin staff
- **Employment Services** — case workers handling job-seeker registration, UIF claims, employer registrations (the largest user group)
- **Finance** — payroll and procurement; this is the group named explicitly in the design constraint and needs guaranteed internet availability
- **IT/Server room** — internal servers (DHCP/DNS) and network management devices
- **Reception/Public access** — front-desk staff and public-facing terminals for job seekers

## 3. Functional requirements derived from the brief

- Logical **segmentation** of each department using VLANs, to contain broadcast traffic and apply department-appropriate access control.
- **DHCP** services so end devices in each VLAN receive addressing automatically.
- **DNS** resolution for internal and external name lookups.
- **Inter-VLAN routing** so departments can reach shared resources (server VLAN) and the internet.
- **Redundant internet path**: a primary ISP link plus a backup ISP link, with the backup taking over automatically if the primary fails — protecting the Finance function specifically, per the design constraint.
- **Default routing configuration** at the edge: a primary default route to the main ISP, and a **floating static default route** (higher administrative distance) to the backup ISP — this is the assigned technical challenge.
- An **addressing plan with built-in headroom**, so CR5's 25% growth is absorbed by unused host addresses already inside each VLAN's subnet — no re-addressing of any device required.

## 4. Non-functional requirements

- **Security**: department traffic separated by VLAN; Finance and IT restricted from general/public access.
- **Reliability**: no single point of failure on the internet edge (dual ISP paths).
- **Scalability**: room for 25% user growth per department without redesign.
- **Documentation & testability**: every design decision and test must be captured in this GitHub portfolio and demonstrated on video.
