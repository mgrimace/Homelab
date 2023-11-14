# Local-only subdomains

You can use access lists in NPM to make some services only work on your local network.

## Basic steps

1. In pi-hole or your DNS host, add a local DNS entry with servicename.mydomain.com pointing to your NPM IP.
2. In NPM create an access list with the only allowed entry as 192.168.0.0/24, which should be my home network (e.g., 192.168.0.1-192.168.0.244).
3. In NPM, create a matching host service.mydomain.com with the IP/Port it uses, then apply the new local only access list. Hit save.

## To add SSL (https) to the local-only domain

- In NPM, click on the SSL tab
- Drop down "None" for encryption and choose "Request a new SSL certificate"
- Enable "Force SSL", "HTTP/2 Support", "HSTS Enabled", and "Use a DNS challenge"
- Under "DNS Provider", choose CloudFlare
- Here you, will need a Cloudflare API token to enter into `credentials file content`
- In a new tab, visit Cloudflare, profile, create token: `use Template`, select `edit zone dns`
  - under permissions, + Add more with the options: `zone, zone, read`
  - under Zone resources, selected my domain in the far right field, 
  - Set the TTL as long as you like, then probably set yourself a reminder for when it expires.
- Click Continue to Summary at the bottom
- Click Create token
- Click Copy on your API token
- Back in NPM, under `Credentials file content`, change the token to the token you copied from the CloudFlare page
- Enter your email at the bottom and agree to the terms
- Click Save

## Troubleshooting

- First, use a private/incognito tab to visit the site if you've been making changes in NPM, sometimes the current browser needs to clear the cache/restart for changes to take effect.
- If you made changes to the access list, you need to edit the host and re-save it. 
- If you made changes to the access list, you may also need to restart the npm docker container.
- If you have two piholes (primary, secondary) ensure that the your local DNS records are in sync (e.g., using gravity-sync)
- In PiHole, under settings / DNS, deselect (uncheck) the options `never forward non-FQDN A and AAAA queries` and `never forward reverse lookups for private IP ranges`.
- Then, be sure to use the Flush DNS Cache option in Pi-Hole settings. 
- On Mac computers, use the following terminal command to flush the DNS cache:

```bash
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
```

**Note:** Adding Authentik appears to supercede this, and will allow  access to the proxy host, but of course with Authentik authentication.



