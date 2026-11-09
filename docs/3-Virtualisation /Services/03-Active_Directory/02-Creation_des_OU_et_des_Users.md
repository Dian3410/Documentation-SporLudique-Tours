# Création Automatisée des Unités Organisationnelles et des Utilisateurs

Dans le cadre du déploiement de l'infrastructure du site de Tours (domaine `trs.tours.sportludique.fr`), la création des comptes utilisateurs et de l'arborescence des Unités Organisationnelles (OU) a été automatisée via un script PowerShell.

Cette démarche permet de garantir :
- Un gain de temps significatif lors de l'importation massive des collaborateurs.
- Une standardisation des identifiants (SamAccountName et UPN).
- Un placement automatique et dynamique des utilisateurs dans l'OU correspondant à leur service.

## 1. Structure du fichier source (CSV)

Le script se base sur un fichier d'export CSV (séparateur point-virgule) listant l'ensemble du personnel. Le fichier doit impérativement contenir les entêtes suivantes : `NOM;PRENOM;SERVICE`.

**Exemple de structure `ImportADUsers.csv` :**
```
csv
NOM;PRENOM;SERVICE
BADOLA;Jean-Eric;PRODUCTION
BEUMENN;Pauline;INFORMATIQUE_DSI
BEROCHE JECF;Joëlle;RESSOURCES_HUMAINES
BOUDERGUA;Maryse;DIRECTION_ADMINISTRATION
BRAEUX;Grands Parents;COMPTABILITE_FINANCE
```

## 2. Création du script powershell

Le script suivant lit le fichier CSV, crée l'arborescence des services si elle n'existe pas, génère des identifiants propres (sans accents ni espaces, limités à 20 caractères) et provisionne les comptes dans l'Active Directory.

### script powershell
```
#1.PARAMÈTRES DE BASE

$Csvfile = "C:\temp\ImportADUsers.csv"
$Users = Import-Csv$Csvfile -Delimiter ";"

$Password = "mot_de_passe_à_définir"

#Suffixe UPN correspondant au domaine du site de Tours
$UPNSuffix = "trs.tours.sportludique.fr" 
#Racine du domaine en format LDAP
$DomainRoot = "DC=trs,DC=tours,DC=sportludique,DC=fr"

# 2. FONCTIONS UTILITAIRES

# Supprime les accents, les espaces et les tirets pour générer des logins propres
Function Clean-String {
    param([string]$Text)
    $Normalized =$Text.Normalize([Text.NormalizationForm]::FormD)
    $Cleaned = -join ($Normalized.ToCharArray() | Where-Object { 
        [Globalization.CharUnicodeInfo]::GetUnicodeCategory($_) -ne [Globalization.UnicodeCategory]::NonSpacingMark 
    })
    return $Cleaned -replace " ", "" -replace "-", "" -replace "'", ""
}


#3. CRÉATION DE L'ARBORESCENCE (OU)

Import-Module ActiveDirectory

$ParentOUName = "Services"
$ParentOUPath = "OU=$ParentOUName,$DomainRoot"

#Création de l'OU racine "Services" si inexistante
if (-not (Get-ADOrganizationalUnit -Filter "Name -eq '$ParentOUName'")) {
    New-ADOrganizationalUnit -Name $ParentOUName -Path$DomainRoot
    Write-Host "Création de l'OU principale : $ParentOUName" -ForegroundColor Cyan
}


#4. BOUCLE D'IMPORTATION DES UTILISATEURS
foreach ($User in$Users) {
    try {
        # --- GÉNÉRATION DE L'IDENTIFIANT ---
        $cleanPrenom = Clean-String -Text$User.PRENOM
        $cleanNom = Clean-String -Text$User.NOM
        $samAccount = "$($cleanPrenom.ToLower()).$($cleanNom.ToLower())"
        
        # Limite AD : le SamAccountName ne doit pas dépasser 20 caractères
        if ($samAccount.Length -gt 20) {
            $samAccount =$samAccount.Substring(0, 20)
        }
        
        # --- GESTION DES SOUS-UNITÉS ORGANISATIONNELLES ---
        $OUName =$User.SERVICE
        $OUPath = "OU=$OUName,$ParentOUPath" 

        # Création de l'OU du service à la volée si elle n'existe pas
        if (-not (Get-ADOrganizationalUnit -Filter "Name -eq '$OUName'")) {
            New-ADOrganizationalUnit -Name $OUName -Path$ParentOUPath
            Write-Host "Création de la sous-OU du service : $OUName" -ForegroundColor Cyan
        }

        # --- PRÉPARATION DES ATTRIBUTS ---
        $NewUserParams = @{
            Name                  = "$($User.PRENOM) $($User.NOM)"
            GivenName             = $User.PRENOM
            Surname               = $User.NOM
            DisplayName           = "$($User.PRENOM) $($User.NOM)"
            SamAccountName        = $samAccount
            UserPrincipalName     = "$samAccount@$UPNSuffix" 
            Department            = $User.SERVICE
            Path                  = $OUPath
            AccountPassword       = (ConvertTo-SecureString "$Password" -AsPlainText -Force)
            Enabled               = $true
            ChangePasswordAtLogon = $true # L'utilisateur change son mot de passe au 1er login
        }

        # --- CRÉATION DE L'UTILISATEUR ---
        if (Get-ADUser -Filter "SamAccountName -eq '$samAccount'") {
            Write-Host "L'utilisateur $samAccount existe déjà, ignoré." -ForegroundColor Yellow
        }
        else {
            New-ADUser @NewUserParams
            Write-Host "Succès : L'utilisateur $samAccount a été créé dans $OUName." -ForegroundColor Green
        }
    }
    catch {
        Write-Host "Échec de la création pour $($User.NOM) $($User.PRENOM) - $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

## 3. Explication des composants clés

### Normalisation des identifiants (Clean-String)
L'Active Directory n'autorise pas les espaces, les accents ou la plupart des caractères spéciaux dans les noms de connexion (sAMAccountName). Le script intègre une fonction qui :

- Convertit les caractères accentués en leur équivalent standard (ex: é -> e).

- Supprime les espaces et les tirets (ex: Jean-Eric -> jeaneric).

- Tronque la chaîne générée à 20 caractères maximum pour respecter les limites strictes du système pré-Windows 2000 toujours en vigueur.

### Création dynamique de l'arborescence
Le script ne nécessite pas de créer les Unités Organisationnelles à l'avance. Il lit la valeur de la colonne SERVICE dans le CSV. S'il rencontre un service inédit, il l'instancie automatiquement en tant que sous-OU dans l'OU parente Services.