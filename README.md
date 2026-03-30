# Lab-2-GNS3 — Implémentation d'une Infrastructure 3-Tier Virtualisée

## 🎯 Objectifs et Démarche

Ce lab réseau a été conçu et déployé dans le cadre de ma montée en compétences en infrastructure réseau d'entreprise. Il me permet de me familiariser avec GNS3 comme environnement de simulation et VMware Workstation comme hyperviseur hôte, afin de reproduire une architecture réseau proche d'un environnement de production réel.

L'objectif est de franchir un cap supplémentaire en travaillant avec des images IOS réelles sur GNS3, afin de se confronter à des comportements et des contraintes absents des simulateurs pédagogiques.

🛠️ Compétences techniques validées :

* **L2 (Commutation)** : VLANs, Trunks (802.1Q), EtherChannel (LACP), Spanning-Tree (PVST+, PortFast, BPDU Guard).
* **L3 (Routage)** : Routage Inter-VLAN (SVI), OSPF, HSRP (redondance de passerelle). Services & Sécurité : DHCP, NAT/PAT (Overload), ACLs étendues, SSHv2, NTP.
* **Services d'annuaire** : Active Directory Domain Services (AD DS), DNS intégré, domaine Atlas.local, gestion via PowerShell (Windows Server Core)

💡 [à rédiger]

---

## 1. 🗺️ Vue d'ensemble

><img width="1386" height="597" alt="image" src="https://github.com/user-attachments/assets/51cb8fd8-e0be-430f-91fe-dbf8f2f164cf" />





Équipement | Rôle & Fonctionnalités Clés | Image IOS |
| :--- | :--- | :--- |
| **R-EDGE-01** | **Routeur Core / Bordure** : Routage OSPF (Area 0), NAT/PAT, ACLs | `Cisco vIOS 15.9(3)M6` |
| **SW-DIST-01** | **Distribution L3** : Gateway HSRP (Active), Routage Inter-VLAN, EtherChannel | `Cisco vIOSL2` |
| **SW-DIST-02** | **Distribution L3** : Gateway HSRP (Standby), Redondance L3, EtherChannel | `Cisco vIOSL2` |
| **SW-ACC-01** | **Accès L2** : Segmentation VLANs, Port-Security, STP Optimization | `Cisco vIOSL2` |
| **SW-ACC-02** | **Accès L2** : Segmentation VLANs, DHCP Snooping, DAI | `Cisco vIOSL2` |
| **SW-ACC-03** | **Accès L2** : Connectivité Endpoints, Trunking 802.1Q | `Cisco vIOSL2` |
| **NAT Node** | **Sortie Internet** : Pontage vers l'interface réseau physique (GNS3 Cloud) | `Built-in Node` |
| **PC-01 à 06** | **Postes Clients** : Validation connectivité, Tests DHCP, ACL & ICMP | `VPCS` |	
| **WS-DC-01** | **Contrôleur de Domaine** : AD DS, DNS intégré, domaine Atlas.local, gestion PowerShell (Core) | `Windows Server 2022 Core` |

---


## 2. 🛠️ Implémentation technique

### Phase 1 — Configuration de base & Sécurité

Déploiement d'une configuration uniformisée sur l'ensemble des équipements et verrouillage des accès avant tout déploiement de service réseau.

- **Accès distant** : SSHv2 exclusif avec clé RSA 2048 bits et désactivation de Telnet
- **Identité & Accès** : Compte admin local privilege 15, enable secret haché
- **Console & VTY** : Authentification locale, timeout 10 min, logging synchronous
- **Sécurité DNS** : Désactivation du DNS lookup pour éviter les délais en CLI
- **Bannière** : Message d'avertissement légal sur toutes les lignes d'accès
- **Horodatage (NTP)** : Synchronisation temporelle sur le serveur 192.168.20.10 pour la cohérence des logs.
- **Management** : SVI VLAN 99 dédiée sur chaque équipement (192.168.99.0/24)

🔗 [Consulter les configs](/configs/01_base_setup.md) 🧪 [Consulter les tests](/test/01_base_setup.md)

---

### Phase 2 — Segmentation VLAN & Trunks

Segmentation VLAN, agrégation des liens de distribution (LACP) et sécurisation des ports d'accès

