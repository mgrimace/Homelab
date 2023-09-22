# Using Access Lists in NPM to only allow access to reverse proxied sites from local connections

- When I add a new proxy host, I give it a name, e.g.,service.mydomain.com` then, under SSL I select my cloudflare origin certificate, and check the two top options `force ssl` and `http/2 support`
- If I select `public` it work, the service will be available publiclly. However, I want to use an access list to make sure that I can only reach it from connections from my IP (e.g., my local network), or specific IPs
- I created an access list that has my local network allowed `192.168.0.0/16`. I also tried adding my router external IP from ipchicken.com
- In PiHole, I added `service.mydomain.com` and set its IP to my *NPM* local IP.
- In PiHole, under settings / DNS, deselect (uncheck) the options `never forward non-FQDN A and AAAA queries` and `never forward reverse lookups for private IP ranges`.
- If you have two piholes (primary, secondary) ensure that the your local DNS records are in sync (e.g., using gravity-sync)
- Then, be sure to use the Flush DNS Cache option in Pi-Hole settings. 
- On Mac computers, use the following terminal command to flush the DNS cache:

```bash
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
```

**Note:** Adding Authentik appears to supercede this, and will allow  access to the proxy host, but of course with Authentik authentication.