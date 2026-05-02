# DNS Design and Redundancy Approach

## Goal

Provide reliable DNS filtering by advertising two Pi-hole endpoints to client VLANs through OPNsense DHCP, with enough redundancy to survive a single instance being unavailable.

Both Pi-hole instances are currently placed on the Management / Servers VLAN after the OPNsense cutover.

---

## Current Design

DHCP is handled by OPNsense, which distributes two Pi-hole DNS servers to client VLANs:

| Priority | Instance |
|---|---|
| DNS #1 | `pihole` (CT 101, `proxmox-02`) |
| DNS #2 | `pihole-b` (CT 102, `proxmox-03`) |

<img width="600" height="752" alt="image" src="https://github.com/user-attachments/assets/83589356-3da5-4ac6-ab0d-3d72d8045127" />

*OPNsense DHCP scope distributing both Pi-hole instances as DNS servers. Full addresses are redacted.*

---

## Network Placement

Before the OPNsense cutover, Pi-hole DNS was distributed to LAN clients by the ISP router.

That is no longer the active design. OPNsense now provides DHCP, and both Pi-hole instances are placed on the Management / Servers VLAN with other core infrastructure services.

Client VLANs receive Pi-hole as their DNS resolver through OPNsense DHCP. Firewall policy allows client VLANs to query Pi-hole and allows the Pi-hole hosts to reach upstream DNS.

---

## Redundancy Behavior

Client DNS behavior is not fully deterministic. Most clients prefer the first advertised DNS server and fall back to the second only on failure, while some clients may prefer whichever server responds faster.

In practice, this means `pihole` handles the majority of queries under normal conditions. The second instance primarily provides resilience during maintenance or an outage on `proxmox-02`, not active load balancing.

Because the two instances run on separate Proxmox nodes, a single node outage does not remove the entire DNS filtering layer.

---

## Upstream DNS

Both instances use Google as upstream DNS.

<img width="819" height="324" alt="image" src="https://github.com/user-attachments/assets/5fa9b4f8-2acd-4683-b062-7ab6bbb31b5d" />
<img width="861" height="292" alt="image" src="https://github.com/user-attachments/assets/4c18ed94-b106-4efb-85fb-a5a3e44b1ce3" />

---

## Firewall Consideration

The DNS design requires two separate firewall paths:

- Client VLANs must be able to query the Pi-hole instances
- Pi-hole instances must be able to reach upstream DNS resolvers

During the OPNsense cutover, this required an explicit firewall rule allowing the Pi-hole hosts to reach upstream DNS while still keeping client DNS centralized through Pi-hole.
