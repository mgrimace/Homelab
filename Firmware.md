# Updating system BIOS and firmware from the Proxmox CLI

I'm using https://github.com/fwupd/fwupd which is descirbed as making firmware updates on Linux safe and reliable. Be sure to back everything up first. This should update the system and any attached devices (e.g., SSDs, etc.) - basically anything with firmware submitted to the Linux Vendor Firmware Service 

## Installation

1. In the Proxmox Node >_shell: `apt install fwupd`
2. Then, `fwupdmgr get-devices`
3. `fwupdmgr refresh`
4. `fwupdmgr get-updates`
5. `fwupdmgr update`

Reboot