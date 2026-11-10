# Mise en place de HTTPS sur le site Web SportLudique

## 1. Préparation des certificats

> Pour activer le protocole HTTPS sur le site SportLudique, le serveur Web doit posséder un certificat SSL/TLS signé par une autorité de certification (CA). Dans notre infrastructure, le certificat du serveur Web est signé par l'autorité de certification intermédiaire de Tours, elle-même rattachée à l'autorité racine.

### La chaîne de confiance est donc composée de trois éléments :

```
- Certificat racine : sportludique-root.crt.pem
- Certificat intermédiaire : ca-intermediaire.crt
- Certificat serveur : web.crt
```

> Le navigateur doit pouvoir vérifier cette chaîne afin de confirmer l'identité du serveur Web.

## 1.1. Génération du certificat serveur

> L'ancien certificat web.crt étant auto-signé, il doit être remplacé par un certificat signé par l'autorité de certification intermédiaire. La demande de signature du certificat (web.csr) ayant déjà été créée, nous allons utiliser celle-ci pour générer le nouveau certificat.

- Exécutez la commande suivante depuis le répertoire contenant les dossiers serveur et autorite :

```
sudo openssl x509 -req -days 365 \
    -in serveur/demandes/web
    -CAserial autorite/ca.srl \.csr \
    -CA autorite/certificats/ca-intermediaire.crt \
    -CAkey autorite/cles/ca-tours.key \
    -CAserial autorite/ca.srl \
    -CAcreateserial \
    -out serveur/certificats/web.crt \
    -copy_extensions copy
```

### Explication de la commande

- ` openssl x509 :` utilise OpenSSL pour générer un certificat X.509.
- ` req :` indique que la source est une demande de signature de certificat (CSR).
- ` days 365 :` définit la durée de validité du certificat à 365 jours.
- ` in :` indique le fichier contenant la demande de signature du certificat.
- ` CA :` indique le certificat de l'autorité de certification qui va signer le certificat serveur.
- ` CAkey :` indique la clé privée de l'autorité de certification intermédiaire.
- ` CAserial :` indique le fichier contenant le numéro de série utilisé pour les certificats signés.
- ` CAcreateserial :` crée le fichier de numéro de série s'il n'existe pas.
- ` out :` indique le chemin et le nom du certificat généré.
- ` copy_extensions copy :` copie les extensions présentes dans la demande CSR vers le certificat généré, notamment les extensions SAN si elles sont présentes.

> Attention : la clé privée de l'autorité intermédiaire (ca-tours.key) doit rester protégée. Elle ne doit jamais être copiée sur le serveur Web ni rendue accessible aux utilisateurs.

## 1.2. Vérification du certificat généré

- Vérifiez les informations du certificat serveur : ` openssl x509 -in serveur/certificats/web.crt -noout -text`

### Cette commande affiche les informations du certificat, notamment :

- Le nom du propriétaire du certificat.
- L'autorité de certification qui l'a signé.
- La date de début et de fin de validité.
- Le numéro de série.
- Les noms DNS autorisés (SAN).
- Vérifiez que le certificat est bien signé par l'autorité intermédiaire :

```
openssl verify \
    -CAfile autorite/certificats/ca-intermediaire.crt \
    serveur/certificats/web.crt
```

> Cette vérification utilise le certificat intermédiaire comme autorité de confiance. Pour vérifier toute la chaîne de certification, utilisez le certificat racine et le certificat intermédiaire :

```
openssl verify \
    -CAfile sportludique-root.crt.pem \
    -untrusted autorite/certificats/ca-intermediaire.crt \
    serveur/certificats/web.crt
```

## 2. Création de la chaîne de confiance

> Le serveur Web doit présenter son certificat ainsi que le certificat de l'autorité intermédiaire afin de permettre aux navigateurs de vérifier la chaîne de certification. Nous allons donc créer un fichier fullchain.pem.

- Placez-vous à la racine du répertoire contenant les dossiers serveur et autorite. Exécutez la commande suivante :

```
cat serveur/certificats/web.crt \
    autorite/certificats/ca-intermediaire.crt \
    > serveur/certificats/fullchain.pem
```

### Explication de la commande

- `cat :` affiche ou concatène le contenu de plusieurs fichiers.
- `web.crt :` correspond au certificat du serveur Web.
- `ca-intermediaire.crt :` correspond au certificat de l'autorité intermédiaire.

- `fullchain.pem :` contient le certificat serveur suivi du certificat intermédiaire.

