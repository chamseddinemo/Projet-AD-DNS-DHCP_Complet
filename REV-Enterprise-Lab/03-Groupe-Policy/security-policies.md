# Politiques de sécurité

## 📋 Description

Ce document présente les politiques de sécurité implémentées dans le domaine REV.LOCAL pour renforcer la protection des ressources et des données.

## 🎯 Objectifs

- Implémenter des politiques de mot de passe robustes
- Configurer le verrouillage des comptes
- Sécuriser les sessions utilisateur
- Protéger contre les menaces courantes

## 🔐 Politique de mot de passe

### Configuration
- **Longueur minimale** : 6 caractères
- **Complexité** : Activée (majuscule, minuscule, chiffre, caractère spécial)
- **Historique** : Mémoriser les 3 derniers mots de passe
- **Durée de vie** : 60 jours
- **Avertissement** : 14 jours avant expiration

### Procédure de configuration
1. **Ouvrir "Gestion des stratégies de groupe"**
2. **Créer un nouveau GPO** : "REV-Security-Policy"
3. **Éditer la stratégie**
4. **Naviguer vers** : Configuration ordinateur → Stratégies → Paramètres Windows → Stratégies de compte → Stratégie de mot de passe

5. **Configurer les paramètres** :
   - Longueur minimale du mot de passe : 6
   - Complexité des mots de passe : Activé
   - Historique des mots de passe : Mémoriser 3
   - Durée de vie maximale : 60 jours
   - Durée de vie minimale : 1 jour

## 🔒 Politique de verrouillage de compte

### Configuration
- **Seuil de verrouillage** : 5 tentatives échouées
- **Durée de verrouillage** : 30 minutes
- **Compteur de réinitialisation** : 30 minutes

### Procédure de configuration
1. **Dans le même GPO**
2. **Naviguer vers** : Stratégie de verrouillage de compte
3. **Configurer les paramètres** :
   - Seuil de verrouillage : 5 tentatives
   - Durée de verrouillage : 30 minutes
   - Compteur de réinitialisation : 30 minutes

## 🖥️ Politiques de session

### Configuration du verrouillage d'écran
- **Délai d'inactivité** : 15 minutes
- **Verrouillage automatique** : Activé
- **Protection par mot de passe** : Requise

### Procédure de configuration
1. **Créer un nouveau GPO** : "REV-Session-Policy"
2. **Naviguer vers** : Configuration utilisateur → Stratégies → Paramètres Windows → Scripts
3. **Configurer** le script de verrouillage

### Script de verrouillage (screensaver.scr)
```powershell
# Script PowerShell pour le verrouillage automatique
param(
    [int]$TimeoutMinutes = 15
)

$TimeoutSeconds = $TimeoutMinutes * 60
while ($true) {
    Start-Sleep -Seconds $TimeoutSeconds
    rundll32.exe user32.dll,LockWorkStation
}
```

## 🛡️ Politiques de sécurité Windows

### Configuration des paramètres locaux
1. **Dans le GPO de sécurité**
2. **Naviguer vers** : Configuration ordinateur → Paramètres Windows → Paramètres de sécurité → Stratégies locales → Options de sécurité

### Paramètres clés
| Paramètre | Valeur | Description |
|-----------|--------|-------------|
| Accès réseau : Nommer anonymiquement les comptes | Désactivé | Empêche l'accès anonyme |
| Comptes : Invité du domaine | Désactivé | Désactive le compte invité |
| Accès réseau : Partages pouvant être consultés anonymement | Aucun | Pas de partage anonyme |
| Connexion utilisateur : Ne pas stocker de mot de passe | Activé | Empêche le stockage des mots de passe |

## 🔍 Politiques d'audit

### Configuration de l'audit
1. **Naviguer vers** : Stratégies d'audit → Audit de la stratégie de connexion
2. **Configurer les événements à auditer** :
   - ✅ Succès de l'audit - Connexion du compte
   - ✅ Échec de l'audit - Connexion du compte
   - ✅ Succès de l'audit - Gestion des comptes
   - ✅ Échec de l'audit - Gestion des comptes

### Types d'événements audités
- **Connexions réussies** : Suivi des accès
- **Échecs de connexion** : Détection des tentatives d'intrusion
- **Gestion des comptes** : Modifications de comptes et groupes
- **Accès aux objets** : Accès aux fichiers et dossiers sensibles

## 🚨 Politiques de contrôle d'accès

### Restrictions d'accès aux applications
1. **Créer un GPO** : "REV-Application-Restrictions"
2. **Naviguer vers** : Configuration utilisateur → Stratégies → Paramètres Windows → Sécurité → Stratégies de restriction logicielle

### Règles de restriction
- **Exécutables Windows** : Autorisés
- **Programmes installés** : Autorisés selon la liste blanche
- **Fichiers .exe** : Interdits sauf exceptions
- **Scripts** : Interdits sauf administrateurs

### Exceptions autorisées
- Microsoft Office
- Navigateurs web autorisés
- Outils de productivité
- Applications métier

## 📊 Monitoring et alertes

### Configuration des alertes de sécurité
1. **Surveiller les événements critiques** :
   - Verrouillage de compte
   - Connexions multiples échouées
   - Modifications de groupes sensibles
   - Accès aux ressources critiques

### Script de monitoring
```powershell
# Script de surveillance des événements de sécurité
$events = Get-WinEvent -LogName Security -MaxEvents 100 | Where-Object {
    $_.Id -eq 4625 -or $_.Id -eq 4740
}

foreach ($event in $events) {
    if ($event.Id -eq 4625) {
        Write-Warning "Échec de connexion détecté : $($event.Properties[5].Value)"
    }
    if ($event.Id -eq 4740) {
        Write-Warning "Compte verrouillé : $($event.Properties[0].Value)"
    }
}
```

## 🔄 Maintenance des politiques

### Révision mensuelle
1. **Analyser les journaux d'audit**
2. **Vérifier l'efficacité des politiques**
3. **Ajuster les paramètres si nécessaire**
4. **Mettre à jour les listes blanches**

### Tests réguliers
1. **Test de mot de passe** : Valider la complexité
2. **Test de verrouillage** : Simuler des échecs
3. **Test d'accès** : Valider les restrictions
4. **Test d'audit** : Vérifier la journalisation

## 🚨 Dépannage

### Problèmes courants
1. **Utilisateurs fréquemment verrouillés**
   - Vérifier la politique de verrouillage
   - Former les utilisateurs aux mots de passe
   - Considérer l'augmentation du seuil

2. **Applications bloquées**
   - Ajouter à la liste blanche
   - Vérifier les signatures numériques
   - Créer des règles spécifiques

3. **Audit trop verbeux**
   - Ajuster les niveaux d'audit
   - Filtrer les événements non critiques
   - Optimiser le stockage des journaux

### Commandes utiles
```powershell
# Vérifier les politiques appliquées
gpresult /r

# Forcer la mise à jour des politiques
gpupdate /force

# Vérifier les paramètres de mot de passe
net accounts

# Consulter les événements de sécurité
Get-WinEvent -LogName Security -MaxEvents 20 | Format-Table TimeCreated, Id, Message
```

## 📸 Captures d'écran

### Configuration de la politique de mot de passe
![Password Policy](../screenshots/03-group-policy/password-policy.png)

### Paramètres de verrouillage de compte
![Account Lockout](../screenshots/03-group-policy/account-lockout.png)

### Configuration de l'audit
![Audit Configuration](../screenshots/03-group-policy/audit-configuration.png)

---

**Document** : Politiques de sécurité  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
