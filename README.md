# Lab-2-GNS3 — Implémentation d'une Infrastructure 3-Tier Virtualisée
Statut du projet : 85% — Connectivité et services opérationnels. Phase finale de sécurisation et documentation des tests en cours.

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

><img width="1104" height="510" alt="image" src="https://github.com/user-attachments/assets/e3612f0a-fbec-43cf-b609-8e69168c7e3c" />



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

🔗 [Consulter les configs](/configs/01_base_setup.md) 🧪 [Consulter les tests](/test/01_base_setup.md)

---

### Phase 2 — Segmentation VLAN & Trunks

Segmentation VLAN, agrégation des liens de distribution (LACP) et sécurisation des ports d'accès

- **Segmentation VLAN** : Création de 7 VLANs (Users, Servers, RH, Guest, Management, Native, BlackHole) pour une isolation stricte des flux et une réduction des domaines de diffusion.
- **Trunking & Sécurité Native** : Standardisation du protocole **IEEE 802.1Q**. Migration du VLAN natif vers le **VLAN 100** pour prévenir le *VLAN Hopping* et filtrage explicite des VLANs autorisés (`allowed vlan`).
- **EtherChannel (LACP)** : Agrégation des liens physiques entre les switches de distribution (`SW-DIST-01/02`) en un **Port-Channel (Po1)** logique de 2 Gbps via le protocole standard LACP.
- **Optimisation Spanning-Tree (PVST+)** : 
    - Configuration pour garantir une topologie sans boucle.
    - **PortFast** : Activation sur les ports d'accès pour une connectivité immédiate des endpoints.
    - **BPDU Guard** : Protection contre l'injection de switches non autorisés (err-disable automatique).
- **VLAN BlackHole (999)** : Neutralisation de tous les ports inutilisés par assignation à un VLAN non routé et extinction administrative (`shutdown`).

🔗 [Consulter les configs](/configs/02_VLAN.md) 🧪 [Consulter les tests](/test/02_VLAN.md)

---

### Phase 3 — Haute Disponibilité L3 (HSRP)

Mise en place de la redondance de la passerelle par défaut pour assurer la continuité de service des utilisateurs.

- **Protocole** : HSRPv2 (protocole propriétaire Cisco) avec IP virtuelles (VIP) en `.1`. 
- **Élection** : **SW-DIST-01** configuré en "Active" (Priority 110 + Preempt).
- **Relais DHCP** : Activation du `ip helper-address` pointant vers le Windows Server (192.168.20.10).

🔗 [Consulter les configs](/configs/03_hsrp.md) 🧪 [Consulter les tests](/test/03_hsrp.md)

---

### Phase 4 — Routage Dynamique (OSPF)

Mise en place des liens routés et du protocole OSPF pour automatiser la diffusion des réseaux VLAN vers le cœur de réseau.

- **Architecture** : Liens Point-à-Point (/30) entre Distribution et Core via ports routés (`no switchport`).
- **OSPFv2** : Processus en Area 0 avec annonces des réseaux VLAN.
- **Sécurisation** : Utilisation de `passive-interface` sur les VLANs utilisateurs pour limiter l'exposition du protocole.

🔗 [Consulter les configs](/configs/04_ospf.md) 🧪 [Consulter les tests]()

---

### Phase 5 — Services IP & NAT

Objectif : Permettre aux clients internes d'accéder au Web, de résoudre des noms de domaine et d'obtenir automatiquement leurs configurations IP.

*   **NAT/PAT (Overload)** : Configuration du NAT sur le **R-EDGE-01** pour masquer les réseaux privés derrière l'IP publique dynamique fournie par le NAT Node GNS3.
    *   *ACL de translation* : Autorisation des réseaux `192.168.0.0/16`.
    *   *Interface Outside* : Utilisation de l'interface `GigabitEthernet0/2` connectée au nœud NAT.
*   **Routage vers Internet** : Mise en place d'une route statique par défaut (`0.0.0.0/0`) pointant vers l'IP du prochain saut (Next-Hop) du NAT Node.
*   **Services Windows Server (Core)** :
    *   **DHCP** : Scopes configurés pour chaque VLAN avec option DNS (192.168.20.10) et Gateway (VIP HSRP).
    *   **DNS** : Configuration des *Forwarders* (8.8.8.8, 1.1.1.1) sur le serveur Windows pour permettre la résolution de noms externes depuis les VPCS.
*   **Propagation de la route** : Utilisation de la commande `default-information originate` dans le processus OSPF pour annoncer la sortie Internet à l'ensemble des switches de distribution.


🔗 [Consulter les configs](configs/05_NAT&IP.md) 🧪 [Consulter les tests]()

---

### Phase 6 — ACLs & Sécurité

Objectif : Isoler totalement le trafic de gestion et verrouiller l'accès aux plans de contrôle (Control Plane) des équipements.

*(à rédiger)*
---

🔗 [Consulter les configs]() 🧪 [Consulter les tests]()

---

## 4. 🔍 Troubleshooting

*(à rédiger)*
---

## 5. 📄 Configs & Fichiers

*(à rédiger)*
---
