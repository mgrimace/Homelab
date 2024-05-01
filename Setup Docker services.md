# Docker apps and services

## Setup your Debian docker server

I am using the following [TTECK](https://tteck.github.io/Proxmox/) script, by entering this into the proxmox node shell:

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/docker.sh)"
```

Select advanced, then choose the following options:

```bash
Debian 12
Privileged #so we can enable CIFS
Set Root Password: your choice #we'll use this for now, then set up a user/SSH key and disable root login later.
Container ID: default or your choice
Hostname: default or your choice
Disk Size: 64 gb #My current setup uses around 20 gb, set 28 gb+ here, I'm using 64 to be safe, feel free to use less
Allocate CPU cores: 10 #use as many as you feel you can spare, my current setup runs around 40% of 10 CPUs
Allocate RAM: 32768 #this is 32 gb, my current setup uses around 7 gb
Set a bridge: default
Set a Static IPv4 CIDR Address (/24): dhcp or static ip #I change this manually to a static IP after it obtains its address. Either way you'll eventually have to set this as static.
Remaining options: default
Enable Root SSH Access: yes #I do this so I can SSH in and set everything up rather than using the shell in Proxmox. I will disable this later
```

Once the script begins to run, be patient, it will then ask you for some additional settings:

```bash
would you like to add Portainer? N #we'll be adding this via our own compose later
would you like to add the Portainer Agent? N
would you like to add Docker Compse? Y
```

The Docker LXC is now created. I just use my router app to find the IP of the device 'docker'

Go to Proxmox, Docker LXC, Network, and change IP from DHCP to static, and write your new IP/24, and options and enable CIFS.

Let's SSH in.

### Some security considerations

First we need to create a new user, give them sudo privileges, add an SSH key, then disable root/password login via SSH. My best understanding is that since LXCs share processes with the host, we don't want to be using root here in case our LXC is ever compromised.

SSH in, and create your user 

```bash
adduser [your user name] #then give them a strong password, use the defaults for the rest
usermod -aG sudo [your user name]
```

Log out, and SSH back in as your new user

1. Follow this guide to create an SSH key and upload it to your docker server: https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-debian-11
2. I used Termius, and went to settings/keychains, generated a new key, export to host, and sent to docker. I also set up a default identity here using the key for login
3. Next, disable password login: `sudo nano /etc/ssh/sshd_config` , find `PasswordAuthentication` and set to `no`
4. Optionally disable root login here as well.

#### Optional: Run docker commands as your user without sudo/password

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

Replace `$USER` with your username. This allows you to run commands like `docker compose up -d` without your sudo password. You'll likely need to restart Docker via `sudo systemctl restart docker` and log-out/log-back-in to your SSH session. 

### Mount media share to your LXC

On the LXC in proxmox, go to options, features, and ensure CIFS is enabled (and nesting)

On the container >_console, log-in,

install `apt install cifs-utils`

Add your credentials (i.e., your SMB user and password to `sudo nano /root/.storage_credentials`

```bash
username=[username] #type it in as-is, without the []
password=[your password] #type it in as-is, without the []
```

make sure your credentials are only readable by root `chmod 600 /root/.storage_credentials`

`nano /etc/fstab` to mount your smb:

```bash
#media
//[OMV IP]/data /mnt/media cifs nobrl,noperm,rw,credentials=/root/.storage_credentials,uid=1000,gid=1000,file_mode=0660,dir_mode=0770 0 0
```

Note, in the above example, we added `nobrl` which is required for Calibre since our library is mounted on a shared drive.

Reboot and test that it mounts correctly by visiting `cd /mnt/media`

### Optional: add themepark

Themepark allows you to theme various compatible apps for a consistent look, as well as to add 'dark mode' to unuspported apps.

Generally, themepark can be added with an environment arg in most containers; however, a few (namely, qbittorrent and npm) are easist to use when they are installed locally. Install using the following:

```bash
sudo apt-get install jq curl && sudo bash -c "$(curl -fsSL https://theme-park.dev/fetch.sh)" /home/user/docker/appdata/themepark/
```

Noting in this, and in the crontab entry below to change 'user' to your actual username. Now, we can add a cron job to keep the files up to date using `crontab -e` and adding the following line:

```bash
0 1 * * 0 sudo bash -c "$(curl -fsSL https://theme-park.dev/fetch.sh)" /home/user/docker/appdata/themepark/
```

## Setup docker and apps

### Create the file structure

I'm going to be using `/home/user/docker` as my base docker path, with individual app configs at `/home/user/docker/appdata/[app]`and my compose files in ``/home/user/docker/compose`. My present setup is to have multiple apps in logical stacks like media, networking, etc.

