# Création de VM windows client sur proxmox

**Général**

```noeud``` *proxmox-02*

- A noter: N'hésitez pas à changer de noeud pour effectuer de la haute disponibilité en cas de défaillance d'un noeud

```VM ID:``` *202*

```Nom:``` *TRS-WIN11-CLIENT*

- Par convention de nommage, nous affectons le nom de notre secteur à l'en tête du nom de la vm.
Dans notre cas *"TRS"*

```Pool de ressources:``` *"groupe-tours-2027"*

**Système d'exploitation**

```Stockage:``` *local-data*

```Image-ISO:``` *Win11_25H2_French_x64.iso*

```Type:``` *Microsoft Windows*

**Système**

```Stockage EFI:``` *zfs-1*

```Stockage TPM:``` *zfs-1*

**Disques**

```Stockage:``` *zfs-1*

```Taille du disque (Go):``` *30*

**Processeur**

```Socket:``` *1*

```Coeurs :``` *2*

**Mémoire**

```4096 MIB (4Go)```

**Réseau**

```Pont:``` *sptrs231 ( vlan client)*

```Modèle:``` *VirtIO (paravirtualisé)*

**Confirmation**

- Vérifié l'ensemble des ressources modifier au besoin sinon créer la vm.

**Version de windows**

- Pour une version légère de windows nous utiliserons *windows famille N*

**Ajout de carte réseau**

- Dans proxmox;

*Éteignez la vm puis, cliquez sur **Hardware**, ajoutez **CD/DVD Drive** séléctionnez le driver iso **virtio-win-0.1.302.iso**.*

- Installer l'iso puis redémarrez la vm pour appliquer les modifications. 

**Configuration réseaux**

``` IP fixe:``` 
- Dans *panneau de configuration* > *Réseau et internet* > *Centre réseau et partage* > *Ethernet* > *Propriétés* > *Protocole internet version 4 (TCP/IPv4)* > *Utiliser l'adresse IP suivante*

- **Adresse IP:** ```172.28.64.3``` 
- **Masque de sous-réseau:** ```255.255.255.0``` 
- **Passerelle par défaut** ```172.28.64.254``` 
- **Utiliser l'adresse de serveur DNS suivante:** ```172.28.65.1``` 

``` IP automatique:``` 
- Dans *panneau de configuration* > *Réseau et internet* > *Centre réseau et partage* > *Ethernet* > *Propriétés* > *Protocole internet version 4 (TCP/IPv4)* > *Obtenir une adresse IP automatiquement*

- **Utiliser l'adresse de serveur DNS suivante:** ```172.28.65.1``` 
