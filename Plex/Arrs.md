## Setting up the Arrs using webUI

### qBittorrent 

default user/pass is admin/adminadmin - see trash guide: https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/ for basic setup and work through that.

I added categories for 4k, such as tv4k, movies4k as well so that I can optionally use Overseerr to request in 4k for a few shows if I want.

**Enable Dark mode:** In webUI, select use alternative webui :vue. It looks better, and select dark mode via the toggle in the bottom left sidebard

### Prowlarr

created an account, and setup my first torrent indexer

we'll come back here with our *arr APIs later

### Radarr, Radarr4k, Sonarr, Sonarr4k

The general idea here is to launch the webUI, grab the API key from settings, general, go back to Prowlarr and setup the apps, then go back and work through each.

Once you have that sorted, go to the app, and add your downloader qBittorent. Be sure to set the proper category (e.g., Radar = movies, Radar4k = movies4k, which are not the default categories in the settings menu). Then go to media management, turn on advanced, and rename movies, skip free space check, use hard links, import extra files (srt = subtitles), and add the root folder (e.g., media/media/movies for Radarr, media/media/movies4k for Radarr4k).

Do this for each.

Then in Plex, make sure you add these folders, e.g., movies monitor both movies and movies4k so all those movies all show up together.

Setup qualitfy profiles to make sure the quality is appropriate for the files are grabbed for your system - https://trash-guides.info/Radarr/

Setup naming schemes

Setup custom profiles to make the right filetypes are grabbed (and avoided) for your system (same link as above)

I'm going to use the following profile for most 720/1080p content https://trash-guides.info/Radarr/radarr-setup-quality-profiles/#trash-quality-profiles . Within this profile, it directs you which custom profiles to setup as well. Also making sure I setup the 'unwanted' custom formats.

I also merged qualities: https://trash-guides.info/Radarr/Tips/Merge-quality/

Repeat for each Arr

Optional: Link the libraries of the regular and 4k versions of Sonarr and Radarr (I'm using option 2 to occassionaly request 4k versions of things) - https://trash-guides.info/Radarr/Tips/Sync-2-radarr-sonarr/ - I've decided not to do this, as I will be using overseer to handle requests and it gives me the option to 'download in 4k' if I setup radarr 4k separately (rather than using radar non-4k to handle and sync things)

make sure 'unmonitor' is checked in media settings so that when shows are deleted you don't automatically download them again.

## Setup overseer for requests

Launch the overseerr webUI, add your Arrs, set both the regular and 4k versions as 'default' and select the quality profiles that will be used automatically. I'm setting up a second user for my family to request



