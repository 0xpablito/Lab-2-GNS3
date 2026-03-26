# Phase 5 — Services IP & NAT

**Cible :** `R-CORE`, `WS-DC-01` & `SW-DIST-01/02`

## 📋 Configuration

### 1. Configuration du NAT et de la sortie Internet (R-CORE)
```
interface GigabitEthernet0/2
 ip address dhcp
 ip nat outside
 no shutdown

interface GigabitEthernet0/0
 ip nat inside
interface GigabitEthernet0/1
 ip nat inside

access-list 1 permit 192.168.0.0 0.0.255.255 ! ACL pour définir le trafic interne autorisé à être translaté

ip nat inside source list 1 interface GigabitEthernet0/2 overload ! Activation du PAT (NAT Overload)

ip route 0.0.0.0 0.0.0.0 192.168.122.1 ! Route par défaut pointant vers l'IP du NAT Node GNS3 (Next-Hop)

router ospf 1
 default-information originate ! Propagation de la route par défaut via OSPF
```
### 2. Configuration du Relais DHCP (SW-DIST-01 & SW-DIST-02)
Configuration identique sur les deux switches de distribution pour la redondance
```
interface Vlan 10
 ip helper-address 192.168.20.10
interface Vlan 30
 ip helper-address 192.168.20.10
interface Vlan 40
 ip helper-address 192.168.20.10
```
### 3. Configuration du Services Windows Server
Configuration réalisée via PowerShell pour l'IP statique, le DNS et les Scopes DHCP.
```
# 1. Configuration IP Statique
# Configuration IP Statique
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.20.10 -PrefixLength 24 -DefaultGateway 192.168.20.1

# --- 2. AD DS & DNS ---
# Installation des rôles
Install-WindowsFeature -Name AD-Domain-Services, DNS -IncludeManagementTools

# Promotion du contrôleur de domaine (Forest)
Import-Module ADDSDeployment
Install-ADDSForest -DomainName "atlas.local" -SafeModePassword (Read-Host -AsSecureString) -Force

# Configuration des Forwarders DNS (pour l'accès Internet des clients)
Add-DnsServerForwarder -IPAddress "8.8.8.8", "1.1.1.1"

# --- 3. DHCP SERVER ---
# Installation du rôle DHCP
Install-WindowsFeature -Name DHCP -IncludeManagementTools

# Autorisation du serveur dans l'Active Directory (Crucial pour le fonctionnement)
Add-DhcpServerInDC -DnsName "WS-DC-01.atlas.local" -IPAddress 192.168.20.10

# Création des Scopes DHCP pour les VLANs
# VLAN 10 - Users
Add-DhcpServerv4Scope -Name "VLAN-10-Users" -StartRange 192.168.10.100 -EndRange 192.168.10.200 -SubnetMask 255.255.255.0
Set-DhcpServerv4OptionValue -ScopeId 192.168.10.0 -OptionId 3 -Value 192.168.10.1
Set-DhcpServerv4OptionValue -ScopeId 192.168.10.0 -OptionId 6 -Value 192.168.20.10

# VLAN 30 - RH
Add-DhcpServerv4Scope -Name "VLAN-30-RH" -StartRange 192.168.30.100 -EndRange 192.168.30.200 -SubnetMask 255.255.255.0
Set-DhcpServerv4OptionValue -ScopeId 192.168.30.0 -OptionId 3 -Value 192.168.30.1
Set-DhcpServerv4OptionValue -ScopeId 192.168.30.0 -OptionId 6 -Value 192.168.20.10

# VLAN 40 - Guest
Add-DhcpServerv4Scope -Name "VLAN-40-Guest" -StartRange 192.168.40.100 -EndRange 192.168.40.200 -SubnetMask 255.255.255.0
Set-DhcpServerv4OptionValue -ScopeId 192.168.40.0 -OptionId 3 -Value 192.168.40.1
Set-DhcpServerv4OptionValue -ScopeId 192.168.40.0 -OptionId 6 -Value 192.168.20.10
```
