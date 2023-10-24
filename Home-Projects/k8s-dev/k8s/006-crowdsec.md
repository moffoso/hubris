### installing crowdsec
add repo
```bash
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | sudo bash
```

install crowdsec
```bash
sudo apt install crowdsec
```

install firewall-iptables bouncer
```bash
sudo apt install crowdsec-firewall-bouncer-iptables
```

add security engine to console
```bash
sudo cscli console enroll [join code]
```

repeated the process for worker

### configuring ufw
if not installed
```bash
sudo apt install ufw
```

open necessary ports
```bash
sudo ufw allow 22/tcp
sudo ufw allow 9090/tcp
```

configure ufw
```bash
sudo ufw allow 6443/tcp
sudo ufw allow 2379:2380/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 10259/tcp
sudo ufw allow 10257/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 30000:32767/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

allow cloudflare ip's
```bash
sudo ufw allow from 173.245.48.0/20 to any port 80,443 proto tcp
sudo ufw allow from 103.21.244.0/22 to any port 80,443 proto tcp
sudo ufw allow from 103.22.200.0/22 to any port 80,443 proto tcp
sudo ufw allow from 103.31.4.0/22 to any port 80,443 proto tcp
sudo ufw allow from 141.101.64.0/18 to any port 80,443 proto tcp
sudo ufw allow from 108.162.192.0/18 to any port 80,443 proto tcp
sudo ufw allow from 190.93.240.0/20 to any port 80,443 proto tcp
sudo ufw allow from 188.114.96.0/20 to any port 80,443 proto tcp
sudo ufw allow from 197.234.240.0/22 to any port 80,443 proto tcp
sudo ufw allow from 198.41.128.0/17 to any port 80,443 proto tcp
sudo ufw allow from 162.158.0.0/15 to any port 80,443 proto tcp
sudo ufw allow from 104.16.0.0/13 to any port 80,443 proto tcp
sudo ufw allow from 104.24.0.0/14 to any port 80,443 proto tcp
sudo ufw allow from 172.64.0.0/13 to any port 80,443 proto tcp
sudo ufw allow from 131.0.72.0/22 to any port 80,443 proto tcp
sudo ufw allow from 2400:cb00::/32 to any port 80,443 proto tcp
sudo ufw allow from 2606:4700::/32 to any port 80,443 proto tcp
sudo ufw allow from 2803:f800::/32 to any port 80,443 proto tcp
sudo ufw allow from 2405:b500::/32 to any port 80,443 proto tcp
sudo ufw allow from 2405:8100::/32 to any port 80,443 proto tcp
sudo ufw allow from 2a06:98c0::/29 to any port 80,443 proto tcp
sudo ufw allow from 2c0f:f248::/32 to any port 80,443 proto tcp
```

### Troubleshooting ufw
testpage times out when trying to reach it per test.moff.ch

1. Check ufw logs with dmesg
```bash
sudo dmesg | grep '\[UFW'
# There were multiple logs with the following entry
# [112422.703481] [UFW BLOCK] IN=enp1s0 OUT= MAC=52:54:00:ca:5b:01:52:54:00:33:37:35:08:00 SRC=10.130.5.130 DST=10.130.5.131 LEN=110 TOS=0x00 PREC=0x00 TTL=64 ID=27126 PROTO=UDP SPT=54140 DPT=8472 LEN=90 
```

2. Understanding the log
	1. SRC is the source port of the package
	2. DST is where the package is supposed to go
	3. PROTO is the protocol used
	4. SPT is the port it's being sent from
	5. DPT is the port it's being sent to

the solution in this case is to allow 8472/udp on worker@10.130.5.131
```bash
sudo ufw allow 8472/udp
```

