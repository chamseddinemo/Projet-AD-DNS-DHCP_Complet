# Mappage de lecteurs réseau

## 📋 Description

Ce document explique comment mapper le lecteur H pour les utilisateurs HR vers le partage HR du département.

## 🎯 Objectifs

- Mapper le lecteur H pour tous les utilisateurs HR
- Pointer vers le partage HR du département
- Persister le mappage après redémarrage
- Faciliter l'accès aux fichiers HR

## 📁 Configuration du partage HR

### Création du dossier partagé
1. **Créer le dossier** : D:\Shares\HR
2. **Configurer les permissions NTFS** :
   - HR-Group : Modifier (créer, éditer)
   - HR-Group : Refuser Supprimer
   - Administrateurs : Contrôle total

### Création du partage réseau
```powershell
# Créer le partage HR avec permissions spécifiques
New-SmbShare -Name "HR" -Path "D:\Shares\HR" -FullAccess "HR-Group", "Administrators" -ReadAccess "Authenticated Users"
```

## 📝 Procédure de mappage via GPO

### Étape 1 : Création du GPO
1. **Ouvrir "Gestion des stratégies de groupe"**
2. **Créer un nouveau GPO** : "REV-HR-Drive-Mapping"
3. **Lier le GPO** à l'OU HR

### Étape 2 : Configuration du mappage
1. **Éditer le GPO**
2. **Naviguer vers** : Configuration utilisateur → Stratégies → Paramètres Windows → Lecteurs réseau
3. **Clic droit → Nouveau → Lecteur mappé**

4. **Configurer le lecteur H** :
   - **Action** : Créer
   - **Lecteur** : H:
   - **Chemin d'accès** : \\file.rev.local\HR
   - **Se reconnecter** : Coché
   - **Étiquette** : HR Department

### Étape 3 : Filtrage de sécurité
1. **Clic droit sur le GPO → Filtrage de sécurité**
2. **Désactiver "Utilisateurs authentifiés"**
3. **Ajouter le groupe** : HR-Group
4. **Appliquer** les permissions

## 🔧 Configuration alternative via script

### Script PowerShell
```powershell
# Script de mappage pour les utilisateurs HR
$driveLetter = "H:"
$sharePath = "\\file.rev.local\HR"
$persist = $true

try {
    # Vérifier si le lecteur existe déjà
    $existingDrive = Get-PSDrive -Name $driveLetter.Replace(":", "") -ErrorAction SilentlyContinue
    
    if ($existingDrive) {
        # Supprimer le mappage existant
        Remove-PSDrive -Name $driveLetter.Replace(":", "") -Force
        net use $driveLetter /delete /y
    }
    
    # Créer le nouveau mappage
    New-PSDrive -Name $driveLetter.Replace(":", "") -PSProvider FileSystem -Root $sharePath -Persist $persist
    
    Write-Host "Lecteur H: mappé avec succès vers $sharePath"
    
} catch {
    Write-Error "Erreur lors du mappage du lecteur H: $_"
}
```

### Configuration du script dans GPO
1. **Naviguer vers** : Configuration utilisateur → Stratégies → Paramètres Windows → Scripts
2. **Ajouter le script** à l'ouverture de session
3. **Spécifier le chemin** du script PowerShell
4. **Configurer** les paramètres si nécessaire

## 🔍 Validation du mappage

### Test pour les utilisateurs HR
1. **Se connecter** en tant qu'utilisateur HR
2. **Vérifier** que le lecteur H: apparaît dans l'Explorateur
3. **Accéder** au lecteur H:
4. **Tester** la création et modification de fichiers
5. **Vérifier** que la suppression est bloquée

### Commandes de validation
```powershell
# Vérifier les lecteurs réseau mappés
Get-PSDrive

# Vérifier les connexions réseau
net use

# Tester l'accès au partage
Test-Path "\\file.rev.local\HR"

# Vérifier les permissions
Get-Acl "\\file.rev.local\HR"
```

## 📊 Tableau des permissions

| Groupe | Permissions NTFS | Permissions de partage | Accès final |
|---------|------------------|----------------------|--------------|
| HR-Group | Modifier (créer, éditer) | Contrôle total | Créer, éditer, PAS supprimer |
| Administrateurs | Contrôle total | Contrôle total | Contrôle total |
| SYSTEM | Contrôle total | Contrôle total | Contrôle total |
| Autres utilisateurs | Aucun | Aucun | Aucun accès |

## 🚨 Dépannage

### Problèmes courants
1. **Lecteur H: n'apparaît pas**
   - Vérifier la liaison du GPO
   - Confirmer le filtrage de sécurité
   - Forcer la mise à jour : `gpupdate /force`

2. **Accès refusé au partage**
   - Vérifier les permissions NTFS
   - Confirmer les permissions de partage
   - Valider l'appartenance à HR-Group

3. **Utilisateurs peuvent supprimer des fichiers**
   - Vérifier les permissions NTFS
   - Confirmer la configuration "Refuser Supprimer"
   - Appliquer les permissions héritées

### Commandes utiles
```powershell
# Forcer la mise à jour des GPO
gpupdate /force

# Vérifier les GPO appliquées
gpresult /r

# Recréer le mappage réseau
net use H: \\file.rev.local\HR /persistent:yes

# Supprimer un mappage
net use H: /delete

# Vérifier les permissions NTFS
icacls "\\file.rev.local\HR"
```

## 📈 Monitoring

### Surveillance des mappages
```powershell
# Script pour vérifier les mappages des utilisateurs HR
$users = Get-ADGroupMember -Identity "HR-Group"

foreach ($user in $users) {
    $userName = $user.SamAccountName
    
    # Vérifier si le lecteur H est mappé
    try {
        $driveTest = Test-Path "H:\"
        if ($driveTest) {
            Write-Host "✅ Lecteur H: mappé pour $userName"
        } else {
            Write-Host "❌ Lecteur H: non mappé pour $userName"
        }
    } catch {
        Write-Host "⚠️ Erreur pour $userName : $_"
    }
}
```

### Rapport d'utilisation
- **Utilisateurs HR** : [Nombre] avec lecteur H: mappé
- **Accès réussis** : [Nombre] par jour
- **Erreurs d'accès** : [Nombre] par jour
- **Espace utilisé** : [Taille] sur le partage HR

## 📸 Captures d'écran

### Configuration du GPO de mappage
![Drive Mapping GPO](../screenshots/05-file-server/drive-mapping-gpo.png)

### Configuration des permissions NTFS
![NTFS Permissions](../screenshots/05-file-server/ntfs-permissions-hr.png)

### Validation du mappage
![Drive Mapping Validation](../screenshots/05-file-server/drive-mapping-validation.png)

---

**Document** : Mappage de lecteurs réseau  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
