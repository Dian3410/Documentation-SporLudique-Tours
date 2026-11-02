# Configuration du resolver DNS

- Installer unbound

```sudo apt install unbound```

- Créer un fichier:

```sudo mkdir /etc/unbound/unbound.conf.d/unbound.conf```


# Configuration du fichier :

```
server:
        # Autorise la réponse malgré l'invalidation DNSSEC
        val-permissive-mode: yes

        # Cache la version de Unbound
        hide-version:yes
        hide-identity:yes

        do-ip4: yes # Autorise les connexions DNS en IPv4
        do-ip6: no # Désactive les connexions DNS en IPv6

        verbosity: 1 # Affiche des informations utiles sans trop détaillés

        # Ip autoriser pour communiquer avec Unbound
        interface: 172.28.65.1
        interface: 127.0.0.1

        access-control: 172.28.64.0/18 allow # Réseau autoriser à utilisez le resolveur

        # Zone DNS
        stub-zone:
                name: "tours.sportludique.fr." # Zone autoriser
                stub-addr: 192.168.233.1 # Ip DNS autoritaire

        # Zone de redirection
        forward-zone:
                name: "." # Toutes les requêtes DNS qui pointe vers une autre zone que tours.sportludique.fr sont renvoyer au resolver ci-dessous
                forward-addr: 121.183.90.205 # Resolver des profs
```

## Vérification de la configuration et le bon fonctionnement du service.

- Vérifier le status du service (UP/DOWN): ```sudo systemctl status unbound```

- Vérifier la configuration du fichier unbound: ```sudo unbound-checkconf```

- Démarrer le service unbound: ```sudo systemctl start unbound```

- Redémarrer le service unbound: ```sudo systemctl restart unbound```

- Installer l'outils dig afin d'effectuer une requête vers le resolver: ```sudo apt install dnsutils```


## Vérifier que le resolver répond au requêtes dns

```
dig @127.0.0.1 tours.sportludique.fr

; <<>> DiG 9.20.29-1~deb13u1-Debian <<>> @127.0.0.1 tours.sportludique.fr
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54553
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;tours.sportludique.fr.         IN      A

;; AUTHORITY SECTION:
tours.sportludique.fr.  3508    IN      SOA     ns2.tours.sportludique.fr. admin.tours.sportludique.fr. 2026101401 604800 86400 2419200 86400

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Mon Nov 02 14:27:57 UTC 2026
;; MSG SIZE  rcvd: 96

```
