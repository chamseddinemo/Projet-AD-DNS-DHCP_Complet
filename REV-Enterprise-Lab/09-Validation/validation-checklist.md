# Checklist de validation

## 📋 Description

Ce document présente la checklist complète de validation pour l'ensemble des services configurés dans le laboratoire REV-Enterprise-Lab.

## 🎯 Objectifs

- Valider le fonctionnement de tous les services
- Vérifier la conformité des configurations
- Documenter les résultats des tests
- Identifier les problèmes potentiels

## ✅ Checklist principale

### Infrastructure de base
- [ ] **Serveur PDC** : Installé et opérationnel
- [ ] **Active Directory** : Domaine rev.local fonctionnel
- [ ] **DNS** : Résolution de noms interne/externe
- [ ] **DHCP** : Attribution d'adresses IP
- [ ] **Réseau** : Connectivité complète
- [ ] **Sécurité** : Politiques appliquées

### Services Active Directory
- [ ] **Contrôleur de domaine** : Promotion réussie
- [ ] **Structure OU** : Créée selon le plan
- [ ] **Utilisateurs** : Créés et organisés
- [ ] **Groupes** : Configurés et peuplés
- [ ] **Ordinateurs** : Joint au domaine
- [ ] **Stratégies de groupe** : Liées et appliquées

### Services réseau
- [ ] **DHCP** : Étendue configurée
- [ ] **Réservations** : Configurées pour les serveurs
- [ ] **DNS** : Zones créées et fonctionnelles
- [ ] **Round-robin** : Configuré pour www.rev.local
- [ ] **Redirecteurs** : Configurés pour Internet

### Services de fichiers
- [ ] **Partages réseau** : Créés et sécurisés
- [ ] **Permissions NTFS** : Configurées correctement
- [ ] **Quotas** : Appliqués par utilisateur
- [ ] **Mappage lecteurs** : Configuré via GPO

### Sécurité
- [ ] **Politique de mot de passe** : Configurée et appliquée
- [ ] **Verrouillage de compte** : Configuré et testé
- [ ] **Restrictions utilisateur** : Appliquées par département
- [ ] **Audit de sécurité** : Activé et fonctionnel

## 📊 Tests détaillés

### Test 1 : Authentification Active Directory

#### Scénario 1 : Connexion utilisateur standard
1. **Utilisateur** : john.smith (HR)
2. **Poste** : HR-PC01
3. **Actions** :
   - Se connecter avec mot de passe temporaire
   - Changer le mot de passe
   - Accéder aux ressources HR
4. **Résultat attendu** : ✅ Succès
5. **Résultat obtenu** : [ ]

#### Scénario 2 : Connexion utilisateur IT
1. **Utilisateur** : admin.user (IT)
2. **Poste** : IT-PC01
3. **Actions** :
   - Se connecter au domaine
   - Accéder aux outils d'administration
   - Gérer les utilisateurs
4. **Résultat attendu** : ✅ Succès
5. **Résultat obtenu** : [ ]

### Test 2 : Services DHCP

#### Scénario 1 : Attribution IP automatique
1. **Client** : Ordinateur portable non configuré
2. **Actions** :
   - Connecter au réseau
   - Demander une adresse IP via DHCP
   - Vérifier la configuration obtenue
3. **Résultat attendu** : Adresse IP dans 192.168.1.40-230
4. **Résultat obtenu** : [ ]

#### Scénario 2 : Réservation DHCP
1. **Client** : HRPC01
2. **Actions** :
   - Connecter au réseau
   - Vérifier l'adresse IP obtenue
   - Confirmer la réservation
3. **Résultat attendu** : 192.168.1.200
4. **Résultat obtenu** : [ ]

### Test 3 : Services DNS

#### Scénario 1 : Résolution interne
1. **Commande** : `nslookup pdc.rev.local`
2. **Résultat attendu** : 192.168.1.10
3. **Résultat obtenu** : [ ]

#### Scénario 2 : Résolution externe
1. **Commande** : `nslookup google.com`
2. **Résultat attendu** : Adresse IP de google.com
3. **Résultat obtenu** : [ ]

#### Scénario 3 : Round-robin
1. **Commande** : `nslookup www.rev.local` (multiple fois)
2. **Résultat attendu** : Alternance entre 192.168.1.8 et 192.168.1.9
3. **Résultat obtenu** : [ ]

### Test 4 : Accès aux partages réseau

#### Scénario 1 : Accès départemental
1. **Utilisateur** : john.smith (HR)
2. **Actions** :
   - Accéder à \\file.rev.local\HR
   - Créer un fichier
   - Modifier le fichier
3. **Résultat attendu** : ✅ Accès complet
4. **Résultat obtenu** : [ ]

