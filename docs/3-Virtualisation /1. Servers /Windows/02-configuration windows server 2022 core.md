# Création de VM windows CORE sur proxmox

**Windows server CORE:**

```Général``` *Noeud pour héberger la VM: (proxmox-01,proxmox-02..)*

- A noter: N'hésitez pas à changer de noeud pour effectuer de la haute disponibilité en cas de défaillance d'un noeud

```VM ID:``` *ID du vlan ou au choix*

```Nom:``` *TRS_NOM_VM*

- Par convention de nommage, nous affectons le nom de notre secteur à l'en tête du nom de la vm.
Dans notre cas *"TRS"*

```Pool de ressources:``` *"groupe-tours-2027"*

```Système d'exploitation```

```Stockage:``` *local-data*

```Image-ISO:``` *fr-fr_windows_server_2022*

```Type:``` *Microsoft Windows*

**Système**

```Stockage EFI:``` *zfs-1*

```Stockage TPM:``` *zfs-1*

**Disques**

```Stockage:``` *zfs-1*

```Taille du disque (Go):``` *20*

**Processeur**

```Socket:``` *1*

```Coeurs :``` *2*

**Mémoire**

```2048 MIB (2Go)```

**Réseau**

```Pont:``` Séléctionner le vlan requis (sptrs230,231..)

```Modèle:``` VirtIO (paravirtualisé)

**Confirmation**

- Vérifié l'ensemble des ressources modifier au besoin sinon créer la vm.

**Configuration de l'installation de la VM** 

- Lors de l'installation la vm, le message suivant s'affichera, **"séléctionner le système d'exploitation à installer".**

Séléctionner ```"Windows Server 2022 Standard Evaluation``` pour posséder une interface core.




