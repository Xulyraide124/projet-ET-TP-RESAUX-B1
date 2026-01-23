# Compte-Rendu TP2 : Exploration réseau sous Linux

**Étudiant :** [Ton Nom]

**Contexte :** Simulation de réseau local via deux machines virtuelles (VM) Ubuntu en mode *Host-Only* et *NAT*.

---

## I. Exploration de la pile TCP/IP locale

### 1. Affichage d'informations

Sur la VM **cour**, l'affichage des interfaces réseau montre deux cartes principales :

* **enp0s3 (NAT) :** IP `10.0.2.15/24` (Accès Internet).
* **enp0s8 (Host-Only) :** IP `192.168.106.4/24` (Réseau local privé).

**Passerelle (Gateway) :**

```bash
cour@cour-VirtualBox:~$ ip route | grep default
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100

```

### 2. Scan de réseau avec Nmap

Utilisation de `nmap` pour découvrir les hôtes sur le réseau privé `192.168.106.0/24` :

```bash
cour@cour-VirtualBox:~$ nmap -sn 192.168.106.0/24
Nmap scan report for 192.168.106.2
Host is up (0.012s latency).
Nmap scan report for 192.168.106.3
Host is up (0.0097s latency).
Nmap scan report for cour-VirtualBox (192.168.106.4)
Host is up (0.0071s latency).

```

*Analyse :* On identifie la VM **boot** (`.3`), la VM **cour** (`.4`) et le service VirtualBox (`.2`).

---

## II. Exploration de la pile TCP/IP en duo

### 1. Messagerie instantanée avec Netcat (`nc`)

Test de communication entre **cour** (client) et **boot** (serveur) sur le port `8888`.

**Côté Serveur (boot) :**

```bash
boot@boot:~$ nc -l -p 8888
j'aime la vie
toto aime la vie

```

**Côté Client (cour) :**

```bash
cour@cour-VirtualBox:~$ nc 192.168.106.3 8888
j'aime la vie
toto aime la vie

```

### 2. Analyse de trafic avec Tshark

Capture des paquets TCP lors de la session Netcat pour vérifier l'encapsulation et la confidentialité.

```bash
cour@cour-VirtualBox:~$ sudo tshark -i enp0s8 -Y tcp.port==8888 -x
0040  98 57 63 20 20 6c 27 68 73 69 74 6f 69 72 20 61   .Wc  l'hsitoir a
0050  20 61 72 65 20 6c 27 69 73 68 6f 6f 69 72 65 20    are l'ishooire 
0060  64 65 20 6c 61 20 76 69 65 0a                     de la vie.

```

*Conclusion :* Les données circulent en clair dans la charge utile (payload) du segment TCP.

### 3. Manipulation du Pare-feu (UFW)

Configuration du firewall pour autoriser le SSH (port 22) et le chat (port 8888) avant activation.

```bash
cour@cour-VirtualBox:~$ sudo ufw allow 22
cour@cour-VirtualBox:~$ sudo ufw allow 8888/tcp
cour@cour-VirtualBox:~$ sudo ufw enable

```

---

## III. Outils et Protocoles Élémentaires

### 1. DNS (Domain Name System)

Résolution de nom (Forward) et résolution inverse (Reverse) via `nslookup`.

* **Forward :** `google.com` -> `172.217.20.46`
* **Reverse :** ```bash
cour@cour-VirtualBox:~$ nslookup 78.78.21.21
21.21.78.78.in-addr.arpa  name = host-78-78-21-21.mobileonline.telia.com.

```

### 2. DHCP (Dynamic Host Configuration Protocol)
Observation du cycle **DORA** (Discover, Offer, Request, Ack) pour l'obtention d'une IP.

```bash
boot@boot:~$ sudo dhclient -v enp0s3
DHCPDISCOVER on enp0s3 to 255.255.255.255 port 67 interval 3
DHCPOFFER of 10.0.2.15 from 10.0.2.2
DHCPREQUEST for 10.0.2.15 on enp0s3 to 255.255.255.255 port 67
DHCPACK of 10.0.2.15 from 10.0.2.2
bound to 10.0.2.15 -- renewal in 32667 seconds.

```

Le serveur DHCP identifié est `10.0.2.2`.

---

## Bilan

Ce TP a permis de valider la configuration manuelle et dynamique des interfaces Linux, l'utilisation d'outils de diagnostic réseau (`ping`, `nmap`, `tshark`) et la compréhension des flux de données à travers un pare-feu.

---