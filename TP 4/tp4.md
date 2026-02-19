


# TP – Administration SSH et Serveur Web Nginx

**Nom :** Ulysse Prévost Lacaze  
**Classe :** B1  
**OS hôte :**  Windows  
**OS VM :** Ubuntu     

---

## Partie 1 – Mise en place de l’environnement virtualisé

### 1. Installation de VirtualBox et création de la VM

- RAM : 4 Go  
- Disque : 25 Go  
- Réseau : NAT ET HOST ONLY  

✅ VM créée et démarrée avec succès.

---

### 2. Vérification de l’adresse IP de la VM

**Commande utilisée :**
```bash
ip a
````

**Sortie :**

```bash
cour@cour-VirtualBox:/etc/nginx/ssl$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:41:36:1e brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 75350sec preferred_lft 75350sec
    inet6 fd17:625c:f037:2:5d14:8bc6:d5c0:1ff9/64 scope global temporary dynamic
       valid_lft 85994sec preferred_lft 13994sec
    inet6 fd17:625c:f037:2:a00:27ff:fe41:361e/64 scope global dynamic mngtmpaddr
       valid_lft 85994sec preferred_lft 13994sec
    inet6 fe80::a00:27ff:fe41:361e/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:71:fb:37 brd ff:ff:ff:ff:ff:ff
    inet 192.168.106.9/24 brd 192.168.106.255 scope global dynamic noprefixroute enp0s8
       valid_lft 361sec preferred_lft 361sec
    inet6 fe80::58ae:df9f:7837:4e26/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

**Test de connectivité depuis la machine hôte :**

```bash
PS C:\Users\ulysse\.ssh> ping 192.168.106.9
```

**Résultat :**

```text
Envoi d’une requête 'Ping'  192.168.106.9 avec 32 octets de données :
Réponse de 192.168.106.9 : octets=32 temps=1 ms TTL=64
Réponse de 192.168.106.9 : octets=32 temps=1 ms TTL=64
Réponse de 192.168.106.9 : octets=32 temps=2 ms TTL=64
Réponse de 192.168.106.9 : octets=32 temps<1ms TTL=64

Statistiques Ping pour 192.168.106.9:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 2ms, Moyenne = 1ms
```

👉 La VM est bien accessible depuis la machine hôte.

---

## Partie 2 – Serveur SSH

### 1. Installation du serveur SSH

**Commande :**

```bash
sudo apt install openssh-server
```


---

### 2. Vérification du service SSH

**Commande :**

```bash
systemctl status ssh
```

**Sortie :**

```text
[sudo] Mot de passe de cour :
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)
     Active: active (running) since Thu 2026-02-19 19:33:07 CET; 2h 11min ago
TriggeredBy: ● ssh.socket
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 12266 (sshd)
      Tasks: 1 (limit: 4551)
     Memory: 1.2M (peak: 4.3M)
        CPU: 1.600s
     CGroup: /system.slice/ssh.service
             └─12266 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

```

**Vérification du port d’écoute :**

```bash
ss -tuln | grep ssh
```

---

### 3. Connexion SSH depuis la machine cliente

```bash
ssh cour@192.168.106.9
```

✅ Connexion réussie.

---

### 4. Authentification par clé SSH

**Génération de la clé :**

```bash
ssh-keygen
```

**Copie de la clé sur le serveur :**

```bash
Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub" | Out-String | ssh cour@192.168.106.9 "cat >> ~/.ssh/authorized_keys"
```

**Test de connexion sans mot de passe :**

```bash
ssh cour@192.168.106.9
```

**Sortie :**
```POWERSHELL
PS C:\Users\ulysse\.ssh> ssh cour@192.168.106.9

Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-37-generic x86_64)



* Documentation: https://help.ubuntu.com

* Management: https://landscape.canonical.com

* Support: https://ubuntu.com/pro



La maintenance de sécurité étendue pour Applications n'est pas activée.



175 mises à jour peuvent être appliquées immédiatement.

Pour afficher ces mises à jour supplémentaires, exécuter : apt list --upgradable



8 mises à jour de sécurité supplémentaires peuvent être appliquées avec ESM Apps.

En savoir plus sur l'activation du service ESM Apps at https://ubuntu.com/esm



*** Le système doit être redémarré ***

Last login: Thu Feb 19 19:26:04 2026 from 192.168.106.1

cour@cour-VirtualBox:~$ 
```
👉 La connexion se fait sans mot de passe.

