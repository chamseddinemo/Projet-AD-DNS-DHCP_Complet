<h1>🏢 REV-Enterprise-Lab</h1>

<h2>📌 Description du projet</h2>
<p>
REV-Enterprise-Lab est un environnement de laboratoire Windows Server conçu pour simuler les opérations IT d'une entreprise réelle.
Ce projet démontre l'implémentation et la gestion des services Windows Server essentiels.
</p>
![Disk Restriction](./screenshots/GPO/Capture d’écran (417).png)
<hr>

<h2>🎯 Objectifs</h2>
<ul>
<li>Simuler un environnement Windows Server d'entreprise</li>
<li>Déployer Active Directory avec structure organisationnelle</li>
<li>Implémenter des stratégies de groupe pour la sécurité</li>
<li>Configurer les services réseau (DHCP, DNS)</li>
<li>Établir une infrastructure de serveur de fichiers sécurisée</li>
<li>Documenter tous les processus pour le support technique</li>
</ul>

<hr>

<h2>🛠️ Technologies utilisées</h2>
<table>
<tr><th>Technologie</th><th>Version</th><th>Usage</th></tr>
<tr><td>Windows Server</td><td>2019/2022</td><td>Serveur principal</td></tr>
<tr><td>Windows 10</td><td>Entreprise</td><td>Poste client</td></tr>
<tr><td>Active Directory</td><td>Services de domaine</td><td>Gestion des identités</td></tr>
<tr><td>Stratégies de groupe</td><td>Console de gestion</td><td>Application des politiques</td></tr>
<tr><td>DHCP</td><td>Rôle</td><td>Gestion des adresses IP</td></tr>
<tr><td>DNS</td><td>Rôle</td><td>Résolution de noms</td></tr>
<tr><td>Serveur de fichiers</td><td>Rôle</td><td>Stockage centralisé</td></tr>
</table>

<hr>

<h2>🏗️ Détails de l'environnement</h2>

<h3>Configuration réseau</h3>
<ul>
<li><b>Nom de domaine :</b> rev.local</li>
<li><b>Contrôleur de domaine :</b> PDC.rev.local</li>
<li><b>IP serveur :</b> 192.168.1.10</li>
<li><b>Client :</b> HRPC01.rev.local</li>
<li><b>IP réservée :</b> 192.168.1.200</li>
</ul>

<h3>Plan d’adressage IP</h3>
<table>
<tr><th>Composant</th><th>Adresse IP</th><th>Masque</th><th>Passerelle</th></tr>
<tr><td>PDC</td><td>192.168.1.10</td><td>255.255.255.0</td><td>192.168.1.1</td></tr>
<tr><td>Client HRPC01</td><td>192.168.1.200</td><td>255.255.255.0</td><td>192.168.1.1</td></tr>
<tr><td>DHCP</td><td>192.168.1.40-230</td><td>255.255.255.0</td><td>192.168.1.1</td></tr>
</table>

<hr>

<h2>📋 Architecture</h2>

<pre>
REV ENTERPRISE LAB
│
├── PDC (192.168.1.10)
├── File Server (192.168.1.15)
└── Client (192.168.1.200)

Services:
- Active Directory
- DHCP
- DNS

Departments:
HR | HK | Sales | IT
</pre>

<hr>

<h2>✨ Fonctionnalités</h2>

<h3>🔐 Gestion des identités</h3>
<ul>
<li>Active Directory Domain Services</li>
<li>OU par département</li>
<li>Gestion utilisateurs et groupes</li>
</ul>

<h3>📋 GPO</h3>
<ul>
<li>Blocage CMD et Control Panel</li>
<li>Restriction stockage externe</li>
<li>Stratégies de mot de passe</li>
</ul>

<h3>🌐 Réseau</h3>
<ul>
<li>DHCP avec scope + réservation</li>
<li>DNS avec load balancing</li>
</ul>

<h3>💾 File Server</h3>
<ul>
<li>Dossiers partagés</li>
<li>Permissions NTFS</li>
<li>Quotas disque</li>
</ul>

<h3>🔒 Sécurité</h3>
<ul>
<li>Complexité mot de passe</li>
<li>Account lockout</li>
<li>Restrictions utilisateurs</li>
</ul>

<h3>🖥️ Client</h3>
<ul>
<li>Join domain</li>
<li>Admin local via groupe IT</li>
<li>GPO appliquées</li>
</ul>

<hr>

<h2>📸 Screenshots</h2>

<p><b>Architecture</b></p>
<img src="screenshots/architecture.png" width="600">

<p><b>Active Directory</b></p>
<img src="screenshots/ad.png" width="600">

<p><b>GPO</b></p>
<img src="screenshots/gpo.png" width="600">

<p><b>Network</b></p>
<img src="screenshots/network.png" width="600">

<p><b>File Server</b></p>
<img src="screenshots/fileserver.png" width="600">

<p><b>Client</b></p>
<img src="screenshots/client.png" width="600">

<p><b>Security</b></p>
<img src="screenshots/security.png" width="600">

<hr>

<h2>🎓 Compétences</h2>
<table>
<tr><th>Catégorie</th><th>Compétences</th></tr>
<tr><td>Serveur</td><td>Windows Server, rôles</td></tr>
<tr><td>AD</td><td>Domain Controller, OU</td></tr>
<tr><td>GPO</td><td>Policies, restrictions</td></tr>
<tr><td>Réseau</td><td>DHCP, DNS</td></tr>
<tr><td>Sécurité</td><td>Policies, accès</td></tr>
<tr><td>Client</td><td>Join domain</td></tr>
</table>

<hr>

<h2>🚀 Améliorations futures</h2>
<ul>
<li>Ajouter un deuxième Domain Controller</li>
<li>Implémenter AD CS</li>
<li>Configurer VPN</li>
<li>Ajouter monitoring</li>
<li>Backup serveur</li>
</ul>

<hr>

<h2>✅ Validation</h2>
<table>
<tr><th>Composant</th><th>Test</th><th>Status</th></tr>
<tr><td>Active Directory</td><td>Login utilisateur</td><td>✅</td></tr>
<tr><td>GPO</td><td>gpupdate</td><td>✅</td></tr>
<tr><td>DHCP</td><td>IP assignée</td><td>✅</td></tr>
<tr><td>DNS</td><td>Résolution</td><td>✅</td></tr>
<tr><td>File Server</td><td>Accès</td><td>✅</td></tr>
<tr><td>Sécurité</td><td>Lockout</td><td>✅</td></tr>
</table>

<hr>

<h2>📚 Structure</h2>
<pre>
01-Architecture/
02-Active-Directory/
03-Group-Policy/
04-Network-Services/
05-File-Server/
06-Client-Configuration/
07-Security/
08-Applications/
09-Validation/
docs/
screenshots/
</pre>

<hr>

<h2>🤝 Contribuer</h2>
<p>
Projet utilisé comme lab et portfolio IT.
</p>

<hr>

<p><b>Status :</b> ✅ Complet</p>
<p><b>Version :</b> 1.0.0</p>
<p><b>Dernière mise à jour :</b> Mai 2026</p>
