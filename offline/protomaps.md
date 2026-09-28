# Protomaps

[Protomaps](https://protomaps.com/) is an open-source stack for self-hosted vector maps. The core format is **PMTiles** — a single-file archive of map tiles that supports HTTP range requests, making it ideal for local serving with no database required.

The server used here is [`go-pmtiles`](https://github.com/protomaps/go-pmtiles), a lightweight Go binary that serves any `.pmtiles` file over HTTP.

## Setup

- Create folder `/volume1/docker/protomaps` and create or update here the following:
  - An `.env` file with your data (e.g.):
    ```text
    HOST_IP=192.168.1.1
    WEB_PORT=8080
    PMTILES_FILE=my-region.pmtiles
    ```
- Create shared folder `/volume1/maps` as the library for `.pmtiles` data files
  - Give **read** permissions to user `ContainerManager`
  - Download one or more `.pmtiles` extracts into this folder (see [Map data](#map-data) below)
- **Container Manager**
- Create new project **protomaps** on above folder with the content below
- Control Panel → Login Portal → Advanced → **Reverse Proxy**
- Create a new proxy for Protomaps, e.g. `https://maps.your.domain` → `http://192.168.1.1:8080`
- Enjoy **Protomaps**

## Assumptions

- You have installed **Container Manager**
- The folder `/volume1/docker` has been created

## docker-compose.yml

```yaml
services:
  protomaps:
    image: ghcr.io/protomaps/go-pmtiles:latest
    container_name: protomaps
    restart: unless-stopped
    ports:
      - ${WEB_PORT}:8080
    volumes:
      - /volume1/maps:/data:ro
    command: serve /data/${PMTILES_FILE} --cors=*
    healthcheck:
      test: wget -qO- http://localhost:8080/ || exit 1
```

{% hint style="info" %}
The `/volume1/maps` folder is mounted read-only (`ro`) inside the container. The server process itself never writes to your map data.
{% endhint %}

## Map data

`.pmtiles` files are published as daily builds by the Protomaps project.

- **Source**: [maps.protomaps.com/builds](https://maps.protomaps.com/builds/) — regional extracts and planet builds
- **Storage estimates**:

  | Coverage | Approx. size |
  |---|---|
  | Single country (e.g. Italy) | 1 – 4 GB |
  | Continent (e.g. Europe) | 20 – 40 GB |
  | Planet | ~100 GB |

Download the extract you need directly to `/volume1/maps`:

```shell
wget -P /volume1/maps https://build.protomaps.com/<date>/<region>.pmtiles
```

{% hint style="warning" %}
Consider creating `/volume1/maps` as a **Synology shared folder with a storage quota** so map data does not compete with other volumes (e.g. Nextcloud, Immich).
{% endhint %}

## Map updates

Map data updates are **not automatic** — the container only serves whatever file is mounted. Container image updates are handled by [Watchtower](../applications/watchtower.md) as usual.

### Semi-automated updates via Task Scheduler

The `go-pmtiles` CLI supports **range-request based fetching**: it downloads only the changed tile ranges rather than the full file, making incremental updates fast and bandwidth-efficient.

You can schedule this with the **Synology Task Scheduler**:

1. Control Panel → Task Scheduler → Create → **Scheduled Task** → User-defined script
2. Set the schedule (e.g. monthly — Protomaps publishes daily builds but map data rarely needs weekly refreshes for home use)
3. Use a script like the following:

```shell
#!/bin/bash
MAPS_DIR="/volume1/maps"
REGION="italy"  # adjust to your region slug
BUILD_DATE=$(date +%Y-%m-%d)
REMOTE_URL="https://build.protomaps.com/${BUILD_DATE}/${REGION}.pmtiles"
LOCAL_FILE="${MAPS_DIR}/${REGION}.pmtiles"

# Download only changed tile ranges (incremental)
/usr/local/bin/pmtiles fetch "${REMOTE_URL}" "${LOCAL_FILE}"
```

{% hint style="info" %}
**Small region?** For country-level files (< 4 GB) a simpler full re-download with `wget -O` is often more practical than a range-fetch. Use whichever approach fits your bandwidth.
{% endhint %}

The `pmtiles` binary can be downloaded from the [go-pmtiles releases page](https://github.com/protomaps/go-pmtiles/releases) and placed at `/usr/local/bin/pmtiles` on the NAS.

## References

- [Protomaps — Self-hosting](https://docs.protomaps.com/deploy/self-hosting)
- [go-pmtiles GitHub](https://github.com/protomaps/go-pmtiles)
- [Protomaps daily builds](https://maps.protomaps.com/builds/)
- [Project Nomad](https://github.com/Crosstalk-Solutions/project-nomad/)