#### Scénario 2 : Accès inter-départemental
1. **Utilisateur** : john.smith (HR)
2. **Actions** :
   - Accéder à \\file.rev.local\Sales
   - Tenter de créer un fichier
3. **Résultat attendu** : ❌ Accès refusé
4. **Résultat obtenu** : [ ]

### Test 5 : Politiques de sécurité

#### Scénario 1 : Politique de mot de passe
1. **Utilisateur** : jane.doe (HR)
2. **Actions** :
   - Tenter un mot de passe simple : "password"
   - Tenter un mot de passe complexe : "Password123!"
3. **Résultat attendu** : Rejeté, puis accepté
4. **Résultat obtenu** : [ ]

#### Scénario 2 : Restrictions utilisateur
1. **Utilisateur** : david.jones (Sales)
2. **Actions** :
   - Tenter d'ouvrir l'invite de commandes
   - Tenter d'ouvrir le panneau de configuration
3. **Résultat attendu** : ❌ Accès refusé
4. **Résultat obtenu** : [ ]

## 🔍 Validation des commandes

### Commandes de test
```powershell
# Test de connectivité réseau
Test-NetConnection -ComputerName pdc.rev.local -Port 445

# Test de résolution DNS
Resolve-DnsName -Name www.rev.local -Server pdc.rev.local

# Test d'authentification AD
Get-ADUser -Filter {Enabled -eq $true} | Measure-Object

# Test des partages réseau
Test-Path "\\file.rev.local\Public"

# Test des stratégies de groupe
gpresult /r
```

### Scripts de validation
```powershell
# Script de validation complète
function Test-REVInfrastructure {
    Write-Host "Validation de l'infrastructure REV-Enterprise-Lab" -ForegroundColor Green
    
    # Test AD
    Write-Host "Test Active Directory..."
    try {
        $adTest = Get-ADDomain
        Write-Host "✅ Active Directory fonctionnel" -ForegroundColor Green
    } catch {
        Write-Host "❌ Active Directory en erreur" -ForegroundColor Red
    }
    
    # Test DNS
    Write-Host "Test DNS..."
    try {
        $dnsTest = Resolve-DnsName -Name pdc.rev.local
        Write-Host "✅ DNS fonctionnel" -ForegroundColor Green
    } catch {
        Write-Host "❌ DNS en erreur" -ForegroundColor Red
    }
    
    # Test DHCP
    Write-Host "Test DHCP..."
    $dhcpTest = Get-DhcpServerv4ScopeStatistics -ComputerName pdc.rev.local
    if ($dhcpTest) {
        Write-Host "✅ DHCP fonctionnel" -ForegroundColor Green
    } else {
        Write-Host "❌ DHCP en erreur" -ForegroundColor Red
    }
    
    # Test partages
    Write-Host "Test partages réseau..."
    $shareTest = Test-Path "\\file.rev.local\Public"
    if ($shareTest) {
        Write-Host "✅ Partages réseau fonctionnels" -ForegroundColor Green
    } else {
        Write-Host "❌ Partages réseau en erreur" -ForegroundColor Red
    }
}

# Exécuter la validation
Test-REVInfrastructure
```

## 📈 Rapport de validation

### Format du rapport
```
RAPPORT DE VALIDATION - REV-Enterprise-Lab
Date : [Date du test]
Opérateur : [Nom du testeur]

RÉSUMÉ DES TESTS
- Tests exécutés : [Nombre]
- Tests réussis : [Nombre]
- Tests échoués : [Nombre]
- Taux de réussite : [Pourcentage]%

DÉTAILS PAR SERVICE
Active Directory : [État]
DNS : [État]
DHCP : [État]
Fichiers : [État]
Sécurité : [État]

PROBLÈMES IDENTIFIÉS
1. [Description du problème]
2. [Description du problème]

RECOMMANDATIONS
1. [Recommandation 1]
2. [Recommandation 2]
```

## 🚨 Critères de succès

### Validation réussie
- **Taux de réussite** : ≥ 95%
- **Services critiques** : 100% fonctionnels
- **Sécurité** : Politiques appliquées
- **Performance** : Temps de réponse acceptable

### Validation partielle
- **Taux de réussite** : 80-94%
- **Problèmes mineurs** : Documentés avec solutions
- **Services critiques** : Fonctionnels
- **Plan d'action** : Défini

### Validation échouée
- **Taux de réussite** : < 80%
- **Services critiques** : Non fonctionnels
- **Problèmes majeurs** : Identifiés
- **Intervention requise** : Immédiate

## 📸 Captures d'écran

### Rapport de validation
![Validation Report](../screenshots/09-validation/validation-report.png)

### Test de connectivité
![Connectivity Test](../screenshots/09-validation/connectivity-test.png)

### Validation des services
![Services Validation](../screenshots/09-validation/services-validation.png)

---

**Document** : Checklist de validation  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
