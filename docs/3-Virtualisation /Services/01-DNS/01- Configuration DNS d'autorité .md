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
 
        //Ecoute sur les interfaces IPv4
        listen-on { any; };
        listen-on-v6 { none; };
        dnssec-validation no;
 
        //pour desactiver la resolution
        recursion no;
        allow-query { any; }; //on accepte les requete de tout le monde
};
```

## 4. Configuration des "Views" (Split DNS)

Les "views" permettent au DNS de renvoyer des informations différentes en fonction de l'adresse IP source.

Éditez le fichier de configuration locale pour déclarer les `Views`:
```
sudo nano /etc/bind/named.conf.local
```


```
#  Définition des adresses IP considérées comme "Internes"
acl "reseaux-internes" {
    127.0.0.1;
    172.28.65.1;
    192.168.233.0/24;
};
 
# La vue INTERNE
view "interne" {
    match-clients { reseaux-internes; };
 
    zone "tours.sportludique.fr" {
        type master;
        file "/etc/bind/db.tours.sportludique.fr.interne";
    };
};
 
# La vue EXTERNE
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
@       IN      SOA     ns2.tours.sportludique.fr. admin.tours.sportludique.fr. (
                          2026101401 ; Serial (Date du jour + version 01)
                          604800     ; Refresh
                          86400      ; Retry
                          2419200    ; Expire
                          86400 )    ; Negative Cache TTL
;
; Serveur de noms principal
@       IN      NS      ns2.tours.sportludique.fr.
 
; Enregistrements de la DMZ
ns2     IN      A       192.168.233.1
mail    IN      A       192.168.233.2
www     IN      A       192.168.233.3
 
; On confie le sous-domaine "ad" au serveur Windows DC1
ad      IN      NS     trs-dc1.trs.tours.sportludique.fr.
 
; Glue record : L'adresse IP du DC1 dans le Vlan 232
trs-dc1.trs  IN      A       172.28.65.120
```

5.2 Fichier "Externe" (Filtré)
```
sudo nano /etc/bind/db.tours.sportludique.fr.externe
```
```
 
$TTL    86400
@       IN      SOA     ns2.tours.sportludique.fr. admin.tours.sportludique.fr. (
                          2026101402 ; Serial
                          604800
                          86400
                          2419200
                          86400 )
;
@       IN      NS      ns2.tours.sportludique.fr.
 
; Seuls les services publics de la DMZ sont publiés ici
ns2     IN      A       221.87.137.1
mail    IN      A       221.87.137.1
www     IN      A       221.87.137.1
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

## Vérifier que le serveur autoritaire répond au requêtes dns

```
dig  www.tours.sportludique.fr
 
; <<>> DiG 9.20.30-1~deb13u1-Debian <<>> www.tours.sportludique.fr
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52457
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
 
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.tours.sportludique.fr.     IN      A
 
;; ANSWER SECTION:
www.tours.sportludique.fr. 85062 IN     A       221.87.137.1
 
;; Query time: 4 msec
;; SERVER: 121.183.90.205#53(121.183.90.205) (UDP)
;; WHEN: Mon Nov 02 16:36:44 CET 2026
;; MSG SIZE  rcvd: 70
```