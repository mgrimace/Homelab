# Mounting a USB drive for regular backups

## Format and mount the backup drive

### Step 1

Plug the USB drive into your system. In Proxmox, go to `disks` and note the name of the disk, alternatively you can use `fdisk -l` in the proxmox shell. Mine is /dev/sdb

### Step 2 

Set the file system before mounting it, this will erase the drive

`mkfs.ext4 /dev/sdb`

### Step 3

Create a backups directory to mount to

`mkdir /mnt/backups`

### Step 4

Mount the backup drive to the directory

`mount /dev/sdb /mnt/backups`

### Step 5

Now add the USB drive mount point in the Proxmox web UI to use for backups. This is done in **Datacenter** / **Storage**. Select ***Directory\*** from the ***Add\*** pulldown, then enter the mount point you created and configure it with a content type of VZDump backup file.

## Create an fstab entry, so the drive is automatically mounted at system reboots

### Step 1

Determine the UUID of the drive. You can use...

- `ls -l /dev/disk/by-uuid` or `lsblk -f`
- The UUID will look like...``063c75bc-bcc6-4fa5-8417-a7987a26dccb`

### Step 2

Add an entry in /etc/fstab using the UUID to mount the drive when the system boots. The following format is used for /etc/fstab...

`[Device] [Mount Point] [File System Type] [Options] [Dump] [Pass]`

Using UUID, the entry will look something like...

`UUID=063c75bc-bcc6-4fa5-8417-a7987a26dccb /mnt/backups ext4 defaults,noatime,nofail 0 2`

The USB now will be mounted when the system boots up, and can be mounted manually with the mount command if for some reason it disconnects.

## Make sure Proxmox knows the backups directory is an externally managed mount point and consider the storage offline if it isn't mounted

Edit storage.cfg found in /etc/pve/

add

`is mountpoint 1` to the directory, for example:

```
dir: backups
	path /mnt/backups
	content backup,images
	prune-backups keep-all=1
	shared 0
	is_mountpoint 1
```



