# Création certificat TLS/SSL

## 1. Préparation de l'environnement

> Avant de commencer la création des certificats, nous devons installer les outils nécessaires à la gestion de notre infrastructure PKI (Public Key Infrastructure). Nous utiliserons OpenSSL pour générer les clés privées, créer les demandes de signature de certificats (CSR) et générer les certificats X.509. L'outil tree permettra quant à lui de visualiser l'organisation des différents répertoires.

### 1.1. Mise à jour des paquets

- Mettez à jour la liste des paquets disponibles :

```
sudo apt update
```

> Cette commande actualise la liste des paquets disponibles dans les dépôts configurés sur le système.

### 1.2. Installation des outils

> Installez OpenSSL et tree :

```
sudo apt install openssl tree
```

> Explication des commandes :

- `apt` : permet de gérer les paquets logiciels sur les distributions Linux basées sur Debian.
- `install` : permet d'installer les paquets demandés.
- `openssl` : fournit les outils nécessaires à la gestion des certificats et des clés cryptographiques.
- `tree` : permet d'afficher l'arborescence des fichiers et des répertoires.

## 2. Création de l'arborescence PKI

> Nous allons maintenant créer une structure de répertoires permettant de séparer les éléments de l'autorité de certification de ceux du serveur Web. Cette organisation permet de distinguer les certificats publics, les clés privées et les demandes de signature de certificats.

### 2.1. Création du répertoire principal

> Créez le répertoire principal de notre infrastructure PKI :

```
sudo mkdir -p ~/pki-group
```

> Ce répertoire contiendra l'ensemble des éléments nécessaires à la gestion des certificats de notre infrastructure SportLudique.

### 2.2. Création des répertoires de l'autorité de certification

> Créez le répertoire principal de l'autorité :

```
sudo mkdir -p ~/pki-group/autorite
```

> Créez le répertoire destiné aux certificats :

```
sudo mkdir -p ~/pki-group/autorite/certificats
```

> Créez le répertoire destiné aux clés privées :

```
sudo mkdir -p ~/pki-group/autorite/cles
```

> Créez le répertoire destiné aux demandes de signature :

```
sudo mkdir -p ~/pki-group/autorite/demandes
```

### 2.3. Création des répertoires du serveur Web

> Créez le répertoire principal du serveur :

```
sudo mkdir -p ~/pki-group/serveur
```

> Créez le répertoire destiné aux certificats :

```
sudo mkdir -p ~/pki-group/serveur/certificats
```

> Créez le répertoire destiné aux clés privées :

```
sudo mkdir -p ~/pki-group/serveur/cles
```

> Créez le répertoire destiné aux demandes de signature :

```
sudo mkdir -p ~/pki-group/serveur/demandes
```

> Explication des commandes :

- `mkdir` : permet de créer un répertoire.
- `-p` : permet de créer les répertoires parents s'ils n'existent pas déjà.
- `~/` : représente le répertoire personnel de l'utilisateur.
- `autorite/` : contient les éléments associés à l'autorité de certification.
- `serveur/` : contient les éléments associés au serveur Web.
- `certificats/` : stocke les certificats publics.
- `cles/` : stocke les clés privées.
- `demandes/` : stocke les demandes de signature de certificats (CSR).

> Important : les clés privées de l'autorité de certification doivent être protégées et ne doivent jamais être distribuées au serveur Web ou aux utilisateurs.

## 3. Création de la clé privée de l'autorité de certification

> Nous allons maintenant générer la clé privée de notre autorité de certification, appelée ca-tours. Cette clé permettra notamment de signer les certificats des serveurs de notre infrastructure. Elle sera protégée par une phrase secrète afin de limiter les risques en cas d'accès non autorisé au fichier.

### 3.1. Génération de la clé privée

> Placez-vous dans le répertoire pki-group :

```
cd ~/pki-group
```

> Générez la clé privée de l'autorité de certification :

