# Restrictions utilisateur

## 📋 Description

Ce document présente les restrictions utilisateur implémentées dans le domaine REV.LOCAL pour contrôler l'accès aux fonctionnalités système et maintenir la sécurité.

## 🎯 Objectifs

- Désactiver l'invite de commandes pour les départements non techniques
- Restreindre l'accès au panneau de configuration
- Limiter les options de menu contextuel
- Contrôler l'accès aux outils système

### Configuration des quotas
![Quota Configuration](../Capture%20d%27écran/quota/Capture%20d%27écran%20(477).png)

## 🚫 Restrictions par département

### HR et Sales (Restrictions strictes)
- **Invite de commandes** : Désactivée
- **Panneau de configuration** : Désactivé
- **Gestionnaire de tâches** : Désactivé
- **Éditeur de registre** : Désactivé
- **Options Internet** : Limitées

### HK (Restrictions modérées)
- **Invite de commandes** : Désactivée
- **Panneau de configuration** : Accès limité
- **Gestionnaire de tâches** : Désactivé
- **Éditeur de registre** : Désactivé

### IT (Accès complet)
- **Toutes les fonctionnalités** : Activées
- **Outils d'administration** : Disponibles
- **Accès système** : Complet

## 📝 Procédure de configuration

### Création du GPO de restrictions
1. **Ouvrir "Gestion des stratégies de groupe"**
2. **Créer un nouveau GPO** : "REV-User-Restrictions"
3. **Lier le GPO** aux OU appropriées

### Configuration des restrictions
1. **Éditer le GPO**
2. **Naviguer vers** : Configuration utilisateur → Stratégies → Modèles d'administration → Panneau de configuration

## 🔧 Restrictions spécifiques

### Invite de commandes
**Chemin** : Configuration utilisateur → Stratégies → Modèles d'administration → Système
- **Empêcher l'accès à l'invite de commandes** : Activé
- **Appliquer à** : HR-Group, Sales-Group, HK-Group
- **Exception** : IT-Group

### Panneau de configuration
**Chemin** : Configuration utilisateur → Stratégies → Modèles d'administration → Panneau de configuration
- **Prohiber l'accès au Panneau de configuration et PC** : Activé
- **Appliquer à** : HR-Group, Sales-Group
- **Exception** : IT-Group

### Gestionnaire de tâches
**Chemin** : Configuration utilisateur → Stratégies → Modèles d'administration → Système
- **Empêcher l'accès au Gestionnaire de tâches** : Activé
- **Appliquer à** : HR-Group, Sales-Group, HK-Group
- **Exception** : IT-Group

### Éditeur de registre
**Chemin** : Configuration utilisateur → Stratégies → Modèles d'administration → Système
- **Empêcher l'accès aux outils de modification du Registre** : Activé
- **Appliquer à** : HR-Group, Sales-Group, HK-Group
- **Exception** : IT-Group

## 🖱️ Restrictions du menu contextuel

### Suppression de "Propriétés" du PC
**Chemin** : Configuration utilisateur → Stratégies → Modèles d'administration → Bureau
- **Supprimer la commande "Propriétés" de l'élément Ordinateur** : Activé
- **Appliquer à** : HR-Group, Sales-Group, HK-Group

### Restrictions du menu Démarrer
**Chemin** : Configuration utilisateur → Stratégies → Modèles d'administration → Menu Démarrer et barre des tâches
- **Supprimer "Exécuter" du menu Démarrer** : Activé
- **Supprimer "Rechercher" du menu Démarrer** : Activé
- **Appliquer à** : HR-Group, Sales-Group

## 🔒 Restrictions des explorateurs

### Options des dossiers
**Chemin** : Configuration utilisateur → Stratégies → Modèles d'administration → Composants Windows → Explorateur Windows
- **Supprimer l'onglet "Outils des dossiers" des Options des dossiers** : Activé
- **Appliquer à** : HR-Group, Sales-Group, HK-Group

### Restrictions Internet Explorer
**Chemin** : Configuration utilisateur → Stratégies → Modèles d'administration → Composants Windows → Internet Explorer
- **Désactiver la page "Options Internet"** : Activé
- **Empêcher l'exécution des programmes** : Activé
- **Appliquer à** : HR-Group, Sales-Group

## 📊 Tableau des restrictions

| Fonctionnalité | HR | Sales | HK | IT |
|----------------|----|-------|----|----|
| Invite de commandes | ❌ | ❌ | ❌ | ✅ |
| Panneau de configuration | ❌ | ❌ | ⚠️ | ✅ |
| Gestionnaire de tâches | ❌ | ❌ | ❌ | ✅ |
| Éditeur de registre | ❌ | ❌ | ❌ | ✅ |
| Propriétés PC | ❌ | ❌ | ❌ | ✅ |
| Options Internet | ❌ | ❌ | ⚠️ | ✅ |
| Menu Exécuter | ❌ | ❌ | ⚠️ | ✅ |

Légende : ✅ = Activé, ❌ = Désactivé, ⚠️ = Limité

## 🔄 Filtrage de sécurité

### Configuration du filtrage
1. **Dans le GPO de restrictions**
2. **Clic droit sur le GPO → Filtrage de sécurité**
3. **Désactiver "Utilisateurs authentifiés"**
4. **Ajouter les groupes appropriés** :
   - HR-Group
   - Sales-Group
   - HK-Group

### Exceptions pour IT
1. **Créer un GPO séparé** : "REV-IT-Access"
2. **Lier uniquement à l'OU IT**
3. **Configurer les permissions inversées**

## 🚨 Dépannage

### Problèmes courants
1. **Restrictions trop strictes**
   - Vérifier le filtrage de sécurité
   - Confirmer l'appartenance aux groupes
   - Tester avec différents utilisateurs

2. **IT n'a pas accès**
   - Vérifier la liaison du GPO IT-Access
   - Confirmer l'ordre de traitement des GPO
   - Valider les permissions de filtrage

3. **Restrictions non appliquées**
   - Forcer la mise à jour : `gpupdate /force`
   - Vérifier l'ordre de liaison des GPO
   - Confirmer l'héritage des GPO

### Commandes utiles
```powershell
# Vérifier les GPO appliquées
gpresult /r

# Vérifier les restrictions spécifiques
gpresult /scope user /v

# Forcer la mise à jour des politiques
gpupdate /force /target:user

# Réinitialiser les GPO locales
gpupdate /force /boot
```

## 📈 Monitoring des restrictions

### Surveillance des tentatives de contournement
1. **Auditer les accès** :
   - Tentatives d'accès aux outils système
   - Exécution de commandes bloquées
   - Modifications de registre tentées

### Script de monitoring
```powershell
# Script pour surveiller les tentatives de contournement
$blockedEvents = Get-WinEvent -LogName Application -MaxEvents 100 | Where-Object {
    $_.Message -like "*blocked*" -or $_.Message -like "*denied*"
}

foreach ($event in $blockedEvents) {
    Write-Warning "Tentative de contournement détectée : $($event.TimeCreated) - $($event.Message)"
}
```

## 📸 Captures d'écran

### Configuration des restrictions
![Restrictions Configuration](../screenshots/03-group-policy/restrictions-configuration.png)

### Filtrage de sécurité
![Security Filtering](../screenshots/03-group-policy/security-filtering.png)

### Validation des restrictions
![Restrictions Validation](../screenshots/03-group-policy/restrictions-validation.png)

---

**Document** : Restrictions utilisateur  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
