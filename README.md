## **1. Introduction**
Dans une PME de 100 postes, l’entreprise souhaite centraliser la gestion des utilisateurs, des ressources et de la sécurité via un serveur Windows Server 2022 configuré en contrôleur de domaine.
Ce TP vous permettra de :
Compléter et configurer l’infrastructure Active Directory existante
Déployer et tester DHCP et DNS
Organiser les OU et groupes de sécurité
Créer des partages réseau avec permissions combinées (NTFS + partage)
Appliquer des GPO pour sécuriser et personnaliser les postes clients
Vérifier et valider toute l’infrastructure
Serveur de référence : SRV-AD
Domaine : entreprise.local
IP : 192.168.77.10/24

## **2. Objectifs pédagogiques**

Après ce TP, vous serez capable de :

- Vérifier et configurer la connectivité réseau des serveurs et postes clients
- Installer et configurer les rôles **DHCP** et **DNS** sur Windows Server 2022
- Organiser et structurer un domaine **Active Directory** avec OU, utilisateurs et groupes de sécurité
- Créer des **partages réseau** avec permissions combinées NTFS et partage
- Appliquer des **GPO ciblées** pour sécuriser et personnaliser les postes clients
- Tester et valider le fonctionnement des services et des politiques appliquées
- Comprendre l’importance de la centralisation et de la sécurité dans une infrastructure d’entreprise
















Infrastructure Active Directory – Windows Server 2022

## Présentation du projet
Déploiement complet d'une infrastructure pour une PME de 100 postes visant à centraliser la gestion des utilisateurs, des ressources et de la sécurité.

* **Domaine :** `entreprise.local`
* **Serveur :** Windows Server 2022 (SRV-AD)
* **Services :** AD DS, DNS, DHCP, Serveur de fichiers.

---

##Réalisations techniques
* **DHCP & DNS :** Configuration d'une étendue IP (192.168.77.100-200) et résolution de noms.
* **Active Directory :** Création d'Unités d'Organisation (OU) et groupes (Compta, RH, IT, Stagiaires).
* **GPO :** Blocage du Panneau de configuration et redirection des dossiers utilisateurs.
* **Sécurité :** Gestion des partages réseau avec permissions NTFS et partage (méthode AGDLP).

---

## 🔍 Validation
* Test d'attribution IP automatique.
* Vérification de l'application des stratégies de groupe via `gpresult /r`.
