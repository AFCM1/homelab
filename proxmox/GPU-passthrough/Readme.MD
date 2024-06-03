
## Quick tutorial to setup a GPU passthrough - NVIDIA GPU and LXC Jellyfin ✍️

Some tutorials use the nvidia driver, others use the vfio-pci. I couldn't make vfio-pci works but it works great with the nvidia one.

**Use case: 
PVE 8.2.2  
CPU: Ryzen 5 3600  
GPU: GTX970  
Nvidia driver version: 550.78  
LXC Jellyfin container setup with [tteck helper script](https://helper-scripts.com/scripts)**  

### On promox

:one: Make sure to enable Intel VT-d & VT-x and IOMMU in your bios.
:two: Check that IOMMU is enable with this command:
> dmesg | grep -e DMAR -e IOMMU  
> [    0.725409] pci 0000:00:00.2: AMD-Vi: IOMMU performance counters supported  
[    0.732964] perf/amd_iommu: Detected AMD IOMMU #0 (2 banks, 4 counters/bank).

:three: Edit **/etc/default/grub** and add this line:
>if using Intel CPU: intel_iommu=on  
>if using AMD CPU: amd_iommu=on  
>  GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt nofb nomodeset video=efifb:off"

:four: Update the grub to apply the changes
> update-grub

:five: Create/Edit **/etc/modprobe.d/blacklist.conf** so these modules won't be loaded during boot:
> blacklist nouveau  
blacklist radeon  
blacklist nvidiafb  
blacklist nvidia_drm

:six: Reboot
> reboot

:seven: Download and Install the Nvidia driver from [Nvidia website](https://www.nvidia.com/Download/index.aspx)
> Find your driver and right-click on "Agree & Download" copy the link  
> mkdir /opt/nvidia && cd /opt/nvidia  
> wget LINK  
> chmod +x NVIDIA_DRIVER_FILE.run  
> ./NVIDIA_DRIVER_FILE.run

:eight: Edit **/etc/modules-load.d/modules.conf** and add these Nvidia modules:
> nvidia 
> nvidia-modeset 
> nvidia_uvm

:nine: Create/Edit **/etc/udev/rules.d/70-nvidia.rules** and add these lines:  
> KERNEL=="nvidia", RUN+="/bin/bash -c '/usr/bin/nvidia-smi -L && /bin/chmod 666 /dev/nvidia*'"  
> KERNEL=="nvidia_modeset", RUN+="/bin/bash -c '/usr/bin/nvidia-modprobe -c0 -m && /bin/chmod 666 /dev/nvidia-modeset*'"  
> KERNEL=="nvidia_uvm", RUN+="/bin/bash -c '/usr/bin/nvidia-modprobe -c0 -u && /bin/chmod 666 /dev/nvidia-uvm*'"  

:one::zero: Check that the Nvidia driver is running fine:  

![enter image description here](https://i.imgur.com/2RJSwCl.png)

:one::one:  Before mounting the modules to the container we will need to get some numbers on the proxmox host:
> ls -l /dev/nv*  
crw-rw-rw- 1 root root 195,   0 Jun  3 21:14 /dev/nvidia0  
crw-rw-rw- 1 root root 195, 255 Jun  3 21:14 /dev/nvidiactl  
crw-rw-rw- 1 root root 195, 254 Jun  3 21:14 /dev/nvidia-modeset  
crw-rw-rw- 1 root root 236,   0 Jun  3 21:14 /dev/nvidia-uvm  
crw-rw-rw- 1 root root 236,   1 Jun  3 21:14 /dev/nvidia-uvm-tools  
crw------- 1 root root  10, 144 Jun  3 21:14 /dev/nvram  

Write down the number (195,0) (195,255) (195,254) etc... We will need it in the next steps.
-   **`195:255`**: These are the major and minor numbers of the device:
    
    -   **`195`**: The major number identifies the driver associated with the device. In this case, major number 195 typically corresponds to NVIDIA devices (such as the NVIDIA GPU driver).
    -  **`255`**: The minor number is used by the kernel to identify a specific device among those that the driver manages. Minor number 255 is usually the control device for NVIDIA.


:one::two: Edit the Jellyfin LXC conf file **/etc/pve/lxc/(ID).conf** and add these lines. The first lxc.group corresponds to the first lxc.mount etc. 
The numbers must match the order and each /dev/nvidia* from last step:
> lxc.cgroup.devices.allow: c 195:0 rw  
lxc.cgroup.devices.allow: c 195:255 rw  
lxc.cgroup.devices.allow: c 195:254 rw  
lxc.cgroup.devices.allow: c 236:0 rw  
lxc.cgroup.devices.allow: c 236:1 rw  
lxc.cgroup.devices.allow: c 10:144 rw  
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file  
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file  
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file  
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file  
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file  
lxc.mount.entry: /dev/nvram dev/nvram none bind,optional,create=file  

### On Jellyfin LXC

:one: Start the Jellyfin LXC container.
:two: Download and Install the same Nvidia driver but with ``--no-kernel-module`` this time:
> cd /opt/nvidia
> ./NVIDIA_DRIVER_FILE --no-kernel-module

:three: Reboot
> reboot
> 
:four: Check that the driver is running fine:
![enter image description here](https://i.imgur.com/okgh4zS.png)

:five: Log into Jellyfin web page and enable the hardware acceleration.
Choose Nvidia NVENC.
Check on the [Nvidia Matrix](https://developer.nvidia.com/video-encode-and-decode-gpu-support-matrix-new) what tick you should tick/not tick.  
:six: We will now check that the transcoding is working.  
Play a 4K movie for example, if the movie is playing that's a good news.  
Confirm it by clicking on the wheel then Playback info  
![enter image description here](https://i.imgur.com/UkS617U.jpeg)

:six: Last step to confirm the transcoding is using the GPU. On promox runs this command:
![enter image description here](https://i.imgur.com/I6XWs4Q.png)
:seven: Congrats the GPU passthrough is now working 💪
