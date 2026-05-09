# Restrictions logicielles et médias

## 📋 Description

Ce document explique comment bloquer l'ajout de fichiers exécutables et médias par les utilisateurs via les stratégies de groupe.

## 🎯 Objectifs

- Empêcher l'installation de logiciels non autorisés
- Bloquer l'exécution de fichiers exécutables externes
- Restreindre l'accès aux fichiers médias
- Maintenir la sécurité du poste de travail

## 🔐 Configuration des restrictions logicielles

### Création du GPO
1. **Ouvrir "Gestion des stratégies de groupe"**
2. **Créer un nouveau GPO** : "REV-Software-Restrictions"
3. **Lier le GPO** aux OU HR, HK, Sales

### Configuration des restrictions
1. **Éditer le GPO**
2. **Naviguer vers** : Configuration utilisateur → Stratégies → Paramètres Windows → Sécurité → Stratégies de restriction logicielle

3. **Configurer les règles** :
   - **Niveau d'application** : Non configuré (par défaut)
   - **Type d'application** : Uniquement les fichiers exécutables
   - **Exécution des logiciels** : Interdite

## 📝 Procédure détaillée

### Étape 1 : Configuration principale
1. **Dans le GPO REV-Software-Restrictions**
2. **Stratégies de restriction logicielle** :
   - Double-cliquer sur "Exécution des logiciels"
   - Sélectionner "Interdite"
   - Cliquer sur "OK"

### Étape 2 : Règles de chemin
1. **Naviguer vers** : Règles de chemin
2. **Clic droit → Nouvelle règle de chemin**
3. **Configurer les chemins autorisés** :
   - **Chemin** : %ProgramFiles%\*
   - **Niveau de sécurité** : Non restreint
   - **Description** : Programmes installés légalement

4. **Ajouter d'autres chemins autorisés** :
   - **Chemin** : %SystemRoot%\*
   - **Niveau de sécurité** : Non restreint
   - **Description** : Fichiers système Windows

### Étape 3 : Règles de type de fichier
1. **Naviguer vers** : Règles de type de fichier
2. **Clic droit → Nouvelle règle de type de fichier**
3. **Configurer les extensions bloquées** :
   - **Extension** : .exe
   - **Niveau de sécurité** : Interdit
   - **Description** : Fichiers exécutables

4. **Ajouter d'autres extensions** :
   - **Extension** : .msi
   - **Niveau de sécurité** : Interdit
   - **Description** : Fichiers d'installation

   - **Extension** : .bat
   - **Niveau de sécurité** : Interdit
   - **Description** : Fichiers batch

   - **Extension** : .cmd
   - **Niveau de sécurité** : Interdit
   - **Description** : Fichiers de commande

## 🎵 Restrictions des fichiers médias

### Règles pour les fichiers médias
1. **Ajouter les extensions médias à bloquer** :
   - **Extension** : .mp3
   - **Niveau de sécurité** : Interdit
   - **Description** : Fichiers audio MP3

   - **Extension** : .mp4
   - **Niveau de sécurité** : Interdit
   - **Description** : Fichiers vidéo MP4

   - **Extension** : .avi
   - **Niveau de sécurité** : Interdit
   - **Description** : Fichiers vidéo AVI

   - **Extension** : .mkv
   - **Niveau de sécurité** : Interdit
   - **Description** : Fichiers vidéo MKV

## 🔧 Configuration avancée

### Exceptions pour les utilisateurs IT
1. **Créer un GPO séparé** : "REV-IT-Software-Allowed"
2. **Lier uniquement** à l'OU IT
3. **Configurer** :
   - **Exécution des logiciels** : Non configurée
   - **Règles de chemin** : Autoriser tous les chemins
   - **Règles de type de fichier** : Aucune restriction

### Filtrage de sécurité
1. **Dans le GPO REV-Software-Restrictions**
2. **Clic droit → Filtrage de sécurité**
3. **Désactiver "Utilisateurs authentifiés"**
4. **Ajouter les groupes** :
   - HR-Group : Appliquer les restrictions
   - HK-Group : Appliquer les restrictions
   - Sales-Group : Appliquer les restrictions

