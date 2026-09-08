# Documentation de l'infrastructure — Site de Tours (TRS)

Bienvenue sur la documentation technique officielle du site de **Tours**, développée dans le cadre du projet d'infrastructure **SportLudique**.

Ce portail centralise l'ensemble des procédures, configurations réseau, déploiements de services et règles d'administration propres à l'îlot de Tours.

!!! info "Repères rapides — Site TRS"
    * **Domaine Active Directory :** `trs.tours.sportludique.fr` (NetBIOS : `TRS`)
    * **Préfixe obligatoire des VMs :** `TRS-` (ex. : `TRS-DC01`, `TRS-FW01`)
    * **VLAN Management (imposé) :** `130` (`172.28.130.0/24`)
    * **Plage VLANs attribuée :** `230` à `239`

## Navigation rapide

* **[Infrastructure](infra/topologie.md)** — Topologie globale, plan d'adressage IP et découpage des VLANs.
* **[Conventions TRS](infra/conventions.md)** — Règles de nommage, brassage physique et consignes du laboratoire.
* **[Annexes](annexes/memo.md)** — Aide-mémoire des commandes système et identifiants techniques.
