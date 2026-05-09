# Gestion des utilisateurs

## 📋 Description

Ce document détaille la procédure de création et de gestion des utilisateurs dans le domaine REV.LOCAL selon la structure organisationnelle définie.

## 🎯 Objectifs

- Créer des utilisateurs selon les conventions de nommage
- Organiser les utilisateurs par département
- Configurer les propriétés utilisateur de base
- Appliquer les politiques de sécurité appropriées

## 👥 Conventions de nommage

### Format des noms d'utilisateur
- **Format** : `prenom.nom`
- **Exemples** : `john.smith`, `jane.doe`, `mike.wilson`
- **Domaine** : `rev.local`
- **Format complet** : `REV\john.smith`

### Mot de passe initial
- **Format** : `Temp2026!`
- **Expiration** : Changement obligatoire à la première connexion
- **Complexité** : Requise (majuscule, minuscule, chiffre, caractère spécial)

## 📝 Procédure de création d'utilisateur

### Méthode graphique

1. **Ouvrir "Utilisateurs et ordinateurs Active Directory"**
   - Démarrer → Outils d'administration

2. **Naviguer vers l'OU appropriée**
   - Exemple : `DEPARTMENTS/HR/Users`

3. **Créer un nouvel utilisateur**
   - Clic droit → Nouvel utilisateur
   - Remplir le formulaire :

| Champ | Exemple | Description |
|-------|---------|-------------|
| Prénom | John | Prénom de l'utilisateur |
| Initiale | A | Initiale du deuxième prénom (optionnel) |
| Nom | Smith | Nom de famille |
| Nom d'ouverture de session | john.smith | Format prenom.nom |
| Mot de passe | Temp2026! | Mot de passe temporaire |
| Confirmer | Temp2026! | Confirmation du mot de passe |

4. **Configurer les options**
   - ☑ L'utilisateur doit changer le mot de passe à la prochaine ouverture de session
   - ☐ L'utilisateur ne peut pas changer le mot de passe
   - ☐ Le mot de passe n'expire jamais
   - ☐ Le compte est désactivé

5. **Finaliser**
   - Cliquer sur "Suivant" puis "Terminer"

### Méthode PowerShell

```powershell
# Créer un utilisateur HR
New-ADUser -Name "John Smith" `
    -GivenName "John" `
    -Surname "Smith" `
    -SamAccountName "john.smith" `
    -UserPrincipalName "john.smith@rev.local" `
    -Path "OU=Users,OU=HR,OU=DEPARTMENTS,DC=rev,DC=local" `
    -AccountPassword (ConvertTo-SecureString "Temp2026!" -AsPlainText -Force) `
    -Enabled $true `
    -ChangePasswordAtLogon $true
```

## 👤 Utilisateurs par département

### Département HR (Ressources Humaines)

| Nom complet | Nom d'utilisateur | Poste | Email |
|-------------|-------------------|-------|-------|
| John Smith | john.smith | Directeur HR | john.smith@rev.local |
| Jane Doe | jane.doe | Responsable RH | jane.doe@rev.local |
| Mike Wilson | mike.wilson | Conseiller RH | mike.wilson@rev.local |

### Département HK (House Keeping)

| Nom complet | Nom d'utilisateur | Poste | Email |
|-------------|-------------------|-------|-------|
| Robert Brown | robert.brown | Superviseur HK | robert.brown@rev.local |
| Lisa Davis | lisa.davis | Agent d'entretien | lisa.davis@rev.local |
| Tom Miller | tom.miller | Technicien HK | tom.miller@rev.local |

### Département Sales (Ventes)

| Nom complet | Nom d'utilisateur | Poste | Email |
|-------------|-------------------|-------|-------|
| David Jones | david.jones | Directeur des ventes | david.jones@rev.local |
| Sarah Wilson | sarah.wilson | Responsable commercial | sarah.wilson@rev.local |
| Chris Taylor | chris.taylor | Vendeur | chris.taylor@rev.local |

### Département IT (Technologies)

| Nom complet | Nom d'utilisateur | Poste | Email |
|-------------|-------------------|-------|-------|
| Admin User | admin.user | Administrateur système | admin.user@rev.local |
| Tech Support | tech.support | Support technique | tech.support@rev.local |
| Network Admin | network.admin | Administrateur réseau | network.admin@rev.local |

## 🔧 Configuration des propriétés utilisateur

### Informations de base
1. **Clic droit sur l'utilisateur → Propriétés**
2. **Onglet "Général"**
   - Téléphone : +33 1 23 45 67 89
   - Email : utilisateur@rev.local
   - Page web : www.rev.local

3. **Onglet "Adresse"**
   - Rue : 123 Rue de l'Entreprise
   - Ville : Paris
   - Code postal : 75001
   - Pays : France

4. **Onglet "Téléphone"**
   - Téléphone professionnel : +33 1 23 45 67 89
   - Portable : +33 6 12 34 56 78

### Options de compte
1. **Onglet "Compte"**
   - Heures de connexion : 8h00-18h00 (lundi-vendredi)
   - Postes de connexion : Tous les postes
   - Options de mot de passe : Selon la politique du domaine

### Appartenance aux groupes
1. **Onglet "Membre de"**
   - Ajouter aux groupes de département
   - Ajouter aux groupes de sécurité appropriés
   - Ajouter aux groupes de distribution

## 🔄 Cycle de vie des utilisateurs

### Création
1. **Valider** les informations avec le manager
2. **Créer** le compte utilisateur
3. **Ajouter** aux groupes appropriés
4. **Configurer** les propriétés
5. **Communiquer** les identifiants

### Modification
1. **Changement de poste** : Mettre à jour les groupes
2. **Changement de département** : Déplacer dans l'OU appropriée
3. **Promotion** : Ajouter aux groupes de supervision

### Désactivation
1. **Désactiver** le compte utilisateur
2. **Déplacer** vers OU "Disabled Users"
3. **Archiver** les données si nécessaire
4. **Supprimer** après 90 jours

## 🚨 Dépannage courant

### Problèmes fréquents
1. **Connexion impossible**
   - Vérifier le format du nom d'utilisateur
   - Confirmer le mot de passe
   - Vérifier que le compte est activé

2. **Mot de passe expiré**
   - Forcer le changement de mot de passe
   - Vérifier la politique de mot de passe

3. **Accès refusé**
   - Vérifier l'appartenance aux groupes
   - Confirmer les permissions sur les ressources

### Commandes utiles
```powershell
# Rechercher un utilisateur
Get-ADUser -Filter {Name -like "*smith*"}

# Vérifier l'état d'un compte
Get-ADUser -Identity john.smith -Properties Enabled,LastLogonDate

# Réinitialiser un mot de passe
Set-ADAccountPassword -Identity john.smith -Reset -NewPassword (ConvertTo-SecureString "NewPassword123!" -AsPlainText -Force)

# Débloquer un compte
Unlock-ADAccount -Identity john.smith
```

## 📸 Captures d'écran

### Assistant de création d'utilisateur
![New User Wizard](../screenshots/02-active-directory/new-user-wizard.png)

### Propriétés utilisateur
![User Properties](../screenshots/02-active-directory/user-properties.png)

### Gestion des groupes
![Group Membership](../screenshots/02-active-directory/group-membership.png)

---

**Document** : Gestion des utilisateurs  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