5. **Exclure IT-Group** :
   - Ne PAS ajouter IT-Group au filtrage de sécurité

## 📊 Tableau des restrictions

| Groupe | Exécutables (.exe) | Installations (.msi) | Médias (.mp3/.mp4) | Scripts (.bat/.cmd) | GPO appliqué |
|---------|-------------------|-------------------|-------------------|-------------------|---------------|
| HR-Group | ❌ Bloqué | ❌ Bloqué | ❌ Bloqué | ❌ Bloqué | REV-Software-Restrictions |
| HK-Group | ❌ Bloqué | ❌ Bloqué | ❌ Bloqué | ❌ Bloqué | REV-Software-Restrictions |
| Sales-Group | ❌ Bloqué | ❌ Bloqué | ❌ Bloqué | ❌ Bloqué | REV-Software-Restrictions |
| IT-Group | ✅ Autorisé | ✅ Autorisé | ✅ Autorisé | ✅ Autorisé | REV-IT-Software-Allowed |

## 🔍 Validation des restrictions

### Test pour les utilisateurs HR
1. **Se connecter** en tant qu'utilisateur HR
2. **Tenter d'exécuter** un fichier .exe externe
3. **Vérifier** que l'exécution est bloquée
4. **Tenter d'installer** un logiciel
5. **Vérifier** que l'installation échoue

### Test pour les utilisateurs IT
1. **Se connecter** en tant qu'utilisateur IT
2. **Tenter d'exécuter** un fichier .exe
3. **Vérifier** que l'exécution réussit
4. **Tenter d'installer** un logiciel
5. **Vérifier** que l'installation réussit

### Commandes de validation
```powershell
# Vérifier les GPO appliquées
gpresult /r

# Vérifier les restrictions de logiciels
Get-AppLockerPolicy -Effective

# Tester l'exécution d'un fichier
Start-Process -FilePath "test.exe" -ErrorAction SilentlyContinue
```

## 🚨 Dépannage

### Problèmes courants
1. **Restrictions trop strictes**
   - Ajouter des exceptions pour les logiciels légitimes
   - Créer des règles de chemin spécifiques
   - Valider les applications métier

2. **IT n'a pas accès aux logiciels**
   - Vérifier la liaison du GPO IT
   - Confirmer le filtrage de sécurité
   - Valider l'ordre de traitement des GPO

3. **Applications métier bloquées**
   - Ajouter des règles de chemin spécifiques
   - Créer des règles de hachage
   - Valider les chemins d'installation

### Commandes utiles
```powershell
# Forcer la mise à jour des GPO
gpupdate /force

# Vérifier les restrictions actives
Get-AppLockerFileInformation

# Réinitialiser les politiques de restriction
Set-AppLockerPolicy -PolicyType None

# Journaliser les tentatives bloquées
Get-WinEvent -LogName Microsoft-Windows-AppLocker/MSI and Script -MaxEvents 20
```

## 📈 Monitoring

### Surveillance des tentatives bloquées
```powershell
# Script pour surveiller les tentatives d'exécution bloquées
$events = Get-WinEvent -LogName Microsoft-Windows-AppLocker/EXE and DLL -MaxEvents 100 | Where-Object {
    $_.LevelDisplayName -eq "Warning"
}

foreach ($event in $events) {
    Write-Warning "Tentative d'exécution bloquée : $($event.TimeCreated) - $($event.Message)"
}
```

### Rapport de conformité
- **Utilisateurs HR** : 100% de restriction appliquée
- **Utilisateurs HK** : 100% de restriction appliquée
- **Utilisateurs Sales** : 100% de restriction appliquée
- **Utilisateurs IT** : 100% d'accès autorisé
- **Alertes** : Tentatives de contournement

## 📸 Captures d'écran

### Configuration des restrictions logicielles
![Software Restrictions](../screenshots/03-group-policy/software-restrictions-config.png)

### Règles de type de fichier
![File Type Rules](../screenshots/03-group-policy/file-type-rules.png)

### Filtrage de sécurité
![Security Filtering](../screenshots/03-group-policy/security-filtering-restrictions.png)

---

**Document** : Restrictions logicielles et médias  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