```
sudo openssl genpkey -algorithm RSA -aes-256-cbc \
    -pkeyopt rsa_keygen_bits:4096 \
    -out autorite/cles/ca-tours.key
```

> Explication de la commande :

- `openssl genpkey` : permet de générer une nouvelle clé privée.
- `-algorithm RSA` : indique que l'algorithme cryptographique utilisé est RSA.
- `-aes-256-cbc` : chiffre la clé privée à l'aide de l'algorithme AES-256-CBC.
- `-pkeyopt rsa_keygen_bits:4096` : définit la taille de la clé RSA à 4096 bits.
- `-out` : indique le chemin et le nom du fichier dans lequel sera enregistrée la clé privée.
- `ca-tours.key` : correspond au fichier contenant la clé privée de l'autorité.

> Lors de l'exécution de la commande, OpenSSL demande de saisir une phrase secrète afin de protéger la clé privée.Cette phrase devra être saisie à nouveau lors des opérations nécessitant l'utilisation de cette clé.

### 3.2. Vérification de la présence de la clé privée

> Vérifiez que le fichier a bien été créé :

```
ls -l autorite/cles/ca-tours.key
```

> Explication de la commande :

- `ls` : permet d'afficher le contenu d'un répertoire ou les informations d'un fichier.
- `-l` : affiche les informations détaillées, notamment les permissions, le propriétaire, le groupe et la taille du fichier.

> Le fichier ca-tours.key doit être présent dans le répertoire autorite/cles/.

### 3.3. Vérification de la validité de la clé privée

> Vérifiez que la clé privée est valide :

```
sudo openssl pkey \
    -in autorite/cles/ca-tours.key \
    -noout \
    -check
```

> Explication de la commande :

- `openssl pkey` : permet de manipuler et de vérifier les clés privées et publiques.
- `-in` : indique le fichier contenant la clé à vérifier.
- `-noout` : empêche l'affichage de la clé dans le terminal.
- `-check` : demande à OpenSSL de vérifier la cohérence de la clé.

- Le résultat attendu est :

```
Key is valid
```

> Ce résultat confirme que la clé privée RSA a été correctement générée et qu'elle est exploitable.

## 4. Création du certificat de l'autorité de certification

> Maintenant que la clé privée de notre autorité est créée, nous allons générer son certificat. Ce certificat sera auto-signé : l'autorité de certification signe elle-même son propre certificat à l'aide de sa clé privée.

### 4.1. Génération du certificat auto-signé

> Exécutez la commande suivante :

```
sudo openssl req -new -x509 -days 365 \
    -key autorite/cles/ca-tours.key \
    -out autorite/certificats/ca-tours.crt
```

> Explication de la commande :

- `openssl req` : permet de créer et de gérer des demandes de certificats ainsi que des certificats auto-signés.
- `-new` : indique la création d'un nouvel objet de certification.
- `-x509` : demande la génération directe d'un certificat X.509 auto-signé.
- `-days 365` : définit la durée de validité du certificat à 365 jours.
- `-key` : indique la clé privée utilisée pour signer le certificat.
- `-out` : indique le fichier de sortie contenant le certificat généré.

> OpenSSL demande ensuite plusieurs informations relatives à l'identité de l'autorité de certification :

- Pays : FR.
- Région : France.
- Ville : Tours.
- Organisation : tours.sportludique.
- Nom commun (Common Name) : trs.

> Le certificat obtenu est auto-signé. Il constitue une autorité de confiance uniquement pour les systèmes sur lesquels son certificat public est installé et reconnu.

### 4.2. Vérification des informations du certificat

> Affichez les informations principales du certificat :

```
openssl x509 \
    -in autorite/certificats/ca-tours.crt \
    -noout \
    -subject \
    -issuer \
    -dates
```

> Explication de la commande :

- `openssl x509` : permet de consulter et de manipuler les certificats au format X.509.
- `-in` : indique le fichier contenant le certificat.
- `-noout` : empêche l'affichage du certificat complet.
- `-subject` : affiche l'identité du propriétaire du certificat.
- `-issuer` : affiche l'identité de l'autorité ayant signé le certificat.
- `-dates` : affiche les dates de début et de fin de validité.

