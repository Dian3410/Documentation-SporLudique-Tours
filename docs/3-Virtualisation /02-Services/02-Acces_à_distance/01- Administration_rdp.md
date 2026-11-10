# Administration à distance des serveurs Windows (RDP)

## 1. Contexte et architecture d'administration
Afin de respecter les principes d'administration du système d'information de SportLudique, l'administration des serveurs s'effectue à distance depuis le poste client d'administration Linux.

* **Poste d'administration (Linux) :** Utilisation du client graphique **Remmina** (ou `xfreerdp`).
* **Serveur rebond / GUI (`TRS-DC1` / `172.28.65.120`) :** Accès initial en RDP via Remmina.
* **Administration secondaire :** Prise de main à distance en RDP depuis l'interface du serveur windows GUI vers les autres serveurs windows du domaine (notamment les serveurs Windows Server Core).

---

## 2. Activation du Bureau à distance sur Windows Server

Par défaut, le service de Bureau à distance (RDP sur le port `3389`) est désactivé sur Windows Server.

### Procédure d'activation (Interface Graphique - Server Manager)
1. Ouvrez le **Gestionnaire de serveur** (*Server Manager*).
2. Cliquez sur la section **Serveur local** (*Local Server*).
3. Repérez la ligne **Bureau à distance** (*Remote Desktop*) qui affiche l'état `Désactivé`.
4. Cliquez sur **Désactivé** pour ouvrir la fenêtre de configuration des propriétés système.
5. Cochez l'option **Autoriser les connexions à distance à cet ordinateur** (*Allow remote connections to this computer*).
6. **Sécurité (NLA) :** Veillez à laisser cochée l'option *"N'autoriser que la connexion des ordinateurs exécutant le Bureau à distance avec authentification NLA"* (Network Level Authentication).
   > **Note de sécurité :** L'authentification NLA force l'utilisateur à s'authentifier au niveau du réseau avant d'établir la session graphique, protégeant ainsi le serveur contre les attaques par force brute et le déni de service.
7. Validez en cliquant sur **OK**.

---

## 3. Configuration du Pare-feu Windows
Pour que les flux RDP puissent atteindre le serveur :

- S'assurez que la règle de pare-feu **Bureau à distance (Trafic entrant - TCP-In)** sur le port `3389` est bien activée.


---

## 4. Connexion depuis le poste d'administration Linux (Remmina)

Depuis le poste client d'administration Linux :

1. Lancez l'application **Remmina**.
2. Créez un nouveau profil de connexion :
   * **Protocole :** `RDP - Remote Desktop Protocol`
   * **Serveur :** `172.28.65.120` (Adresse IP de `WIN-GUI`)
   * **Nom d'utilisateur :** `nom_de_l'admin`
   * **Mot de passe :** `********`
   * **Domaine :** `trs.tours.sportludique.fr`
3. Dans l'onglet **Afficheur**, sélectionnez la résolution adaptée à votre écran.
4. Enregistrez et lancez la connexion.

Une fois connecté sur `TRS-GUI`, vous pouvez ouvrir une session RDP vers d'autres serveurs du domaine (notamment les contrôleurs de domaine secondaires ou les serveurs en mode Core) en utilisant l'utilitaire le bureau à distance de Windows.