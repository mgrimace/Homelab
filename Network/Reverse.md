# Creating a reverse proxy and to access your apps and services remotely

1. [Setup](#Conceptual_overview)
2. [User management](Network/Users.md)
3. [Specific app configs](Network/Authentik_Configs.md)

## Conceptual overview

Overall, we're going to buy a domain (mydomain.com) and use NGINX as a reverse proxy to shuttle the website A.mydomain.com to Container A on my home network. We'll be using Cloudflre origin and edge certificates to add  layer for security and encryption between my home and the internet, and we'll also use Authentik to user handle logins and for additional security.

Here are the three guides I followed, in order:

1. https://www.youtube.com/watch?v=h1a4u72o-64&t=441s

2. https://www.youtube.com/watch?v=c6Y6M8CdcQ0&t=1170s

3. https://www.youtube.com/watch?v=CPURnYaW3Zk&t=301s

Written guides: https://geekscircuit.com/set-up-authentik-sso-with-nginx-proxy-manager/

## Prepare

### Buy a domain

I used porkbun.com to buy a cheap domain name, here I'll call it mydomain.com

Then, we'll register for a free Cloudflare account

### Cloudflare setup

#### On Cloudflare

- Add your new domain, selecting the free plan

- For DNS records, select `cname`, and point to your router's DDNS e.g., x.tplink.com. If you don't have ddns, point directly to your router's external address, or read up on DuckDNS (I didn't do this since I have my own).
- Hit continue, it will tell you to change the nameservers

#### Back to your domain issuer