> Pour un certificat auto-signé, les informations subject et issuer doivent correspondre, car le certificat a été signé par sa propre autorité.

### 4.3. Affichage détaillé du certificat

> Affichez l'ensemble des informations du certificat :

```
openssl x509 \
    -in autorite/certificats/ca-tours.crt \
    -noout \
    -text
```

> Cette commande affiche les informations détaillées du certificat, notamment :

- La version du certificat.
- Le numéro de série.
- L'algorithme de signature.
- L'identité du propriétaire.
- L'identité de l'émetteur.
- Les dates de validité.
- La clé publique.
- Les extensions X.509.

## 5. Génération de la clé privée du serveur Web

> Nous allons maintenant créer la clé privée qui sera utilisée par le serveur Web SportLudique. Cette clé sera différente de celle de l'autorité de certification. Elle permettra au serveur Web de prouver son identité lors des connexions HTTPS.

### 5.1. Génération de la clé privée

> Exécutez la commande suivante :

```
sudo openssl genpkey -algorithm RSA \
    -pkeyopt rsa_keygen_bits:3072 \
    -out serveur/cles/web.key
```

> Explication de la commande :

- `openssl genpkey` : permet de générer une nouvelle clé privée.
- `-algorithm RSA` : définit l'utilisation de l'algorithme RSA.
- `-pkeyopt rsa_keygen_bits:3072` : définit la taille de la clé RSA à 3072 bits.
- `-out` : indique le fichier dans lequel sera enregistrée la clé privée.
- `web.key` : correspond à la clé privée du serveur Web.

> Contrairement à la clé privée de l'autorité, cette clé n'est pas chiffrée par une phrase secrète. Cela permet notamment au serveur Apache de démarrer automatiquement sans demander de mot de passe. Elle doit cependant être protégée par des permissions strictes.

## 6. Création de la demande de signature du serveur Web (CSR)

> Une fois la clé privée du serveur créée, nous allons générer une demande de signature de certificat (CSR). Cette demande contient les informations d'identification du serveur ainsi que sa clé publique. Elle sera transmise à l'autorité de certification afin qu'elle puisse signer le certificat serveur.

### 6.1. Génération de la demande CSR

> Exécutez la commande suivante :

```
sudo openssl req -new -sha256 \
    -key serveur/cles/web.key \
    -out serveur/demandes/web.csr \
    -addext "subjectAltName=DNS:www.tours.sportludique.fr,DNS:tours.sportludique.fr"
```

> Explication de la commande :

- `openssl req` : permet de créer une demande de signature de certificat.
- `-new` : indique la création d'une nouvelle demande CSR.
- `-sha256` : définit l'algorithme de hachage SHA-256 pour la signature de la demande.
- `-key` : indique la clé privée du serveur Web utilisée pour créer la demande.
- `-out` : indique le fichier dans lequel sera enregistrée la demande CSR.
- `-addext` : permet d'ajouter une extension au certificat demandé.
- `subjectAltName` : définit les noms alternatifs du serveur.
- `DNS:www.tours.sportludique.fr` : ajoute le nom DNS principal du site.
- `DNS:tours.sportludique.fr` : ajoute le nom DNS secondaire du site.

> L'extension SAN (Subject Alternative Name) permet d'associer plusieurs noms DNS à un même certificat. Elle est indispensable pour que les navigateurs puissent vérifier que le certificat correspond bien au nom de domaine utilisé lors de la connexion HTTPS.

### 6.2. Vérification de la demande CSR

> Affichez les informations contenues dans la demande :

```
openssl req \
    -in serveur/demandes/web.csr \
    -noout \
    -text
```

> Explication de la commande :

- `openssl req` : permet de consulter une demande de signature de certificat.
- `-in` : indique le fichier contenant la demande CSR.
- `-noout` : empêche l'affichage du contenu brut de la demande.
- `-text` : affiche les informations détaillées de la demande.

