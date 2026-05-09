# Vue d'ensemble de l'architecture

## 📋 Description

Ce document présente l'architecture générale du laboratoire REV-Enterprise-Lab, conçue pour simuler un environnement d'entreprise réel avec des services Windows Server essentiels.

## 🏗️ Structure de l'infrastructure

### Composants principaux
- **Contrôleur de domaine (PDC)** : 192.168.1.10
- **Serveur de fichiers** : 192.168.1.15
- **Poste client Windows 10** : 192.168.1.200
- **Passerelle réseau** : 192.168.1.1

### Services déployés
- Active Directory Domain Services
- DHCP Server
- DNS Server
- File Server
- Group Policy Management

## 🌐 Topologie réseau

```
Internet
    |
    ├── Routeur (192.168.1.1)
    │
    ├── PDC.rev.local (192.168.1.10)
    │   ├── Active Directory
    │   ├── DHCP Server
    │   └── DNS Server
    │
    ├── File-Server.rev.local (192.168.1.15)
    │   └── Partages réseau
    │
    └── HRPC01.rev.local (192.168.1.200)
        └── Windows 10 Enterprise
```

## 👥 Structure organisationnelle

### Départements
- **HR** : Ressources Humaines
- **HK** : House Keeping
- **Sales** : Ventes
- **IT** : Technologies de l'Information

### Unités d'organisation (OU)
Chaque département a sa propre OU dans Active Directory pour une gestion centralisée des politiques et des permissions.

## 🔐 Sécurité implémentée

- Stratégies de mot de passe complexes
- Verrouillage de compte après échecs
- Restrictions d'accès par département
- Contrôle des médias amovibles

## 📊 Flux de travail typique

1. **Utilisateur** se connecte au domaine `rev.local`
2. **Authentification** via Active Directory
3. **Application** des stratégies de groupe
4. **Accès** aux ressources partagées selon le département
5. **Surveillance** via les journaux d'événements

## 🎯 Objectifs atteints

- ✅ Infrastructure de domaine fonctionnelle
- ✅ Services réseau configurés
- ✅ Sécurité renforcée
- ✅ Gestion centralisée
- ✅ Documentation complète

## 📸 Captures d'écran

### Schéma d'architecture complet
![Architecture Overview](../screenshots/01-architecture/architecture-overview.png)

### Topologie réseau détaillée
![Network Topology](../screenshots/01-architecture/network-topology.png)

### Vue des services déployés
![Services Overview](../screenshots/01-architecture/services-overview.png)

---

**Document** : Vue d'ensemble de l'architecture  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
