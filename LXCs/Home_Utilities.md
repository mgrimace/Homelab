# Home utilities

1. [Pi-hole](#Pihole)
2. [Homebridge](#Homebridge)

## Pihole

- install the PiHole LXC via the proxmox >_shell terminal `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/pihole.sh)"` - use advance settings and set a root password, static IP, and enable root SSH.
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

## Alt: Running cloudblock on a Debian LXC (not working at present)

create a Ubuntu LXC using: `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/ubuntu.sh)"` use advanced settings, and select 6gb hd, 1024 ram, root password, enable root ssh.

First, let's create a user called *pi*, `adduser pi`, give them a password, and give them sudo privileges `usermod -aG sudo pi`

Switch to the pi user `su pi` 

Install some dependencies

```bash
# Ansible + Git
sudo apt update && sudo apt -y upgrade
sudo apt install git python3-pip
pip3 install --user --upgrade ansible

# Add .local/bin to $PATH
echo PATH="\$PATH:~/.local/bin" >> .bashrc
source ~/.bashrc

# Install the community ansible collection
ansible-galaxy collection install community.general
```

We should be able to follow the ubuntu deployment here https://github.com/chadgeary/cloudblock/tree/master/playbooks#ubuntu-deployment

```bash
ansible-playbook cloudblock_amd64.yml --extra-vars="doh_provider=$doh_provider dns_novpn=$dns_novpn wireguard_peers=$wireguard_peers vpn_traffic=$vpn_traffic docker_network=$docker_network docker_gw=$docker_gw docker_doh=$docker_doh docker_pihole=$docker_pihole docker_wireguard=$docker_wireguard docker_webproxy=$docker_webproxy wireguard_network=$wireguard_network wireguard_hostname=$wireguard_hostname"
```

## Homebridge

Run this script from the shell of proxmox, it will automatically create a homebridge LXC with 1 CPU, 1 Gb Ram, and 8 Gb storage:

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/homebridge.sh)"
```

Remember to set a static IP for the LXC in proxmox