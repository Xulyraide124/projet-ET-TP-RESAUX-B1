

# Compte-Rendu TP2 - Exploration Réseau

**Nom :** Ulysse Prevost Lacaze

**Groupe :** VM1 et VM2

**Date :** 23 Janvier 2026

---

## I. Exploration de la pile TCP/IP (individuel)

### 1. Affichage d'informations

**Commande utilisée :** `ip a`

```bash

boot@boot:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:3b:4e:77 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84227sec preferred_lft 84227sec
    inet6 fd17:625c:f037:2:a00:27ff:fe3b:4e77/64 scope global dynamic mngtmpaddr noprefixroute
       valid_lft 86363sec preferred_lft 14363sec
    inet6 fe80::a00:27ff:fe3b:4e77/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:e8:88:08 brd ff:ff:ff:ff:ff:ff
    inet 192.168.106.3/24 metric 100 brd 192.168.106.255 scope global dynamic enp0s8
       valid_lft 339sec preferred_lft 339sec
    inet6 fe80::a00:27ff:fee8:8808/64 scope link
       valid_lft forever preferred_lft forever
```

* **Interface WiFi (enp0s3) :** IP `10.0.2.15`, MAC `08:00:27:3b:4e:77`
* **Interface Ethernet (enp0s8) :** IP `192.168.106.3`, MAC `08:00:27:e8:88:08`
* **Gateway :** Trouvée avec `ip route show default` -> `default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100`

---

## II. Exploration de la pile TCP/IP (en duo)

### 3. Modification d'adresse IP

J'ai changé mon IP manuellement pour qu’elle passe  de 192.168.106.4 a  192.168.106.100.

```bash
boot@boot:~$ ip a
...
boot@boot:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:3b:4e:77 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 83967sec preferred_lft 83967sec
    inet6 fd17:625c:f037:2:a00:27ff:fe3b:4e77/64 scope global dynamic mngtmpaddr noprefixroute
       valid_lft 86103sec preferred_lft 14103sec
    inet6 fe80::a00:27ff:fe3b:4e77/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:e8:88:08 brd ff:ff:ff:ff:ff:ff
    inet 192.168.106.100/24 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fee8:8808/64 scope link
       valid_lft forever preferred_lft forever

```

**Test de connectivité :**

```bash
boot@boot:~$ ping 192.168.106.4
PING 192.168.106.4 (192.168.106.4) 56(84) bytes of data.
64 bytes from 192.168.106.4: icmp_seq=1 ttl=64 time=6.98 ms
64 bytes from 192.168.106.4: icmp_seq=2 ttl=64 time=1.52 ms
64 bytes from 192.168.106.4: icmp_seq=3 ttl=64 time=2.07 ms
^C
--- 192.168.106.4 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2011ms
rtt min/avg/max/mdev = 1.524/3.526/6.984/2.455 ms

```

### 4. Utilisation comme Gateway

J'ai coupé mon accès internet direct (`enp0s3 down`) et ajouté mon binôme comme passerelle.

```bash
boot@boot:~$ sudo ip link set enp0s3 down
boot@boot:~$ sudo ip route del default
sudo ip route add default via 192.168.106.4 dev enp0s8
RTNETLINK answers: No such process

```

**Preuve de fonctionnement :**

```bash

boot@boot:~$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=254 time=37.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=254 time=26.7 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=254 time=34.6 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2046ms
rtt min/avg/max/mdev = 26.679/32.822/37.157/4.464 ms

```

### 5. Chat privé avec `nc`
le firewall:
```bash
boot@boot:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
8888/tcp                   ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
8888/tcp (v6)              ALLOW       Anywhere (v6)

boot@boot:~$
```
Connexion au serveur de mon binôme :
sur ma deuxième vm nomé cour
```bash
nc -l -p 8888
```
puis sur ma premiere vm 
```bash
boot@boot:~$ nc 192.168.106.4 8888
toto
aled
DIO

```
et voici l'aoutpout de la deuxième vm :
```bash
cour@cour-VirtualBox:~$ nc -l -p 8888
toto
tralla
hi boy
salut je suis jean pierre pole naref
JOJO
toto
aled
DIO 
```

### 6. Analyse de trafic (tcpdump et wireshark)

Capture des messages `nc` sur le port 8888 :

