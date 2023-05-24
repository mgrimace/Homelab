# Connect the various Dockers to a single, central, Portainer. 

create a ubuntu lxc to host portainer, I used the script, then:

- Add ibramenu
- installed the basics
- installed docker socket
- installed portainer

Now, one-by-one, go to each LXC where you have Docker. On anything LXC's where the Docker apps were installed by Ibramenu, remove app armour:

`apt remove app armor` then reboot

Then on each docker lxc, paste the following into the >_console to install a portainer agent

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

Don't forget to add portainer to your homepage!