# Structure des unités d'organisation

## 📋 Description

Ce document présente la structure des unités d'organisation (OU) pour le domaine REV.LOCAL, conçue pour une gestion efficace des ressources par département.

## 🏗️ Structure principale

### Hiérarchie des OU
```
rev.local
├── _ADMIN
│   ├── Service Accounts
│   ├── Groups
│   └── Computers
├── DEPARTMENTS
│   ├── HR
│   │   ├── Users
│   │   ├── Computers
│   │   └── Resources
│   ├── HK
│   │   ├── Users
│   │   ├── Computers
│   │   └── Resources
│   ├── Sales
│   │   ├── Users
│   │   ├── Computers
│   │   └── Resources
│   └── IT
│       ├── Users
│       ├── Computers
│       └── Resources
├── SERVICE_ACCOUNTS
├── GROUPS
└── COMPUTERS
    ├── Servers
    ├── Workstations
    └── Laptops
```

## 📋 Description des OU principales

### _ADMIN
- **Usage** : Comptes et groupes administratifs
- **Permissions** : Accès restreint aux administrateurs
- **Contenu** : Comptes de service, groupes de sécurité, ordinateurs administratifs

### DEPARTMENTS
- **Usage** : Organisation par département fonctionnel
- **Structure** : Chaque département a ses propres OU Users, Computers, Resources
- **Gestion** : Déléguée aux responsables de département

### SERVICE_ACCOUNTS
- **Usage** : Comptes de service pour applications
- **Politiques** : Mot de passe complexe, expiration désactivée
- **Sécurité** : Accès limité aux services nécessaires

### GROUPS
- **Usage** : Groupes de sécurité et de distribution
- **Organisation** : Par type et par département
- **Maintenance** : Mise à jour régulière

### COMPUTERS
- **Usage** : Classification des ordinateurs par type
- **Gestion** : Politiques spécifiques par type d'équipement
- **Maintenance** : Cycle de vie des équipements

## 👥 Structure par département

### HR (Ressources Humaines)
```
DEPARTMENTS/HR/
├── Users/
│   ├── john.smith
│   ├── jane.doe
│   └── mike.wilson
├── Computers/
│   ├── HR-PC01
│   ├── HR-PC02
│   └── HR-Laptop01
└── Resources/
    ├── HR-Printer
    └── HR-Scanner
```

### HK (House Keeping)
```
DEPARTMENTS/HK/
├── Users/
│   ├── robert.brown
│   ├── lisa.davis
│   └── tom.miller
├── Computers/
│   ├── HK-PC01
│   ├── HK-PC02
│   └── HK-Tablet01
└── Resources/
    ├── HK-Cleaner01
    └── HK-Cleaner02
```

### Sales (Ventes)
```
DEPARTMENTS/Sales/
├── Users/
│   ├── david.jones
│   ├── sarah.wilson
│   └── chris.taylor
├── Computers/
│   ├── Sales-PC01
│   ├── Sales-PC02
│   └── Sales-Laptop01
└── Resources/
    ├── Sales-Printer
    └── Projector01
```

### IT (Technologies de l'Information)
```
DEPARTMENTS/IT/
├── Users/
│   ├── admin.user
│   ├── tech.support
│   └── network.admin
├── Computers/
│   ├── IT-PC01
│   ├── IT-PC02
│   └── IT-Laptop01
└── Resources/
    ├── IT-Server01
    └── Network-Tools
```

## 🛠️ Procédure de création

### Création des OU principales
1. **Ouvrir "Utilisateurs et ordinateurs Active Directory"**
2. **Clic droit sur "rev.local" → Nouveau → Unité d'organisation**
3. **Nom** : `_ADMIN`
4. **Protection contre la suppression accidentelle** : Cocher
5. **Répéter** pour chaque OU principale

### Création des sous-OU
1. **Sélectionner l'OU parente**
2. **Clic droit → Nouveau → Unité d'organisation**
3. **Nom** : Nom de la sous-OU
4. **Protection** : Cocher pour les OU critiques

### Délégation des permissions
1. **Clic droit sur l'OU → Délégation de contrôle**
2. **Ajouter** le groupe ou utilisateur
3. **Sélectionner les tâches à déléguer**
4. **Définir** le niveau de permissions

## 🔐 Stratégies de groupe par OU

### _ADMIN
- **Politique** : Politique de sécurité renforcée
- **Restrictions** : Accès administratif complet
- **Audit** : Journalisation complète

### DEPARTMENTS
- **Politique** : Politique de département
- **Applications** : Logiciels spécifiques au département
- **Restrictions** : Selon les besoins du département

### SERVICE_ACCOUNTS
- **Politique** : Politique de comptes de service
- **Sécurité** : Restrictions d'ouverture de session
- **Audit** : Surveillance des connexions

## 📊 Gestion des déplacements

### Transfert entre départements
1. **Localiser** l'utilisateur dans l'OU actuelle
2. **Faire glisser** vers la nouvelle OU
3. **Valider** l'appartenance aux groupes
4. **Mettre à jour** les profils utilisateur

### Sortie d'employé
1. **Désactiver** le compte utilisateur
2. **Déplacer** vers OU "_ADMIN/Disabled Users"
3. **Archiver** les données si nécessaire
4. **Supprimer** après période de rétention

## 🚨 Bonnes pratiques

### Nomenclature
- **OU administratives** : Préfixe `_`
- **OU départementales** : Nom en anglais
- **OU fonctionnelles** : Nom descriptif

### Sécurité
- **Protection contre suppression** : Toujours cocher
- **Délégation minimale** : Donner seulement les permissions nécessaires
- **Audit régulier** : Vérifier les permissions

### Maintenance
- **Nettoyage mensuel** : Supprimer les objets orphelins
- **Révision trimestrielle** : Valider la structure
- **Documentation** : Garder les schémas à jour

## 📸 Captures d'écran

### Structure OU complète
![OU Structure](../screenshots/02-active-directory/ou-structure.png)

### Délégation de contrôle
![Delegation Control](../screenshots/02-active-directory/delegation-control.png)

### Propriétés d'OU
![OU Properties](../screenshots/02-active-directory/ou-properties.png)

---

**Document** : Structure des unités d'organisation  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