```bash
cour@cour-VirtualBox:~$ sudo tcpdump -i enp0s8 port 8888 -A
cour@cour-VirtualBox:~$ sudo tcpdump -i enp0s8 port 8888 -A
[sudo] Mot de passe de cour :
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes
13:07:11.873129 IP 192.168.106.100.41938 > cour-VirtualBox.8888: Flags [P.], seq 983171098:983171103, ack 3451165140, win 502, options [nop,nop,TS val 1418693509 ecr 2208524992], length 5
E..9y.@.@.kX..jd..j...".:..................
T.....j.toto

13:07:11.873283 IP cour-VirtualBox.8888 > 192.168.106.100.41938: Flags [.], ack 5, win 510, options [nop,nop,TS val 2208567729 ecr 1418693509], length 0
E..4Nw@.@.....j...jd".......:.......U......
....T...
13:07:19.986110 IP 192.168.106.100.41938 > cour-VirtualBox.8888: Flags [P.], seq 5:10, ack 1, win 502, options [nop,nop,TS val 1418701622 ecr 2208567729], length 5
E..9y.@.@.kW..jd..j...".:...........PG.....
T..6....aled

13:07:19.986289 IP cour-VirtualBox.8888 > 192.168.106.100.41938: Flags [.], ack 10, win 510, options [nop,nop,TS val 2208575841 ecr 1418701622], length 0
E..4Nx@.@.....j...jd".......:..$....U......
..1aT..6
13:07:24.268706 IP 192.168.106.100.41938 > cour-VirtualBox.8888: Flags [P.], seq 10:14, ack 1, win 502, options [nop,nop,TS val 1418705905 ecr 2208575841], length 4
E..8y.@.@.kW..jd..j...".:..$........]U.....
T.....1aDIO

13:07:24.268936 IP cour-VirtualBox.8888 > 192.168.106.100.41938: Flags [.], ack 14, win 510, options [nop,nop,TS val 2208580124 ecr 1418705905], length 0
E..4Ny@.@.....j...jd".......:..(....U......
..B.T...
13:08:39.538950 IP 192.168.106.100.41938 > cour-VirtualBox.8888: Flags [F.], seq 14, ack 1, win 502, options [nop,nop,TS val 1418781184 ecr 2208580124], length 0
E..4y.@.@.kZ..jd..j...".:..(...............
T.....B.
13:08:39.540118 IP cour-VirtualBox.8888 > 192.168.106.100.41938: Flags [F.], seq 1, ack 15, win 510, options [nop,nop,TS val 2208655395 ecr 1418781184], length 0
E..4Nz@.@.....j...jd".......:..)....U......
..h#T...
13:08:39.540951 IP 192.168.106.100.41938 > cour-VirtualBox.8888: Flags [.], ack 2, win 502, options [nop,nop,TS val 1418781186 ecr 2208655395], length 0
E..4y.@.@.kY..jd..j...".:..)...............
T.....h#
^C
9 packets captured
9 packets received by filter
0 packets dropped by kernel

```

> **Note :** On remarque que le texte circule en clair. Un attaquant pourrait intercepter la conversation.

---

## III. Autres outils et protocoles

### 1. DHCP

* **Serveur DHCP :** `10.0.2.2` (via `nmcli device show enp0s3`)
* **Renouvellement du bail :**

```bash
boot@boot:~$ sudo dhclient -v -r enp0s3  # Relâcher l'IP
sudo dhclient -v enp0s3     # Demander une nouvelle
Killed old client process
Internet Systems Consortium DHCP Client 4.4.3-P1
Copyright 2004-2022 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/enp0s3/08:00:27:3b:4e:77
Sending on   LPF/enp0s3/08:00:27:3b:4e:77
Sending on   Socket/fallback
xid: warning: no netdev with useable HWADDR found for seed's uniqueness enforcement
xid: rand init seed (0x690c6455) built using gethostid
DHCPRELEASE of 10.0.2.15 on enp0s3 to 10.0.2.2 port 67 (xid=0x7d4ad985)
Internet Systems Consortium DHCP Client 4.4.3-P1
Copyright 2004-2022 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/enp0s3/08:00:27:3b:4e:77
Sending on   LPF/enp0s3/08:00:27:3b:4e:77
Sending on   Socket/fallback
xid: warning: no netdev with useable HWADDR found for seed's uniqueness enforcement
xid: rand init seed (0x690c6455) built using gethostid
DHCPDISCOVER on enp0s3 to 255.255.255.255 port 67 interval 3 (xid=0x85d94a7d)
DHCPOFFER of 10.0.2.15 from 10.0.2.2
DHCPREQUEST for 10.0.2.15 on enp0s3 to 255.255.255.255 port 67 (xid=0x7d4ad985)
DHCPACK of 10.0.2.15 from 10.0.2.2 (xid=0x85d94a7d)
Error: ipv4: Address already assigned.
Setting LLMNR support level "yes" for "2", but the global support level is "no".
bound to 10.0.2.15 -- renewal in 32923 seconds.

```

