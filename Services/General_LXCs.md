# General LXC info

This is what I'm using for 'general' LXCs, mounting the shared data, etc., for projects that don't have a pre-built script, or if I don't want to use the pre-built script. I generally start with Ubuntu.

See https://tteck.github.io/Proxmox/ for a variety of pre-built scripts. I use the `Ubuntu` LXC as my starting point. These scripts are entered into your 'node' >_shell (e.g., Proxmox).

## Ubuntu 

Create Ubuntu in an LXC. I use advanced settings and set it to privileged (if I'm mounting media share, otherwise not), and set a static IP addressing with `IP/24`.

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/ubuntu.sh)"
```

If you're installing multiple Ubuntu LXCs (for various projects) and/or you use the automatic rather than the advanced options for the script, you can set a static IP and change the container name later by visiting the container in proxmox, use the DNS, hostname option to change the name of the container

## Mount media share to your LXC

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

## Install Docker and Docker Compose V2

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
  
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Enable root SSH

Many of your LXCs will use root by default, and you can enable ssh (e.g., to use Termius or other SSH software) with the following. In your container:

```bash
sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
sudo systemctl restart ssh

#then you need to add a root password

sudo passwd
```

