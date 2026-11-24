# Tableau récapitulatif des règles

---

### 1. Autorise l'administration du firewall via le vlan management uniquement

| État | Action | Source                    | Destination                       | Port dest             | Protocole | Inspection de sécurité |
|:----:|:------:|:--------------------------|:----------------------------------|:----------------------|:---------:|:----------------------:|
| on   | passer | vlan_mana (10.0.130.0/24) | Firewall_vlan_mana (10.0.130.250) | port 1300 TCP, https  | /         | IPS                    |

---

### 2. Autorise tout les réseaux à accéder aux différents services de la DMZ PUBLC

| État | Action | Source | Destination  | Port dest                 | Protocole | Inspection de sécurité |
|:----:|:------:|:------:|:-------------|:--------------------------|:---------:|:----------------------:|
| on   | passer | any    | Network_dmz1 | http,https,dns,smtps,imaps | /         | IPS                    |

---

### 3. Autorise tous les réseaux à communiquer avec la DMZ PUBLIC

| État | Action | Source | Destination  | Port dest | Protocole | Inspection de sécurité |
|:----:|:------:|:------:|:-------------|:---------:|:---------:|:----------------------:|
| on   | passer | any    | Network_dmz1 | any       | icmp      | IPS                    |

---

### 4. Autorise tous les réseaux à communiquer avec internet (via un ping)

| État | Action | Source      | Destination | Port dest | Protocole | Inspection de sécurité |
|:----:|:------:|:------------|:------------|:---------:|:---------:|:----------------------:|
| on   | passer | All_network | internet    | any       | icmp      | IPS                    |

---

### 5. Autorise tous les réseaux à communiquer avec internet (via un ping)

| État | Action | Source      | Destination | Port dest | Protocole | Inspection de sécurité |
|:----:|:------:|:------------|:------------|:---------:|:---------:|:----------------------:|
| on   | passer | All_network | internet    | any       | http,https | IPS                   |

---

### 6. Autorise le resolver interne (vlan server) à communiquer avec le resolver externe (resolver prof)

| État | Action | Source                                  | Destination  | Port dest | Protocole | Inspection de sécurité |
|:----:|:------:|:----------------------------------------|:-------------|:---------:|:---------:|:----------------------:|
| on   | passer | dns resolver/interface: vlan_intercoLAN | resolver_prof | dns       | /         | IPS                    |

---

### 7. Autorise la DMZ PUBLIC à accéder à internet

| État | Action | Source  | Destination | Port dest | Protocole    | Inspection de sécurité |
|:----:|:------:|:--------|:------------|:---------:|:-------------|:----------------------:|
| on   | passer | Network | internet    | any       | dns,http,https | IPS                  |

---