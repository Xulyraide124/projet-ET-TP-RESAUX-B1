
---

# 📄 README – TP3 Cryptographie avec OpenSSL

**B1 Linux – TP3**
Nom : Ulysse Prévost Lacaze
Date : 13/02/2026

---

# I. Base64

## 1. Génération du fichier binaire

Commande utilisée :

```bash
dd if=/dev/urandom of=data.bin bs=1k count=100
```

Vérification taille :

```bash
ls -lh data.bin
```

**Output :**

```
cour@cour-VirtualBox:~/Documents$ dd if=/dev/urandom of=data.bin bs=1k count=100
100+0 enregistrements lus
100+0 enregistrements écrits
102400 octets (102 kB, 100 KiB) copiés, 0,00164092 s, 62,4 MB/s
cour@cour-VirtualBox:~/Documents$ ls -lh data.bin
-rw-rw-r-- 1 cour cour 100K févr. 13 11:34 data.bin
```

---

## 2. Encodage Base64

```bash
openssl base64 -in data.bin -out data.b64
```

Affichage :

```bash
cat data.b64
```

**Output (extrait de la fin) :**

```
P4KBxECZ4pJ2Ea36O1Uod3KbLe2zvF4o306cTgAApFmBt06A7PFOzzhDZqYagKV3
okT+ehwxYJRjV6F1liaiBuR5alcCDgTS/X/FSwwy/D8722mrT6Ey7jgM0sePuD1o
ASL8HCUblywPgt5KMgyh88WRS1F4cEMmRvcZGUELyNsxq1dPDgow0NjrINJiaj4y
CcKB8qQyYYwa2Qikq12NHqw2XiF0H7CqWIf+n7aklVfBL7GTZQlkPVnORQINHi3R
C8VypwAhDv5F074sgAY9BMtsdADQYBPU/v958QkrXL+rLhKBIWglnNfYPQXhsTVK
Li2K5C6WspiWdeaoEJRoMucF+6qOn+DwY9a3ogUB3ZCzH6DWl0+BtcRt5HpcfJnl
r7lxgTRyejpNpdftwSTG0jvWNu7r31HCblCYl02T3PAVObeQd7RzkyVz1rRT0vOr
1A2nUQHAjDrq5JscdEbMLgaX6I3wBZSrO7we4MzCFmnZs233woP8RJx0+9re2DWS
qqc5kDZKhSX/iqaQCMapAGXnlca1vuQLygiMbWVwXh6KASAVt60uC81djZLYfz+d
9tWsZeyU6LjOl82MPSbEvCkM5VqfRwO713Ae6P2Fhn3Nz/MkRsTih+on6YpiTStM
exjHEBNRTc40wmKgiwMNC0RrSnzT3+qzcK7U+VHFhxx0a63GAWDS+8AQV5vQroGy
7/wNU7v2KvEbff5YLRCG3AeswOnJ7P5xsvfgeLRKTSw/tw+ZEgfkVRmrCaZcmb8r
qi89gvHG1/+52nobBpkdtIVuKbLegJDbsh+RcI0z7i2VzJVc6KSBh7DUc6gEUPF0
HoygNEysc4VsJ4aopBAQFi/FW/DbTPyE9YN7Q/epe8G4Lcz9p8qGblvNitz+EHv3
1emvZQsQdLqHC8OGhV3uqC57dNxI0ZLxxJ5C5owrW5voGnGb/iF2TMYb9OVy6a8y
+WDSuIT3/SrE/arEuj6oZew/1/NbT3nV39r2D+MpinEJCVRLcfMM4TiHsC1XaOO9
xHDYdbqimQeM9kBOD9man1aV9A6N4RsB8xRWu+tk0WbHeje7cotydpY0PFE5MHZG
K+jkS3Z0m879Ri/HSYWQXQ==
```

Comparaison tailles :

```bash
ls -lh data.bin data.b64
```

**Output :**

```
-rw-rw-r-- 1 cour cour 136K févr. 13 11:34 data.b64
-rw-rw-r-- 1 cour cour 100K févr. 13 11:34 data.bin
```

Observation :
data.b64 est plus lourd que data.bin de exatement 36 kilo octé.

---

## 3. Décodage

```bash
openssl base64 -d -in data.b64 -out data_restored.bin
```

Vérification :

```bash
diff -s data.bin data_restored.bin
```




**Output :**

```bash
Les fichiers data.bin et data_restored.bin sont identiques
```

---

## 4. Réponses aux questions

1. Base64 est-il un chiffrement ?
   → Non. Base64 n’est pas un chiffrement mais un encodage.
    Il ne protège pas les données et ne nécessite aucune clé.
    Toute personne peut décoder un contenu Base64.

2. Pourquoi la taille change-t-elle ?
   → Base64 transforme des données binaires en caractères ASCII.
    Il encode 3 octets (24 bits) en 4 caractères, ce qui augmente la taille du fichier.