- **Segmentation VLAN** : Création de 7 VLANs (Users, Servers, RH, Guest, Management, Native, BlackHole) pour une isolation stricte des flux et une réduction des domaines de diffusion.
- **Trunking & Sécurité Native** : Standardisation du protocole **IEEE 802.1Q**. Migration du VLAN natif vers le **VLAN 100** pour prévenir le *VLAN Hopping* et filtrage explicite des VLANs autorisés (`allowed vlan`).
- **EtherChannel (LACP)** : Agrégation des liens physiques entre les switches de distribution (`SW-DIST-01/02`) en un **Port-Channel (Po1)** logique de 2 Gbps via le protocole standard LACP.
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


🔗 [Consulter les configs](configs/05_NAT&IP.md) 🧪 [Consulter les tests](test/05_IP&NAT.md)

---

### Phase 6 — ACLs & Sécurité

Objectif : Sécuriser les plans de contrôle et de données en limitant les flux au strict nécessaire (Principe du moindre privilège).

*   **Sécurisation VTY (Management)** : Limitation des accès SSH exclusivement à l'adresse IP de l'administrateur (`192.168.99.50`) via une ACL standard appliquée sur les lignes VTY.
*   **Isolation Guest (VLAN 40)** : Mise en place d'une ACL étendue interdisant tout accès aux segments internes tout en autorisant les services essentiels (DNS/DHCP) et l'accès Internet (NAT).
*   **Hardening Serveur (VLAN 20)** : Filtrage des flux entrants vers le Contrôleur de Domaine :
    *   Accès complet pour le VLAN Management.
    *   Services métiers (SMB, LDAP) limités aux VLANs Users et RH.
    *   Autorisation du trafic "Established" pour permettre les mises à jour Windows Update (retour de flux).
*   **Sécurité Couche 2** :
    *   **DHCP Snooping & DAI** : Protection contre les serveurs DHCP pirates et l'empoisonnement ARP (ARP Spoofing) par validation des requêtes sur les ports *untrusted*.
    *   **Port-Security** : Limitation à 2 adresses MAC par port avec apprentissage `sticky` pour prévenir les attaques de type MAC Flooding.

🔗 [Consulter les configs](configs/06_SECURITY.md) 🧪 [Consulter les tests]()

---

## 4. 🔍 Troubleshooting
Si la résolution de problèmes rencontrés lors de la configuration peut paraître longue et fastidieuse, c'est précisément dans ces phases de dépannage que l'assimilation des concepts techniques se confirme. C'est en confrontant la théorie à la réalité du terrain que l'on passe de l'application de commandes à la compréhension réelle

### 1. Échec de l'obtention d'adresse IP (DHCP Snooping)
* **Symptôme** : Les postes clients (VPCS) affichent un échec lors de la requête DHCP (`Can't find DHCP server` ou boucle de `DDD`), alors que le service fonctionnait avant l'activation de la sécurité.
* **Diagnostic** : Par défaut, le **DHCP Snooping** considère tous les ports comme "untrusted" (non sûrs). Le commutateur d'accès bloquait les paquets `DHCPOFFER` et `DHCPACK` renvoyés par le serveur Windows car ils arrivaient sur le ports d'accès du switch SW-ACC-02 non configurés explicitement en mode "trust".
* **Résolution** : 
  * Configuration du port de confiance sur ls switche d'accès : `ip dhcp snooping trust` sur le port relié au serveur Windows.
  * Cela autorise le switch à laisser passer les réponses provenant du serveur légitime vers les clients.

### 2. Isolation réseau et blocage ARP (DAI)
* **Symptôme** : Perte totale de connectivité (Ping impossible) vers la Gateway, même pour les PC ayant déjà une adresse IP correcte ou pour le VLAN de Management.
* **Diagnostic** : Le **Dynamic ARP Inspection (DAI)** vérifie la validité des paquets ARP en consultant la *DHCP Binding Table*. Deux causes ont provoqué le blocage :
  * 1. **PC déjà connectés** : Les PC ayant obtenu une IP avant l'activation du Snooping n'étaient pas encore inscrits dans la table ; le DAI a donc rejeté leur trafic ARP.
  * 2. **Adressage Statique** : Le VLAN 99 (Management) et le VLAN 20 (Serveur) utilisent des IPs statiques. Étant absents de la table de liaison DHCP, ils ont été systématiquement bloqués par le DAI.
* **Résolution** :
  * **Pour les clients** : Réinitialisation de l'adressage (`ip dhcp`) pour forcer une nouvelle adresse et l'ajouter dans la table du switch.
  * **Pour le Management/Serveurs** : Exclusion de ces VLANs de l'inspection ARP via la commande `no ip arp inspection vlan 20,99`.

*(à rédiger)*
---

## 5. 📄 Configs & Fichiers

*(à rédiger)*
---
