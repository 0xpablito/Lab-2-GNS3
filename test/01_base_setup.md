# Tests de Validation : Phase 1 — Configuration de Base & Sécurité

Ce document regroupe les tests techniques validant la configuration de sécurité initiale et les paramètres d'administration des équipements (Switches et Routeurs).

---

### 1. Sécurité du système et Chiffrement
**Objectif :** Vérifier que les mots de passe ne sont pas stockés en clair et que l'accès au mode privilégié est strictement protégé.

*   **Commande exécutée :** `show running-config`
*   **Points de contrôle :** 
    *   Présence du `enable secret` (hachage de type 5 ou 9).
    *   Chiffrement global des mots de passe via `service password-encryption`.
    *   Vérification que l'utilisateur `admin` possède bien le `privilege 15`.
><img width="1015" height="277" alt="image" src="https://github.com/user-attachments/assets/12232123-020c-4b5c-a67d-0b862eafe10c" />


### 2. Administration à distance (SSHv2)
**Objectif :** S'assurer que le protocole de gestion est sécurisé et que les clés cryptographiques sont conformes aux standards actuels.

*   **Commande exécutée :** `show ip ssh`
*   **Résultat attendu :** 
    *   `SSH Enabled - version 2.0`.
    *   Clé RSA confirmée à **2048 bits**.
    *   Paramètres de timeout et de tentatives d'authentification actifs.
><img width="1027" height="561" alt="image" src="https://github.com/user-attachments/assets/4f3d768e-8bca-49fc-888d-53c2563fe442" />

### 3. Test de connexion "End-to-End" via SSH
**Objectif :** Valider l'accès réel depuis le segment de management et confirmer l'affichage des avertissements légaux.

*   **Action :** Connexion depuis le terminal du **PC-MGMT** (192.168.99.50) vers un équipement.
*   **Commande :** `ssh -l admin 192.168.99.11` (Exemple vers SW-ACC-01)
*   **Points de contrôle :** 
    *   Affichage de la **bannière MOTD** : `Acces restreint - atlas.local`.
    *   Demande d'authentification sur la base locale.
    *   Accès réussi au prompt privilège.
><img width="1701" height="590" alt="image" src="https://github.com/user-attachments/assets/82651b95-36d6-4238-b1f1-55e0f1d33619" />

### 4. Blocage des protocoles non sécurisés
**Objectif :** Apporter la preuve que le protocole Telnet est désactivé au profit du flux chiffré SSH.

*   **Commande :** `telnet 192.168.99.11`
*   **Résultat attendu :** 
    *   `Connection closed by foreign host` ou `Connection refused`.
    *   Preuve que seul `transport input ssh` est configuré sur les lignes VTY.
><img width="1550" height="103" alt="image" src="https://github.com/user-attachments/assets/6e4d1e1a-4198-425a-b910-2ef2d0ab5170" />

### 5. Ergonomie et Confort de l'Administrateur
**Objectif :** Vérifier l'activation des paramètres empêchant le blocage de la console lors de l'administration.

*   **Test Logging Sync :** Générer un log (ex: `no shutdown` sur une interface). Le log ne doit pas couper la saisie de la commande en cours.
*   **Test No Domain-Lookup :** Saisir une commande erronée au mode privilège (ex: `azerty`). 
*   **Résultat attendu :** L'équipement doit rendre la main immédiatement sans tenter de résolution DNS ("Translating...").

---