3. Pourcentage d’augmentation ?
   → L’augmentation est d’environ 33 %.
    Dans notre cas, le fichier passe de 100K à 136K (environ +36%).

4. Méthode rigoureuse de comparaison ?
   → Calculer et comparer une empreinte cryptographique (ex : SHA-256) :
   ```bash
   sha256sum fichier1 fichier2
   ```
   Si les hash sont identiques, les fichiers sont strictement identiques. 

---

# II. Chiffrement symétrique – AES

## 1. Création du fichier

```bash
nano confidentiel.txt
```

Contenu :

```
Ulysse Prévost Lacaze
13/02/2026

Ceci est un document confidentiel.
Il contient des informations privées.
Ne pas partager.
Linux est puissant.
Fin du message.
```

---

## 2. Chiffrement

```bash
openssl enc -aes-256-cbc -salt -pbkdf2 -md sha256 -in confidentiel.txt -out confidentiel.enc
```

Vérification type :

```bash
file confidentiel.enc
```

**Output :**

```
confidentiel.enc: openssl enc'd data with salted password

```

---

## 3. Déchiffrement

```bash
openssl enc -d -aes-256-cbc -pbkdf2 -md sha256 -in confidentiel.enc -out confidentiel_dechiffre.txt
```

Comparaison :

```bash
diff -s confidentiel.txt confidentiel_dechiffre.txt
```

**Output :**

```
Les fichiers confidentiel.txt et confidentiel_dechiffre.txt sont identiques
```

---

## 4. Rechiffrement

```bash
openssl enc -aes-256-cbc -salt -pbkdf2 -md sha256 -in confidentiel.txt -out confidentiel2.enc
```

Comparaison :

```bash
diff confidentiel.enc confidentiel2.enc
```

**Output :**

```
Les fichiers binaires confidentiel.enc et confidentiel2.enc sont différents
```

Analyse :
Les fichiers binaires confidentiel.enc et confidentiel2.enc sont différents

---

## 5. Questions

1. Pourquoi les deux fichiers chiffrés sont-ils différents ?
   → Parce qu’un sel aléatoire est utilisé lors du chiffrement.
    Même avec le même mot de passe et le même fichier, le résultat change.

2. Rôle du sel ?
   → Le sel ajoute une valeur aléatoire avant la dérivation de clé.
    Il permet :

    * d’éviter les rainbow tables
    * d’empêcher que deux fichiers identiques produisent le même chiffrement
    * de renforcer la sécurité contre les attaques par dictionnaire


3. Que se passe-t-il si une option change ?
   → Le déchiffrement échoue.
    On obtient soit une erreur, soit un fichier corrompu.
    Les paramètres doivent être strictement identiques à ceux utilisés lors du chiffrement.


4. Pourquoi PBKDF2 ?
   → PBKDF2 sert à dériver une clé forte à partir d’un mot de passe.
    Il applique plusieurs itérations de hachage, ce qui ralentit les attaques par force brute.

5. Différence encodage / chiffrement ?
   → 
    * Encodage → transforme un format (pas sécurisé)
    * Chiffrement → protège les données avec une clé

    L’encodage est réversible par tous.
    Le chiffrement est réversible uniquement avec la clé.

---

# III. Cryptographie asymétrique – RSA

## 1. Génération des clés

```bash
openssl genpkey -algorithm RSA -aes-256-cbc -out rsa_private.pem -pkeyopt rsa_keygen_bits:2048
```

```bash
openssl rsa -in rsa_private.pem -pubout -out rsa_public.pem
```

Affichage clé privée :

```bash
openssl rsa -in rsa_private.pem -text -noout
```

**Output (extrait) :**

```
coefficient:
    2b:a6:40:ab:8c:b6:b6:8b:40:76:cb:b5:81:71:b9:
    75:29:3e:67:f5:df:23:09:d9:26:bf:a0:d4:68:f6:
    58:76:ec:78:9d:14:74:4a:7b:8f:cd:b6:28:99:6b:
    9e:4a:a6:1f:f7:b8:69:ab:17:dc:ab:9e:a0:e1:1e:
    dd:d2:f9:94:d0:a6:c5:40:6e:64:5b:86:16:ff:64:
    12:e2:da:f4:73:4d:c9:4d:68:cd:4c:fe:7e:8e:7b:
    af:45:fb:64:75:68:c1:b3:d0:de:9b:c7:3a:50:82:
    54:e2:ff:8b:16:bb:e7:87:59:ce:f8:61:fd:61:2a:
    9e:61:4e:3a:10:27:f7:bc
```

Affichage clé publique :

```bash
openssl rsa -pubin -in rsa_public.pem -text -noout
```

**Output :**

