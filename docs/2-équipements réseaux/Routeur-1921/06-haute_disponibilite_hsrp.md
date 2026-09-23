# 06 Haute Disponibilité de l'Accès Internet (HSRP)

## 1. Objectif et Concepts

L'objectif de cette mise en place est d'assurer une redondance de la passerelle vers Internet pour l'infrastructure du site. 

Si un seul routeur relie le réseau à Internet, il constitue un **SPOF (Single Point of Failure)** : sa panne ou la coupure de son lien WAN suffit à provoquer l'indisponibilité totale du service. Pour pallier cela, nous utilisons **HSRP (Hot Standby Router Protocol)**.

HSRP permet à deux routeurs (R1 et R2) de partager une adresse IP virtuelle (VIP). Les équipements du réseau local utiliseront cette VIP comme passerelle par défaut. 
* **R1** possède le rôle **Active** : il transmet le trafic en situation normale.
* **R2** possède le rôle **Standby** : il écoute le réseau et prend le relais instantanément en cas de défaillance de R1.

### Tableau d'adressage (VLAN 230 - Interco)

- Pour les besoins de la haute disponibilité de l'accès à Internet ces modifications ont été aporter:
le nom du premier routeur mis en place (routeur_tours) à été modifier en routeur_tours_R1
**L'adresse du routeur dans le vlan d'interconnexion a été modifier de `192.168.230.254` à `192.168.230.251`**
pour plus de précision sur le vlan d'interconnexion voir Infrastructure/contention

| Équipement | Rôle HSRP | IP Réelle | Priorité |
| :--- | :--- | :--- | :--- |
| **Routeur 1 (R1)** | Active | 192.168.230.251/24 | 110 |
| **Routeur 2 (R2)** | Standby | 192.168.230.252/24 | 100 |
| **Passerelle Commune** | **VIP** | **192.168.230.254/24** | - |

---

## 2. Configuration HSRP avec suivi de lien (Tracking)

Il y a un piège classique en redondance : si la liaison WAN (Internet) de R1 est coupée, mais que son interface LAN reste active, R1 conservera son rôle *Active*. Le trafic sera alors envoyé vers un routeur qui ne peut plus joindre Internet.

Pour éviter cela, on configure un **suivi (track)** sur l'interface WAN (ici `GigabitEthernet0/1`). Si le WAN tombe, la priorité HSRP de R1 diminue de 20 (passant de 110 à 90). R2, avec sa priorité de 100, deviendra alors le nouveau routeur *Active* grâce à l'option `preempt`.

### Configuration du Routeur 1 (Active)

- Suivi de la liaison vers internet (liaison WAN) 
```
routeur_tours_R1(config)# track 10 interface GigabitEthernet0/1 20
```
- Configuration HSRP sur la sous-interface LAN (VLAN d'interco)
```
routeur_tours_R1(config)# interface GigabitEthernet0/0.230
routeur_tours_R1(config-subif)# ip address 192.168.230.251 255.255.255.0
routeur_tours_R1(config-subif)# standby 1 ip 192.168.230.254
routeur_tours_R1(config-subif)# standby 1 priority 110
routeur_tours_R1(config-subif)# standby 1 preempt
```