> Vérifiez notamment la présence de l'extension Subject Alternative Name :

```
X509v3 Subject Alternative Name:
    DNS:www.tours.sportludique.fr, DNS:tours.sportludique.fr
```

> Cette vérification permet de confirmer que les noms DNS nécessaires ont bien été intégrés à la demande de signature.

## 7. Signature du certificat serveur par l'autorité de certification

> Nous allons maintenant utiliser l'autorité de certification ca-tours pour signer la demande CSR du serveur Web. Cette opération permet de transformer la demande CSR en un véritable certificat X.509 signé par notre autorité de certification. Le certificat généré pourra ensuite être utilisé par Apache pour établir des connexions HTTPS.

### 7.1. Génération du certificat serveur signé

> Exécutez la commande suivante depuis le répertoire pki-group :

```
sudo openssl x509 -req -days 365 \
    -in serveur/demandes/web.csr \
    -CA autorite/certificats/ca-tours.crt \
    -CAkey autorite/cles/ca-tours.key \
    -CAserial autorite/ca.srl \
    -CAcreateserial \
    -out serveur/certificats/web.crt \
    -copy_extensions copy
```

> Explication de la commande :

- `openssl x509` : permet de générer et de manipuler des certificats X.509.
- `-req` : indique que la source est une demande de signature de certificat (CSR).
- `-days 365` : définit la durée de validité du certificat serveur à 365 jours.
- `-in` : indique le fichier contenant la demande CSR du serveur Web.
- `-CA` : indique le certificat de l'autorité qui va signer le certificat serveur.
- `-CAkey` : indique la clé privée de l'autorité utilisée pour effectuer la signature.
- `-CAserial` : indique le fichier contenant le numéro de série des certificats signés.
- `-CAcreateserial` : permet de créer automatiquement le fichier de numéro de série s'il n'existe pas.
- `-out` : indique le chemin et le nom du certificat serveur généré.
- `-copy_extensions copy` : permet de recopier les extensions présentes dans la demande CSR vers le certificat final, notamment les extensions SAN.

> Attention : cette opération nécessite l'utilisation de la clé privée de l'autorité de certification. La phrase secrète de ca-tours.key sera donc demandée. La clé privée de l'autorité doit rester protégée et ne doit jamais être copiée sur le serveur Web.

### 7.2. Vérification des informations du certificat serveur

> Affichez les informations principales du certificat :

```
openssl x509 \
    -in serveur/certificats/web.crt \
    -noout \
    -subject \
    -issuer \
    -dates \
    -ext subjectAltName
```

> Explication de la commande :

- `openssl x509` : permet de consulter un certificat X.509.
- `-in` : indique le certificat serveur à consulter.
- `-noout` : empêche l'affichage du certificat complet.
- `-subject` : affiche l'identité du serveur.
- `-issuer` : affiche l'identité de l'autorité ayant signé le certificat.
- `-dates` : affiche les dates de validité du certificat.
- `-ext subjectAltName` : affiche uniquement l'extension SAN du certificat.

> Le résultat doit notamment afficher :

- Le nom du serveur Web dans le champ subject.
- Le nom de l'autorité de certification dans le champ issuer.
- Une durée de validité de 365 jours.
- Les deux noms DNS dans l'extension SAN.

### 7.3. Vérification de la signature du certificat

> Vérifiez que le certificat serveur est bien signé par l'autorité :

```
openssl verify \
    -CAfile autorite/certificats/ca-tours.crt \
    serveur/certificats/web.crt
```

> Explication de la commande :

- `openssl verify` : permet de vérifier la validité d'un certificat.
- `-CAfile` : indique le fichier contenant le certificat de l'autorité de confiance.
- `serveur/certificats/web.crt` : correspond au certificat serveur à vérifier.

> Le résultat attendu est :

```
serveur/certificats/web.crt: OK
```

> Ce résultat indique que le certificat serveur a été correctement signé par l'autorité de certification fournie et que la vérification de sa signature est réussie.

