# 07 — Final Validation, Reboot Recovery, Limitations and Lessons

[← Logging](06-logging-and-hardening.md) | [Back to project overview](README.md)

## Test matrix (21 November 2026)

| Test | Observation | Outcome |
|---|---|---|
| Server WireGuard interface | `wg0`, `10.20.0.1/24` | Pass |
| Active WireGuard port | UDP `51820` | Pass |
| Windows peer | `10.20.0.2/32`, recent handshake and bidirectional transfer | Pass |
| Android peer | `10.20.0.3/32`, prior handshake and bidirectional transfer | Pass |
| Windows ping before reboot | 4/4 replies, 0% loss, avg 395 ms | Functional; high RTT |
| Windows DNS | `nslookup example.com 10.20.0.1` returned A and AAAA | Pass |
| Android DNS | Unbound logs from `10.20.0.3` | Pass |
| Android website | `https://example.com` loaded with mobile data and VPN on | Pass |
| Security group cleanup | Temporary UDP `4430` deleted; only `51820` listed | Pass |
| Unbound socket binding | Only `10.20.0.1:53` and `127.0.0.1:53` | Pass |
| WireGuard config validation | `wg-quick strip wg0` successful | Pass |
| Backup comparison | `Backup verified: IDENTICAL` | Pass |
| EC2 reboot recovery | WireGuard and Unbound both `active` | Pass |
| Persistent DNS logs | Pre-reboot entries still queryable after reboot | Pass |
| Windows ping after reboot | 4/4 replies, 0% loss, avg 476 ms | Functional; high RTT |
| Windows DNS after reboot | `example.com` returned A and AAAA | Pass |

## Recovery evidence

After reboot:

```text
$ systemctl is-active wg-quick@wg0
active
$ systemctl is-active unbound
active
$ ip -br addr show wg0
wg0 UNKNOWN 10.20.0.1/24
```

`UNKNOWN` is a normal operational state for some virtual/tunnel interfaces and did not indicate failure here. Windows successfully pinged the tunnel endpoint and resolved DNS afterward.

## What I learned

1. **Cloud routing and security groups solve different problems.** The route table determines where traffic goes; the SG determines whether relevant traffic is allowed.
2. **An active VPN UI is not proof of working connectivity.** Verify handshake, encrypted transfer, ICMP and application-layer DNS separately.
3. **Logs are a form of evidence.** `tcpdump` helped with transient packet visibility; journald preserved DNS events across a reboot.
4. **Change control matters even in a lab.** The UDP 4430 experiment needed an explicit rollback and SG cleanup.
5. **Hardening should be tested, not assumed.** I checked listener addresses, inbound rules, config permissions and recovery.
6. **Separate observations from hypotheses.** The office Wi-Fi issue and high latency do not have proven single root causes.

## Limitations and next improvements

- **High RTT:** 395–476 ms average in observed Windows tests. Investigate path quality and client networks before calling this performant.
- **Not a full tunnel:** Internet traffic is not NATed through EC2; the private subnet is not yet accessible through this lab's VPN routing.
- **Single instance:** No failover or high availability.
- **Dynamic public IP:** A stop/start may require updating client endpoints.
- **No central monitoring:** DNS logs are local to the instance; CloudWatch/CloudTrail/GuardDuty are future learning stages.
- **No production security assessment:** This is an educational implementation, not an audited VPN product.
- **Cost management:** Continue reviewing EC2, EBS, public IPv4 and monitoring charges; shut down or clean up when not needed.

## What's next in my learning journey?

Return to the broader [Cloud Security & DevSecOps roadmap](../../README.md), beginning with AWS monitoring and threat detection, then DevSecOps pipelines and infrastructure as code. I will document those as separate projects instead of mixing them into this VPN lab.
