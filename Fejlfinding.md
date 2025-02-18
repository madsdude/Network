Test af WAN-linje

1. BGP

Kommando:
```
show bgp all sum
```
Viser en oversigt over alle BGP-naboer og deres status.

2. BGP CORE

Kommando:
```
show bgp all sum | inc x.x.x.x
```
Filtrerer BGP-oversigten for en specifik IP-adresse.

3. Interface

WAN IP:
```
show ip int bri
```
Viser en oversigt over alle interfaces og deres IP-adresser.

4. CPE IP

Kommandoer:
```
show ip route vrf Internet
ping vrf Internet x.x.x.x
```
Viser IP-ruter for VRF 'Internet' og tester forbindelsen til en CPE IP.

5. Tunneler

R1-CORE-DK-R3

R2-CORE-DK-R4

Eksterne Kommandoer
```
show bgp all neighbors x.x.x.x received-routes
show bgp all neighbors x.x.x.x routes
```
Viser modtagne BGP-ruter og annoncerede ruter til en specifik nabo.

