# Internal DNS

## Goal

Provide stable internal hostnames for infrastructure and selected services without relying on public DNS or the ISP connection.

The internal namespace is:

```text
home.arpa
```

---

## DNS Servers

Both Pi-hole instances contain matching local DNS records.

This keeps name resolution available if one Pi-hole instance is offline and matches the existing redundant DNS design.

Client VLANs continue to receive both Pi-hole servers through OPNsense DHCP.

---

## Infrastructure Names

Infrastructure management interfaces use DNS names directly instead of being placed behind the reverse proxy.

Examples include:

```text
proxmox-01.home.arpa
proxmox-02.home.arpa
proxmox-03.home.arpa
proxmox-04.home.arpa
pbs.home.arpa
opnsense.home.arpa
pihole-a.home.arpa
pihole-b.home.arpa
omada.home.arpa
```

The normal management ports are still used where required.

This keeps the access model simple and avoids adding another dependency in front of core infrastructure management.

---

## Application Records

Selected application names point to the Caddy reverse proxy rather than directly to the backend service.

Examples documented in this repository include:

```text
proxy.home.arpa
grafana.home.arpa
dashy.home.arpa
uptime.home.arpa
homeassistant.home.arpa
```

These records resolve to the reverse proxy, which then forwards the request to the correct internal backend.

Additional internal application records may follow the same pattern without being documented individually here.

---

## Pi-hole Configuration Note

During implementation, the secondary Pi-hole needed its interface behavior changed from a local-only request model so routed internal VLAN clients could query it correctly.

The change did not expose DNS to the WAN. OPNsense firewall policy continues to control which internal networks are allowed to reach DNS.

---

## OPNsense Rebind Protection

OPNsense initially rejected access through its new internal hostname because DNS-rebind protection did not recognize the name.

The internal hostname was added under the OPNsense alternate-hostname setting while global DNS-rebind protection remained enabled.

This allowed the expected management name without disabling the broader protection.

---

## Validation

DNS validation included:

- Querying each Pi-hole independently
- Confirming normal client resolution through the configured DNS pair
- Confirming infrastructure names resolve to their direct management addresses
- Confirming proxied application names resolve to the Caddy address
- Testing name resolution from more than one internal VLAN

Both Pi-hole instances should continue to contain matching local DNS records.
