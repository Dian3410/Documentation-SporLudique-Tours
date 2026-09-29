#  vlans configurés sur sur le switch

| Vlan | ID vlan | Interfaces associée  | Adresse réseau/IP de l'interface du Vlan | Mode |
|---|---|---|---|---|
| vlan management | 130 | Gi1/0/24, Gi1/0/23 | 10.0.130.0/24| access |
| vlan interco WAN | 230 | Gi1/0/19 | 192.168.230.0/24 | access |
| Vlan clients | 231 | Gi/0/20  | 172.28.64.0/24 | access |
| vlan servers | 232 | Gi1/0/3 | 172.28.65.0/24 | access |
| Vlan DMZ | 233 | Gi/0/18  | 192.168.233.0/24 | access |
| Vlan interco_LAN | 234 | Gi/0/17 | 192.168.234.0/24 | access |