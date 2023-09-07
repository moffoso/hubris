### Check interface details with ethool
Interface names can be inspected with following command
```bash
ip a
```

Once the name of the interface is known we can check if WoL is supported. Replace interface with the interface name
```bash
sudo ethtool interface
```

Example output 
```bash
dan@vault:~$ sudo ethtool enp0s31f6
Settings for enp0s31f6:
        Supported ports: [ TP ]
        Supported link modes:   10baseT/Half 10baseT/Full
                                100baseT/Half 100baseT/Full
                                1000baseT/Full
        Supported pause frame use: No

		Supports auto-negotiation: Yes
        Supported FEC modes: Not reported
        Advertised link modes:  10baseT/Half 10baseT/Full
                                100baseT/Half 100baseT/Full
                                1000baseT/Full
        Advertised pause frame use: No
        Advertised auto-negotiation: Yes
        Advertised FEC modes: Not reported
        Speed: 1000Mb/s
        Duplex: Full
        Auto-negotiation: on
        Port: Twisted Pair
        PHYAD: 1
        Transceiver: internal
        MDI-X: on (auto)
        Supports Wake-on: pumbg
        Wake-on: d
        Current message level: 0x00000007 (7)
                               drv probe link
        Link detected: yes
```

Notice following lines
```bash
        Supports Wake-on: pumbg
        Wake-on: d
```

Supports Wake-on: pumbg

p   Wake on PHY activity
u   Wake on unicast messages
m   Wake on multicast messages
b   Wake on broadcast messages
a   Wake on ARP
g   Wake on MagicPacket™
s   Enable SecureOn™ password for MagicPacket™
d   Disable (wake on nothing). This option clears all previous options.

#### Modify interface settings with ethtool
Change Wake-on setting to MagicPacket with following command
```bash
sudo ethtool -s interface wol g
```
