# Phase 6 : Rapport de Validation de la Sécurité

L'objectif de cette phase de tests est de confirmer que les mécanismes de durcissement (Hardening) protègent efficacement l'infrastructure sans perturber les services critiques.

---

### 1. Validation du DHCP Snooping & Binding Table
*   **Objectif** : Vérifier que seuls les ports de confiance (Trust) peuvent répondre aux requêtes DHCP.
*   **Test effectué** : Demande d'IP via DHCP sur `PC1` (VLAN 10).
*   **Commande de vérification** : `show ip dhcp snooping binding` sur les switches d'accès.
*   **Résultat attendu** : L'adresse MAC du PC doit être associée à son IP et à son interface dans la table du switch. Les serveurs DHCP non autorisés doivent être bloqués.
><img width="1205" height="395" alt="image" src="https://github.com/user-attachments/assets/7b06e03d-63e8-404c-8aa9-f0d501fcb9c6" />


### 2. Validation du Dynamic ARP Inspection (DAI)
*   **Objectif** : Empêcher les attaques de type ARP Poisoning.
*   **Test effectué** : Tentative de Ping vers la Gateway Virtuelle (192.168.10.1) depuis `PC1`.
*   **Commande de vérification** : `show ip arp inspection statistics`
*   **Résultat attendu** : Le trafic ARP légitime est autorisé car présent dans la table DHCP Snooping. Les paquets ARP falsifiés sont jetés (dropped).
><img width="1692" height="338" alt="image" src="https://github.com/user-attachments/assets/69350edd-7e1a-4b3b-bcd2-fc5300054f8c" />


### 3. Validation du Port-Security
*   **Objectif** : Limiter l'accès physique aux ports du switch.
*   **Test effectué** : Vérification de l'apprentissage des adresses MAC "Sticky".
*   **Commande de vérification** : `show port-security interface <id>` et `show port-security address`.
*   **Résultat attendu** : Le switch doit afficher les adresses MAC enregistrées. Si une 3ème adresse MAC est branchée sur un port limité à 2, l'interface doit se bloquer (err-disabled) ou rejeter le trafic.
><img width="1187" height="365" alt="image" src="https://github.com/user-attachments/assets/89e51de0-388d-4e8e-be3f-9d780ad65307" />


### 4. Validation des ACLs & Accès Management (VLAN 99)
*   **Objectif** : Isoler les flux sensibles et restreindre l'administration.
*   **Tests effectués** :
    1. Tentative de SSH vers `SW-DIST-01` depuis `PC1` (VLAN 10).
    2. Tentative de SSH vers `SW-DIST-01` depuis `MGMT-PC` (VLAN 99).
*   **Résultat attendu** : 
    1. Le flux depuis le VLAN 10 doit être rejeté par l'ACL VTY (Connection refused).
    2. Le flux depuis le VLAN 99 doit être autorisé (Accès au prompt Password).
><img width="1212" height="583" alt="image" src="https://github.com/user-attachments/assets/d70b649b-69a5-49d8-bea6-2adb81f0eaa9" />



### 5. Connectivité Finale & Internet (NAT)
*   **Objectif** : Appliquer le principe du moindre privilège et vérifier que le cœur de données (serveur WS-DC-01) est protégé contre les accès non autorisés, notamment depuis le VLAN Guest.
*   **Tests effectués** : 
    1. Tentative de résolution DNS depuis `PC1` (VLAN 10) vers le serveur -> **Réussite**.
    2. Tentative de connexion (Ping ou scan de port) depuis le **VLAN Guest** (VLAN 40) vers le serveur -> **Échec (Bloqué)**.
*   **Commande de vérification** : `show ip access-lists` sur les switches de distribution pour vérifier l'incrémentation des compteurs (matches) de l'ACL.
*   **Résultat attendu** : La résolution DNS est fonctionnelle pour les utilisateurs internes. Tout flux non essentiel provenant du VLAN Guest est systématiquement rejeté par l'ACL restrictive, comme en témoignent les "matches" sur les lignes de deny.
><img width="1158" height="428" alt="image" src="https://github.com/user-attachments/assets/08e45c76-4624-47cf-87ec-566d11ca9f3f" />

