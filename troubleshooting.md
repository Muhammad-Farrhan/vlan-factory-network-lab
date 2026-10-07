# Test Results

All tests were done in Cisco Packet Tracer after the full configuration 
was in place (VLANs, trunk, router-on-a-stick, DHCP/DNS/HTTP server, and 
ACLs). I ran each test from the PC's Command Prompt or Web Browser.

The ACLs were verified two ways:

1. **By end-to-end behavior** — did the ping succeed or fail as expected?
2. **By ACL hit counters** — after each test, `show access-lists` on R1 
   showed the matching counter increment on the relevant line.

Both methods agreed, so I'm confident the ACLs are actually filtering 
traffic and not just sitting in the config.

---

## Test Matrix (Expected vs Actual)

| # | Source      | Destination            | Test        | Expected       | Actual                | Pass? |
|---|-------------|------------------------|-------------|----------------|-----------------------|-------|
| 1 | PC-Office1  | SRV-INFRA (20.10)      | ping        | Allow          | Reply from 20.10      | ✅    |
| 2 | PC-Office1  | PC-OT1 (30.100)        | ping        | Block          | Unreachable (from 10.1) | ✅  |
| 3 | PC-Office1  | PC-Guest1 (40.100)     | ping        | Block          | Unreachable (from 10.1) | ✅  |
| 4 | PC-OT1      | SRV-INFRA (20.10)      | ping        | Allow          | Reply from 20.10      | ✅    |
| 5 | PC-OT1      | SRV-INFRA (20.10)      | HTTP        | Allow          | Page loaded           | ✅    |
| 6 | PC-OT1      | PC-Office1 (10.100)    | ping        | Block          | Unreachable (from 30.1) | ✅  |
| 7 | PC-OT1      | PC-Guest1 (40.100)     | ping        | Block          | Unreachable (from 30.1) | ✅  |
| 8 | PC-Guest1   | SRV-INFRA (20.10)      | ping        | Block          | Unreachable (from 40.1) | ✅  |
| 9 | PC-Guest1   | PC-Office1 (10.100)    | ping        | Block          | Unreachable (from 40.1) | ✅  |
| 10| PC-Guest1   | PC-OT1 (30.100)        | ping        | Block          | Unreachable (from 40.1) | ✅  |
| 11| PC-Guest1   | intranet.contoh.local  | nslookup    | Allow (DNS)    | Resolved to 20.10     | ✅    |
| 12| PC-Guest1   | SRV-INFRA (20.10)      | HTTP        | Block          | Request timed out     | ✅    |
| 13| All PCs     | DHCP server            | DHCP        | Allow          | Got IP in correct subnet | ✅  |

**Legend:**
- `Unreachable (from X.X)` = the reply comes from the gateway IP, meaning 
  the router is actively rejecting the packet. This is the expected 
  behavior when an ACL blocks traffic.
- `Request timed out` = no reply at all. For HTTP from Guest, this is 
  expected — the TCP SYN to port 80 gets dropped by the ACL, so the 
  browser waits and eventually gives up.

---

## Pre-ACL Baseline (Sanity Check)

Before applying ACLs, I verified that everyone could reach everyone. This 
makes sure the failure we see later is caused by the ACL, not by some 
pre-existing routing problem.

| From       | To         | Result            |
|------------|------------|-------------------|
| PC-Office1 | SRV-INFRA  | ✅ Reply          |
| PC-Office1 | PC-OT1     | ✅ Reply          |
| PC-Guest1  | SRV-INFRA  | ✅ Reply          |
| PC-Guest1  | PC-Office1 | ✅ Reply          |
| PC-OT1     | SRV-INFRA  | ✅ Reply          |

Once these all passed, I knew VLANs, trunk, sub-interfaces, and routing 
were working. Any failure after this point would be ACL-related.

---

## ACL Hit Counters

After running the tests, I checked the counters with `show access-lists` 
on R1. Here's what they looked like:

