# Setup Recyclarr

[Recyclarr](https://recyclarr.dev/wiki/) is a handy service to keep [TRaSH guide](https://trash-guides.info/) quality profiles in sync with the guide.

The default instructions are difficult to comprehend for a novice, so here's how I set it up. My goals were to add the default web-1080p profile for radarr and sonarr, and the default web-2160p profile for radarr4k and sonarr4k.

Pre-setup prep: I went into my sonarr and radarr containers and deleted all existing custom formats that I had created a while back

## Create the service

Add recyclar to your media stack compose. If you're using one similar to mine, add:

```yaml
  recyclarr:
    #description: sync, TRaSH guide quality profiles; https://recyclarr.dev/
    <<: *homelab-default
    container_name: recyclarr
    image: ghcr.io/recyclarr/recyclarr
    volumes:
      - ${DOCKERDIR}/appdata/recyclarr/config:/config
```

Then run the following command to create the default config and file structure:

```bash
docker compose run --rm recyclarr config create
```

I received an error here, and had to make sure my non-root user had read-write privileges by: 

```bash
sudo chown -R username:username /home/username/docker/appdata/recyclarr/config/
sudo chmod -R 700 /home/username/docker/appdata/recyclarr/config
```

where, of course, I replaced `username` with my username. Then, repeating the above config create command worked

## Setup the config

Next, I navigated to the config recyclarr.yml. 

- First, delete everything after the initial commented-out instructions. Then, navigate to https://recyclarr.dev/wiki/guide-configs/

### Setup Sonarr

- Select sonarr first, and chose the WEB-1080p option. Copy and paste the block of code into the config.
- Enter your base url, which can be the container name and port, e.g.,`http://sonarr:8989`. Also enter your api_key from the app.
- Note, in the first 'include' section, it speaks to optionally choosing the `alternative` profile. FYI the alternative profiles includes 720p if you prefer.
- Next, go back and select the WEB-2160p option, and copy/paste that into your config below.
- Note: since we're already working in sonarr, even though it's a different instance, delete the `sonarr:`  line from this code. You will start now with `web-2160p-v4` after the instruction block, and keep the indentation the same.
- Add `http://sonarr4k:8989` <-- use the original service port, not the remapped one, and add the API. 

### Setup Radar

- repeat the same steps as above, copy the HD Blurry + WEB profile, update the base url and API key, then add the UHD Bluray + WEB profile removing the first `radarr:` line. 
- Note, if you were to add the anime profiles, you'd do the same, remove `radarr:` line, since we already told the config we're working in radarr on the first HD profile.

## Sync

Run a test sync via:
```bash
docker compose run --rm recyclarr sync
```

Check the logs and make sure the profiles are updated or created, and all the new custom formats and things are created.

If so, make sure to use `docker compose up -d` to actually start recyclarr. It will update daily.
