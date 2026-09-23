
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

- On configure ensuite une route par défaut afin que le routeur puisse envoyer les paquets vers le réseau externe via la passerelle `221.87.137.2`.

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
