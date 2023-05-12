## Plex LXC installation

Ok, I switched conceptually from a VM to an LXC. The main rationale is that the Linux containers are much lighter weight. Also, as far as I understand, the LXCs will only use the resources as-needed vs., a VM which allocates the full resources to the virtual computer (i.e., if I assign 8 gigs of ram, that ram is gone from the pool regardless if the VM actually needs it or not).

In principle, I'm following this [guide](https://youtu.be/SP2ZCDiQrvU) for setting up the LXC, and this [guide](https://www.youtube.com/watch?v=8NZSCTfvgII) for setting up Plex using Ibramenu

## Create Plex LXC container

- Go to local, CT Templates, Templates, download ubuntu 20.04 template
- Create CT
  - Uncheck unprivelged
  - Use template
  - Space? I gave it 32 gb? 
  - I give it 4-6 cores
  - I give it 4096 or more memory
  - Set a static IP in network e.g., 192.168.0.134/24
- Don't start automatically, go to otions, start at boot, and change the start order to run after your vm
- Go to features, and enable nesting, NFS, and cifs/smb

## Pass through the iGPU to the Plex container for hardware transcoding (step 1, on proxmox)

To pass through hardware transcoding, go back to your node (proxmox), select >_Shell, `nano etc/pve/lxc/mycontainer#.conf` and add the lines

```bash
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.cgroup2.devices.allow: c 29:0 rwm
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=file 
```

## Setup the Plex LXC container

Start, go to >_Console and log in using `root` and your password

### Install ibramenu

```bash
wget -qO ./i https://raw.githubusercontent.com/ibracorp/ibramenu/main/ibrainit.sh && chmod +x i && ./i
```

reboot, then relogin and type`ibramenu`

select option 2 to install all the basic tools and select all steps all in one

I used the default `ibranet` when selected a docker network name 

### Link SMB share

go to 5 and select SMB mount

enter your SMB share user account data

## Install plex

Use ibramenu to go to media players, and install plex

note the output for plex's address

### Pass through iGPU in docker compose file (step 2, on LXC)

We'll need to add the following line to our docker compose file

```dockerfile
    devices:
      - /dev/dri:/dev/dri 
```

![img](https://cdn.discordapp.com/attachments/946314371796172820/1105872377482584074/image.png)

- Exit the ibramenu, and go to cd /opt/appdata/plex
- nano compose.yaml
- paste that line under volumes
- save and exit, then run:

```bash
sudo usermod -a -G video root
chmod -R 777 /dev/dri
docker exec plex apt-get -y update
docker exec plex apt-get -y install i965-va-driver vainfo
docker restart plex
```

## Setup Arrs and qbittorrent

- Create a new LXC, priveledged, 8 gb, 2 core, 4096 ram
- go to options turn on nesting, NFS and SMB
- Run Ibramenu as above, but install whatever *arr containers you want, as well as qbittorrent, link the SMB share

## Setup shared media drive file structure

- Mount your share on your own computer, and start setting up the file structure per trash guide
- add whatever foldres (I added tv4k and movies4k and tv-anime)