---

## Partie 3 – Sécurisation SSH

### 1. Modification de la configuration SSH

**Fichier modifié :**

```bash
sudo nano /etc/ssh/sshd_config
```

**Paramètres changés :**

```text
PermitRootLogin no
PasswordAuthentication no
Port 2222
```

---

### 2. Redémarrage du service SSH

```bash
sudo systemctl restart ssh
```

---

### 3. Test de connexion avec le nouveau port

```bash
ssh -i $env:USERPROFILE\.ssh\id_ed25519 -p 2222 cour@192.168.106.9
```
**Sortie :**
```text

Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-37-generic x86_64)



* Documentation: https://help.ubuntu.com

* Management: https://landscape.canonical.com

* Support: https://ubuntu.com/pro



La maintenance de sécurité étendue pour Applications n'est pas activée.



175 mises à jour peuvent être appliquées immédiatement.

Pour afficher ces mises à jour supplémentaires, exécuter : apt list --upgradable



8 mises à jour de sécurité supplémentaires peuvent être appliquées avec ESM Apps.

En savoir plus sur l'activation du service ESM Apps at https://ubuntu.com/esm



*** Le système doit être redémarré ***

Last login: Thu Feb 19 19:26:04 2026 from 192.168.106.1

cour@cour-VirtualBox:~$ 
```
✅ Connexion fonctionnelle.

---

### 4. Alias SSH

**Fichier :**

```bash
New-Item -Path "$env:USERPROFILE\.ssh\config" -ItemType File
notepad $env:USERPROFILE\.ssh\config
```

**Contenu :**

```text
Host tp4
    HostName 192.168.106.9
    User cour
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

**Connexion simplifiée :**

```bash
ssh tp4
```

---

## Partie 4 – Transfert de fichiers

### 1. SCP

```bash
scp fichier.txt serveur-tp:/home/etudiant/
```



### 2. RSYNC

```bash
rsync -avz dossier_tp/ tp4:/home/cour/dossier_tp/
```

👉 Synchronisation réussie.

---

## Partie 5 – Analyse des logs et sécurité

### 1. Logs SSH

```bash
sudo tail -f /var/log/auth.log
```

**Observation :**

```text
[sudo] Mot de passe de cour :
2026-02-19T21:35:01.879642+01:00 cour-VirtualBox CRON[15813]: pam_unix(cron:session): session closed for user root
2026-02-19T21:44:33.525297+01:00 cour-VirtualBox sudo:     cour : TTY=pts/1 ; PWD=/etc/nginx/ssl ; USER=root ; COMMAND=/usr/bin/systemctl status ssh
2026-02-19T21:44:33.561139+01:00 cour-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by cour(uid=1000)
2026-02-19T21:44:35.073774+01:00 cour-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
2026-02-19T21:45:04.527102+01:00 cour-VirtualBox CRON[15857]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
2026-02-19T21:45:04.592550+01:00 cour-VirtualBox CRON[15857]: pam_unix(cron:session): session closed for user root
2026-02-19T21:55:01.462620+01:00 cour-VirtualBox CRON[15888]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
2026-02-19T21:55:01.490481+01:00 cour-VirtualBox CRON[15888]: pam_unix(cron:session): session closed for user root
2026-02-19T21:59:38.830754+01:00 cour-VirtualBox sudo:     cour : TTY=pts/1 ; PWD=/etc/nginx/ssl ; USER=root ; COMMAND=/usr/bin/tail -f /var/log/auth.log
2026-02-19T21:59:38.846687+01:00 cour-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by cour(uid=1000)
```

---

### 2. Fail2Ban

**Installation :**

```bash
sudo apt install fail2ban
```

**Statut :**

```bash
sudo systemctl status fail2ban
```

**status de fail2ban :**

```text
cour@cour-VirtualBox:/etc/nginx/ssl$ sudo systemctl status fail2ban
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/usr/lib/systemd/system/fail2ban.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-02-19 22:00:32 CET; 2min 40s ago
       Docs: man:fail2ban(1)
   Main PID: 16379 (fail2ban-server)
      Tasks: 5 (limit: 4551)
     Memory: 23.7M (peak: 24.4M)
        CPU: 2.084s
     CGroup: /system.slice/fail2ban.service
             └─16379 /usr/bin/python3 /usr/bin/fail2ban-server -xf start
