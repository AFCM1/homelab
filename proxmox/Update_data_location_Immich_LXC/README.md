## Tutorial to change Immich data location  (LXC) ✍️

**Use case:**  
Promox PVE 8.2.4  
LXC Immich  
I want to have my Immich data to be located on my Synology  
Synology target location for data: /backup/immich  

On synology make sure to create the necessary sub-folders in /backup/immich/: 
- backups
- encoded-video
- library
- profile
- thumbs
- upload

Add the shared folder **backup** in the NFS perms on Synology like this:
![enter image description here](https://i.imgur.com/aldEjlc.png)

### On proxmox node  

:one: Create the folders where the shares will be mounted:  
> mkdir -p /mnt/backup/immich  

2️ Edit /etc/fstab and add these lines:

Check the uid and gid of the immich user on LXC so you can edit the fstab accordingly:
> immich:~# id immich  
> uid=999(immich) gid=995(immich) groups=995(immich),44(video),104(render),10000(lxc_shares)

Then edit the fstab on Promox:  
>//SYNO_IP/backup/immich /mnt/backup/immich cifs _netdev,x-systemd.automount,noatime,uid=100999,gid=100995,dir_mode=0770,file_mode=0770,user=SYNO_USERNAME,pass=SYNO_PASSWD 0 0

The shared folder will be mounted on /mnt/backup on the proxmox node. Replace SYNO_IP with your Synology IP.  



:three: Reload systemd:  
>systemctl daemon-reload  

:four: Mount shares: 
>mount -a

:five: Add the pointing point to your Linux Container:  
> vim /etc/pve/lxc/ID.conf

Add the following lines into the file:  

>mp0: /mnt/backup/immich/,mp=/mnt/backup/immich

### On LXC  
1️⃣: Start your Immich LXC, create this group and add the user running the jellyfin app into this group:

> groupadd -g 10000 lxc_shares  
> usermod -aG lxc_shares immich

2️ Your shares will appeared in /mnt

:three: To change the upload location, edit 'IMMICH_MEDIA_LOCATION' in /opt/immich/.env
> IMMICH_MEDIA_LOCATION=/mnt/backup/immich

:four: Delete the symlink **upload** 'located' in /opt/immich/app & /opt/immich/app/machine-learning  
And recreate the symlink **upload** in /opt/immich/app & /opt/immich/app/machine-learning to your new upload location  
> ln -sd /mnt/backup/immich /opt/immich/app/upload  
> ln -sd /mnt/backup/immich /opt/immich/app/machine-learning/upload

:five: Reboot the LXC and then check if somethings fails.  
Log location: /var/log/immich/web.log

### Notes  
Perms on shared fold on LXC:  
> immich:/mnt# ll  
> drwxr-xr-x 3 root lxc_shares 4096 Jul 25 23:21 backup  
> immich:/mnt# cd backup/  
> immich:/mnt/backup# ll  
> drwxrwx--- 2 immich immich 0 Jul 26 00:59 immich  

Perms on shared fold on proxmox node:  
> root@proxmox:/mnt/backup# ll  
> drwxrwx--- 2 100999 100995 0 Jul 26 00:59 immich  
