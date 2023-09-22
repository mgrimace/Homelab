# Homepage

[Homepage](https://gethomepage.dev/en/installation/) is a neat way to collect your various service WebUI URLs in one convenient place. 

My homepage configs, compose, and screenshots are found at [services/homepage](/Services/homepage)

<p align="center">
  <img src="homepage/image_1.png?v=2" />
</p>

## Setup

### Create a directory

```bash
cd /opt/appdata/
mkdir homepage
cd homepage mkdir config
cd ..
```

### Create a secrets file

**Note:** I recently added a .secrets.env file. This allows me to keep all my passwords, API keys, and IP addresses out of my services.yaml. 

Inside the secrets file, you can start adding labels, for example:

```yaml
HOMEPAGE_VAR_SERVER_IP=192.168.x
HOMEPAGE_VAR_DOMAIN=mydomain.com
HOMEPAGE_VAR_PLEX_IP=192.168.y
HOMEPAGE_VAR_USER=username
HOMEPAGE_VAR_USER_EMAIL=my email
HOMEPAGE_VAR_PLEX_KEY=key 1
HOMEPAGE_VAR_SONARR_KEY=key 2
```

You'll use these values, e.g. `{{HOMEPAGE_VAR_PLEX_KEY}}` instead of writing out your Plex key in your configuration file below. Refer the sample services.yaml to see how the labels work. If this doesn't make sense, you can skip this step, and remove the line from the compose file for now then come back to it later.

## Install via docker compose

See my compose, for example: [compose.yaml](/Services/homepage/compose.yaml)

### Configure Homepage

Your homepage is at: IP:3000

Your config files live in /opt/appdata/homepage/config. Use the [guide](https://gethomepage.dev/en/configs/services/) to set up various services and widgets.

#### Show the Docker container status

First, ensure Docker Proxy was added via Ibramenu. 

### docker.yaml

```
my-docker:
    host: dockerproxy
    port: 2375
```

Then in services.yaml, add the following lines to each of your services (just before widget, this is just an example, so be sure to use the right names for your project)

```
server: my-docker
container: sonarr # The name of the container you'd like to connect
```

