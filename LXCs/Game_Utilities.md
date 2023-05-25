# Game Utilities

These are game utilities to run on the server mostly relating to scraping and automatically redeeming free games.

1. [ArchiSteamFarm](#ArchiSteamFarm) - automatically farms steam cards, but I use it to find/redeem free steam games via a plugin
2. Epic - not yet setup, this is for redeeming free Epic games.

## ArchiSteamFarm

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

## Epic

Install a docker script image, likely 6 gb (since it will use chrome) and 512 ram, select y to docker compose: `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/docker.sh)"`

### Create the file structure

```bash
cd /opt/

mkdir appdata && cd appdata

mkdir epic && cd epic

mkdir config && cd ..

nano compose.yaml
```

### Create the compose file

```bash
services:
  epicgames-freegames:
    restart: unless-stopped
    container_name: epicgames-freegames
    environment:
      - TZ=Canada/Toronto
    volumes:
      - /opt/appdata/epic/config:/usr/app/config
    image: charlocharlie/epicgames-freegames:latest
    ports:
      - 3000:3000
```

### Create the config file

```bash
cd config

nano config.json
```

See [instructions](https://github.com/claabs/epicgames-freegames-node) for filling out the config, this is the tricky part for me. For now I'm making this local-only. As in, it should work and let me answer captchas when I'm connected to my home network (until I figure out reverse proxies, etc.)

```json
{
  "runOnStartup": true,
  "cronSchedule": "5 16 * * *",
  "logLevel": "info",
  "webPortalConfig": {
    "baseUrl": "http://[localIP]:3000",
  },
  "accounts": [
    {
      "email": "example@gmail.com",
      "password": "abc1234",
      "totp": "EMNCF83ULU3K3PXPJBSWY3DPEHPK3PXPJWY3DPEHPK3YI69R39NE"
    },
  ],
  "notifiers": [
    // You may configure as many of any notifier as needed
    // Here are some examples of each type
    {
      "type": "email",
      "smtpHost": "smtp.gmail.com",
      "smtpPort": 587,
      "emailSenderAddress": "hello@gmail.com",
      "emailSenderName": "Epic Games Captchas",
      "emailRecipientAddress": "hello@gmail.com",
      "secure": false,
      "auth": {
          "user": "hello@gmail.com",
          "pass": "abc123",
      },
    },
    {
      "type": "discord",
      "webhookUrl": "https://discord.com/api/webhooks/123456789123456789/A-abcdefghijklmn-abcdefghijklmnopqrst12345678-abcdefghijklmnop123456",
      // Optional list of users or roles to mention
      "mentionedUsers": ["914360712086843432"],
      "mentionedRoles": ["734548250895319070"],
    },
    {
      "type": "telegram",
      // Optional Custom TELEGRAM server URL
      "apiUrl": "https://api.telegram.org",
      "token": "644739147:AAGMPo-Jz3mKRnHRTnrPEDi7jUF1vqNOD5k",
      "chatId": "-987654321",
    },
    {
      "type": "apprise",
      "apiUrl": "http://192.168.1.2:8000",
      "urls": "mailto://user:pass@gmail.com",
    },
    {
      "type": "pushover",
      "token": "a172fyyl9gw99p2xi16tq8hnib48p2",
      "userKey": "uvgidym7l5ggpwu2r8i1oy6diaapll",
    },
    {
      "type": "gotify",
      "apiUrl": "https://gotify.net",
      "token": "SnL-wAvmfo_QT",
    },
    {
      "type": "homeassistant",
      "instance": "https://homeassistant.example.com",
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
      "notifyservice": "mobile_app_smartphone_name",
    },
    {
      "type": "bark",
      // your bark key
      "key": "xxxxxxxxxxxxxxxxxxxxxx",
      // bark title, optional, default: 'epicgames-freegames'
      "title": "epicgames-freegames",
      // bark group, optional, default: 'epicgames-freegames'
      "group": "epicgames-freegames",
      // bark private service address, optional, default: 'https://api.day.app'
      "apiUrl": "https://api.day.app"
    },
    {
        "type": "ntfy",
        "webhookUrl": "https://ntfy.example.com/mytopic",
        "priority": "urgent",
        "token": "tk_mytoken"
    },
  ],
}
```

### Start the container

use `docker compose up -d` to start the container! 