- Go back to porkbun, then click your new domain, expand, and change 'authoritative name servers' ([guide](https://kb.porkbun.com/article/22-how-to-change-your-nameservers))
- then jump back to cloudflare and hit 'check name servers'
- In the next screen (quick start guide):
  - turn on always use https, otherwise use the default options

- Wait for it to confirm (it could take up to 24hrs, and you'll get an email when it's done)

#### Back to Cloudflare

Go to DNS/settings and enable dnssec

Copy the info over to your porkbun dashboard (https://kb.porkbun.com/article/93-how-to-install-dnssec)

Next go back to DNS/Records on cloudflare, add record, cname `*`.*yourdomain*, target your ddns. This will point anything that ends with yourdomain.com, such as overseerr.yourdomain.com back to your server

on Cloudflare, for our reverse proxy we'll only need two entries, since we'll be using a 'wildcard' `*` entry. We'll use our reverse proxy to handle the subdomains like overseerr.mydomain.com and point them to the right containers on our network

```bash
Type: Cname, name: * target: @ 

Type: Cname: name: @ target: DDNS
```

#### Create SSL certificates

- This secures the connection between your host (computer) and Cloudflare (end-to-end)
- Go to SSL/TLS, change the option to `full/strict`
- Edge certificates are created automatically (basically cloudflare -> internet)
- We need origin certifications (basically home -> cloudflare), select origin certificates, click create
- Let Cloudfare generate a private key, then add your root domain e.g., `mydomain.com` and a wildcard domain e.g, `*.mydomain.com` which will cover all your services that you forward  (e.g., overseer.mydomain.com). Choose to have it not expire for 15 years (max).
- Copy the text of the certificate as Cloudfarecertificate.pem (e.g., use Sublime, Notepad++, etc.), and the key as Cloudflarekey.key. Make sure they aren't saved as .key*.txt*, remove the .txt.

## Set up the reverse proxy

**Note:** NPM = NGINX proxy manager, I'll probably use the terms interchangeably until I edit this guide better.

### Create a Ubuntu LXC

- I do so using a script, which is run in your node >_shell: `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/ubuntu.sh)"`
- Select advanced options, and choose unprivileged, 2 cores, 2048 mb ram, and 6-10 gb storage

### Install ibramenu on your Ubuntu LXC

- Install ibramenu using: `wget -qO ./i https://raw.githubusercontent.com/ibracorp/ibramenu/main/ibrainit.sh && chmod +x i && ./i`
- Then use `ibramenu`, and install the basics (option 2, then option 1 to install everything), then go to networking and install NPM. Note if you want to add this to your homepage, add docker proxy while here.
- After installing NPM, it will give you the webUI connection details

### Set up your router

- Go to your router and forward your nginx container 

- create first entry NGINX http, internal 80, exernal 80, TCP&UDP

- second entry NGINX https, internal 443, external 443, TCP&UDP

### Setup NPM

- Go to ssl, add ssl + custom, use the Cloudflare .key first, and certificate .pem second that you created earlier, no intermediate needed.

- Go back to proxy hosts and add your first app, we'll use overseerr.mydomain.com as a test run.


### Add apps/services in NPM

- Go to hosts tab in NPM, add new host

- domain name = [appname].yourdomain.com

- Forward hostname/IP = IP of the LXC, Forward Port = webui PORT

- cache assts (or not, doesn't work well with some), block common exploits, websockets support

- select SSL, and choose your cloudflare certificate, and select Force SSL and Http/2 support.

- don't select HSTS for now.

- Repeat, being sure to create one for NPM and one for 'auth' which we're about to setup now.

- auth.mydomain.com

- name authentik-server forward 9000

- block common exploits, websockets support, setup SSL per above


## Authentik

**Video resource:** https://www.youtube.com/watch?v=Nh1qiqCYDt4

Authentik adds user control and a login prior to forwarding your app. For example, if someone browses to overseer.mydomain.com they'll be redirected first to Authentik to log-in, before even getting to overseer. This adds a layer of security and user control, and can be added or not added to various apps. I'm exploring how to use it for single-sign on (SSO) as well. 

In the same Ubuntu LXC

<u>use ibramenu to install Authentik (found under security)</u>

You can change to the latest version of authentik by opening the docker compose file found in /opt/appdata/authentik by changing the line

`    image: ${AUTHENTIK_IMAGE:-ghcr.io/goauthentik/server}:${AUTHENTIK_TAG:-2023.5.3}` Where _Tag_ is what you change to the newest version #. You have to do this for both the server and the worker.

### Setup Authentik

- install, go to the webui for authentik and create a username and password
- Setup authentication for a test app 

- First make sure you have a working proxied app in NPM, like overseerr


#### In Authentik 

- go to the webadmin panel, and go to applications/providers

- select proxy provider, we need to do this per app

- Name: Overseerr ForwardAuth

- Authorization flow: ... <u>implicit</u>

- Forward Auth (single)

- External host: https://overseerr.mydomain.com

- scroll down and select advanced, unauthenticated paths add:


```bash
^/api/.*
^/api2/.*
^/identity/.*
^/triggers/.*
^/meshagents.*
^/meshsettings.*
^/agent.*
^/control.*
^/meshrelay.*
^/ui.*
```

##### Go to applications

- create app Overseerr

- Slug overseer

- create new provider, select proxy, call it Overseerr ForwardAuth

- authorization flow, choose either, choose implicit.

- Forward auth (single application)

- external host: https://overseerr.yourdomain.com

- then go back and make sure the new provider is selected for overseerr


**Stop here:** go back to proxmox, go to the LXC hosting this, and recreate your containers (to update any changes thus far)

- `go to cd /opt/appdata/npm and recreate the container docker compose up -d`
- `go to cd /opt/appdata/authentik and recreate the container docker compose up -d`

### Back in Authentik

#### Now log back into authentik webui and go to outposts

- Select edit for the existing option, and highlight the new app you want to redirect to Authentik. You'll have to do this for each new app you add.
- Make sure the auth address is not your internal IP but auth.yourdomain.com (so you can authenticate while not at home) hit update.
- **Note:** If you don't see the configuration script below (e.g., this doesn't seem to work on Safari) switch to chrome or another browser. Hold control to select multipe entries, and 

### Go back to NPM webUI

- select your app in proxy hosts, go to edit, and add the following to advanced, noting that you need to change the line that says


`    proxy_pass          http://auth.example.com/outpost.goauthentik.io;` to your http://authentikIP:9000/ or the external path https://auth.yourdomain.com/

```yaml
# Increase buffer size for large headers
# This is needed only if you get 'upstream sent too big header while reading response
# header from upstream' error when trying to access an application protected by goauthentik
proxy_buffers 8 16k;
proxy_buffer_size 32k;

location / {
    # Put your proxy_pass to your application here
    proxy_pass          $forward_scheme://$server:$port;

    # authentik-specific config
    auth_request        /outpost.goauthentik.io/auth/nginx;
    error_page          401 = @goauthentik_proxy_signin;
    auth_request_set $auth_cookie $upstream_http_set_cookie;
    add_header Set-Cookie $auth_cookie;

    # translate headers from the outposts back to the actual upstream
    auth_request_set $authentik_username $upstream_http_x_authentik_username;
    auth_request_set $authentik_groups $upstream_http_x_authentik_groups;
    auth_request_set $authentik_email $upstream_http_x_authentik_email;
    auth_request_set $authentik_name $upstream_http_x_authentik_name;
    auth_request_set $authentik_uid $upstream_http_x_authentik_uid;

    proxy_set_header X-authentik-username $authentik_username;
    proxy_set_header X-authentik-groups $authentik_groups;
    proxy_set_header X-authentik-email $authentik_email;
    proxy_set_header X-authentik-name $authentik_name;
    proxy_set_header X-authentik-uid $authentik_uid;
}

# all requests to /outpost.goauthentik.io must be accessible without authentication
location /outpost.goauthentik.io {
    proxy_pass          http://auth.example.com/outpost.goauthentik.io;
    # ensure the host of this vserver matches your external URL you've configured
    # in authentik
    proxy_set_header    Host $host;
    proxy_set_header    X-Original-URL $scheme://$http_host$request_uri;
    add_header          Set-Cookie $auth_cookie;
    auth_request_set    $auth_cookie $upstream_http_set_cookie;

    # required for POST requests to work
    proxy_pass_request_body off;
    proxy_set_header Content-Length "";
}

# Special location for when the /auth endpoint returns a 401,
# redirect to the /start URL which initiates SSO
location @goauthentik_proxy_signin {
    internal;
    add_header Set-Cookie $auth_cookie;
    return 302 /outpost.goauthentik.io/start?rd=$request_uri;
    # For domain level, use the below error_page to redirect to your authentik server with the full redirect path
    # return 302 https://authentik.company/outpost.goauthentik.io/start?rd=$scheme://$http_host$request_uri;
}

```

## Add new apps to Authentik

### Overall:

1. add the app in NPM + SSL + advanced code snippet above
2. add the app provider in authentik
3. add the app in authentik, 
4. add the app to the outposts in authentik

### Some apps require specific setup

- Resource: https://goauthentik.io/integrations/
- My notes: [Authentik Configs](Network/Authentik_configs.md) 

## Change domain?

Follow guide and generate a new SSL origin certificate.

### In NPM

- Add custom SSL to NPM

- each proxy, change domain e.g., overseerr.old.com -> overseerr.new.com, change SSL cert.


### In Authentik

- change each provder from external host overseerr.old.com -> overserr.new.com
- change outpost config (open in chrome, not safari, and change from autho.old.com -> auth.new.com)

