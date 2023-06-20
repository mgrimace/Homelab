# General LXC info

General info for managing new LXCs, mounting the shared data, installing Docker, enabling root SSH.

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

