# Other LXCs

[TOC]



## Homebridge

Run this script from the shell of proxmox, it will automatically create a homebridge LXC with 1 CPU, 1 Gb Ram, and 8 Gb storage:

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/homebridge.sh)"
```

remember to set a static IP for the LXC in proxmox

## Ubuntu in an LXC (for scripts, etc)

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/ubuntu.sh)"
```

remember to set a static IP for the LXC in proxmox

If you're installing multiple Ubuntu LXCs (for various projects), use the DNS, hostname option to change the name of the container

### ArchisteamFarm

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
wget #add link to the latest .dll release from https://github.com/maxisoft/ASFFreeGames
sudo reboot
```

## Homepage

Install the LXC image from your node shell using

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/homepage.sh)"
```

Configuration [guide](https://gethomepage.dev/en/configs/services/), and configs (bookmarks.yaml, services.yaml, widgets.yaml) path: `/opt/homepage/config/`

Homepage Interface: IP:3000

## Epic game claimer

add instuctions here for an LXC for this



