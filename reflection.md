# Reflection

## What was built

A segmented network for the Taung Labour Department office on `192.168.41.0/24`, with five department VLANs (Management, Employment Services, Finance, IT/Servers, Reception), a router-on-a-stick edge router (R-EDGE) handling inter-VLAN routing, DHCP relay, NAT, and dual-ISP default routing, plus an access-control restriction keeping the public Reception VLAN out of Finance and IT.

## Design decisions

- **VLSM addressing sized for 25% growth (CR5) from the start.** Rather than adding a new subnet later, every department's subnet was sized against its post-growth headcount, so the change request is absorbed by unused host addresses already inside each VLAN — no renumbering needed.
- **Floating static default route for the backup ISP** (administrative distance 5 vs. the primary's 1) — this is the assigned Default Routing challenge, and directly satisfies the design constraint that Finance needs a backup internet path, since the whole network (Finance included) fails over together.
- **Reception restricted from Finance and IT via an extended ACL** on R-EDGE's VLAN 50 sub-interface, with an explicit DNS permit statement first so Reception can still resolve names against the server in the IT VLAN even though general traffic to that subnet is blocked.
- **Left every other department fully open to each other** — no ACLs between Management, Employment Services, Finance, and IT — since the brief didn't require broader segmentation and it wasn't worth the added complexity for this challenge's scope.

## Problems hit and how they were solved

1. **Switches rejecting every config line.** Early on, config commands were being typed at the `Switch>` prompt (user EXEC mode) instead of `Switch(config)#`. Fixed by always running `enable` then `configure terminal` before pasting any config.

2. **Router's third interface couldn't be created.** R-EDGE (a 4321/4331) only has two onboard Gigabit ports, but three physical links were needed (2 WAN + 1 LAN trunk). A `NIM-ES2-4` module was tried first to add a third port, but every attempt to create a dot1Q sub-interface on it failed with `%Cannot create sub-interface` — this module is actually a small Layer-2 switch, not a routed interface, and even `no switchport` couldn't convert it. Fixed by removing `NIM-ES2-4` and fitting a `NIM-2T` (serial) module instead, moving the Backup ISP link onto the new `Serial0/1/0` port and keeping the LAN trunk on the onboard `Gi0/0/0`, which reliably supports sub-interfaces.

3. **Serial clock rate.** Checked `show controllers serial0/1/0` before assuming a manual `clock rate` was needed — it turned out Packet Tracer had already assigned one automatically on the DCE end, so no extra config was required there.

4. **Primary ISP link showing "up" on one side but "down" on protocol.** `show ip interface brief` on both R-EDGE and ISP-PRIMARY revealed the IP address had been configured on `Gi0/0/1`, but the actual cable was plugged into `Gi0/0/0`. The interface *numbering* doesn't matter — only that the config matches whichever port is physically cabled. Fixed by moving the IP configuration to the correct, cabled interface.

5. **`ping 8.8.8.8` failing from every PC.** This wasn't a routing/NAT fault — `8.8.8.8` didn't exist anywhere in the simulated topology yet. Fixed by adding a loopback interface (`8.8.8.8/32`) on both ISP-PRIMARY and ISP-BACKUP to act as simulated internet hosts, plus a return route back to `192.168.41.0/24`.

6. **`show ip nat translations` staying empty even with a live ping.** Investigation traced this to `GigabitEthernet0/0/1` (the Primary ISP link) having been left `shutdown` from an earlier failover test. With the primary interface down, the router's default route correctly failed over to the backup path (proving the routing design itself works) — but the single `ip nat inside source list NAT_ACL interface GigabitEthernet0/0/1 overload` statement only knew about the primary interface, so NAT had nothing to translate through while traffic was actually flowing over the backup. This confirmed, in practice, the single-interface NAT limitation flagged as a design trade-off from the start. Fixed two ways: brought the primary interface back up for normal operation, and added a second NAT overload statement (`ip nat inside source list NAT_ACL interface Serial0/1/0 overload`) plus a matching loopback and return route on ISP-BACKUP, so the network now also has working internet access while genuinely running on the backup path — not just a routing table that says it should.

## What this demonstrates

The troubleshooting process above (mode errors → hardware module limitation → cabling mismatch → missing simulated destination → single-interface NAT limitation) mirrors a real-world deployment process: verify software config, then hardware capability, then physical layer, then reachability, then edge-case behaviour under failure — in that order — rather than assuming the first layer that looks wrong is the actual fault. Item 6 in particular moved the dual-ISP NAT redundancy from an "optional extension" noted in the config comments to something actually implemented and tested, once the failover test itself revealed it was needed for the network to be genuinely resilient rather than resilient on paper.

## CR5 (25% growth) — confirmation

No VLAN, subnet, or gateway address changed to accommodate the 25% growth figure in the brief. Every department subnet already contains at least 2–3x headroom over its post-growth user count (see `docs/ip-addressing-plan.md`), so adding the extra ~25% of users to any department only consumes addresses that were already free inside that VLAN's existing range.
