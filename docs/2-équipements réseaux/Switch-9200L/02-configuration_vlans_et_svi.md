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

- Pour tester l'accessibilité du vlan_interco on affectera l'interface Gi1/0/19 à ce vlan 
```
Switch_tours(config)# interface gigabitEthernet 1/0/19
Switch_tours(config-if)# switchport mode access
Switch_tours(config-if)# switchport access vlan 230
Switch_tours(config-if)# end
```


### VLAN 232 : vlan_serveurs

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
Switch_tours(config-if)# switchport trunk allows vlan 130,230,232
```