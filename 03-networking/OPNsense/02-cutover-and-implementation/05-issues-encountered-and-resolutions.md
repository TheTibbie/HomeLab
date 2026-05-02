# Issues Encountered and Resolutions

## Overview

This page documents the major issues encountered during the OPNsense VLAN cutover and how they were resolved.

The goal is to show the troubleshooting and recovery process behind the migration. The cutover included routing issues, switch recovery, Proxmox quorum loss, Corosync recovery, DNS fixes, Omada adoption work, and service validation.

Full IP addresses, MAC addresses, WAN details, serial numbers, device-specific identifiers, and sensitive service URLs are intentionally redacted throughout. Commands are included where useful, with sensitive values replaced by placeholders.

---

## Issue Summary

| Issue | Result |
|---|---|
| LAN parent interface retained old staging IP | Resolved |
| Server VLAN was not viable with current switching | Resolved as design adjustment |
| Proxmox quorum loss and stale Corosync ring addresses | Resolved |
| Pi-hole upstream DNS was blocked | Resolved |
| Managed switch was factory reset | Resolved |
| Omada required a temporary/default management path | Controlled transitional state |
| AP adoption depended on the temporary management path | Resolved, transitional path retained |
| Monitoring and dashboard targets needed updates | Mostly resolved, ongoing cleanup |
| PBS post-cutover migration and backup path validation | Resolved |

---

## LAN Parent Interface Retained Old Staging IP

During the cutover, OPNsense still had an old staging IP on the LAN parent interface.

This created a routing issue because the WAN interface and LAN parent interface were both tied to the old upstream network range. OPNsense selected the wrong path for default routing, which broke or destabilized outbound access.

### Evidence and Checks

Traffic was tested from a workstation after moving it onto the new workstation VLAN.

```powershell
Test-NetConnection <external-ip> -Port 443
```

Packet captures were also used from OPNsense to confirm traffic behavior on the workstation VLAN and WAN side. The important finding was that outbound traffic was reaching the WAN path, but the route table still showed the wrong default path because the LAN parent retained an old IP.

### Resolution

The old IP configuration was removed from the LAN parent interface.

Action taken in OPNsense:

```text
Interfaces > LAN parent interface
IPv4 Configuration Type: None
Save
Apply Changes
```

The LAN parent interface was left as the VLAN trunk parent only. The VLAN interfaces carry the actual internal gateway roles.

After the change, OPNsense selected the WAN interface as the correct default route and internet access stabilized.

### Validation

```powershell
Test-NetConnection <external-ip> -Port 443
```

Expected result:

```text
TcpTestSucceeded : True
```

**Status:** Resolved.

---

## Server VLAN Deferred

The original staged design placed server workloads in a dedicated server VLAN. During implementation, that design had to be adjusted.

Several Proxmox nodes and supporting services were still connected through an unmanaged switch. Because that switch cannot pass tagged VLAN traffic, the systems behind it had to share a single untagged VLAN.

### Evidence and Checks

The unmanaged switch was connected behind a managed switch access/uplink port. Since unmanaged switches cannot tag or separate VLANs per downstream device, all systems behind that path had to share the same untagged VLAN.

The port role was reviewed as part of the switch recovery:

```text
Infrastructure uplink:
- Native / untagged VLAN: Management / Servers
- Tagged VLANs: transitional tags only where required
```

### Resolution

Rather than redesign the switching layer mid-cutover, management and server workloads were consolidated into the Management / Servers VLAN for this phase.

The dedicated server VLAN remains staged for future use when the physical switching layer can support it cleanly.

### Validation

The priority was confirming that critical infrastructure remained reachable after consolidation:

```powershell
Test-NetConnection <opnsense-management-gateway> -Port 443
Test-NetConnection <proxmox-node> -Port 8006
Test-NetConnection <omada-controller> -Port 8043
```

**Status:** Resolved as a design adjustment.

---

## Proxmox Quorum Loss and Corosync Recovery

During the initial cutover, the Proxmox cluster lost quorum after nodes began moving from the old flat network to the new Management / Servers VLAN.

This mattered because Proxmox depends on quorum for normal cluster filesystem access. With quorum lost, the usual cluster configuration path was not safely writable, which made normal cluster changes harder during the cutover.

The root cause was stale Corosync ring addressing. Corosync still referenced the old flat-network addresses even though the nodes were being moved onto the new management network.

### Evidence and Checks

Quorum was checked from the Proxmox shell:

```bash
pvecm status
```

The output showed the node did not have quorum. The Corosync configuration still referenced the old flat-network addresses.

```bash
grep -nE 'name:|ring0_addr' /etc/pve/corosync.conf
```

