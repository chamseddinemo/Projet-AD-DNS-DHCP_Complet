# Conception réseau

## 📋 Description

Ce document détaille la conception réseau du laboratoire REV-Enterprise-Lab, optimisée pour les besoins d'une entreprise de taille moyenne.

## 🌐 Configuration réseau

### Segment principal
- **Réseau** : 192.168.1.0/24
- **Passerelle** : 192.168.1.1
- **DNS primaire** : 192.168.1.10 (PDC)
- **DNS secondaire** : 8.8.8.8 (Google DNS)

### Sous-réseaux prévus
- **Serveurs** : 192.168.1.10-19
- **Imprimantes** : 192.168.1.20-29
- **Clients fixes** : 192.168.1.100-199
- **Clients DHCP** : 192.168.1.40-230

## 📊 Plan d'adressage IP

| Type | Plage d'adresses | Usage |
|------|------------------|-------|
| Serveurs | 192.168.1.10-19 | Infrastructure critique |
| Imprimantes | 192.168.1.20-29 | Périphériques d'impression |
| Réservés | 192.168.1.30-39 | Équipements réseau |
| DHCP | 192.168.1.40-230 | Postes clients |
| Exclusions | 192.168.1.80-85 | Réservé pour usage futur |

## 🔧 Équipements réseau

### Routeur principal
- **Modèle** : Cisco ISR 4321
- **Rôle** : Routage, NAT, Firewall de base
- **IP** : 192.168.1.1

### Switch principal
- **Modèle** : Cisco Catalyst 2960-X
- **Ports** : 48 ports Gigabit
- **VLANs** : VLAN 1 (par défaut), VLAN 10 (Serveurs)

### Points d'accès WiFi
- **SSID** : REV-Corp
- **Sécurité** : WPA2-Enterprise
- **Authentification** : RADIUS via Active Directory

## 🚀 Optimisations réseau

### QoS (Quality of Service)
- **VoIP** : Priorité haute
- **Applications critiques** : Priorité moyenne
- **Navigation web** : Priorité normale
- **Téléchargements** : Priorité basse

### Load Balancing
- **DNS round-robin** pour www.rev.local
- **Répartition de charge** entre serveurs web

## 🔒 Sécurité réseau

### Segmentation
- **VLAN 10** : Serveurs (accès restreint)
- **VLAN 20** : Clients (accès normal)
- **VLAN 30** : Invités (accès Internet seulement)

### Filtrage
- **Firewall** : Bloque ports non essentiels
- **ACL** : Contrôle d'accès inter-VLAN
- **IDS/IPS** : Détection d'intrusions

## 📈 Surveillance réseau

### Outils utilisés
- **Wireshark** : Analyse de paquets
- **Nagios** : Surveillance de la disponibilité
- **PRTG** : Monitoring des performances

### Alertes configurées
- **Perte de connectivité** : Alertes immédiates
- **Utilisation élevée** : Alertes à 80%
- **Équipements hors ligne** : Alertes après 5 minutes

## 🔄 Haute disponibilité

### Redondance
- **Double routeur** : Configuration HSRP
- **Liens multiples** : Fibre + Cuivre
- **Alimentation** : UPS + Générateur

### Basculement automatique
- **Temps de basculement** : < 30 secondes
- **Tests mensuels** : Validation du plan de continuité

## 📸 Captures d'écran

### Schéma réseau complet
![Network Diagram](../screenshots/01-architecture/network-diagram.png)

### Configuration VLAN
![VLAN Configuration](../screenshots/01-architecture/vlan-config.png)

### Monitoring réseau
![Network Monitoring](../screenshots/01-architecture/network-monitoring.png)

---

**Document** : Conception réseau  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
