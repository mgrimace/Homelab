# Homepage

[Homepage](https://gethomepage.dev/en/installation/) is a neat way to collect your various service WebUI URLs in one convenient place. I've added this to my Docker LXC which contains my *arr* services.

## Set up directories

```bash
cd /opt/appdata/
mkdir homepage
cd homepage mkdir config
cd ..
```

## Docker Compose file

```dockerfile
version: "3.3"
services:
  homepage:
    image: ghcr.io/benphelps/homepage:latest
    container_name: homepage
    ports:
      - 3000:3000
    volumes:
      - /opt/appdata/homepage/config:/app/config # Make sure your local config directory exists
      - /var/run/docker.sock:/var/run/docker.sock # (optional) For docker integrations, see alternative methods
      - /mnt/media:/media # this will mount your media share to homepage so you can show stats like free space
    environment:
      PUID: 1000
      PGID: 1000
    networks:
      - homelab
    restart: unless-stopped
    security_opt:
      - apparmor:unconfined

networks:
  homelab:
    driver: bridge
    external: true
```

## Configure Homepage

Your homepage is at: IP:3000

Your config files live in /opt/appdata/homepage/config. Use the [guide](https://gethomepage.dev/en/configs/services/) to set up various services and widgets.

#### Services.yaml 

use the the actual links and API tokens/passwords for the various services you're running.

```yaml
---
# For configuration options and examples, please see:
# https://gethomepage.dev/en/configs/services

- Media:
    - Plex:
        icon: plex.png
        href: IP:32400/web
        description: Plex server
        widget:
            type: plex
            url: IP:32400
            key: [hidden]

    - Overseerr:
        icon: overseerr.png
        href: IP:5055
        description: request shows and movies
        widget:
            type: overseerr
            url: IP:5055
            key: [hidden]

    - Calibre-web:
        icon: calibre.png
        href: IP:8083
        description: book library

- Media Management:
    - Radarr:
        icon: radarr.png
        href: IP:7878
        description: manage movies
        widget:
            type: radarr
            url: IP:7878
            key: [hidden]

    - Radarr 4K:
        icon: radarr-light.png
        href: IP:7879
        description: manage movies (4k)
        widget:
            type: radarr
            url: IP:7879
            key: [hidden]
    - Sonarr:
        icon: sonarr.png
        href: IP:8989
        description: manage TV shows
        widget:
            type: sonarr
            url: IP:8989
            key: [hidden]

    - Sonarr 4k:
        icon: sonarr.png
        href: IP:8990
        description: manage TV show (4k)
        widget:
            type: sonarr
            url: IP:8990
            key: [hidden]

    - Prowlarr:
        icon: prowlarr.png
        href: IP:9696
        description: Manage indexers for the Arr services
        widget:
            type: prowlarr
            url: IP:9696
            key: [hidden]

    - qBittorrent:
        icon: qbittorrent.png
        href: IP:8080
        description: manage downloads
        widget:
            type: qbittorrent
            url: IP:8080
            username: [hidden]
            password: [hidden]

- Home Utilities:

    - Cloudblock (Primary DNS):
        icon: pi-hole.png
        href: https://IP/admin
        description: Primary Cloudblock on Raspberry Pi
        widget:
            type: pihole
            url: https://IP
            key: [hidden]

    - Pi-Hole (Secondary DNS):
        icon: pi-hole.png
        href: IP/admin
        description: Secondary Pi-Hole on Proxmox
        widget:
            type: pihole
            url: IP
            key: [hidden]

    - Homebridge:
        icon: homebridge.png
        href: IP:8581
        description: Homekit smarthome bridge
        widget:
            type: homebridge
            url: IP:8581
            username: [hidden]
            password: [hidden]

- Game Utilities:

    - Minecraft:
        icon: minecraft.png
        description: Ubuntu (Oracle) - IP:19132
        widget:
            type: minecraft
            url: udp://IP:19132
    - ArchiSteamFarm:
        icon: archisteamfarm.png
        href: IP:1242
        description: Steam card farmer, and free game collector

- System:
    - Proxmox:
        icon: proxmox.png
        href: IP:8006
        description: my homelab
        widget:
            type: proxmox
            url: IP:8006
            username: api@pam!homepage
            password: [hidden]

    - Open Media Vault:
        icon: openmediavault.png
        href: IP
        description: NAS storage

    - Portainer(Arr):
        icon: portainer.png
        href: IP:9001
        description: manage arr LXC containers
        widget:
            type: portainer
            url: IP:9001
            env: 2
            key: [hidden]

    - Portainer(Plex):
        icon: portainer.png
        href: IP:9001
        description: manage plex LXC containers
        widget:
            type: portainer
            url: IP:9001
            env: 2
            key: [hidden]
```

#### Settings.yaml 

the overall layout settings

```yaml
---
# For configuration options and examples, please see:
# https://gethomepage.dev/en/configs/settings

title: homelab

headerStyle: underlined

Providers:
  openweathermap: openweathermapapikey
  weatherapi: weatherapiapikey

layout:
  Media:
    style: row
    columns: 3
  Media Management:
    style: row
    columns: 3

fiveColumns: true
```

#### Widgets.yaml

 The header setup

```yaml
---
# For configuration options and examples, please see:
# https://gethomepage.dev/en/configs/widgets

- logo:

- resources:
    cpu: true
    memory: true
    expanded: true
    disk:
      - /media #my share storage, I removed the container storage

- openmeteo:
    label: [my town]
    latitude: [hidden]
    longitude: [hidden]
    units: metric
    cache: 5

- search:
    provider: duckduckgo
    target: _blank
```

## Show the Docker container status

First, ensure Docker Proxy was added via Ibramenu. 

### docker.yaml

```
my-docker:
    host: dockerproxy
    port: 2375
```

Then in services.yaml, add the following lines to each of your services (just before widget, this is just an example, so be sure to use the right names for your project)

```
server: my-docker
container: sonarr # The name of the container you'd like to connect
```

