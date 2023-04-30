# Homelab
My first ever home server setup and installation notes. I don't know what I'm doing. No one should follow these steps under any circumstances. Please feel free to contribute suggestions and advice!

## Goals

My primary goal is to create a small, low-power, always on Plex server, which will use the *arr suite to automate obtaining and organizing media, alongside Overseerr as a front-end to handle requests for media from the family. My secondary goals are to create a virtual ebook library using Calibre server for our various Kindles and Kobos; run homebridge to make our smart devices connect into Apple Homekit; possibly a minecraft server; and a lightweight virtual machine to test scripts and bots.

### A note on documentation

I intend to use this project to learn how to create a home server, learn Proxmox, virtualization, networking, and all the other fun things that come along with it. I am comfortable using SSH and the command line, and I have set up some small Raspberry Pi home projects previously (i.e., PiHole, homebridge using docker containers). This readme will serve as my collection of my notes, resources, and step-by-step instructions. 

## The hardware

Micro Lenovo M920Q, I7-8700T, 16gb RAM, 512GB NVME (main), 2TB 2.5" SSD (media)

## The plan

Using Proxmox, the main NVME will host various Virtual Machines (VMs), and Linux Containers (LXCs). One VM in particular will function as a network accessible storage (NAS) operating system (OS) to share the second attached SSD media drive to the VMs and over the network. 

NVME  (main drive)

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

SSD - media storage for NAS, Plex, and Calibre 

### Setup Guides and resources

- https://trash-guides.info - details the setup of the various *arr services, as well as hard linking. Hard linking reduces wear and tear on the media drive. 
- https://youtu.be/p6aSlcbDHqc - youtube videos that detail installing and setting up Plex as a Ubuntu VM. Uses TrueNAS as the NAS, which is overkill for my setup, but the principle should be similar (i.e., create a SMB/CIFS share Media <-> Plex)
- https://tteck.github.io/Proxmox/ - for homebridge (automation/homebridge) and any other LXCs that have convenient setup scripts (e.g., secondary pi-hole LXC is an option)





