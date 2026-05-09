# Configuration du serveur DNS

## 📋 Description

Ce document présente la configuration du serveur DNS dans le domaine REV.LOCAL pour la résolution de noms et la gestion des enregistrements DNS.

## 🎯 Objectifs

- Configurer un serveur DNS fiable et performant
- Gérer les zones DNS principales
- Implémenter le round-robin pour la répartition de charge
- Assurer la résolution de noms interne et externe

## 🛠️ Prérequis

### Configuration système
- **Windows Server 2019/2022** installé
- **Rôle DNS** installé
- **Adresse IP statique** : 192.168.1.10
- **Intégration Active Directory** : Activée

### Installation du rôle DNS
1. **Gestionnaire de serveur** → Ajouter des rôles
2. **Sélectionner** : Serveur DNS
3. **Configurer** les options de base
4. **Créer les zones** principales

## 🌐 Configuration des zones

### Zone principale
- **Nom** : rev.local
- **Type** : Zone principale intégrée à Active Directory
- **Réplication** : À tous les contrôleurs de domaine dans cette forêt
- **Mises à jour dynamiques** : Sécurisées uniquement

### Zone de recherche inversée
- **Nom** : 1.168.192.in-addr.arpa
- **Type** : Zone principale intégrée à Active Directory
- **Réplication** : À tous les contrôleurs de domaine

### Zone de recherche directe externe
- **Nom** : rev-corp.com (si applicable)
- **Type** : Zone secondaire
- **Serveur maître** : DNS externe du fournisseur

## 📝 Enregistrements DNS configurés

### Enregistrements A principaux
| Nom | Type | Adresse IP | Description |
|-----|------|------------|-------------|
| pdc | A | 192.168.1.10 | Contrôleur de domaine principal |
| file | A | 192.168.1.15 | Serveur de fichiers |
| www | A | 192.168.1.8 | Serveur web principal |
| www | A | 192.168.1.9 | Serveur web secondaire |
| mail | A | 192.168.1.10 | Serveur de messagerie |
| vpn | A | 192.168.1.10 | Serveur VPN |

### Enregistrements CNAME
| Nom | Type | Cible | Description |
|-----|------|-------|-------------|
| intranet | CNAME | www.rev.local | Intranet de l'entreprise |
| portal | CNAME | www.rev.local | Portail d'entreprise |
| helpdesk | CNAME | www.rev.local | Portail help desk |

### Enregistrements MX
| Préférence | Serveur de messagerie | Description |
|------------|----------------------|-------------|
| 10 | mail.rev.local | Serveur de messagerie principal |

### Enregistrements SRV
| Service | Protocole | Priorité | Poids | Port | Cible |
|---------|-----------|----------|-------|-------|-------|
| _ldap | _tcp | 0 | 100 | 389 | pdc.rev.local |
| _kerberos | _tcp | 0 | 100 | 88 | pdc.rev.local |
| _gc | _tcp | 0 | 100 | 3268 | pdc.rev.local |

## 🔄 Configuration du Round-Robin

### Configuration pour www.rev.local
1. **Créer deux enregistrements A** :
   - www.rev.local → 192.168.1.8
   - www.rev.local → 192.168.1.9

2. **Activer le round-robin** :
   - Dans les propriétés du serveur DNS
   - Onglet "Avancé"
   - Cocher "Activer le round-robin"

3. **Activer le masquage de sous-réseau** :
   - Cocher "Activer le masquage de sous-réseau"
   - Améliore la répartition de charge

### Test du round-robin
```powershell
# Test de résolution multiple
for ($i=1; $i -le 5; $i++) {
    $result = nslookup www.rev.local
    Write-Host "Test $i : $result"
    Start-Sleep -Seconds 1
}
```

## 🔧 Configuration avancée

### Transferts de zone
- **Autoriser les transferts** : Vers les serveurs spécifiés
- **Serveurs autorisés** : 192.168.1.11 (BDC)
- **Notifications** : Activées

