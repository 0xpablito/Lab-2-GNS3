#  Tests de Validation : Phase 3 — Haute Disponibilité L3 (HSRP)

Ce document valide la redondance de la passerelle par défaut (Gateway) et la capacité de basculement automatique entre les switches de distribution.

---

### 1. Vérification des rôles et des IPs Virtuelles (VIP)
**Objectif :** Confirmer que l'élection HSRP s'est déroulée correctement et que chaque VLAN possède sa passerelle virtuelle active sur le switch prioritaire.

*   **Commande exécutée :** `show standby brief`
*   **Points de contrôle :** 
    *   **Interface :** Vlan 10, 20, 30, 40, 99.
    *   **State :** `Active` sur SW-DIST-01 (Priority 110) / `Standby` sur SW-DIST-02.
    *   **Virtual IP :** Toutes les VIP doivent être en `.1`.
><img width="1330" height="196" alt="image" src="https://github.com/user-attachments/assets/10245d06-cf3b-4d56-bc54-ecd790a07dcb" />


### 2. Test de connectivité à la Passerelle Virtuelle
**Objectif :** S'assurer qu'un hôte final peut joindre sa passerelle virtuelle, validant ainsi la configuration de l'interface SVI.

*   **Action :** Ping depuis le **PC-01** (VLAN 10) vers sa gateway virtuelle.
*   **Commande :** `ping 192.168.10.1`
*   **Résultat attendu :** Réponse ICMP immédiate.
><img width="989" height="204" alt="image" src="https://github.com/user-attachments/assets/e4ff6b2e-3db7-4076-aad3-fcc30f5fcbe2" />


### 3. Mécanisme de Failover (Basculement)
**Objectif :** Vérifier que le switch de secours prend le relais instantanément en cas de défaillance du switch principal.

*   **Action :** Coupure administrative de l'interface VLAN sur **SW-DIST-01** (ou extinction du switch).
*   **Résultat attendu :** 
    *   Sur **SW-DIST-02**, le log console affiche : `%HSRP-6-STATECHANGE: Vlan10 Grp 10 state Standby -> Active`.
    *   Le flux réseau continue de passer sans interruption majeure.
><img width="1123" height="510" alt="image" src="https://github.com/user-attachments/assets/a33c7c5a-586e-43fe-8cc6-564199a99b08" />


### 4. Test de Préemption (Preempt)
**Objectif :** Valider que le switch principal reprend automatiquement son rôle d'Active dès son retour en ligne grâce à sa priorité supérieure.

*   **Action :** Rallumer / Réactiver les interfaces sur **SW-DIST-01**.
*   **Résultat attendu :** SW-DIST-01 redevient `Active` et SW-DIST-02 repasse en `Standby`.
><img width="1091" height="204" alt="image" src="https://github.com/user-attachments/assets/8a8a3057-a4bd-4c9e-9085-9295a8c96584" />


### 5. Validation du Relais DHCP (Helper Address)
**Objectif :** Confirmer que les switches de distribution sont prêts à relayer les requêtes DHCP vers le serveur Windows (VLAN 20).

*   **Commande exécutée :** `show ip interface vlan 10 | include Helper`
*   **Points de contrôle :** Présence de la ligne `Helper address is 192.168.20.10`.


---
