# TP2-Réseaux
**I. ARP**
1. Echange ARP

```
[Y@node1 ~]$ ping 10.2.1.12
PING 10.2.1.12 (10.2.1.12) 56(84) bytes of data.
[...]
--- 10.2.1.12 ping statistics ---
13 packets transmitted, 13 received, 0% packet loss, time 12075ms
rtt min/avg/max/mdev = 0.822/1.236/1.494/0.181 ms
```
Table ARP de la machine n°1:
```
[Y@node1 ~]$ ip neigh show
192.168.1.254 dev enp0s8  FAILED
10.2.1.12 dev enp0s8 lladdr 08:00:27:e2:e3:55 STALE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:31 REACHABLE
```
Table ARP de la machine n°2:
```
[Y@node2 ~]$ ip neigh show
10.2.1.11 dev enp0s8 lladdr 08:00:27:b0:ed:a7 STALE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:31 REACHABLE
192.168.1.254 dev enp0s8  FAILED
```
Adresse mac corespondant à l'ip précédement ping:
```
[Y@node1 ~]$ ip neigh show 10.2.1.12
10.2.1.12 dev enp0s8 lladdr 08:00:27:e2:e3:55 STALE
```

En effectuant par la suite la commande "ip a" sur la seconde machine, on peut déterminer que les adresses mac correspondent:

```
[Y@node2 ~]$ ip a
[...]
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:e2:e3:55 brd ff:ff:ff:ff:ff:ff
[...]
```

2. Analyse de trames

| ordre | type trame | source            | destination       |
| ----- | ---------- | ----------------- | ----------------- |
| 1     | SSH        | 10.2.1.11         | 10.2.1.1          |
| 2     | TCP        | 10.2.1.1          | 10.2.1.11         |
| 3     | ARP        | PcsCompu_e2:e3:55 | Broadcast         |
| 4     | ARP        | PcsCompu_b0:ed:a7 | PcsCompu_e2:e3:55 |
| 6     | ICMP       | 10.2.1.12         | 10.2.1.11         |
| 5     | ICMP       | 10.2.1.12         | 10.2.1.11         |
| 7     | ICMP       | 10.2.1.12         | 10.2.1.11         |
| 8     | ICMP       | 10.2.1.12         | 10.2.1.11         |
| 9     | ICMP       | 10.2.1.12         | 10.2.1.11         |
| 10    | ICMP       | 10.2.1.12         | 10.2.1.11         |

**II. Routage**
1. Mise en place du routage

Activation du routage:
```
[Y@router ~]$ sudo firewall-cmd --get-active-zone
public
  interfaces: enp0s3 enp0s8
```

