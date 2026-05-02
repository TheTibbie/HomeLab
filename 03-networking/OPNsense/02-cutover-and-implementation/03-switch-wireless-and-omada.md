# Switch, Wireless, and Omada

## Overview

This page documents the managed switching, wireless, and Omada configuration after the OPNsense VLAN cutover. The managed switch and wireless AP provide the physical and wireless VLAN layer for the environment. OPNsense handles routing and firewall policy while Omada manages the switch and AP configuration.

Full IP addresses, MAC addresses, serial numbers, SSID names where needed, and device-specific identifiers are intentionally redacted throughout.

---

## Omada Role

Omada manages the switch and wireless AP from a central interface. Current responsibilities include:

- Managing the VLAN-aware switch configuration
- Managing AP adoption and wireless networks
- Mapping wireless networks to VLAN roles
- Monitoring switch and AP status
- Applying port and wireless configuration consistently

The Omada Controller runs as a self-hosted service on the Management / Servers VLAN alongside other infrastructure services.
<img width="890" height="709" alt="image" src="https://github.com/user-attachments/assets/86dbf10b-7efd-495e-a973-53b28c5bbbcb" />

---

## Managed Switch Role

The managed switch provides the VLAN-aware switching layer between OPNsense, wired clients, wireless infrastructure, and the Proxmox/infrastructure uplink.

Current responsibilities include:

- Carrying tagged VLAN traffic between OPNsense and the switching layer
- Presenting untagged access ports for endpoint devices
- Carrying tagged VLANs to the wireless AP
- Providing an infrastructure uplink for Proxmox nodes and supporting services
- Maintaining a recovery access path for emergency troubleshooting

---

## Switch Port Role Summary

Exact live port mappings are intentionally generalized. The important point is the role each connection provides.

| Port | Role | VLAN Behavior | Purpose |
|---|---|---|---|
| Port 1 | Unused | - | - |
| Port 2 | Unused | - | - |
| Port 3 | Unused | - | - |
| Port 4 | Wireless AP trunk | Tagged VLANs for wireless networks | Carries workstation, IoT, and guest VLANs to the AP |
| Port 5 | Workstation access | Untagged access VLAN | Places a trusted workstation into the workstation VLAN |
| Port 6 | Infrastructure uplink | Untagged Management / Servers VLAN, plus transitional tags where required | Connects Proxmox nodes, PBS, and supporting services |
| Port 7 | OPNsense uplink | Tagged trunk | Carries VLAN traffic between OPNsense and the managed switch |
| Port 8 | Recovery port | Untagged recovery/default network | Provides emergency access if switch or management configuration breaks |

This keeps the routing and firewall role on OPNsense while the switch handles VLAN tagging and endpoint placement.
The switch is adopted in Omada and currently operating with this role-based layout.

<img width="866" height="413" alt="image" src="https://github.com/user-attachments/assets/aa37254e-832a-4b94-ba96-743923b013ca" />

---

## Wireless AP Role

The AP is adopted into Omada and provides VLAN-backed wireless networks. It receives tagged VLANs from the managed switch and places clients into the correct VLAN based on which SSID they join. This allows one physical AP to support multiple security zones without separate wireless hardware.

| Wireless Role | VLAN Role | Purpose |
|---|---|---|
| Workstation wireless | Workstations | Trusted wireless clients |
| IoT wireless | IoT | Smart TVs, wireless test clients, and future IoT devices |
| Guest wireless | Guest | Guest access with restricted internal reachability |
<img width="1831" height="253" alt="image" src="https://github.com/user-attachments/assets/15ef4c5f-f385-44f0-847d-3a5fb24ea995" />


---

## Wireless Validation

Wireless connectivity was validated with live client devices after AP adoption:

- Tested wireless clients join the intended SSID and land in the expected VLAN
- Internet access confirmed from wireless clients
- Workstation and IoT wireless networks confirmed functional
- AP remains adopted and managed in Omada

Guest wireless is present as a separate role and should remain isolated from internal infrastructure.

---

## Switch Recovery Context

During the cutover, the managed switch was factory reset while troubleshooting adoption and addressing behavior. This wiped the switch VLAN configuration and caused a temporary outage.

Recovery required rebuilding a minimal VLAN configuration through the switch standalone interface before Omada management could be restored. The recovery sequence focused on restoring critical paths first:

- OPNsense trunk
- Management / Servers infrastructure uplink
- Trusted workstation access
- Recovery port
- Switch management access
- Omada adoption path

Once those paths were restored, the switch was re-adopted into Omada and port configuration was corrected from the controller.

The full issue and fix are in `05-issues-encountered-and-resolutions.md`.

---

## Temporary Omada Management Path

A temporary management/adoption path remains in place for Omada. The switch and AP were adopted through a default management path during recovery. The switch is reachable through the intended management VLAN, but Omada still identifies it through the temporary/default path.

This is a controlled transitional state, not the intended long-term management model.

The temporary path should not be removed until:

- The switch remains reachable through the intended management VLAN
- The AP remains reachable through the intended management VLAN
- Omada can manage both devices without relying on the temporary path
- A confirmed recovery method exists before making the change
- Current switch and AP configuration is backed up or documented

---

## Operational Notes

- Do not remove the temporary management path until the replacement path is confirmed
- Do not factory reset the switch without a complete recovery plan
- Use TCP-based service checks when firewall policy may block ICMP
- Keep a recovery port available during switch and AP changes
- Validate Omada configuration changes one port or device at a time
- Confirm client VLAN placement after any wireless or port-profile changes

The previous recovery showed that switch management access is a critical dependency. Future changes should preserve at least one known-good path back into the switch before making modifications.

---

## Follow-Up Items

- Move switch and AP management fully onto the intended management VLAN when safe
- Remove the temporary Omada adoption path after validation
- Confirm switch and AP remain manageable after any management-path change
- Back up or document the final Omada configuration
- Keep the recovery port available until the design is fully stabilized
- Review whether additional managed switch capacity is needed before reintroducing a dedicated server VLAN

---

## Status

The managed switch and wireless AP are adopted into Omada and operational. The switch provides the VLAN-aware access layer, the AP provides VLAN-backed wireless networks across workstation, IoT, and guest roles, and Omada manages both devices from a central interface. The temporary management path remains in place as a controlled transitional state until it can be safely removed.
