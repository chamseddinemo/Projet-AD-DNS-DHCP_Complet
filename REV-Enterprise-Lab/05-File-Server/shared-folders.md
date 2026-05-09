# Configuration des dossiers partagés

## 📋 Description

Ce document présente la configuration des dossiers partagés sur le serveur de fichiers REV-Enterprise-Lab pour l'accès centralisé aux données.

## 🎯 Objectifs

- Créer des partages sécurisés par département
- Configurer les permissions NTFS appropriées
- Implémenter des quotas de disque
- Faciliter l'accès aux données partagées

## 📁 Structure des dossiers partagés

### Arborescence principale
```
D:\Shares\
├── Public\
│   ├── Documents\
│   ├── Images\
│   └── Templates\
├── HR\
│   ├── Employees\
│   ├── Policies\
│   └── Reports\
├── HK\
│   ├── Schedules\
│   ├── Supplies\
│   └── Reports\
├── Sales\
│   ├── Clients\
│   ├── Proposals\
│   └── Reports\
└── IT\
    ├── Documentation\
    ├── Scripts\
    └── Tools\
```

## 📝 Procédure de création des partages

### Étape 1 : Création des dossiers
1. **Ouvrir l'Explorateur de fichiers**
2. **Naviguer vers** D:\Shares
3. **Créer les dossiers** selon la structure ci-dessus
4. **Définir les permissions NTFS** de base

### Étape 2 : Configuration des permissions NTFS

#### Dossier Public
```powershell
# Créer le dossier Public
New-Item -Path "D:\Shares\Public" -ItemType Directory -Force

# Configurer les permissions NTFS
icacls "D:\Shares\Public" /grant "Authenticated Users:(OI)(CI)(M)"
icacls "D:\Shares\Public" /grant "Administrators:(OI)(CI)(F)"
icacls "D:\Shares\Public" /grant "SYSTEM:(OI)(CI)(F)"
```

#### Dossier HR
```powershell
# Créer le dossier HR
New-Item -Path "D:\Shares\HR" -ItemType Directory -Force

# Configurer les permissions NTFS
icacls "D:\Shares\HR" /grant "HR-Group:(OI)(CI)(M)"
icacls "D:\Shares\HR" /grant "Administrators:(OI)(CI)(F)"
icacls "D:\Shares\HR" /grant "SYSTEM:(OI)(CI)(F)"
icacls "D:\Shares\HR" /deny "Authenticated Users:(OI)(CI)(M)"
```

### Étape 3 : Partage des dossiers

#### Partage Public
```powershell
# Créer le partage Public
New-SmbShare -Name "Public" -Path "D:\Shares\Public" -FullAccess "Authenticated Users" -ReadAccess "Everyone"
```

#### Partage HR
```powershell
# Créer le partage HR
New-SmbShare -Name "HR" -Path "D:\Shares\HR" -FullAccess "HR-Group" -ReadAccess "Administrators"
```

## 🔐 Configuration des permissions

### Tableau des permissions

| Partage | Groupe | Permissions NTFS | Permissions de partage |
|---------|--------|------------------|------------------------|
| Public | Authenticated Users | Modifier | Contrôle total |
| Public | Administrateurs | Contrôle total | Contrôle total |
| HR | HR-Group | Modifier | Contrôle total |
| HR | Administrateurs | Contrôle total | Contrôle total |
| HK | HK-Group | Modifier | Contrôle total |
| HK | Administrateurs | Contrôle total | Contrôle total |
| Sales | Sales-Group | Modifier | Contrôle total |
| Sales | Administrateurs | Contrôle total | Contrôle total |
| IT | IT-Group | Contrôle total | Contrôle total |
| IT | Administrateurs | Contrôle total | Contrôle total |

### Permissions spécifiques

#### Dossier Public
- **Authenticated Users** : Lecture, écriture, modification
- **Administrateurs** : Contrôle total
- **SYSTEM** : Contrôle total
- **Accès anonyme** : Aucun

#### Dossiers départementaux
- **Groupe du département** : Lecture, écriture, modification
- **Administrateurs** : Contrôle total
- **SYSTEM** : Contrôle total
- **Autres utilisateurs** : Aucun accès

## 💾 Configuration des quotas

### Paramètres généraux
- **Quota par défaut** : 2GB par utilisateur
- **Seuil d'avertissement** : 1.8GB
- **Seuil de blocage** : 2GB
- **Dépassement autorisé** : Non

