#  Tests de Validation : Phase 5 — Services IP & NAT

Ce document valide la sortie Internet, la distribution automatique des adresses IP via le serveur Windows (Relais DHCP) et la résolution de noms (DNS).

---

### 1. Validation du NAT (PAT Overload)
**Objectif :** Vérifier que le routeur R-CORE translate les adresses privées (RFC 1918) en l'adresse publique de l'interface Gi0/2.

*   **Action :** Lancer un ping vers `8.8.8.8` depuis le **PC6** (ou un client du VLAN 10).
*   **Commande sur R-CORE :** `show ip nat translations`
*   **Résultat attendu :** 
    *   Apparition d'une ligne `icmp` dans la table.
    *   **Inside Local :** IP du PC (ex: 192.168.99.50).
    *   **Inside Global :** IP publique de Gi0/2 (ex: 192.168.122.103).
><img width="1133" height="415" alt="image" src="https://github.com/user-attachments/assets/61567b62-163f-450c-8fb1-424bd2d16787" />


---

### 2. Attribution DHCP via Windows Server (Relais DHCP)
**Objectif :** Confirmer que la chaîne "Client -> Relay (DIST) -> Serveur (WS-DC-01)" fonctionne.

*   **Action :** Demander une IP en DHCP sur un poste de travail (VLAN 10, 30 ou 40).
*   **Commande (VPCS) :** `ip dhcp`
*   **Résultat attendu :** 
    *   Réception d'une IP dans le pool défini (ex: 192.168.10.100).
    *   Attribution automatique de la Gateway virtuelle (.1) et du DNS (.10).
><img width="1149" height="437" alt="image" src="https://github.com/user-attachments/assets/07e4114e-5a97-4d9f-98ed-e75a762f025d" />


---

### 3. Résolution de noms DNS (Interne & Externe)
**Objectif :** Valider que le serveur Windows résout les requêtes pour le domaine `atlas.local` et redirige le reste vers Internet.

*   **Actions :** 
    1.  Résolution interne : `nslookup ws-dc-01.atlas.local`
    2.  Résolution externe : `ping google.com`
*   **Résultat attendu :** 
    *   Le serveur répond avec son IP (192.168.20.10) pour les requêtes internes.
    *   Les forwarders DNS (8.8.8.8) permettent de joindre les domaines externes.
><img width="842" height="406" alt="image" src="https://github.com/user-attachments/assets/8c656497-8f82-42e9-adec-2fdea0604533" />


---

### 4. État des Scopes DHCP (Windows Server)
**Objectif :** Confirmer via PowerShell que les étendues sont actives et distribuent des baux.

*   **Commande (PowerShell - Admin) :** `Get-DhcpServerv4Scope`
*   **Points de contrôle :** 
    *   État : `Active`.
    *   Vérification des baux en cours : `Get-DhcpServerv4Lease -ScopeId 192.168.10.0`.
><img width="881" height="131" alt="image" src="https://github.com/user-attachments/assets/b33aa6df-a9d6-4d6a-abe6-05270a34fabd" />


---

### 5. Propagation de la Route par Défaut (OSPF)
**Objectif :** Vérifier que tous les équipements connaissent la route vers Internet.

*   **Commande sur SW-DIST-01 :** `show ip route 0.0.0.0`
*   **Résultat attendu :** 
    *   La route doit apparaître comme `O*E2 0.0.0.0/0`.
    *   Le "Next Hop" doit être l'IP de R-CORE (10.0.0.1nsl).
><img width="1126" height="283" alt="image" src="https://github.com/user-attachments/assets/3da0fff9-9558-4259-b7f5-efc2ee0f20c2" />