```
/home/user/docker/
│
├── appdata/                  # Configuration data for Docker applications
│   ├── app1/                 # Configuration data for Application 1
│   ├── app2/                 # Configuration data for Application 2
│   └── ...                   # Additional application configurations
│
└── compose/                  # Docker Compose files for managing multi-container Docker applications
    ├── stack1/               # Docker Compose files for Stack 1
    ├── stack2/               # Docker Compose files for Stack 2
    └── ...                   # Additional Docker Compose files

```

### Create the docker network

Let's create our app docker network 'homelab' using `sudo docker network create homelab`

### Create .env files

Each stack will also have its own .env file. I've included samples .env for each compose, but you'll need to customize them and rename them `.env`. As well, some apps like Homepage, can use secrets that you can load into compose from a separate .env file, such as `.homepage.env`.

For Authentik, we can use the following commands to create secure passes and keys, taking note to update `user` with your actual username. I'm adding copying the output of the following commands to both the .authentik.env and the .env in that compose file.

```bash
echo "PG_PASS=$(openssl rand -base64 36)" >> /home/user/docker/compose/network/.authentik.env
echo "AUTHENTIK_SECRET_KEY=$(openssl rand -base64 36)" >> /home/user/docker/compose/network/.authentik.env
```

Take a look at the various example.envs and customize the rest of the .env entries as required.

### Optional: copy over previous configs

I am going to copy over all of my previous from /opt/appdata/[app]/configs to /home/user/docker/appdata/[app]/configs. If you're starting fresh, you can skip this step.

## Create and launch you compose.yamls

See my .compose files

Customize the various compose.yamls, add, remove, and change services, ports, etc. to your liking. When you're ready deploy everything by moving to each stack folder (e.g., /compose/media), then use the command `docker compose up -d`

## Setup Arr services

### qBittorrent 

- default user/pass is admin/adminadmin - see trash guide: https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/ for basic setup and work through that.

- I added categories for 4k, such as tv4k, movies4k as well so that I can optionally use Overseerr to request in 4k for a few shows if I want.


### Prowlarr

- created an account, and setup my first torrent indexer

- we'll come back here with our *arr APIs later


### Radarr, Radarr4k, Sonarr, Sonarr4k

- The general idea here is to launch the webUI, grab the API key from settings, general, go back to Prowlarr and setup the apps, then go back and work through each.

- Once you have that sorted, go to the app, and add your downloader qBittorent. Be sure to set the proper category (e.g., Radar = movies, Radar4k = movies4k, which are not the default categories in the settings menu). Then go to media management, turn on advanced, and rename movies, skip free space check, use hard links, import extra files (srt = subtitles), and add the root folder (e.g., media/media/movies for Radarr, media/media/movies4k for Radarr4k).

- Do this for each.

- Then in Plex, make sure you add these folders, e.g., movies monitor both movies and movies4k so all those movies all show up together.


### Quality profiles

- Setup qualitfy profiles to make sure the quality is appropriate for the files are grabbed for your system - https://trash-guides.info/Radarr/
- Setup naming schemes

- Setup custom profiles to make the right filetypes are grabbed (and avoided) for your system (same link as above)

- I'm going to use the following profile for most 720/1080p content https://trash-guides.info/Radarr/radarr-setup-quality-profiles/#trash-quality-profiles . Within this profile, it directs you which custom profiles to setup as well. Also making sure I setup the 'unwanted' custom formats.

- I also merged qualities: https://trash-guides.info/Radarr/Tips/Merge-quality/

- Repeat for each Arr
- Optional: Link the libraries of the regular and 4k versions of Sonarr and Radarr (I'm using option 2 to occassionaly request 4k versions of things) - https://trash-guides.info/Radarr/Tips/Sync-2-radarr-sonarr/ - I've decided not to do this, as I will be using overseer to handle requests and it gives me the option to 'download in 4k' if I setup radarr 4k separately (rather than using radar non-4k to handle and sync things)
- make sure 'unmonitor' is checked in media settings so that when shows are deleted you don't automatically download them again.

### Setup overseer for requests

- Launch the overseerr webUI, add your Arrs, set both the regular and 4k versions as 'default' and select the quality profiles that will be used automatically. I'm setting up a second user for my family to reques

## Troubleshooting Docker

to make docker print out what it is reading from the compose file. So you can preview changes you make to the compose file while editing, without having to start the container each time

```
docker compose config
```

 or 

```
docker compose config radarr
```

