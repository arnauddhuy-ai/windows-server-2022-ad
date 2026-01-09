# TP : Infrastructure Active Directory, DHCP, DNS, GPO et Partages

## 0. Prérequis matériels et logiciels

### 0.1 Machines virtuelles nécessaires

**Configuration minimale :**
- **1 serveur** : Windows Server 2022 (SRV-AD)
- **2 postes clients** : Windows 10 ou Windows 11

**Configuration recommandée pour tests complets :**
- **1 serveur** : Windows Server 2022 (SRV-AD)
- **4 postes clients** : Windows 10 ou Windows 11 (un par OU : Comptabilité, RH, IT, Stagiaires)

### 0.2 Spécifications techniques

| Machine | Système d'exploitation | RAM | Disque dur | Processeur | Réseau |
|---------|------------------------|-----|------------|------------|--------|
| **SRV-AD** | Windows Server 2022 | 4 GB | 60 GB | 2 vCPU | Host-only |
| **Clients** | Windows 10/11 | 2 GB | 40 GB | 1-2 vCPU | Host-only |

### 0.3 Logiciels requis

- **VMware Workstation** (ou VMware Player, VirtualBox)
- **ISO Windows Server 2022**
- **ISO Windows 10/11**

### 0.4 État initial requis

Avant de commencer ce TP, le serveur **SRV-AD** doit avoir :
- Windows Server 2022 installé
- Rôle **AD DS** installé et configuré
- Serveur promu en **contrôleur de domaine** pour le domaine `entreprise.local`
- Nom de machine : `SRV-AD`
- Les rôles DHCP et DNS seront installés durant ce TP

### 0.5 Durée estimée

