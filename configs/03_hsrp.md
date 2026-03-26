# Phase 3 — Haute Disponibilité L3 (HSRP)

**Cible :** `SW-DIST-01/02`

## 📋 Configuration

### 1. Configuration SW-DIST-01 (Active/Master)
Priorité augmentée à 110 pour forcer l'élection et activation du mode preempt.
```
 --- Configuration HSRP VLAN 10 (Users) ---
interface Vlan 10
 ip address 192.168.10.2 255.255.255.0
 standby version 2
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt
 ip helper-address 192.168.20.10  ! Relais vers le serveur DHCP (VLAN 20)

 --- Configuration HSRP VLAN 20 (Servers) ---
interface Vlan 20
 ip address 192.168.20.2 255.255.255.0
 standby version 2
 standby 20 ip 192.168.20.1
 standby 20 priority 110
 standby 20 preempt

 --- Configuration HSRP VLAN 30 (RH) ---
interface Vlan 30
 ip address 192.168.30.2 255.255.255.0
 standby version 2
 standby 30 ip 192.168.30.1
 standby 30 priority 110
 standby 30 preempt
 ip helper-address 192.168.20.10

 --- Configuration HSRP VLAN 40 (Guest) ---
interface Vlan 40
 ip address 192.168.40.2 255.255.255.0
 standby version 2
 standby 40 ip 192.168.40.1
 standby 40 priority 110
 standby 40 preempt

 --- Configuration HSRP VLAN 99 (Management) ---
interface Vlan 99
 ip address 192.168.99.2 255.255.255.0
 standby version 2
 standby 99 ip 192.168.99.1
 standby 99 priority 110
 standby 99 preempt
```
### 2. Configuration SW-DIST-02 (Standby)
Priorité par défaut (100) pour rester en écoute.
```
interface Vlan 10
 ip address 192.168.10.3 255.255.255.0
 standby version 2
 standby 10 ip 192.168.10.1
 ip helper-address 192.168.20.10

interface Vlan 20
 ip address 192.168.20.3 255.255.255.0
 standby version 2
 standby 20 ip 192.168.20.1

interface Vlan 30
 ip address 192.168.30.3 255.255.255.0
 standby version 2
 standby 30 ip 192.168.30.1
 ip helper-address 192.168.20.10

interface Vlan 40
 ip address 192.168.40.3 255.255.255.0
 standby version 2
 standby 40 ip 192.168.40.1

interface Vlan 99
 ip address 192.168.99.3 255.255.255.0
 standby version 2
 standby 99 ip 192.168.99.1