### 2. DNS

**Lookup google.com :**

```bash
boot@boot:~$ nslookup google.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   google.com
Address: 172.217.20.46
Name:   google.com
Address: 2a00:1450:4007:80f::200e

boot@boot:~$
```
**Lookup ynov.com :**
```bash
boot@boot:~$ nslookup ynov.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   ynov.com
Address: 104.26.10.233
Name:   ynov.com
Address: 104.26.11.233
Name:   ynov.com
Address: 172.67.74.226
Name:   ynov.com
Address: 2606:4700:20::681a:ae9
Name:   ynov.com
Address: 2606:4700:20::ac43:4ae2
Name:   ynov.com
Address: 2606:4700:20::681a:be9

boot@boot:~$

```

**Reverse Lookup :**

```bash
boot@boot:~$ nslookup 78.78.21.21
21.21.78.78.in-addr.arpa        name = host-78-78-21-21.mobileonline.telia.com.

Authoritative answers can be found from:

boot@boot:~$ nslookup 92.16.54.88
88.54.16.92.in-addr.arpa        name = host-92-16-54-88.as13285.net.

Authoritative answers can be found from:
boot@boot:~$
```

## réponse aux question :
**À quoi sert la gateway (passerelle) dans le réseau d'Ingésup ?**
La passerelle est le "point de sortie" du réseau local. Son rôle est de transférer les paquets de données qui ne sont pas destinés au réseau interne (comme une requête vers `google.fr`) vers l'extérieur (Internet). Sans gateway, tu peux communiquer avec les autres étudiants du réseau Ingésup, mais tu ne peux pas accéder au Web.
* **Calcul des adresses disponibles (Exemple pour un réseau `/24`) :**
* **Première IP disponible :** C'est l'adresse juste après l'adresse de réseau (ex: `192.168.1.1`).
* **Dernière IP disponible :** C'est l'adresse juste avant l'adresse de broadcast (ex: `192.168.1.254`).
* *Note :* L'adresse de réseau (la première) et le broadcast (la dernière) ne sont jamais attribuables à un PC.


* **Que se passe-t-il en cas de conflit d'IP (doublon) ?**
Si deux machines ont la même IP, le réseau devient instable. Pour l'envoi de messages vers l'extérieur, cela peut fonctionner, mais pour la réception, les commutateurs (switches) et le routeur ne sauront pas à quelle machine physique (adresse MAC) livrer les paquets. C'est le principe du facteur qui a deux maisons avec le même numéro : il ne sait pas dans quelle boîte aux lettres déposer le courrier.
* **Qu'est-ce qu'un bail DHCP (DHCP Lease) ?**
C'est une "location" temporaire d'adresse IP. Le serveur DHCP te prête une adresse pour une durée déterminée (ex: 24h). À la fin de cette durée, ton PC doit demander au serveur s'il peut garder l'adresse ou en obtenir une nouvelle.
* **Fonctionnement de DHCP dans les grandes lignes :**
C'est un dialogue en 4 étapes (souvent appelé **DORA**) :
1. **D**iscovery : Le client cherche un serveur DHCP.
2. **O**ffer : Le serveur propose une IP.
3. **R**equest : Le client accepte la proposition.
4. **A**cknowledgment : Le serveur valide et donne les infos (IP, masque, gateway, DNS).



---

## Bilan

Pendant ce TP, j'ai appris à configurer manuellement des interfaces réseau, à manipuler des routes (gateway) et à utiliser des outils de diagnostic (`ping`, `dig`) et d'analyse (`tcpdump`). J'ai compris l'importance de la passerelle pour sortir d'un réseau local et le rôle crucial du DHCP pour l'automatisation de la configuration IP.

