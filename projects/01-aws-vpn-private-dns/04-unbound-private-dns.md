# 04 — Unbound Private DNS Resolver over WireGuard

[← WireGuard](03-wireguard-setup.md) | [Next →](05-troubleshooting-diary.md)

## Why add DNS?

I wanted the VPN to do something observable and useful. A private resolver lets me see client DNS activity, practice restricting network services, and generate investigation evidence similar to what a SOC analyst reviews.

## Install and configure

```bash
sudo apt update
sudo apt install -y unbound dnsutils
```

The lab used `/etc/unbound/unbound.conf.d/wireguard.conf`:

```yaml
server:
    interface: 127.0.0.1
    interface: 10.20.0.1
    access-control: 127.0.0.0/8 allow
    access-control: 10.20.0.0/24 allow
    access-control: 0.0.0.0/0 refuse
    do-ip4: yes
    do-ip6: no
    hide-identity: yes
    hide-version: yes
```

**Important distinction:** `do-ip6: no` controls Unbound's use of IPv6 transport; it does **not** mean Unbound cannot answer AAAA record queries. We observed AAAA and A results from clients.

```bash
sudo unbound-checkconf
sudo systemctl enable --now unbound
sudo ss -tulnp | grep ':53 '
dig @10.20.0.1 example.com
```

## What I verified

- Unbound listened on **TCP/UDP** `10.20.0.1:53` and `127.0.0.1:53`.
- `systemd-resolved` also used `127.0.0.53` and `127.0.0.54`—normal local stub listeners.
- Unbound did **not** bind to `0.0.0.0:53` or the EC2 public-facing address.
- AWS inbound security group allowed only UDP `51820`, not public port 53.
- Windows `nslookup example.com 10.20.0.1` returned valid IPv4/IPv6 answers.
- Android DNS queries from `10.20.0.3` appeared in server-side captures and later in persistent logs.

## Lessons

`Server: UnKnown` in Windows `nslookup` did not mean the lookup failed; it usually reflected missing reverse DNS for the resolver IP. The meaningful evidence was the successful answer section and server-side logs. A DNS service listening on a private tunnel address is a more controlled design than exposing an open resolver to the internet.
