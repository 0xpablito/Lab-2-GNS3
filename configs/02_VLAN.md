# Phase 2 — Segmentation VLAN & Trunks

## 📖 Logique de Déploiement
L'objectif est d'isoler les flux par vlan afin de limiter la surface d'attaque. Nous utilisons le standard IEEE 802.1Q pour le tagging des trames sur les trunk.

## 🛠️ Configuration Types

### 1. Base de données VLAN (Sur tous les switches)
```
vlan 10
 name Users
vlan 20
 name Servers
vlan 30
 name RH
vlan 40
 name Guest
vlan 99
 name Management
vlan 100
 name Native_VLAN
vlan 999
 name BlackHole
```
### 2. Configuration des Trunks (Exemple SW-ACC-01)
Sécurisation via VLAN Natif 100 et filtrage des VLANs autorisés.
```
interface range Gi0/0 - 1
 description TRUNK_DIST
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 10,20,30,40,99,100
 switchport nonegotiate
```
### 3. Sécurisation des accès
```
interface range Gi1/0 - 3, Gi2/0 - 3
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
```
