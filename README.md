# Overview

These are my install notes for creating my homelab. The goal is a low-power, always-on media server, which will use the *arr suite to automate obtaining and organizing media, alongside Overseerr as a front-end to handle requests from the family. Along the way, I have added a dashboard, networking including reverse proxy and authentication, and various useful home services (e.g., home assistant, vaultwarden, calibre, pi-hole, etc.). All docker services are setup using docker compose, and all composes are provided.

## Overview

```bash
Home Server: Micro Lenovo M920Q, I7-8700T, 32gb RAM
│
└── Proxmox (Host System)
    │   ├── Storage
    │   │   └── 512GB NVME (Used by VMs and Containers)
    │   │   
    ├── Virtual Machines
    │   ├── OMV (OpenMediaVault) NAS
    │   │   └── Shares
    │   │       └── 2TB SSD (Shared as media storage to LXC Containers via SMB/CIFS)
    │   │   
    └── LXC Containers
        ├── Pi-Hole (DNS adblocking)
        ├── Home Assistant (Smart devices and automation)
        ├── Scrypted (Doorbell and security cameras)
        ├── Vaultwarden (Family passwords)
        ├── Docker (Main Docker Services)
        └── Plex (Media Server with iGPU passthrough)
```

## Guides

### Initial setup

- [Setup Proxmox](Setup%20Proxmox.md)
- [Setup OMV](Setup%20OMV.md)
- [Setup Docker services](Setup%20Docker%20services.md)
- [Create Plex LXC](Create%20Plex%20LXC.md)
- [Setup Themepark](Setup%20Themepark.md)
- [Setup System Backups](Setup%20System%20Backups.md)

### Network and reverse proxying

- [Setup Networking](Setup%20Networking.md)
- [Create Local Subdomains with SSL](Create%20Local%20Subdomains%20with%20SSL.md)
- [App specific configs for Authentik](App%20specific%20configs%20for%20Authentik.md)
- Optional:  [Setup Crowdsec with NPM.md](Setup%20Crowdsec%20with%20NPM.md) 

## Lay summary

Using Proxmox, the main NVME storage hosts various Virtual Machines (VMs), and Linux Containers (LXCs). Broadly speaking, LXCs share host resources and will be preferred over VMs, which reserve the resources that are allocated to them (whether they need them or not). One VM in particular will function as a network accessible storage (NAS) operating system (OS) to share the second attached SSD media drive to the LXCs and over the network. 

Open Media Vault (OMV) will be used share 2TB SSD media drive via SMB/CIFS to containers and as NAS. OMV functions best as a VM in this setup.

The majority of my apps are going to be in the Docker LXC. These will be setup using docker compose.yaml files, which I've grouped based on function.  

## List of Docker Services

These are the current docker services that I'm using in my homelab. They are deployed using docker compose, and organized into logical 'stacks'. All compose files can be found in the [compose](compose) folder.

```bash
Apps
  - homepage
  - NTFY
  - iSponsorBlockTV
  - Mealie
  - Wallos

Books
  - Calibre
  - Calibre-web
  - Readarr

Bots
  - Epic free-game claimer
  - ArchiSteamFarm steam free-game claimer
  - Twitch prime and GOG free-game claimer
  - Tracker auto-login

Manga
  - Kavita
  - Kapowarr
  - Komf
  - Tranga

Media
  - Overseer
  - Prowlarr
  - qBittorrent
  - Radarr/Radarr4k
  - Sonarr/Sonarr4k
  - Stash
  - Tautuli
  - Unpackerr
  - Watcharr

Networking
  - Authentik
  - NPM

System
  - Dockerproxy
  - Dockge
  - Portainer
  - PGadmin
  - Uptime-Kuma
  - Watchtower
```

# Support this project

If you found my work here at all helpful, please consider donating whatever you can at the link below. I do my best to keep things up to date and as beginner-friendly as possible, and this is all done in my spare time. Thank you and take good care.

[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=R4QX73RWYB3ZA)
