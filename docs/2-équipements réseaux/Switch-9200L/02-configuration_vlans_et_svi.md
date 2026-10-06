# Configuration des VLANs et du routage (SVI)

## 1. Activation du routage global
Avant de configurer les interfaces, on active le routage de niveau 3 sur le switch :
```
switch_tours(config)# ip routing
```

## 2. Création des VLANs

### VLAN 230 : vlan d'interconnexion

- Pour assurer la liaison entre le routeur exétérieur (FAI1) et switch L3, le vlan d'interconnexion a été mise en place (192.168.230.0/24)
```
Switch(config)# vlan 230
Switch(config-vlan)# name vlan_interco
```
- Adresse du switch dans le vlan 230
```
Switch_tours(config)# interface vlan 230
Switch_tours(config-if)# ip address 192.168.230.1 255.255.255.0
```

- Rappel: le port Gi1/0/22 est configurer en trunk et assure la liaison physique entre le routeur et le switch L3

- On configure le port Gi1/0/22 en mode trunk et on autorise les VLAN 130 et 230 sur ce trunk 
À chaque ajout d’un nouveau VLAN, celui-ci devra également être ajouté à la liste des VLAN autorisés sur ce port en trunk.
```
Switch_tours(config)# interface gigabitEthernet 1/0/22
Switch_tours(config-if)# switchport mode trunk
Switch_tours(config-if)# switchport trunk allows vlan 130,230
```
⚠️ **Attention :** Sur les switchs Cisco, lorsqu’on modifie la liste des VLAN autorisés sur une interface **trunk**, il faut renseigner **toute la liste des VLAN** à autoriser.  
Sinon, les VLAN précédemment autorisés seront remplacés et seul le VLAN indiqué dans la nouvelle commande sera conservé.
S
- Pour tester l'accessibilité du vlan_interco on affectera l'interface Gi1/0/19 à ce vlan 
```
Switch_tours(config)# interface gigabitEthernet 1/0/19
Switch_tours(config-if)# switchport mode access
Switch_tours(config-if)# switchport access vlan 230
Switch_tours(config-if)# end
switch_tours#write memory
```

### VLAN 231 : vlant_clients

nommage du vlan 231
```
switch_tours(config)#vlan 231
switch_tours(config-vlan)#name vlan_clients
```
configuration du vlan clients
```
switch_tours(config)#interface gigabitEthernet 1/0/20
switch_tours(config-if)#switchport mode access
switch_tours(config-if)#switchport access vlan 231
```


Adressage ip du vlan clients (gateway)
```
switch_tours(config)#interface vlan 231
switch_tours(config-if)#ip address 172.28.64.254 255.255.255.0 (gateway vlan clients)
```

ajout du vlan clients au port trunk
```
switch_tours(config)#interface gigabitEthernet 1/0/22
switch_tours(config-if)#switchport trunk allowed vlan 130,230,231
```
je précise les vlans précédent pour ne pas écraser les autres vlans !

configurer la route du vlan_clients
```
switch_tours(config)#interface vlan 231
switch_tours(config-if)#ip address 172.28.64.254 255.255.255.0
switch_tours#write memory
```
### VLAN 232 : vlan_servers

- Ce vlan contiendra les serveurs internes de SportLudique Tours avec comme adresse réseau `172.28.65.0/24`

```
Switch_tours(config)# vlan 232
Switch_tours(config-vlan)# name vlan_servers
```
- configuration de la svi du vlan 232 (vlan_servers)
```
Switch_tours(config)# interface vlan 232
Switch_tours(config-if)# ip address 172.28.65.254 255.255.255.0
```

- On affecte le port Gi1/0/3 en mode access au vlan 232 
```
Switch_tours(config)# interface gigabitEthernet 1/0/3
Switch_tours(config-if)# switchport mode access
Switch_tours(config-if)# switchport access vlan 232
Switch_tours(config-if)# end
```
!! Atention ne pas oublier d'autoriser ce vlan sur les port en trunk reliés aux routeurs:
```
Switch_tours(config)# interface gigabitEthernet 1/0/22
Switch_tours(config-if)# switchport mode trunk
Switch_tours(config-if)# switchport trunk allows vlan 130,230-232
switch_tours#write memory
```
### VLAN 233 : vlan_dmz

- Ce vlan dmz comme son nom l'indique est la dmz qui contiendra les services exposés depuis internet

```
Switch_tours(config)# vlan 233
Switch_tours(config-vlan)# name vlan_dmz
```
- Ce vlan ne nécéssite pas de configurer la SVI sur le switch de niveau 3 car il sert juste a faire la liaison au niveau de la couche 2

- On affecte le port Gi1/0/18 en mode access au vlan 233
```
Switch_tours(config)# interface gigabitEthernet 1/0/18
Switch_tours(config-if)# switchport mode access
Switch_tours(config-if)# switchport access vlan 233
Switch_tours(config-if)# end
```
!! Atention ne pas oublier d'autoriser ce vlan sur les port en trunk reliés aux routeurs:
```
Switch_tours(config)# interface gigabitEthernet 1/0/22
Switch_tours(config-if)# switchport mode trunk
Switch_tours(config-if)# switchport trunk allows vlan 130,230-233
switch_tours#write memory
```

### VLAN 234 : interco_LAN

nommage du vlan 234
switch_tours(config)#vlan 234
switch_tours(config-vlan)#name interco_LAN

Configuration du vlan interco_LAN

switch_tours(config)#interface gigabitEthernet 1/0/17
switch_tours(config-if)#switchport mode access
switch_tours(config-if)#switchport access vlan 234

Adressage ip du vlan interco_LAN (gateway)
switch_tours(config)#interface vlan 234
switch_tours(config-if)#ip address 192.168.234.1 255.255.255.0

Ajout du vlan au port trunk

switch_tours(config)#interface gigabitEthernet 1/0/22
switch_tours(config-if)#switchport trunk allowed vlan 130,230,231,232,234
je précise les vlans précédent pour ne pas écraser les autres vlans !

switch_tours(config)#interface gigabitEthernet 1/0/21
switch_tours(config-if)#switchport trunk allowed vlan 130,230,231,232,234

Je précise également les vlans précédent pour ne pas écraser les autres vlans !

On sauvegarde
switch_tours#write memory


