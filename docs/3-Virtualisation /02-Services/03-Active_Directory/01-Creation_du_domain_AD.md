# Configuration Initiale et Création du Domaine (DC1)

Ce document détaille les étapes de préparation et de promotion de notre premier contrôleur de domaine (DC1) sur Windows Server 2022 Core. La procédure se divise en deux phases : la configuration de base via l'utilitaire natif `SConfig`, puis la promotion du domaine via PowerShell.

## 1. Configuration initiale via SConfig

Dès le premier démarrage de Windows Server Core, l'utilitaire de configuration du serveur s'affiche (ou peut être lancé en tapant `sconfig` dans l'invite de commande). C'est ici que nous effectuons la préparation du système.

###  Configuration de l'IP Statique
Le contrôleur de domaine nécessite une adresse IP fixe pour garantir la stabilité des services AD et DNS.
1. Dans le menu SConfig, tapez **`8`** pour accéder aux **Paramètres réseau**
2. Sélectionnez l'index de votre carte réseau (généralement `1`).
3. Choisissez l'option pour définir l'adresse IP de l'adaptateur, sélectionnez le mode Statique (`S`), puis entrez les valeurs :
   - **Adresse IP :** `172.28.65.120`
   - **Masque de sous-réseau :** `255.255.255.0`
   - **Passerelle par défaut :** `172.28.65.254`
4. Choisissez ensuite l'option pour définir les serveurs DNS. Saisissez `127.0.0.1` (localhost) puisque ce serveur hébergera son propre service DNS.

###  Activation du Bureau à distance (RDP)
Afin de faciliter l'administration future du serveur sans passer par la console Proxmox, le RDP a été activé.
1. De retour au menu principal, tapez **`7`** pour configurer le **Bureau à distance**
2. Tapez `E` (Enable) pour l'activer.
3. Choisissez l'option `1` pour n'autoriser que les clients exécutant le Bureau à distance avec authentification au niveau du réseau (plus sécurisé).

###  Changement du nom de l'ordinateur
1. Dans le menu principal, tapez **`2`** pour modifier le **Nom de l'ordinateur**
2. Saisissez le nouveau nom : `TRS-DC1`.
3. Le système vous demandera de redémarrer le serveur pour appliquer ce changement. Acceptez le redémarrage.

---

## 2. Création du domaine Active Directory via PowerShell

- Une fois le serveur redémarré (avec sa nouvelle IP et son nom `TRS-DC1`), reconnectez-vous. 

SConfig se lance automatiquement, tapez **`15`** pour **Quitter vers la ligne de commande (PowerShell)**

Dans l'invite PowerShell, exécutez les commandes suivantes pour installer les rôles requis et promouvoir le serveur en tant que premier contrôleur de domaine de la forêt `trs.tours.sportludique.fr`.

###  Installation du rôle AD DS
Cette commande installe les binaires d'Active Directory Domain Services ainsi que les outils de gestion associés.

```
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

###  Promotion du contrôleur de domaine

Cette commande configure la nouvelle forêt Active Directory et installe automatiquement le rôle DNS (indispensable au bon fonctionnement de l'AD).
PowerShell
```
Install-ADDSForest -DomainName "trs.tours.sportludique.fr" -InstallDns -Force
```

- Une fois le processus terminé, le serveur redémarrera automatiquement. Au redémarrage, vous pourrez vous connecter avec le compte TRS\Administrateur, confirmant ainsi que le domaine est opérationnel.
