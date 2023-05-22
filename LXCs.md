# Other LXCs

## Table of contents

1. [Homebridge](#Homebridge)
2. [Ubuntu](#Ubuntu)
3. [ArchiSteamFarm](#ArchiSteamFarm)
5. [Pihole and Pihole sync](#pihole)

## Homebridge

Run this script from the shell of proxmox, it will automatically create a homebridge LXC with 1 CPU, 1 Gb Ram, and 8 Gb storage:

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/homebridge.sh)"
```

remember to set a static IP for the LXC in proxmox

## Ubuntu 

Create Ubuntu in an LXC (for scripts, etc):

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/ubuntu.sh)"
```

remember to set a static IP for the LXC in proxmox

If you're installing multiple Ubuntu LXCs (for various projects), use the DNS, hostname option to change the name of the container

## ArchisteamFarm

Login in to your new Ubuntu LXC, and create a new user called asf

```bash
useradd -m asf
passwd asf #give yourself a password and write it down somewhere
usermod -aG sudo asf #this gives you sudo priveleges, I'm not sure we want to do that
su asf #changes to asf user, you'll need to enter your new password
chsh -s /usr/bin/bash #gives you bash 
sudo reboot
```

#### Add .net

```bash
sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-6.0
```

#### Install ArchiSteamFarm

```bash
mkdir ArchiSteamFarm
cd ArchiSteamFarm/
wget https://github.com/JustArchiNET/ArchiSteamFarm/releases/download/[find newest release for ASF-linux-x64 and enter it here] #you'll need the find the latest version
unzip ASF-linux-x64.zip
rm ASF-linux-x64.zip
chmod +x ArchiSteamFarm
```

#### Create ASF and bot config files

We need to make config files, and the easiest way to do this is to use the ASF config generator on your computer, then copy their contents to files on your server.

1. go to: https://justarchinet.github.io/ASF-WebConfigGenerator/#/
2. Select ASF first, use: https://steamid.io to find your SteamID64 
3. click advanced options and add an IPCPassword to enable remote access (do not select headless)
4. Hit Download to download ASF.json
5. Then select the BOT tab, and make a bot by creating a name and adding your steam name and password. Then be sure to select "enabled"
6. Hit Download to download [yourbotname].json

#### Upload or copy your config files

1. Use console to access your ubuntu server
2. `cd ArchiSteamFarm/config`
3. `Sudo nano ASF.json` - copy/paste the contents from the ASF.json confing file you generated earlier (i.e., open it using a basic text editor)
4. `control+x`, then `y` to save/create the config file
5. `sudo nano [yourbotname].json` - copy/paste the contents from the [yourbotname].json you generated earlier
6. `control+x`, then `y` to save/create the config file
7. troubleshooting: you may have to give write permission to the config folder using something like `sudo chmod 700 ArchiSteamFarm/config`

#### Accessing the web-ui for your ASF

1. Create an ASF.config file with an IPC password as above.
2. Access the web ui by creating an IPC.config file in ~/ArchiSteamFarm/config
3. `sudo nano IPC.config`
4. Paste the following into the config file

5. ```json
   {
   	"Kestrel": {
   		"Endpoints": {
   			"HTTP": {
   				"Url": "http://*:1242"
   			}
   		}
   	}
   }
   ```

6. `control+x`, `y` to save the new config file

7. Access the web ui by going to `http://ubuntuIP:1242/`

8. You can also create and manage bots using the webUI

#### Enter Steamguard token for your bot

1. You'll likely need to enter your steamguard information here, click your bot in the webUI and it should prompt you to enter the code (alternatively, click the `start` icon to start up the bot)

#### Install the steam game grabber plugin

```bash
su - asf
cd ArchiSteamFarm/plugins
wget https://github.com/maxisoft/ASFFreeGameshttps://github.com/maxisoft/ASFFreeGames #be sure to update with the latest .dll release 
sudo reboot
```

## Pihole

- install the PiHole LXC via the proxmox >_shell terminal `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/pihole.sh)"` - use advance settings and set a root password, static IP, and enable root SSH
- reboot the LXC
- Then run `pihole -a -p` to set a password.
- Pihole admin is found via IP/admin

### Setup gravity sync

Since this is my secondary PiHole (my first lives on separate hardware), I want to install gravity sync to keep them both identical.

#### on both pihole systems

- Run `curl -sSL https://raw.githubusercontent.com/vmstan/gs-install/main/gs-install.sh | bash` and go through the setup
- You'll need your user passwords for both instances
- afterwards, run `gravity-sync push` on the local machine that you want to send to the other, remote machine
- If you need to remove the ssh connection details of the remote (e.g., something went wrong), you can run `ssh-keygen -f "/home/pi/.ssh/known_hosts" -R "[remoteIP]"` This assumes that your local user is called `pi`. On the proxmox lxc use `ssh-keygen -R [remoteIP]` instead.
- You can reset your config file by re-running `gravity-sync config` 

#### Troubleshooting remote not detected error on LXC

on the local machine, go to `cd /etc/gravity-sync`, `nano gravity-sync.rsa.pub` and copy the public address. On the remote machine, go to `~/.ssh`, `nano authorized_keys`, and paste the public key into this file. Then reset the config on the lxc using gravity-sync config. It should now connect. 

## Alt: Running cloudblock on a Debian LXC (not working at present)

create a Ubuntu LXC using: `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/ubuntu.sh)"` use advanced settings, and select 6gb hd, 1024 ram, root password, enable root ssh.

First, let's create a user called *pi*, `adduser pi`, give them a password, and give them sudo privileges `usermod -aG sudo pi`

Switch to the pi user `su pi` 

Install some dependencies

```bash
# Ansible + Git
sudo apt update && sudo apt -y upgrade
sudo apt install git python3-pip
pip3 install --user --upgrade ansible

# Add .local/bin to $PATH
echo PATH="\$PATH:~/.local/bin" >> .bashrc
source ~/.bashrc

# Install the community ansible collection
ansible-galaxy collection install community.general
```

We should be able to follow the ubuntu deployment here https://github.com/chadgeary/cloudblock/tree/master/playbooks#ubuntu-deployment

```bash
ansible-playbook cloudblock_amd64.yml --extra-vars="doh_provider=$doh_provider dns_novpn=$dns_novpn wireguard_peers=$wireguard_peers vpn_traffic=$vpn_traffic docker_network=$docker_network docker_gw=$docker_gw docker_doh=$docker_doh docker_pihole=$docker_pihole docker_wireguard=$docker_wireguard docker_webproxy=$docker_webproxy wireguard_network=$wireguard_network wireguard_hostname=$wireguard_hostname"
```

