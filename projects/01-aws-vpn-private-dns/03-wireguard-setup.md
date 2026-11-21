# 03 — WireGuard: Server, Windows Client and Android Client

[← EC2/IAM](02-ec2-iam-linux.md) | [Next →](04-unbound-private-dns.md)

## Goal and scope

Provide encrypted connectivity to the AWS server and its private DNS service, using **split tunneling** rather than routing all browsing traffic through AWS.

## Install WireGuard on EC2

```bash
sudo apt update
sudo apt install -y wireguard
sudo install -d -m 700 /etc/wireguard
```

Generate a **unique** server key pair locally, keeping the private key private:

```bash
sudo sh -c 'umask 077; wg genkey > /etc/wireguard/server.key'
sudo sh -c 'wg pubkey < /etc/wireguard/server.key > /etc/wireguard/server.pub'
```

Create separate client key pairs using each client's WireGuard app. Never commit or paste private keys into public documentation.

## Server configuration pattern

`/etc/wireguard/wg0.conf` (example only; **replace placeholders**):

```ini
[Interface]
Address = 10.20.0.1/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY_NOT_FOR_GIT>

[Peer]
PublicKey = <WINDOWS_PUBLIC_KEY>
AllowedIPs = 10.20.0.2/32

[Peer]
PublicKey = <ANDROID_PUBLIC_KEY>
AllowedIPs = 10.20.0.3/32
```

```bash
sudo chmod 600 /etc/wireguard/wg0.conf
sudo sysctl -w net.ipv4.ip_forward=1
# Persistent setting used in lab: /etc/sysctl.d/99-wireguard.conf
sudo systemctl enable --now wg-quick@wg0
sudo wg show wg0
```

**Note:** IPv4 forwarding was enabled during the lab, but that alone does not make this a full-tunnel VPN. There is no general internet NAT configuration in this project.

## Client configuration pattern

Windows (`10.20.0.2/32`) and Android (`10.20.0.3/32`) each use their own private key:

```ini
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY_NOT_FOR_GIT>
Address = 10.20.0.2/32
DNS = 10.20.0.1
MTU = 1420

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = <YOUR_EC2_PUBLIC_IP>:51820
AllowedIPs = 10.20.0.0/24
PersistentKeepalive = 25
```

Use `10.20.0.3/32` on Android. The DNS setting directs client lookups to Unbound; the `AllowedIPs` entry routes that private DNS destination through the tunnel.

## Validation

```bash
sudo wg show wg0
sudo ss -lunp | grep 51820
```

On Windows:

```powershell
ping 10.20.0.1
nslookup example.com 10.20.0.1
```

Observed after the final EC2 reboot: 4/4 ping replies, 0% loss; DNS lookup succeeded. Android generated DNS events from `10.20.0.3` and successfully loaded `https://example.com` over mobile data with the VPN active.

## Why the first attempt didn't work

The AWS inbound rule initially allowed UDP **5180**, not **51820**. Correcting the security group fixed the basic connectivity problem on the personal hotspot. Corporate office Wi-Fi had separate intermittent/filtered behavior; it was not conclusively attributed to one root cause. Full timeline: [troubleshooting diary](05-troubleshooting-diary.md).
