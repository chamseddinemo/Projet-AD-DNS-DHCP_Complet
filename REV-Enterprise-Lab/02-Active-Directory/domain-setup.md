# Configuration du domaine Active Directory

## 📋 Description

Ce document explique la procédure de configuration du domaine Active Directory REV.LOCAL pour l'environnement REV-Enterprise-Lab.

## 🎯 Objectifs

- Installer les services de domaine Active Directory
- Promouvoir le serveur en contrôleur de domaine
- Configurer les paramètres de base du domaine
- Valider le fonctionnement du domaine

## 🛠️ Prérequis

### Configuration système
- **Windows Server 2019/2022** installé
- **Adresse IP statique** : 192.168.1.10
- **Nom du serveur** : PDC
- **Mise à jour Windows** : Appliquées

### Configuration réseau
```
Adresse IP : 192.168.1.10
Masque : 255.255.255.0
Passerelle : 192.168.1.1
DNS préféré : 127.0.0.1 (pointe vers lui-même)
```

## 📝 Procédure d'installation

### Étape 1 : Installation du rôle AD DS

1. **Ouvrir le Gestionnaire de serveur**
   - Démarrer → Gestionnaire de serveur

2. **Ajouter un rôle**
   - Cliquer sur "Ajouter des rôles et fonctionnalités"
   - Type d'installation : "Installation basée sur un rôle"
   - Sélection du serveur : PDC

3. **Sélectionner les rôles**
   - Cocher "Services de domaine Active Directory"
   - Ajouter les fonctionnalités requises
   - Suivant jusqu'à la confirmation

4. **Installer**
   - Cliquer sur "Installer"
   - Attendre la fin de l'installation

### Étape 2 : Promotion en contrôleur de domaine

1. **Promouvoir le serveur**
   - Notification : "Promouvoir ce serveur en contrôleur de domaine"
   - Ou : Gestionnaire de serveur → Drapeau → "Promouvoir ce serveur"

2. **Configuration du déploiement**
   - Sélectionner : "Ajouter une nouvelle forêt"
   - Nom de domaine racine : `rev.local`

3. **Paramètres du contrôleur de domaine**
   - Mot de passe Mode de restauration : `ComplexPassword123!`
   - Conserver ce mot de passe en sécurité

4. **Options DNS**
   - Cocher "Serveur DNS"
   - Ne pas cocher "Serveur de catalogue global" (uniquement pour premier DC)

5. **Chemins NETBIOS et d'installation**
   - NETBIOS : `REV` (généré automatiquement)
   - Chemins : Conserver les valeurs par défaut

6. **Vérification**
   - Vérifier toutes les options
   - Cliquer sur "Suivant" puis "Installer"

### Étape 3 : Configuration post-installation

1. **Redémarrage**
   - Le serveur redémarre automatiquement
   - Se connecter avec le compte Administrateur : `REV\Administrateur`

2. **Vérification DNS**
   - Ouvrir "Gestionnaire DNS"
   - Vérifier les zones créées automatiquement

3. **Validation du domaine**
   - Ouvrir "Utilisateurs et ordinateurs Active Directory"
   - Vérifier les conteneurs par défaut

## ✅ Validation de l'installation

### Tests de base
1. **Connexion au domaine**
   ```
   Utilisateur : REV\Administrateur
   Mot de passe : [mot de passe défini]
   ```

2. **Vérification DNS**
   ```powershell
   nslookup rev.local
   nslookup pdc.rev.local
   ```

3. **Test de réplication**
   ```powershell
   dcdiag /e /c /v
   ```

### Services à vérifier
- **Active Directory Domain Services** : En cours d'exécution
- **DNS Server** : En cours d'exécution
- **Netlogon** : En cours d'exécution
- **Kerberos Key Distribution Center** : En cours d'exécution

## 🔧 Configuration avancée

### Paramètres du domaine
1. **Ouvrir "Utilisateurs et ordinateurs Active Directory"**
2. **Clic droit sur "rev.local" → Propriétés**
3. **Onglet "Général"**
   - Description : "Domaine principal REV Enterprise"
4. **Onglet "Stratégies de groupe"**
   - Lier les stratégies par défaut

### Configuration des sites
1. **Ouvrir "Sites et services Active Directory"**
2. **Vérifier le site par défaut "Default-First-Site-Name"**
3. **Renommer en "REV-Site"** si nécessaire

## 🚨 Dépannage courant

### Problèmes fréquents
1. **Échec de promotion**
   - Vérifier la configuration IP
   - S'assurer que le DNS pointe vers 127.0.0.1

2. **DNS ne fonctionne pas**
   - Redémarrer le service DNS
   - Vérifier les enregistrements SRV

3. **Connexion impossible**
   - Vérifier le mot de passe du Mode de restauration
   - Utiliser le bon format : `DOMAINE\utilisateur`

### Commandes utiles
```powershell
# Vérifier l'état du contrôleur de domaine
dcdiag

# Vérifier la réplication
repadmin /showrepl

# Vérifier les événements AD
Get-WinEvent -LogName "Directory Service" -MaxEvents 20
```

## 📸 Captures d'écran

### Assistant d'installation AD DS
![AD DS Installation](../screenshots/02-active-directory/ad-ds-installation.png)

### Configuration du domaine
![Domain Configuration](../screenshots/02-active-directory/domain-configuration.png)

### Validation post-installation
![Post Installation Validation](../screenshots/02-active-directory/post-install-validation.png)

---

**Document** : Configuration du domaine Active Directory  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
