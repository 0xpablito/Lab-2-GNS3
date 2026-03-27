#  Tests de Validation : Phase 2 — Infrastructure de Commutation (L2)

Ce document valide la segmentation réseau (VLANs), l'agrégation de liens (EtherChannel) et la sécurisation du Spanning-Tree.

---

### 1. État de la base de données VLAN
**Objectif :** Confirmer que la segmentation est cohérente sur l'ensemble de la topologie.

*   **Commande exécutée :** `show vlan brief`
*   **Points de contrôle :** 
    *   Présence des VLANs 10, 20, 30, 40, 99, 100 et 999.
    *   Nommage conforme (Users, Servers, RH, Guest, MGMT, Native, BlackHole).
><img width="1037" height="390" alt="image" src="https://github.com/user-attachments/assets/2e08a9a6-8d4b-4096-b511-91331f89b6fd" />


### 2. Validation de l'EtherChannel (LACP)
**Objectif :** Vérifier que les deux liens physiques entre les switches de Distribution sont agrégés en un seul lien logique de 2 Gbps.

*   **Commande exécutée :** `show etherchannel summary`
*   **Résultat attendu :** 
    *   Groupe `1`, Port-channel `Po1`.
    *   Protocol : `LACP`.
    *   Status : `(SU)` (S = Layer2, U = In use).
    *   Ports participants : `(P)` (Bundled in port-channel).
><img width="828" height="140" alt="image" src="https://github.com/user-attachments/assets/01447c68-e44b-4019-8b74-b2ee1420c7d4" />


### 3. État des Trunks (802.1Q)
**Objectif :** S'assurer que le trafic inter-VLAN circule correctement et que le VLAN Natif est sécurisé.

*   **Commande exécutée :** `show interfaces trunk`
*   **Points de contrôle :** 
    *   Mode : `on` (ou `desirable/active` selon config).
    *   Native VLAN : `100` (et non le VLAN 1 par défaut).
    *   VLANs allowed : Liste restreinte (10, 20, 30, 40, 99, 100).
><img width="1665" height="542" alt="image" src="https://github.com/user-attachments/assets/5b0508da-a3ef-442d-b5b5-2d0e21fc4ccf" />


### 4. Sécurisation Spanning-Tree (PortFast & BPDU Guard)
**Objectif :** Garantir la stabilité de la topologie et la protection contre les boucles ou switches non autorisés.

*   **Commande exécutée :** `show spanning-tree interface [Nom_Interface] detail`
*   **Points de contrôle :** 
    *   Vérifier que le port utilisateur est en mode `Edge` (PortFast).
    *   Vérifier que `BPDU guard is enabled` est présent.
><img width="1779" height="338" alt="image" src="https://github.com/user-attachments/assets/d8715e99-8e42-4893-9b5b-3828c53cb7d3" />


### 5. Isolation des ports inutilisés (BlackHole)
**Objectif :** Preuve de l'application de la politique de sécurité "Zéro Confiance" sur les ports vacants.

*   **Commande exécutée :** `show ip interface brief` ou `show interface status`
*   **Résultat attendu :** 
    *   Ports inutilisés affichés en `administratively down`.
    *   Affectation au VLAN 999.
><img width="1556" height="567" alt="image" src="https://github.com/user-attachments/assets/bbf3d49e-01ed-4c21-a329-b427b7ce651e" />


---