> Important : le certificat racine n'est généralement pas ajouté au fichier fullchain.pem. Il doit être installé dans le magasin de certificats de confiance du navigateur ou du système client.

> `serveur/certificats/fullchain.pem :` redirige le résultat vers un fichier.

## 3. Installation des certificats sur le serveur Web

> Nous allons maintenant créer les dossiers destinés à accueillir les certificats et la clé privée sur le serveur Apache. Ces fichiers seront stockés dans des répertoires spécifiques afin de séparer les certificats publics de la clé privée du serveur Web.

### 3.1. Création des dossiers de certificats

- Créez le dossier destiné aux certificats publics :

```
sudo mkdir -p /etc/ssl/sportludique/certs
```

- Créez le dossier destiné aux clés privées :

```
sudo mkdir -p /etc/ssl/sportludique/private
```

### Explication des commandes

- `mkdir :` permet de créer un répertoire.
- `-p :` permet de créer les répertoires parents s'ils n'existent pas et évite une erreur si le dossier existe déjà.
- `/etc/ssl/sportludique/certs :` contiendra les certificats publics du serveur Web.
- `/etc/ssl/sportludique/private :` contiendra la clé privée du serveur Web.

### 3.2. Copie du certificat serveur

> Nous allons maintenant copier le certificat serveur dans le répertoire prévu à cet effet.

- Exécutez la commande suivante depuis le répertoire contenant le dossier `serveur` :

```
sudo cp serveur/certificats/web.crt \
    /etc/ssl/sportludique/certs/web.crt
```

> Cette commande copie le certificat serveur `web.crt` dans le répertoire des certificats publics d'Apache.

### 3.3. Copie de la chaîne de confiance

> Le serveur Web doit également disposer du fichier `fullchain.pem`, qui contient le certificat serveur suivi du certificat de l'autorité intermédiaire. Nous allons donc copier ce fichier dans le répertoire des certificats.

- Exécutez la commande suivante :

```
sudo cp serveur/certificats/fullchain.pem \
    /etc/ssl/sportludique/certs/fullchain.pem
```

### Explication de la commande

- `cp :` permet de copier un fichier.
- `serveur/certificats/fullchain.pem :` correspond au fichier contenant la chaîne de certification.
- `/etc/ssl/sportludique/certs/fullchain.pem :` indique le chemin de destination du fichier sur le serveur Apache.

> Important : le fichier `fullchain.pem` contient le certificat serveur et le certificat intermédiaire. Le certificat racine n'est généralement pas ajouté à cette chaîne, car il doit être installé dans le magasin de certificats de confiance des clients.

### 3.4. Copie de la clé privée

> Nous allons maintenant installer la clé privée correspondant au certificat serveur. Cette clé doit être stockée dans un répertoire sécurisé et accessible uniquement aux utilisateurs autorisés.

- Exécutez la commande suivante :

```
sudo cp serveur/cles/web.key \
    /etc/ssl/sportludique/private/web.key
```

> Si votre clé privée est stockée dans un autre répertoire, adaptez le chemin source de la commande.

- Modifiez le propriétaire de la clé privée :

```
sudo chown root:root /etc/ssl/sportludique/private/web.key
```

- Protégez les permissions du fichier :

```
sudo chmod 600 /etc/ssl/sportludique/private/web.key
```

### Explication des commandes

- `chown root:root :` attribue le fichier à l'utilisateur `root` et au groupe `root`.
- `chmod 600 :` autorise uniquement le propriétaire du fichier à lire et à modifier son contenu.
- `/etc/ssl/sportludique/private/web.key :` correspond au chemin de la clé privée du serveur Web.

> Attention : la clé privée `web.key` est un élément sensible de l'infrastructure. Elle ne doit jamais être communiquée aux utilisateurs, copiée dans un répertoire accessible publiquement ou distribuée aux clients.

## 4. Activation du module SSL d'Apache

> Pour permettre à Apache de gérer les connexions HTTPS, nous devons activer son module SSL. Ce module permet notamment d'utiliser les directives de configuration nécessaires à la mise en place du chiffrement TLS.

### 4.1. Activation du module SSL

- Exécutez la commande suivante :

```
sudo a2enmod ssl
```

### 4.2. Vérification du module SSL

- Vérifiez que le module SSL est correctement chargé :

```
sudo apache2ctl -M | grep ssl
```

### Explication des commandes

