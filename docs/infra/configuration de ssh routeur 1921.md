**#  Configuration de SSH sur routeur Cisco 1921**

1 **Réinitialisation du routeur**

- Redémarrer le routeur et pendant le redémarrage appuyer sur **Ctrl + Pause** afin d'accéder au mode ROMMON.

```
rommon 1 > confreg 0x2142
rommon 2 > reset
```

- Après le redémarrage, le routeur demande si l'on souhaite entrer dans la boîte de dialogue de configuration initiale.

```
Would you like to enter the initial configuration dialog? [yes/no]:
```

- Répondre **no** à cette question.

2 **Configuration de SSH sur le routeur**

```
Routeur# conf t
```

- On définit le nom du routeur.

```
Routeur(config)# hostname routeur_tours
```

- On définit le nom de domaine afin d'établir la génération des clés RSA.

```
routeur_tours(config)# ip domain-name tours.local
```

- On attribue le niveau de privilège le plus élevé aux admin

```
routeur_tours(config)# username morgan privilege 15 secret Ad@78Mi
```

- On génère les clés RSA utilisées pour sécuriser les connexions SSH.

```
routeur_tours(config)# crypto key generate rsa
```

- On utilise une clé RSA de 2048 bits.

```
routeur_tours(config)# 2048 bits
```

**ne clé RSA de 2048 bits offre un bon niveau de sécurité tout en consommant relativement peu de ressources.**

- On active la version la plus récente de SSH.

```
routeur_tours(config)# ip ssh version 2
```

- On active le système AAA pour gérer l'authentification et les autorisations des utilisateurs.

```
routeur_tours(config)#aaa new-model
```

- On indique que les utilisateurs doivent être authentifiés avec les comptes locaux du routeur.

```
routeur_tours(config)#aaa authentication login default local
```

- On vérifie les droits de l'utilisateur avant de lui donner accès au mode EXEC.

```
routeur_tours(config)#aaa authorization exec default local
```

- On configure les lignes virtuelles (VTY) pour effectuer des connexions à distance.

```
routeur_tours(config)# line vty 0 4
```

- On applique la méthode d'authentification AAA configurée aux connexions VTY.

```
routeur_tours(config)#login authentication default
```

- On autorise uniquement les connexions SSH.

```
routeur_tours(config)# transport input ssh
```

3 Configuration de l'IP du routeur

- Le routeur utilise une sous-interface afin de communiquer avec le VLAN 130.

```
routeur_tours(config)# interface gigabitEthernet 0/0.130
```

- On associe la sous-interface au VLAN 130 grâce à l'encapsulation 802.1Q.

```
routeur_tours(config-subif)#encapsulation dot1Q 130
```

- Conformément au réseau du VLAN 130 `10.0.130.0/24`, l'adresse IP `10.0.130.254` est attribuée à la sous-interface du routeur.

```
routeur_tours(config-subif)#ip address 10.0.130.254 255.255.255.0
```

4 Configuration de l'interface du routeur

- L'interface `GigabitEthernet 0/0` du routeur est reliée au switch sur un port configuré en mode trunk.

- La sous-interface `GigabitEthernet 0/0.130` permet de transporter le VLAN 130 entre le switch et le routeur.

```
routeur_tours(config)# interface gigabitEthernet 0/0
routeur_tours(config-if)# no shutdown
```

- La liaison entre le routeur et le switch utilise donc le VLAN 130 sur une liaison trunk.

5 Récapitulatif de la configuration

- Interface du routeur :

```
GigabitEthernet 0/0.130
```

- Adresse IP du routeur :

```
10.0.130.254/24
```
- VLAN utilisé :

```
VLAN 130
```
- Port du switch connecté au routeur :

```
Port 22
```
- Mode du port :

```
Trunk
```
- Port utilisé pour SSH :

```
22
```

6 Sauvegarde des conf

- Une fois la configuration terminée, on sauvegarde la configuration afin qu'elle soit conservée après le redémarrage du routeur.

```
routeur_tours# write memory
```

- On peut vérifier la configuration SSH avec :

```
routeur_tours#show ip ssh
```

- n peut vérifier l'état des interfaces avec :

```
routeur_tours#show ip interface brief
```

