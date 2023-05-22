# Homepage LXC

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

Here are mine for examples...