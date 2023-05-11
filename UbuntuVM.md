# Ubuntu VM with Plex

In proxmox, download a Ubuntu server iso, then create a new VM. I'm following this [guide](https://www.youtube.com/watch?v=p6aSlcbDHqc&t=474s), but notably I'll be installing the VM to my NVME, and not the shared storage.

**Note:** It will likely be smarter to host plex as an LXC container, and then all the arr software in a separate LXC. I'm going to follow through with a VM for now, but may shift to that setup later.

## VM settings

Here's the settings I used for the OMV virtual machine

| General                                                      | OS                                                  | System     | Disks                                                      | CPU                                                          |
| ------------------------------------------------------------ | --------------------------------------------------- | ---------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| **name:** ubuntu-plex                                        | Select the Ubuntu server iso you downlaoded earlier | no changes | Storage: `local-lvm` (my host NVME), cache `write-through` | **Number of cores:** `6` (or more), select the type as `host` from the dropdown list |
| **advanced:** start/shutdown order: `2`, check `start at boot` |                                                     |            | Check `ssd emulation`                                      | **Memory:** `8192`, uncheck ballooning                       |

Click finish, start the VM, then click console to start the Ubuntu install.

I used the default options up to the Network connections, then selected the network `ens18` and manually changed the IPv4 from DHCP to static using the following

``` bash
Subnet: 192.168.0.0/24 #router address, but with zero in the last place
Address: #the same address that was already assigned#
Gateway: 192.168.0.1 #router address
Name servers: 192.168.0.1 #which should use my router, which will relay requests to my pihole
```

Continue to guided storage configuration, and uncheck `setup this disk as an LVM group`

Then proceed

### Profile Setup

Your server's name: give it something easy to find on a network, then pick a username and password

### Install OpenSSH server

Select yes to install OpenSSH server

Proceed, and wait for installation to complete

Reboot, then wait for it to reboot, then stop the VM, go to the VM in proxmox, and select hardware, and click CD/DVD Drive, edit, and `do not use any media` so that it doesn't try and boot from the installer file.

## Pass through the integrated gpu for hardware transcoding

### Enable IOMMU

When you pass through your CPU's HD 630 integrated GPU, it will likely give you the error `IOMMU not present`. First, you want to make sure that you have enabled VT-d and virtualization in the bios. For me, this was found by restarting, using `F1` during boot, going to advanced, cpu, and enabling the options there. 

Then, to enable IOMMU in the shell, following this [guide](https://pve.proxmox.com/wiki/PCI_Passthrough#Enable_the_IOMMU). Go to the node, and select >_Shell. Then work through the following commands

```bash
nano /etc/default/grub
#find the following line
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
#change it to
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommmu=pt"
#save and exit
update-grub
nano /etc/modules
#add the following (copy/paste)
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
#save and exit
```

### Enable CPU iGPU hardware passthrough

Go to the VM, select hardware, add, PCI device, and select the option with [UHD Graphics 630], check `all functions`

Start your VM!

### SSH into your VM using terminal (or use the console in proxmox)

- `ssh username@[ubuntuplexIP]`
- optional, setup SSH keys to simplify. On your mac terminal 
- `/usr/bin/ssh-copy-id username@ubuntuIP` enter your password, then when you reconnect, it should allow you to ssh in without your password
- If you don't have that command, use this [guide](https://www.ssh.com/academy/ssh/copy-id#how-ssh-copy-id-works) to install ssh-copy-id on your Mac


## Get setup

Get your VM up to date

```bash
sudo su

sudo apt-get update && apt-get upgrade -y

reboot

apt-get install qemu-guest-agent
```

Go back to proxmox, go to the VM, then options, QEMU Guest Agent, edit, and check both options 

