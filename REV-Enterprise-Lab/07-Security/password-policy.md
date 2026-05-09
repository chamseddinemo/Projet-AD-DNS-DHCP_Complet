# Politique de mot de passe

## 📋 Description

Ce document présente la politique de mot de passe implémentée dans le domaine REV.LOCAL pour renforcer la sécurité des comptes utilisateurs.

## 🎯 Objectifs

- Imposer des mots de passe complexes
- Définir une durée de vie appropriée
- Maintenir un historique des mots de passe
- Prévenir la réutilisation des anciens mots de passe

## 🔐 Configuration de la politique

### Paramètres principaux
- **Longueur minimale** : 6 caractères
- **Complexité** : Activée
- **Historique** : 3 mots de passe mémorisés
- **Durée de vie maximale** : 60 jours
- **Durée de vie minimale** : 1 jour
- **Avertissement** : 14 jours avant expiration

### Exigences de complexité
- **Majuscules** : Au moins 1 caractère
- **Minuscules** : Au moins 1 caractère
- **Chiffres** : Au moins 1 chiffre
- **Caractères spéciaux** : Au moins 1 caractère (!@#$%^&*)

## 📝 Procédure de configuration

### Via l'interface graphique
1. **Ouvrir "Gestion des stratégies de groupe"**
2. **Créer un nouveau GPO** : "REV-Password-Policy"
3. **Éditer le GPO**
4. **Naviguer vers** : Configuration ordinateur → Stratégies → Paramètres Windows → Stratégies de compte → Stratégie de mot de passe

5. **Configurer les paramètres** :
   - Longueur minimale du mot de passe : 6
   - Complexité des mots de passe : Activé
   - Mémoriser les 3 derniers mots de passe
   - Durée de vie maximale : 60 jours
   - Durée de vie minimale : 1 jour

### Via PowerShell
```powershell
# Configurer la politique de mot de passe
Set-ADDefaultDomainPasswordPolicy -Identity rev.local `
    -ComplexityEnabled $true `
    -MinPasswordLength 6 `
    -MaxPasswordAge (New-TimeSpan -Days 60) `
    -MinPasswordAge (New-TimeSpan -Days 1) `
    -PasswordHistoryCount 3
```

## 🔍 Validation de la politique

### Test de complexité
1. **Tenter de créer un mot de passe simple** :
   - `password` → Rejeté
   - `Password123` → Accepté
   - `Password!` → Accepté

2. **Tester la longueur minimale** :
   - `Pass1!` → Accepté (6 caractères)
   - `Pass1` → Rejeté (5 caractères)

### Test de l'historique
1. **Changer le mot de passe** : `Password1!`
2. **Tenter de réutiliser** : `Password1!` → Rejeté
3. **Utiliser un nouveau mot de passe** : `Password2!` → Accepté

## 📊 Monitoring de l'expiration

### Rapport d'expiration des mots de passe
```powershell
# Script pour trouver les comptes avec mot de passe expirant bientôt
$days = 14
$expiryDate = (Get-Date).AddDays($days)

Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $false} -Properties PasswordLastSet, DisplayName |
    Where-Object { $_.PasswordLastSet -and ($_.PasswordLastSet.AddDays(60) -lt $expiryDate) } |
    Select-Object DisplayName, @{Name="PasswordExpires";Expression={$_.PasswordLastSet.AddDays(60)}} |
    Sort-Object PasswordExpires
```

### Notifications automatiques
```powershell
# Script pour envoyer des notifications d'expiration
$users = Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $false} -Properties EmailAddress, PasswordLastSet

foreach ($user in $users) {
    $expiryDate = $user.PasswordLastSet.AddDays(60)
    $daysUntilExpiry = ($expiryDate - (Get-Date)).Days
    
    if ($daysUntilExpiry -le 14 -and $daysUntilExpiry -gt 0) {
        $subject = "Votre mot de passe expire dans $daysUntilExpiry jours"
        $body = "Cher utilisateur, votre mot de passe expire le $expiryDate. Veuillez le mettre à jour."
        
        # Envoyer l'email (configuration requise)
        # Send-MailMessage -To $user.EmailAddress -Subject $subject -Body $body
    }
}
```

## 🚨 Dépannage

### Problèmes courants
1. **Utilisateurs ne peuvent pas changer de mot de passe**
   - Vérifier la politique de durée de vie minimale
   - Confirmer que l'utilisateur n'est pas forcé de changer
   - Valider les permissions du compte

2. **Complexité non appliquée**
   - Vérifier que la complexité est activée
   - Confirmer l'application du GPO
   - Forcer la mise à jour : `gpupdate /force`

3. **Historique non respecté**
   - Vérifier le nombre de mots de passe mémorisés
   - Confirmer que le GPO est appliqué
   - Tester avec différents utilisateurs

### Commandes utiles
```powershell
# Vérifier la politique de mot de passe actuelle
Get-ADDefaultDomainPasswordPolicy

# Vérifier la date de dernière modification du mot de passe
Get-ADUser -Identity john.smith -Properties PasswordLastSet

# Forcer un utilisateur à changer son mot de passe
Set-ADUser -Identity john.smith -ChangePasswordAtLogon $true

# Réinitialiser un mot de passe
Set-ADAccountPassword -Identity john.smith -Reset -NewPassword (ConvertTo-SecureString "NewPassword123!" -AsPlainText -Force)
```

## 📈 Bonnes pratiques

### Recommandations aux utilisateurs
- **Utiliser des phrases de passe** : Plus faciles à mémoriser
- **Éviter les informations personnelles** : Dates, noms, etc.
- **Utiliser des mots de passe uniques** : Par service/application
- **Ne jamais partager les mots de passe**

### Exemples de mots de passe sécurisés
- `MonEntreprise2026!` ✅
- `Password123!` ✅
- `Rev-Secure-2026` ✅
- `password` ❌
- `123456` ❌
- `revlocal` ❌

## 📋 Checklist de validation

### Configuration de la politique
- [ ] Longueur minimale configurée à 6 caractères
- [ ] Complexité activée
- [ ] Historique de 3 mots de passe
- [ ] Durée de vie maximale de 60 jours
- [ ] Durée de vie minimale de 1 jour
- [ ] Avertissement 14 jours avant expiration

### Tests de validation
- [ ] Mot de passe simple rejeté
- [ ] Mot de passe complexe accepté
- [ ] Longueur minimale respectée
- [ ] Historique fonctionnel
- [ ] Expiration appliquée

### Monitoring
- [ ] Rapport d'expiration généré
- [ ] Notifications configurées
- [ ] Alertes de sécurité activées

## 📸 Captures d'écran

### Configuration de la politique de mot de passe
![Password Policy Configuration](../screenshots/07-security/password-policy-config.png)

### Test de complexité
![Password Complexity Test](../screenshots/07-security/password-complexity-test.png)

### Rapport d'expiration
![Password Expiration Report](../screenshots/07-security/password-expiration-report.png)

---

**Document** : Politique de mot de passe  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
