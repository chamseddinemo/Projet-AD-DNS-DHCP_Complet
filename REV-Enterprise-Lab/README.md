# Guide Help Desk - REV Enterprise

## 🎯 Objectif du projet
Guide simple pour le support technique des services IT de base de l'entreprise REV.

## 📋 Services principaux

### 🔐 Gestion des utilisateurs
- **Création de compte** : Demander nom, prénom, département
- **Réinitialisation mot de passe** : Vérifier identité avant
- **Déblocage compte** : 3 tentatives échouées = blocage

### 🌐 Connexion réseau
- **WiFi** : REV-Corp (mot de passe: Rev2026!)
- **VPN** : vpn.rev.local (pour télétravail)
- **Problèmes réseau** : Vérifier câble + redémarrer PC

### 💾 Accès aux fichiers
- **Lecteur H:** : Dossier personnel de l'utilisateur
- **Lecteur P:** : Partage départemental
- **Lecteur S:** : Documents partagés entreprise

### 🖨️ Imprimantes
- **Imprimante par défaut** : HP-OfficeJet-Dept
- **Impression couleur** : HP-Color-Central (sur autorisation)
- **Problème d'impression** : Redémarrer spooler d'impression

## 🆘 Problèmes courants

### Mot de passe oublié
1. Vérifier identité (téléphone/email)
2. Réinitialiser mot de passe temporaire
3. Forcer changement à la prochaine connexion

### PC lent
1. Vérifier espace disque (minimum 10% libre)
2. Nettoyer fichiers temporaires
3. Redémarrer PC
4. Si persiste : escalader au support niveau 2

### Accès refusé
1. Vérifier groupe de l'utilisateur
2. Confirmer permissions sur le dossier
3. Si nécessaire : ajouter aux groupes appropriés

## 📞 Contacts importants

### Support interne
- **Help Desk Principal** : 01 23 45 67 89
- **Support Niveau 2** : 01 23 45 67 90
- **Administrateur système** : admin@rev.local

### Informations réseau
- **Domaine** : rev.local
- **Serveur principal** : PDC.rev.local
- **Passerelle** : 192.168.1.1
- **DNS** : 192.168.1.10

## 🔍 Checklist diagnostic

### Avant d'escalader
- [ ] PC redémarré ?
- [ ] Câble réseau branché ?
- [ ] Mot de passe correct (MAJuscules) ?
- [ ] Espace disque suffisant ?
- [ ] Antivirus à jour ?

### Informations à collecter
- Nom de l'utilisateur
- Numéro de poste
- Message d'erreur exact
- Heure du problème
- Depuis quand le problème existe

## 📝 Procédures rapides

### Création utilisateur standard
1. Ouvrir "Utilisateurs et ordinateurs Active Directory"
2. Clic droit sur le département → Nouvel utilisateur
3. Remplir formulaire (nom, prénom, login)
4. Mot de passe temporaire : Temp2026!
5. Cocher "L'utilisateur doit changer le mot de passe"
6. Ajouter aux groupes de base

### Réinitialisation mot de passe
1. Trouver l'utilisateur dans AD
2. Clic droit → Réinitialiser le mot de passe
3. Nouveau mot de passe temporaire
4. Cocher "L'utilisateur doit changer le mot de passe"
5. Décocher "Verrouiller le compte"

---

**Version simplifiée Help Desk**  
**Dernière mise à jour** : Mai 2026  
**Contact** : helpdesk@rev.local
