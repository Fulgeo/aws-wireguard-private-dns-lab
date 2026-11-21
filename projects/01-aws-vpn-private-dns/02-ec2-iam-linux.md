# 02 — EC2, IAM Role, Session Manager and Linux Administration

[← Networking](01-aws-network-foundations.md) | [Next →](03-wireguard-setup.md)

## Goal

Deploy a small Linux host without exposing SSH to the public internet, and learn the distinction between an **instance IAM role** and network permissions.

## What I configured

1. Launched a `t3.micro` Ubuntu EC2 instance in the **public subnet** (`10.10.1.0/24`). The instance had a public IPv4 address for WireGuard client access.
2. Created/attached an instance role with AWS managed policy `AmazonSSMManagedInstanceCore`.
3. Connected using **EC2 → Connect → Session Manager** through AWS Console.
4. Left public SSH (TCP 22) closed in the security group.
5. Used Linux commands to inspect network interfaces, service state, listening ports and logs.

## Why these controls matter

- **IAM role:** grants the instance permission to communicate with Systems Manager services. It is not a substitute for operating-system authorization or network controls.
- **Session Manager:** provides a management path without adding a public SSH inbound rule. Availability also depends on SSM Agent and network access to SSM endpoints.
- **Security group:** controls allowed traffic at the EC2 network interface; it does not configure WireGuard or DNS itself.

## Commands I used during verification

```bash
ip -br addr
systemctl is-active wg-quick@wg0
systemctl is-active unbound
ss -tulnp
journalctl -u unbound --no-pager | tail -n 20
```

## Linux skills I practiced

- Editing `/etc` configuration files and validating them before restart.
- Using `systemctl` for service state and automatic startup.
- Reading `journalctl` to investigate events.
- Checking listening sockets with `ss` and traffic with `tcpdump`.
- Using restrictive permissions (`0600`) for WireGuard configuration.
- Backing up configuration before changing it.

## Boundaries and open questions

The instance's public IP was not an Elastic IP; it may change after stop/start. A **reboot** generally preserves the address, but that is different from stop/start. The lab did not implement high availability, patch automation, or a full disaster recovery procedure.
