# B2 / TP1-Réseaux (Yann FARANT)

**I.	Exploration locale en solo**
1.	Affichage d'informations sur la pile TCP/IP locale

Affichage des infos des cartes réseaux de votre pc.

Ligne de commande utilisé: « ipconfig /all »

Avec l'interface graphique:
![](https://i.imgur.com/yNtyYiA.png)

Affichage de la gateway.

*A quoi sert la gateway dans le réseau d'ynov?*

-> La gateway assure la connexion de tout autre réseau sur celui d'ynov


2. Modifications des informations

        A. Modification d'adresse IP (part 1)
    
![](https://i.imgur.com/hC6Fidh.jpg)
![](https://i.imgur.com/H0FA3j1.jpg)

*Expliquez pourquoi c'est possible de perdre son accès internet en faisant cette opération.*

-> Utiliser la même adresse ip qu'un autre utilisateur du réseau ne permet pas de nous différencier sur ce même réseau. Ne sachant pas à quelle machine envoyer les données, ces dernières ne sont alors pas transmis.

    B. Table ARP

![](https://i.imgur.com/oo87xDt.jpg)

-> J'ai repéré cette adresse mac en comparant les adresses ips ci dessus avec celle de ma "gateway" a l'aide de la commande "ipconfig /all".

-> Liste des ips qui ont été ping:
![](https://i.imgur.com/7N6U7ab.jpg)

    C. nmap
    
![](https://i.imgur.com/pIJbiTf.jpg)

    D. Modification d'adresse IP (part 2)
![](https://i.imgur.com/PW4OE4M.jpg)

-> Résultats sur le terminal après changement de l'adresse ip:

```
C:\Users\yann>ipconfig

Windows IP Configuration

[...]

Wireless LAN adapter Wi-Fi:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::f11f:de91:2e6f:b5e9%5
   IPv4 Address. . . . . . . . . . . : 10.33.0.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.33.3.253

[...]

C:\Users\yann>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:
Reply from 8.8.8.8: bytes=32 time=23ms TTL=115
Reply from 8.8.8.8: bytes=32 time=25ms TTL=115
Reply from 8.8.8.8: bytes=32 time=19ms TTL=115
Reply from 8.8.8.8: bytes=32 time=119ms TTL=115

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 19ms, Maximum = 119ms, Average = 46ms
```
    
II. Exploration locale en duo
3. Modification d'adresse IP

-> Configuration du réseau:

```
Windows IP Configuration


Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::f5e2:8100:473b:c787%6
   IPv4 Address. . . . . . . . . . . : 192.168.10.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.252
   Default Gateway . . . . . . . . . :
```
   
-> Communication avec mon partenaire:

```
C:\Users\yann>ping 192.168.10.1

Pinging 192.168.10.1 with 32 bytes of data:
Reply from 192.168.10.1: bytes=32 time=1ms TTL=128
Reply from 192.168.10.1: bytes=32 time=1ms TTL=128
Reply from 192.168.10.1: bytes=32 time=1ms TTL=128
Reply from 192.168.10.1: bytes=32 time=1ms TTL=128

Ping statistics for 192.168.10.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 1ms, Average = 1ms
```
    
-> Affchage sur la table ARP:
```
C:\Users\yann>arp -g

Interface: 10.33.0.2 --- 0x5

[...]

Interface: 192.168.10.2 --- 0x6
  Internet Address      Physical Address      Type
  192.168.10.1          98-fa-9b-fc-f9-b8     dynamic
  192.168.10.3          ff-ff-ff-ff-ff-ff     static
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.251           01-00-5e-00-00-fb     static
  224.0.0.252           01-00-5e-00-00-fc     static
  239.255.255.250       01-00-5e-7f-ff-fa     static
  255.255.255.255       ff-ff-ff-ff-ff-ff     static
  
  [...]
```

4. Utilisation d'un des deux comme gateway

```
Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::f5e2:8100:473b:c787%6
   IPv4 Address. . . . . . . . . . . : 192.168.10.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.252
   Default Gateway . . . . . . . . . : 192.168.10.1
```
   
```
C:\Users\yann>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:
Reply from 8.8.8.8: bytes=32 time=29ms TTL=114
Reply from 8.8.8.8: bytes=32 time=24ms TTL=114
Reply from 8.8.8.8: bytes=32 time=33ms TTL=114
Reply from 8.8.8.8: bytes=32 time=34ms TTL=114

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 24ms, Maximum = 34ms, Average = 30ms
```

![](https://i.imgur.com/khEUHyP.jpg)

5. Petit chat privé
    
```
C:\Users\yann\Desktop\netcat-1.11> .\nc.exe 192.168.10.1 6868
coucou !
salut
C'est moi !
Etape 5 accomplie
Etape réussi par Théo !
ET yann!
```

 6. Firewall

![](https://i.imgur.com/8BorgAq.jpg)

```
C:\Users\yann\Desktop\netcat-1.11> .\nc.exe 192.168.10.1 6565
Alors ca marche avec le pare feu ?
Niquel sergent
```
**III. Manipulations d'autres outils/protocoles côté client**
1. DHCP

PS: Je n'ai plus acccès au réseau wi-fi d'Ynov car j'y suis pas retourné.

-> A l'aide de la commande "ipconfig /all" 
![](https://i.imgur.com/vXm2kSn.png)


2. DNS

![](https://i.imgur.com/mIxP7et.png)

![](https://i.imgur.com/EuZCuue.png)

-> La premère partie de la commande correspondant au point de départ de la requête et la deuxième renvoies le nom du domaine en plus de leur adresse ip.

![](https://i.imgur.com/dHlgt3J.png)

-> La première ip provient d'un site du nom homerun, et la deuxième d'un compte mail situé a poitié. 