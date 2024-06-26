# Specific Configs

1. Overseerr, and single-sign-on (SSO) using your plex account
2. Calibre-Web single-sign-on with plex account
3. Proxmox

## Overseer

### Use logged-in plex credentials as SSO for Overseerr

Customization > Property Mappings > Create > Scope Mapping, give it a name (e.g., Plex SSO), and copy/paste the following script, being sure to change `base_url = "http://overseerr:5055"` to your actual local overseerr, for example `base_url = "[http://192.168.xx.yy:5055](http://192.168.xx.yy:5055/)" or https://overseerr.mydomain.com. 

```
from authentik.sources.plex.models import PlexSourceConnection
import json

connection = PlexSourceConnection.objects.filter(user=request.user).first()
if not connection:
    ak_logger.info("PLEX: No Plex connection found")
    return {}

base_url = "http://overseerr:5055"
end_point = "/api/v1/auth/plex"
token = "${overseer_token}"

headers = {
    "Content-Type": "application/json",
}

data = {
    "authToken": connection.plex_token
}


response = requests.post(base_url + end_point,
                         headers=headers, data=json.dumps(data))

if (response.status_code == 200):

    sid_value = response.cookies.get("connect.sid")
    cookie_obj = f"connect.sid={sid_value}"
    ak_logger.info("PLEX: The request was a success!")
    return {
        "ak_proxy": {
            "user_attributes": {
                "additionalHeaders": {
                    "Set-Cookie": cookie_obj
                }
            }
        }
    }
else:
    ak_logger.info(f"PLEX: The request failed with: {response.text}")
    return {}
```

Edit the Organizr Proxy Provider to include your Scope Mapping under Advanced protocol settings, e.g., go to advanced, Scope Mapping, and highlight Plex SSO, then update.

## Calibre-web 

You can automatically log-in to calibre web using authentik if the usernames are the same. Keep that in mind when adding other users.

Same steps as default, but in NPM, advanced, add (noting the proxy buffers)

```yaml
proxy_buffer_size 128k;
proxy_buffers 4 256k;
proxy_busy_buffers_size 256k;

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

### Set-up auto-login using headers from authentik

Go to calibre-web settings, basic configuration, allow reverse rpxoy authetnication, add `X-authentik-username` 

Make sure your authentik username is the same as calibre-webs (e.g., I changed my calibre username from admin to akadmin to match authentik)

## Proxmox VE

To access your proxmox VE remotely we need to do two things, the first is setup a forwardauth provider to authenticate with Authentik first, then we can use Authentik as an OpenID provider to almost get a single-sign-on experience.

Note, for now, this will result in *two* Proxmox providers, and two Proxmox apps in Authentik. I'm not sure if there's a smarter way to do it.

### Setup ForwardAuth in Authentik

This is basically the same as usual

**1. Under Providers:**

- create provider with name: Proxmox ForwardAuth
- Authorization flow: default-provider-authorization-implicit-consent (Authorize Application)
- Forward auth (single application)
- external host:[https://proxmox.domain.com](https://proxmox.domain.com/)

**2. Under Applications:**

- create application with name: Promxox ForwardAuth
- slug: proxmox-fowardauth
- provider: proxmox
- engine policy+any
- Launch URL:[https://proxmox.domain.com](https://proxmox.domain.com/)

Don't forget to add the exceptions under advanced, unauthenticated paths so that the API works well with Homepage

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

**3. In Outpost**

- in you current working "Outpost" enable under "Applications" select "proxmox"

### Setup NPM

Create your reverse proxy in NPM, e.g., proxmox.yourdomain.com. Use your Cloudflare SSL certificate as usual. Be sure to use http<u>s</u>://IP:8006.

#### Advanced config

```bash
# Increase buffer size for large headers
# This is needed only if you get 'upstream sent too big header while reading response
# header from upstream' error when trying to access an application protected by goauthentik
proxy_buffers 8 16k;
proxy_buffer_size 32k;

# Make sure not to redirect traffic to a port 4443
port_in_redirect off;

location / {
    # Put your proxy_pass to your application here
    proxy_pass          $forward_scheme://$server:$port;
    # Set any other headers your application might need
    # proxy_set_header Host $host;
    # proxy_set_header ...
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_http_version 1.1;

    ##############################
    # authentik-specific config
    ##############################
    auth_request     /outpost.goauthentik.io/auth/nginx;
    error_page       401 = @goauthentik_proxy_signin;
    auth_request_set $auth_cookie $upstream_http_set_cookie;
    add_header       Set-Cookie $auth_cookie;

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
    proxy_pass              http://IP:9000/outpost.goauthentik.io;
    # ensure the host of this vserver matches your external URL you've configured
    # in authentik
    proxy_set_header        Host $host;
    proxy_set_header        X-Original-URL $scheme://$http_host$request_uri;
    add_header              Set-Cookie $auth_cookie;
    auth_request_set        $auth_cookie $upstream_http_set_cookie;
    proxy_pass_request_body off;
    proxy_set_header        Content-Length "";
}

# Special location for when the /auth endpoint returns a 401,
# redirect to the /start URL which initiates SSO
location @goauthentik_proxy_signin {
    internal;
    add_header Set-Cookie $auth_cookie;
    return 302 /outpost.goauthentik.io/start?rd=$request_uri;
    # For domain level, use the below error_page to redirect to your authentik server with the full redirect path
    # return 302 https://auth.yourdomain.com/outpost.goauthentik.io/start?rd=$scheme://$http_host$request_uri;
}
```

### Setup openID

See here for setting up Authentik: https://goauthentik.io/integrations/services/proxmox-ve/

Basically follow the same instructions, except in `Providers` omit the port number (:8006) and feel free to leave the launch URL blank in the `Application`. The port is already specified by NPM.

### login

Then, once you go to promox.yourdomain.com, it'll pop-up the login screen, select `Authentik` from the Realm drop-down, then click `login openID redirect`, and it should log you in, and create a user for you based on your proxmox username, e.g., `user@authentik`.

### make your authentik user a proxmox admin

Youll need to make this user and administrator, log back into proxmox as root.

Go back to the Permissions tab (which looks just like a down arrow) and actually set the admin permissions for the user (or alternatively, create an 'admins' group)

