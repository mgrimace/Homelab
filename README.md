# Homelab
My first ever home server setup and installation notes. I don't know what I'm doing. No one should follow these steps under any circumstances. Please feel free to contribute suggestions and advice.

**NB:** This is a work in progress

## Table of contents

1. [Overview](##Overview])
2. [Hardware installation](Hardware.md/#Hardware)
3. [Proxmox installation and setup](Proxmox.md/#Proxmox)
4. [Open Media Vault installation and setup](OMV.md)
5. [Create Plex and Arr LXC containers](PlexLXC.md)
6. [Setup the Arrs for organizing media](Arrs.md)
7. Alternative: [Create a Ubuntu virtual machine with Plex](UbuntuVM.md)

# Overview

## Goals

### Primary goal

Create a small, low-power, always-on Plex server, which will use the *arr suite to automate obtaining and organizing media, alongside Overseerr as a front-end to handle requests for media from the family. 

### Secondary goals

- Virtual ebook library using Calibre 
- Homebridge to make our smart devices connect into Apple Homekit
- Minecraft server
- A lightweight virtual machine to test scripts and bots.

### A note on documentation

I intend to use this project to learn how to create a home server, learn Proxmox, virtualization, networking, and all the other fun things that come along with it. I am comfortable using SSH and the command line, and I have set up some small Raspberry Pi home projects [previously](https://github.com/mgrimace/PiHole-Wireguard-and-Homebridge-on-Raspberry-Pi-Zero-2) (i.e., PiHole, homebridge using docker containers). This readme will serve as my collection of my notes, resources, and step-by-step instructions. 

## The hardware

Micro Lenovo M920Q, I7-8700T, 16gb RAM, 512GB NVME (main), 2TB 2.5" SSD (media)

## The plan

Using Proxmox, the main NVME will host various Virtual Machines (VMs), and Linux Containers (LXCs). One VM in particular will function as a network accessible storage (NAS) operating system (OS) to share the second attached SSD media drive to the VMs and over the network. 

### NVME - main drive

- Proxmox as the main 'OS' - it manages the virtualization: https://www.proxmox.com/en/
  - VM1: Open-media vault or Unraid (to share the second SSD media drive as a NAS and to the other VMs) 
  - VM2: ubuntu with Plex + the various 'arr' softwares in containers
    - Dockstarter seems to be an easy way to set up the required software: https://dockstarter.com
    - Ibramenu is a potential alternative, which seems to have a focus on Plex in particular and may be better integrated (e.g., setup and integrated properly with hardlinks and trash guides, see below): https://ibramenu.io/
  - VM3 or LXC: Calibre (ebook) server for our various Kindles/Kobos 
    - Likely this will consist of Calibre which is needed to organize the database (and serves as the main GUI) + Calibre Server (?) + Calibre web (?) which is a web front-end. This looks like it will be complicated, especially with Readarr integration.
  - LXC1: homebridge (to make smarthome stuff appear in apple homekit, offloaded from pi zero). See scripts link below.
  - VM4 (?): Something to run python scripts and bots, maybe a light ubuntu or dietpi or something small 
  - VM5 (?): minecraft server  

### SSD - media storage

- This drive will primarily serve as network accessible storage for media for Plex and Calibre 
- The file structure will be organized for hardlinking, following: https://trash-guides.info/Hardlinks/Hardlinks-and-Instant-Moves/

## Setup Guides and resources

### Arrs 

The arr suite automatically downloads and organizes your media. The ones I plan to use are:

- [Radarr](https://github.com/Radarr/Radarr): Manages your movie library
- [Sonarr](https://github.com/Sonarr/Sonarr): Manages your TV library
- [Lidarr](https://github.com/lidarr/Lidarr): Manages your music ilbrary
- [Prowlarr](https://github.com/Prowlarr/Prowlarr): Index manager for *arrs
- [Bazarr](https://www.bazarr.media): Subtitles companion app
- [Readarr](https://github.com/Readarr/Readarr): Book, Magazine, Comics Ebook and Audiobook Manager and Automation

I also plan to use [Overseerr](https://overseerr.dev) as a front-end to handle discovery and requests for new media for the family. Overseer integrates with the arr suite.

### Guides

- https://trash-guides.info - details the setup of the various *arr services, as well as hard linking. Hard linking reduces wear and tear on the media drive. 
- https://youtu.be/p6aSlcbDHqc - youtube videos that detail installing and setting up Plex as a Ubuntu VM. Uses TrueNAS as the NAS, which is overkill for my setup, but the principle should be similar (i.e., create a SMB/CIFS share Media <-> Plex)
- https://tteck.github.io/Proxmox/ - for homebridge (automation/homebridge) and any other LXCs that have convenient setup scripts (e.g., secondary pi-hole LXC is an option)

### Cool things to try

- [Homepage](https://gethomepage.dev/en/installation/) - a nice visual dashboard of all the things that are running, e.g., proxmox stats, downloads, etc. 
- Free game scraping bots (likely running together on a light VM):
  - Epic: https://github.com/claabs/epicgames-freegames-node
  - Steam: https://github.com/JustArchiNET/ArchiSteamFarm + this plugin: https://github.com/maxisoft/ASFFreeGames