- `a2enmod ssl :` active le module SSL d'Apache.
- `apache2ctl -M :` affiche la liste des modules Apache actuellement chargés.
- `grep ssl :` filtre les résultats afin d'afficher uniquement les lignes contenant le terme `ssl`.

> Le module SSL permet notamment d'utiliser les directives suivantes :
>
> - `SSLEngine :` active le chiffrement SSL/TLS.
> - `SSLCertificateFile :` indique le chemin du certificat serveur.
> - `SSLCertificateKeyFile :` indique le chemin de la clé privée correspondante.

## 5. Configuration du VirtualHost HTTPS

> Nous allons maintenant créer un VirtualHost dédié aux connexions HTTPS. Contrairement au VirtualHost HTTP, qui écoute sur le port 80, le VirtualHost HTTPS écoutera sur le port 443. Il permettra aux utilisateurs d'accéder au site SportLudique à travers une connexion sécurisée.

### 5.1. Création du fichier de configuration

- Créez le fichier de configuration HTTPS :

```
sudo nano /etc/apache2/sites-available/sportludique-ssl.conf
```

> `nano` est un éditeur de texte qui permet de créer et de modifier directement des fichiers de configuration depuis le terminal.

### 5.2. Configuration du VirtualHost

- Insérez la configuration Apache suivante dans le fichier :

```
<VirtualHost *:443>

    ServerName www.tours.sportludique.fr

    DocumentRoot /var/www/sportludique-public

    <Directory /var/www/sportludique-public>
        Options None
        AllowOverride None
        Require all granted
    </Directory>

    SSLEngine on

    SSLCertificateFile /etc/ssl/sportludique/certs/fullchain.pem
    SSLCertificateKeyFile /etc/ssl/sportludique/private/web.key

    ErrorLog ${APACHE_LOG_DIR}/sportludique-ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/sportludique-ssl_access.log combined

</VirtualHost>
```

### Explication de la configuration

- `VirtualHost *:443 :` configure le site pour écouter sur le port HTTPS 443.
- `ServerName :` définit le nom de domaine associé au site Web.
- `DocumentRoot :` indique le répertoire contenant les fichiers du site Web.
- `Directory :` définit les autorisations d'accès au répertoire du site.
- `Options None :` désactive les options supplémentaires d'Apache pour ce répertoire.
- `AllowOverride None :` interdit la modification des règles Apache à l'aide d'un fichier `.htaccess`.
- `Require all granted :` autorise tous les clients à accéder au contenu du répertoire.
- `SSLEngine on :` active le chiffrement SSL/TLS pour ce VirtualHost.
- `SSLCertificateFile :` indique le chemin du fichier contenant le certificat serveur et le certificat intermédiaire.
- `SSLCertificateKeyFile :` indique le chemin vers la clé privée correspondant au certificat serveur.
- `ErrorLog :` définit le fichier dans lequel seront enregistrées les erreurs du site HTTPS.
- `CustomLog :` définit le fichier dans lequel seront enregistrées les requêtes reçues par le site HTTPS.
- `combined :` définit le format des informations enregistrées dans le journal d'accès.

> Important : Apache utilise ici le fichier `fullchain.pem`, qui contient le certificat serveur suivi du certificat intermédiaire. Il n'est donc pas nécessaire de renseigner séparément la directive `SSLCertificateChainFile`.

## 6. Activation du VirtualHost HTTPS

> Maintenant que le fichier de configuration est créé, nous allons activer le site HTTPS dans Apache. Nous vérifierons ensuite que la configuration est correcte avant de recharger le service Web.

### 6.1. Activation du site HTTPS

- Activez le VirtualHost HTTPS :

```
sudo a2ensite sportludique-ssl.conf
```

> Cette commande active le site en créant un lien symbolique dans le répertoire `/etc/apache2/sites-enabled/`.

### 6.2. Vérification de la configuration Apache

- Vérifiez la syntaxe de la configuration globale d'Apache :

```
sudo apache2ctl configtest
```

- Le résultat attendu est :

```
Syntax OK
```

> Cette commande permet de vérifier la syntaxe des fichiers de configuration Apache avant leur application. Si le résultat est `Syntax OK`, la configuration ne présente pas d'erreur de syntaxe.

### 6.3. Rechargement du service Apache

- Rechargez la configuration Apache sans arrêter le service :

```
sudo systemctl reload apache2
```

> Cette commande applique la nouvelle configuration sans arrêter complètement le serveur Web.

### 6.4. Vérification de l'état du service Apache

