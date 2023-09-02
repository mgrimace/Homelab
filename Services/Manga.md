# Manga and comics

## Table of contents

1. [Manga Readers](#Manga_readers)
2. [Manga Manager](#Manga_managers)

## Manga readers

After testing both [Komga](https://komga.org) and [Kavita](https://kavitareader.com), **Kavita** IMO is the better choice.

**Rationale:** Paperback (iPad/iOS) can sync read progress back to Kavita server, but can't sync back with Komga. It was broken in v8 of Paperback and expected in v9 according to their discord. If you read on another device, and want progress synced back to server, then go with Kavita at the moment.  

### Kavita compose

```yaml
version: '3.9'
services:
    kavita:
        image: kizaing/kavita:latest    # Change latest to nightly for latest develop builds (can't go back to stable)
        container_name: kavita
        volumes:
            - /mnt/media/media/manga:/manga # Manga is just an example you can have the name you want. See the following
            - /mnt/media/media/comics:/comics          # Use as many as you want
            - /opt/appdata/kavita/config:/kavita/config     # Change './data if you want to have the config files in a different place.
                                        # /kavita/config must not be changed
        environment:
            - TZ=America/Toronto
        ports:
            - "5000:5000" # Change the public port (the first 5000) if you have conflicts with other services
        restart: unless-stopped
        networks:
            - homelab
        security_opt:
            - apparmor:unconfined

networks:
  homelab:
    driver: bridge
    external: true
```



## Manga manager

Comics and Manga Downloaders (in my attempt to find an 'arr' to work with Komga):

- Tranga: https://github.com/C9Glax/tranga - manga focused downloader. Simple and promising but still has a number of bugs including Komga integration, in development

- Kapowarr: https://casvt.github.io/Kapowarr  - the most *arr* looking comic downloader I've seen. Great for comics, not great for Manga (gets metadata from comic-vine). Active Discord.

- Kaizoku: https://github.com/oae/kaizoku - my chosen manga downloader after testing above. Downloads and grabs metadata from anilist and syncs with Komga and Kavita. A bit tricky to setup, but generallly works well with series monitoring, etc. that you'd expect from an *arr*.

**Note:** You can use both Kaizoku (manga) and Kapowarr (comics) as downloaders, as Kavita can manage libraries for both (e.g., my Kavita compose  has separate libraries mounted).

### Kaizoku compose

```yaml
version: '3'

volumes:
  db:
  redis:

services:
  app:
    container_name: kaizoku
    image: ghcr.io/oae/kaizoku:latest
    environment:
      - DATABASE_URL=postgresql://kaizoku:kaizoku@db:5432/kaizoku
      - KAIZOKU_PORT=3000
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - PUID=1000
      - PGID=1000
      - TZ=America/Toronto
    volumes:
      - /mnt/media/media/manga:/data
      - /opt/appdata/kaizoku/config:/config
      - /opt/appdata/kaizoku/logs:/logs
    depends_on:
      db:
        condition: service_healthy
    ports:
      - '3000:3000'
    restart: always
    security_opt:
      - apparmor:unconfined
    networks:
      - homelab
  redis:
    image: redis:7-alpine
    volumes:
      - redis:/data
    security_opt:
      - apparmor:unconfined
    networks:
      - homelab
  db:
    image: postgres:alpine
    restart: unless-stopped
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U kaizoku']
      interval: 5s
      timeout: 5s
      retries: 5
    environment:
      - POSTGRES_USER=kaizoku
      - POSTGRES_DB=kaizoku
      - POSTGRES_PASSWORD=kaizoku
    volumes:
      - db:/var/lib/postgresql/data
    security_opt:
      - apparmor:unconfined
    networks:
      - homelab

networks:
  homelab:
    driver: bridge
    external: true
```

#### Anilist integration into Kaizoku

To set the anilist integration, use `docker exec kaizoku mangal integration anilist` from the command line of the conainter (e.g., by SSH)

### Kapowarr compose

```yaml
version: '3.3'
services:
  kapowarr:
    container_name: kapowarr
    volumes:
      - '/opt/appdata/kapowarr/kapowarr-db:/app/db'
      - '/mnt/media/torrents/manga:/app/temp_downloads' #temp downloads folder
      - '/mnt/media/media/manga:/manga' #manga root folder
      - '/mnt/media/media/comics:/comics' #comics root folder
    ports:
      - '5656:5656'
    image: 'mrcas/kapowarr:latest'
    restart: unless-stopped
    networks:
      - homelab
    security_opt:
      - apparmor:unconfined

networks:
  homelab:
    driver: bridge
    external: true
```

