# Lab-2-GNS3 — Implémentation d'une Infrastructure 3-Tier Virtualisée

## 🎯 Objectifs et Démarche

Ce lab réseau a été conçu et déployé dans le cadre de ma montée en compétences en infrastructure réseau d'entreprise. Il me permet de me familiariser avec GNS3 comme environnement de simulation et VMware Workstation comme hyperviseur hôte, afin de reproduire une architecture réseau proche d'un environnement de production réel.

L'objectif est de franchir un cap supplémentaire en travaillant avec des images IOS réelles sur GNS3, afin de se confronter à des comportements et des contraintes absents des simulateurs pédagogiques.

🛠️ Compétences techniques validées :

* **L2 (Commutation)** : VLANs, Trunks (802.1Q), EtherChannel (LACP), Spanning-Tree (PVST+, PortFast, BPDU Guard).
* **L3 (Routage)** : Routage Inter-VLAN (SVI), OSPF, HSRP (redondance de passerelle). Services & Sécurité : DHCP, NAT/PAT (Overload), ACLs étendues, SSHv2, isolation VLAN Guest/RH.

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

---


## 2. 🛠️ Implémentation technique

### Phase 1 — Configuration de base & Sécurité

Déploiement d'une configuration uniformisée sur l'ensemble des équipements
et verrouillage des accès avant tout déploiement de service réseau.

- **Accès distant** : SSHv2 exclusif avec clé RSA 2048 bits et désactivation de Telnet
- **Identité & Accès** : Compte admin local privilege 15, enable secret haché
- **Console & VTY** : Authentification locale, timeout 10 min, logging synchronous
- **Sécurité DNS** : Désactivation du DNS lookup pour éviter les délais en CLI
- **Bannière** : Message d'avertissement légal sur toutes les lignes d'accès
- **Management** : SVI VLAN 99 dédiée sur chaque équipement (192.168.99.0/24)

🔗 [Consulter les configs](/configs/01_base_setup.md) 🧪 [Consulter les tests]()

---

### Phase 2 — Segmentation VLAN & Trunks

*(à rédiger)*

🔗 [Consulter les configs]() 🧪 [Consulter les tests]()

---

### Phase 3 — Spanning-Tree & EtherChannel

*(à rédiger)*

🔗 [Consulter les configs]() 🧪 [Consulter les tests]()

---

### Phase 4 — Routage Inter-VLAN & OSPF

*(à rédiger)*

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

