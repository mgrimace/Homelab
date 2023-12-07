# Local-only subdomains

You can use access lists in NPM to make some services only work on your local network. Please see new note at the bottom if you use Safari.

There are two options to make this work:

**Option 1** - use a service.**local.**mydomain.com address. Rationale: keeps everything local *obviously* local, and doesn't need an access list setup in NPM since Cloudflare isn't set up for `*.*.mydomain.com`. If you decide to go this route, follow-along below and add the **.local.** Feel free to skip adding the access list in NPM (or add it anyways for extra reassurance).

**Option 2** - use your existing service.mydomain.com address. Rationale: keeps all addresses the same and easy to type, but requires an access list and doesn't inherently give you a sense of whether it's *actually* local-only or not.

## Basic steps

1. In pi-hole or your DNS host, add a local DNS entry with servicename.mydomain.com pointing to your NPM IP.
2. In NPM create an access list with the only allowed entry as 192.168.0.0/24, which should be my home network (e.g., 192.168.0.1-192.168.0.244).
3. Add the access list to any services you want to keep local.
4. **Note:** Whenever you edit the access list in NPM, you have to go in to each proxy host that uses this list and hit save again.
5. In NPM, create a matching host service.mydomain.com with the IP/Port it uses, then apply the new local only access list. Hit save.

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

## Clear DNS caches and test it out

You'll likely need to clear your DNS cache in Pi-Hole and on the computer/device. It's also a good idea to restart the npm container at this point, as certain changes/edits need a restart to apply.

- Use the Flush DNS Cache option in Pi-Hole settings. 
- On iPhone, toggle airplane mode on/off
- On Mac computers, use the following terminal command to flush the DNS cache:

```bash
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
```

## Troubleshooting Safari

Safari may give you errors browsing to your local sites, where Chrome and Firefox will appear to work fine. This is intensely frustrating. Here's my understanding of the issue, and the solution after working it through:

- Safari via both Mac and iOS appear to make requests seemingly randomly via A (IPv4) and AAAA (IPv6) regardless of whether or not IPv6 is enabled at the router, etc.
- I had added an entry for `service.local.domain.com` to my NPM container IP, and needed to repeat the same entry with it's IPv6 address.
- I found the IPv6 address by going to the container and using `ip a` and picking the entry at eth0
- I added that to Pi-Hole
- An alternative option is to disable IPv6 in MacOS and Safari: https://www.comparitech.com/blog/vpn-privacy/disable-ipv6-on-devices/

## Other troubleshooting

- First, use a private/incognito tab to visit the site if you've been making changes in NPM, sometimes the current browser needs to clear the cache/restart for changes to take effect.
- If you get a 403 error, you may need to disable HTTP/2 support in NPM
- If you have two piholes (primary, secondary) ensure that the your local DNS records are in sync (e.g., using gravity-sync)
- In PiHole, under settings / DNS, deselect (uncheck) the options `never forward non-FQDN A and AAAA queries` and `never forward reverse lookups for private IP ranges`.
- **Note:** Adding Authentik appears to supercede this, and will allow  access to the proxy host, but of course with Authentik authentication.



