# Issues Encountered and Fixes

## Overview

The internal DNS and reverse proxy project was relatively small, but several issues surfaced during implementation.

The useful part of this project was not only getting Caddy online. It also required validating DNS behavior across VLANs, application proxy trust, monitoring resolution, and recovery behavior.

---

## OPNsense Hostname Blocked by DNS-Rebind Protection

OPNsense initially rejected access through the new internal hostname.

### Cause

The hostname was not recognized as an allowed management name, so DNS-rebind protection blocked the request.

### Resolution

The OPNsense internal hostname was added to the alternate-hostname setting.

Global DNS-rebind protection remained enabled.

### Validation

The OPNsense interface loaded successfully through the new `home.arpa` name while normal rebind protection remained active.

---

## Secondary Pi-hole Did Not Answer Routed Clients Correctly

The secondary Pi-hole initially used a local-only listening posture.

### Cause

DNS clients on other internal VLANs are routed to Pi-hole rather than being on the same local subnet.

### Resolution

Pi-hole interface behavior was changed to permit requests from the internal routed networks.

OPNsense firewall rules remain responsible for restricting who can reach DNS, so this did not create WAN exposure.

### Validation

Each Pi-hole was queried independently from a workstation and returned the expected local DNS records.

---

## Home Assistant Returned HTTP 400 Through Caddy

Home Assistant worked directly but returned HTTP 400 when accessed through the reverse proxy.

### Cause

The Caddy host was not listed as a trusted proxy.

### Resolution

Home Assistant was configured to use forwarded headers and trust the reverse-proxy address.

Configuration validation was completed before restarting Home Assistant.

### Validation

The proxied Home Assistant URL returned successfully after restart.

---

## Blackbox Exporter Could Not Resolve `home.arpa`

Browser access to the reverse proxy worked, but the monitoring probe initially failed.

### Cause

The Blackbox Exporter container did not have the internal Pi-hole servers configured as usable external DNS resolvers.

### Resolution

Both Pi-hole addresses were added to the Blackbox Exporter Docker configuration.

Only the Blackbox container was recreated.

### Validation

The container reported the expected DNS servers and the reverse-proxy probe returned success.

---

## Duplicate Manual Backup Left an LVM Snapshot

A duplicate manual backup of the reverse-proxy LXC was started accidentally and interrupted.

### Cause

The interrupted backup left an orphaned LVM snapshot after the backup process had already stopped.

### Resolution

The active backup process was checked first. After confirming no backup process remained, the orphaned snapshot was removed.

### Validation

The storage layout was checked afterward and only the normal guest disk remained.

The successful PBS backup created before the duplicate attempt was unaffected.

---

## Current Status

The issues above are resolved.

The final implementation provides redundant internal DNS, a dedicated reverse proxy, direct backend fallback access, monitoring of the proxy path, and both PBS and configuration-level recovery options.
