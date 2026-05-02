# Purpose and Scope

## Purpose

This homelab is a continuously evolving environment built to demonstrate real infrastructure practice. The goal is not just to get services running, but to own the environment: plan the design, document the reasoning, maintain operational hygiene, validate changes, recover from issues, and improve over time.

Specifically, it covers:

- **Platform ownership:** virtualization design, capacity decisions, guest placement, storage strategy, and service standardization
- **Operational maturity:** backups, retention policy, integrity validation, monitoring, and recovery thinking
- **Networking fundamentals:** DNS filtering, VLAN segmentation, firewall policy, DHCP, NTP, managed switching, and wireless VLAN mapping
- **Systems administration:** Active Directory deployment, DNS validation, Group Policy, file share permissions, backup and recovery, automation, and audit logging
- **Documentation discipline:** decisions recorded with context, validation, issues encountered, and follow-up work

---

## Scope

This repo documents:

- High-level architecture and the reasoning behind major decisions
- Proxmox cluster design, node roles, guest placement, storage, and backups
- OPNsense routing, firewall policy, VLAN segmentation, DHCP, and post-cutover validation
- Pi-hole DNS filtering and redundancy
- Uptime Kuma monitoring and Dashy service dashboard placement
- Active Directory lab implementation and validation
- Service inventory and placement, including what runs where and why
- Operational posture, including backup approach, monitoring coverage, recovery thinking, and hardening work
- Issues encountered during implementation and how they were resolved
- Roadmap items, cleanup work, and planned improvements

---

## Documentation Model

The repo separates planning, implementation, and current-state documentation.

Some folders document the current operational environment. Other folders, such as the OPNsense preparation section, are retained as historical planning records. This is intentional. Real infrastructure work includes initial assumptions, staged designs, implementation changes, troubleshooting, and final validation.

Where the final implementation diverged from the original plan, the cutover and implementation documentation explains what changed and why.

---

## What This Repo Is Not

This is not a production hardening guide, and it is not a step-by-step tutorial intended for copy/paste deployment.

The documentation is written for an audience that can understand infrastructure decisions, operational tradeoffs, and validation evidence. The goal is to show how the environment is designed, operated, and improved, not to hand-hold through every command.
