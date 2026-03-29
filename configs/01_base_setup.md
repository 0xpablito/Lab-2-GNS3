# Configuration de Base et Sécurité
**Cible :** `SW-ACC-01`, `SW-ACC-02`, `SW-ACC-03`,`SW-DIST-01`,`SW-DIST-02`,`R-CORE`


### 📋 Script de configuration

### Identification et DNS
```
hostname SW-ACC-1            ! Adapter selon l'équipement
no ip domain-lookup          ! Évite les blocages en cas d'erreur de frappe
```
### Sécurisation des accès 
```
enable secret es3QPih5MXP3AES   ! Mot de passe du mode privilégié (haché)
service password-encryption     ! Chiffre les mots de passe visibles dans la config
```
### Création d'un compte administrateur
```
username admin privilege 15 secret es3QPih5MXP3AES
```
### Paramètres SSH 
```
ip domain-name atlas.local       ! Définit le suffixe DNS nécessaire à la génération de la clé
crypto key generate rsa modulus 2048  ! Génère la clé de chiffrement RSA 2048 bits
ip ssh version 2                 ! Force l'usage du protocole SSH version 2 (plus sécurisé)
```   
### Configuration de la console (Accès physique) 
```
line console 0
 logging synchronous             ! Évite que les logs coupent la saisie
 login local                     ! Utilise le compte admin créé plus haut
 exec-timeout 10 0               ! Déconnexion après 10 min d'inactivité
```
### Configuration VTY (Accès distant)
```
line vty 0 4
 login local                     ! Authentification via la base de données locale
 transport input ssh              ! Autorise uniquement le SSH (Telnet bloqué)
 exec-timeout 10 0               ! Déconnexion après 10 min d'inactivité
```
### Synchronisation Horloge (NTP)
```
ntp server 192.168.20.10         ! Synchronise l'horloge sur le serveur Windows
```
### Message d'avertissement 
```
banner motd ^
------------------------------------------------
   Acces restreint - atlas.local
   Toute connexion non autorisee est interdite
------------------------------------------------
^
