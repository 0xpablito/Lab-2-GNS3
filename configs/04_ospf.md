# Phase 4 — Routage Dynamique OSPF 

**Cible :** `R-EDGE-01` & `SW-DIST-01/02`

## 📋 Configuration

### 1. Configuration du Routeur R-EDGE-01
```
interface Gi0/0
 description LINK_TO_SW-DIST-01
 ip address 10.0.0.1 255.255.255.252
 no shutdown

interface Gi0/1
 description LINK_TO_SW-DIST-02
 ip address 10.0.0.5 255.255.255.252
 no shutdown
router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.0.4 0.0.0.3 area 0
 network 1.1.1.1 0.0.0.0 area 0
 default-information originate
```
### 2. Configuration SW-DIST-01
```
ip routing
interface Gi0/0
 description LINK_R-CORE
 no switchport
 ip address 10.0.0.2 255.255.255.252
 no shutdown
router ospf 1
 router-id 2.2.2.2
 network 10.0.0.0 0.0.0.3 area 0  ! Annonce du lien d'interconnexion
 network 192.168.0.0 0.0.255.255 area 0  ! Annonce de tous les VLANs de production (VLAN 10, 20, 30, 40, 99)
 passive-interface default
 no passive-interface Gi0/0
```
### 3. Configuration SW-DIST-02
```
ip routing
interface Gi0/0
 description LINK_R-CORE
 no switchport
 ip address 10.0.0.6 255.255.255.252
 no shutdown
router ospf 1
 router-id 3.3.3.3
 network 10.0.0.4 0.0.0.3 area 0
 network 192.168.0.0 0.0.255.255 area 0
 passive-interface default
 no passive-interface Gi0/0
