# Proxmox

## Install proxmox on your server hardware

1. Create a bootable USB using RUFUS, and write the latest Proxmox ISO to the disk
2. Boot into the USB, and select install Proxmox VE
3. Agree to the EULA
4. Select the drive that you want to install Proxmox to (I used the NVME)
5. Select options and decide on ext4 or xfs as the filesystem (I chose xfs, I don't have strong rationale for either option)
6. Continue until the Network settings, then select the eth (ethernet) option, and be sure to fill in your default Gateway. 
7. My PiHole is my DNS server; however, my router acts as a DNS relay so I entered my Gateway (router) address as my DNS server address. 
8. Install, reboot, and take note of the IP address you'll use to connect

Also, set this (and all vms) as static IPs in your router.

## Setup

Use this post install script `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"` I said 'yes' to all options except for adding the 'test' repo. Note I disabled high availability to save resources and may want to re-enable that later if I'm adding more clustered nodes.

### Alternatively

- Log in from your browser, and select your node (mine's called proxmox), then updates, repositories.

- Disable the `enterprise` repository, and add the `pve-no-subscription` repo. Go up to updates, hit refresh and update your system.

- To remove the nag screen, I followed this [guide](https://dannyda.com/2020/05/17/how-to-remove-you-do-not-have-a-valid-subscription-for-this-server-from-proxmox-virtual-environment-6-1-2-proxmox-ve-6-1-2-pve-6-1-2/). Input the following line of code in the shell: `sed -i.backup -z "s/res === null || res === undefined || \!res || res\n\t\t\t.data.status.toLowerCase() \!== 'active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service`

- Clear your browser's cache, then restart your browser. The nag screen should be gone.