The old ring addresses had to be replaced with the new Management / Servers VLAN addresses.

### Resolution

Because quorum was lost, the normal cluster filesystem path could not be relied on for safe editing. The corrected Corosync configuration was staged outside the normal cluster filesystem path and then copied directly to the affected nodes.

Actions taken:

```bash
cp /etc/corosync/corosync.conf /root/corosync.conf.new
```

The staged file was edited so each node used its new management VLAN ring address.

```text
nodelist {
  node {
    name: <proxmox-node-1>
    ring0_addr: <new-management-address>
  }
  node {
    name: <proxmox-node-2>
    ring0_addr: <new-management-address>
  }
  node {
    name: <proxmox-node-3>
    ring0_addr: <new-management-address>
  }
  node {
    name: <proxmox-node-4>
    ring0_addr: <new-management-address>
  }
}

config_version: <incremented-version>
```

The corrected file was written into place and copied to the other nodes.

```bash
cat /root/corosync.conf.new | tee /etc/corosync/corosync.conf
scp /root/corosync.conf.new <node>:/etc/corosync/corosync.conf
```

Corosync was restarted on the affected nodes.

```bash
systemctl restart corosync
```

### Validation

Cluster membership and quorum were checked again:

```bash
pvecm status
cat /etc/pve/.members
```

Proxmox API access was also validated from a working node:

```bash
for n in <node-1> <node-2> <node-3> <node-4>; do
  echo "===== Testing $n ====="
  timeout 15 pvesh get /nodes/$n/status --output-format json-pretty >/dev/null
  echo "EXIT=$?"
done
```

Expected result:

```text
EXIT=0
```

After the corrected configuration was applied, quorum returned and Proxmox management access stabilized.

**Status:** Resolved.

---

## Pi-hole Upstream DNS Blocked

After Pi-hole moved into the Management / Servers VLAN, DNS enforcement needed to be adjusted.

Clients were expected to use Pi-hole for DNS, but the Pi-hole instances also needed to reach upstream DNS resolvers. The firewall policy initially restricted DNS too tightly, which prevented Pi-hole from resolving upstream reliably.

### Evidence and Checks

Pi-hole configuration was checked from the Proxmox host using container execution.

```bash
pct exec <pihole-ct-id> -- cat /etc/pihole/pihole.toml | grep -i "listen\|interface\|bind"
```

Pi-hole interface and listening ports were checked:

```bash
pct exec <pihole-ct-id> -- ip -br addr
pct exec <pihole-ct-id> -- ss -lntup | grep ':53'
```

Local DNS testing was performed inside the Pi-hole container:

```bash
pct exec <pihole-ct-id> -- dig @127.0.0.1 google.com +short
pct exec <pihole-ct-id> -- dig +tcp @127.0.0.1 google.com +short
```

Pi-hole service status and logs were reviewed:

```bash
pct exec <pihole-ct-id> -- systemctl status pihole-FTL --no-pager -l
pct exec <pihole-ct-id> -- journalctl -u pihole-FTL -n 80 --no-pager
pct exec <pihole-ct-id> -- pihole-FTL --test
```

Container firewall state was checked:

```bash
pct exec <pihole-ct-id> -- nft list ruleset
pct exec <pihole-ct-id> -- iptables -S
```

The important finding was that Pi-hole was running and listening, but the surrounding firewall policy still needed to allow Pi-hole itself to reach upstream DNS.

### Resolution

Pi-hole was adjusted to listen appropriately after the VLAN move, and OPNsense was updated to allow Pi-hole upstream DNS.

Pi-hole configuration adjustment:

```text
/etc/pihole/pihole.toml
listeningMode = "ALL"
```

OPNsense firewall action:

```text
Interface: Management / Servers
Action: Pass
Source: Pi-hole hosts / Pi-hole alias
Destination: Any
Ports: DNS ports
Purpose: Allow Pi-hole upstream DNS
```

This preserved the intended DNS model:

- Clients use Pi-hole
- Direct client DNS bypass is restricted
- Pi-hole can reach upstream DNS
- DNS visibility remains centralized

### Validation

DNS was tested again after the rule and configuration changes:

```bash
pct exec <pihole-ct-id> -- dig @127.0.0.1 google.com +short
```

Client-side DNS was validated from a workstation:

```powershell
nslookup google.com
```

**Status:** Resolved.

---

## Managed Switch Factory Reset

The managed switch was factory reset while troubleshooting adoption and addressing behavior. This wiped the VLAN and port configuration and caused a temporary outage.

The reset affected:

- VLAN tagging
- Workstation access
- Infrastructure connectivity
- Switch management
- Omada adoption

### Evidence and Checks

After reset, the workstation could not reach the expected VLAN network and received no valid address. The switch fell back to its default local management state.

A workstation was temporarily configured with a static address on the switch default management range.

```text
Workstation temporary static IP: <temporary-default-range-address>
Subnet mask: <redacted>
Gateway: blank
DNS: blank
```

The switch standalone UI was accessed directly:

```text
https://<switch-default-ip>
```

### Resolution

A minimal VLAN configuration was rebuilt through the switch standalone interface. The goal was not to rebuild everything at once. The priority was restoring critical paths.

Critical paths restored first:

- OPNsense trunk
- Management / Servers infrastructure uplink
- Trusted workstation access
- Recovery port
- Switch management access
- Omada adoption path

Standalone switch actions:

```text
Create Management / Servers VLAN
- Infrastructure uplink: untagged
- OPNsense uplink: tagged

Set PVIDs
- Infrastructure uplink PVID: Management / Servers VLAN
- OPNsense uplink PVID: default/native
```

Workstation VLAN was restored next:

```text
Create Workstations VLAN
- Workstation port: untagged
- OPNsense uplink: tagged

Set PVIDs
- Workstation port PVID: Workstations VLAN
```

A recovery port was left available on the default/recovery network.

### Validation

Connectivity back to OPNsense was tested from a Proxmox node:

```bash
curl -kI --connect-timeout 5 https://<opnsense-management-gateway>
```

A successful response confirmed the path was restored:

```text
HTTP/2 403
server: OPNsense
```

Workstation and infrastructure paths were tested from the workstation:

```powershell
Test-NetConnection <omada-controller> -Port 8043
Test-NetConnection <proxmox-node> -Port 8006
Test-NetConnection <opnsense-management-gateway> -Port 443
```

After the critical paths were restored, the switch was re-adopted into Omada and the port configuration was corrected from the controller.

**Status:** Resolved.

---

## Temporary Omada Management Path

After switch recovery, Omada still identified the switch through a temporary/default management path.

The switch is reachable through the intended management VLAN, but Omada still depends on the temporary/default path for the current management state. Removing that path too early could cause another management lockout.

### Evidence and Checks

The switch was reachable through the intended management path, but Omada still discovered or managed it through the default path.

Connectivity to the switch management interface was tested:

```bash
curl -kI --connect-timeout 5 https://<switch-management-ip>
```

A response confirmed switch reachability:

```text
HTTP/1.0 501 Not Implemented
Server: Web Switch
```

Omada Controller ports were checked from the controller host:

```bash
pct exec <omada-ct-id> -- ss -lntup | grep -E '8043|8088|2981|27001'
```

Switch-to-controller reachability was tested from the switch CLI:

```text
ping <omada-controller-ip>
show controller
```

### Resolution

A temporary VLAN/default management path was created and left in place.

Temporary Proxmox host VLAN interface:

```bash
ip link add link vmbr0 name vmbr0.1 type vlan id 1
ip addr add <temporary-vlan1-address>/24 dev vmbr0.1
ip link set vmbr0.1 up
curl -kI --connect-timeout 5 https://<switch-default-ip>
```

Temporary Omada container VLAN interface:

```bash
pct exec <omada-ct-id> -- bash -lc 'ip link add link eth0 name eth0.1 type vlan id 1 && ip addr add <temporary-vlan1-address>/24 dev eth0.1 && ip link set eth0.1 up && curl -kI --connect-timeout 5 https://<switch-default-ip>'
```

Switch CLI checks:

```text
enable
configure
controller inform-url <omada-controller-address>
controller discover
show controller
```

The temporary path remains in place until the switch and AP can be safely moved fully onto the intended management model.

Before removing it, the following must be confirmed:

- Switch remains reachable through the intended management VLAN
- AP remains reachable through the intended management VLAN
- Omada can manage both devices without the temporary path
- Current switch and AP configuration is backed up or documented
- A recovery method is available before making the change

**Status:** Controlled transitional state.

---

## AP Adoption and Wireless VLAN Mapping

The AP depended on the managed switch and Omada management path. It could not be treated as a standalone wireless task because wireless VLAN mapping depended on the switch trunk and Omada configuration.

### Evidence and Checks

The AP was confirmed in Omada after switch recovery. Its port and adoption path were reviewed before applying wireless VLAN configuration.

Checks performed:

```text
Omada > Devices
Confirm AP is visible
Confirm AP is adopted
Confirm AP uplink port
Confirm AP port carries required VLANs
```

### Resolution

The AP was adopted into Omada after the switch was stable.

