## Tutorial to mount network shares on Linux Container (LXC) ✍️

**Use case:**  
Promox PVE 8.2.2  
I have data on my Synology - Path will be /Download  
I have data in a TrueNas VM - Path will be /mnt/truenas
I have Jellyfin running in a LXC  
I want to access my media files stored in my synology and truenas on Jellyfin 

Make sure to add the shared folder **Download** in the NFS perms on Synology like this:
![enter image description here](https://i.imgur.com/aldEjlc.png)

### On proxmox node  

:one: Create the folders where the shares will be mounted:  
> mkdir -p /mnt/synology/Download  
> mkdir /mnt/truenas

2️ Edit /etc/fstab and add these lines:

>//SYNO_IP/Download /mnt/synology/Download cifs _netdev,x-systemd.automount,noatime,uid=100000,gid=110000,dir_mode=0770,file_mode=0770,user=SYNO_USERNAME,pass=SYNO_PASSWD 0 0

The share will be mounted on /mnt/synology/Download on the proxmox node. Replace SYNO_IP with your Synology IP.  

>//TRUENAS_IP/media /mnt/truenas cifs _netdev,x-systemd.automount,noatime,uid=100000,gid=110000,dir_mode=0770,file_mode=0770,user=TRUENAS_USERNAME,pass=TRUENAS_PASSWD 0 0

The share will be mounted on /mnt/synology/Download on the proxmox node. Replace TRUENAS_IP with your TrueNas VM IP.

:three: Reload systemd:  
>systemctl daemon-reload  

:four: Mount shares: 
>mount -a

:five: Add the pointing point to your Linux Container:  
> vim /etc/pve/lxc/101.conf

Add the following lines into the file:  

>mp0: /mnt/truenas/,mp=/mnt/truenas  
>mp1: /mnt/synology/Download/,mp=/mnt/synology/Download

:six: Start your Jellyfin LXC, create this group and add the user running the jellyfin app into this group:

> groupadd -g 10000 lxc_shares  
> usermod -aG lxc_shares root

More detailed tutorial : https://forum.proxmox.com/threads/tutorial-mounting-nfs-share-to-an-unprivileged-lxc.138506/

:seven: Your shares will appeared in /mnt



