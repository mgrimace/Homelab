# Create and manage users in Authentik

## Create new users

### Use Plex credentials to login/create users

in authentik -> Sources

Add *Plex* as a *source*

- Name: Choose a name
- Slug: Set a slug
- User matching mode: I selected `link to a user with identical email....` so that I can link my Plex to my admin account without creating separate users
- Under protocol settings:
- Client ID: Set a unique Client Id or leave the generated ID
- Press *Load Servers* to login to plex and pick the authorized Plex Servers for "allowed users"
- I selected the option to `allow friends to authenticate via Plex...`, that way I can easily add new users. Note that the person will receive an error unless you add them as a friend within Plex first by going to: `https://app.plex.tv/desktop/#!/friends`. This prevents anyone with Plex from creating an Authentik user account.

Save, and you now have Plex as a source.

### Add plex to login page

1. Access the **Flows** section
2. Click on **default-authentication-flow**
3. Click the **Stage Bindings** tab
4. Chose **Edit Stage** for the *default-authentication-identification* stage
5. Under **Sources** you should see the additional sources you have configured. Click all applicable sources to have them displayed on the Login Page

#### Use logged-in Plex credentials as SSO for Overseerr

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

## Add users the old-fashioned way

Video resource: https://www.youtube.com/watch?v=mGOTpRfulfQ

## Ensure only the right people see your apps

- Use groups, found under the admin interface, directory, groups
- I made sure that I was in the `Authentik Admins` group and created an `Authentik Users` group with two 'child' groups: `friends` and `family`.
- Go to each Application, and click it to see it's metadata, select the `bindings` tab.
- I bound each app to the `authentik admin` group, then  I added the full `authentik user` group to the apps where I want *both* friends and family to have access, and then the respective child groups to the apps where I just want one or the other.