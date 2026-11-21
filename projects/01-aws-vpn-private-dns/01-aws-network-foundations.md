# 01 — AWS Network Foundations: VPC, Subnets, Routing and Internet Gateway

[← Overview](README.md) | [Next →](02-ec2-iam-linux.md)

## Learning objective

Build the network foundation **before** deploying VPN software. I wanted to connect familiar concepts—VLANs, subnets, routing and firewalls—to AWS VPC components.

## What I built in AWS Console

| Order | Resource | Actual lab design | Why |
|---|---|---|---|
| 1 | Region | `ap-southeast-2` (Sydney) | Region available to my account; other regions were restricted by organization policy |
| 2 | VPC | `10.10.0.0/16` | Private address space for lab resources |
| 3 | Public subnet | `10.10.1.0/24`, AZ `ap-southeast-2a` | Place the internet-reachable VPN EC2 |
| 4 | Private subnet | `10.10.2.0/24`, same AZ | Practice isolation; reserved for future use |
| 5 | Internet Gateway | Attached to VPC | Enable internet routing for public subnet |
| 6 | Public route table | `0.0.0.0/0 → Internet Gateway` | Allow traffic toward internet gateway |
| 7 | Route table association | Public subnet associated with public route table | Route selection applies at subnet level |
| 8 | Private subnet routing | Local-only; no internet default route | No NAT Gateway expense or unneeded connectivity |
| 9 | Security group | Inbound UDP `51820` from `0.0.0.0/0` | Allow WireGuard clients to connect |

### Console walkthrough

1. **VPC → Your VPCs → Create VPC**: create `10.10.0.0/16`.
2. **VPC → Subnets → Create subnet**: select the VPC, then create `10.10.1.0/24` and `10.10.2.0/24` in the chosen AZ.
3. **VPC → Internet Gateways → Create internet gateway**: attach it to the VPC.
4. **VPC → Route Tables → Create route table**: add `0.0.0.0/0` targeting the Internet Gateway; associate only the public subnet.
5. Confirm the private subnet's associated route table has **local** VPC routes only. Avoid assigning an internet default route by accident.
6. **EC2 → Security Groups → Create security group**: allow inbound **Custom UDP 51820**, source `0.0.0.0/0`. Do not open TCP 22 or port 53 publicly.

> A subnet isn't public merely because its name says “public.” Public routing requires a route to an Internet Gateway, and the instance needs an appropriate public address and security rules. Routing and access control are different layers.

## What I checked

- The public route table points to the IGW and is associated with the intended subnet.
- The private subnet remains isolated from direct internet routing.
- The security group is limited to UDP 51820 after cleanup.

## Mistakes and learning

**Wrong WireGuard port in the security group:** I initially configured **UDP 5180** instead of **UDP 51820**. The tunnel could appear enabled on a client without useful connectivity. After checking the server listen port and security group, I corrected the rule. See the [troubleshooting diary](05-troubleshooting-diary.md).

**Region restrictions:** I encountered organization-level restrictions outside Sydney. The lesson was to distinguish AWS permission/policy failures from network or instance misconfiguration.

## Cost and scope

I intentionally avoided a NAT Gateway. This private subnet is **not** yet a working private workload accessed through WireGuard. Routing VPN clients to it would require additional route and forwarding design, plus appropriate security rules. Public IPv4, EC2 and storage can still incur charges.