```

---

## Partie 6 – Tunnel SSH

### 1. Tunnel local

```bash
ssh -L 8080:localhost:80 tp4
```

👉 Le service web distant est accessible localement.
```text
PS C:\Users\ulysse\.ssh> ssh -L 8080:localhost:80 tp4
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-37-generic x86_64)
 
 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro
 
La maintenance de sécurité étendue pour Applications n'est pas activée.
 
175 mises à jour peuvent être appliquées immédiatement.
Pour afficher ces mises à jour supplémentaires, exécuter : apt list --upgradable
 
8 mises à jour de sécurité supplémentaires peuvent être appliquées avec ESM Apps.
En savoir plus sur l'activation du service ESM Apps at https://ubuntu.com/esm
 
*** Le système doit être redémarré ***
Last login: Thu Feb 19 19:35:47 2026 from 192.168.106.1

```
---

### 2. Tunnel distant

```bash
ssh -R 2222:localhost:22 tp4
```

👉 Accès distant fonctionnel.
```text
cour@cour-VirtualBox:~$ ssh -R 2222:localhost:22 tp4
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-37-generic x86_64)
 
 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro
 
La maintenance de sécurité étendue pour Applications n'est pas activée.
 
175 mises à jour peuvent être appliquées immédiatement.
Pour afficher ces mises à jour supplémentaires, exécuter : apt list --upgradable
 
8 mises à jour de sécurité supplémentaires peuvent être appliquées avec ESM Apps.
En savoir plus sur l'activation du service ESM Apps at https://ubuntu.com/esm
 
*** Le système doit être redémarré ***
Last login: Thu Feb 19 19:49:05 2026 from 192.168.106.1

```
---

## Partie 7 – Nginx et HTTPS

### 1. Installation de Nginx

```bash
sudo apt install nginx
```

---

### 2. Création du site web

```bash
sudo mkdir -p /var/www/site-tp
```

**index.html :**

```html
<h1>Bienvenue sur le site du TP SSH & Nginx</h1>
```

---

### 3. Configuration HTTP

**Fichier Nginx :**

```text
/etc/nginx/sites-available/site-tp
```
```yaml
server {
    listen 80;
    server_name localhost;
    root /var/www/site-tp;
    index index.html;
}

server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate /etc/nginx/ssl/site-tp.crt;
    ssl_certificate_key /etc/nginx/ssl/site-tp.key;

    root /var/www/site-tp;
    index index.html;
}
```

👉 Site accessible en HTTP.

---

### 4. Certificat HTTPS auto-signé

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout site-tp.key -out site-tp.crt
```

---

### 5. Redirection HTTP → HTTPS

👉 Configuration effectuée et testée.

**Test :**

```bash
 curl.exe -k https://192.168.106.9
<h1>Bienvenue sur le site TP Nginx </h1>
```

---

## Partie 8 – Firewall et permissions

### 1. Firewall UFW

```bash
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

**Statut :**

```bash
sudo ufw status
```
**Sortie :**
```
cour@cour-VirtualBox:/etc/nginx/ssl$ sudo ufw status
État : actif

Vers                       Action      De
----                       ------      --
22                         ALLOW       Anywhere
8888/tcp                   ALLOW       Anywhere
Nginx Full                 ALLOW       Anywhere
2222/tcp                   ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
8888/tcp (v6)              ALLOW       Anywhere (v6)
Nginx Full (v6)            ALLOW       Anywhere (v6)
2222/tcp (v6)              ALLOW       Anywhere (v6)

cour@cour-VirtualBox:/etc/nginx/ssl$
```

---

### 2. Permissions

```bash
sudo chown -R www-data:www-data /var/www/site-tp
sudo chmod -R 755 /var/www/site-tp
```

---

## Partie 9 – Validation finale

✅ SSH sécurisé (clé + port personnalisé)
✅ Fail2Ban actif
✅ Transferts SCP / SFTP / RSYNC
✅ Nginx HTTP + HTTPS
✅ Redirection HTTP → HTTPS
✅ Firewall et permissions correctes

---

## Conclusion

Ce TP m’a permis de mettre en place une infrastructure serveur sécurisée avec SSH et Nginx, ainsi que de comprendre les bonnes pratiques de sécurisation réseau et système. Bien évidemment j'ai dû résumer le rendu à cause de contraintes de mise en page.


