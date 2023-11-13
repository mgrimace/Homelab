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
