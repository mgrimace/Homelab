iGPU passthrough on LXC (e.g., plex docker)

1. and for iGPU passthrough you need to add these 2 lines into your LXC container

   ```
   lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
   lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=file 
   ```

1. edit the file

   ```
    /etc/pve/lxc/<container#>.conf
   ```

   

   ![Image](https://media.discordapp.net/attachments/946314371796172820/1105871134156656781/image.png?width=1100&height=422)

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

   

   ![Image](https://media.discordapp.net/attachments/946314371796172820/1105872377482584074/image.png?width=420&height=700)

lxc.cgroup2.devices.allow: c 226:0 rwm lxc.cgroup2.devices.allow: c 226:128 rwm lxc.cgroup2.devices.allow: c 29:0 rwm

