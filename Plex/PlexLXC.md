# Plex

While most of my services live on my Docker LXC (e.g., Arrs, etc.) I used a separate, bare-metal Plex LXC installation so that I don't cut off m family streams when I'm messing around with my other services.

## Script installation

Run this script from the Proxmox node >_shell:

`bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/plex.sh)"`

Source: https://tteck.github.io/Proxmox/

Use the advanced setup option and ensure that the container is priveledged to be able to mount the CIFS/SMB share.

Stop the container if it started automatically, go to options, features, and ensure nesting, and CIFS are enabled. Restart the container

### Mount the CIFS share

On the LXC in proxmox, go to options, features, and ensure CIFS is enabled (and nesting)

On the container >_console, log-in,

install `apt install cifs-utils`

`nano /etc/fstab` to mount your smb:

```bash
#media
//[IP]/data /mnt/media       cifs    noperm,iocharset=utf8,rw,credentials=/root/.storage_credentials,uid=1000,gid=1000,file_mode=0660,dir_mode=0770 0       0
```

Add your credentials (i.e., your SMB user and password to `nano /root/.storage_credentials`

```bash
username=[username] #type it in as-is, without the []
password=[your password] #type it in as-is, without the []
```

## Docker only - pass through the iGPU to the Plex container for hardware transcoding 

Note this step optional and only needed if you manually installed Plex via docker rather than bare-metal using the script. 

### Step 1, on proxmox

To pass through hardware transcoding, go back to your node (proxmox), select >_Shell, `nano etc/pve/lxc/mycontainer#.conf` and add the lines

```bash
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.cgroup2.devices.allow: c 29:0 rwm
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=file 
```

### Step 2, on LXC

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

