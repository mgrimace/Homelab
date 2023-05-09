# Open Media Vault

- Download ISO in proxmox
- Create a new VM

## VM settings

### General

- Name it omv-nas 
- click advanced
- set start/shutdown order: 1, startup delay 60 (so this starts first then makes sure all the shares are passed to the other VMs to avoid errors)

### OS

- Pick the ISO

### System

- Check QEMU agent

### Disks

- Set a disk size of 8GiB for OMV
- Check SSD emulation

### CPU

- 2 cores, select the type as 'host'
- Memory, type 4096, uncheck ballooning
- then select next, don't select start automatically, since we'll be passing through our media drive before we start it up.

## Pass through the storage drive

We'll be passing through the media drive to OMV to handle sharing it as a NAS.

Select the node (proxmox), select disks, then find the SSD, note its name. I'm using steps from this [guide](https://dannyda.com/2020/08/26/how-to-passthrough-hdd-ssd-physical-disks-to-vm-on-proxmox-vepve/).

1. In the node (proxmox), click shell, then`apt install lshw` - which shows relevant data for storage disks
2. `lshw -class disks -class storage` - I copied and pasted the info from the output here re. my disk
3. Find the disk ID `ls -l /dev/disks/by-id/` - find the serial that matches the device info from the previous command: `ata-Samsung_SSD_870_QVO_2TB_SERIAL` (note: actual serial redacted)
4. Add the disk to the OMV: `qm set 100 -scsi1 /dev/disk/by-id/ata-Samsung_SSD_870_QVO_2TB_SERIAL` 
   1. where 100 = what we set as our OMV VM ID, and scsi1 is the virtual port number (since our NVME is already scsi0)

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

### Create a new filesystem

- Go to storage, File Systems, select + and create a new filesystem
- Select your device (your SSD), and we'll keep it the default EXT4 for now because I don't know better.
- Next, mount your new filesystem

### Create a new share 

We're going to structure the filesystem based on Trash [Guide](https://trash-guides.info/Hardlinks/How-to-setup-for/Native/)

- Go to storage, shared folders, and create a new share. Name it `data`.
- Go to services, SMB/CIFS, and create a new SMB share, selecting the `data` folder. 

### Create users

Following this [guide](https://www.techrepublic.com/article/add-users-groups-openmediavault/). 

- Go to Users, Users, the hit the plus symbol to create a user. Give them a name, email, and password, then create.

- Go back to the user page, then select your new user click `shared folder permissions` to give the user access to your new shared folder

