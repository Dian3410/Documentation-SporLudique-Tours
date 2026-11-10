# Création d'un site Web HTTP – SportLudique

## 1. Création et configuration de la VM Debian

- Créez ou clonez une machine virtuelle Debian.
- Ajoutez les cartes réseau correspondant aux VLAN 233 (DMZ) et 130 (Management).
- Configurez les adresses IP statiques sur les deux cartes réseau :

| Carte réseau | VLAN | Adresse IP | Rôle |
| --- | --- | --- | --- |
| Carte 1 | DMZ (233) | `192.168.233.2` | Adresse du serveur Web |
| Carte 2 | Management (130) | `10.0.130.112` | Adresse du serveur web |

## 2. Installation d'Apache

### Exécutez les commandes suivantes une fois que votre VM est connectée au réseau.


- Mettez à jour la liste des paquets disponibles.

```
sudo apt update
```

- Installez Apache (serveur Web) et Curl (outil de test HTTP).

```
sudo apt install apache2 curl
```

- Activez Apache au démarrage du système et lancez le service.

```
sudo systemctl enable --now apache2
```

- Vérifiez l'état du service Apache.

```
sudo systemctl status apache2
```

- Affichez les ports TCP en écoute ainsi que les processus associés.

```
sudo ss -lntp
```

- Testez l'accès au serveur Web en local à l'aide du protocole HTTP.

```
curl -I http://127.0.0.1/
```

## 3. Création du site Web

- Créez le dossier qui contiendra les fichiers de votre site Web en lui attribuant les permissions `755`.

```
sudo install -d -m 755 /var/www/sportludique-public
```

> **Permissions `755` :** le propriétaire dispose de tous les droits (lecture, écriture et exécution), tandis que les autres utilisateurs peuvent uniquement lire le contenu et accéder au dossier, sans pouvoir le modifier.

- Créez la page d'accueil du site en éditant le fichier ci-dessous

```
sudo nano /var/www/sportludique-public/index.html
```

- Insérez le code HTML suivant dans le fichier :

```
<!doctype html>
<html lang="fr">
<head>
    <meta charset="utf-8">
    <title>SportLudique — Site public</title>
</head>
<body>
    <h1>SportLudique — Site public</h1>
    <p>Ville : Tours — Serveur : WEB1</p>
</body>
</html>
```

> Enregistrez le fichier avec `Ctrl + O`, puis appuyez sur `Entrée`. Quittez ensuite l'éditeur avec `Ctrl + X`.

- Définissez les permissions du fichier pour autoriser sa lecture par tous les utilisateurs et son écriture uniquement par le propriétaire.

```
sudo chmod 644 /var/www/sportludique-public/index.html
```

## 4. Configuration du VirtualHost Apache

- Créez le fichier de configuration du site.

```
sudo touch /etc/apache2/sites-available/sportludique-public.conf
```

- Ouvrez le fichier afin de pouvoir le modifier.

```
sudo nano /etc/apache2/sites-available/sportludique-public.conf
```

- Insérez la configuration Apache suivante :

```
<VirtualHost *:80>
    ServerName www.ville.sportludique.fr
    DocumentRoot /var/www/sportludique-public

    <Directory /var/www/sportludique-public>
        Options None
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/sportludique-public_error.log
    CustomLog ${APACHE_LOG_DIR}/sportludique-public_access.log combined
</VirtualHost>
```

### Explication de la configuration

- `VirtualHost *:80` : configure le site pour écouter sur le port HTTP 80.
- `ServerName` : définit le nom de domaine associé au site.
- `DocumentRoot` : indique le dossier contenant les fichiers du site Web.
- `Directory` : définit les autorisations d'accès au dossier du site.
- `Options None` : désactive les options supplémentaires d'Apache pour ce dossier.
- `AllowOverride None` : interdit la modification des règles Apache à l'aide d'un fichier `.htaccess`.
- `Require all granted` : autorise tous les clients à accéder au contenu du dossier.
- `ErrorLog` : définit le fichier dans lequel seront enregistrées les erreurs.
- `CustomLog` : définit le fichier dans lequel seront enregistrées les requêtes reçues par le site.

## 5. Activation du site

- Activez le VirtualHost dans Apache.

```
sudo a2ensite sportludique-public.conf
```

- Vérifiez que la configuration Apache ne contient aucune erreur.

```
sudo apache2ctl configtest
```

- Rechargez la configuration Apache sans arrêter le service.

```
sudo systemctl reload apache2
```

- Affichez les VirtualHosts configurés ainsi que leur association aux ports et aux noms de domaine.

```
sudo apache2ctl -S
```

## 6. Tests du site Web

- Vérifiez que le nom de domaine est correctement résolu par le serveur DNS.

```
nslookup www.ville.sportludique.fr
```

### Cette commande permet de vérifier que le nom de domaine correspond bien à l'adresse IP du serveur Web.

- Testez l'accès au site à l'aide de son nom de domaine et affichez les en-têtes de la réponse HTTP.

```
curl -I http://www.ville.sportludique.fr/
```

## 7. Consultation des journaux (logs)

- Consultez en direct les connexions et les requêtes reçues par votre site Web.

```
sudo tail -f /var/log/apache2/sportludique-public_access.log
```

- Affichez les 30 dernières lignes du journal des erreurs Apache.

```
sudo tail -n 30 /var/log/apache2/sportludique-public_error.log
```

- Affichez les 50 derniers événements enregistrés pour le service Apache.

```
sudo journalctl -u apache2 -n 50 --no-pager
```

## 8. Résultat attendu

> À la fin de cette configuration, vous devez obtenir les résultats suivants : 

- Votre VM Debian possède deux cartes réseau configurées avec des adresses IP statiques.
- Apache est installé, activé et fonctionnel.
- Votre site SportLudique est hébergé dans le dossier `/var/www/sportludique-public`.
- Le VirtualHost est configuré pour écouter sur le port 80.
- Le nom de domaine `www.ville.sportludique.fr` pointe vers le serveur Web.
- Votre site est accessible en HTTP depuis le réseau autorisé.

### Adresse du site Web

[http://www.ville.sportludique.fr/](<http://www.ville.sportludique.fr/?utm_source=chatgpt.com>)

### Attention

> Pour que votre nom de domaine fonctionne correctement, vous devez également :

- Créer l'enregistrement DNS correspondant afin de faire pointer `www.ville.sportludique.fr` vers l'adresse IP `192.168.233.2`.
- Autoriser le trafic HTTP entrant sur le port 80 au niveau des règles de filtrage du réseau DMZ.

> Une fois ces éléments configurés, vous pourrez accéder à votre site SportLudique depuis le réseau autorisé.