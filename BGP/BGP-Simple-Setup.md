# Cisco Router BGP Konfiguration og Test

Denne guide indeholder konfiguration for to Cisco routere (R1 og R2) med BGP peering, route-maps, prefix-lister samt kommandoer til test af reachability og BGP-session.

---

## Router1 (AS 65001)

```plaintext
! Hostname
hostname Router1

! Basis systemkonfiguration
no ip domain-lookup
service timestamps debug datetime msec
service timestamps log datetime msec
!
! Logging
logging buffered 51200 warnings
no logging console
!
! Interfaces
interface FastEthernet0/0
 ip address 192.168.12.1 255.255.255.252
 no shutdown
 duplex auto
 speed auto
!
interface FastEthernet0/1
 no ip address
 shutdown
 duplex auto
 speed auto
!

! Prefix-list - tillad kun 10.1.1.0/24 udgående
ip prefix-list PL-OUT seq 5 permit 10.1.1.0/24

! Route-map til udgående annoncer
route-map RM-OUT permit 10
 match ip address prefix-list PL-OUT

! Prefix-list til indgående ruter - tillad alle
ip prefix-list PL-IN seq 5 permit 0.0.0.0/0 le 32

! Route-map til indgående ruter (tillad alle)
route-map RM-IN permit 10
 match ip address prefix-list PL-IN

! BGP konfiguration
router bgp 65001
 neighbor 192.168.12.2 remote-as 65002
 neighbor 192.168.12.2 description Peer til Router2
 neighbor 192.168.12.2 route-map RM-OUT out
 neighbor 192.168.12.2 route-map RM-IN in
 network 10.1.1.0 mask 255.255.255.0
!
```

Router2 (AS 65002)

```plaintext

! Hostname
hostname Router2

! Basis systemkonfiguration
no ip domain-lookup
service timestamps debug datetime msec
service timestamps log datetime msec
!
! Logging
logging buffered 51200 warnings
no logging console
!
! Interfaces
interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.252
 no shutdown
 duplex auto
 speed auto
!
interface FastEthernet0/1
 no ip address
 shutdown
 duplex auto
 speed auto
!

! Prefix-list - tillad kun 10.2.2.0/24 udgående
ip prefix-list PL-OUT seq 5 permit 10.2.2.0/24

! Route-map til udgående annoncer
route-map RM-OUT permit 10
 match ip address prefix-list PL-OUT

! Prefix-list til indgående ruter - tillad alle
ip prefix-list PL-IN seq 5 permit 0.0.0.0/0 le 32

! Route-map til indgående ruter (tillad alle)
route-map RM-IN permit 10
 match ip address prefix-list PL-IN

! BGP konfiguration
router bgp 65002
 neighbor 192.168.12.1 remote-as 65001
 neighbor 192.168.12.1 description Peer til Router1
 neighbor 192.168.12.1 route-map RM-OUT out
 neighbor 192.168.12.1 route-map RM-IN in
 network 10.2.2.0 mask 255.255.255.0
!
Test af reachability og BGP session
Test interface reachability
På begge routere kan du teste reachability til peer:
```

ping 192.168.12.X
(Hvor 192.168.12.X er IP-adressen på den anden routers FastEthernet0/0 interface.)

Test BGP session status
For at se status på BGP sessionen:

```plaintext

show ip bgp summary
For detaljer om BGP nabo:
```

```plaintext

show ip bgp neighbors 192.168.12.X
```

Opsummering
Router1 og Router2 peerer hinanden via BGP over netværket 192.168.12.0/30.

Route-maps og prefix-lister filtrerer udgående og indgående ruter.

Ping og BGP show-kommandoer bruges til at teste forbindelsen og sessionen.

