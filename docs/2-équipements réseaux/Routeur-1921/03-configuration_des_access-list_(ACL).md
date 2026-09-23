
### Mise en place de l'ACL pour le NAT

- Une ACL standard est créée afin d'identifier le réseau d'interco `192.168.230.0/24` dont les adresses doivent être traduites par le NAT.

```
routeur_tours(config)# ip access-list standard ACL_NAT
routeur_tours(config-std-nacl)# permit 192.168.230.0 0.0.0.255
```

- On applique ensuite la règle NAT sur l'interface WAN `GigabitEthernet0/1`.
- L'option `overload` permet à plusieurs machines du réseau LAN d'utiliser l'adresse IP de l'interface WAN pour accéder au réseau externe.

```
routeur_tours(config)# ip nat inside source list ACL_NAT interface GigabitEthernet0/1 overload
routeur_tours(config)# end
```

- Le réseau principal de SportLudique Tours (172.28.64.0/18) est intégré à l'ACL standard ACL_NAT car ses adresses privées doivent être traduites (nattées) pour accéder à Internet.

```
routeur_tours(config)# ip access-list standard ACL_NAT
routeur_tours(config-std-nacl)# permit 172.28.64.0 0.0.63.255
```
