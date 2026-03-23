# Phase 2 — Infrastructure de Commutation (L2)

**Cible :** `SW-ACC-01/02/03` & `SW-DIST-01/02`

## 📋 Configuration

### 1. Base de données VLAN
```
vlan 10  | name Users
vlan 20  | name Servers
vlan 30  | name RH
vlan 40  | name Guest
vlan 99  | name Management
vlan 100 | name Native_VLAN
vlan 999 | name BlackHole
```
### 2. Agrégation de liens (EtherChannel LACP)
Fusion des liens Gi0/1 et Gi1/1 en un Port-Channel (Po1) de 2 Gbps pour assurer la redondance et la répartition de charge.
```
interface range Gi0/1, Gi1/1
channel-group 1 mode active
interface Port-channel 1
 description LINK_DIST
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 10,20,30,40,99,100
```
### 3. Trunks
Configuration sur tous les liens montants et descendants.
```
interface range   (Adapter selon le switch)
 description  (Adapter selon le switch)
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 10,20,30,40,99,100
 switchport nonegotiate
```
### 4. Accès Terminaux & Sécurité (Edge Ports)
Configuration d'un port utilisateur (ex: PC-01) avec protection contre les boucles.
```
interface Gi3/2
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast        ! Passage immédiat en Forwarding
 spanning-tree bpduguard enable ! Protection contre switches sauvages
 no shutdown
```
### 5. Sécurisation des ports inutilisés
Tous les ports non utilisés sont envoyés vers un VLAN non routé et étient.
```
interface range Gi1/0 - 3, Gi2/0 - 3
 switchport mode access
 switchport access vlan 999
 shutdown