- Vérifiez que le service Apache fonctionne correctement :

```
sudo systemctl status apache2
```

> Cette commande permet de consulter l'état actuel du service Apache et de vérifier qu'il est bien actif.

### 6.5. Vérification de l'écoute sur le port HTTPS

- Vérifiez que le port HTTPS 443 est bien en écoute :

```
sudo ss -lntp | grep :443
```

> Cette commande permet de vérifier qu'un service écoute bien sur le port TCP 443, utilisé par défaut pour les connexions HTTPS.

## 7. Installation du certificat racine sur le navigateur

> Pour que les navigateurs reconnaissent le certificat présenté par le serveur Web, ils doivent faire confiance à l'autorité de certification racine. Dans notre infrastructure, il faut donc distribuer et installer le certificat racine suivant :

> `sportludique-root.crt.pem`

### 7.1. Procédure d'installation

- Récupérez le fichier `sportludique-root.crt.pem` depuis votre autorité de certification.
- Ouvrez les paramètres de sécurité du navigateur Web.
- Accédez à la gestion des certificats.
- Importez le certificat `sportludique-root.crt.pem` dans les autorités de certification.
- Autorisez la confiance pour l'identification des sites Web.

> Les menus peuvent varier selon le navigateur utilisé. Le certificat racine doit être installé dans le magasin de certificats de confiance du navigateur ou du système utilisé par le client.

> Attention : n'importez jamais la clé privée de l'autorité de certification. Seul le certificat public racine doit être distribué aux clients.

## 8. Tests du site Web en HTTPS

> Nous allons maintenant effectuer plusieurs vérifications afin de confirmer que le site SportLudique est accessible en HTTPS et que la chaîne de certification fonctionne correctement. Ces tests permettront de contrôler la résolution DNS, la connexion HTTPS et la validité du certificat présenté par Apache.

### 8.1. Vérification de la résolution DNS

- Vérifiez que le nom de domaine pointe vers l'adresse IP du serveur Web :

```
nslookup www.tours.sportludique.fr
```

> Cette commande permet de vérifier que le serveur DNS retourne bien l'adresse IP associée au nom de domaine. L'adresse IP attendue dans notre infrastructure est `192.168.233.2`.

### 8.2. Test de la connexion HTTPS

- Testez la réponse du serveur en HTTPS :

```
curl -I https://www.tours.sportludique.fr/
```

### Explication de la commande

- `curl :` permet d'effectuer des requêtes vers un serveur distant.
- `-I :` permet d'envoyer une requête HTTP de type HEAD afin de récupérer uniquement les en-têtes de la réponse.
- `https://www.tours.sportludique.fr/ :` correspond à l'adresse HTTPS du site SportLudique.

> Cette commande permet notamment de vérifier :
>
> - Que la connexion TLS peut être établie.
> - Que le certificat présenté est reconnu par le système.
> - Que le serveur Web répond correctement.
> - Que le site est accessible à travers le protocole HTTPS.

### 8.3. Vérification détaillée du certificat

> Nous allons maintenant utiliser OpenSSL afin d'afficher les informations relatives au certificat présenté par Apache.

- Exécutez la commande suivante :

```
openssl s_client \
    -connect www.tours.sportludique.fr:443 \
    -servername www.tours.sportludique.fr \
    -showcerts
```

### Explication de la commande

- `openssl s_client :` permet d'établir une connexion TLS avec un serveur distant.
- `-connect :` indique le nom de domaine et le port auxquels se connecter.
- `-servername :` transmet le nom de domaine au serveur lors de la négociation TLS (SNI).
- `-showcerts :` affiche les certificats transmis par le serveur.

> Cette commande permet de vérifier :
>
> - Le certificat présenté par le serveur.
> - Le certificat intermédiaire transmis.
> - La négociation TLS.
> - Les informations relatives à la chaîne de certification.

### 8.4. Vérification de la chaîne de certification

> Nous allons maintenant vérifier que le certificat présenté par Apache peut être validé à l'aide du certificat racine de notre infrastructure.

- Exécutez la commande suivante :

```
openssl s_client \
    -connect www.tours.sportludique.fr:443 \
    -servername www.tours.sportludique.fr \
    -verify_return_error \
    -CAfile sportludique-root.crt.pem
```

- Le résultat attendu à la fin de la vérification est :

```
Verify return code: 0 (ok)
```

