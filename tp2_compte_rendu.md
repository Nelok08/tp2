# TP2 : Routage, DHCP et DNS
 Sommaire

- [TP2 : Routage, DHCP et DNS](#tp2--routage-dhcp-et-dns)
- [I. Routage](#i-routage)
- [II. Serveur DHCP](#ii-serveur-dhcp)
- [III. ARP](#iii-arp)
  - [1. Les tables ARP](#1-les-tables-arp)
  - [2. ARP poisoning](#2-arp-poisoning)


# I. Routage

**Topo 1**

➜ **Tableau d'adressage**

ip a (machine node):
```bash
[user@node1tp2 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:50:ad:df brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.1/24 brd 10.2.1.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe50:addf/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:a4:17:1f brd ff:ff:ff:ff:ff:ff
    inet 192.168.151.10/24 brd 192.168.151.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea4:171f/64 scope link
       valid_lft forever preferred_lft forever
```

ip a (machine routeur):
```bash
[user@router ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:f9:58:ca brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.254/24 brd 10.2.1.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fef9:58ca/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d4:76:0f brd ff:ff:ff:ff:ff:ff
    inet 192.168.151.3/24 brd 192.168.151.255 scope global dynamic noprefixroute enp0s8
       valid_lft 350sec preferred_lft 350sec
    inet6 fe80::a00:27ff:fed4:760f/64 scope link
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:82:1c:b5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.181/24 brd 192.168.122.255 scope global dynamic noprefixroute enp0s9
       valid_lft 3352sec preferred_lft 3352sec
    inet6 fe80::df91:cbe5:3195:42c8/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

☀️ **Configuration de `router.tp2.efrei`**

```bash
[user@router ~]$ ping google.com
PING google.com (142.250.179.78) 56(84) bytes of data.
64 bytes from par21s19-in-f14.1e100.net (142.250.179.78): icmp_seq=1 ttl=112 time=67.5 ms
64 bytes from par21s19-in-f14.1e100.net (142.250.179.78): icmp_seq=2 ttl=112 time=29.5 ms
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 29.509/48.529/67.549/19.020 ms

[user@router ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:f9:58:ca brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.254/24 brd 10.2.1.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fef9:58ca/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d4:76:0f brd ff:ff:ff:ff:ff:ff
    inet 192.168.151.3/24 brd 192.168.151.255 scope global dynamic noprefixroute enp0s8
       valid_lft 468sec preferred_lft 468sec
    inet6 fe80::a00:27ff:fed4:760f/64 scope link
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:82:1c:b5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.181/24 brd 192.168.122.255 scope global dynamic noprefixroute enp0s9
       valid_lft 3468sec preferred_lft 3468sec
    inet6 fe80::df91:cbe5:3195:42c8/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

☀️ **Configuration de `node1.tp2.efrei`**

**prouvez avec une commande ping que node1.tp2.efrei peut joindre router.tp2.efrei :**
```bash
[user@node1tp2 ~]$ ping 10.2.1.254
PING 10.2.1.254 (10.2.1.254) 56(84) bytes of data.
64 bytes from 10.2.1.254: icmp_seq=1 ttl=64 time=2.80 ms
64 bytes from 10.2.1.254: icmp_seq=2 ttl=64 time=1.35 ms
64 bytes from 10.2.1.254: icmp_seq=3 ttl=64 time=1.44 ms
^C
--- 10.2.1.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.347/1.861/2.797/0.662 ms
```
**prouvez que vous avez un accès internet depuis node1.tp2.efrei désormais, avec une commande ping :**

```bash
[user@node1tp2 ~]$ ping google.com
PING google.com (142.250.201.174) 56(84) bytes of data.
64 bytes from par21s23-in-f14.1e100.net (142.250.201.174): icmp_seq=1 ttl=113 time=30.8 ms
64 bytes from par21s23-in-f14.1e100.net (142.250.201.174): icmp_seq=2 ttl=113 time=24.4 ms
64 bytes from par21s23-in-f14.1e100.net (142.250.201.174): icmp_seq=3 ttl=113 time=26.2 ms
64 bytes from par21s23-in-f14.1e100.net (142.250.201.174): icmp_seq=4 ttl=113 time=22.1 ms
64 bytes from par21s23-in-f14.1e100.net (142.250.201.174): icmp_seq=5 ttl=113 time=25.9 ms
^C
--- google.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4411ms
rtt min/avg/max/mdev = 22.090/25.885/30.812/2.858 ms

```



- ajoutez une route par défaut qui passe par `router.tp2.efrei`
```bash
NAME=enp0s3
DEVICE=enp0s3

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.2.1.1
NETMASK=255.255.255.0

GATEWAY=10.2.1.254
DNS1=1.1.1.1
```

- utilisez une commande `traceroute` pour prouver que vos paquets passent bien par `router.tp2.efrei` avant de sortir vers internet

```bash
[user@node1tp2 ~]$ traceroute google.com
traceroute to google.com (142.250.179.110), 30 hops max, 60 byte packets
 1  _gateway (10.2.1.254)  1.958 ms  1.820 ms  1.700 ms
 2  192.168.122.1 (192.168.122.1)  9.644 ms  9.556 ms  9.309 ms
 3  10.0.3.2 (10.0.3.2)  9.221 ms  9.379 ms  9.215 ms
```

# II. Serveur DHCP

**__Topo 2__**

➜ **Tableau d'adressage**

☀️ **Test du DHCP** sur `node1.tp2.efrei`

➜ config du dhcp
```bash

config du dhcp : 

#create new
#specify domain name
option domain-name     "srv.world";

#specify DNS server's hostname or IP address
option domain-name-servers     1.1.1.1;

#default lease time
default-lease-time 600;

#max lease time
max-lease-time 7200;

#this DHCP server to be declared valid
authoritative;

#specify network address and subnetmask
subnet 10.2.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.2.1.200 10.2.1.253;
    # specify broadcast address
    option broadcast-address 10.2.1.255;
    # specify gateway
    option routers 10.2.1.254;
```
➜ test du dhcp, on obtient une nouvelle ip 

```bash

[user@node1tp2 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:50:ad:df brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.200/24 brd 10.2.1.255 scope global dynamic noprefixroute enp0s3
       valid_lft 338sec preferred_lft 338sec
    inet6 fe80::a00:27ff:fe50:addf/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:a4:17:1f brd ff:ff:ff:ff:ff:ff
    inet 192.168.151.10/24 brd 192.168.151.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea4:171f/64 scope link
       valid_lft forever preferred_lft forever
```

➜ ping google.com

```bash

[user@node1tp2 ~]$ ping google.com
PING google.com (142.250.179.78) 56(84) bytes of data.
64 bytes from par21s19-in-f14.1e100.net (142.250.179.78): icmp_seq=1 ttl=111 time=17.5 ms
64 bytes from par21s19-in-f14.1e100.net (142.250.179.78): icmp_seq=2 ttl=111 time=19.6 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 17.478/18.545/19.612/1.067 ms
```

🌟 **BONUS**

```bash
NAME=enp0s3
DEVICE=enp0s3

BOOTPROTO=dhcp
ONBOOT=yes
DNS1=1.1.1.1
```

# III. ARP

## 1. Les tables ARP

```bash

[user@router ~]$ ip neigh show
10.2.1.253 dev enp0s3 lladdr 08:00:27:aa:60:ae REACHABLE
10.2.1.200 dev enp0s3 lladdr 08:00:27:50:ad:df STALE
192.168.151.1 dev enp0s8 lladdr 08:00:27:d6:a2:83 STALE
192.168.151.2 dev enp0s8 lladdr 0a:00:27:00:00:0d DELAY
192.168.122.1 dev enp0s9 lladdr 52:54:00:3f:74:8f REACHABLE
```

## 2. ARP poisoning

☀️ **Exécuter un simple ARP poisoning**
```
Changer ip mac de enp0s3: 
sudo ip neigh change 10.2.1.200 lladdr aa:bt:cc:dy:me:ff dev enp0s3

```



