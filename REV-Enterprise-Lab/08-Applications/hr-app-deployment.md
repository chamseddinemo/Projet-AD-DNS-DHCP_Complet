# Déploiement de l'application HR

## 📋 Description

Ce document explique comment créer un raccourci vers hrapp.rev.local pour tous les utilisateurs HK sur leur bureau via les stratégies de groupe.

## 🎯 Objectifs

- Créer un raccourci vers hrapp.rev.local pour tous les utilisateurs HK
- Déployer le raccourci via GPO
- Assurer l'accès facile à l'application HR
- Valider le fonctionnement du raccourci

## 🌐 Configuration DNS

### Création de l'enregistrement DNS
1. **Ouvrir la console DNS**
2. **Naviguer vers la zone rev.local**
3. **Créer un enregistrement A** :
   - Nom : hrapp
   - Type : A
   - Adresse IP : 192.108.1.10 (serveur PDC)
   - Description : Application HR

### Commande PowerShell
```powershell
# Créer l'enregistrement DNS pour hrapp
Add-DnsServerResourceRecord -ZoneName rev.local -Name hrapp -A -IPv4Address 192.108.1.10
```

## 📝 Procédure de déploiement

### Étape 1 : Création du GPO
1. **Ouvrir "Gestion des stratégies de groupe"**
2. **Créer un nouveau GPO** : "REV-HR-App-Shortcut"
3. **Lier le GPO** à l'OU HK

### Étape 2 : Configuration du raccourci
1. **Éditer le GPO**
2. **Naviguer vers** : Configuration utilisateur → Stratégies → Paramètres Windows → Raccourcis
3. **Clic droit → Nouveau raccourci**

4. **Configurer le raccourci** :
   - **Nom** : HR Application
   - **Cible** : http://hrapp.rev.local
   - **Démarrer dans** : Laisser vide
   - **Raccourci clavier** : Laisser vide
   - **Commentaire** : Accès à l'application HR

### Étape 3 : Filtrage de sécurité
1. **Clic droit sur le GPO → Filtrage de sécurité**
2. **Désactiver "Utilisateurs authentifiés"**
3. **Ajouter le groupe** : HK-Group
4. **Appliquer** les permissions

## 🔧 Configuration avancée

### Options du raccourci
- **Icône** : Icône personnalisée HR
- **Exécuter** : Normal
- **Changer d'icône** : Icône d'application

### Script alternatif
Si le GPO de raccourci ne fonctionne pas, utiliser un script :

```powershell
# Script de création de raccourci
$shortcutPath = "$env:USERPROFILE\Desktop\HR Application.lnk"
$targetPath = "http://hrapp.rev.local"

# Créer l'objet raccourci
$shell = New-Object -ComObject WScript.Shell
$shortcut = $shell.CreateShortcut($shortcutPath)
$shortcut.TargetPath = $targetPath
$shortcut.Description = "Accès à l'application HR"
$shortcut.Save()
```

### Configuration du script dans GPO
1. **Naviguer vers** : Configuration utilisateur → Stratégies → Paramètres Windows → Scripts
2. **Ajouter le script** à l'ouverture de session
3. **Spécifier le chemin** du script PowerShell

## ✅ Validation

### Test du raccourci
1. **Se connecter** en tant qu'utilisateur HK
2. **Vérifier** que le raccourci apparaît sur le bureau
3. **Cliquer** sur le raccourci
4. **Valider** l'ouverture de hrapp.rev.local

### Test de résolution DNS
```powershell
# Tester la résolution DNS
nslookup hrapp.rev.local

# Test de connectivité
Test-NetConnection -ComputerName hrapp.rev.local -Port 80
```

### Test du GPO
```powershell
# Vérifier les GPO appliquées
gpresult /r

# Forcer la mise à jour
gpupdate /force
```

## 🚨 Dépannage

### Problèmes courants
1. **Raccourci n'apparaît pas**
   - Vérifier la liaison du GPO
   - Confirmer le filtrage de sécurité
   - Forcer la mise à jour : `gpupdate /force`

2. **Le raccourci ne fonctionne pas**
   - Vérifier l'enregistrement DNS
   - Confirmer la connectivité réseau
   - Valider l'URL cible

3. **DNS ne résout pas**
   - Vérifier l'enregistrement A
   - Confirmer la configuration DNS du client
   - Tester avec `nslookup`

### Commandes utiles
```powershell
# Vérifier l'enregistrement DNS
Get-DnsServerResourceRecord -ZoneName rev.local -Name hrapp

# Supprimer et recréer l'enregistrement
Remove-DnsServerResourceRecord -ZoneName rev.local -Name hrapp -RRType A -Force
Add-DnsServerResourceRecord -ZoneName rev.local -Name hrapp -A -IPv4Address 192.108.1.10

# Vérifier les GPO appliquées
Get-GPResultantSetOfPolicy -User $env:USERNAME
```

## 📊 Monitoring

### Surveillance du déploiement
```powershell
# Script pour vérifier la présence du raccourci
$users = Get-ADUser -Filter {Enabled -eq $true} -Properties ProfilePath

foreach ($user in $users) {
    $desktopPath = "\\file.rev.local\Profiles\$($user.SamAccountName)\Desktop"
    $shortcutPath = "$desktopPath\HR Application.lnk"
    
    if (Test-Path $shortcutPath) {
        Write-Host "✅ Raccourci trouvé pour $($user.SamAccountName)"
    } else {
        Write-Host "❌ Raccourci manquant pour $($user.SamAccountName)"
    }
}
```

### Rapport de déploiement
- **Utilisateurs ciblés** : Tous les membres de HK-Group
- **Raccourcis créés** : [Nombre]
- **Raccourcis manquants** : [Nombre]
- **Taux de réussite** : [Pourcentage]%

## 📸 Captures d'écran

### Configuration du GPO de raccourci
![Shortcut GPO](../screenshots/08-applications/shortcut-gpo.png)

### Enregistrement DNS
![DNS Record](../screenshots/08-applications/dns-record.png)

### Validation du raccourci
![Shortcut Validation](../screenshots/08-applications/shortcut-validation.png)

---

**Document** : Déploiement de l'application HR  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
