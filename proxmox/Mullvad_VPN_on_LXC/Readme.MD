
## Quick tutorial to setup a Mullvad VPN connection on a LXC  ✍️

### On your proxmox node

:one: Install the mullvad app following the instructions from their [website](https://mullvad.net/en/help/install-mullvad-app-linux)

:two: Add these permissions in the LXC conf file in /etc/pve/lxc/[LXC_ID].conf
> lxc.cgroup2.devices.allow: c 10:200 rwm  
> lxc.mount.entry: /dev/net dev/net none bind,create=dir


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

## If your mullvad account expired

### On your mullvad account  
:one: [Generate a new configuration file](https://mullvad.net/en/account/wireguard-config)

:two: Generate a new key and select locations

:three: Download zip archive

:four: Unzip archive


### On your LXC
:one: Copy paste and replace the .conf files into your lxc /etc/wireguard

:two: Set your mullvad account number
> mullvad account login 1234....

Automate the mullvad connection with a job in /etc/systemd/system:
> cat wg-quick-abc.service
[Unit]
Description=WireGuard VPN abc
After=network.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/bin/sleep 60
ExecStartPost=/usr/bin/wg-quick up /etc/wireguard/abc.conf
ExecStop=/usr/bin/wg-quick down /etc/wireguard/abc.conf

[Install]
WantedBy=multi-user.target


