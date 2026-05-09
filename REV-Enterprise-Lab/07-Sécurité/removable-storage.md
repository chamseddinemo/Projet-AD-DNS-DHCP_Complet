# Restriction du stockage amovible

## 📋 Description

Ce document explique comment désactiver l'accès au stockage amovible pour les utilisateurs HR tout en excluant le groupe IT-Group.

## 🎯 Objectifs

- Désactiver l'accès USB pour les utilisateurs HR
- Autoriser l'accès USB pour les membres de IT-Group
- Maintenir la sécurité des données
- Prévenir la fuite de données

## 🔐 Configuration de la politique

### Création du GPO
1. **Ouvrir "Gestion des stratégies de groupe"**
2. **Créer un nouveau GPO** : "REV-USB-Restriction"
3. **Lier le GPO** à l'OU HR

### Configuration des restrictions
1. **Éditer le GPO**
2. **Naviguer vers** : Configuration ordinateur → Stratégies → Modèles d'administration → Système → Accès au stockage amovible

3. **Configurer les restrictions** :
   - **Accès au stockage amovible** : Désactivé
   - **Lecteurs de disquette** : Désactivé
   - **Lecteurs de CD/DVD** : Désactivé
   - **Périphériques USB** : Désactivés

## 📝 Procédure détaillée

### Étape 1 : Configuration principale
1. **Dans le GPO REV-USB-Restriction**
2. **Naviguer vers** : Configuration ordinateur → Stratégies → Modèles d'administration → Système
3. **Accès au stockage amovible** :
   - Double-cliquer sur "Accès au stockage amovible"
   - Sélectionner "Désactivé"
   - Cliquer sur "OK"

### Étape 2 : Configuration des lecteurs spécifiques
1. **Lecteurs de disquette** :
   - Naviguer vers : Accès au stockage amovible → Accès aux lecteurs de disquette
   - Double-cliquer et sélectionner "Désactivé"

2. **Lecteurs de CD/DVD** :
   - Naviguer vers : Accès au stockage amovible → Accès aux lecteurs de CD et DVD
   - Double-cliquer et sélectionner "Désactivé"

3. **Périphériques USB** :
   - Naviguer vers : Accès au stockage amovible → Accès aux périphériques de stockage USB
   - Double-cliquer et sélectionner "Désactivé"

### Étape 3 : Filtrage de sécurité
1. **Clic droit sur le GPO → Filtrage de sécurité**
2. **Désactiver "Utilisateurs authentifiés"**
3. **Ajouter les groupes** :
   - HR-Group : Appliquer la restriction
   - HK-Group : Appliquer la restriction
   - Sales-Group : Appliquer la restriction

4. **Exclure IT-Group** :
   - Ne PAS ajouter IT-Group au filtrage de sécurité
   - Créer un GPO séparé pour IT avec accès autorisé

## 🔧 Configuration alternative pour IT

### GPO IT avec accès USB
1. **Créer un GPO** : "REV-IT-USB-Allowed"
2. **Lier uniquement** à l'OU IT
3. **Configurer** :
   - Accès au stockage amovible : Activé
   - Lecteurs de disquette : Activé
   - Lecteurs de CD/DVD : Activé
   - Périphériques USB : Activés

## 📊 Tableau des permissions

| Groupe | Accès USB | Lecteurs CD/DVD | Lecteurs disquette | GPO appliqué |
|--------|------------|----------------|-------------------|---------------|
| HR-Group | ❌ Désactivé | ❌ Désactivé | ❌ Désactivé | REV-USB-Restriction |
| HK-Group | ❌ Désactivé | ❌ Désactivé | ❌ Désactivé | REV-USB-Restriction |
| Sales-Group | ❌ Désactivé | ❌ Désactivé | ❌ Désactivé | REV-USB-Restriction |
| IT-Group | ✅ Activé | ✅ Activé | ✅ Activé | REV-IT-USB-Allowed |

## 🔍 Validation des restrictions

### Test pour les utilisateurs HR
1. **Se connecter** en tant qu'utilisateur HR
2. **Brancher une clé USB**
3. **Vérifier** :
   - La clé n'apparaît pas dans l'Explorateur
   - Message d'accès refusé si tentative d'accès
   - Gestionnaire de périphériques ne montre pas la clé

### Test pour les utilisateurs IT
1. **Se connecter** en tant qu'utilisateur IT
2. **Brancher une clé USB**
3. **Vérifier** :
   - La clé apparaît dans l'Explorateur
   - Accès normal aux fichiers
   - Gestionnaire de périphériques montre la clé

### Commandes de validation
```powershell
# Vérifier les GPO appliquées
gpresult /r

# Vérifier les périphériques USB
Get-WmiObject -Class Win32_LogicalDisk | Where-Object {$_.DriveType -eq 2}

# Vérifier les restrictions de registre
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Policies\Microsoft\Windows\RemovableStorageDevices"
```

## 🚨 Dépannage

### Problèmes courants
1. **Restrictions non appliquées aux utilisateurs HR**
   - Vérifier la liaison du GPO
   - Confirmer le filtrage de sécurité
   - Forcer la mise à jour : `gpupdate /force`

2. **Utilisateurs IT n'ont pas accès USB**
   - Vérifier la liaison du GPO IT
   - Confirmer l'ordre de traitement des GPO
   - Valider l'appartenance à IT-Group

3. **Restrictions appliquées à tout le monde**
   - Vérifier le filtrage de sécurité
   - Confirmer que IT-Group est bien exclu
   - Créer un GPO de blocage pour IT

### Commandes utiles
```powershell
# Forcer la mise à jour des GPO
gpupdate /force

# Vérifier les GPO appliquées à un utilisateur
gpresult /user:username /r

# Nettoyer les politiques locales
gpupdate /target:computer /force

# Redémarrer le service de stratégie de groupe
Restart-Service -Name gpsvc
```

## 📈 Monitoring

### Surveillance des tentatives d'accès
```powershell
# Script pour surveiller les tentatives d'accès USB bloquées
$events = Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {
    $_.ProviderName -eq "Microsoft-Windows-Kernel-PnP" -and
    $_.Message -like "*USB*" -and
    $_.LevelDisplayName -eq "Warning"
}

foreach ($event in $events) {
    Write-Warning "Tentative d'accès USB bloquée : $($event.TimeCreated) - $($event.Message)"
}
```

### Rapport de conformité
- **Utilisateurs HR** : 100% de restriction appliquée
- **Utilisateurs IT** : 100% d'accès autorisé
- **Alertes** : Tentatives d'accès non autorisées
- **Maintenance** : Vérification mensuelle des GPO

## 📸 Captures d'écran

### Configuration des restrictions USB
![USB Restrictions](../screenshots/07-security/usb-restrictions.png)

### Filtrage de sécurité
![Security Filtering](../screenshots/07-security/security-filtering.png)

### Validation des restrictions
![Restrictions Validation](../screenshots/07-security/restrictions-validation.png)

---

**Document** : Restriction du stockage amovible  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
