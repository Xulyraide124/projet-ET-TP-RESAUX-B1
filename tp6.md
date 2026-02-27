
---

# TP – Mise en place d'un serveur OpenVPN sur Ubuntu Server

## Objectifs

À l'issue de ce TP, vous devrez être capables de :

* Installer et configurer OpenVPN sur Ubuntu Server
* Mettre en place une infrastructure de certificats (PKI)
* Comprendre le rôle du NAT et du routage IP
* Générer un profil client fonctionnel
* Diagnostiquer un service système en échec

---

## Mise en place

**Ressources nécessaires :**

* Une VM Ubuntu Server LTS
* Un réseau NAT
* Accès SSH configuré pour administration

---

## Préparation du système

**Étapes :**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install openvpn easy-rsa -y
```

---

## Partie 1 : Comprendre la PKI

### Questions

1. À quoi sert une autorité de certification (CA) ?
 *Réponse : C'est un tiers de confiance qui vérifie l'identité d'une entité et signe son certificat numérique pour garantir son authenticité.*

2. Quelle différence entre clé privée et certificat ?
   *Réponse : La clé privée est un code secret utilisé pour déchiffrer ou signer, tandis que le certificat est un document public prouvant votre identité.*

3. Pourquoi un serveur VPN a-t-il besoin de certificats ?
   *Réponse : Ils servent à authentifier mutuellement le serveur et l'utilisateur tout en chiffrant la connexion pour sécuriser le tunnel de données.*

---

## Création de l'infrastructure Easy-RSA

**Étapes principales :**

```bash
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-req server nopass
./easyrsa sign-req server server
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1
./easyrsa gen-dh
openvpn --genkey --secret ta.key
```

### Questions

* Où Easy-RSA crée-t-il ses fichiers ?
  *Réponse : Ils sont créés par défaut dans un répertoire nommé pki/ situé à l'intérieur du dossier où vous avez initialisé Easy-RS*

* Que contient le dossier `pki/` ?
  *Réponse : Il contient l'infrastructure de clés publiques : la clé privée de l'Autorité (CA), les certificats émis, les requêtes (CSR) et la base de données des signatures.*

* Quelle est la différence entre `gen-req` et `sign-req` ?
  *Réponse :gen-req génère une paire de clés et une demande de signature (CSR), tandis que sign-req permet à la CA d'approuver et de transformer cette demande en certificat valide.*

* Que se passe-t-il si vous oubliez de signer un certificat ?
  *Réponse :Le certificat reste une simple "demande" inutilisable ; sans la signature de la CA, les clients rejetteront la connexion car l'identité n'est pas authentifiée*

---

## Partie 2 : Configuration du serveur OpenVPN

**Fichier :** `/etc/openvpn/server/server.conf`

**Éléments à inclure :**

* Port d'écoute : …
* Protocole : …
* Interface virtuelle : …
* Réseau attribué aux clients : …
* Références aux certificats : …

### Questions

* Que signifie `dev tun` ?
  *Réponse :Il définit un tunnel de niveau 3 (IP), créant une interface réseau virtuelle routée pour transporter le trafic internet.*

* Quelle est la différence entre UDP et TCP pour un VPN ?
  *Réponse :UDP est privilégié pour sa rapidité et l'absence de latence liée au contrôle d'erreurs, contrairement au TCP plus lent.*

* Quelle plage IP choisir pour le VPN ? Pourquoi ?
  *Réponse :Utilisez un sous-réseau privé (ex: 10.8.0.0/24) distinct de votre LAN local pour éviter les conflits d'adressage et le chevauchement.*

---

## Routage et NAT

**Commandes et configuration :**

```bash
sudo nano /etc/sysctl.conf
# activer net.ipv4.ip_forward=1

sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```

### Questions

* Où se configure le paramètre `ip_forward` ?
  *Réponse : Il se configure temporairement dans /proc/sys/net/ipv4/ip_forward ou de manière permanente dans le fichier /etc/sysctl.conf.*

* Quelle commande permet d'afficher les règles NAT actuelles ?
  *Réponse : Utilisez la commande iptables -t nat -L -n -v ou nft list table nat selon votre gestionnaire de pare-feu.*

* Pourquoi faut-il "masquerader" le réseau VPN ?
  *Réponse : Il remplace l'IP source du client VPN par celle du serveur pour que les paquets retour sachent comment revenir via Internet.*

---

## Démarrage et analyse du service

```bash
sudo systemctl start openvpn-server@server
sudo systemctl status openvpn-server@server
```

### Questions

* Quelle commande permet d'afficher les logs système d'un service ?
  *Réponse : Utilisez journalctl -u nom_du_service pour isoler les messages système spécifiques à ce service.*

* Quelle est la différence entre `status` et `journalctl` ?
  *Réponse : systemctl status donne l'état actuel et les 10 dernières lignes, alors que journalctl permet de consulter l'historique complet.*

* Les chemins vers les certificats sont-ils corrects ?
  *Réponse : Ils sont corrects s'ils pointent vers les fichiers .crt et .key existants, souvent dans /etc/openvpn/ ou /etc/openvpn/server/.*

---

## Partie 3 : Création du profil client

**Exemple de fichier `.ovpn` :**

```text
client
dev tun
proto udp
remote <IP_SERVEUR> 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert client1.crt
key client1.key
tls-auth ta.key 1
cipher AES-256-CBC
```

### Questions

* Comment intégrer un certificat directement dans un fichier `.ovpn` ?
  *Réponse : Utilisez les balises XML <ca>, <cert> et <key> pour y copier-coller le contenu texte de vos fichiers de certificat.*

* Pourquoi la clé privée ne doit-elle jamais être partagée publiquement ?
  *Réponse : Sa divulgation permettrait à n'importe qui de déchiffrer vos communications et d'usurper votre identité sur le réseau VPN.*

---

## Tests et validation

* Établir une connexion VPN depuis un client
* Vérifier l'adresse IP obtenue : `curl ifconfig.me`
* Vérifier l'accès Internet via le tunnel

### Questions

* Comment vérifier que votre trafic passe par le VPN ?
  *Réponse : Consultez un site comme ifconfig.me ou utilisez curl pour confirmer que votre adresse IP publique affichée est celle du serveur VPN.*

* Que se passe-t-il si le port 1194 est bloqué ?
  *Réponse : La connexion échouera totalement par défaut ; il faudra alors configurer OpenVPN sur un autre port, comme le 443 (HTTPS) pour contourner le pare-feu.*

---

## Partie 4 : Bonus

* Ajouter plusieurs clients
* Mettre en place une révocation de certificat
* Passer le VPN en TCP
* Activer une authentification par mot de passe en complément des certificats

---