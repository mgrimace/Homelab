# Homelab Documentation

A low-power, always-on home server built around the \*arr suite and self-hosted services using Proxmox, Docker, and NAS storage.

>Note: This update is a major restructuring of my Docker setup, and overall simplification of my documentation. I've reorganized the docker directory structure to `/opt/homelab/[apps, books, bots, envs, finances, manga, media, networking, system, themepark, tunnels]`, with each stack containing its own docker-compose file and app-specific configs. All configuration and secrets are in `/opt/homelab/envs`. This makes the entire setup easier to backup and redeploy. I've also replaced NGINX Proxy Manager with [Pangolin](https://docs.fossorial.io/) for reverse proxy, which includes Crowdsec and identity management built-in. Previous documentation has been moved to `/docs/`.

## Table of Contents

- [Quickstart](#quickstart)
- [Overview](#overview)
- [Hardware & Architecture](#hardware--architecture)
- [Proxmox](#proxmox)
- [Backup & Storage](#backup--storage)
- [OpenMediaVault (OMV)](#openmediavault-omv)
- [Docker Setup](#docker-setup)
- [Arr Services](#arr-services)
- [Recyclarr](#recyclarr)
- [Networking & Remote Access](#networking--remote-access)
- [Forgejo Backup](#forgejo-backup)
- [SOPS Encrypted Environment Files](#sops-encrypted-environment-files)
- [Support Me](#support)

---

## Quickstart

Get Docker services up and running quickly on a new system:

```bash
# Clone the repository
cd /opt/
git clone https://github.com/mgrimace/Homelab.git
cd homelab

# Configure or restore environment files
# Copy example envs
cp envs/global.example.env envs/global.env
# ... copy other stacks .env files
# Edit each env
nano envs/global.env      # Edit PID, GID, TZ
nano envs/media.env       # Configure media stack
nano envs/books.env       # Configure books stack
# ... edit other stack .env files

# Deploy stacks
cd /opt/homelab/media
docker compose up -d

cd /opt/homelab/books
docker compose up -d

# ... repeat for other stacks
```

---

## Overview

This homelab uses **Proxmox** as the host system, running both Virtual Machines (VMs) and Linux Containers (LXCs). LXCs are preferred where possible since they share host resources, while VMs reserve allocated resources regardless of usage.

**Architecture summary:**
- **Primary storage**: 512GB NVME (VMs and containers)
- **Media storage**: 2TB SSD (shared via NAS over SMB/CIFS)
- **OS**: OpenMediaVault (OMV) as a VM, serving the 2TB SSD to LXCs and the network
- **Services**: Docker containers deployed via docker-compose in logical stacks on a single LXC
- **Spouse/partner focused**: Important services like Plex and Vaultwarden run as separate LXCs to avoid interruptions

---

## Hardware & Architecture

**Host:** Micro Lenovo M920Q with i7-8700T, 32GB RAM

```
Home Server: Micro Lenovo M920Q, I7-8700T, 32gb RAM
│
└── Proxmox (Host System)
    │
    ├── Storage
    │   └── 512GB NVME (VMs and Containers)
    │
    ├── Virtual Machines
    │   └── OMV (OpenMediaVault) NAS
    │       └── 2TB SSD (Shared as media storage via SMB/CIFS)
    │
    └── LXC Containers
        ├── Pi-Hole (DNS adblocking)
        ├── Home Assistant (Smart devices and automation)
        ├── Scrypted (Doorbell and security cameras)
        ├── Vaultwarden (Family passwords)
        ├── Docker (Main Docker services)
        └── Plex (Media server with iGPU passthrough)
```

---

## Proxmox

### Installation

1. Create a bootable USB with RUFUS or Balena Etcher and write the latest Proxmox ISO
2. Boot from USB and select "Install Proxmox VE"
3. Agree to the EULA
4. Select your NVME drive for installation
5. Choose filesystem: ext4 or xfs (xfs recommended, but both work)
6. Configure networking: fill in default gateway; use your router's IP as DNS (my Pi-Hole is behind the router as a relay)
7. Install and reboot; note the IP address for web access
8. **Set this and all VMs as static IPs in your router**

### Post-Installation

1. Use community scripts from https://community-scripts.github.io/ProxmoxVE/ for:
   - Proxmox post-install script
   - Scaling governor and microcode updates
   - PVE tune-up scripts

2. **Install powertop for idle power saving:**
   ```bash
   apt install powertop
   powertop --auto-tune
   ```
   Add to cron to run on reboot:
   ```bash
   @reboot sleep 60 && /usr/sbin/powertop --auto-tune
   ```

3. **Install Glances for system monitoring:**
   - Follow: https://www.derekseaman.com/2023/04/home-assistant-monitor-proxmox-with-glances.html
   - To show SMB/CIFS shares:
     ```bash
     cd ~/.config/glances
     nano glances.conf
     # Add: [fs]
     #      allow=cifs
     systemctl restart glances.service
     ```

### Optional Configurations

**Disable swap prioritization (prevents LXC swap thrashing):**
```bash
nano /etc/sysctl.conf
# Add: vm.swappiness=0
```

**Enable iGPU passthrough for LXCs:**

Edit `/etc/pve/lxc/<container#>.conf` and add:
```
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=file
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.cgroup2.devices.allow: c 29:0 rwm
```

In the container's docker-compose, add:
```yaml
devices:
  - /dev/dri:/dev/dri
```

---

## Backup & Storage

### Mount USB Backup Drive

**Format and mount:**
```bash
# 1. Identify USB drive
fdisk -l  # note the drive name, e.g., /dev/sdb

# 2. Format (erases drive)
mkfs.ext4 /dev/sdb

# 3. Create mount point
mkdir /mnt/backups

# 4. Mount
mount /dev/sdb /mnt/backups
```

**Add to Proxmox web UI:**
- Go to **Datacenter → Storage → Add Directory**
- Set mount point to `/mnt/backups`
- Set content type to "VZDump backup file"

**Make mount persistent with fstab:**

1. Get UUID:
   ```bash
   lsblk -f  # or: ls -l /dev/disk/by-uuid
   ```

2. Add to `/etc/fstab`:
   ```bash
   UUID=063c75bc-bcc6-4fa5-8417-a7987a26dccb /mnt/backups ext4 defaults,noatime,nofail 0 2
   ```

3. Edit `/etc/pve/storage.cfg` and add `is_mountpoint 1`:
   ```
   dir: backups
       path /mnt/backups
       content backup,images
       prune-backups keep-all=1
       shared 0
       is_mountpoint 1
   ```

---

## OpenMediaVault (OMV)

OpenMediaVault functions as the NAS, sharing the 2TB SSD to containers and the network over SMB/CIFS.

### VM Configuration

Download the OMV ISO in Proxmox and create a new VM with these settings:

| Setting | Value |
|---------|-------|
| **Name** | omv-nas |
| **OS** | OMV ISO (downloaded) |
| **System** | Enable QEMU agent |
| **Disk** | 8GB for OMV OS; enable SSD emulation |
| **CPU** | 2 cores (type: host) |
| **Memory** | 2048MB (disable ballooning) |
| **Start/Shutdown** | Order: 1, Delay: 60s, Start at boot: checked |

Do **not** start automatically; we'll pass the storage drive first.

### Pass Storage Drive to OMV

The 2TB SSD will be passed directly to the OMV VM. Follow this guide: https://dannyda.com/2020/08/26/how-to-passthrough-hdd-ssd-physical-disks-to-vm-on-proxmox-vepve/

1. SSH into Proxmox and run:
   ```bash
   apt install lshw
   lshw -class disks -class storage
   # Note the disk serial
   
   ls -l /dev/disk/by-id/
   # Find the matching disk ID: ata-Samsung_SSD_870_QVO_2TB_SERIAL
   ```

2. Pass disk to OMV (replace 100 with your VM ID):
   ```bash
   qm set 100 -scsi1 /dev/disk/by-id/ata-Samsung_SSD_870_QVO_2TB_SERIAL
   ```

3. In Proxmox UI, ensure iothread is **enabled** for the second drive in hardware settings

4. Start the OMV VM

### OMV Installation & Setup

1. Use Console to install OMV
   - Default hostname: `openmediavault.local`
   - When asked about multiple storage devices, install OMV OS to NVME (8GB QEMU HARDDISK), not the 2TB SSD
   - After reboot, remove the installation media: VM → Hardware → CD/DVD Drive → "Do not use any media"
   - Note the OMV IP address (e.g., `192.168.0.125`)

2. **Web UI setup:**
   - Navigate to the OMV IP in your browser
   - Login: `admin` / `openmediavault`
   - Change password: click gear icon → "Change Password"

3. **System updates:**
   ```bash
   # In Proxmox console:
   omv-upgrade
   ```

4. **Set static IP:**
   - Go to **System → Network → Your Connection**
   - Change IPv4 method from DHCP to Static
   - Enter current IP, netmask (e.g., 255.255.255.0), gateway (e.g., 192.168.0.1)
   - Set DNS to your router IP (e.g., 192.168.0.1 for Pi-Hole relay)

5. **Create filesystem:**
   - **Storage → File Systems → +**
   - Select your 2TB device, keep EXT4, mount it

6. **Create shares:**
   - **Storage → Shared Folders → +**
   - Name: `data`
   - **Services → SMB/CIFS → +**
   - Select the `data` folder
   - **Services → SMB/CIFS → Settings**
   - Enable SMB/CIFS

7. **Create SMB user:**
   - **Users → Users → +**
   - Create user with name, email, password
   - Select the user, then **Shared Folder Permissions**
   - Grant read/write access to `data` folder

### Link OMV to Proxmox

1. In Proxmox: **Datacenter → Storage → Add SMB/CIFS**
2. Fill in:
   - **ID**: `vdisk-nas` (or name of choice)
   - **Server**: OMV IP address
   - **Username/Password**: SMB user credentials
   - **Share**: Select `data`

3. **Backup OMV OS only (not storage):**
   - Go to OMV VM → **Hardware**
   - Select the 2TB storage device → Edit
   - Uncheck **Backup**

---

## Docker Setup

### Create Debian LXC

Use community helper scripts from https://community-scripts.github.io/ProxmoxVE/ to create a Debian LXC, then:

1. In Proxmox UI: Docker LXC → **Options** → Enable CIFS and nesting

>Troubleshooting: I received an error with a recent Docker update:
   ```
   Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: open sysctl net.ipv4.ip_unprivileged_port_start file: reopen fd 8: permission denied: unknown
   ```
   To fix this, in Proxmox shell: `nano /etc/pve/lxc/<CT#>.conf` (e.g., 133.conf), then add: `lxc.apparmor.profile: unconfined`, then restart the container with `pct reboot <CT#>`

2. SSH into the container

### Security Setup

Create a non-root user and set up SSH key authentication:

```bash
adduser [your_username]
# Enter strong password, use defaults for other prompts

usermod -aG sudo [your_username]
```

Log out and SSH back as the new user.

**Set up SSH keys (use DigitalOcean guide):** https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-debian-11

**Disable password authentication:**
```bash
sudo nano /etc/ssh/sshd_config
# Find PasswordAuthentication and set to: no
# Optionally disable root login
sudo systemctl restart sshd
```

**Run docker without sudo (optional):**
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
sudo systemctl restart docker
# Log out and back in
```

### Mount Media Share

1. Install CIFS utilities:
   ```bash
   apt install cifs-utils
   ```

2. Add credentials (OMV SMB user):
   ```bash
   sudo nano /root/.storage_credentials
   # Add:
   # username=[your_username]
   # password=[your_password]
   
   chmod 600 /root/.storage_credentials
   ```

3. Mount in fstab:
   ```bash
   sudo nano /etc/fstab
   # Add:
   # //[OMV_IP]/data /mnt/media cifs nobrl,noperm,rw,credentials=/root/.storage_credentials,uid=1000,gid=1000,file_mode=0660,dir_mode=0770 0 0
   
   # Note: nobrl is required for Calibre on shared drives
   ```

4. Test mount:
   ```bash
   sudo mount -a
   cd /mnt/media
   ```

### File Structure

Use `/opt/homelab` as the base docker path, organizing services into logical stacks:

```
/opt/homelab/
├── envs/
│   ├── global.env
│   ├── books.env
│   ├── media.env
│   └── ...
│
├── apps/
│   ├── docker-compose.yml
│   ├── app1/
│   │   ├── configs/
│   │   └── data/
│   └── app2/
│       ├── configs/
│       └── data/
│
├── media/
│   ├── docker-compose.yml
│   ├── radarr/
│   ├── sonarr/
│   └── ...
│
└── themepark/
```
>Note: I've separated some services into `/tunnels`, specifically for services that I prefer to use with Cloudflare Tunnels rather than Pagolin.  

### Docker Network & Environment

1. Create the docker network:
   ```bash
   sudo docker network create homelab
   ```
>Note: review the compose files for other `networks` to create if you'd like to keep some services on separate docker networks (e.g., `finances`)

2. Create `.env` files in `/opt/homelab/envs/`:
   - `global.env`: PID, GID, TZ (shared across all stacks)
   - `media.env`, `books.env`, etc.: Stack-specific variables

### Deploy Services

```bash
cd /opt/homelab/media
docker compose up -d
```

### Theme Park (Optional)

Add custom themes and dark mode to compatible apps.

**Install:**
```bash
sudo apt-get install jq curl && sudo bash -c "$(curl -fsSL https://theme-park.dev/fetch.sh)" /opt/homelab/themepark/
```

**Update weekly with cron:**
```bash
crontab -e
# Add: 0 1 * * 0 sudo bash -c "$(curl -fsSL https://theme-park.dev/fetch.sh)" /opt/homelab/themepark/
```

**NGINX Proxy Manager (NPM):**
```yaml
volumes:
  - /opt/homelab/themepark/98-themepark-npm:/etc/cont-init.d/99-themepark
```

Edit `98-themepark-nginx-proxy-manager` line 42 to change theme (default: `organizr`)

**qBittorrent:**

First, log into qBittorrent web UI: **Settings → WebUI → Add custom HTTP headers**:
```yaml
content-security-policy: default-src 'self'; style-src 'self' 'unsafe-inline' theme-park.dev raw.githubusercontent.com use.fontawesome.com; img-src 'self' theme-park.dev raw.githubusercontent.com data:; script-src 'self' 'unsafe-inline'; object-src 'none'; form-action 'self'; frame-ancestors 'self'; font-src use.fontawesome.com;
```

Then in compose:
```yaml
environment:
  - TP_HOTIO=true
  - TP_THEME=organizr
volumes:
  - /opt/homelab/themepark/98-themepark-qbittorrent:/etc/cont-init.d/98-themepark
```

---

## Arr Services

Use https://trash-guides.info/ for comprehensive setup of quality profiles, media structures, and best practices.

### qBittorrent

1. Login with default: `admin` / `adminadmin`
2. Follow Trash guide: https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/
3. Create categories: `tv`, `tv4k`, `movies`, `movies4k` for 4K organization

### Prowlarr

1. Create an account
2. Set up torrent indexers
3. Return here after configuring Radarr, Sonarr, etc. (you'll need their API keys)

### Radarr, Radarr4K, Sonarr, Sonarr4K

For each app:

1. Launch web UI and copy API key from **Settings → General**
2. Add to Prowlarr
3. Go back to app and add qBittorrent downloader
   - Set correct category (e.g., `movies` for Radarr, `movies4k` for Radarr4K)
4. Go to **Media Management → Advanced**
   - Enable: Rename movies, Skip free space check, Use hardlinks
   - Import extra files (subtitles)
   - Add root folders: `/mnt/media/movies`, `/mnt/media/movies4k`, etc.
5. In Plex, add both regular and 4K folders to the same library so content appears together

### Overseerr

Set up for family requests:

1. Launch web UI
2. Add all Arr instances
3. Set both regular and 4K versions as "default"
4. Select quality profiles for auto-requests
5. Create additional users for family members

---

## Recyclarr

Recyclarr automatically syncs TRaSH guide quality profiles to your \*arr apps, eliminating manual setup.

Reference: https://recyclarr.dev/wiki/ and https://trash-guides.info/

### Initial Setup

1. Create default config and file structure:
   ```bash
   cd /opt/homelab/media
   docker compose run --rm recyclarr config create
   ```

2. Fix permissions if needed:
   ```bash
   sudo chown -R [username]:[username] ./recyclarr/config/
   sudo chmod -R 700 ./recyclarr/config
   ```

3. Edit `recyclarr/config/recyclarr.yml`:
   - Delete everything after initial commented instructions
   - Go to https://recyclarr.dev/wiki/guide-configs/

### Configure Sonarr

1. Select **Sonarr → WEB-1080p**, copy the code block
2. Enter base URL (e.g., `http://sonarr:8989`) and API key
3. Return and select **WEB-2160p** for Sonarr4K
4. Copy the code, but delete the `sonarr:` line (keep same indentation)
5. Enter Sonarr4K URL and API key

### Configure Radarr

Repeat the same process with HD Blurray + WEB and UHD Bluray + WEB profiles.

### Test & Deploy

```bash
# Test sync
docker compose run --rm recyclarr sync

# If successful, start recyclarr (syncs daily)
docker compose up -d
```

---

## Networking & Remote Access

Set up NGINX Proxy Manager with Cloudflare for secure, encrypted remote access to your services.

**References:**
- https://www.youtube.com/watch?v=h1a4u72o-64
- https://www.youtube.com/watch?v=c6Y6M8CdcQ0
- https://www.youtube.com/watch?v=CPURnYaW3Zk
- https://geekscircuit.com/set-up-authentik-sso-with-nginx-proxy-manager/

### Prerequisites

1. **Buy a domain** (e.g., porkbun.com)
2. **Register free Cloudflare account**

### Cloudflare Setup

1. Add your domain to Cloudflare (free plan)

2. Add DNS records (or use wildcard):
   - Type: CNAME, Name: `*`, Target: Your DDNS (e.g., `x.tplink.com`) or router IP

3. Update nameservers at your domain registrar (see Porkbun guide: https://kb.porkbun.com/article/22-how-to-change-your-nameservers)

4. Wait for nameserver confirmation (up to 24 hours)

5. Enable DNSSEC on Cloudflare and update in domain registrar (https://kb.porkbun.com/article/93-how-to-install-dnssec)

6. Create SSL certificates:
   - **SSL/TLS → Change to "Full/Strict"**
   - Edge certificates (Cloudflare → Internet) are automatic
   - **Origin Certificates → Create**
   - Add: `mydomain.com` and `*.mydomain.com`
   - Set expiration to 15 years
   - Copy certificate as `Cloudflare-certificate.pem`
   - Copy private key as `Cloudflare-key.key`

### Router Configuration

Forward ports in your router:

- **Entry 1**: NGINX HTTP - Internal 80, External 80, TCP & UDP
- **Entry 2**: NGINX HTTPS - Internal 443, External 443, TCP & UDP

### NGINX Proxy Manager (NPM) Setup

1. **SSL → Add SSL → Custom**
   - Upload Cloudflare `.key` (private key)
   - Upload Cloudflare `.pem` (certificate)

2. **Proxy Hosts → Add Proxy Host**
   - **Domain Name**: `service.yourdomain.com`
   - **Forward Hostname/IP**: Container IP or LXC IP
   - **Forward Port**: Service port
   - **Proxy settings**: Cache assets, Block common exploits, WebSocket support
   - **SSL**: Select Cloudflare certificate, Force SSL, HTTP/2 Support

### Local-Only Subdomains

Create internal-only subdomains (e.g., `service.local.mydomain.com`) that stay local and don't expose to the internet.

1. **Pi-Hole → Local DNS Records**
   - Add: `npm.local.mydomain.com` → NPM IP (both IPv4 and IPv6)

2. **NPM → Proxy Host**
   - Domain: `npm.local.mydomain.com`
   - Forward to: npm container/IP and port

3. **Pi-Hole → Local DNS Records → CNAME**
   - Add: `sonarr.local.mydomain.com` → `npm.local.mydomain.com`
   - Add: `radarr.local.mydomain.com` → `npm.local.mydomain.com`
   - Etc.

This way, if NPM's IP changes, you only update one record in Pi-Hole.

**Bulk add via Pi-Hole config:**

Edit `pihole.toml`:
```
cnameRecords = [
  "sonarr.local.mydomain.com,npm.local.mydomain.com",
  "radarr.local.mydomain.com,npm.local.mydomain.com",
  "plex.local.mydomain.com,npm.local.mydomain.com"
]
```

### Add HTTPS to Local Subdomains

1. **NPM → SSL → Dropdown "None" → "Request a new SSL certificate"**
2. Enable: Force SSL, HTTP/2 Support, HSTS Enabled, Use DNS challenge
3. **DNS Provider**: Cloudflare
4. **Credentials file content**: Enter Cloudflare API token (see below)
5. Email and agree to terms

**Generate Cloudflare API Token:**
- Cloudflare → Profile → Create Token
- Template: `Edit zone DNS`
- Permissions: `Zone, Zone, Read`
- Zone resources: Select your domain
- Set TTL and expiration
- Copy token and paste into NPM

### Clear DNS Caches

After making changes, clear caches:

**Pi-Hole:** Settings → Flush DNS Cache

**Device DNS caches:**
- **iPhone**: Toggle airplane mode on/off
- **Mac**: 
  ```bash
  sudo dscacheutil -flushcache
  sudo killall -HUP mDNSResponder
  ```

### Troubleshooting

**Firefox and Pi-Hole (.local sites):**

Create `/etc/dnsmasq.d/20-override-https-rr.conf` with entries:
```bash
dns-rr=https://service.local.mydomain.com,65,000100
```

For Pi-Hole V6, edit `/etc/pihole/pihole.toml`:
```
etc_dnsmasq_d = true
```

Then restart: `pihole restartdns`

**Safari issues:**

Safari randomly uses IPv4 (A) and IPv6 (AAAA) records. Add both to Pi-Hole:
- IPv4: Standard entry
- IPv6: Get from container with `ip a` (eth0 entry), add to Pi-Hole

**Other issues:**
- Use incognito tab (clear browser cache)
- Disable HTTP/2 in NPM if you see 403 errors
- Sync local DNS records if using multiple Pi-Holes
- In Pi-Hole DNS settings, uncheck "never forward non-FQDN" and "never forward reverse lookups"

---

## Forgejo Backup

Back up your entire homelab configuration to a self-hosted Forgejo instance for version control and easy deployment across multiple systems.

### SSH Key Setup for Forgejo

Set up SSH keys first for passwordless git operations:

1. **Generate SSH key:**
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   # Save as ~/.ssh/id_ed25519
   ```

2. **Add public key to Forgejo:**
   - Forgejo → Settings → SSH/GPG Keys
   - Paste contents of `~/.ssh/id_ed25519.pub`

3. **Configure SSH for Forgejo host:**
   ```bash
   # Optional: Edit ~/.ssh/config
   Host forgejo.mydomain.com
       User git
       Port 222
       IdentityFile ~/.ssh/id_ed25519
       IdentitiesOnly yes
   ```

### Initial Repository Setup (First System)

1. **Create repository on Forgejo:**
   - Log into your Forgejo instance (e.g., `forgejo.local.mydomain.com`)
   - Create a new private repository named `homelab`
   - Note the SSH URL: `ssh://forgejo.mydomain.com/user/homelab.git`

2. **Initialize git repository:**
   ```bash
   cd /opt/homelab
   git init
   git config user.name "Your Name"
   git config user.email "your.email@example.com"
   ```

3. **Add remote:**
   ```bash
   git remote add origin ssh://forgejo.mydomain.com/user/homelab.git
   ```

4. **Commit and push:**
   ```bash
   git add .
   git commit -m "Initial homelab configuration"
   git branch -M main
   git push -u origin main
   ```

### Cloning on New Systems

1. **Clone the repository:**
   ```bash
   cd /opt/
   git clone ssh://forgejo.mydomain.com/user/homelab.git
   cd homelab
   ```

2. **Restore environment files:**
   
   Since `.env` files are typically excluded for security, you'll need to recreate or restore them:

   ```bash
   # Option 1: Manually create new .env files
   mkdir -p envs
   nano envs/global.env      # Enter: PUID=1000, PGID=1000, TZ=America/Toronto
   nano envs/media.env       # Stack-specific variables
   nano envs/books.env       # Stack-specific variables
   # ... repeat for other stacks
   ```

3. **Mount NAS and deploy:**
   ```bash
   # Ensure SMB share is mounted (see Docker Setup section)
   
   cd /opt/homelab/media
   docker compose up -d
   
   cd /opt/homelab/books
   docker compose up -d
   # ... deploy other stacks
   ```

### Pulling Updates

When you make changes on one system and want to pull them to another:

```bash
cd /opt/homelab
git pull origin main
# Rebuild any affected services
docker compose up -d
```

---

## SOPS Encrypted Environment Files

Sensitive environment files (`.env`) can be safely versioned and backed up using [Mozilla SOPS](https://github.com/mozilla/sops) with [age](https://github.com/FiloSottile/age) encryption. This keeps your credentials secure while maintaining your normal Docker workflow.

**References:**
- https://technotim.live/posts/secret-encryption-sops/

### Setup

```bash
# Install latest version of SOPS
SOPS_LATEST_VERSION=$(curl -s "https://api.github.com/repos/getsops/sops/releases/latest" | grep -Po '"tag_name": "v\K[0-9.]+')
curl -Lo sops.deb "https://github.com/getsops/sops/releases/download/v${SOPS_LATEST_VERSION}/sops_${SOPS_LATEST_VERSION}_amd64.deb"
sudo apt --fix-broken install ./sops.deb
rm -rf sops.deb
sops -version

# Install latest version of AGE
AGE_LATEST_VERSION=$(curl -s "https://api.github.com/repos/FiloSottile/age/releases/latest" | grep -Po '"tag_name": "v\K[0-9.]+')
curl -Lo age.tar.gz "https://github.com/FiloSottile/age/releases/latest/download/age-v${AGE_LATEST_VERSION}-linux-amd64.tar.gz"
tar xf age.tar.gz
sudo mv age/age /usr/local/bin
sudo mv age/age-keygen /usr/local/bin
age -version
age-keygen -version

# Configure a key
mkdir -p ~/.config/sops/age
age-keygen -o ~/.config/sops/age/key.txt
grep public ~/.config/sops/age/key.txt

#optionally, set:
export SOPS_AGE_KEY_FILE=~/.config/sops/age/key.txt
```

Copy the printed **public key** (starts with `age1...`) — you’ll use it for encryption.

### Encrypting Environment Files

```bash
# For each .env do the following:
sops --encrypt --age age1yourpublickey /opt/homelab/envs/apps.env > /opt/homelab/envs/apps.env.enc
# Or if you set it above, you can encrypt each env without pasting key
sops --encrypt --age $(cat $SOPS_AGE_KEY_FILE |grep -oP "public key: \K(.*)") /opt/homelab/envs/apps.env > /opt/homelab/envs/apps.env.enc
# Or encrypt all envs in the /envs/ folder, skipping the example.envs
for file in /opt/homelab/envs/*.env; do [[ "$file" == *.example.env ]] && continue; sops --encrypt --age $(cat $SOPS_AGE_KEY_FILE | grep -oP "public key: \K(.*)") "$file" > "${file}.enc"; done
```

>Note: Optionally safely delete the plaintext file:
```bash
shred -u /opt/homelab/envs/apps.env
```

### Decrypting for Use

Before deploying a stack:
```bash
sops --decrypt /opt/homelab/envs/apps.env.enc > /opt/homelab/envs/apps.env
docker compose -f /opt/homelab/apps/docker-compose.yml up -d
# Or all envs via
for file in /opt/homelab/envs/*.env.enc; do sops --decrypt "$file" > "${file%.enc}"; done
```

### Editing Encrypted Files

You can directly edit an encrypted file — SOPS decrypts it on the fly and re-encrypts when you save:
```bash
sops --input-type dotenv --output-type dotenv /opt/homelab/envs/apps.env.enc
```

---

## Support

Pull requests to contribute improvements are welcome and encouraged. I'm still learning and doing this in my limited spare time, please be patient with any problems and contribute wherever you can.

🙏 If you've found this project helpful, and/or you'd like to support further development:

<p align="center">
  <a href="https://www.buymeacoffee.com/cammaratam" target="_blank">
    <img src="https://cdn.buymeacoffee.com/buttons/v2/default-orange.png" alt="Buy Me A Coffee" height="60">
  </a>
  <a href="https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=R4QX73RWYB3ZA" target="_blank">
    <img src="https://img.shields.io/badge/Donate-PayPal-green.svg" alt="Donate via PayPal" height="60">
  </a>
</p>
