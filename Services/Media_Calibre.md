

# Calibre

Calibre can be integrated into your *arr* LXC which contains all your docker, or if you want to, in a separate LXC.

## Add Calibre-web to your Docker LXC

I used the following compose file to add this to my existing Arr Docker LXC, which also adds Calibre-Web to my 'homelab' Docker Network. **Note:** I already had a working Calibre Library on my mounted media share. See the follow-up instructions for how to create a Calibre, Calibre-web stack with it's own 'ebooks' docker network if you don't have a library setup already.

### Compose file

```docker
services:
  calibre-web:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: calibre-web
    environment:
      - PUID=1000
      - PGID=1000
      - DOCKER_MODS=linuxserver/mods:universal-calibre
    volumes:
      - /opt/appdata/calibre/calibre-web:/config
      - /opt/appdata/custom-scripts/calibre-web/custom-cont-init.d:/custom-cont-init.d
      - /mnt/media/media/books:/books
    restart: unless-stopped
    ports:
      - 8083:8083
    networks:
      - homelab
    security_opt:
      - apparmor:unconfined

networks:
  homelab:
    driver: bridge
    external: true
```

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

## Optional: Create a Calibre + Calibre-Web Docker Stack

Compose.yaml

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

### Start Calibre and Calibre-Web

Install and start Calibre and Calibre-Web `docker compose up -d `

Calibre: IP:8080

Calibre-web: IP:8030

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

if you've already marked books as 'read' in your Calibre library, go to admin/ui configuration and select 'link read/unread status to calibre column'. Note you don't want to do this if you have multiple users (so read status remains per-user)

You can also toggle dark mode in this menu

#### Hide books with certain tags

I don't want all the kids books showing up in my library, go to settings, edit users, denied tags, and add `kids`. Then tag any kids book with the `kids` tag. On the other hand, I *only* want kids accounts to see kids books, so I'd add the `kids` tags to the `allowed tags` column for any of their accounts

You can remove 'random books' by clicking your user icon, as well as various sidebar lists

if you made it this far, **backup your container now**, this was an enormous pain to get here with a lot of trial and error.

## Calibre-web book conversion and kepubify

Add `      - DOCKER_MODS=linuxserver/mods:universal-calibre` under environments in your docker compose file for calibre-web services. 

```bash
docker compose pull
docker compose up -d
docker image prune
```

Wait a minute for Calibre-web to restart, it takes longer than you think.

In calibre-web, go to settings, basic config settings, external binaries and add `usr/bin/ebook-convert` to convert and `/usr/bin/kepubify` to the kepubify pathway.

While in settings, turn on uploads, and be sure to also go to your user and enable uploads for your user.

## Add Readarr

A note on RAM: After some trial and error, Reader requires bumping up the ram during the initial 'identifying book' phase if you have a larger library (e.g., I have about 350 books). I bumped it up to 4096 and I temporarily added a few more cores (dropping back down to 2 gb ram and 2 cores after the library loads). Then monitor and adjust ram/cores accordingly.

Be sure to add a new path in your media share if you haven't already `data/torrents/books`, I'm already using `data/media/books` as my Calibre library. In qBittorrent, be sure to add a new tag `books` with the path `books`.

In your calibre compose.yaml, add the following after the calibre-web entry, before the final networks entry

```dockerfile
  readarr:
    image: lscr.io/linuxserver/readarr:nightly
    container_name: readarr
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - /opt/appdata/calibre/readarr:/config 
      - /mnt/media/media/books:/books
      - /mnt/media/torrents/books:/downloads
    ports:
      - 8787:8787
    restart: unless-stopped
    networks:
      - ebooks
```

Get things up and running, fingers crossed it works

```bash
docker compose pull
docker compose up -d
docker image prune
```

Go to the Readarr webUI, using IP:8787

### Connect Readarr to Calibre

- You will see a relatively empty screen with a `+` button which says 'Add root folder'. Click it

- Your path *should* look like `/books/Calibre Library`
- Be sure to tick the `Use Calibre` checkbox
- The `host` is your machine's IP address, OR `calibre`
- If the `host` is `calibre`, then the `port` is `8081`, BUT if you use the machine IP address, then the port is whatever you changed it to in your calibre docker-compose service block
- Remember the username and password we set up in Calibre under 'Sharing over the net`? Use those credentials here
- The `Calibre Library` can either be blank, or `Calibre_Library`

- Next to `Convert to format` I have chosen `azw3`. This means that when readarr sends the downloaded ebook file to calibre, regardless of what it is, calibre will automatically convert it to my chosen preference
- Hit `Save`

Back on the 'Media Management' screen, you should now have a fully set up root folder called Calibre (or whatever you named it). If you had books in it already, you should see bottom left that readarr has already started indexing them, and they'll soon be available to view in your 'Library'.

I also needed to add a remote path mappings because my Calibre server uses a different mounted/mapped pathway to the Library.

- host: `calibre`
- Remote path: `/config/Calibre Library/`
- Lcoal path: `/books/Calibre Library/`



