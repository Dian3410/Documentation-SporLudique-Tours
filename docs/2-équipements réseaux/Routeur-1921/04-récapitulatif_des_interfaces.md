
### Tableau récapitulatif des interfaces

| Interface | Type | Liaison / Utilisation | VLAN | Adresse IP |
|---|---|---|---:|---|
| `GigabitEthernet 0/0` | Interface physique | Liaison trunk vers le switch | — | — |
| `GigabitEthernet 0/1` | Interface physique | Liaison WAN vers le réseau externe | — | R1: `221.87.137.1` R2: `183.44.37.2`|
| `GigabitEthernet 0/0.130` | Sous-interface | Vlan de management | 130 | `10.0.130.254/24` |
| `GigabitEthernet 0/0.230` | Sous-interface | Vlan interco_WAN | 230 | `192.168.230.254/24` |

### Récapitulatif des liaisons

- `GigabitEthernet 0/0` → liaison **trunk** vers le switch.
- `GigabitEthernet 0/1` → liaison **WAN**
- `GigabitEthernet 0/0.130` → sous-interface dédiée au **VLAN 130 (management)**.
- `GigabitEthernet 0/0.230` → sous-interface dédiée au **VLAN 230 (Vlan interco_WAN)**.