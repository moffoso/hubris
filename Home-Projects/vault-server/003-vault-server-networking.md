### creating bridge0
00-isntaller-config.yaml in /etc/netplan
```bash
# This is the network config written by 'subiquity'
network:
 version: 2
 renderer: NetworkManager
 ethernets:
    enp0s31f6:
      dhcp4: true
 bridges:
   bridge0:
     dhcp4: yes
     interfaces:
       - enp0s31f6  
```

apply changes with
```bash
sudo netplan apply
```

### WoL
First I need to enable WoL in the bios. Here is a step by step example for H170m-plus Motherboard.
1. Press Delete to enter BIOS during the boot process
2. In the BIOS follow this path Advanced > Advanced > Advanced\APM Configuration > Power On By PCI-E/PCI
3. Enable

find where ethertool is located
```bash
which ethtool
```
this delivers following result: /usr/sbin/ethtool

create a service in /etc/systemd
```bash
sudo nano wol.service
```

```bash
[Unit]
Description=Enable Wake on Lan

[Service]
Type=oneshot
ExecStart = /sbin/ethtool -s enp0s31f6 wol g

[Install]
WantedBy=basic.target
```

enable the created service
```bash 
sudo systemctl daemon-reload
sudo systemctl enable wol.service
```

check the status of the service
```bash
sudo systemctl status wol
```
