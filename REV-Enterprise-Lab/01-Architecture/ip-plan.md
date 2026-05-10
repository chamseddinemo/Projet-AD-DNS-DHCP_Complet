# Plan d'adressage IP

## 📋 Description

Ce document présente le plan d'adressage IP complet pour l'infrastructure REV-Enterprise-Lab, assurant une gestion optimale des ressources réseau.

## 🏢 Réseau principal

### Informations générales
- **Réseau** : 192.168.1.0/24
- **Masque de sous-réseau** : 255.255.255.0
- **Passerelle** : 192.168.1.1
- **Domaine DNS** : rev.local

### Serveurs DNS
- **Primaire** : 192.168.1.10 (PDC.rev.local)
- **Secondaire** : 8.8.8.8 (Google DNS)

### Équipements réseau (192.168.1.20-29)
| IP | Équipement | Type | Statut |
|----|------------|------|--------|
| 192.168.1.1 | Router-Gateway | Routeur | ✅ Actif |
| 192.168.1.2 | Switch-Core | Switch principal | ✅ Actif |
| 192.168.1.20 | Printer-HR | Imprimante HR | ✅ Actif |
| 192.168.1.21 | Printer-Sales | Imprimante Ventes | ✅ Actif |
| 192.168.1.22 | Printer-IT | Imprimante IT | ✅ Actif |
| 192.168.1.23 | AP-Corp-1 | Point d'accès WiFi | ✅ Actif |
| 192.168.1.24 | AP-Corp-2 | Point d'accès WiFi | ✅ Actif |

### Réservations DHCP (192.168.1.30-39)
| IP | Nom d'hôte | Utilisateur | Département |
|----|------------|-------------|-------------|
| 192.168.1.30 | Director-PC | Directeur | Direction |
| 192.168.1.31 | IT-Manager-PC | Manager IT | IT |
| 192.168.1.32 | HR-Manager-PC | Manager HR | HR |
| 192.168.1.33 | Sales-Manager-PC | Manager Ventes | Sales |

### Plage DHCP principale (192.168.1.40-230)
- **Début** : 192.168.1.40
- **Fin** : 192.168.1.230
- **Masque** : 255.255.255.0
- **Passerelle** : 192.168.1.1
- **DNS** : 192.168.1.10, 8.8.8.8
- **Durée de bail** : 8 jours

### Exclusions DHCP (192.168.1.80-85)
- **Plage exclue** : 192.168.1.80-85
- **Raison** : Réservée pour usage futur (serveurs spécialisés)

### Réservations spécifiques
| IP | Nom d'hôte | MAC | Utilisation |
|----|------------|-----|-------------|
| 192.168.1.200 | HRPC01.rev.local | 00:1A:2B:3C:4D:5E | Poste client HR |

## 🌐 Configuration DNS

### Enregistrements A principaux
```
rev.local.                IN  A   192.168.1.10
www.rev.local.             IN  A   192.168.1.8
www.rev.local.             IN  A   192.168.1.9
pdc.rev.local.             IN  A   192.168.1.10
file.rev.local.            IN  A   192.168.1.15
```

### Enregistrements CNAME
```
mail.rev.local.            IN  CNAME   pdc.rev.local.
intranet.rev.local.       IN  CNAME   www.rev.local.
```

### Round Robin Load Balancing
```
www.rev.local.             IN  A   192.168.1.8
www.rev.local.             IN  A   192.168.1.9
```

## 🔐 Sécurité IP

### Filtrage par adresse
- **Accès admin** : 192.168.1.10-15 uniquement
- **Accès client** : 192.168.1.40-230
- **Accès invité** : 192.168.1.231-254 (limité)

### Restrictions réseau
- **Ports bloqués** : 135-139, 445 (sauf serveurs)
- **Protocoles autorisés** : HTTP, HTTPS, DNS, DHCP, RDP

## 📈 Gestion et monitoring

### Surveillance des adresses
- **Utilisation actuelle** : 35%
- **Adresses disponibles** : 165
- **Seuil d'alerte** : 80%

### Rapports automatiques
- **Utilisation DHCP** : Quotidien
- **Conflits IP** : Immédiat
- **Adresses expirées** : Hebdomadaire

## 🔄 Plan de croissance

### Extension prévue
- **Nouveau sous-réseau** : 192.168.2.0/24
- **Date prévue** : Q4 2026
- **Raison** : Croissance de l'entreprise

### Migration progressive
- **Phase 1** : Serveurs critiques
- **Phase 2** : Départements spécifiques
- **Phase 3** : Généralisation

## 📸 Captures d'écran

### Tableau d'adressage complet
![IP Address Table](../screenshots/01-architecture/ip-address-table.png)

### Configuration DHCP
![DHCP Configuration](../screenshots/01-architecture/dhcp-config.png)

### Console DNS
![DNS Console](../screenshots/01-architecture/dns-console.png)

---

**Document** : Plan d'adressage IP  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
