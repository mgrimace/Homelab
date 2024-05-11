# Setup Crowdsec with NPM in Docker

We will be setting up Crowdsec in docker to work with our existing NPM setup. We will do this mainly by swapping my default /networking/compose.yaml with /networking/crowdsec-compose.yaml that I created found in [compose/networking](compose/networking). It is based on the official resource found [here](https://github.com/crowdsecurity/example-docker-compose/tree/main/npm). I made this brief guide because the other resources (e.g., YouTube, blog posts) are out of date or missing key steps.

The following steps will work with an existing NPM setup. However, a few preparations steps are required first, and some minor changes after to make sure our port doesn't confict with qbittorrent.

## Preparation

First, rename compose.yaml to compose.yaml.bak, and then rename crowdsec-compose.yaml to compose.yaml making it our new docker compose file. Don't start the container yet, it's important we do the next steps first

### Create your acquis.yaml

First, we need to create the acquis.yaml file in ${DOCKERDIR}/appdata/crowdsec, which should not exist yet. Navigate to /appdata, then create the crowdsec folder `mkdir crowdsec && cd crowdsec` and create the acquis.yaml in the folder using `sudo nano acquis.yaml`. Paste the following:
```yaml
filenames:
  - /var/log/npm/*.log
labels:
  type: nginx-proxy-manager
```

### Update the .env file

Next, we will we move back to /compose/networking and we will be adding two entries to our .env here:

```bash
##CROWDSEC
ENROLL_KEY=
CROWDSEC_BOUNCER_APIKEY=
```

#### Enrollment key

- The enrollment key can be found by creating an account at [app.crowdsec.net](app.crowdsec.net).
- The key will be found under 'Enroll your CrowdSec Security Engine is as easy as:` `sudo cscli console enroll <ENROLL_KEY>` copy and paste this key into your .env file

#### Bouncer API key

- You will create your npm-bouncer API key using the following:

```bash
docker compose up crowdsec -d
docker compose exec crowdsec cscli bouncer add npm-bouncer
docker compose down
```

- Copy/paste the key into your .env

## Start compose.yaml

Note the main changes are to the NPM image, which is a drop-in replacement that enables Crowdsec to work. We also added the Crowdsec Bouncer in the NPM environment. Under Crowdsec, the main thing is to make sure that in volumes, that the /npm/data/logs is the correct path to your NPM logs. Otherwise, we should be able to start the containers with `docker compose up -d`.

## Post-install, change the port in the right places

Stop Crowdsec with `docker stop crowdsec`. Next navigate to /appdata/crowdsec/config. We need to change two files here:

- in config.yaml - change the port to 8888 on line 37: `    listen_uri: 0.0.0.0:8888` 
- in local_api_credentials.yaml - make the same change to the port to 8888

Restart Crowdsec with `docker start crowdsec`. 

## Visit app.crowdsec.net 

visit app.crowdsec.com and confirm your new system. You should see here your new Security Engine with 1 remediation component and a number of scenarios. You can and should add some blocklists here too (I picked three and set the default to 'ban'). 

You can also confirm its running by using `docker exec -it crowdsec cscli metrics` and amking sure you see some npm logs in 'acquisition metrics' with lines read.