# TP3 : Progressons vers le réseau d'infrastructure
## I. (mini)Architecture réseau
1. Adressage

-> Tableau de réseaux
| Nom du réseau | Adresse du réseau |     Masque      | Nombre de clients possibles | Adresse passerelle | Adresse broadcast |
|:-------------:|:-----------------:|:---------------:|:---------------------------:|:------------------:|:-----------------:|
|    client1    |    10.3.0.128     | 255.255.255.192 |             61              |     10.3.0.190     |    10.3.0.191     |
|    server1    |     10.3.0.0      | 255.255.255.128 |             125             |     10.3.0.126     |    10.3.0.127     |
|    server2    |    10.3.0.192     | 255.255.255.240 |             13              |     10.3.0.206     |    10.3.0.207     |

-> Tableau de machines
| Nom machine        | Adresse de la machine dans le réseau client1 | Adresse de la machine dans le réseau server1 | Adresse de la machine dans le réseau server2 | Adresse de passerelle |
|:------------------ |:--------------------------------------------:|:--------------------------------------------:|:--------------------------------------------:|:---------------------:|
| router.tp3         |                10.3.0.190/26                 |                10.3.0.126/25                 |                10.3.0.206/28                 |       Carte NAT       |
| dhcp.client1.tp3   |                10.3.0.131/26                 |                      /                       |                      /                       |     10.3.0.190/26     |
| marcel.client1.tp3 |                10.3.0.132/26                 |                      /                       |                      /                       |     10.3.0.190/26     |
| dns1.server1.tp3   |                      /                       |                   10.3.0.2/25                   |                      /                       |     10.3.0.126/25     |

2. Routeur

-> Accès internet et configuration de la passerelle
```
[Y@router ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc 
[...]
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:47:90:e0 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 84530sec preferred_lft 84530sec
    inet6 fe80::a00:27ff:fe47:90e0/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:68:7e:24 brd ff:ff:ff:ff:ff:ff
    inet 10.3.0.190/26 brd 10.3.0.191 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe68:7e24/64 scope link
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d0:14:2f brd ff:ff:ff:ff:ff:ff
    inet 10.3.0.126/25 brd 10.3.0.127 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed0:142f/64 scope link
       valid_lft forever preferred_lft forever
5: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:ae:31:a7 brd ff:ff:ff:ff:ff:ff
    inet 10.3.0.206/28 brd 10.3.0.207 scope global noprefixroute enp0s10
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feae:31a7/64 scope link
       valid_lft forever preferred_lft forever
```
-> accès internet
```
[Y@router ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=17.7 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=115 time=17.5 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=115 time=17.10 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=115 time=17.10 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=115 time=17.3 ms
^C
--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 17.257/17.662/17.972/0.291 ms
```
-> Résolution de noms
```
[Y@router ~]$ dig ynov.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16630
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; ANSWER SECTION:
ynov.com.               4107    IN      A       92.243.16.143

;; Query time: 21 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Thu Sep 30 11:11:26 EDT 2021
;; MSG SIZE  rcvd: 53hostname
```
-> Nom de la machine
```
[Y@router ~]$ hostname
router.tp3
[Y@router ~]$ cat /etc/hostname
router.tp3
[Y@router ~]$
```
-> Activation du routage
```
[Y@router ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
success
```


---

## II. Services d'infra
1. Serveur DHCP

-> Machine du serveur DHCP
```
[Y@dhcp ~]$ hostname
dhcp.client1.tp3
```
```
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
option domain-name      "client1.tp3";

default-lease-time 600;

max-lease-time 7200;

authoritative;

subnet 10.3.0.128 netmask 255.255.255.192 {
        range dynamic-bootp 10.3.0.129 10.3.0.191;
        option broadcast-address 10.3.0.191;
        option routers 10.3.0.190;
}
```

-> Machine Marcel
```
[Y@marcel ~]$ hostname
marcel.client1.tp3
```

```
[Y@marcel ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=17.1 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=17.7 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=16.0 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=114 time=16.6 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 16.018/16.858/17.693/0.617 ms
```

```
[Y@marcel ~]$ dig ynov.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43426
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 7b5604fb8bb514bea9c83ac9616c48e0f5a567cb3b3c2f94 (good)
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; ANSWER SECTION:
ynov.com.               10800   IN      A       92.243.16.143

;; Query time: 60 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Sun Oct 17 12:01:36 EDT 2021
;; MSG SIZE  rcvd: 81
```

