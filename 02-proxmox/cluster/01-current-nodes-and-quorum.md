# Current Membership and Quorum

## Cluster Identity

| Detail | Value |
|---|---|
| Cluster name | `Homelab` |
| Transport | `knet` |
| Secure auth | `on` |
| Quorum provider | `corosync_votequorum` |
| Health | Quorate, 4 nodes, 4 expected votes, quorum 3 |

---

## Node Membership

Corosync `ring0` addresses now use the Management / Servers VLAN. All four nodes are active members with healthy vote state.

| Node | ring0 address |
|---|---|
| `proxmox-01` | Management / Servers VLAN address |
| `proxmox-02` | Management / Servers VLAN address |
| `proxmox-03` | Management / Servers VLAN address |
| `proxmox-04` | Management / Servers VLAN address |

Full IP addresses are intentionally redacted.

---

## Post-Cutover Corosync Update

During the OPNsense VLAN cutover, the Proxmox nodes were moved from the old flat network to the Management / Servers VLAN.

Corosync initially still referenced the old flat-network addresses, which caused the cluster to lose quorum during the initial cutover. The Corosync ring addresses were updated to the new Management / Servers VLAN addresses, the configuration version was incremented, and Corosync was restarted so the cluster could reform.

The cutover and recovery process is documented in:

`../../03-networking/OPNsense/02-cutover-and-implementation/05-issues-encountered-and-resolutions.md`

---

## Host Networking Model

Each node routes primary traffic through `vmbr0`, a Linux bridge bound to the physical interface referred to as `nic0` in this environment. VMs and containers attach to this bridge for network access.

After the OPNsense cutover, Proxmox management access is handled through the Management / Servers VLAN instead of the old flat LAN.

---

## Quorum and Split-Brain Protection

Quorum is the safety mechanism that prevents split-brain scenarios in a multi-node cluster.

When a node or partition loses quorum, cluster-wide operations are blocked by design. The cluster will not take potentially unsafe actions when it has an incomplete view of membership.

With 4 nodes and quorum at 3, the cluster tolerates one node loss before going non-quorate.

---

## Validation References

Non-invasive commands used to verify cluster and networking state:

```bash
pvecm status                 # Cluster health, quorum, and vote state
pvecm nodes                  # Node membership list
grep -nE 'name:|ring0_addr' /etc/pve/corosync.conf
ip route | grep default      # Default route per node
ip -br link                  # Interface and bridge state
```

![pvecm status output](https://github.com/user-attachments/assets/191ef506-8181-4882-916a-9edd85870224)

![Node membership output](https://github.com/user-attachments/assets/cd56b02c-0d7d-4094-a05a-d9ccdbc96d01)
