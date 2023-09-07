## The problem, actually a feature: [_br_netfilter_](http://ebtables.netfilter.org/documentation/bridge-nf.html)

The explanation is that the [bridge netfilter code](http://ebtables.netfilter.org/documentation/bridge-nf.html) is [](https://github.com/moby/libnetwork/blob/64b7a4574d1426139437d20e81c0b6d391130ec8/drivers/bridge/bridge.go#L383) for internal container isolation: intended among other usages for stateful bridge firewalling or for leveraging _iptables_' matches and targets from bridge path without having to (or being able to) duplicate them all in _ebtables_. Quite disregarding network layering, the ethernet bridge code, at network layer 2, now makes upcalls to _iptables_ working at IP level, ie network layer 3. It can be enabled only globally before [](https://kernelnewbies.org/Linux_5.3#Networking-"Support-for-pernet-sysctl-in-br_netfilter") (but Docker doesn't handle the new kernel 5.3 features): either for host and every containers, or for none. Once understood what's going and knowing what to look for, adapted choices can be made.

The netfilter project describes the various [`ebtables`/`iptables` interactions](http://ebtables.netfilter.org/br_fw_ia/br_fw_ia.html) when _br_netfilter_ is enabled. Especially of interest is the [](http://ebtables.netfilter.org/br_fw_ia/br_fw_ia.html#section7) explaining why some rules without apparent effect are sometimes needed to avoid unintended effects from the bridge path, like using:

> ```
> iptables -t nat -A POSTROUTING -s 172.16.1.0/24 -d 172.16.1.0/24 -j ACCEPT
> iptables -t nat -A POSTROUTING -s 172.16.1.0/24 -j MASQUERADE
> ```

to avoid two systems on the same LAN to be NATed by... the bridge (see example below).

You have a few choices to avoid your problem, but the choice you took is probably the best if you don't want to know all the details nor verify if some iptables rules (sometimes hidden in other namespaces) would be disrupted:

- permanently prevent the _br_netfilter_ module to be loaded. Usually `blacklist` isn't enough, `install` must be used. This is a choice prone to issues for applications relying on _br_netfilter_: obviously Docker, Kubernetes, ...
    
    ```
      echo install br_netfilter /bin/true > /etc/modprobe.d/disable-br-netfilter.conf
    ```
    
- Have the module loaded, but disable its effects: same results with regard to Docker. For _iptables_' effects that is:
    
    ```
      sysctl -w net.bridge.bridge-nf-call-iptables=0
    ```
    

If putting this at startup, the module should be loaded first or this toggle won't exist yet.

These two previous choices will for sure disrupt _iptables_ match [](https://manpages.debian.org/iptables-extensions.8#physdev): The _xt_physdev_ module when itself loaded, auto-loads the _br_netfilter_ module (this would happen even if a rule added from a container triggered the loading). Now _br_netfilter_ won't be loaded, `-m physdev` will probably never match.

- Work around br_netfilter's effect when needed, like OP: add those apparent no-op rules in various chains (PREROUTING, FORWARD, POSTROUTING) as described in [](http://ebtables.netfilter.org/br_fw_ia/br_fw_ia.html#section7). For example:
    
    ```
      iptables -t nat -A POSTROUTING -s 172.18.0.0/16 -d 172.18.0.0/16 -j ACCEPT
    
      iptables -A FORWARD -i br0 -o br0 -j ACCEPT
    ```
    

Those rules should never match because traffic in the same IP LAN is not routed, except for some rare DNAT setups. But thanks to _br_netfilter_ they do match, because they are first called for _switched_ frames ("upgraded" to IP packets) traversing the _bridge_. Then they are called again for _routed_ packets traversing the _router_ to an unrelated interface (but won't match then).

- Don't put an IP on the bridge: put that IP on one end of a `veth` interface with its other end on the bridge: this should ensure that the bridge won't interact with routing, but that's not what are doing most container/VM common products.
    
- You can even hide the bridge in its own isolated network namespace (that would only be helpful if wanting to isolate from other _ebtables_ rules this time).
    
- Switch everything to [_nftables_](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page) which among stated goals will avoid these [bridge interaction issues](https://www.netdevconf.org/1.1/proceedings/papers/Bridge-filter-with-nftables.pdf). For now the bridge firewalling has no stateful support available, it's still [WIP](https://patchwork.ozlabs.org/project/netfilter-devel/list/?series=102903&state=*) but is promised to be cleaner when available, because there won't be any "upcall".
    

You should search what triggers the loading of _br_netfilter_ (eg: `-m physdev`) and see if you can avoid it or not, to choose how to proceed.

## Minimal Docker integration

When the breakage happens in the host initial network namespace where Docker is running rather than in a new (eg: container) network namespace, OP's rule should be added to the [](https://docs.docker.com/network/iptables/#add-iptables-policies-before-dockers-rules) rather than alone because Docker usually inserts its own rules before what can be already found. This could even be added in some network startup script.

Here's an [idempotent](https://en.wikipedia.org/wiki/Idempotence) method for OP's case. This chain is created by Docker if it didn't exist before, so ignoring a failure makes it work wether started before or after Docker. Likewise `-I` is needed because Docker (or some versions of it) might append a dummy `-j RETURN` rule to `DOCKER-USER`, so `-I` makes it work whether started before or after Docker.

```
iptables -N DOCKER-USER 2>/dev/null || true
iptables -C DOCKER-USER -i br0 -o br0 -j ACCEPT >/dev/null 2>&1 || 
    iptables -I DOCKER-USER -i br0 -o br0 -j ACCEPT
```

---

## Example with network namespaces

Let's reproduce some effects using a network namespace. Note that nowhere any _ebtables_ rule will be used. Also note that this example relies on the usual legacy [`iptables`](https://manpages.debian.org/iptables/iptables-legacy.8), not [iptables over nftables](https://manpages.debian.org/iptables/iptables-nft.8) as enabled by default on Debian buster.

Let's reproduce a simple case similar with many container usages: a router 192.168.0.1/192.0.2.100 doing NAT with two hosts behind: 192.168.0.101 and 192.168.0.102, linked with a bridge on the router. The two hosts can communicate directly on the same LAN, through the bridge.

```
#!/bin/sh

for ns in host1 host2 router; do
    ip netns del $ns 2>/dev/null || :
    ip netns add $ns
    ip -n $ns link set lo up
done

ip netns exec router sysctl -q -w net.ipv4.conf.default.forwarding=1

ip -n router link add bridge0 type bridge
ip -n router link set bridge0 up
ip -n router address add 192.168.0.1/24 dev bridge0

for i in 1 2; do
    ip -n host$i link add eth0 type veth peer netns router port$i
    ip -n host$i link set eth0 up
    ip -n host$i address add 192.168.0.10$i/24 dev eth0
    ip -n host$i route add default via 192.168.0.1
    ip -n router link set port$i up master bridge0
done

#to mimic a standard NAT router, iptables rule voluntarily made as it is to show the last "effect"
ip -n router link add name eth0 type dummy
ip -n router link set eth0 up
ip -n router address add 192.0.2.100/24 dev eth0
ip -n router route add default via 192.0.2.1
ip netns exec router iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j MASQUERADE
```

Let's load the kernel module _br_netfilter_ (to be sure it won't be later) and disable its effects with the (not-per-namespace) toggle _bridge-nf-call-iptables_, available only in initial namespace:

```
modprobe br_netfilter
sysctl -w net.bridge.bridge-nf-call-iptables=0
```

Warning: again, this can disrupt _iptables_ rules like [](https://manpages.debian.org/iptables-extensions.8#physdev) anywhere on the host or in containers which rely on _br_netfilter_ loaded and enabled.

Let's add some icmp ping traffic counters.

```
ip netns exec router iptables -A FORWARD -p icmp --icmp-type echo-request
ip netns exec router iptables -A FORWARD -p icmp --icmp-type echo-reply
```

Let's ping:

```
# ip netns exec host1 ping -n -c2 192.168.0.102
PING 192.168.0.102 (192.168.0.102) 56(84) bytes of data.
64 bytes from 192.168.0.102: icmp_seq=1 ttl=64 time=0.047 ms
64 bytes from 192.168.0.102: icmp_seq=2 ttl=64 time=0.058 ms

--- 192.168.0.102 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1017ms
rtt min/avg/max/mdev = 0.047/0.052/0.058/0.009 ms
```

The counters won't match:

```
# ip netns exec router iptables -v -S FORWARD
-P FORWARD ACCEPT -c 0 0
-A FORWARD -p icmp -m icmp --icmp-type 8 -c 0 0
-A FORWARD -p icmp -m icmp --icmp-type 0 -c 0 0
```

Let's enable _bridge-nf-call-iptables_ and ping again:

```
# sysctl -w net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-iptables = 1
# ip netns exec host1 ping -n -c2 192.168.0.102
PING 192.168.0.102 (192.168.0.102) 56(84) bytes of data.
64 bytes from 192.168.0.102: icmp_seq=1 ttl=64 time=0.094 ms
64 bytes from 192.168.0.102: icmp_seq=2 ttl=64 time=0.163 ms

--- 192.168.0.102 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1006ms
rtt min/avg/max/mdev = 0.094/0.128/0.163/0.036 ms
```

This time switched packets got a match in iptables' filter/FORWARD chain:

```
# ip netns exec router iptables -v -S FORWARD
-P FORWARD ACCEPT -c 4 336
-A FORWARD -p icmp -m icmp --icmp-type 8 -c 2 168
-A FORWARD -p icmp -m icmp --icmp-type 0 -c 2 168
```

Let's put a DROP policy (which zeroes the default counters) and try again:

```
# ip netns exec host1 ping -n -c2 192.168.0.102
PING 192.168.0.102 (192.168.0.102) 56(84) bytes of data.

--- 192.168.0.102 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1008ms

# ip netns exec router iptables -v -S FORWARD
-P FORWARD DROP -c 2 168
-A FORWARD -p icmp -m icmp --icmp-type 8 -c 4 336
-A FORWARD -p icmp -m icmp --icmp-type 0 -c 2 168
```

The bridge code filtered the switched frames/packets via iptables. Let's add the bypass rule (which will zero again the default counters) like in OP and try again:

```
# ip netns exec router iptables -A FORWARD -i bridge0 -o bridge0 -j ACCEPT
# ip netns exec host1 ping -n -c2 192.168.0.102
PING 192.168.0.102 (192.168.0.102) 56(84) bytes of data.
64 bytes from 192.168.0.102: icmp_seq=1 ttl=64 time=0.132 ms
64 bytes from 192.168.0.102: icmp_seq=2 ttl=64 time=0.123 ms

--- 192.168.0.102 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1024ms
rtt min/avg/max/mdev = 0.123/0.127/0.132/0.012 ms

# ip netns exec router iptables -v -S FORWARD
-P FORWARD DROP -c 0 0
-A FORWARD -p icmp -m icmp --icmp-type 8 -c 6 504
-A FORWARD -p icmp -m icmp --icmp-type 0 -c 4 336
-A FORWARD -i bridge0 -o bridge0 -c 4 336 -j ACCEPT
```

Let's see what is now actually received on host2 during a ping from host1:

```
# ip netns exec host2 tcpdump -l -n -s0 -i eth0 -p icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
02:16:11.068795 IP 192.168.0.1 > 192.168.0.102: ICMP echo request, id 9496, seq 1, length 64
02:16:11.068817 IP 192.168.0.102 > 192.168.0.1: ICMP echo reply, id 9496, seq 1, length 64
02:16:12.088002 IP 192.168.0.1 > 192.168.0.102: ICMP echo request, id 9496, seq 2, length 64
02:16:12.088063 IP 192.168.0.102 > 192.168.0.1: ICMP echo reply, id 9496, seq 2, length 64
```

... instead of source 192.168.0.101. The MASQUERADE rule was also called from the bridge path. To avoid this either add (as explained in [](http://ebtables.netfilter.org/br_fw_ia/br_fw_ia.html#section7)'s example) an exception rule before, or state a non-bridge outgoing interface, if possible at all (now it's available you can even use `-m physdev` if it has to be a bridge...).