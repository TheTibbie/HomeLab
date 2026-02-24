# DNS Design and Redundancy Approach

## Goal

Provide reliable DNS filtering by advertising two Pi-hole endpoints to LAN clients via router DHCP, with enough redundancy to survive a single instance being unavailable.

---

## Current design

DHCP is handled by the ISP router, which distributes two DNS servers to all LAN clients:

| Priority | Instance |
|---|---|
| DNS #1 | `pihole` (CT 101, `proxmox-02`) |
| DNS #2 | `pihole-b` (CT 102, `proxmox-03`) |

<img width="884" height="466" alt="image" src="https://github.com/user-attachments/assets/8163cb28-7b91-41d6-a92c-d4e93e66de9a" />


---

## Redundancy behavior

Client DNS behavior isn't fully deterministic — most clients prefer the first advertised server and fall back to the second only on failure, though some will prefer whichever responds faster. In practice, this means `pihole` handles the majority of queries under normal conditions. The second instance primarily provides resilience during maintenance or an outage on `proxmox-02`, not active load balancing.

---

## Upstream DNS

Both instances use Google as upstream DNS.

<img width="819" height="324" alt="image" src="https://github.com/user-attachments/assets/5fa9b4f8-2acd-4683-b062-7ab6bbb31b5d" />
<img width="861" height="292" alt="image" src="https://github.com/user-attachments/assets/4c18ed94-b106-4efb-85fb-a5a3e44b1ce3" />

