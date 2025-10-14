

# Proxmox

## Install proxmox on your server hardware

1. Create a bootable USB using RUFUS or Balena Etcher, and write the latest Proxmox ISO to the disk
2. Boot into the USB, and select install Proxmox VE
3. Agree to the EULA
4. Select the drive that you want to install Proxmox to (I used the NVME)
5. Select options and decide on ext4 or xfs as the filesystem (I chose xfs, I don't have strong rationale for either option)
6. Continue until the Network settings, then select the eth (ethernet) option, and be sure to fill in your default Gateway. 
7. My PiHole is my DNS server; however, my router acts as a DNS relay so I entered my Gateway (router) address as my DNS server address. 
8. Install, reboot, and take note of the IP address you'll use to connect

Also, set this (and all vms) as static IPs in your router.

## Post-install

Use the handy Proxmox post-install Tteck script. Enter it into the node >_shell: `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"` 

### Idle power saving

Use the TTECK script to set your scaling governor to power-save: `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/scaling-governor.sh)"`

Use `apt install powertop` to install powertop, then run `powertop --auto-tune` to set additional powersaving states for your processor. This must be re-run on reboot, but can be set with cron.

cron entry for auto-tune: `@reboot sleep 60 && /usr/sbin/powertop --auto-tune`

### Install Glances

I simply refer to this guide which describes the process clearer than I can: https://www.derekseaman.com/2023/04/home-assistant-monitor-proxmox-with-glances.html

After installing glances on proxmox, for my SSD/network share to show up, in the Proxmox node shell enter:

```bash
cd ~/.config/glances
nano glances.conf
```

Then add the following line:

```bash
[fs]
allow=cifs
```

Optionally, add other filesystems depending on your network shares.

Save and exit, then `systemctl restart glances.service`

### Disable SWAP on LXCs

For some reason, certain apps and services on my LXCs use swap despite having free memory available, resulting in slow performance and wear and tear on my drive. Technically, I'm not disabling swap, just setting it to only use swap as a last resort. To do so, in the Proxmox node shell: `nano /etc/sysctl.conf` and add the line `vm.swappiness=0`

### Optional: setup iGPU passthrough for LXCs (return later after creating an LXC)

This is done automatically for relevant services if you're using TTECK scripts. However, you may want passthrough for various Docker services as well.

1. For iGPU passthrough you need to add these 2 lines into your LXC container

   ```
   lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
   lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=file 
   ```

1. edit the file

   ```
    /etc/pve/lxc/<container#>.conf
   ```


and

1. add these 3 lines (may only need the first 2 not sure)

   ```
   lxc.cgroup2.devices.allow: c 226:0 rwm
   lxc.cgroup2.devices.allow: c 226:128 rwm
   lxc.cgroup2.devices.allow: c 29:0 rwm
   ```

   (see above, above the highlighted) to the lxc conf

and edit the compose in plex docker:

1. ```
   devices:
       - /dev/dri:/dev/dri 
   ```

