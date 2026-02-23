# Current Membership and Quorum

## Cluster identity

| Detail | Value |
|---|---|
| Cluster name | `Homelab` |
| Transport | `knet` |
| Secure auth | `on` |
| Quorum provider | `corosync_votequorum` |
| Health | Quorate — 4 nodes, 4 expected votes, quorum 3 |

---

## Node membership

Corosync `ring0` addresses are on the main LAN (`192.168.x.x/24`). All four nodes are active members with healthy vote state.

| Node | ring0 address |
|---|---|
| `proxmox-01` | `192.168.x.x` |
| `proxmox-02` | `192.168.x.x` |
| `proxmox-03` | `192.168.x.x` |
| `proxmox-04` | `192.168.x.x` |

---

## Host networking model

Each node routes primary traffic through `vmbr0`, a Linux bridge bound to the physical interface (referred to as `nic0` in this environment). VMs and containers attach to this bridge for LAN access.

---

## Quorum and split-brain protection

Quorum is the safety mechanism that prevents split-brain scenarios in a multi-node cluster. When a node or partition loses quorum, cluster-wide operations are blocked by design — the cluster won't take potentially unsafe actions on an incomplete view of membership. With 4 nodes and quorum at 3, the cluster tolerates one node loss before going non-quorate.

---

## Validation references

Non-invasive commands used to verify cluster and networking state:
```bash
pvecm status       # Cluster health, quorum, and vote state
pvecm nodes        # Node membership list
ip route | grep default    # Default route per node
ip -br link        # Interface and bridge state
```

![pvecm status output](https://github.com/user-attachments/assets/191ef506-8181-4882-916a-9edd85870224)

![Node membership output](https://github.com/user-attachments/assets/cd56b02c-0d7d-4094-a05a-d9ccdbc96d01)