### Configuration des quotas
```powershell
# Activer les quotas sur le volume
fsutil quota enforce D:

# Configurer les quotas par utilisateur
fsutil quota modify D: 2147483648 2147483648 1932735283 HR-Group
fsutil quota modify D: 2147483648 2147483648 1932735283 HK-Group
fsutil quota modify D: 2147483648 2147483648 1932735283 Sales-Group
```

### Rapports de quotas
- **Rapport quotidien** : Utilisation des quotas
- **Alertes** : À 80% et 100% d'utilisation
- **Nettoyage** : Fichiers temporaires automatiques

## 📊 Monitoring et maintenance

### Surveillance des partages
```powershell
# Obtenir les statistiques des partages
Get-SmbShare | Select-Object Name, Path, Description

# Obtenir les sessions actives
Get-SmbSession | Select-Object UserName, ClientComputerName, NumOpens

# Obtenir les fichiers ouverts
Get-SmbOpenFile | Select-Object Path, ClientComputerName, SessionID
```

### Script de monitoring
```powershell
# Script pour surveiller l'utilisation des partages
$shares = Get-SmbShare
foreach ($share in $shares) {
    $usage = Get-ChildItem -Path $share.Path -Recurse -ErrorAction SilentlyContinue | 
             Measure-Object -Property Length -Sum
    $sizeGB = [math]::Round($usage.Sum / 1GB, 2)
    Write-Host "Partage $($share.Name) : $sizeGB GB utilisés"
}
```

### Maintenance régulière
- **Nettoyage des fichiers temporaires** : Quotidien
- **Vérification des permissions** : Hebdomadaire
- **Analyse des quotas** : Mensuelle
- **Sauvegarde des partages** : Quotidienne

## 🔧 Mappage des lecteurs réseau

### Configuration via GPO
1. **Créer un GPO** : "REV-Drive-Mapping"
2. **Éditer le GPO**
3. **Naviguer vers** : Configuration utilisateur → Stratégies → Paramètres Windows → Scripts (connexion/déconnexion)
4. **Configurer** le script de connexion

### Script de mappage
```powershell
# Script PowerShell pour mapper les lecteurs
param(
    [string]$Department = ""
)

# Mapper le lecteur H (personnel)
if ($Department -eq "HR") {
    New-PSDrive -Name "H" -PSProvider FileSystem -Root "\\file.rev.local\HR\%username%" -Persist
} elseif ($Department -eq "HK") {
    New-PSDrive -Name "H" -PSProvider FileSystem -Root "\\file.rev.local\HK\%username%" -Persist
} elseif ($Department -eq "Sales") {
    New-PSDrive -Name "H" -PSProvider FileSystem -Root "\\file.rev.local\Sales\%username%" -Persist
}

# Mapper le lecteur P (partage départemental)
if ($Department -eq "HR") {
    New-PSDrive -Name "P" -PSProvider FileSystem -Root "\\file.rev.local\HR" -Persist
} elseif ($Department -eq "HK") {
    New-PSDrive -Name "P" -PSProvider FileSystem -Root "\\file.rev.local\HK" -Persist
} elseif ($Department -eq "Sales") {
    New-PSDrive -Name "P" -PSProvider FileSystem -Root "\\file.rev.local\Sales" -Persist
}

# Mapper le lecteur S (partage public)
New-PSDrive -Name "S" -PSProvider FileSystem -Root "\\file.rev.local\Public" -Persist
```

## 🚨 Dépannage

### Problèmes courants
1. **Accès refusé au partage**
   - Vérifier les permissions NTFS
   - Confirmer les permissions de partage
   - Valider l'appartenance aux groupes

2. **Lecteurs réseau non mappés**
   - Vérifier le script de connexion
   - Confirmer la connectivité réseau
   - Valider les permissions d'exécution

3. **Quotas dépassés**
   - Vérifier l'utilisation du disque
   - Nettoyer les fichiers inutiles
   - Ajuster les quotas si nécessaire

### Commandes utiles
```powershell
# Vérifier les permissions NTFS
icacls "D:\Shares\HR"

# Vérifier les permissions de partage
Get-SmbShareAccess -Name "HR"

# Réinitialiser les permissions
icacls "D:\Shares\HR" /reset

# Vérifier l'utilisation des quotas
fsutil quota query D:
```

## 📸 Captures d'écran

### Gestion des partages
![Share Management](../screenshots/05-file-server/share-management.png)

### Configuration des permissions NTFS
![NTFS Permissions](../screenshots/05-file-server/ntfs-permissions.png)

### Configuration des quotas
![Quota Configuration](../screenshots/05-file-server/quota-configuration.png)

---

**Document** : Configuration des dossiers partagés  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