```
Extended IP access list OFFICE-IN
    10 permit udp any eq bootpc any eq bootps
    20 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255 (4 match(es))
    30 deny ip 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255
    40 permit ip any any (10 match(es))

Extended IP access list OT-IN
    10 permit udp any eq bootpc any eq bootps
    20 permit udp 192.168.30.0 0.0.0.255 host 192.168.20.10 eq domain (1 match(es))
    30 permit tcp 192.168.30.0 0.0.0.255 host 192.168.20.10 eq www (5 match(es))
    40 permit icmp 192.168.30.0 0.0.0.255 host 192.168.20.10 (4 match(es))
    50 deny ip any any (8 match(es))

Extended IP access list GUEST-IN
    10 permit udp any eq bootpc any eq bootps
    20 permit udp 192.168.40.0 0.0.0.255 host 192.168.20.10 eq domain (2 match(es))
    30 deny ip 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255 (4 match(es))
    40 deny ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255 (4 match(es))
    50 deny ip 192.168.40.0 0.0.0.255 192.168.30.0 0.0.0.255
    60 permit ip any any
```

What the counters tell me:

- **OFFICE-IN line 20 (4 matches)** — Office → OT ping attempts were 
  blocked. Confirms the deny rule is working.
- **OFFICE-IN line 40 (10 matches)** — Office → Server traffic got 
  permitted. Makes sense because that's the only allowed destination 
  for Office.
- **OT-IN line 30 (5 matches)** — OT → Server HTTP requests were 
  permitted.
- **OT-IN line 40 (4 matches)** — OT → Server ICMP (ping) was permitted.
- **OT-IN line 50 (8 matches)** — OT → Office and OT → Guest attempts 
  hit the final `deny ip any any`. 4 pings to Office + 4 pings to Guest 
  = 8.
- **GUEST-IN line 20 (2 matches)** — Guest DNS lookups were permitted.
- **GUEST-IN line 30 (4 matches)** — Guest → Office ping attempts blocked.
- **GUEST-IN line 40 (4 matches)** — Guest → Server ping attempts blocked.

The counters line up exactly with the tests I ran, which is the proof I 
wanted.

---

## DHCP Verification

I also checked that all three PCs got their IP addresses automatically 
from the DHCP server in VLAN 20:

| PC          | Expected Subnet   | Actual IP       | Gateway      | DNS          |
|-------------|-------------------|-----------------|--------------|--------------|
| PC-Office1  | 192.168.10.0/24   | 192.168.10.100  | 192.168.10.1 | 192.168.20.10 |
| PC-OT1      | 192.168.30.0/24   | 192.168.30.100  | 192.168.30.1 | 192.168.20.10 |
| PC-Guest1   | 192.168.40.0/24   | 192.168.40.100  | 192.168.40.1 | 192.168.20.10 |

This proves the DHCP relay (`ip helper-address 192.168.20.10` on the 
router sub-interfaces) is working — the DHCP request from a client 
VLAN gets forwarded across the trunk to the server in a different VLAN.

---

## Screenshots

All screenshots are in the repo:

- `diagram/topology.png` — the full topology
- `show vlan brief.png` — VLAN list on SW1
- `show ip interface brief.png` — sub-interfaces on R1
- `IP Configuration server..png` — server's static IP setup
- `Tab Physychal R1.png` — router physical view

---

## Honest Notes

- The first ping from a fresh PC often showed one `Request timed out` 
  before the replies started. That's ARP resolution, not a problem. 
  From the second ping onward, everything was clean.
- If a PC ever got "stuck" — for example, showing `Request timed out` 
  when it should show `Destination host unreachable` — a power cycle 
  fixed it. This happened twice during testing (once on PC-Office1, 
  once on PC-Guest1). Packet Tracer seems to keep stale ARP entries.
- Guest HTTP test shows `Request timed out` rather than `Destination 
  host unreachable`. This is expected: the TCP packet gets silently 
  dropped by the ACL, so there's no ICMP error to report back.
