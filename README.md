# Cloud Security & DevSecOps — My Project-Based Learning Journey

**From SOC L1 investigations to building, securing, and observing cloud infrastructure.**

I work as an L1 cybersecurity/SOC analyst. In my day-to-day work I investigate alerts and review evidence. I started this repository because I wanted to understand the systems behind those alerts: how cloud networks are built, how access is controlled, how services communicate, and what operational evidence is available when something fails.

This is **a learning journal, not a claim of production expertise**. I document the sequence of work, my mistakes, troubleshooting hypotheses, what actually fixed an issue, and what remains unverified. My method is **build → break or observe → investigate → fix → validate → document**.

## Learning direction

My longer-term goal is to develop **AWS Cloud Security and DevSecOps** skills through practical projects. The roadmap below is a working learning plan, not a verbatim copy of an earlier course syllabus.

| Stage | Focus | Status |
|---|---|---|
| 01 | AWS fundamentals, networking, IAM and Linux | Practiced in Project 01 |
| 02 | Secure connectivity, private DNS and monitoring | Project 01 validated |
| 03 | AWS logging: CloudTrail, CloudWatch, VPC Flow Logs | Planned |
| 04 | Threat detection and incident investigation: GuardDuty | Planned |
| 05 | DevSecOps foundations: Git, CI/CD, secrets and scanning | Planned |
| 06 | Infrastructure as Code and security automation | Planned |

The sequence may change as I learn. Each project will have its own folder and evidence-based write-up.

## Project index

### [Project 01 — AWS WireGuard VPN + Private DNS](projects/01-aws-vpn-private-dns/README.md)

I built an AWS VPC from scratch, deployed a Linux EC2 instance, managed it through Session Manager, installed WireGuard, connected Windows and Android clients, added an Unbound DNS resolver, implemented DNS query logging, and tested recovery after reboot.

**Read in order:**

1. [Project overview and architecture](projects/01-aws-vpn-private-dns/README.md)
2. [01 — AWS foundations: VPC, subnets, routes, Internet Gateway](projects/01-aws-vpn-private-dns/01-aws-network-foundations.md)
3. [02 — EC2, IAM, Session Manager and Linux](projects/01-aws-vpn-private-dns/02-ec2-iam-linux.md)
4. [03 — WireGuard server and client setup](projects/01-aws-vpn-private-dns/03-wireguard-setup.md)
5. [04 — Private DNS with Unbound](projects/01-aws-vpn-private-dns/04-unbound-private-dns.md)
6. [05 — Troubleshooting diary: mistakes, evidence and fixes](projects/01-aws-vpn-private-dns/05-troubleshooting-diary.md)
7. [06 — DNS logging, retention and security controls](projects/01-aws-vpn-private-dns/06-logging-and-hardening.md)
8. [07 — Validation, reboot recovery and limitations](projects/01-aws-vpn-private-dns/07-validation-and-lessons.md)

## What I want readers to take away

I don't want to present a polished configuration as if everything worked on the first try. For me, cloud security learning is also about **understanding failure modes**, testing assumptions with logs and packet captures, and knowing what a test does *not* prove.

## Repository safety

All configurations are **sanitized examples**. Never upload WireGuard private keys, real client configurations, AWS credentials, account identifiers, public endpoints, packet captures, or sensitive DNS logs. Lab private CIDRs are shown to explain architecture. Costs and AWS availability vary by account and region.

## Sharing

A draft LinkedIn announcement is in [LINKEDIN_POST.md](LINKEDIN_POST.md). Feedback and constructive technical corrections are welcome.
