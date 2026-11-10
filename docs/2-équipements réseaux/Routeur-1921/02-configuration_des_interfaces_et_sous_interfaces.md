
# 02 **Configuration des Interfaces et sous interfaces**

## Configuration de l'interface WAN

- L'interface `GigabitEthernet 0/1` est utilisée pour la connexion WAN vers le réseau externe.
- L'adresse IP `221.87.137.1/30` est attribuée à cette interface.
- L'interface est définie comme l'interface extérieure du NAT avec `ip nat outside`.

```
routeur_tours(config)# interface gigabitEthernet 0/1
routeur_tours(config-if)# ip address 221.87.137.1 255.255.255.252
routeur_tours(config-if)# ip nat outside
routeur_tours(config-if)# no shutdown
routeur_tours(config-if)# exit
routeur_tours(config)# exit
```

- On configure ensuite une route par défaut afin que le routeur puisse envoyer les paquets vers le réseau externe via la passerelle `221.87.137 2`.
- On configure également une route vers le réseau `192.168.230.0/24`

```
routeur_tours(config)# ip route 0.0.0.0 0.0.0.0 221.87.137.2   
```

## Configuration de la sous interface Gi0/0.230

- La sous-interface `GigabitEthernet 0/0.230` est utilisée pour le VLAN 230.
- Elle est associée au VLAN 230 grâce à l'encapsulation 802.1Q.
- L'adresse IP `192.168.230.254/24` est utilisée comme passerelle du Vlan 230.
- La sous-interface est définie comme interface intérieure du NAT avec `ip nat inside`.

```
routeur_tours(config)# interface gigabitEthernet 0/0.230
routeur_tours(config-subif)# encapsulation dot1Q 230
routeur_tours(config-subif)# ip address 192.168.230.254 255.255.255.0
routeur_tours(config-subif)# ip nat inside
routeur_tours(config-subif)# no shutdown
routeur_tours(config-subif)# exit
```

## Redirection de port 

### port 53

- Lorsque **des requêtes DNS externes** sont effectuées, on doit rediriger ces requêtes vers le **serveur DNS autoritaire**.

----------------

- On effectue donc une redirection de port vers le port **53**, à l'adresse **du DNS autoritaire 192.168.233.1**


```
routeur_tours_R1(config)#ip nat inside source static udp 192.168.233.1 53 interface GigabitEthernet0/1 53
routeur_tours_R1(config)#ip nat inside source static tcp 192.168.233.1 53 interface GigabitEthernet0/1 53
```

### port 80 et 443
- Lorsque **des requêtes ** sont effectuées, on doit rediriger ces requêtes vers le **serveur web**.
- On effectue donc une redirection de port vers le port **80** et **443**, à l'adresse **du serveur web 192.168.233.2**

```
routeur_tours_R1(config)#ip nat inside source static tcp 192.168.233.2 80  interface GigabitEthernet0/1 80
routeur_tours_R1(config)#ip nat inside source static tcp 192.168.233.2 443 interface GigabitEthernet0/1 443
```