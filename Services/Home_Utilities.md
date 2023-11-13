# Pihole

- install the PiHole bare-metal in an LXC via the Proxmox >_shell terminal `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/pihole.sh)"` - use advance settings and set a root password, static IP, and enable root SSH.
- This can take a while so let it run (alternatively use 'verbose mode' when installing the script to see what is happening during the 'updating container OS' phase which happens to take the longest)
- reboot the LXC
- Then run `pihole -a -p` to set a password.
- Pihole admin is found via IP/admin

### Setup gravity sync

Since this is my secondary PiHole (my first lives on separate hardware), I want to install gravity sync to keep them both identical.

#### on both pihole systems

- Run `curl -sSL https://raw.githubusercontent.com/vmstan/gs-install/main/gs-install.sh | bash` and go through the setup
- You'll need your user passwords for both instances
- afterwards, run `gravity-sync push` on the local machine that you want to send to the other, remote machine
- If you need to remove the ssh connection details of the remote (e.g., something went wrong), you can run `ssh-keygen -f "/home/pi/.ssh/known_hosts" -R "[remoteIP]"` This assumes that your local user is called `pi`. On the proxmox lxc use `ssh-keygen -R [remoteIP]` instead.
- You can reset your config file by re-running `gravity-sync config` 
- Note gravity-sync auto completely fills your LXC logs, so for now keep that off.

#### Troubleshooting remote not detected error on LXC

on the local machine, go to `cd /etc/gravity-sync`, `cat gravity-sync.rsa.pub` and copy the public address. On the remote machine, go to `~/.ssh`, `nano authorized_keys`, and paste the public key into this file. Then reset the config on the lxc using gravity-sync config. It should now connect. You may have to do this on both machines if you've recreated an LXC and had done this before (i.e., the raspberry pi may have the 'old' ssh key still installed).

## Install Unbound 

Follow the official directions: https://docs.pi-hole.net/guides/dns/unbound/
