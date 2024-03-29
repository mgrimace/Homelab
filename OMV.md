# Open Media Vault

- Download ISO in proxmox

- Create a new VM

  

## VM settings

Here's the settings I used for the OMV virtual machine

| General                                                      | OS                                        | System             | Disks                                                | CPU                                                          |
| ------------------------------------------------------------ | ----------------------------------------- | ------------------ | ---------------------------------------------------- | ------------------------------------------------------------ |
| **name:** omv-nas (or any name that suits you)               | Select the OMV iso you downlaoded earlier | Check `QEMU agent` | Set a disk size of 8GiB for the OMV operating system | **Number of cores:** `2` , select the type as `host` from the dropdown list |
| **advanced:** start/shutdown order: `1`, startup delay `60` *, check `start at boot` |                                           |                    | Check `ssd emulation`                                | **Memory:** `~~4096~~`2048 (OMV uses less than 1gb itself + OS ram recommendation 512-2gb, monitor and increase if necessary), uncheck ballooning |

**This ensures that the NAS starts first and makes the shares available before the other VMs (e.g., Plex) start*

Select next, **do not** select `start automatically`, since we'll be passing through our media drive before we start up OMV.

## Pass through the storage drive

We'll be passing through the media drive to OMV to handle sharing it as a NAS.

Select the node (proxmox), select disks, then find the SSD, note its name. I'm using steps from this [guide](https://dannyda.com/2020/08/26/how-to-passthrough-hdd-ssd-physical-disks-to-vm-on-proxmox-vepve/).

1. In the node (proxmox), click shell, then`apt install lshw` - which shows relevant data for storage disks
2. `lshw -class disks -class storage` - I copied and pasted the info from the output here re. my disk
3. Find the disk ID `ls -l /dev/disks/by-id/` - find the serial that matches the device info from the previous command: `ata-Samsung_SSD_870_QVO_2TB_SERIAL` (note: actual serial redacted)
4. Add the disk to the OMV: `qm set 100 -scsi1 /dev/disk/by-id/ata-Samsung_SSD_870_QVO_2TB_SERIAL` 
   1. where 100 = what we set as our OMV VM ID, and scsi1 is the virtual port number (since our NVME is already scsi0)

make sure iothread is enabled for second drive in hardware

Go back to your 100 OMV, make sure the new storage is there under disks, then hit start!

## Proceed with the OMV installation

Select Console to see the installation of OMV

### Console

- Select install and go through the various options, my hostname is the default `openmediavault.local`, enter a password and save it somewhere
- !!  Partition disks, you will get a warning that more than one storage device has been detected
- I'm installing the OMV OS to the NVME, and not the SSD I just added. This is showing up in my installer as SCSI3 - 8.6 GB QEMU HARDDISK (rather than the 2.0TB option)
- Continue through the default options, then hit finish installation. 
- It will give you a warning message that you need to remove the installation media after hitting 'reboot'. 
  - Hit ok, then minimize or close the console
  - In proxmox, go to your VM (100 omv-nas)
  - Select hardware
  - Click CD/DVD Drive, edit 
  - Select `do not use any media`, then click OK.

Go back to the console, and note your OMV IP address! Mine is `192.168.0.125`

## OpenMediaVault

- Connect to the IP address in your browser, and enter `admin`, `openmediavault` as the login and password.
- Select Storage, Disks and you should see two disks - the install drive (8gb), and your SSD (2TB)
- (optional) Select System, Workbench, and change the auto logout to 30 mins
- select the gear in the upper corner and `change password` to change your admin password from the default.
- Go to System, Update Management, and perform updates.
  - I kept getting the new upgrade 'held back' in the web console, so I had to go to the command console (in proxmox), log in using `root` and password from the install, then use the command `omv-upgrade` 
- Toggle dark mode by clicking the user icon after the update.

### Set a static IP in OMV

- Go to system, network, and select your network connection
- Under IPv4, change the method from `DHCP` to `Static` in the dropdown, enter the current IP that you use to access OMV.
- Enter the netmask (mine is 255.255.255.0)
- Enter the gateway (mine is 192.168.0.1)
- Be sure to also enter DNS (I use my router, 192.168.0.1 to forward DNS to my Pi-Hole)

### Create a new filesystem

- Go to storage, File Systems, select + and create a new filesystem
- Select your device (your SSD), and we'll keep it the default EXT4 for now because I don't know better.
- Next, mount your new filesystem

### Create a new share 

We're going to structure the filesystem based on Trash [Guide](https://trash-guides.info/Hardlinks/How-to-setup-for/Native/)

- Go to storage, shared folders, and create a new share. Name it `data`.
- Go to services, SMB/CIFS, and create a new SMB share, selecting the `data` folder. 
- Go to services, SMB/CIFS, settings, and hit `enable`

### Create users

Following this [guide](https://www.techrepublic.com/article/add-users-groups-openmediavault/). 

- Go to Users, Users, the hit the plus symbol to create a user. Give them a name, email, and password, then create.

- Go back to the user page, then select your new user click `shared folder permissions` to give the user access to your new shared folder
- Go to starge, shared folders, select `data` and select `permissions` and toggle read/write. Save, and confirm.

### Access the NAS storage share on your computer

On a Mac, open Finder, the select `go` from the menubar, and `connect to server`, then input smb://OMV-IP-ADDRESS. It should ask for credentials for the user you created above. It will appear in finder and on your sidebar under `locations`

## Link storage to proxmox

- Go back to your proxmox webUI, and select datacenter, storage, add new storage
- Select SMB/CIFS
- ID: give it a name (I called mine `vdisk-nas`)
- Server: use the IP address of your OMV server
- Username/Password: add your OMV username/password (that you generated for the share above). 
  - Note if you used keychain to remember the password, don't let it fill in your email, just the username.
- Then it should scan the share, and you can select `data` from share

## Backup the VM, but not the storage

To backup your OMV OS, but not the attached storage, go to proxmox, hardware, and select the attached storage (e.g., 2 TB disk). Click edit, and uncheck 'backup'. That way only the VM will be backed up.

## Note

Make sure that in your hardware tab in proxmox, that your storage drive has io thread checked/enabled.