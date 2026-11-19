---
title: "Homelab IPAM"
description: "Building a small, dependable view of the devices on my network — with automatic discovery, useful hostnames, and history that survives a restart."
layout: portfolio
eyebrow: "Project case study"
repo: "https://github.com/kallum-cooper/homelab-ipam"
---

## The problem

A network scan tells me what is responding right now. It does not give me a lasting picture of the lab: devices sleep, hostnames can be missing, and a failed lookup can turn a recognisable machine into an anonymous address.

Homelab IPAM brings those observations into one lightweight dashboard. The scope is deliberately focused: discover devices on the local network, enrich their names, and retain recently seen entries.

![Homelab IPAM dashboard showing IP addresses, hostnames, MAC addresses, and last-seen times for fictional devices.](/images/case-studies/ipam-dashboard.png)

*Dashboard screenshot using fictional device data.*

## How it works

The application is a Node.js service packaged for Docker. An initial scan runs at startup, followed by an automatic scan every five minutes. The dashboard can also request a scan manually.

1. **Discover:** `arp-scan` collects addresses and MAC addresses from the local network. MAC addresses provide a stable entry identifier when processing scan results.
2. **Enrich:** an optional UniFi lookup matches clients by MAC address. A configured friendly name takes priority, followed by the UniFi hostname, then reverse DNS.
3. **Retain:** results merge into a JSON state file. If a fresh lookup has no hostname, a previously recorded name is preserved. Devices remain available until they have been absent for 48 hours.
4. **Present:** the web interface reads `/api/status` and uses `POST /api/scan` for a manual refresh.

## Decisions that matter

### Keep discovery useful when lookups fail

UniFi is optional. If its lookup fails, enrichment falls back to reverse DNS. If that fails too, the IP and MAC entry still remains useful. Discovery does not depend on every naming service being available.

### Retain observations without claiming a device is online

A retained entry is a record of a device seen recently. Its last-seen time matters: the 48-hour window stops a sleeping device disappearing immediately, while still allowing stale entries to expire.

### Use a file for a small amount of state

The service persists state in `/data/ipam-state.json`, backed by a named Docker volume. Updates write a temporary file and rename it into place. This keeps deployment small and avoids introducing a database service for a compact inventory.

### Bound scanning and avoid overlapping work

The scan manager shares one in-flight scan between concurrent requests. The scanner runs with a timeout. If a scan fails, the API reports the error while retaining the available inventory and the last successful scan timestamp.

## Deployment

The Docker image runs as the non-root `node` user. Compose drops the default capabilities and adds `NET_RAW` for network scanning. Host networking lets discovery operate from the Docker host's network namespace.

```sh
git clone https://github.com/kallum-cooper/homelab-ipam.git
cd homelab-ipam
docker compose up -d
```

The dashboard listens on port `8787`. Optional UniFi configuration is supplied separately, including a CA certificate for a controller using a private certificate.

## Validation and boundaries

The repository includes Node test suites for state retention, hostname enrichment, scan coordination, HTTP behaviour, and the interface. They can be run with:

```sh
node --test src/test/*.test.mjs
```

This is a LAN discovery and inventory dashboard. ARP discovery is limited to the network reachable by the scanner; it does not establish a complete inventory across routed VLANs. The current HTTP handlers do not implement authentication, so the deployment is intended for a trusted network.

## What this project demonstrates

The interesting part is how the service handles incomplete observations: preserving useful names, retaining devices between sightings, keeping optional integrations optional, and reporting failures without discarding the inventory. Those choices make a small tool easier to operate day to day.

[Explore the source on GitHub](https://github.com/kallum-cooper/homelab-ipam) · [See how this portfolio is hosted](/how-this-site-works/)
