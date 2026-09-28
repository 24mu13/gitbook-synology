# Offline Resilience

The services documented in this section follow an **offline-first** philosophy: they run entirely within your home LAN and keep working when internet connectivity is lost — whether due to an ISP outage, travel, or a deliberate air-gap.

The concept is inspired by [Project Nomad](https://github.com/Crosstalk-Solutions/project-nomad/), which bundles a curated stack of self-hosted services designed to be fully autonomous and internet-independent.

## Already offline-resilient

Several services documented elsewhere in this handbook are naturally offline-resilient once set up:

| Service | Section | Notes |
|---|---|---|
| **Nextcloud** | [Container Manager → Nextcloud](../applications/nextcloud.md) | Files, calendar, contacts — fully local |
| **Home Assistant** | [Container Manager → Home Assistant](../applications/home-assistant.md) | Local home automation, no cloud needed |

## Services in this section

| Service                                       | Purpose |
|-----------------------------------------------|---|
| [Protomaps](protomaps.md)                     | Offline vector map tile server |
| Models and [OpenUI](https://openwebui.com/) ? |                                |

## General notes

- All services here are deployed via **Container Manager** on DSM and follow the same conventions as the rest of this handbook.
- Map data and other offline datasets live in dedicated **shared folders** on the NAS (e.g. `/volume1/maps`) so storage quotas and snapshots can be managed independently from container config.
- Container image updates are handled by [Watchtower](../applications/watchtower.md) as usual; **dataset updates** (e.g. map tiles) require separate scheduling — each service page documents the recommended approach.

