Ulysse Prevost Lacaze 
---

# Rapport de TP5 : pfSense – Bases d’un pare-feu

**Date :** 26 Février 2026

**Infrastructure :** Lab Virtualisé (pfSense & Ubuntu)

---

## Partie 1 – Prise en main et sécurisation

### 1. Accès à l’interface

* **Adresse IP du LAN :** `192.168.92.2` (Interface d'administration).
* **Adresse IP du WAN :** `10.0.2.15` (Obtenue par le NAT VirtualBox pour l'accès internet).
* **Utilisation du HTTPS :** Pour chiffrer les communications entre le navigateur de l'admin et le pare-feu, évitant l'interception du mot de passe sur le réseau.
* **Changement des identifiants :** Les identifiants par défaut (`admin/pfsense`) sont connus de tous. Ne pas les changer expose l'équipement à une prise de contrôle immédiate.

### 2. Sécurisation de l’accès administrateur

* **Gestion des utilisateurs :** Menu `System > User Manager`.
* **Mot de passe robuste :** Un mélange de majuscules, minuscules, chiffres et caractères spéciaux, sans lien avec l'utilisateur.
* **Priorité de sécurisation :** L'accès admin est le "cerveau" du réseau. S'il est compromis, l'attaquant peut désactiver toute la sécurité ou espionner tout le trafic.

---

## Partie 2 – Comprendre les interfaces réseau

### 3. Vérification des interfaces

* **Interface Internet (WAN) :** `em0` (ou `enp0s3` côté hyperviseur).
* **Interface interne (LAN) :** `em1` (ou `enp0s8` côté hyperviseur) avec l'IP `192.168.92.2`.
* **Inversion des interfaces :** Le pare-feu tenterait de protéger internet contre votre réseau local, bloquerait l'accès web des clients et exposerait l'interface d'administration sur le réseau public.

---

## Partie 3 – Configuration des services réseau

### 4. DHCP

* **Pourquoi DHCP :** Facilité de gestion, évite les erreurs de configuration manuelle et les conflits d'adresses IP.
* **Plage choisie :** `192.168.92.10` à `192.168.92.100`.
* **Adresses à exclure :** L'adresse du pare-feu lui-même (`.2`) et les adresses réservées aux serveurs fixes.
* **Vérification Ubuntu :** La VM a bien obtenu l'IP `192.168.92.12` via l'interface `enp0s8`.

### 5. DNS

* **Pare-feu en serveur DNS :** Il joue le rôle de relais (Forwarder/Resolver) pour centraliser et sécuriser les requêtes des clients internes.
* **Ping 8.8.8.8 OK mais DNS HS :** La connectivité IP est fonctionnelle, mais la résolution de noms ne l'est pas (problème de service DNS ou de port 53 bloqué).

---

## Partie 4 – Autoriser l’accès Internet

### 6. Règles de pare-feu

* **Configuration :** Règle "Partie 4-6" autorisant `Protocol: Any`, `Source: LAN net`, `Destination: Any`.
* **Tests réussis :** Ping vers `8.8.8.8` opérationnel.

### 7. NAT

* **Nécessité du NAT :** Les adresses privées (`192.168.x.x`) ne sont pas routables sur internet. Le NAT remplace l'IP source par l'IP publique du WAN.
* **Différence Auto/Manuel :** L'automatique génère les règles dès qu'une interface est ajoutée ; le manuel permet un contrôle granulaire sur quel flux est traduit et comment.
* **Vérification :** Via l'onglet `Status > States`, on observe les traductions d'adresses actives.

---

## Partie 5 – Filtrage

### 8 & 9. Blocage spécifique et Alias

* **Méthode :** Création d'un alias `SITES_INTERDITS`![yoyo](image\image.png).
* **Pourquoi l'alias :** Centralisation. On modifie une seule liste pour impacter plusieurs règles, rendant la configuration maintenable.
* **Blocage IP vs Nom :** Le blocage par IP peut être contourné si le site change d'IP ou utilise un CDN (Content Delivery Network).
* **HTTPS :** Le pare-feu bloque la connexion initiale (IP destination), même si le contenu reste chiffré.
* **Preuve:**  
```text
cour@cour-VirtualBox:~$ ping -4 -I enp0s8 pornhub.com
PING pornhub.com (66.254.114.41) from 192.168.92.12 enp0s8: 56(84) bytes of data.
^C
--- pornhub.com ping statistics ---
19 packets transmitted, 0 received, 100% packet loss, time 19406ms

cour@cour-VirtualBox:~$
```

---

## Partie 6 – Aller plus loin

### 10. Blocage réseaux sociaux

* **Alias utilisé :** `RS` contenant les IPs de Facebook.![yoyo](image\image2.png)
* **Règle sous un "Pass Any" :** Elle serait inefficace car pfSense s'arrête à la première règle correspondante. Si le "Pass" est au-dessus, le trafic est autorisé avant d'être filtré.
* **preuve** :

    ``` text
    cour@cour-VirtualBox:~$ ping -4 -I enp0s8  57.144.222.1
    PING 57.144.222.1 (57.144.222.1) from 192.168.92.12 enp0s8: 56(84) bytes of data.
    ^C
    --- 57.144.222.1 ping statistics ---
    7 packets transmitted, 0 received, 100% packet loss, time 7092ms

    cour@cour-VirtualBox:~$
    ```


### 11. Règles horaires

* **Utilité en entreprise :** Limiter l'accès aux sites non-essentiels durant les heures de production pour garantir la bande passante et la productivité.
* **Mise en place :** Horaire `Heures_Bureau` appliqué à la règle `RS` (icône horloge jaune visible).
![yoyo](image\image3.png)

### 12. Serveur web local

* **Protéger le LAN :** Même en interne, un pare-feu empêche la propagation de menaces entre machines d'un même réseau (mouvement latéral).

### 13. Logs et analyse

* **Observations :** Les logs montrent clairement les paquets bloqués (croix rouge) vers les IPs de Facebook et Pornhub.
* **Identification :** En survolant l'icône dans les logs, pfSense indique la règle exacte ayant provoqué le rejet.
![yoyo](image\image4.png)

### 14. DMZ

* **Définition :** Zone isolée accueillant des serveurs publics.
* **Isolation :** Empêche un serveur compromis d'attaquer le réseau local (LAN).
* **Flux :** LAN vers DMZ (Autorisé), DMZ vers LAN (Interdit par défaut).

### 15. Filtrage MAC

* **Sécurité :** Faible. Une adresse MAC peut être falsifiée par logiciel (**MAC Spoofing**).

### 16. Portail captif

* **Utilisation :** WiFi public ou invités. Permet d'imposer une authentification ou l'acceptation de CGU avant l'accès internet.

### 17. Sauvegarde / Restauration

* **Essentiel :** Pour restaurer le service après une panne matérielle ou annuler une configuration erronée ayant coupé le réseau.


## Conclusion

Ce TP m’a permis de configurer un pare-feu fonctionnel avec pfSense et de tester DHCP, NAT et filtrages avancés. J’ai pu appréhender les bonnes pratiques de sécurisation réseau et la logique des règles. 


---