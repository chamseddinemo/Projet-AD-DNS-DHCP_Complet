# Gestion des comptes ordinateurs

## 📋 Description

Ce document explique la procédure de création et de gestion des comptes ordinateurs dans le domaine REV.LOCAL pour les postes de travail et serveurs.

## 🎯 Objectifs

- Créer des comptes ordinateurs selon les conventions
- Organiser les ordinateurs par type et par département
- Configurer les jointures au domaine
- Appliquer les politiques appropriées

## 💻 Conventions de nommage

### Postes de travail
- **Format** : `[DEPARTEMENT]-PC[NN]`
- **Exemples** : `HR-PC01`, `SALES-PC02`, `IT-PC01`
- **Lettres département** : HR, HK, SALES, IT

### Ordinateurs portables
- **Format** : `[DEPARTEMENT]-LAPTOP[NN]`
- **Exemples** : `HR-LAPTOP01`, `SALES-LAPTOP01`

### Serveurs
- **Format** : `[FONCTION]-[TYPE][NN]`
- **Exemples** : `FILE-SRV01`, `WEB-SRV01`, `DB-SRV01`

### Équipements spéciaux
- **Imprimantes** : `PRINTER-[LOCATION][NN]`
- **Tablettes** : `[DEPARTEMENT]-TABLET[NN]`

## 📝 Procédure de jointure au domaine

### Méthode graphique (Windows 10)

1. **Ouvrir les paramètres système**
   - Démarrer → Paramètres → Système
   - À propos de → Modifier les paramètres

2. **Modifier le nom et l'appartenance au domaine**
   - Cliquer sur "Modifier les paramètres"
   - Changer le nom de l'ordinateur si nécessaire
   - Cliquer sur "Modifier"

3. **Joindre le domaine**
   - Sélectionner "Domaine"
   - Entrer : `rev.local`
   - Cliquer sur "OK"

4. **Authentification**
   - Nom d'utilisateur : `REV\Administrateur`
   - Mot de passe : [mot de passe administrateur]
   - Cliquer sur "OK"

5. **Redémarrage**
   - Confirmer avec "OK"
   - Redémarrer l'ordinateur

### Méthode PowerShell

```powershell
# Joindre un ordinateur au domaine
Add-Computer -DomainName rev.local -Credential REV\Administrateur -Restart

# Renommer et joindre en une seule commande
Add-Computer -NewName "HR-PC01" -DomainName rev.local -Credential REV\Administrateur -Restart
```

## 🏢 Organisation des comptes ordinateurs

### Structure des OU
```
COMPUTERS/
├── Servers/
│   ├── FILE-SRV01
│   ├── WEB-SRV01
│   └── DB-SRV01
├── Workstations/
│   ├── HR/
│   │   ├── HR-PC01
│   │   └── HR-PC02
│   ├── HK/
│   │   ├── HK-PC01
│   │   └── HK-PC02
│   ├── Sales/
│   │   ├── Sales-PC01
│   │   └── Sales-PC02
│   └── IT/
│       ├── IT-PC01
│       └── IT-PC02
└── Laptops/
    ├── HR-LAPTOP01
    ├── Sales-LAPTOP01
    └── IT-LAPTOP01
```

### Ordinateurs par département

#### HR (Ressources Humaines)
| Nom | Type | Adresse IP | Utilisateur principal |
|-----|------|------------|---------------------|
| HR-PC01 | Poste de travail | 192.168.1.101 | john.smith |
| HR-PC02 | Poste de travail | 192.168.1.102 | jane.doe |
| HR-LAPTOP01 | Ordinateur portable | DHCP/DHCP | mike.wilson |

#### HK (House Keeping)
| Nom | Type | Adresse IP | Utilisateur principal |
|-----|------|------------|---------------------|
| HK-PC01 | Poste de travail | 192.168.1.111 | robert.brown |
| HK-PC02 | Poste de travail | 192.168.1.112 | lisa.davis |
| HK-TABLET01 | Tablette | DHCP | tom.miller |

