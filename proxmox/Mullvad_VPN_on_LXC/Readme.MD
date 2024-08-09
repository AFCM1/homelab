
## Quick tutorial to setup a Mullvad VPN connection on a LXC  ✍️

### On your proxmox node

:one: Install the mullvad app following the instructions from their [website](https://mullvad.net/en/help/install-mullvad-app-linux)

:two: Add these permissions in the LXC conf file in /etc/pve/lxc/[LXC_ID].conf
> lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir


### On your Linux container

:three: Start your container

:four: Install the mullvad app following the instructions from their [website](https://mullvad.net/en/help/install-mullvad-app-linux)

:five: Set your mullvad account number
> mullvad account login 1234....

:six: Set the server location
> mullvad relay set location se mma

se is the country code  
mma is the city name

:seven: Start the connection
> mullvad connect

Other useful commands can be find on their [website](https://mullvad.net/en/help/how-use-mullvad-cli)

:eight: Check that you get the VPN server IP
> curl -s checkip.dyndns.org|sed -e 's/.\*Current IP Address: //' -e 's/<.*$//'
