# Specific Configs

1. Plex, Overseerr, and single-sign-on (SSO) using your plex account
2. Home Assistant
3. Calibre-Web single-sign-on with plex account

## Use Plex credentials to login/create users

in authentik -> Sources

Add *Plex* as a *source*

- Name: Choose a name
- Slug: Set a slug
- Client ID: Set a unique Client Id or leave the generated ID
- Press *Load Servers* to login to plex and pick the authorized Plex Servers for "allowed users"
- Decide if *anyone* with a plex account can authenticate or only friends you share with

Save, and you now have Plex as a source.

Add plex to login page

1. Access the **Flows** section
2. Click on **default-authentication-flow**
3. Click the **Stage Bindings** tab
4. Chose **Edit Stage** for the *default-authentication-identification* stage
5. Under **Sources** you should see the additional sources you have configured. Click all applicable sources to have them displayed on the Login Page

### Use logged-in plex credentials as SSO for Overseerr

Customization > Property Mappings > Create > Scope Mapping, give it a name (e.g., Plex SSO), and copy/paste the following script, being sure to change `base_url = "http://overseerr:5055"` to your actual local overseerr, for example `base_url = "[http://192.168.xx.yy:5055](http://192.168.xx.yy:5055/)"

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

## Home Assistant

Home assistant: https://theprivatesmarthome.com/how-to/put-home-assistant-behind-existing-nginx-proxy-manager/

don't use authentik for home assistant (works, but it's complicated to set up) - use app with your new address!

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

Set-up auto-login using headers from authentik

Go to calibre-web settings, basic configuration, allow reverse rpxoy authetnication, add `X-authentik-username` 

Make sure your authentik username is the same as calibre-webs (e.g., I changed my calibre username from admin to akadmin to match authentik)

Proxmox

See here for setting up Proxmox: https://goauthentik.io/integrations/services/proxmox-ve/

The main thing that I need to add was a new user in Permissions/Users to match my login, then go back to the Permissions tab (which looks just like a down arrow) and actually set the admin permissions for the user (or alternatively, create an 'admins' group)