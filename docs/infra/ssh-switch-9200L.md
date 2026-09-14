#  Configuration de SSH sur switch 9200L
1 **Réinitialisation du switch**
```
Switch> enable
Switch# write erase
Switch# delete flash:vlan.dat
Switch# reload
```

2 **Configuration du VLAN de management**
```
Switch> enable
Switch# conf t
```

- Comme le cahier des charges le demande, on attribue le VLAN 130 comme VLAN de management.
```
Switch(config)# vlan 130
Switch(config-vlan)# name vlan_management
Configuration de SSH sur un switch 9200L
```

- Les ports 23 et 24 sont affectés au VLAN 130 afin d'établir une communication entre les stations d'administration ainsi que les équipements présents dans le VLAN.

```Switch(config)# interface gigabitEthernet 1/0/23```


- On définit que le port 23 appartient à un seul VLAN.

```Switch(config-if)# switchport mode access```

- On définit que seul le VLAN 130 sera actif sur le port 23.

```Switch(config-if)# switchport access vlan 130```

- On établit la même procédure sur le port 24.

```
Switch(config)# interface gigabitEthernet 1/0/24
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 130
```
3 **Adressage de la SVI du switch**

- La SVI est une interface virtuelle associée à un VLAN. Elle permet d'attribuer une adresse IP au switch et donc d'établir une connexion à distance à ce dernier.

```
Switch> enable
Switch# conf t
```

- On procède à l'adressage IP sur l'interface du VLAN concerné.

```
Switch(config)# interface vlan 130
```
- Conformément au réseau du VLAN 130 10.0.130.0/24, l'adresse IP 10.0.130.1 est attribuée à la SVI.

```Switch(config-if)# ip address 10.0.130.1 255.255.255.0```

- On active l'interface virtuelle du VLAN 130.

```Switch(config-if)# no shutdown```

4 **Configuration de SSH sur le switch**
```
Switch> enable
Switch# conf t
```

- On définit le nom du switch et le nom du domaine afin d'établir la génération des clés RSA.

```
Switch(config)# hostname (à compléter)
Switch(config)# ip domain name tours.local
```

- On attribue le niveau de privilège le plus élevé pour les administrateurs.

```
Switch(config)# username dian privilege 15 secret (mots de passe)
Switch(config)# username morgan privilege 15 secret (mots de passe)
Switch(config)# crypto key generate rsa
```

**Une clé RSA de 2048 bits offre un bon niveau de sécurité tout en consommant relativement peu de ressources.**

- On active la version la plus récente de SSH.
```
Switch(config)# ip ssh version 2
```

- On configure les cinq lignes VTY (0 à 4) afin d'autoriser plusieurs connexions d'administration à distance.
```
Switch(config)# line vty 0 4
```

- On autorise uniquement les connexions SSH sur les sessions (lignes VTY) définies.
```
Switch(config)# transport input ssh
```

- On configure l'authentification locale afin que les utilisateurs soient authentifiés à partir des comptes définis sur le switch.
```
Switch(config)# login local
```