### Redirecteurs
- **Redirecteur principal** : 8.8.8.8 (Google DNS)
- **Redirecteur secondaire** : 1.1.1.1 (Cloudflare DNS)
- **Utiliser les redirecteurs racine** : Non

### Options de serveur
- **Options de débogage** : Désactivées en production
- **Journalisation des requêtes** : Activée pour le dépannage
- **Nettoyage automatique** : Activé (7 jours)

## 📊 Monitoring et maintenance

### Surveillance de la résolution DNS
```powershell
# Test de résolution DNS
Test-DnsServer -ComputerName pdc.rev.local -ZoneName rev.local

# Statistiques du serveur DNS
Get-DnsServerStatistics -ComputerName pdc.rev.local

# Surveillance des requêtes
Get-DnsServerQueryStatistics -ComputerName pdc.rev.local
```

### Nettoyage des enregistrements obsolètes
1. **Activer le vieillissement** : Dans les propriétés de la zone
2. **Période de non-rafraîchissement** : 7 jours
3. **Période de rafraîchissement** : 7 jours
4. **Nettoyage automatique** : Activé

### Script de maintenance
```powershell
# Script de maintenance DNS
Clear-DnsServerCache -ComputerName pdc.rev.local

# Nettoyer les enregistrements obsolètes
Invoke-DnsServerZoneScavenge -ComputerName pdc.rev.local

# Exporter la configuration
Export-DnsServerZone -Name rev.local -ComputerName pdc.rev.local -FileName C:\Backup\rev.local.dns
```

## 🔒 Sécurité DNS

### Sécurisation des zones
- **Mises à jour dynamiques** : Sécurisées uniquement
- **Transferts de zone** : Sécurisés
- **Restrictions des requêtes** : Uniquement les clients spécifiés

### Filtrage des requêtes
- **Autoriser les requêtes** : Depuis le réseau interne uniquement
- **Bloquer les requêtes** : Depuis Internet direct
- **Journalisation** : Des requêtes bloquées

### DNSSEC (si applicable)
- **Signature des zones** : Configuration avancée
- **Validation des réponses** : Activée
- **Clés de confiance** : Configurées

## 🚨 Dépannage

### Problèmes courants
1. **Résolution de noms échoue**
   - Vérifier la connectivité réseau
   - Confirmer la configuration DNS du client
   - Valider les enregistrements DNS

2. **Mises à jour dynamiques échouent**
   - Vérifier les permissions sur la zone
   - Confirmer l'intégration AD
   - Valider l'authentification

3. **Round-robin non fonctionnel**
   - Vérifier la configuration du round-robin
   - Confirmer les enregistrements multiples
   - Tester depuis différents clients

### Commandes utiles
```powershell
# Vider le cache DNS local
Clear-DnsClientCache

# Tester la résolution DNS
Resolve-DnsName -Name www.rev.local -Server pdc.rev.local

# Vérifier les enregistrements
Get-DnsServerResourceRecord -ZoneName rev.local -ComputerName pdc.rev.local

# Forcer la réplication DNS
Sync-DnsServerZone -ComputerName pdc.rev.local -Name rev.local
```

## 📈 Performance et optimisation

### Optimisation des performances
- **Cache DNS** : Taille augmentée pour les requêtes fréquentes
- **Threads de traitement** : Configurés selon la charge
- **Mémoire allouée** : Surveillance et ajustement

### Équilibrage de charge
- **Round-robin DNS** : Pour les serveurs web
- **Priorité des enregistrements** : Pour les services critiques
- **Poids des enregistrements** : Pour la répartition avancée

## 📸 Captures d'écran

### Console DNS principale
![DNS Console](../screenshots/04-network-services/dns-console.png)

### Configuration du round-robin
![Round Robin Config](../screenshots/04-network-services/round-robin-config.png)

### Propriétés de la zone
![Zone Properties](../screenshots/04-network-services/zone-properties.png)

---

**Document** : Configuration du serveur DNS  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
