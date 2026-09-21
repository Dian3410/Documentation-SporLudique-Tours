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
```

- Les ports 23 et 24 sont affectés au VLAN 130 afin d'établir une communication entre les stations d'administration ainsi que les équipements présents dans le VLAN.
```
Switch(config)# interface gigabitEthernet 1/0/23
```


- On définit que le port 23 appartient à un seul VLAN.
```
Switch(config-if)# switchport mode access
```

- On définit que seul le VLAN 130 sera actif sur le port 23.
```
Switch(config-if)# switchport access vlan 130
```

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

```
Switch(config-if)# ip address 10.0.130.1 255.255.255.0
```

- On active l'interface virtuelle du VLAN 130.
```
Switch(config-if)# no shutdown
```

4 **Configuration de SSH sur le switch**
```
Switch> enable
Switch# conf t
```

- On définit le nom du switch et le nom du domaine afin d'établir la génération des clés RSA.

```
Switch(config)# hostname switch_tours
switch_tours>(config)# ip domain name tours.local
```

- On attribue le niveau de privilège le plus élevé pour les administrateurs.

```
switch_tours(config)# username dian privilege 15 secret (mots de passe)
switch_tours(config)# username morgan privilege 15 secret (mots de passe)
switch_tours(config)# crypto key generate rsa
```

**Une clé RSA de 2048 bits offre un bon niveau de sécurité tout en consommant relativement peu de ressources.**

- On active la version la plus récente de SSH.
```
switch_tours(config)# ip ssh version 2
```

- On active le système AAA pour gérer l’authentification et les autorisations des utilisateurs.

```
switch_tours(config)#aaa new-model 
```

- On indique que les utilisateurs doivent être authentifiés avec les comptes locaux du switch.

```
switch_tours(config)#aaa authentication login default local
```
- On vérifie les droits de l’utilisateur avant de lui donner accès au mode EXEC.
```
switch_tours(config)#aaa authorization exec default local
```


- On configure les lignes virtuels (VTY) pour effectuer des connexions à distance
```
switch_tours(config)# line vty 0 4
```

- On applique la méthode d’authentification AAA configurée aux connexions VTY.

```
switch_tours(config)#login authentication default
```

- On autorise uniquement les connexions SSH 
```
switch_tours(config)# transport input ssh
```

5 **Sauvegarde des conf**
```
switch_tours# write memory
```
- Si **system ignore startup-config est activé (c'est à dire =1)**, les configurations sauvegardées peuvent être ignorées au redémarrage du switch. Il est donc recommandé de vérifier cette variable lors de la préparation d'un switch neuf ou après une réinitialisation.

vérifier que le system_ignore_startup-config=0
```
switch_tours#show romvar
```
si il est à 1: Passer en Configure Terminal
```
switch_tours(config)#no system ignore startup-config switch all
```
Après on peut sauvegarder de nouveau avec write memory.

6 **Mise en place du vlan 230 (vlan d'interco)**

- Pour assurer la liaison entre le routeur exétérieur (FAI1) et switch L3, le vlan d'interconnexion a été mise en place (192.168.230.0/24)
```
Switch(config)# vlan 230
Switch(config-vlan)# name vlan_interco
```

- Rappel: le port 22 est configurer en trunk et assure la liaison physique entre le routeur et le switch L3

```
Switch(config)# interface gigabitEthernet 1/0/23
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allows vlan 230
Switch(config-if)# ip address 10.0.230.1 255.255.255.0
```
