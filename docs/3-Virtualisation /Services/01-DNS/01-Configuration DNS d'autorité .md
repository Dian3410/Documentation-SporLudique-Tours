# Configuration DNS d'autorité BIND9 — Site de Tours

Cette documentation détaille l'installation et la configuration de notre serveur DNS d'autorité avec le service BIND9, basé sur Debian 13.

## 1. Préparation et rôles

Serveur d'autorité UNIQUEMENT : Aucune récursivité n'est autorisée.

Split DNS (Vues) : Deux zones de recherche DNS, une interne (qui délègue vers l'Active Directory) et une externe (filtrée pour Internet).

NAT/PAT : Redirection de port sur le routeur Cisco et le Stormshield pour rendre le port 53 accessible de l'extérieur.

## 2. Installation de BIND9

Mise à jour du système et installation des paquets nécessaires :
```
sudo apt update 
sudo apt upgrade 
sudo apt install bind9 bind9utils bind9-doc dnsutils 
```

## 3. Configuration de la sécurité (Options globales)

Pour des raisons de sécurité, nous désactivons la résolution (récursivité). Éditez le fichier des options globales :
```
sudo nano /etc/bind/named.conf.options
```
```
options {
    directory "/var/cache/bind";

    // Écoute sur les interfaces IPv4, désactivation IPv6
    listen-on { any; };
    listen-on-v6 { none; };
    dnssec-validation no;

    // SÉCURITÉ : Désactivation de la résolution
    recursion no;
    allow-query { any; }; 
};
```

## 4. Configuration des "Views" (Split DNS)

Les "views" permettent au DNS de renvoyer des informations différentes en fonction de l'adresse IP source.

Éditez le fichier de configuration locale pour déclarer les `Views`:
```
sudo nano /etc/bind/named.conf.local
```


```
acl "reseaux-internes" {
    127.0.0.1;
    172.28.65.1;       # Le résolveur Unbound 
    192.168.233.0/24;  # Le réseau de la DMZ (pour que tout les servers de la DMZ voient l'AD)
};

view "interne" {
    match-clients { reseaux-internes; };
    
    zone "tours.sportludique.fr" {
        type master;
        file "/etc/bind/db.tours.sportludique.fr.interne";
    };
};

view "externe" {
    match-clients { any; };
    
    zone "tours.sportludique.fr" {
        type master;
        file "/etc/bind/db.tours.sportludique.fr.externe";
    };
};
```

## 5. Création des fichiers de zones

5.1 Fichier "Interne" (Avec délégation AD)
```
sudo nano /etc/bind/db.tours.sportludique.fr.interne
```
```
$TTL    86400
@       IN      SOA     ns1.tours.sportludique.fr. admin.tours.sportludique.fr. (
                          2026101401 ; Numéro de série (YYYYMMDDNN)
                          604800     ; Refresh
                          86400      ; Retry
                          2419200    ; Expire
                          86400 )    ; Durée minimale de cache
;
@       IN      NS      ns1.tours.sportludique.fr.

ns1     IN      A       192.168.233.1
mail    IN      A       192.168.233.2
www     IN      A       192.168.233.3

; === DÉLÉGATION ACTIVE DIRECTORY ===
trs     IN      NS      trs-dc1.trs.tours.sportludique.fr.
trs-dc1.trs  IN      A       172.28.65.120
```

5.2 Fichier "Externe" (Filtré)
```
sudo nano /etc/bind/db.tours.sportludique.fr.externe
```
```
$TTL    86400
@       IN      SOA     ns1.tours.sportludique.fr. admin.tours.sportludique.fr. (
                          2026101401 ; Numéro de série
                          604800
                          86400
                          2419200
                          86400 )
;
@       IN      NS      ns1.tours.sportludique.fr.

ns1     IN      A       192.168.233.1
mail    IN      A       192.168.233.2
www     IN      A       192.168.233.3
```

!! Règle d'or : Tout changement dans la configuration des zones requiert l'incrémentation du "Numéro de série" (Serial) pour prendre effet !

## 6. Désactivation des zones par défaut (Correction du crash des Vues)

BIND9 exige que toutes les zones soient dans une vue. Nous devons désactiver les zones par défaut de Debian :
```
sudo nano /etc/bind/named.conf
```

Mettez en commentaire la dernière ligne avec // ou # :
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
// include "/etc/bind/named.conf.default-zones";
```

## 7. Tests et vérification (tester et rajoueter des capture d'écrans ou autre chose à voir avec mon collégue)

1. Vérification de la syntaxe
 
 

```
sudo named-checkconf
sudo named-checkzone tours.sportludique.fr /etc/bind/db.tours.sportludique.fr.interne
sudo named-checkzone tours.sportludique.fr /etc/bind/db.tours.sportludique.fr.externe
```
 2. Redémarrage
```
sudo systemctl restart bind9
sudo systemctl status bind9
```

## Ne pas oublier de faire le PAT sur les routeurs ( la redirection de ports )