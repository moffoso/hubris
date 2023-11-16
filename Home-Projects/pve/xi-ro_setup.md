### sources.list
Per default proxmox contains the enterprise repositories which can only be used with an enterprise license. Running `apt update && apt upgrade` returns a 401 Unauthorized. The solution is to use the No-Subscription repository.
1. navigate to `/etc/apt/sources.list.d` and comment out the content of `cepth.list` and `pve-enterprise.list`
2. navigate to the file `/etc/apt/sources.list` and add the follwing lines
```bash
# Proxmox VE pve-no-subscription repository provided by proxmox.com
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
```
3. Now running `apt update && apt upgrade` will work

### mounting raid from vault-server
The old RAID 1 from the vault-server contains data I would like to keep, check the disks
```bash
lsblk
NAME               MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda                  8:0    0   1.8T  0 disk  
sdb                  8:16   0 931.5G  0 disk  
├─sdb1               8:17   0  1007K  0 part  
├─sdb2               8:18   0     1G  0 part  
└─sdb3               8:19   0 930.5G  0 part  
  ├─pve-swap       253:0    0     8G  0 lvm   [SWAP]
  ├─pve-root       253:1    0    96G  0 lvm   /
  ├─pve-data_tmeta 253:2    0   8.1G  0 lvm   
  │ └─pve-data     253:4    0 794.3G  0 lvm   
  └─pve-data_tdata 253:3    0 794.3G  0 lvm   
    └─pve-data     253:4    0 794.3G  0 lvm   
sdc                  8:32   0   1.8T  0 disk  
```

the devices /dev/sda and /dev/sdc are the the old linux raid members, using mdadm we can reassemble the array
```bash
mdadm --assemble --scan --verbose /dev/md127 /dev/sda /dev/sdc 
```

To make sure that the device md127 is mounted on boot we add to `/etc/fstab`
```bash
# mounting device md127
/dev/md127 /media/vault127 ext4 defaults 0 0
```

Alternative to using the name /dev/md127 use `blkid` to find the UUID of the device and replace `/dev/dm127` in fstab with the UUID
```bash
blkid /dev/md127
/dev/md127: UUID="85a8c387-0158-4942-9d62-fd08765f5132" BLOCK_SIZE="4096" TYPE="ext4"
```

### add storage device
Now that we added the raid, we will use it to store disk images, backups, etc.
In pve navigate to Datacenter -> storage and press Add. Select Directory
- ID -> Whatever you want the storage to be called
- Directory -> path to the directory
- Content -> Here we select basically everything
It will create following directories: `dump images snippets template`
### create user
To manage users in pve's GUI they first need to exist in the linux fs
```bash
useradd [username]
```
This will create a blank user without a home directory, set it up as needed in pve. In my case I created a home directory for the user with `mkhomedir_helper username` and then added the user to the sudo group with `usermod -aG sudo [username]`

### ssh setup
Pve has ssh preconfigured on the root user, I'll be changing it here with basic hardening practises. First lets create a config file in `/etc/ssh/ssd_config.d` called hardening.conf
```bash
PasswordAuthentication no
PermitRootLogin no
ChallengeResponseAuthentication no
UsePAM no
```
add your ssh key to the host
```bash
ssh-copy-id [username]@[ip-address]
```


