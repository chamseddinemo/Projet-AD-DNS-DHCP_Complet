# Gestion des groupes

## 📋 Description

Ce document présente la structure des groupes dans le domaine REV.LOCAL, organisée pour une gestion efficace des permissions et des accès aux ressources.

## 🎯 Objectifs

- Créer des groupes par département et par fonction
- Organiser les permissions de manière hiérarchique
- Faciliter la gestion des accès aux ressources
- Maintenir une structure claire et évolutive

## 👥 Structure des groupes

### Groups de département
| Groupe | Description | OU | Membres typiques |
|--------|-------------|----|------------------|
| HR-Group | Utilisateurs du département RH | GROUPS | john.smith, jane.doe, mike.wilson |
| HK-Group | Utilisateurs du département HK | GROUPS | robert.brown, lisa.davis, tom.miller |
| Sales-Group | Utilisateurs du département Ventes | GROUPS | david.jones, sarah.wilson, chris.taylor |
| IT-Group | Utilisateurs du département IT | GROUPS | admin.user, tech.support, network.admin |

### Groups de sécurité
| Groupe | Description | Permissions |
|--------|-------------|-------------|
| Domain Admins | Administrateurs du domaine | Contrôle total |
| Enterprise Admins | Admins de l'entreprise | Contrôle total forêt |
| Local Admins | Admins locaux postes | Administration locale |
| Remote Desktop Users | Accès distant bureau | Connexion RDP |
| VPN Users | Accès VPN | Connexion VPN |

### Groups de ressources
| Groupe | Description | Ressource |
|--------|-------------|-----------|
| HR-Share-Access | Accès partage HR | \\file\HR |
| HK-Share-Access | Accès partage HK | \\file\HK |
| Sales-Share-Access | Accès partage Ventes | \\file\Sales |
| IT-Share-Access | Accès partage IT | \\file\IT |
| Print-HR | Impression HR | Imprimante HR |
| Print-Sales | Impression Ventes | Imprimante Ventes |

## 📝 Procédure de création de groupe

### Méthode graphique

1. **Ouvrir "Utilisateurs et ordinateurs Active Directory"**
2. **Naviguer vers l'OU "GROUPS"**
3. **Créer un nouveau groupe**
   - Clic droit → Nouveau → Groupe
   - Remplir le formulaire :

| Champ | Valeur | Description |
|-------|--------|-------------|
| Nom du groupe | HR-Group | Nom du groupe |
| Préfixe Windows 2000 | HR-Group | Nom compatible hérité |
| Étendue du groupe | Global | Portée du groupe |
| Type de groupe | Sécurité | Type de groupe |

4. **Configurer les membres**
   - Clic droit sur le groupe → Propriétés
   - Onglet "Membres" → Ajouter
   - Sélectionner les utilisateurs à ajouter

### Méthode PowerShell

```powershell
# Créer un groupe de département
New-ADGroup -Name "HR-Group" `
    -SamAccountName "HR-Group" `
    -GroupCategory Security `
    -GroupScope Global `
    -Path "OU=GROUPS,DC=rev,DC=local"

# Ajouter des utilisateurs au groupe
Add-ADGroupMember -Identity "HR-Group" -Members john.smith, jane.doe, mike.wilson
```

## 🔐 Configuration des permissions

### Permissions sur les partages réseau

#### Partage HR
```powershell
# Créer le groupe d'accès
New-ADGroup -Name "HR-Share-Access" `
    -SamAccountName "HR-Share-Access" `
    -GroupCategory Security `
    -GroupScope Global

# Ajouter les utilisateurs HR
Add-ADGroupMember -Identity "HR-Share-Access" -Members HR-Group
```

#### Permissions NTFS sur \\file\HR
- **HR-Share-Access** : Contrôle total
- **Administrateurs** : Contrôle total
- **Système** : Contrôle total
- **Utilisateurs du domaine** : Aucun accès

### Permissions d'impression

#### Imprimante HR
- **HR-Group** : Imprimer
- **IT-Group** : Gérer l'imprimante
- **Domain Admins** : Contrôle total

## 📊 Groupes imbriqués

### Structure hiérarchique
```
Domain Users (tous les utilisateurs)
├── HR-Group
│   ├── HR-Share-Access
│   └── Print-HR
├── HK-Group
│   ├── HK-Share-Access
│   └── Print-HK
├── Sales-Group
│   ├── Sales-Share-Access
│   └── Print-Sales
└── IT-Group
    ├── IT-Share-Access
    ├── Local Admins
    └── Remote Desktop Users
```

### Avantages de l'imbrication
- **Gestion simplifiée** : Ajouter un utilisateur au groupe de département
- **Héritage automatique** : Accès à toutes les ressources du département
- **Maintenance réduite** : Moins de groupes à gérer individuellement

## 🔄 Gestion du cycle de vie

### Ajout d'un nouvel utilisateur
1. **Créer** le compte utilisateur
2. **Ajouter** au groupe de département approprié
3. **Valider** l'accès aux ressources
4. **Documenter** les permissions

### Changement de département
1. **Retirer** du groupe de département actuel
2. **Ajouter** au nouveau groupe de département
3. **Vérifier** les accès aux ressources
4. **Mettre à jour** les profils utilisateur

### Sortie d'employé
1. **Retirer** de tous les groupes
2. **Désactiver** le compte utilisateur
3. **Archiver** les permissions si nécessaire
4. **Supprimer** après période de rétention

## 🚨 Bonnes pratiques

### Nomenclature
- **Groupes de département** : `[DEPARTEMENT]-Group`
- **Groupes de ressources** : `[RESSOURCE]-[TYPE]-Access`
- **Groupes de sécurité** : `[FONCTION]-Users`

### Sécurité
- **Principe du moindre privilège** : Donner seulement les permissions nécessaires
- **Groupes dédiés** : Un groupe par ressource ou fonction
- **Audit régulier** : Vérifier l'appartenance aux groupes

### Documentation
- **Description claire** : Utilisateur et fonction du groupe
- **Membres documentés** : Liste des membres et raison
- **Permissions tracées** : Ressources accessibles par le groupe

## 📈 Reporting et monitoring

### Rapports réguliers
```powershell
# Liste des groupes et leurs membres
Get-ADGroup -Filter * | ForEach-Object {
    $group = $_
    Get-ADGroupMember -Identity $group | Select-Object @{Name="Group";Expression={$group.Name}}, Name, SamAccountName
} | Export-Csv -Path "C:\Reports\GroupMembership.csv" -NoTypeInformation
```

### Alertes de sécurité
- **Ajout aux groupes sensibles** : Alertes immédiates
- **Modifications de permissions** : Journalisation complète
- **Groupes vides** : Nettoyage mensuel

## 📸 Captures d'écran

### Création de groupe
![New Group](../screenshots/02-active-directory/new-group.png)

### Gestion des membres
![Group Members](../screenshots/02-active-directory/group-members.png)

### Permissions de groupe
![Group Permissions](../screenshots/02-active-directory/group-permissions.png)

---

**Document** : Gestion des groupes  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
