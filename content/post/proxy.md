---
title: "SOCKS5 proxy server in 2 minutes"
date: 2018-04-18T09:00:16+03:00
---

In the [previous post](/post/vpn) we deployed VPN server on GCP instance, now we can easily
run SOCKS5 proxy server on the same instance, for this we only need to install server with command:

{{< highlight bash >}}
sudo apt-get install dante-server
{{< /highlight >}}

and add configuration to file `/etc/danted.conf`

```
logoutput: stderr

internal: eth0 port = 7777
external: eth0
socksmethod: username
user.privileged: root
user.unprivileged: nobody

client pass {
       from: 0.0.0.0/0 port 1-65535 to: 0.0.0.0/0
       log: error
       socksmethod: username
}



socks pass {
       from: 0.0.0.0/0 to: 0.0.0.0/0
       command: bind connect udpassociate
       log: error
       socksmethod: username
}

socks pass {
       from: 0.0.0.0/0 to: 0.0.0.0/0
       command: bindreply udpreply
       log: error
}
```

as always you can change port and maybe you need to change internal and external network interface name,
you can check it with command `ifconfig`, for my GCP instance it's `ens4`

```
aborilov@instance-1:~$ ifconfig
ens4: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1460
        inet 10.128.0.2  netmask 255.255.255.255  broadcast 10.128.0.2
        inet6 fe80::4001:aff:fe80:2  prefixlen 64  scopeid 0x20<link>
        ether 42:01:0a:80:00:02  txqueuelen 1000  (Ethernet)
        RX packets 825689  bytes 820274156 (820.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 922109  bytes 804120538 (804.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 1062089  bytes 180611621 (180.6 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1062089  bytes 180611621 (180.6 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 192.168.246.1  netmask 255.255.255.0  destination 192.168.246.1
        inet6 fe80::9f6a:4697:dc3f:b9db  prefixlen 64  scopeid 0x20<link>
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 100  (UNSPEC)
        RX packets 302281  bytes 37236386 (37.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 622091  bytes 740905694 (740.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

you also need to add `Firewall rule` for you proxy port as we did in the [previous post](/post/vpn)

Now we just need to add user:

```
sudo useradd -m sockduser && sudo passwd sockduser
```
and start the service

```
sudo systemctl enable danted.service
sudo systemctl start danted.service
```

and now you can use this proxy, for example, from telegram :)
