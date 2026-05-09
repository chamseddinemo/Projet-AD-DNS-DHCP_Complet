# REV-Enterprise-Lab

## 🏢 Description du projet

**REV-Enterprise-Lab** est un environnement de laboratoire Windows Server conçu pour simuler les opérations IT d'une entreprise réelle. Ce projet démontre l'implémentation et la gestion des services Windows Server essentiels.

## 🎯 Objectifs

- Simuler un environnement Windows Server d'entreprise
- Déployer Active Directory avec structure organisationnelle
- Implémenter des stratégies de groupe pour la sécurité
- Configurer les services réseau (DHCP, DNS)
- Établir une infrastructure de serveur de fichiers sécurisée
- Documenter tous les processus pour le support technique

## 🛠️ Technologies utilisées

| Technologie | Version | Usage |
|------------|---------|-------|
| Windows Server | 2019/2022 | Serveur principal |
| Windows 10 | Entreprise | Poste client |
| Active Directory | Services de domaine | Gestion des identités |
| Stratégies de groupe | Console de gestion | Application des politiques |
| DHCP | Rôle | Gestion des adresses IP |
| DNS | Rôle | Résolution de noms |
| Serveur de fichiers | Rôle | Stockage centralisé |

## 🏗️ Détails de l'environnement

### Configuration réseau
- **Nom de domaine** : `rev.local`
- **Contrôleur de domaine principal** : `PDC.rev.local`
- **Adresse IP du serveur** : `192.108.1.10`
- **Poste client** : `HRPC01.rev.local`
- **IP client réservée** : `192.168.1.200`

### Plan d'adressage IP
| Composant | Adresse IP | Masque | Passerelle |
|-----------|-------------|--------|------------|
| Serveur PDC | 192.108.1.10 | 255.255.255.0 | 192.108.1.1 |
| Client HRPC01 | 192.168.1.200 | 255.255.255.0 | 192.168.1.1 |
| Plage DHCP | 192.168.1.40-230 | 255.255.255.0 | 192.168.1.1 |

## 📋 Vue d'ensemble de l'architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    REV.ENTERPRISE.LAB                       │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   PDC       │  │  File Server│  │   Client    │         │
│  │192.168.1.10 │  │192.168.1.15 │  │192.168.1.200│         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
├─────────────────────────────────────────────────────────────┤
│                    Services principaux                      │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │Active Dir   │ │   DHCP      │ │    DNS      │            │
│  │Domain Svc   │ │   Server    │ │   Server    │            │
│  └─────────────┘ └─────────────┘ └─────────────┘            │
├─────────────────────────────────────────────────────────────┤
│               Unités d'organisation                         │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐            │
│  │   HR    │ │    HK   │ │  Sales  │ │    IT   │            │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘            │
└─────────────────────────────────────────────────────────────┘
```

## ✨ Fonctionnalités implémentées

### 🔐 Gestion des identités et accès
- Services de domaine Active Directory
- Structure des unités d'organisation par département
- Gestion des utilisateurs et groupes avec conventions de nommage
- Gestion des comptes ordinateurs pour les appareils du domaine

### 📋 Gestion des stratégies
- Objets de stratégie de groupe pour la sécurité
- Restrictions utilisateur (Invite de commandes, Panneau de configuration)
- Contrôles d'accès au stockage amovible
- Stratégies de restriction logicielle
- Stratégies de mot de passe et verrouillage de compte

### 🌐 Services réseau
- Serveur DHCP avec configuration d'étendue et réservations
- Serveur DNS avec enregistrements A et équilibrage de charge round-robin
- Conception de topologie réseau et planification IP

### 💾 Services de fichiers
- Création de dossiers partagés avec autorisations NTFS
- Mappage de lecteurs via stratégie de groupe
- Gestion des quotas de disque
- Contrôles d'accès spécifiques par département

### 🔒 Sécurité renforcée
- Exigences de complexité des mots de passe
- Stratégies de verrouillage de compte
- Restrictions de sécurité des points de terminaison
- Contrôles des médias amovibles

### 🖥️ Gestion des clients
- Procédures de jointure de domaine Windows 10
- Délégation d'administrateur local
- Validation des stratégies de groupe
- Déploiement d'applications via GPO

## 📸 Captures d'écran

### Vue d'ensemble de l'architecture
![Architecture](screenshots/01-architecture/architecture-overview.png)

### Structure Active Directory
![AD Structure](screenshots/02-active-directory/ad-structure.png)

### Gestion des stratégies de groupe
![GPO Management](screenshots/03-group-policy/gpo-management.png)

### Configuration des services réseau
![Network Services](screenshots/04-network-services/network-services.png)

### Configuration du serveur de fichiers
![File Server](screenshots/05-file-server/file-server.png)

### Configuration client
![Client Config](screenshots/06-client-configuration/client-config.png)

### Stratégies de sécurité
![Security Policies](screenshots/07-security/security-policies.png)

## 🎓 Compétences démontrées

| Catégorie | Compétences |
|----------|------------|
| **Administration serveur** | Déploiement Windows Server, Configuration des rôles |
| **Active Directory** | Configuration contrôleur de domaine, Design OU |
| **Stratégies de groupe** | Création GPO, Liaison de politiques |
| **Services réseau** | Configuration DHCP/DNS, Planification IP |
| **Services de fichiers** | Autorisations de partage, Droits NTFS |
| **Sécurité** | Application des politiques, Contrôles d'accès |
| **Gestion client** | Jointures domaine, Validation politiques |
| **Documentation** | Rédaction technique, Procédures |

## 🚀 Améliorations futures

### Améliorations prévues
- **Haute disponibilité** : Implémenter des contrôleurs de domaine supplémentaires
- **Services de certificats** : Déployer AD CS pour la gestion des certificats
- **Accès distant** : Configurer des solutions VPN et DirectAccess
- **Surveillance** : Implémenter System Center Operations Manager
- **Solutions de sauvegarde** : Déployer des stratégies Windows Server Backup

## ✅ Résumé de la validation

Le projet inclut des procédures de validation complètes :

| Composant | Méthode de validation | Statut |
|-----------|----------------------|---------|
| Active Directory | Connexion utilisateur, Appartenance groupe | ✅ Validé |
| Stratégies de groupe | gpupdate, Test politiques | ✅ Validé |
| DHCP | Attribution IP, Test réservation | ✅ Validé |
| DNS | Résolution de noms, Équilibrage charge | ✅ Validé |
| Serveur de fichiers | Test accès, Vérification permissions | ✅ Validé |
| Sécurité | Test politique mot de passe, Verrouillage | ✅ Validé |

## 📚 Structure de la documentation

```
REV-Enterprise-Lab/
├── 01-Architecture/          # Conception réseau et planification
├── 02-Active-Directory/      # Implémentation AD DS
├── 03-Group-Policy/         # Configuration GPO
├── 04-Network-Services/     # Configuration DHCP/DNS
├── 05-File-Server/          # Implémentation services fichiers
├── 06-Client-Configuration/ # Gestion postes de travail
├── 07-Security/             # Stratégies de sécurité
├── 08-Applications/         # Déploiement d'applications
├── 09-Validation/           # Tests et dépannage
├── docs/                    # Documentation du projet
└── screenshots/             # Documentation visuelle
```

## 🤝 Contribuer

Ce projet sert de ressource d'apprentissage complète pour l'administration Windows Server. N'hésitez pas à l'utiliser comme référence pour vos propres projets d'infrastructure d'entreprise.

---

**Statut du projet** : ✅ Complet  
**Dernière mise à jour** : Mai 2026  
**Version** : 1.0.0
