# Homelab
My first ever home server setup and installation notes. I don't know what I'm doing. No one should follow these steps under any circumstances. Please feel free to contribute suggestions and advice.

**NB:** This is a work in progress

## Table of contents

1. [Overview](#Overview])
2. [Hardware installation](Hardware.md)
3. [Proxmox installation and setup](Proxmox.md)
4. [Open Media Vault installation and setup](OMV.md)
5. [Setup USB backup for the system](#Backups.md)
6. [Setup Plex](Plex/PlexLXC.md)
7. [Setup Arrs and other services in a Docker LXC](Plex/Arrs.md)
8. Guides for other services:
   1. [Calibre, Calibre-Web](Services/Media_Calibre.md)
   2. [Pi-Hole](Services/Home_Utilities.md)
   3. [Homepage Dashboard](Services/Dashboards_Homepage.md)
   4. [Game Utilities (e.g., scrape free Steam games)](Services/Game_Utilities.md)
   5. [Manga](Services/Manga.md)
9. [Setup a reverse proxy with NGINX, Cloudflare, and Authentik](Network/Reverse.md)
   1. [Authentik configs for specific services](Network/Authentik_Configs.md)
   2. [Add and manage users via Plex](Network/Users.md)

# Overview

## Goals

### Primary goal

Create a small, low-power, always-on Plex server, which will use the *arr suite to automate obtaining and organizing media, alongside Overseerr as a front-end to handle requests for media from the family. Various other services will be added as well (e.g., Calibre, Pi-Hole, Home Assistant, etc.)

## The hardware

Micro Lenovo M920Q, I7-8700T, 16gb RAM, 512GB NVME (main), 2TB 2.5" SSD (media)

## The plan

Using Proxmox, the main NVME will host various Virtual Machines (VMs), and Linux Containers (LXCs). One VM in particular will function as a network accessible storage (NAS) operating system (OS) to share the second attached SSD media drive to the VMs and over the network. 

**[Storage VM]**

- Open Media Vault (OMV) VM to share 2TB SSD media drive via SMB/CIFS to containers and as NAS

**[Home Assistant OS VM]**

- Home Assistant (supervised version for add-ons)
- Docker-wyze-bridge
- Scrypted
- **note**: OS versions for "supervised" since my doorbell (wyze) needs the bridge and scrypted to be added to HomeKit. HA has replaced Homebridge for me for HomeKit integration.

**[Docker LXC]**

- [Homepage](https://github.com/benphelps/homepage) - dashboard for all the running services
- Arrs and Qbittorrent - to manage media
  - [Overseerr](https://overseerr.dev): front-end to handle discovery and requests for new media for the family. Integrates with the arr suite.
  - [Radarr](https://github.com/Radarr/Radarr): Manages your movie library
  - [Sonarr](https://github.com/Sonarr/Sonarr): Manages your TV library
  - [Lidarr](https://github.com/lidarr/Lidarr): Manages your music ilbrary
  - [Prowlarr](https://github.com/Prowlarr/Prowlarr): Index manager for *arrs
  - [Bazarr](https://www.bazarr.media): Subtitles companion app
  - [Readarr](https://github.com/Readarr/Readarr): Book, Magazine, Comics Ebook and Audiobook Manager and Automation

- Calibre-web - manage book library
- Mealie - recipes
- Kavita - Manga and Comic server
  - [Kaizoku](https://github.com/oae/kaizoku) - manage manga 
  - [Kapowarr](https://casvt.github.io/Kapowarr) - manage comics

- NPM for reverse proxy
- Authentik (redis, postgres, worker, server stack) for secure logins and single-sign-on to services
- Dockerproxy
- Portainer
- Watchtower

**[Plex LXC]**

- "Bare-metal" Plex installation
- **note**: separate from Docker stack so I don't interrupt my family viewing when messing around with other services

**[Pi-Hole LXC]**

- "Bare-metal" Pi-Hole installation

**[Bots Docker LXC]**

- [MSRewards Bot](https://github.com/thearyadev/MSRF) - collect daily MS points

- [Epic games Bot](https://github.com/claabs/epicgames-freegames-node) - claims free Epic games

- [ArchisteamFarm](https://github.com/JustArchiNET/ArchiSteamFarm) + this [plugin](https://github.com/maxisoft/ASFFreeGames) - claims free Steam games

- [Tracker auto-login](https://github.com/mastiffmushroom/TrackerAutoLogin) - automatic daily login to useful sites

  

## Setup Guides and resources

Generally speaking, I'll be using a Ibramenu to handle most of setup for the media-related dockers (e.g., Plex, Arrs). Ibramenu: https://github.com/ibracorp/ibramenu

### Guides

- https://trash-guides.info - details the setup of the various *arr services, as well as hard linking. Hard linking reduces wear and tear on the media drive. 
- https://youtu.be/p6aSlcbDHqc - youtube videos that detail installing and setting up Plex as a Ubuntu VM. Uses TrueNAS as the NAS, which is overkill for my setup, but the principle should be similar (i.e., create a SMB/CIFS share Media <-> Plex)
- https://tteck.github.io/Proxmox/ - for homebridge (automation/homebridge) and any other LXCs that have convenient setup scripts (e.g., secondary pi-hole LXC is an option)

