# Création de VM Linux sur proxmox

**VM temple:**

 ```Général: ``` *Noeud pour héberger la VM: (proxmox-01,proxmox-02..)*

- A noter: N'hésitez pas à changer de noeud pour effectuer de la haute disponibilité en cas de défaillance d'un noeud

```VM ID: ``` *ID du vlan ou au choix*

 ```Nom: ``` *TRS_NOM_VM*

- Par convention de nommage, nous affectons le nom de notre secteur à l'en tête du nom de la vm, dans notre cas *"TRS"*

 ```Pool de ressources: ``` *"groupe-tours-2027"*


**Système d'exploitation**

 ```Stockage:``` *local-data*

 ```Image-ISO:``` *debian-13.6.0-amd64-netinst.iso*

 ```Type:``` *Linux*

**Disques**

 ```Stockage:``` *zfs-1*

 ```Taille du disque:``` *8*

**Processeur**

 ```Socket:``` *1*

 ```Coeurs:``` *2*

**Mémoire**

```2048 MIB (2go)```

**Réseau**

```Pont:``` *Séléctionner le vlan requis (sptrs230,231..)*

```Modèle:``` *Intel E1000*

**Confirmation**

- Vérifié l'ensemble des ressources modifier au besoin sinon créer la vm.
