# Homepage 

[Homepage](https://gethomepage.dev/en/installation/) is a neat way to collect your various service WebUI URLs in one convenient place.

## Installation

I started from a docker image, and selected `advanced` and made it privileged, with 1024 ram, and yes to `docker compose`

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/docker.sh)"
```

Note that there is a pre-made Homepage LXC script here, I tried it but it doesn't use the docker version by default which I prefer.

## Set up directories

```bash
cd /opt
mkdir appdata
cd appdata
mkdir homepage
cd homepage mkdir config
cd ..
```

## Create the docker compose file

You should be in /opt/appdata/homepage by now

`nano compose.yaml`

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
      - /mnt/media:/media
    restart: unless-stopped #this is added from the initial instructions
    environment:
      PUID: 1000
      PGID: 1000
```

Don't run the compose file yet, first...

## (optional) mount shared drive

This is so you can show how much space is available on your homepage. This assumes you have a media share via CIFS/SMB (e.g., OMV), that you've shared a folder called `data`, and that you have a username and login for this share.

`apt install cifs-utils`

Then, `nano /etc/fstab`

Add the following:

```bash
#media
//[Media share IP]/data /mnt/media       cifs     ro,noperm,iocharset=utf8,rw,credentials=/root/.storage_credentials,uid=1000,gid=1000,file_mode=0660,dir_mode=0770 0       0
```

note, I added `ro,` here to be read-only, I'm not sure what the other modes/options do yet

Add your credentials (i.e., your SMB user and password) to `nano /root/.storage_credentials`

```bash
username=[username] #type it in as-is, without the []
password=[your password] #type it in as-is, without the []
```

Reboot the system

## Configure Homepage

Your homepage is at: IP:3000

Your config files live in /opt/appdata/homepage/config. Use the [guide](https://gethomepage.dev/en/configs/services/) to set up various services and widgets.
I don't yet grasp how to use docker socket to link to dockers on different LXCs.

Here are mine for examples:

Services.yaml = the actual links and API tokens/passwords for the various services you're running.

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

Settings.yaml = the overall layout settings

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

Widgets.yaml = the header setup

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