#### Sales (Ventes)
| Nom | Type | Adresse IP | Utilisateur principal |
|-----|------|------------|---------------------|
| Sales-PC01 | Poste de travail | 192.168.1.121 | david.jones |
| Sales-PC02 | Poste de travail | 192.168.1.122 | sarah.wilson |
| Sales-LAPTOP01 | Ordinateur portable | 192.168.1.200 | chris.taylor |

#### IT (Technologies)
| Nom | Type | Adresse IP | Utilisateur principal |
|-----|------|------------|---------------------|
| IT-PC01 | Poste de travail | 192.168.1.131 | admin.user |
| IT-PC02 | Poste de travail | 192.168.1.132 | tech.support |
| IT-LAPTOP01 | Ordinateur portable | DHCP | network.admin |

## 🔧 Configuration post-jointure

### Configuration de base
1. **Vérifier la jointure**
   ```powershell
   $env:COMPUTERNAME
   (Get-WmiObject -Class Win32_ComputerSystem).Domain
   ```

2. **Installer les logiciels de base**
   - Antivirus entreprise
   - Agent de monitoring
   - Logiciels de productivité

3. **Appliquer les stratégies de groupe**
   ```powershell
   gpupdate /force
   ```

### Configuration réseau
1. **Vérifier la configuration IP**
   ```powershell
   ipconfig /all
   ```

2. **Configurer le DNS**
   - Primaire : 192.168.1.10
   - Secondaire : 8.8.8.8

3. **Tester la connectivité**
   ```powershell
   ping pdc.rev.local
   nslookup rev.local
   ```

## 🔐 Sécurité des comptes ordinateurs

### Permissions par défaut
- **Domain Computers** : Appartenance automatique
- **Authenticated Users** : Permissions de base
- **Local System** : Permissions locales complètes

### Restrictions de sécurité
- **Jointures au domaine** : Seuls les administrateurs
- **Désactivation automatique** : Après 90 jours d'inactivité
- **Audit des connexions** : Journalisation activée

### Gestion des mots de passe
- **Mot de passe du compte** : Généré automatiquement
- **Rotation** : Tous les 30 jours
- **Complexité** : 128 caractères aléatoires

## 📊 Gestion du cycle de vie

### Création
1. **Préparer** l'ordinateur (formatage, installation)
2. **Joindre** au domaine
3. **Déplacer** dans l'OU appropriée
4. **Installer** les logiciels requis
5. **Documenter** les informations

### Maintenance
1. **Mises à jour** : Windows et logiciels
2. **Antivirus** : Mises à jour et scans
3. **Sauvegarde** : Données utilisateur
4. **Monitoring** : Performance et sécurité

### Retrait de service
1. **Désactiver** le compte ordinateur
2. **Retirer** du domaine
3. **Archiver** les données
4. **Supprimer** après 30 jours

## 🚨 Dépannage courant

### Problèmes fréquents
1. **Échec de jointure au domaine**
   - Vérifier la connectivité réseau
   - Confirmer la résolution DNS
   - Valider les permissions administratives

2. **Stratégies de groupe non appliquées**
   - Forcer la mise à jour : `gpupdate /force`
   - Vérifier l'appartenance à l'OU
   - Valider les liens GPO

3. **Connexion impossible**
   - Vérifier le mot de passe du compte ordinateur
   - Confirmer l'heure du système
   - Valider la synchronisation Kerberos

### Commandes utiles
```powershell
# Vérifier l'état du domaine
Test-ComputerSecureChannel

# Réparer le canal sécurisé
Test-ComputerSecureChannel -Repair

# Réinitialiser le compte ordinateur
Reset-ComputerMachinePassword

# Vérifier les stratégies appliquées
gpresult /r
```

## 📸 Captures d'écran

### Jointure au domaine
![Domain Join](../screenshots/02-active-directory/domain-join.png)

### Organisation des OU
 Computer OU Structure](../screenshots/02-active-directory/computer-ou-structure.png)

### Propriétés de l'ordinateur
![Computer Properties](../screenshots/02-active-directory/computer-properties.png)

---

**Document** : Gestion des comptes ordinateurs  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
