# Lab-2-GNS3 — Implémentation d'une Infrastructure 3-Tier Virtualisée

## 🎯 Objectifs et Démarche

Ce lab réseau a été conçu et déployé dans le cadre de ma montée en compétences en infrastructure réseau d'entreprise. Il me permet de me familiariser avec GNS3 comme environnement de simulation et VMware Workstation comme hyperviseur hôte, afin de reproduire une architecture réseau proche d'un environnement de production réel.

L'objectif est de franchir un cap supplémentaire en travaillant avec des images IOS réelles sur GNS3, afin de se confronter à des comportements et des contraintes absents des simulateurs pédagogiques.

🛠️ Compétences techniques validées :

* **L2 (Commutation)** : VLANs, Trunks (802.1Q), EtherChannel (LACP), Spanning-Tree (PVST+, PortFast, BPDU Guard).
* **L3 (Routage)** : Routage Inter-VLAN (SVI), OSPF, HSRP (redondance de passerelle). Services & Sécurité : DHCP, NAT/PAT (Overload), ACLs étendues, SSHv2, isolation VLAN Guest/RH.
* **Services d'annuaire** : Active Directory Domain Services (AD DS), DNS intégré, domaine Atlas.local, gestion via PowerShell (Windows Server Core)

💡 [à rédiger]

---

## 1. 🗺️ Vue d'ensemble

*à ajouter*

Équipement | Rôle & Fonctionnalités Clés | Image IOS / Ressource |
| :--- | :--- | :--- |
| **R-EDGE-01** | **Routeur Core / Bordure** : Routage OSPF (Area 0), NAT/PAT, DHCP Server, ACLs | `Cisco vIOS 15.9(3)M6` |
| **SW-DIST-01** | **Distribution L3** : Gateway HSRP (Active), Routage Inter-VLAN, EtherChannel | `Cisco vIOSL2` |
| **SW-DIST-02** | **Distribution L3** : Gateway HSRP (Standby), Redondance L3, EtherChannel | `Cisco vIOSL2` |
| **SW-ACC-01** | **Accès L2** : Segmentation VLANs, Port-Security, STP Optimization | `Cisco vIOSL2` |
| **SW-ACC-02** | **Accès L2** : Segmentation VLANs, DHCP Snooping, DAI | `Cisco vIOSL2` |
| **SW-ACC-03** | **Accès L2** : Connectivité Endpoints, Trunking 802.1Q | `Cisco vIOSL2` |
| **NAT Node** | **Sortie Internet** : Pontage vers l'interface réseau physique (GNS3 Cloud) | `Built-in Node` |
| **PC-01 à 05** | **Postes Clients** : Validation connectivité, Tests DHCP, ACL & ICMP | `VPCS` |	
| **WS-DC-01** | **	Contrôleur de Domaine** : AD DS, DNS intégré, domaine Atlas.local, gestion PowerShell (Core) | `Windows Server 2022 Core` |

---


## 2. 🛠️ Implémentation technique

### Phase 1 — Configuration de base & Sécurité

Déploiement d'une configuration uniformisée sur l'ensemble des équipements et verrouillage des accès avant tout déploiement de service réseau.

- **Accès distant** : SSHv2 exclusif avec clé RSA 2048 bits et désactivation de Telnet
- **Identité & Accès** : Compte admin local privilege 15, enable secret haché
- **Console & VTY** : Authentification locale, timeout 10 min, logging synchronous
- **Sécurité DNS** : Désactivation du DNS lookup pour éviter les délais en CLI
- **Bannière** : Message d'avertissement légal sur toutes les lignes d'accès
- **Management** : SVI VLAN 99 dédiée sur chaque équipement (192.168.99.0/24)

🔗 [Consulter les configs](/configs/01_base_setup.md) 🧪 [Consulter les tests]()

---

### Phase 2 — Segmentation VLAN & Trunks

Cette phase pose les fondations de l'isolation réseau et de la sécurité des flux.

- **Segmentation** : Création de 7 VLANs distincts pour séparer les flux utilisateurs (10), serveurs (20), RH (30), invités (40), management (99), natif (100) et sécurité (999).
- **Trunking** : Standardisation du protocole IEEE 802.1Q sur tous les liens d'interconnexion.
- **Sécurité Native** : Modification du VLAN natif par défaut (VLAN 1) vers le VLAN 100 pour prévenir les attaques de type VLAN Hopping.
- **VLAN BlackHole** : Assignation systématique des ports d'accès non utilisés au VLAN 999 et extinction administrative (`shutdown`).


🔗 [Consulter les configs]() 🧪 [Consulter les tests]()

---

### Phase 3 — Spanning-Tree & EtherChannel

Optimisation de la bande passante inter-distribution et sécurisation de la topologie logique.

- **EtherChannel (LACP)** : Agrégation des liens physiques (Gi0/1 et Gi1/1) entre SW-DIST-01 et SW-DIST-02 en un Port-Channel logique (Po1). Utilisation du protocole standard LACP pour une interopérabilité maximale et un débit de 2 Gbps.
- **Spanning-Tree (PVST+)** : Configuration pour assurer une topologie sans boucle.
- **STP Edge Ports** : Activation de `Spanning-Tree PortFast` sur les interfaces d'accès (PC et Serveurs) pour une mise en service immédiate des ports (passage direct en Forwarding).
- **BPDU Guard** : Protection des ports d'accès contre le raccordement de switches non autorisés (err-disable automatique en cas de détection de BPDU).

🔗 [Consulter les configs]() 🧪 [Consulter les tests]()

---

### Phase 4 — Routage Inter-VLAN & OSPF

Mise en place de la haute disponibilité de la passerelle par défaut pour assurer la continuité de service.

- **Protocole** : Utilisation de HSRP (Hot Standby Router Protocol) version 2.
- **Architecture** : Création d'une IP virtuelle (VIP) se terminant par `.1` dans chaque VLAN (ex: 192.168.10.1).
- **Élection** : 
    - **SW-DIST-01 (Active)** : Priorité forcée à 110 avec mécanisme de `preempt` pour reprendre son rôle après redémarrage.
    - **SW-DIST-02 (Standby)** : Priorité par défaut (100) assurant la reprise instantanée en cas de défaillance du Master.
- **Avantage** : Permet aux clients DHCP d'utiliser une passerelle unique et résiliente, indépendamment de l'état d'un switch physique spécifique.

🔗 [Consulter les configs]() 🧪 [Consulter les tests]()

---

### Phase 5 — HSRP (Redondance de passerelle)

*(à rédiger)*

🔗 [Consulter les configs]() 🧪 [Consulter les tests]()

---

### Phase 6 — Services IP & NAT

*(à rédiger)*

🔗 [Consulter les configs]() 🧪 [Consulter les tests]()

---

### Phase 7 — ACLs & Sécurité

*(à rédiger)*

🔗 [Consulter les configs]() 🧪 [Consulter les tests]()

---

## 4. 🔍 Troubleshooting


---

## 5. 📄 Configs & Fichiers