> Ce résultat indique que la chaîne de certification a été validée à l'aide du certificat racine fourni et que les vérifications effectuées par OpenSSL n'ont pas détecté d'erreur. Le nom de domaine doit également correspondre à celui présent dans le certificat serveur.

### 8.5. Test depuis un navigateur Web

> Nous allons enfin vérifier le fonctionnement du site directement depuis un navigateur Web.

- Ouvrez votre navigateur Web.
- Saisissez l'adresse suivante :

```
https://www.tours.sportludique.fr
```

- Vérifiez que la page SportLudique s'affiche correctement.
- Vérifiez que le navigateur indique une connexion sécurisée.

> Si le certificat est correctement installé et reconnu, le navigateur ne doit afficher aucun avertissement relatif à la confiance du certificat.

## 9. Consultation des journaux HTTPS

> Apache enregistre les connexions et les erreurs liées au VirtualHost HTTPS dans les fichiers de logs définis précédemment. Ces journaux permettent de surveiller l'activité du site Web et de diagnostiquer les éventuels problèmes de configuration ou de connexion au serveur Web.

### 9.1. Consultation des connexions HTTPS

- Consultez en direct les connexions reçues par le site HTTPS :

```
sudo tail -f /var/log/apache2/sportludique-ssl_access.log
```

> Cette commande affiche en temps réel les nouvelles requêtes reçues par le serveur Web.

### 9.2. Consultation des erreurs HTTPS

- Affichez les 30 dernières lignes du journal des erreurs HTTPS :

```
sudo tail -n 30 /var/log/apache2/sportludique-ssl_error.log
```

> Cette commande permet de consulter les dernières erreurs enregistrées par le VirtualHost HTTPS.

### 9.3. Consultation des événements Apache

- Consultez les 50 derniers événements du service Apache :

```
sudo journalctl -u apache2 -n 50 --no-pager
```

### Explication des commandes

- `journalctl :` permet de consulter les journaux du système.
- `-u apache2 :` filtre les événements associés au service Apache.
- `-n 50 :` affiche les 50 dernières lignes.
- `--no-pager :` affiche directement les résultats dans le terminal sans utiliser de visionneuse interactive.

> Ces commandes permettent notamment de diagnostiquer les problèmes de certificat, les erreurs de configuration et les problèmes de connexion au serveur Web.

## 10. Résultat attendu

> À la fin de cette configuration, le site SportLudique doit être accessible à travers une connexion HTTPS sécurisée. Les différents éléments de l'infrastructure doivent être configurés correctement.

### 10.1. Vérification des éléments configurés

À la fin de cette procédure, vous devez obtenir les résultats suivants :

- Le certificat serveur `web.crt` est signé par l'autorité de certification intermédiaire.
- Le fichier `fullchain.pem` contient le certificat serveur et le certificat intermédiaire.
- La clé privée `web.key` est installée et protégée sur le serveur Web.
- Le module SSL d'Apache est activé.
- Le VirtualHost HTTPS est configuré pour écouter sur le port 443.
- Le certificat racine `sportludique-root.crt.pem` est installé dans le magasin de confiance du navigateur.
- Apache fonctionne correctement avec sa configuration HTTPS.
- Le site SportLudique est accessible en HTTPS.
- Le navigateur reconnaît la chaîne de certification et indique une connexion sécurisée.

### 10.2. Adresse du site Web

Le site SportLudique est désormais accessible à l'adresse suivante :

[https://www.tours.sportludique.fr/](<https://www.tours.sportludique.fr/?utm_source=chatgpt.com>)

### 10.3. Points de vigilance

> Attention : pour que votre site HTTPS fonctionne correctement, vous devez également vérifier les éléments suivants :

- Configuration DNS : créer ou vérifier l'enregistrement DNS correspondant afin de faire pointer `www.tours.sportludique.fr` vers l'adresse IP `192.168.233.2`.
- Règles de filtrage réseau : autoriser le trafic HTTPS entrant sur le port TCP 443 au niveau des règles de filtrage du réseau DMZ.
- Extension SAN : vérifier que le certificat serveur contient bien `www.tours.sportludique.fr` dans son extension SAN (Subject Alternative Name).
- Chaîne de certification : vérifier que le certificat intermédiaire est bien signé par l'autorité racine installée sur les navigateurs.
- Correspondance des clés : vérifier que la clé privée installée sur le serveur correspond bien au certificat serveur.

> Une fois tous ces éléments configurés et vérifiés, les utilisateurs pourront accéder au site SportLudique de manière sécurisée grâce au protocole HTTPS.