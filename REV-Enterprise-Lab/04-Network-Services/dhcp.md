# Configuration du serveur DHCP

## 📋 Description

Ce document présente la configuration du serveur DHCP dans le domaine REV.LOCAL pour la gestion automatique des adresses IP des postes clients.

## 🎯 Objectifs

- Configurer un serveur DHCP fiable
- Définir des plages d'adresses appropriées
- Implémenter des réservations pour les équipements critiques
- Assurer la haute disponibilité du service

## 🛠️ Prérequis

### Configuration système
- **Windows Server 2019/2022** installé
- **Rôle DHCP** installé
- **Adresse IP statique** : 192.168.1.10
- **Autorisation DHCP** dans Active Directory

### Installation du rôle DHCP
1. **Gestionnaire de serveur** → Ajouter des rôles
2. **Sélectionner** : Serveur DHCP
3. **Configurer** les options de base
4. **Autoriser** le serveur dans AD

## 📊 Configuration de l'étendue

### Étendue principale
- **Nom** : REV-Corporate-Scope
- **Description** : Plage principale pour les postes clients
- **Adresse de début** : 192.168.1.40
- **Adresse de fin** : 192.168.1.230
- **Masque de sous-réseau** : 255.255.255.0
- **Passerelle** : 192.168.1.1
- **Durée du bail** : 8 jours

### Plages d'exclusion
| Début | Fin | Raison |
|-------|-----|--------|
| 192.168.1.80 | 192.168.1.85 | Réservé pour usage futur |
| 192.168.1.190 | 192.168.1.199 | Réservé pour serveurs spécialisés |

### Réservations configurées
| Adresse IP | Nom d'hôte | MAC | Utilisation |
|------------|------------|-----|------------|
| 192.168.1.200 | HRPC01.rev.local | 00:1A:2B:3C:4D:5E | Poste client HR |
| 192.168.1.201 | Director-PC.rev.local | 00:1A:2B:3C:4D:5F | PC du directeur |
| 192.168.1.202 | IT-Manager-PC.rev.local | 00:1A:2B:3C:4D:60 | PC du manager IT |

## 📝 Procédure de configuration

### Étape 1 : Création de l'étendue
1. **Ouvrir la console DHCP**
2. **Clic droit sur IPv4** → Nouvelle étendue
3. **Assistant de configuration** :
   - Nom : REV-Corporate-Scope
   - Description : Plage principale pour les postes clients
   - Plage IP : 192.168.1.40 - 192.168.1.230
   - Masque : 255.255.255.0
   - Exclusions : 192.168.1.80-85, 192.168.1.190-199

### Étape 2 : Configuration des options
1. **Durée du bail** : 8 jours
2. **Configurer les options maintenant** : Oui
3. **Routeur (passerelle)** : 192.168.1.1
4. **Serveur DNS** : 192.168.1.10
5. **Domaine DNS** : rev.local

### Étape 3 : Création des réservations
1. **Développer** l'étendue → Réservations
2. **Clic droit** → Nouvelle réservation
3. **Configurer** :
   - Adresse IP : 192.168.1.200
   - Adresse MAC : 00:1A:2B:3C:4D:5E
   - Nom : HRPC01
   - Description : Poste client HR

## 🔧 Options DHCP avancées

### Options configurées
| Code | Option | Valeur | Description |
|------|--------|--------|-------------|
| 003 | Routeur | 192.168.1.1 | Passerelle par défaut |
| 006 | Serveur DNS | 192.168.1.10 | Serveur DNS primaire |
| 015 | Nom de domaine | rev.local | Suffixe DNS |
| 044 | Serveurs WINS/NBNS | 192.168.1.10 | Résolution NetBIOS |
| 046 | Type de noeud WINS/NBT | 0x8 | Type hybride |

### Options spécifiques par classe
- **Classe utilisateur IT** : Options de débogage activées
- **Classe utilisateur HR** : Restrictions d'accès réseau

## 🚨 Sécurité DHCP

### Filtrage des adresses MAC
1. **Activer le filtrage** : Autoriser uniquement les listées
2. **Ajouter les adresses MAC autorisées** :
   - 00:1A:2B:3C:4D:5E (HRPC01)
   - 00:1A:2B:3C:4D:5F (Director-PC)
   - 00:1A:2B:3C:4D:60 (IT-Manager-PC)

### Détection des serveurs DHCP non autorisés
1. **Activer la détection** : Dans les propriétés du serveur
2. **Journalisation** : Activer les journaux détaillés
3. **Alertes** : Configurer les notifications par email

## 📊 Monitoring et maintenance

### Surveillance de l'utilisation
```powershell
# Statistiques d'utilisation de l'étendue
Get-DhcpServerv4ScopeStatistics -ComputerName PDC.rev.local

# Utilisation actuelle
Get-DhcpServerv4Scope -ComputerName PDC.rev.local | Select-Object Name, State, AddressUsed, PercentageInUse
```

### Rapports automatiques
- **Utilisation quotidienne** : Rapport à 8h00
- **Alertes de saturation** : > 80% d'utilisation
- **Bails expirés** : Nettoyage hebdomadaire

### Script de monitoring
```powershell
# Script PowerShell pour surveiller l'utilisation DHCP
$scope = Get-DhcpServerv4Scope -ComputerName PDC.rev.local
$stats = Get-DhcpServerv4ScopeStatistics -ComputerName PDC.rev.local

foreach ($stat in $stats) {
    $percentage = ($stat.AddressUsed / $stat.AddressTotal) * 100
    if ($percentage -gt 80) {
        Write-Warning "Étendue $($stat.ScopeId) à $($percentage.ToString('0.0'))% d'utilisation"
    }
}
```

## 🔧 Haute disponibilité

### Configuration du basculement
1. **Serveur secondaire** : BDC.rev.local (192.168.1.11)
2. **Mode de basculement** : Hot standby
3. **Pourcentage de partage** : 80/20
4. **Délai de basculement** : 60 secondes

### Synchronisation des bases de données
- **Réplication** : Toutes les 5 minutes
- **Validation** : Quotidienne
- **Sauvegarde** : Automatique toutes les nuits

## 🚨 Dépannage

### Problèmes courants
1. **Clients ne reçoivent pas d'adresse IP**
   - Vérifier la connectivité réseau
   - Confirmer l'autorisation DHCP
   - Valider la configuration de l'étendue

2. **Conflits d'adresses IP**
   - Activer la détection de conflits
   - Vérifier les réservations
   - Nettoyer les bails périmés

3. **Serveur DHCP non autorisé**
   - Autoriser le serveur dans AD
   - Vérifier les permissions
   - Redémarrer le service

### Commandes utiles
```powershell
# Vérifier l'état du serveur DHCP
Get-DhcpServerv4Statistics -ComputerName PDC.rev.local

# Recréer la base de données DHCP
Repair-DhcpServerv4Database -ComputerName PDC.rev.local

# Exporter la configuration
Export-DhcpServer -ComputerName PDC.rev.local -File C:\Backup\DHCPConfig.xml

# Importer la configuration
Import-DhcpServer -ComputerName BDC.rev.local -File C:\Backup\DHCPConfig.xml
```

## 📸 Captures d'écran

### Configuration de l'étendue DHCP
![DHCP Scope Configuration](../screenshots/04-network-services/dhcp-scope-config.png)

### Réservations DHCP
![DHCP Reservations](../screenshots/04-network-services/dhcp-reservations.png)

### Statistiques d'utilisation
![DHCP Statistics](../screenshots/04-network-services/dhcp-statistics.png)

---

**Document** : Configuration du serveur DHCP  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
