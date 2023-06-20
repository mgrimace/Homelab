# Connect the Docker services on other LXCs to your single, central, Portainer. 

If you happened to create separate docker LXCs, you can connect them back to Portainer in your central Docker LXC (e.g., your Arr Suite). Paste the following into the >_console to install a portainer agent

```bash
docker run -d \
  -p 9001:9001 \
  --name portainer_agent \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \
  portainer/agent:2.18.3
```

Then, back in your portainer, go to home, settings, environments, add  environment, select docker standalone, then 'agent'. This will present you with the above code (or a newer version, use theirs)

Then at the bottom, one-by-one add each docker LXC, give it a name (e.g., docker-arr) and enter the IP:9001. 

Don't forget to add portainer to your homepage! The portainer widget will only show one environment (e.g., your primary Docker env), but you can cut/paste the same service entry and just change the 'env' variable and name (e.g. Portainer (Calibre)), all using the same API token, IP, etc.