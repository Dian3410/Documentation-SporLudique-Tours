#  vlans configurés sur sur le switch

## Tableau récapitulatif des vlan
| Vlan | ID vlan | Interfaces associée  | Adresse réseau/IP de l'interface du vlan | Mode |
|---|---|---|---|---|
| vlan_management | 130 | Gi1/0/24, Gi1/0/23 | 10.0.130.0/24| access |
| interco_WAN | 230 | Gi1/0/19 | 192.168.230.0/24 | access |
| vlan_clients | 231 | Gi/0/20  | 172.28.64.0/24 | access |
| vlan_servers | 232 | - | 172.28.65.0/24 | - |
| vlan_DMZ | 233 | - | 192.168.233.0/24 | - |
| interco_LAN | 234 | - | 192.168.234.0/24 | - |

## Ports particuliers
 
 Ces ports sont souvent en trunk et transporte plusieurs vlan taggués

| Ports | Précisions |
|---|---|
| **Gi1/0/21, Gi1/0/22** | sont configurés en Trunk et sont reliés physiquement aux routeurs R1 (Gi1/0/21) et R2 (Gi1/0/22). Elles autorisent le passage des VLAN 130 et 230.|
| **Gi1/0/1, Gi1/0/2** | Ces deux ports sont regroupés en une agrégation de liens dynamiques (**LACP** / EtherChannel) et configurés en mode Trunk. Cette configuration assure la tolérance de panne et permet de faire passer l'ensemble des VLANs vers les serveurs virtuels Proxmox (via le switch huawei).|
| **Gi1/0/17** | Ce port est configuré en mode Trunk. Il est physiquement relié à l'interface LAN du pare-feu.Il est configuré pour laisser passer spécifiquement Le **VLAN 130** (Management) pour l'administration<br>Le **VLAN 234** (Interco_LAN) pour le transit des paquets entre le pare-feu et le cœur de réseau |