# Jointure de domaine Windows 10

## 📋 Description

Ce document explique la procédure pour joindre un poste Windows 10 au domaine REV.LOCAL et configurer les paramètres de base.

## 🎯 Objectifs

- Joindre un poste Windows 10 au domaine
- Configurer les paramètres réseau appropriés
- Valider l'authentification domaine
- Préparer le poste pour l'utilisation

## 🛠️ Prérequis

### Configuration système
- **Windows 10 Pro/Entreprise** installé
- **Connexion réseau** fonctionnelle
- **Accès administrateur** local
- **Nom du poste** conforme aux conventions

### Configuration réseau
- **Adresse IP** : DHCP ou statique selon le plan
- **DNS primaire** : 192.168.1.10 (PDC)
- **DNS secondaire** : 8.8.8.8
- **Passerelle** : 192.168.1.1

## 📝 Procédure de jointure

### Méthode graphique

#### Étape 1 : Configuration du nom d'ordinateur
1. **Ouvrir les paramètres système**
   - Clic droit sur "Ce PC" → Propriétés
   - Ou : Démarrer → Paramètres → Système → À propos

2. **Renommer le poste**
   - Cliquer sur "Renommer ce PC"
   - Entrer le nom selon la convention : `[DEPARTEMENT]-PC[NN]`
   - Exemple : `HR-PC01`, `SALES-PC02`
   - Redémarrer si nécessaire

#### Étape 2 : Jointure au domaine
1. **Ouvrir les paramètres système**
   - Dans "À propos", cliquer sur "Modifier les paramètres"

2. **Modifier l'appartenance**
   - Cliquer sur "Modifier"
   - Sélectionner "Domaine"
   - Entrer : `rev.local`
   - Cliquer sur "OK"

3. **Authentification**
   - Nom d'utilisateur : `REV\Administrateur`
   - Mot de passe : [mot de passe administrateur]
   - Cliquer sur "OK"

4. **Confirmation**
   - Message de bienvenue dans le domaine
   - Cliquer sur "OK"
   - Redémarrer l'ordinateur

### Méthode PowerShell

```powershell
# Renommer et joindre au domaine
Rename-Computer -NewName "HR-PC01" -Force
Add-Computer -DomainName rev.local -Credential REV\Administrateur -Restart

# Ou en une seule commande
Add-Computer -NewName "HR-PC01" -DomainName rev.local -Credential REV\Administrateur -Restart
```

## 🔍 Validation de la jointure

### Vérification post-jointure
1. **Vérifier le nom du domaine**
   ```powershell
   $env:USERDOMAIN
   # Devrait retourner : REV
   ```

2. **Vérifier le nom complet de l'ordinateur**
   ```powershell
   $env:COMPUTERNAME
   # Devrait retourner : HR-PC01
   ```

3. **Vérifier la connectivité domaine**
   ```powershell
   ping pdc.rev.local
   nslookup rev.local
   ```

### Test de connexion
1. **Se déconnecter** du compte local
2. **Se connecter** avec un compte domaine
   - Utilisateur : `REV\john.smith`
   - Mot de passe : [mot de passe utilisateur]
3. **Valider** l'accès aux ressources réseau

## 🔧 Configuration post-jointure

### Mise à jour des stratégies
```powershell
# Forcer la mise à jour des stratégies de groupe
gpupdate /force

# Redémarrer si nécessaire
gpupdate /force /boot
```

### Installation des logiciels de base
1. **Antivirus entreprise**
   - Déployé via GPO
   - Ou installation manuelle

2. **Agents de monitoring**
   - Agent de surveillance système
   - Agent de gestion des correctifs

3. **Logiciels de productivité**
   - Microsoft Office
   - Navigateurs web autorisés

### Configuration des profils utilisateur
1. **Création des dossiers utilisateur**
   - Dossier personnel sur le serveur de fichiers
   - Mappage des lecteurs réseau

2. **Configuration des favoris**
   - Accès rapide aux partages réseau
   - Liens vers les applications internes

## 🖥️ Configuration des paramètres locaux

### Paramètres régionaux
- **Format** : Français (France)
- **Emplacement** : France
- **Clavier** : Français (AZERTY)

### Paramètres de sécurité
- **Mises à jour Windows** : Automatiques
- **Pare-feu** : Configuré par GPO
- **Contrôle de compte d'utilisateur** : Configuré par GPO

### Paramètres réseau
- **Type de profil** : Privé (entreprise)
- **Découverte réseau** : Activée
- **Partage de fichiers** : Configuré par GPO

## 📊 Validation complète

### Checklist de validation
- [ ] Nom du poste conforme aux conventions
- [ ] Jointure au domaine réussie
- [ ] Connexion avec compte domaine fonctionnelle
- [ ] Stratégies de groupe appliquées
- [ ] Accès aux partages réseau
- [ ] Logiciels de base installés
- [ ] Mises à jour appliquées
- [ ] Antivirus opérationnel

### Tests fonctionnels
1. **Accès aux partages**
   ```powershell
   # Tester l'accès au partage HR
   Test-Path "\\file.rev.local\HR"
   ```

2. **Impression réseau**
   ```powershell
   # Lister les imprimantes disponibles
   Get-Printer
   ```

3. **Connexion Internet**
   ```powershell
   # Tester la connectivité Internet
   Test-NetConnection -ComputerName google.com -Port 443
   ```

## 🚨 Dépannage

### Problèmes courants
1. **Échec de jointure au domaine**
   - Vérifier la connectivité réseau
   - Confirmer la résolution DNS
   - Valider les permissions administratives

2. **Stratégies de groupe non appliquées**
   - Forcer la mise à jour : `gpupdate /force`
   - Vérifier l'appartenance à l'OU
   - Confirmer les liens GPO

3. **Connexion impossible**
   - Vérifier le format du nom d'utilisateur
   - Confirmer le mot de passe
   - Valider l'heure du système

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

# Vérifier la configuration IP
ipconfig /all
```

## 📸 Captures d'écran

### Jointure au domaine
![Domain Join](../screenshots/06-client-configuration/domain-join.png)

### Validation de la jointure
![Domain Join Validation](../screenshots/06-client-configuration/domain-validation.png)

### Application des stratégies
![GPO Application](../screenshots/06-client-configuration/gpo-application.png)

---

**Document** : Jointure de domaine Windows 10  
**Version** : 1.0  
**Dernière mise à jour** : Mai 2026
