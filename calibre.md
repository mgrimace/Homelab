# Calibre

**THIS IS A WIP**

create ubuntu lxc but give 10 gb space for docker, etc,  priveleged for mount

`bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/ubuntu.sh)"`

## Mount your media share

Note, these are generally the same steps to do so on any LXC. However, just added one argument `nobrl,` because Calibre needs it to access the library/database on a share. You can leave that out for other LXCs.

install `apt install cifs-utils`

`nano /etc/fstab` to mount your smb:

```bash
#media
//[IP]/data /mnt/media       cifs    nobrl,noperm,iocharset=utf8,rw,credentials=/root/.storage_credentials,uid=1000,gid=1000,file_mode=0660,dir_mode=0770 0       0
```

Add your credentials (i.e., your SMB user and password to `nano /root/.storage_credentials`

```bash
username=[username] #type it in as-is, without the []
password=[your password] #type it in as-is, without the []
```

Reboot

### Install Calibre and Calibre-Web via Docker

install docker apt install docker.io && apt install docker-compose

cd /opt/ mkdir appdata && cd appdata, mkdir calibre && cd calibre, mkdir calibre-web

nano docker-compose.yml

```dockerfile
services:
  calibre:
    image: ghcr.io/linuxserver/calibre
    container_name: calibre
    environment:
      - PUID=1000 #change this to your user's PID
      - PGID=1000 #change this to your user's PGID
    volumes:
      - /mnt/media/media/books:/config #change before the ':'
      - /mnt/media/media/books/uploads:/uploads #change before the ':'
      - /mnt/media/media/books/plugins:/plugins #change before the ':'
    ports:
      - 8080:8080 #change before the ':' if necessary
      - 8081:8081 #change before the ':' if necessary
    restart: unless-stopped
    networks:
      - ebooks

  calibre-web:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: calibre-web
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - /opt/appdata/calibre/calibre-web:/config
      - /opt/appdata/custom-scripts/calibre-web/custom-cont-init.d:/custom-cont-init.d
      - /mnt/media/media/books:/books
    restart: unless-stopped
    depends_on:
      - calibre
    ports:
      - 8083:8083
    networks:
      - ebooks

networks:
  ebooks:
    external: true
```

Create your ebooks docker network: `docker network create ebooks`

#### Create a patch file to prevent a constant login annoyance in Calibre-web

navigate to `cd /opt/appdata` then `mkdir custom-scripts/calibre-web/custom-cont-init.d`, navigate to this new directory, `cd custom-scripts/calibre-web/custom-cont-init.d`

Create a new patch file `nano patch-session-protection.sh` and paste the following:

```bash
#!/bin/bash

echo "**** patching calibre-web - removing session protection ****"

sed -i "/lm.session_protection = 'strong'/d" /app/calibre-web/cps/__init__.py
sed -i "/if not ub.check_user_session(current_user.id, flask_session.get('_id')) and 'opds' not in request.path:/d" /app/calibre-web/cps/admin.py
sed -i "/logout_user()/d" /app/calibre-web/cps/admin.py
```

### Start Calibre and Calibre-Web

Install and start Calibre and Calibre-Web `docker-compose up -d `

Calibre: IP:8080

Calibre-web: IP:8030

*Don't forget to set static IPs in the LXC network options and on your router*

## Moving over an existing library

- In you <u>local</u> Calibre, go to library, export, and export your entire Calibre library/settings/etc. Choose a save location on your shared drive, I created a folder in /media/books/Calibre Backup (the folder needs to be in /books so your new remote Calibre can see it on the mounted drive). 
- Create a 'Calibre Restore' folder in the same location (/media/books/Calibre Restore) to restore the backup to.
- In your <u>remote</u> Calibre, select Library and restore your library to the Calibre Restore folder. You should see all your books, plugins, etc. 
- Manually delete the contents of /media/books/Calibre Library (which should be the one demo book and the new blank database). Leave the folder
- In your <u>remote</u> Calibre, select Library, and move existing library, and select the now-empty Calibre Library folder. This puts everything back to the default location, where Calibre-Web will expect to see it.

## Setup Calibre

WIP: see guide: https://academy.pointtosource.com/containers/ebooks-calibre-readarr/

setup automatic uploads, and users

## Setup Calibre-web 

use admin, admin123 as default login

add the library by finding the folder called 'books', which we defined as `/mnt/media/media/books`, you should see Calibre Library in there. Select it and let it load up the books!

### sync read status between calibre-web and calibre

if you've already marked books as 'read' in your Calibre library, go to admin/ui configuration and select 'link read/unread status to calibre column' 

You can also toggle dark mode in this menu

#### Hide books with certain tags

I don't want all the kids books showing up in my library, go to settings, edit users, denied tags, and add `kids`. Then tag any kids book with the `kids` tag. On the other hand, I *only* want kids accounts to see kids books, so I'd add the `kids` tags to the `allowed tags` column for any of their accounts

You can remove 'random books' by clicking your user icon, as well as various sidebar lists

if you made it this far, **backup your container now**, this was an enormous pain to get here with a lot of trial and error.

---

get things organized for automatic adding, conversion, sharing, shelves in calibre web, users in calibre web etc.

add kindle previewer with wine to convert to KFX

setup calibre server for kobo?

- - - - 