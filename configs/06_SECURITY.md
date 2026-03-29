# Configuration — Phase 6 : Sécurisation & ACLs

Ce document regroupe l'implémentation technique des politiques de sécurité sur les équipements de distribution (Layer 3) et d'accès (Layer 2).

---

## 1. Distribution (SW-DIST-01 & 02)
*Filtrage inter-VLAN et protection des ressources critiques.*


Appliquée sur la SVI 20 (Sens OUT) pour isoler le Contrôleur de Domaine.
```
ip access-list extended ACL-SERVER
  1. Autorisation totale pour le VLAN Management
 permit ip 192.168.99.0 0.0.0.255 host 192.168.20.10

 2. Services infrastructure (DNS/DHCP) pour tous
 permit udp any host 192.168.20.10 eq domain
 permit udp any host 192.168.20.10 eq bootps

 3. Services métiers (SMB/LDAP) pour Users et RH uniquement
 permit tcp 192.168.10.0 0.0.0.255 host 192.168.20.10 eq 445
 permit tcp 192.168.30.0 0.0.0.255 host 192.168.20.10 eq 445
 permit tcp 192.168.10.0 0.0.0.255 host 192.168.20.10 eq 389
 permit tcp 192.168.30.0 0.0.0.255 host 192.168.20.10 eq 389

  4. Maintenance et retour de flux Internet
 permit icmp any host 192.168.20.10 echo
 permit tcp any host 192.168.20.10 established

 5. Blocage explicite vers le serveur / Autorisation du reste du VLAN
 deny ip any host 192.168.20.10
 permit ip any any
exit

interface Vlan 20
 ip access-group ACL-SERVER out
```

## 2. Accès (SW-ACC-01, 02, 03)
Sécurisation des ports terminaux et protection contre les attaques de couche 2.
```
ip dhcp snooping
ip dhcp snooping vlan 10,30,40
no ip dhcp snooping information option
ip arp inspection vlan 10,30,40

 Configuration des Uplinks vers Distribution
interface range gi0/0 - 1
 ip dhcp snooping trust
 ip arp inspection trust
interface gi3/3 - Lien serveur DHCP
 ip dhcp snooping trust
 ip arp inspection trust
```
Port-Security & STP Guards
```
interface range gi3/2 - 3
  Limitation MAC
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
  Protection STP
 spanning-tree portfast
 spanning-tree bpduguard enable
 spanning-tree guard root
```
## 3. Gestion & Plan de Contrôle (Tous équipements)
Sécurisation des accès administratifs.
```
access-list 99 permit 192.168.99.0 0.0.0.255
line vty 0 4
 access-class 99 in
```
