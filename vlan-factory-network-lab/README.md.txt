# VLAN Factory Network Lab

Simulasi jaringan pabrik semikonduktor PT Contoh Semikon dengan segmentasi VLAN, 
inter-VLAN routing, DHCP relay, DNS internal, dan ACL.

## Skenario
Jaringan flat berbahaya karena semua perangkat bisa saling akses. 
Proyek ini memisahkan jaringan menjadi 4 zona: Office, Server, OT, Guest.

## Topologi
![Topology](diagram/topology.png)

## VLAN & Addressing
| VLAN | Name | Subnet | Gateway |
|------|------|--------|---------|
| 10 | OFFICE | 192.168.10.0/24 | 192.168.10.1 |
| 20 | SERVER | 192.168.20.0/24 | 192.168.20.1 |
| 30 | PRODUCTION-OT | 192.168.30.0/24 | 192.168.30.1 |
| 40 | GUEST | 192.168.40.0/24 | 192.168.40.1 |
| 99 | NATIVE | - | - |

## Access Policy
| From / To | Office | Server | OT | Guest |
|-----------|--------|--------|-----|-------|
| Office | ✅ | ✅ | ❌ | ❌ |
| Server | ✅ | ✅ | ✅ | ❌ |
| OT | ❌ | DNS, HTTP, ping | ✅ | ❌ |
| Guest | ❌ | DNS only | ❌ | ✅ |

## Tech Stack
- Cisco Packet Tracer
- VLAN & 802.1Q trunking
- Router-on-a-stick (inter-VLAN routing)
- DHCP relay (ip helper-address)
- DNS internal
- Extended ACL

## Test Results
See [test-results.md](test-results.md)

## Lessons Learned
- ACL must permit DHCP first (port 68 → 67), otherwise clients lose IP after ACL applied.
- Restarting Packet Tracer PCs clears stale ARP cache.
- Save (Ctrl+S) frequently - Packet Tracer does not autosave.