**3 à 4 heures** (selon le nombre de clients et l'expérience)

---

## 1. Introduction

Dans une PME de 100 postes, l'entreprise souhaite centraliser la gestion des utilisateurs, des ressources et de la sécurité via un serveur Windows Server 2022 configuré en **contrôleur de domaine**.

Ce TP vous permettra de :

- Compléter et configurer l'infrastructure Active Directory existante
- Déployer et tester DHCP et DNS
- Organiser les OU et groupes de sécurité
- Créer des partages réseau avec permissions combinées (NTFS + partage)
- Appliquer des GPO pour sécuriser et personnaliser les postes clients
- Vérifier et valider toute l'infrastructure

**Serveur de référence :** `SRV-AD`

- Domaine : `entreprise.local`
- IP : `192.168.77.10/24`

---

## 2. Objectifs pédagogiques

Après ce TP, vous serez capable de :

- Vérifier et configurer la connectivité réseau des serveurs et postes clients
- Installer et configurer les rôles **DHCP** et **DNS** sur Windows Server 2022
- Organiser et structurer un domaine **Active Directory** avec OU, utilisateurs et groupes de sécurité
- Créer des **partages réseau** avec permissions combinées NTFS et partage
- Appliquer des **GPO ciblées** pour sécuriser et personnaliser les postes clients
- Tester et valider le fonctionnement des services et des politiques appliquées
- Comprendre l'importance de la centralisation et de la sécurité dans une infrastructure d'entreprise

---

## 3. Vérification de la configuration réseau de la VM

### 3.1 Dans VMware

1. Sélectionnez votre VM Windows Server (**SRV-AD**)
2. Menu : **VM > Settings > Network Adapter**
3. Vérifiez le **mode réseau** :

| Mode | Usage pour le TP |
|------|------------------|
| Bridged | VM sur le réseau physique réel |
| NAT | VM partage l'IP de l'hôte pour Internet |
| Host-only | VM isolée mais communique avec hôte et autres VMs |

> **Pour ce TP :** si pas besoin d'Internet → Host-only

### 3.2 Vérification réseau côté serveur

CMD :

```cmd
ipconfig /all
```

> **Vérifiez** que le serveur utilise **son IP comme DNS** (192.168.77.10)

---

## 4. Configuration de l'adresse IP fixe du serveur

- IP : `192.168.77.10`
- Masque : `255.255.255.0`
- Passerelle : `192.168.77.1` (fictive en host-only)
- DNS : `192.168.77.10`

### Vérification CMD :

```cmd
ipconfig /all
ping 192.168.77.10
```

### Astuce :

> Tester la connectivité avec un poste client avant de continuer

---

## 5. Installation et configuration du rôle DHCP

### 5.1 Installation du rôle

1. **Gestionnaire de serveur → Ajouter rôles et fonctionnalités**
2. Sélectionnez **DHCP Server → Installer**
3. Autorisez DHCP dans Active Directory

### 5.2 Création de l'étendue DHCP

1. Ouvrez console **DHCP**
2. IPv4 → clic droit → **Nouvelle étendue**
3. Paramètres :
   - Nom : `Etendue_VM`
   - Plage IP : `192.168.77.100 → 192.168.77.200`
   - Masque : `255.255.255.0`
   - Passerelle (Option 003) : `192.168.77.1`
   - DNS préféré (Option 006) : `192.168.77.10`

### 5.3 Réservation DHCP

Exemple imprimante :

- MAC : `00:11:22:33:44:55`
- IP réservée : `192.168.77.150`

### 5.4 Vérification côté client

1. Carte réseau → **Obtenir IP automatiquement**
2. CMD :

```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
ping 192.168.77.10
```

### Vérifiez :

> IP dans la plage, passerelle correcte, DNS = serveur AD

---

## 6. Configuration DNS

### 6.1 Vérification

- DNS installé automatiquement avec AD
- Permet la résolution de noms pour les services AD

### 6.2 Création d'un enregistrement A

1. Console DNS → zone `entreprise.local` → clic droit → **Nouvel hôte (A)**
2. Nom : `SRV-AD`
3. IP : `192.168.77.10`
4. Test CMD :

```cmd
nslookup SRV-AD.entreprise.local
```

### Résultat attendu :

> IP correcte

---

## 7. Organisation Active Directory

### 7.1 Création des OU

- Comptabilité
- RH
- IT
- Stagiaires

> Facilite la délégation et l'application ciblée des GPO

### 7.2 Création des utilisateurs

| OU | Utilisateurs |
|----|--------------|
| Comptabilité | cdupont, mbernard |
| RH | jdurand, sblanc |
| IT | tpetit, lmartin |
| Stagiaires | jmartin, apaul |

### 7.3 Création des groupes de sécurité

- `G_Comptabilité`, `G_RH`, `G_IT`, `G_Stagiaires`
- Ajouter chaque utilisateur dans son groupe respectif

---

## 8. Partages réseau et permissions

### 8.1 Création des dossiers partagés

Sur SRV-AD, créer :

```
C:\Partages\Compta
C:\Partages\RH
C:\Partages\Public
```

### 8.2 Permissions de partage

| Dossier | Groupe | Accès |
|---------|--------|-------|
| Compta | G_Comptabilité | Lecture/Écriture |
| RH | G_RH | Lecture/Écriture |
| Public | Tout le monde | Lecture |
| Public | G_IT | Lecture/Écriture |

### 8.3 Permissions NTFS

| Dossier | Groupe | Permissions |
|---------|--------|-------------|
| Compta | G_Comptabilité | Modifier / Contrôle total |
| RH | G_RH | Modifier |
| Public | G_IT | Modifier |
| Public | Tout le monde | Lecture seule |

### Résultat attendu :

> Chaque groupe accède uniquement à ses ressources.

---

## 9. Intégration des postes clients au domaine

### 9.1 Vérifier le DNS du client

```cmd
ipconfig /all
```

### Doit être :

> 192.168.77.10

### 9.2 Ajouter le poste au domaine

1. Propriétés système → Modifier → Domaine → `entreprise.local` → authentifier
2. Redémarrer le poste client
3. Connexion utilisateur AD :

```cmd
whoami
```

### Résultat attendu :

> entreprise\utilisateur

---

## 10. Stratégies de groupe (GPO)

### 10.1 GPO Comptabilité_Politique

#### 10.1.1 Création et liaison

- Ouvrir **GPMC** sur SRV-AD
- Naviguer : `Domaines → entreprise.local → OU Comptabilité`
- Clic droit → **Créer une GPO dans ce domaine et la lier ici**
- Nommer : `Comptabilité_Politique`

#### 10.1.2 Paramétrages

**1. Interdire l'accès au Panneau de configuration**

- Naviguer : `Configuration utilisateur → Modèles d'administration → Panneau de configuration`
- Paramètre : **Activé**

**2. Redirection du dossier Documents**

- Naviguer : `Configuration utilisateur → Stratégies → Paramètres Windows → Redirection de dossiers → Documents`
- Type : **De base – Rediriger le dossier Documents de tout le monde vers le même emplacement**
- Cocher : **Créer un dossier pour chaque utilisateur sous le chemin d'accès racine**
- Chemin racine : `\\SRV-AD\Compta`

### Résultat attendu :

> Pour chaque utilisateur de l'OU Comptabilité, un dossier `\\SRV-AD\Compta\nom_utilisateur` est créé automatiquement et le dossier Documents est redirigé.

---

### 10.2 GPO Stagiaires_Politique

#### 10.2.1 Préparation du fond d'écran

1. Sur un poste client (ex. `cdupont`) :
   - Ouvrir `\\SRV-AD\Public\wallpaper.jpg` avec Paint
   - Enregistrer sous JPEG : `C:\Temp\wallpaper_ok.jpg`
   - Vérifier l'affichage
2. Copier sur le serveur :
   - Créer/ouvrir `C:\Public`
   - Copier `wallpaper_ok.jpg`
3. Permissions :
   - Partage réseau : Tout le monde → Lecture
   - NTFS : Tout le monde → Lecture & exécution

> Nom du papaier peint : `\\SRV-AD\Public\wallpaper_ok.jpg`


#### 10.2.2 Création et liaison

- Ouvrir **GPMC**
- Clic droit sur l'OU **Stagiaires → Créer une GPO et la lier ici**
- Nommer : `Stagiaires_Politique`

#### 10.2.3 Paramétrages

**1. Limiter les icônes du Bureau**

Naviguer : `Configuration utilisateur → Modèles d'administration → Bureau`

| Action sur le Bureau | Statut | Icône visible / masquée |
|----------------------|--------|-------------------------|
| Supprimer Poste de travail du Bureau | Désactivé | Icône visible |
| Supprimer l'icône de la Corbeille du Bureau | Désactivé | Icône visible |
| Supprimer l'icône Mes documents du Bureau | Activé | Masquée |
| Cacher l'icône Internet Explorer sur le Bureau | Activé | Masquée |
| Cacher l'icône Emplacements réseau sur le Bureau | Activé | Masquée |

### Résultat attendu :

> Seuls Poste de travail et Corbeille sont visibles après reconnexion

**2. Appliquer le fond d'écran**

- Naviguer : `Configuration utilisateur → Modèles d'administration → Bureau → Bureau → Fond d'écran`
- Activer la stratégie
- Nom du papier peint : `\\SRV-AD\Public\wallpaper_ok.jpg`
- Style : *Remplir* ou *Étendre*

### Résultat attendu :

> Le fond d'écran s'applique automatiquement pour tous les utilisateurs

---

### 10.3 Vérifications

#### 10.3.1 Sur le serveur

- GPO liées aux OU
- Partages et permissions NTFS
- Délégation GPO : `Utilisateurs authentifiés → Lecture + Appliquer la stratégie`

#### 10.3.2 Sur le poste client

```cmd
gpupdate /force
gpresult /r
```

Vérifier :
- Bureau limité aux icônes autorisées
- Fond d'écran appliqué correctement

#### 10.3.3 Tests fonctionnels

| OU | Vérification |
|----|--------------|
| Comptabilité | Panneau de configuration bloqué, Documents redirigés |
| Stagiaires | Bureau limité, fond d'écran appliqué |

---

## 11. Dépannage rapide

- **GPO non appliquée** → vérifier `gpresult /r`, DNS client, liaison GPO, délégation
- **Redirection non créée** → vérifier droits NTFS & partage, option « sous-dossier par utilisateur », redémarrer session
- **Fond d'écran non appliqué** → vérifier chemin UNC accessible, droits lecture sur fichier :

```cmd
reg query "HKCU\Control Panel\Desktop" /v Wallpaper
```

---

## Conclusion

Ce TP vous a permis de mettre en place une infrastructure complète de gestion centralisée pour une PME, incluant :

- L'attribution automatique d'adresses IP via DHCP
- La résolution de noms via DNS
- L'organisation des utilisateurs et ressources dans Active Directory
- La sécurisation des accès aux partages réseau
- L'application de politiques de sécurité et de personnalisation via GPO

Cette infrastructure constitue la base d'une gestion moderne et sécurisée d'un parc informatique d'entreprise.
