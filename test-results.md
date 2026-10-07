# Test Results

## Pre-ACL Connectivity
| From | To | Result |
|------|----|--------|
| PC-Office1 | SRV-INFRA | ✅ Reply |
| PC-OT1 | SRV-INFRA | ✅ Reply |
| PC-Guest1 | SRV-INFRA | ✅ Reply |

## Post-ACL Connectivity
| # | From | To | Expected | Actual |
|---|------|----|----------|--------|
| 1 | Office | Server | ✅ Allow | ✅ Reply |
| 2 | Office | OT | ❌ Deny | ✅ Unreachable |
| 3 | Office | Guest | ❌ Deny | ✅ Unreachable |
| 4 | OT | Server (ping) | ✅ Allow | ✅ Reply |
| 5 | OT | Server (HTTP) | ✅ Allow | ✅ Page loaded |
| 6 | OT | Office | ❌ Deny | ✅ Unreachable |
| 7 | OT | Guest | ❌ Deny | ✅ Unreachable |
| 8 | Guest | Server (ping) | ❌ Deny | ✅ Unreachable |
| 9 | Guest | Office | ❌ Deny | ✅ Unreachable |
| 10 | Guest | DNS | ✅ Allow | ✅ Resolved |
| 11 | Guest | HTTP Server | ❌ Deny | ✅ Timeout |