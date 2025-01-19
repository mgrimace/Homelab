# Create a personal landing page with Linkstack and Cloudflare Tunnels

## Create the Linkstack service using docker compose

We will be setting up \[Linkstack\](linkstack.org) in docker to serve as a personal landing page and virtual business card, which we can share via QR code, etc. We will reverse proxy it using Cloudflare tunnels so that it's secure. I'm using CF tunnels instead of NPM because this will be broadly open on the internet without a login, and I'd prefer Cloudflare to handle that.

I've included my compose file in /tunnels/compose.yaml with /networking/crowdsec-compose.yaml that I created, and can be found in \[compose/tunnels\](compose/tunnels). 

For my use, I'm treating Linkstack like my other services here rather than a website per se, and it will require a bit more setup than usual since we're going to bind-mount the config files rather than use the default volume.

- First, we'll need to go to linkstack.org and download the latest .zip. Extract the zip file (all of it, including the invisible files) to /appdata/linkstack. Then, start the container.

- Once started, use `docker exec -it linkstack /bin/sh` to enter commands from inside the container. Use, `chown -R apache:apache /htdocs` to give the correct permissions to those files. Exit, and restart the container.

- It should work now, and be available at `https://IP:8190` locally, noting the use of https://. Ignore certificate warnings for now.

- Make sure in the config in the webui you select Force HTTPS for your cloudflare tunnel (or reverse proxy) to work. Here, I also set to use my page as the home page, and turned off all the other user features so this just shows me.

## Setup Cloudflare certs for your tunnel and HTTPS
Next in Cloudflare, go to your domain SSL, and origin certs.
- I created an origin certificate in cloudflare with linkstack.mydomain.com (with the exact subdomain specifically as the host, no wildcard, just the specific subdomain).
- Save the certificate as server.pem, and the key as server.key, then put them in your /appdata/certificates folder. We have this folder bind mounted in our compose.yaml (e.g., `${DOCKERDIR}/appdata/certificates:/etc/ssl/apache2`. 
- Restart the linkstack container (no permissions changes needed)

## Create a Cloudflare Tunnel
- There's other great Youtube guides for this already. Generally, go to ZeroTrust, networks, then tunnels, and create a tunnel for your homelab. Note the token it creates. Add this token to your .env in compose/tunnels/.env. Recreate the cloudflare container. 
- Back in the Cloudflare you should see your tunnel as healthy. 
- Create a public hostname and name it linkstack.mydomain.com (or whatever you prefer, making sure your certificate above matches the name). Then, for service use `HTTPS://linkstack:443` for the type (HTTPS), and URL(containername:443). Under additional application settings, TLS, set the Origin Server Name, again, use that same hostname linkstack.mydomain.com that you created with the origin certificate. 

Save. Now you should be able to visit your page at https://linkstack.mydomain.com and have it securely served with Cloudflare Tunnels.