Wireless networks were created for:

- Workstation wireless
- IoT wireless
- Guest wireless

Each wireless network was mapped to its intended VLAN role.

```text
Omada > Wireless Networks
Create or edit SSID
Set VLAN role / VLAN ID
Apply changes
```

### Validation

Live wireless clients were used to confirm that wireless access worked.

Validation included:

```text
Client joins intended SSID
Client receives address from expected VLAN
Client reaches internet
Client remains visible in Omada
```

TCP-based testing was preferred where firewall rules might block ICMP.

```powershell
Test-NetConnection <external-ip> -Port 443
```

**Status:** Resolved, with the temporary Omada management path retained.

---

## Monitoring and Dashboard Updates

After services moved from the old flat network to the Management / Servers VLAN, monitoring targets and dashboard links needed updates.

Old targets could make services look unavailable even when they were reachable at their new locations.

### Actions Taken

Uptime Kuma monitor targets were updated to reflect the migrated services.

```text
Uptime Kuma > Monitor
Edit target
Update hostname/IP/URL
Save
Confirm monitor returns healthy
```

Dashy was moved and reviewed as a convenience dashboard.

```text
Dashy configuration
Update service URLs where needed
Confirm dashboard loads
Confirm links point to current service locations
```

### Validation

Uptime Kuma was used as the stronger validation point for service health.

```text
Confirm migrated service monitors are green
Confirm stale targets are removed or updated
Confirm service URLs are redacted before publishing screenshots
```

Dashy is treated as a convenience dashboard, not the source of truth for service health.

**Status:** Mostly resolved, ongoing cleanup.

---

## Proxmox Backup Server Post-Cutover Migration

Proxmox Backup Server was implemented, tested, and documented before the VLAN cutover.

After the cutover, PBS was migrated onto the Management / Servers VLAN and validated on the new addressing scheme. This confirmed that PBS remained online and usable after the network migration.

### Existing State

PBS was already built and documented before this cutover. The post-cutover work was focused on network placement, reachability, and confirming that Proxmox backup paths still pointed to the correct target.

### Actions Taken After Cutover

Post-cutover validation confirmed:

```text
PBS has the correct Management / Servers VLAN placement
Proxmox nodes can reach PBS on the new network
PBS storage remains available in Proxmox
Existing backup jobs still target the expected datastore
Backup paths have been validated after the VLAN cutover
Maintenance jobs remain scheduled
```

Example validation checks:

```bash
ping <pbs-address>
curl -kI --connect-timeout 5 https://<pbs-address>:8007
```

From Proxmox, the PBS storage target was confirmed:

```text
Datacenter > Storage
Confirm PBS storage target is listed
Confirm datastore target is correct
```

Backup validation should continue as part of normal operations:

```text
Run or review backup jobs
Confirm backups complete successfully
Confirm prune / garbage collection / verify jobs remain scheduled
Continue restore testing periodically
```

Network migrations can silently break backup paths even when the original backup design was already working. Validating PBS after the VLAN cutover confirmed that the backup path survived the network migration.

**Status:** Resolved. PBS is online on the Management / Servers VLAN and backup paths have been validated after the cutover.

---

## Operational Lessons

The cutover produced several practical lessons:

- Keep a known-good recovery path before changing switch management
- Do not remove transitional management paths until the replacement path is proven
- Confirm routing and default gateway behavior before assuming firewall rules are the issue
- Validate DHCP, DNS, routing, and firewall behavior separately
- Use TCP-based checks when firewall policy may block ICMP
- Expect monitoring and dashboard tools to need updates after service moves
- Keep staged designs flexible when physical hardware constraints appear during implementation
- Document temporary decisions clearly so they do not become permanent by accident

---

## Follow-Up Items

- Continue periodic Proxmox Backup Server backup and restore validation
- Remove the temporary Omada management path when safe
- Confirm switch and AP management through the intended Management / Servers VLAN
- Review OPNsense aliases for stale server VLAN references
- Review temporary firewall rules and remove anything no longer needed
- Confirm Uptime Kuma and Dashy do not contain stale service targets
- Decide whether the dedicated server VLAN should be reintroduced after switching changes
- Keep the recovery port available until switch management is fully cleaned up

---

## Status

The major cutover issues have been resolved or intentionally tracked as follow-up work.

The current network is operational behind OPNsense with VLAN segmentation, managed switching, wireless VLAN mapping, restored Proxmox management access, migrated core services, updated monitoring, and Proxmox Backup Server validated on the new addressing scheme. Remaining work is focused on cleanup, periodic backup validation, and hardening rather than restoring basic functionality.
