## Setting up the Arrs using webUI

### qBittorrent 

default user/pass is admin/adminadmin - see trash guide: https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/ for basic setup and work through that.

I added categories for 4k, such as tv4k, movies4k as well so that I can optionally use Overseerr to request in 4k for a few shows if I want.

### Prowlarr

created an account, and setup my first torrent indexer

we'll come back here with our *arr APIs later

### Radarr, Radarr4k, Sonarr, Sonarr4k

The general idea here is to launch the webUI, grab the API key from settings, general, go back to Prowlarr and setup the apps, then go back and work through each.

Once you have that sorted, go to the app, and add your downloader qBittorent. Be sure to set the proper category (e.g., Radar = movies, Radar4k = movies4k, which are not the default categories in the settings menu). Then go to media management, turn on advanced, and rename movies, skip free space check, use hard links, import extra files (srt = subtitles), and add the root folder (e.g., media/media/movies for Radarr, media/media/movies4k for Radarr4k).

Do this for each.

Then in Plex, make sure you add these folders, e.g., movies monitor both movies and movies4k so all those movies all show up together.

Optional: setup qualitfy profiles to make sure the right files are grabbed for your system - https://trash-guides.info/Radarr/

Optional: Link the libraries of the regular and 4k versions of Sonarr and Radarr (I'm using option 2 to occassionaly request 4k versions of things) - https://trash-guides.info/Radarr/Tips/Sync-2-radarr-sonarr/ 

## Setup overseer for requests

Launch the overseerr webUI, add your Arrs, set both the regular and 4k versions as 'default' and select the quality profiles that will be used automatically. I'm setting up a second user for my family to request



