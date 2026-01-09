# Infrastructure Active Directory – Windows Server 2022

## 📝 Présentation du projet
Déploiement complet d'une infrastructure pour une PME de 100 postes visant à centraliser la gestion des utilisateurs, des ressources et de la sécurité.

* **Domaine :** `entreprise.local`
* **Serveur :** Windows Server 2022 (SRV-AD)
* **Services :** AD DS, DNS, DHCP, Serveur de fichiers.

---

## 🛠️ Réalisations techniques
* **DHCP & DNS :** Configuration d'une étendue IP (192.168.77.100-200) et résolution de noms.
* **Active Directory :** Création d'Unités d'Organisation (OU) et groupes (Compta, RH, IT, Stagiaires).
* **GPO :** Blocage du Panneau de configuration et redirection des dossiers utilisateurs.
* **Sécurité :** Gestion des partages réseau avec permissions NTFS et partage (méthode AGDLP).

---

## 🔍 Validation
* Test d'attribution IP automatique.
* Vérification de l'application des stratégies de groupe via `gpresult /r`.