```
Public-Key: (2048 bit)
Modulus:
    00:bb:10:68:3e:fc:62:b5:d6:64:8f:88:1c:5c:dd:
    2a:23:a3:4a:94:b5:ce:d3:8d:f1:59:1d:d2:dc:de:
    2e:3e:1c:59:47:8f:8f:76:1e:38:41:b9:6a:f6:07:
    fa:b8:cb:a5:91:18:0e:aa:78:40:f4:fa:69:78:a8:
    36:02:0c:b8:87:e3:69:49:aa:d1:dd:11:77:e1:5d:
    9c:7f:12:64:c0:22:18:a1:21:dc:59:c7:d8:ba:37:
    7f:1b:8c:81:0a:c9:67:4f:dd:22:e6:b8:55:f6:e3:
    0a:d7:9e:91:e6:22:d2:15:ee:a0:a4:38:a7:72:44:
    74:2d:77:9f:48:a8:ac:0c:b8:76:28:6a:3b:39:55:
    c4:8f:8f:66:ab:ae:bf:b0:41:2e:5e:00:0e:88:ac:
    74:cb:31:85:b2:02:fe:34:78:f1:fe:ab:2d:18:bd:
    0b:a3:b3:6a:ad:67:a5:fe:4f:c0:a3:38:39:81:72:
    d1:ae:7f:07:cf:a9:94:62:31:29:09:87:14:e0:fa:
    97:3d:78:66:cc:a2:fa:a0:3f:77:99:88:6e:51:13:
    11:ca:ca:46:de:4c:2e:a8:4d:92:0d:45:6b:e1:be:
    e9:90:1e:51:30:8d:25:6a:a3:33:cb:14:24:02:2a:
    f9:58:45:f0:d2:90:34:75:ac:3b:df:b2:a0:b9:4c:
    71:d3

```

Comparaison : Les clés publiques et privées contiennent des informations liées (modulus n), mais la clé privée contient des valeurs secrètes (exposant privé, p et q) qui permettent de déchiffrer ou signer.

---

## 2. Chiffrement

```bash
openssl pkeyutl -encrypt -in secret.txt -pubin -inkey rsa_public.pem -out secret.enc
```

Déchiffrement :

```bash
openssl pkeyutl -decrypt -in secret.enc -inkey rsa_private.pem -out secret_dechiffre.txt
```

Vérification :

```bash
diff -s secret.txt secret_dechiffre.txt
```

**Output :**

```

cour@cour-VirtualBox:~/Documents$ openssl pkeyutl -decrypt -in secret.enc -inkey rsa_private.pem -out secret_dechiffre.txt
Enter pass phrase for rsa_private.pem:

```

---

## 3. Questions

1. Pourquoi la clé privée ne doit-elle jamais être partagée ?
   → Parce qu’elle permet :

    * de déchiffrer les données
    * de signer des documents
    Si elle est compromise, toute la sécurité est perdue.


2. Pourquoi RSA n’est-il pas adapté aux gros fichiers ?
   → RSA est lent et limité en taille.
    Il ne peut chiffrer que des petites données (inférieures à la taille de la clé).
    Il est donc inefficace pour les fichiers volumineux.

3. Différences clé publique / privée ?
   → Clé publique :

    * Modulo (n)
    * Exposant public (e)

    Clé privée :

    * Modulo (n)
    * Exposant public (e)
    * Exposant privé (d)
    * Nombres premiers (p et q)
    * Paramètres CRT

    La clé privée contient des éléments secrets absents de la clé publique.


4. Rôle du modulo ?
   → Le modulo (n = p × q) est la base mathématique du système RSA.
    Toutes les opérations de chiffrement et déchiffrement sont faites modulo n.

5. Pourquoi RSA chiffre une clé AES et pas le document entier ?
   → Parce que RSA est lent.
    On utilise un chiffrement hybride :

    * AES → chiffre rapidement les données
    * RSA → chiffre la clé AES

    Cela combine performance et sécurité.

---

# IV. Signature numérique

## 1. Signature

```bash
openssl dgst -sha256 -sign rsa_private.pem -out contrat.sig contrat.txt
```

---

## 2. Vérification

```bash
openssl dgst -sha256 -verify rsa_public.pem -signature contrat.sig contrat.txt
```

**Output :**

```
Verified OK
```

Après modification du fichier :

**Output :**

```
Verification Failure / Verified KO
```

---

## 3. Questions

1. Que se passe-t-il après modification ?
   → La vérification échoue.
    La signature devient invalide.

2. Pourquoi ?
   → Parce que la signature est basée sur le hash du fichier.
    Si le fichier change, son hash change, donc la signature ne correspond plus.

3. Rôle du hachage ?
   → Le hachage permet de signer une empreinte courte plutôt que le fichier complet.
    Cela rend la signature plus rapide et efficace.

4. Différence signature / chiffrement ?
   → Signature :

    * Garantit l’authenticité
    * Utilise la clé privée pour signer
    * Vérifiée avec la clé publique

    Chiffrement :

    * Garantit la confidentialité
    * Chiffré avec clé publique (ou mot de passe)
    * Déchiffré avec clé privée

---


