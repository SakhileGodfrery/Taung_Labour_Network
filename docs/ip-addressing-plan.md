# IP Addressing Plan

Base block: `192.168.41.0/24`. Subnets are sized using VLSM with **25% growth headroom already built into each subnet** (per CR5) — so growth is absorbed from unused host addresses, with no renumbering.

| VLAN | Department | Subnet | Usable range | Broadcast | Gateway (router sub-int) | Current users | +25% growth | Usable hosts |
|---|---|---|---|---|---|---|---|---|
| 10 | Management | 192.168.41.0/27 | .1–.30 | .31 | .1 | 5 | ~7 | 30 |
| 20 | Employment Services | 192.168.41.32/27 | .33–.62 | .63 | .33 | 20 | ~25 | 30 |
| 30 | Finance | 192.168.41.64/28 | .65–.78 | .79 | .65 | 8 | ~10 | 14 |
| 40 | IT / Servers | 192.168.41.80/28 | .81–.94 | .95 | .81 | 5 | ~6 | 14 |
| 50 | Reception / Public | 192.168.41.96/27 | .97–.126 | .127 | .97 | 10 | ~13 | 30 |
| 99 | Switch management (native) | 192.168.41.128/28 | .129–.142 | .143 | .129 | — | — | 14 |
| — | WAN link, R-EDGE ↔ Primary ISP | 192.168.41.144/30 | .145–.146 | .147 | — | — | — | 2 |
| — | WAN link, R-EDGE ↔ Backup ISP | 192.168.41.148/30 | .149–.150 | .151 | — | — | — | 2 |
| — | Reserved for future expansion | 192.168.41.152/29 – .255 | — | — | — | — | — | 104 |

## VLSM working (subnet borrow order)

Sorted largest to smallest host requirement to avoid fragmentation:

1. VLAN 20 (Employment Services, 30 hosts needed) → `/27` → `192.168.41.32/27`
2. VLAN 10 (Management, 30 hosts needed) → `/27` → `192.168.41.0/27`
3. VLAN 50 (Reception, 30 hosts needed) → `/27` → `192.168.41.96/27`
4. VLAN 30 (Finance, 14 hosts needed) → `/28` → `192.168.41.64/28`
5. VLAN 40 (IT/Servers, 14 hosts needed) → `/28` → `192.168.41.80/28`
6. VLAN 99 (switch management, 14 hosts needed) → `/28` → `192.168.41.128/28`
7. WAN links (2 hosts each) → `/30` → `192.168.41.144/30`, `192.168.41.148/30`
8. Remainder (`.152–.255`) left unallocated as documented reserve.

## Notes

- All department subnets already have 2–3× headroom over the post-growth user count, so CR5 is absorbed without touching the addressing scheme.
- WAN link subnets are drawn from the assigned block for simulation purposes (in a real deployment these would typically be ISP-assigned).
- The `.152–.255` range is kept unallocated as a documented reserve, giving further room beyond the 25% CR5 figure if the client grows again later.
