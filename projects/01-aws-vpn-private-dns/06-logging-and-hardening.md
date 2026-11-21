# 06 — DNS Logging, Retention and Security Hardening

[← Troubleshooting](05-troubleshooting-diary.md) | [Next →](07-validation-and-lessons.md)

## Goal

Move from a functioning VPN to a lab that produces useful, retained security evidence without leaving unnecessary ports exposed.

## Enable DNS query logging

Created `/etc/unbound/unbound.conf.d/dns-logging.conf`:

```yaml
server:
    verbosity: 1
    log-queries: yes
    log-replies: no
    use-syslog: yes
```

Then validated and applied:

```bash
sudo unbound-checkconf
sudo systemctl restart unbound
sudo systemctl is-active unbound
journalctl -u unbound --since '5 minutes ago' --no-pager
```

### Evidence

Sanitized example format from actual lab observations:

```text
Nov 21 09:31:35 unbound: info: 10.20.0.2 google.com. A IN
Nov 21 09:33:59 unbound: info: 10.20.0.1 example.org. A IN
Nov 21 09:40:00 unbound: info: 10.20.0.3 mtalk.google.com. A IN
```

These events identify the **client IP**, **queried domain**, **record type**, and **timestamp**. They do not prove that a website was fully visited or that the DNS answer was used by an application.

## Persistent journal retention

Before changing retention, `journalctl --disk-usage` showed approximately **16 MB**. Both `/var/log/journal` and `/run/log/journal` existed, but our configuration grep found no explicit values.

Created `/etc/systemd/journald.conf.d/security-lab.conf`:

```ini
[Journal]
Storage=persistent
SystemMaxUse=100M
RuntimeMaxUse=32M
MaxRetentionSec=7day
```

Applied with `sudo systemctl restart systemd-journald`. The policy applies to the **whole journal**, not only Unbound. Journal size and retention are bounds/targets subject to journald behavior; they are not a guarantee of exactly seven days of logs.

**Validation:** After EC2 reboot, `journalctl -u unbound` still returned the earlier `google.com` and `example.org` queries. This is direct evidence that historical journal entries survived the reboot.

## Security control checklist

| Control | Evidence / scope |
|---|---|
| Public ingress | Only inbound UDP `51820` remained in the security group |
| Public SSH | No inbound TCP 22 rule; used Session Manager |
| Public DNS | No inbound 53 rule; Unbound bound only to loopback and WireGuard address |
| VPN peer segmentation | Separate client keys and `/32` AllowedIPs on server |
| Config file permissions | `wg0.conf` mode `0600`, owned by root |
| Backup | Verified backup byte-for-byte identical with `cmp -s` |
| Service startup | `wg-quick@wg0` and `unbound` active after reboot |
| Query logging | Windows and Android query events visible in journal |

## Security caveats

DNS query logs can reveal sensitive browsing and application metadata. Keep access restricted, avoid publishing raw logs, and review retention against your privacy requirements. This lab did not implement centralized SIEM ingestion, encryption/key rotation procedures, DNS threat blocking, or comprehensive audit coverage.
