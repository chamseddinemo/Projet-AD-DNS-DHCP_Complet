# REV-Enterprise-Lab

## Description du projet

REV-Enterprise-Lab est un environnement de laboratoire Windows Server conçu pour simuler les opérations IT d'une entreprise réelle. Ce projet démontre l'implémentation et la gestion des services Windows Server essentiels.

---

## Objectifs

- Simuler un environnement Windows Server d'entreprise
- Déployer Active Directory avec structure organisationnelle
- Implémenter des stratégies de groupe pour la sécurité
- Configurer les services réseau (DHCP, DNS)
- Établir une infrastructure de serveur de fichiers sécurisée
- Documenter tous les processus pour le support technique

---

## Technologies utilisées

| Technologie | Version | Usage |
|------------|---------|-------|
| Windows Server | 2019/2022 | Serveur principal |
| Windows 10 | Entreprise | Poste client |
| Active Directory | Services de domaine | Gestion des identités |
| Stratégies de groupe | Console de gestion | Application des politiques |
| DHCP | Rôle | Gestion des adresses IP |
| DNS | Rôle | Résolution de noms |
| Serveur de fichiers | Rôle | Stockage centralisé |

---

## Détails de l'environnement

### Configuration réseau
- **Nom de domaine** : `rev.local`
- **Contrôleur de domaine principal** : `PDC.rev.local`
- **Adresse IP du serveur** : `192.168.1.10`
- **Poste client** : `HRPC01.rev.local`
- **IP client réservée** : `192.168.1.200`

### Plan d'adressage IP

| Composant | Adresse IP | Masque | Passerelle |
|-----------|-------------|--------|------------|
| Serveur PDC | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| Client HRPC01 | 192.168.1.200 | 255.255.255.0 | 192.168.1.1 |
| Plage DHCP | 192.168.1.40-230 | 255.255.255.0 | 192.168.1.1 |

---

## Architecture

```
REV ENTERPRISE LAB
│
├── PDC (192.168.1.10)
├── File Server (192.168.1.15)
└── Client (192.168.1.200)

Services:
- Active Directory
- DHCP
- DNS

Departements:
HR | HK | Sales | IT
```

---

## ✨ Fonctionnalités principales

### Gestion des identités
- Active Directory Domain Services
- OU par département
- Gestion utilisateurs et groupes

### GPO (Stratégies de groupe)
- Blocage CMD et Control Panel
- Restriction stockage externe
- Stratégies de mot de passe

### Réseau
- DHCP avec scope + réservation
- DNS avec load balancing

### 💾 File Server
- Dossiers partagés
- Permissions NTFS
- Quotas disque

### Sécurité
- Complexité mot de passe
- Account lockout
- Restrictions utilisateurs

### Client
- Join domain
- Admin local via groupe IT
- GPO appliquées

---

## Captures d'écran du projet

### Configuration des stratégies de groupe (GPO)
photos:
### Gestion des groupes Active Directory
photos:
### Politique de mot de passe
photos:
### Configuration DHCP
photos:
### Mappage des lecteurs
photos:
### Gestion des quotas
photos:
---

## Compétences démontrées

| Catégorie | Compétences |
|----------|------------|
| Serveur | Windows Server, rôles et fonctionnalités |
| AD | Domain Controller, OU, Utilisateurs |
| GPO | Policies, restrictions, applications |
| Réseau | DHCP, DNS, planification IP |
| Sécurité | Politiques, accès, permissions |
| Client | Join domain, configuration |
| Documentation | Rédaction technique, procédures |

---

## ✅ Validation du projet

| Composant | Test | Status |
|-----------|------|--------|
| Active Directory | Login utilisateur | ✅ |
| GPO | gpupdate | ✅ |
| DHCP | IP assignée | ✅ |
| DNS | Résolution | ✅ |
| File Server | Accès | ✅ |
| Sécurité | Lockout | ✅ |

---

## 📚 Structure du projet

```
Projet-AD-DNS-DHCP_Complet/
├── REV-Enterprise-Lab/
│   ├── 01-Architecture/
│   ├── 02-Active-Directory/
│   ├── 03-Groupe-Policy/
│   ├── 04-Services réseau/
│   ├── 05-Serveur de fichiers/
│   ├── 06-Configuration du client/
│   ├── 07-Sécurité/
│   ├── 08-Applications/
│   ├── 09-Validation/
│   ├── Capture d'écran/
│   │   ├── DHCP/
│   │   ├── GPO/
│   │   ├── GROUPES/
│   │   ├── MOT DE PASSE/
│   │   ├── MapDrive/
│   │   ├── modification Disque + lecteur de carte/
│   │   └── quota/
│   ├── documents/
│   └── README.md
└── README.md
```
