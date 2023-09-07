Ubuntu Server 22.04-1 LTS

> <u>Status as of 28.03.2023 16:12</u>
> Nextcloud trusted domains -> add 10.130.5.117

After downloading the latest version of cockpit-navigator:
![Pasted image 20230215234800](IMG/Pasted-image-20230215234800.png)
following workaround
```bash
cd /etc/netplan
sudo nano sudo nano 00-installer-config.yaml 
```
add following line to the bottom of the file: renderer: NetworkManager
![Pasted image 20230215234719](IMG/Pasted-image-20230215234719.png)
Following the official Ubunut website for how to configure Netplan.

this didnt work with the error: invalid yaml: inconsistent indentation
turns out the number of spaces used makes a difference, in this case 
![Pasted image 20230216002613](IMG/Pasted-image-20230216002613.png)
version, renderer and ethernets are properties of the interface and HAVE to be at the same "height"

anyways, it works now

WoL
```bash
sudo ethtool -s enp0s31f6 wol g
sudo ethtool enp0s31f6 #used to check
```

WoL is not working - troubleshooting needed

logged into Nextcloud snap and set up admin
Nextcloud 
admin account: dan pwd: Alh2023ambra3C

create raid array
```bash
sudo mdadm --create --verbose /dev/md127 --level=1 --raid-devices=2 /dev/sda /dev/sdb
```
waiting for disks to sync

Move datedirectory to /media/vault127/nxtcld/data
https://github.com/nextcloud-snap/nextcloud-snap/wiki/Change-data-directory-to-use-another-disk-partition


```bash
sudo snap stop nextcloud

sudo mkdir -p /media/vault127/nxtcld/data
sudo chown -R root:root /media/vault127/nxtcld/data

sudo nano /var/snap/nextcloud/current/nextcloud/config/config.php

# datadirectory => '/media/vault127/nxtcld/data'

sudo mv /var/snap/nextcloud/common/nextcloud/data /media/vault127/nxtcld/data

sudo snap start nextcloud
```

after the addition of a bridge to run the vms on Nextcloud is showing following interface when you try to reach it per 10.130.5.117
![Pasted image 20230328151645](IMG/Pasted-image-20230328151645.png)
the new ip address needs to be added to the trusted domains 
```bash
sudo nano /var/snap/nextcloud/current/nextcloud/config/config.php
```

paddy unnerving2023
rei faithful2023
nano Hubris2023

Virtual Machines
```bash
sudo apt-get install cockpit-machines
```

#### OCC
``` bash
# scan filesystem for all files
sudo nextcloud.occ files:scan --all -vvv
```

### Configure aliases as shortcuts
```bash
nano ~/.bashrc
# scroll down and enter alias in this format
alias k = 'kubectl'
```