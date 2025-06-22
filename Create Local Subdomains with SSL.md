# Create local-only subdomains

We're going to create a few-local only subdomains using service.**local.**mydomain.com address. Rationale: keeps everything local *obviously* local, and doesn't need an access list setup in NPM since Cloudflare isn't set up for `*.*.mydomain.com`. 

## Basic steps

1. In pi-hole or your DNS host, add a local DNS entry with servicename.mydomain.com pointing to your NPM IP.
2. Note: You can quickly add a bulk list of domains by editing the pihole.toml, look for:
```
  # Array of custom DNS records
  # Example: hosts = [ "127.0.0.1 mylocal", "192.168.0.1 therouter" ]
  #
  # Possible values are:
  #     Array of custom DNS records each one in HOSTS form: "IP HOSTNAME"
  hosts = [
    "IP1 name1",
    "IP2 name2"
  ]
```
3. In NPM, create a matching host service.local mydomain.com with the IP/Port it uses.

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

## A note for Firefox and Pi-Hole users

Firefox may give various errors when browsing to your .local site. To fix:

On Pi-Hole
- Create a file in `/etc/dnsmasq.d`. I called it `20-override-https-rr.conf`. Note: if in docker, cd to the volume directory e.g., `/home/user/docker/appdata/pihole/etc-dnsmasq.d/`
- Add a line for each domain in the form with the specific numbers and sytax as follows: `dns-rr=https://service.local.example.com,65,000100`
- [Update for Pihole V6] In the /etc/pihole/pihole.toml configuration file, change the setting misc.etc_dnsmasq_d to true
```
Should FTL load additional dnsmasq configuration files from /etc/dnsmasq.d/?
etc_dnsmasq_d = true
```
- Then restart pihole `pihole restartdns`

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