## 8. Création d'une demande CSR pour l'autorité de certification

> Dans le cadre de notre infrastructure, nous pouvons également créer une demande de signature de certificat pour l'autorité de certification. Cette demande permet notamment de solliciter la signature du certificat de l'autorité par une autorité supérieure, par exemple une autorité racine. Cette opération est différente de la génération d'un certificat auto-signé réalisée précédemment.

### 8.1. Génération de la demande CSR de l'autorité

> Exécutez la commande suivante :

```
sudo openssl req -new -sha256 \
    -key autorite/cles/ca-tours.key \
    -out autorite/demandes/ca-tours.csr \
    -subj "/C=FR/O=SportLudique/CN=CA tours SportLudique"
```

> Explication de la commande :

- `openssl req` : permet de créer une demande de signature de certificat.
- `-new` : indique la création d'une nouvelle demande CSR.
- `-sha256` : définit l'algorithme de hachage SHA-256.
- `-key` : indique la clé privée de l'autorité de certification.
- `-out` : indique le fichier de sortie contenant la demande CSR.
- `-subj` : permet de définir directement les informations d'identification de l'autorité sans passer par les questions interactives.
- `/C=FR` : définit le pays.
- `/O=SportLudique` : définit le nom de l'organisation.
- `/CN=CA tours SportLudique` : définit le nom commun de l'autorité de certification.

> OpenSSL demande la phrase secrète associée à la clé privée ca-tours.key. La demande CSR obtenue pourra être transmise à une autorité supérieure afin qu'elle signe le certificat de notre autorité.

## 9. Transfert des demandes et certificats entre les machines

> Nous allons maintenant transférer les différents fichiers nécessaires entre le poste de travail et la machine dans le réseau prof. Pour cela, nous utiliserons la commande scp (Secure Copy Protocol), qui permet de transférer des fichiers entre deux machines à travers une connexion SSH.

> Les transferts seront effectués à l'aide du compte utilisateur morgan.

### 9.1. Transfert de la demande CSR de l'autorité vers le poste de travail

> Depuis le poste de travail, récupérez la demande CSR présente sur la machine distante :

```
scp morgan@10.0.130.112:~/pki-group/autorite/demandes/ca-tours.csr \
    /home/fontaine/Documents/
```

> Explication de la commande :

- `scp` : permet de copier des fichiers entre deux machines via SSH.
- `morgan` : correspond au nom de l'utilisateur distant.
- `10.0.130.112` : correspond à l'adresse IP de la machine distante.
- `~/pki-group/autorite/demandes/ca-tours.csr` : indique le chemin du fichier à récupérer sur la machine distante.
- `/home/fontaine/Documents/` : indique le répertoire de destination sur le poste local.

> Cette commande permet de récupérer la demande CSR de l'autorité de certification afin de la transmettre à l'infrastructure de certification supérieure.

### 9.2. Transfert du certificat racine vers la machine réseau Mery

> Envoyez le certificat racine vers la machine distante :

```
scp /home/fontaine/Téléchargements/sportludique-root.crt.pem \
    morgan@10.0.130.112:/home/morgan/
```

> Explication de la commande :

- `scp` : permet de transférer un fichier via SSH.
- `/home/fontaine/Téléchargements/sportludique-root.crt.pem` : correspond au chemin local du certificat racine.
- `morgan@10.0.130.112` : désigne l'utilisateur et l'adresse IP de la machine distante.
- `/home/morgan/` : indique le répertoire dans lequel le certificat sera enregistré sur la machine distante.

> Ce transfert permet de mettre à disposition le certificat public de l'autorité racine sur la machine dans le réseau prof. Le certificat racine est un certificat public. Il peut être distribué aux machines qui doivent reconnaître cette autorité comme étant digne de confiance.

### 9.3. Transfert du certificat intermédiaire vers la machine réseau Mery

> Envoyez le certificat intermédiaire vers la machine distante :

