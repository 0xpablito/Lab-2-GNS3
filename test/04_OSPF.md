#  Tests de Validation : Phase 4 — Routage Dynamique (OSPF)

Ce document valide la formation des adjacences OSPF, la convergence du réseau et la propagation des routes entre les switches de distribution et le routeur de bordure (R-CORE).

---

### 1. État des voisins (OSPF Neighbors)
**Objectif :** Vérifier que les équipements ont bien formé des relations de "neighbors" et échangé leurs bases de données (LSDB).

*   **Commande exécutée :** `show ip ospf neighbor`
*   **Résultat attendu :** 
    *   L'état doit être **`FULL`** sur toutes les interfaces participantes.
    *   Sur les liens Multi-accès (Ethernet), un DR (Designated Router) et un BDR doivent être élus.
><img width="1139" height="254" alt="image" src="https://github.com/user-attachments/assets/3bb2c6ed-af61-46cb-aec2-27f1c428b880" />

### 2. Analyse de la Table de Routage OSPF
**Objectif :** Confirmer que les routes vers les différents VLANs et réseaux d'interconnexion sont correctement apprises via le protocole.

*   **Commande exécutée :** `show ip route ospf`
*   **Points de contrôle :** 
    *   Présence des routes marquées par un **`O`** (OSPF intra-area).
    *   Vérification des chemins vers les VLANs de production (10, 20, 30, 40) depuis R-CORE.
><img width="1096" height="378" alt="image" src="https://github.com/user-attachments/assets/1fac8c81-6a96-40cb-8990-265138a5a53b" />


### 3. Propagation de la Route par Défaut (Internet)
**Objectif :** Vérifier que le routeur R-CORE injecte bien la route vers l'extérieur (`0.0.0.0/0`) vers le reste du réseau.

*   **Commande exécutée :** `show ip route 0.0.0.0` sur un switch de distribution.
*   **Résultat attendu :** 
    *   La route doit apparaître comme **`O*E2`** (OSPF external type 2).
    *   La passerelle (Next-hop) doit être l'IP de l'interface de R-CORE.
><img width="1156" height="286" alt="image" src="https://github.com/user-attachments/assets/fbe58b30-c298-4e92-8d75-8e5c0d76b622" />


### 4. Connectivité au "Loopback" de Management
**Objectif :** Valider la portée du routage en joignant l'interface virtuelle de management du routeur depuis n'importe quel point du réseau.

*   **Action :** Ping depuis le PC MGMT vers la Loopback de R-CORE.
*   **Commande :** `ping 1.1.1.1`
*   **Résultat attendu :** Taux de réussite de 100%.
><img width="844" height="210" alt="image" src="https://github.com/user-attachments/assets/97b80f96-ded4-4f85-9926-8e139ca7f084" />


---
