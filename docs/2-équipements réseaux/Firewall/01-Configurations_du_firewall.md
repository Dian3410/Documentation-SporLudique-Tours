# Configuration des interfaces et du routage

Le pare-feu Stormshield SN210 assure le filtrage et le routage entre les différentes zones de l'infrastructure. Sa configuration a été réalisée intégralement via l'interface WEB d'administration.

## 1. Configuration des interfaces réseau

Les interfaces du pare-feu ont été affectées aux différents réseaux de la topologie :

- **L'interface WAN est physiquement relié au port 19 du switch:** Interface connectée vers les routeurs FAI (Réseau `192.168.230.0/24`)

- **L'interface LAN est physiquement relié au port 17 du switch:** Interface connectée au cœur de réseau (Switch L3) (Réseau `192.168.234.0/24`)

- **L'interface DMZ est physiquement relié au port du switch HUAWEI dédier au vlan DMZ:** Interface dédiée aux serveurs exposés (Serveur Web, Serveur Mail, Serveur DNS externe) (Réseau `192.168.233.0/24`)

## 2. Routage statique (Retour vers le LAN)

Afin que le pare-feu puisse joindre les réseaux privés situés derrière le cœur de réseau (Switch 9200L), une route statique a été ajoutée dans l'interface WEB :


| Réseau de Destination/mask | Interface (IP)  | Passerelle  |
|---|---|---|
|172.28.64.0/18|In(192.168.234.254)|192.168.234.1|