```
scp /home/fontaine/Documents/ca-intermediaire.crt \
    morgan@10.0.130.112:/home/morgan/
```

> Explication de la commande :

- `scp` : permet de transférer un fichier entre deux machines.
- `/home/fontaine/Documents/ca-intermediaire.crt` : correspond au chemin local du certificat intermédiaire.
- `morgan@10.0.130.112` : désigne l'utilisateur et l'adresse IP de destination.
- `/home/morgan/` : indique le répertoire de destination sur la machine distante.

> Ce transfert permet de mettre à disposition le certificat de l'autorité intermédiaire sur la machine dans le réseau prof. Le certificat intermédiaire permettra notamment de construire la chaîne de confiance entre le certificat serveur et le certificat racine.

> Attention : seuls les certificats publics doivent être transférés entre les machines. Les clés privées de l'autorité racine et de l'autorité intermédiaire ne doivent jamais être distribuées au serveur Web ou aux utilisateurs.

## 10\. Vérification de l'arborescence finale

> Nous allons enfin vérifier que tous les répertoires et fichiers ont été correctement créés. Cette dernière étape permet de contrôler l'organisation de notre infrastructure PKI avant de poursuivre la configuration HTTPS du serveur Web.

### 10.1. Affichage de l'arborescence

> Placez-vous dans le répertoire principal :

```
cd ~/pki-group
```

> Affichez l'ensemble des répertoires et des fichiers :

```
tree
```

> Explication des commandes :

- `tree` : affiche l'arborescence complète du répertoire courant.
- `cd` : permet de changer de répertoire.

> L'arborescence doit notamment contenir les éléments suivants :

```
pki-group/
├── autorite/
│   ├── certificats/
│   │   └── ca-tours.crt
│   ├── cles/
│   │   └── ca-tours.key
│   ├── demandes/
│   │   └── ca-tours.csr
│   └── ca.srl
│
└── serveur/
    ├── certificats/
    │   └── web.crt
    ├── cles/
    │   └── web.key
    └── demandes/
        └── web.csr
```

> Le fichier ca.srl est créé automatiquement lors de la première signature d'un certificat avec l'option -CAcreateserial. Il permet de conserver le numéro de série utilisé pour les certificats signés par l'autorité.

## 11\. Résultat attendu

> À la fin de cette procédure, l'infrastructure PKI de SportLudique doit disposer des éléments nécessaires à la gestion des certificats de l'autorité et du serveur Web.

### 11.1. Vérification des éléments créés

> À la fin de cette procédure, vous devez disposer des éléments suivants :

- La clé privée de l'autorité de certification ca-tours.key.
- Le certificat auto-signé de l'autorité ca-tours.crt.
- La demande CSR de l'autorité ca-tours.csr.
- La clé privée du serveur Web web.key.
- La demande CSR du serveur Web web.csr.
- Le certificat serveur signé web.crt.
- Le fichier de numéro de série ca.srl.
- Le certificat racine sportludique-root.crt.pem.
- Le certificat intermédiaire ca-intermediaire.crt.

### 11.2. Points de vigilance

> Plusieurs éléments doivent être vérifiés afin de garantir le bon fonctionnement de notre infrastructure de certification :

- Protection des clés privées : les fichiers .key doivent être protégés et ne doivent pas être distribués.
- Validité des certificats : vérifier les dates de début et de fin de validité.
- Extension SAN : vérifier que les noms DNS du serveur sont présents dans le certificat final.
- Signature : vérifier que le certificat serveur est bien signé par l'autorité attendue.
- Chaîne de confiance : vérifier que le certificat intermédiaire est bien signé par l'autorité racine.
- Transfert des fichiers : vérifier que les certificats publics ont été correctement transférés vers la machine réseau Mery.

> Une fois ces vérifications effectuées, les certificats et les clés nécessaires à la mise en place de HTTPS sont disponibles.

> Nous pourrons ensuite procéder à l'installation du certificat serveur et de sa chaîne de confiance sur Apache afin de sécuriser les connexions au site SportLudique.