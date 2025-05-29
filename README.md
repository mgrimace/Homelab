# Overview

These are my install notes for creating my homelab. The goal is a low-power, always-on media server, which will use the \*arr suite to automate obtaining and organizing media, alongside Overseerr as a front-end to handle requests from the family. Along the way, I have added a dashboard, networking including reverse proxy and authentication, and various useful home services (e.g., home assistant, vaultwarden, calibre, pi-hole, etc.). All docker services are setup using docker compose, and all composes are provided.

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
        ...
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

The majority of my apps are going to be in the Docker LXC. These will be setup using docker compose.yaml files, which I've grouped based on function. In a few cases, I have apps/services running as LXCs directly so they aren't interrupted.

## List of Apps and Services 

These are the current services that I'm using in my homelab. They are mainly deployed using docker compose, and organized into logical 'stacks' on my Docker LXC with a few services as LXCs (e.g., Plex, Vaultwarden). All LXCs are installed by convenient scripts from: https://community-scripts.github.io/ProxmoxVE/. All compose files can be found in the [compose](compose) folder.

---

## 📱 Apps

- **[binwiederhier/ntfy](https://github.com/binwiederhier/ntfy)** – Push notifications via HTTP/S or CLI  
- **[coder/code-server](https://github.com/coder/code-server)** – Run VS Code in the browser  
- **[dullage/flatnotes](https://github.com/dullage/flatnotes)** – Self-hosted note-taking app with flat-file storage  
- **[gethomepage/homepage](https://github.com/gethomepage/homepage)** – Modern, customizable homepage for your server  
- **[glanceapp/glance](https://github.com/glanceapp/glance)** – Dashboard for self-hosted services  
- **[linkstackorg/linkstack](https://github.com/linkstackorg/linkstack)** – Self-hosted link landing page  

---

## 💸 Automation

- **[TheNetsky/Microsoft-Rewards-Script](https://github.com/TheNetsky/Microsoft-Rewards-Script)** – Automate Microsoft Rewards point farming via search emulation 
- **[dgtlmoon/changedetection.io](https://github.com/dgtlmoon/changedetection.io)** – Monitor changes to websites over time  

---

## 📚 Books

- **[janeczku/calibre-web](https://github.com/janeczku/calibre-web)** – Web-based UI for Calibre e-book library with Kobo Sync
- **[kovidgoyal/calibre](https://github.com/kovidgoyal/calibre)** – Powerful e-book management  

---

## 💰 Finances

- **[ellite/Wallos](https://github.com/ellite/Wallos)** – Open-source personal finance manager  
- **[simonwep/ocular](https://github.com/simonwep/ocular)** – Simple budget tracker  

---

## 🎮 Game Claimers

- **[C4illin/ASFclaim](https://github.com/C4illin/ASFclaim)** – ArchiSteamFarm Steam claimer  
- **[JustArchiNET/ArchiSteamFarm](https://github.com/JustArchiNET/ArchiSteamFarm)** – Steam idler and free-game claimer  
- **[Smart123s/ItchClaim](https://github.com/Smart123s/ItchClaim)** – Itch.io free-game claimer  
- **[claabs/epicgames-freegames-node](https://github.com/claabs/epicgames-freegames-node)** – Epic free games notifier and claimer  
- **[mastiffmushroom/TrackerAutoLogin](https://github.com/mastiffmushroom/TrackerAutoLogin)** – Auto-login to trackers  
- **[maxisoft/ASFFreeGames](https://github.com/maxisoft/ASFFreeGames)** – ASF plugin to auto-claim Steam free games  
- **[vogler/free-games-claimer](https://github.com/vogler/free-games-claimer)** – Claims free games from Epic Games, Amazon Prime, GOG  

---

## 📚 Manga

- **[Suwayomi/Suwayomi-Server](https://github.com/Suwayomi/Suwayomi-Server)** – Self-hosted manga reader and downloader  

---

## 📺 Media

- **[davidnewhall/unpackerr](https://github.com/davidnewhall/unpackerr)** – Unpacker for *Arr downloads  
- **[linuxserver-labs/docker-plextraktsync](https://github.com/linuxserver-labs/docker-plextraktsync)** – Plex-Trakt sync tool  
- **[Prowlarr/Prowlarr](https://github.com/Prowlarr/Prowlarr)** – Indexer manager for *Arr apps  
- **[qbittorrent/qBittorrent](https://github.com/qbittorrent/qBittorrent)** – Cross-platform BitTorrent client  
- **[Radarr/Radarr](https://github.com/Radarr/Radarr)** – Movie downloader and manager  
- **[recyclarr/recyclarr](https://github.com/recyclarr/recyclarr)** – Sync quality profiles across Radarr/Sonarr  
- **[Sonarr/Sonarr](https://github.com/Sonarr/Sonarr)** – TV show downloader and manager  
- **[sct/overseerr](https://github.com/sct/overseerr)** – Request management for Radarr/Sonarr with user-friendly UI  
- **[sbondCo/Watcharr](https://github.com/sbondCo/Watcharr)** – Streaming watchlist manager and frontend for *Arr apps  
- **[stashapp/stash](https://github.com/stashapp/stash)** – Adult media manager  
- **[Tautulli/Tautulli](https://github.com/Tautulli/Tautulli)** – Plex stats and monitoring  

---

## 🌐 Networking

- **[goauthentik/authentik](https://github.com/goauthentik/authentik)** – Identity provider for SSO and access control  
- **[jc21/nginx-proxy-manager](https://github.com/jc21/nginx-proxy-manager)** – GUI for managing reverse proxies  

---

## 🔐 Passwords

- **[dani-garcia/vaultwarden](https://github.com/dani-garcia/vaultwarden)** – Lightweight alternative to Bitwarden server  

---

## 🖥️ System Utilities

- **[amir20/dozzle](https://github.com/amir20/dozzle)** – Real-time log viewer for Docker containers  
- **[containrrr/watchtower](https://github.com/containrrr/watchtower)** – Auto-updates Docker containers  
- **[louislam/dockge](https://github.com/louislam/dockge)** – GUI for Docker Compose  
- **[louislam/uptime-kuma](https://github.com/louislam/uptime-kuma)** – Uptime monitoring and status pages  
- **[postgres/pgadmin4](https://github.com/postgres/pgadmin4)** – PostgreSQL admin panel  

---

## 📦 Misc Tools

- **[dmunozv04/iSponsorBlockTV](https://github.com/dmunozv04/iSponsorBlockTV)** – Skip YouTube sponsors on smart TVs  

---
# Support this project

If you found my work here at all helpful, please consider donating whatever you can at the link below. I do my best to keep things up to date and as beginner-friendly as possible, and this is all done in my spare time. Thank you and take good care.

[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=R4QX73RWYB3ZA)