```
[Y@marcel ~]$ sudo traceroute -i enp0s8 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (10.3.0.190)  0.944 ms  0.900 ms  0.882 ms
 [...]
```

2. Serveur DNS
->  Mettre en place une machine qui fera office de serveur DNS
```
[Y@ds1 ~]$ sudo cat /etc/named.conf
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { any; };
        directory        "/var/named";
        dump-file        "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { any; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */

        recursion no;

        dnssec-enable yes;
        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

zone "server1.tp3" IN {
        type master;
        file "server1.tp3.forward";
        allow-update { none; };
};

zone "server2.tp3" IN {
        type master;
        file "server2.tp3.forward";
        allow-update { none; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

```
[Y@ds1 ~]$ sudo cat /var/named/server1.tp3.forward
$TTL 86400
@   IN  SOA     dns1.server1.tp3. root.server1.tp3. (
         2021101801  ;Serial
         3600        ;Refresh
         1800        ;Retry
         604800      ;Expire
         86400       ;Minimum TTL
)
        ; Set your Name Servers here
@         IN  NS      dns1.server1.tp3.
         ; define Name Server's IP address
@         IN  A       10.3.0.2

dns1    IN A    10.3.0.2
```

```
[Y@ds1 ~]$ sudo named-checkconf
[Y@ds1 ~]$ sudo named-checkzone server1.tp3 /var/named/server1.tp3.forward
zone server1.tp3/IN: loaded serial 2021101801
OK
[Y@ds1 ~]$ sudo named-checkzone server2.tp3 /var/named/server2.tp3.forward
zone server2.tp3/IN: loaded serial 2021101801
OK
```
```
[Y@ds1 ~]$ sudo firewall-cmd --add-service=dns --permanent
Warning: ALREADY_ENABLED: dns
success
```
```
[Y@ds1 ~]$ sudo systemctl enable --now named.service
Created symlink /etc/systemd/system/multi-user.target.wants/named.service → /usr/lib/systemd/system/named.service.
[Y@ds1 ~]$ systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2021-10-18 00:39:05 EDT; 9s ago
```
-> Tester le DNS depuis marcel.client1.tp3
```
[Y@marcel ~]$ dig dns1.server1.tp3 @10.3.0.2

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> dns1.server1.tp3 @10.3.0.2
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17795
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 4fb32b6865d4c458af51fd49616cfbf7b9bcf36e3dfcaafe (good)
;; QUESTION SECTION:
;dns1.server1.tp3.              IN      A

;; ANSWER SECTION:
dns1.server1.tp3.       86400   IN      A       10.3.0.2

;; AUTHORITY SECTION:
server1.tp3.            86400   IN      NS      dns1.server1.tp3.

;; Query time: 3 msec
;; SERVER: 10.3.0.2#53(10.3.0.2)
;; WHEN: Mon Oct 18 00:45:43 EDT 2021
;; MSG SIZE  rcvd: 103
```

-> Configurez l'utilisation du serveur DNS sur TOUS vos noeuds
```
[Y@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
[...]
option domain-name      "client1.tp3";

option domain-name_servers dns1.server1.tp3;
[...]
```

3. Get deeper
-> A. DNS forwarder
```
[Y@ds1 ~]$ sudo cat /etc/named.conf
[...]
        recursion yes;
```

-> B. On revient sur la conf du DHCP
```
[Y@johnny ~]$ ip a
[...]
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:c8:fb:9e brd ff:ff:ff:ff:ff:ff
    inet 10.3.0.130/26 brd 10.3.0.191 scope global dynamic noprefixroute enp0s8
       valid_lft 415sec preferred_lft 415sec
    inet6 fe80::a00:27ff:fec8:fb9e/64 scope link
       valid_lft forever preferred_lft forever
[Y@johnny ~]$ dig ynov.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 38794
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 29b175f60dcc6730a9fdf770616d05f92fddeaf26a0dba25 (good)
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; Query time: 1 msec
;; SERVER: 10.3.0.2#53(10.3.0.2)
;; WHEN: Mon Oct 18 01:28:24 EDT 2021
;; MSG SIZE  rcvd: 65
```