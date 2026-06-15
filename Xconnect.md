Cisco Xconnect Guide
====================

1\. Hvad er Xconnect?
---------------------

**Xconnect** bruges til at lave en **Layer 2-forbindelse gennem et Layer 3/MPLS-core-netværk**.

Man kan tænke på det som en "virtuel kabel-forlængelse" mellem to sites.

Eksempel:

```
CE1 --- PE1 ==== MPLS CORE ==== PE2 --- CE2        |                         |        |------ Xconnect ---------|
```

CE-routerne eller switchene tror, at de er forbundet direkte på Layer 2, selvom trafikken reelt bliver transporteret gennem et MPLS-netværk.

Cisco kalder det ofte:

-   **EoMPLS** -- Ethernet over MPLS
-   **L2VPN**
-   **Pseudowire**
-   **Xconnect**
-   **VPWS** -- Virtual Private Wire Service

Cisco beskriver EoMPLS som en måde at transportere Ethernet over MPLS, enten i **port mode** eller **VLAN mode**. Nyere IOS XE-platforme understøtter også nyere L2VPN "protocol-based CLI", mens ældre `xconnect`-syntaks stadig ofte ses i labs og produktion.

* * * * *

2\. Hvad bruges Xconnect til?
=============================

Xconnect bruges typisk til:

```
EPL / Ethernet Private LineLayer 2 site-to-site forbindelseVLAN transport mellem to lokationerKunde-L2 gennem ISP MPLS-coreTransparent transport af kundens trafik
```

Eksempel:

```
Kunde VLAN 100 i A-site skal forbindes direkte til VLAN 100 i B-site.ISP'en router ikke kundens trafik.ISP'en transporterer kun Layer 2 gennem MPLS.
```

Det betyder:

-   ISP-core ser ikke kundens IP-routing.
-   Kunden kan selv køre OSPF, BGP, static routing eller bare Layer 2 over forbindelsen.
-   PE-routerne laver kun "wire service".

* * * * *

3\. Vigtige begreber
====================

PE
--

**Provider Edge**\
Routeren i ISP-netværket, hvor kundens CE kobles på.

```
CE --- PE --- P --- P --- PE --- CE
```

P-router
--------

**Provider router**\
Core-router i MPLS-netværket. Den kender normalt ikke kundens VLAN eller IP-net.

CE
--

**Customer Edge**\
Kundens router, switch eller firewall.

Pseudowire
----------

En virtuel Layer 2-forbindelse mellem to PE-routere.

VC ID
-----

**Virtual Circuit ID**\
Et ID, der skal matche i begge ender af xconnect.

Eksempel:

```
PE1: xconnect 2.2.2.2 100 encapsulation mplsPE2: xconnect 1.1.1.1 100 encapsulation mpls
```

Her er `100` VC ID'et.

Transport label
---------------

MPLS-label, der får pakken gennem core-netværket.

VC label
--------

Label, der identificerer selve pseudowiren/xconnecten.

* * * * *

4\. Krav før Xconnect virker
============================

Før du laver xconnect, skal MPLS-core være i orden.

Minimumskrav:

```
1\. PE1 og PE2 skal kunne nå hinandens loopback-adresser.2\. MPLS skal være aktivt i core.3\. LDP skal være oppe mellem MPLS-routerne.4\. PE-routerne skal have label reachability til hinanden.5\. VC ID skal matche i begge ender.6\. Encapsulation skal passe.7\. Kundevendt interface må normalt ikke have IP-adresse.
```

Typisk underliggende setup:

```
OSPF/ISIS i coreMPLS LDP i coreLoopback som LDP router-idXconnect mellem PE-loopbacks
```

* * * * *

5\. Simpel topologi
===================

```
CE1             PE1              P              PE2             CE2 |              |                |               |               | | Gi0/1        | Gi0/1          |               | Gi0/1         | Gi0/1 |--------------|                |               |---------------|                \              MPLS             /                 \____________ CORE ___________/
```

Loopbacks:

```
PE1 Loopback0: 1.1.1.1/32PE2 Loopback0: 2.2.2.2/32
```

Kunde-VLAN:

```
VLAN 100 transporteres mellem CE1 og CE2
```

* * * * *

6\. MPLS-core grundkonfiguration
================================

PE1
---

```
hostname PE1interface Loopback0 ip address 1.1.1.1 255.255.255.255interface GigabitEthernet0/0 description TO-P-CORE ip address 10.0.12.1 255.255.255.252 mpls ip no shutdownrouter ospf 1 router-id 1.1.1.1 network 1.1.1.1 0.0.0.0 area 0 network 10.0.12.0 0.0.0.3 area 0mpls label protocol ldpmpls ldp router-id Loopback0 force
```

PE2
---

```
hostname PE2interface Loopback0 ip address 2.2.2.2 255.255.255.255interface GigabitEthernet0/0 description TO-P-CORE ip address 10.0.23.2 255.255.255.252 mpls ip no shutdownrouter ospf 1 router-id 2.2.2.2 network 2.2.2.2 0.0.0.0 area 0 network 10.0.23.0 0.0.0.3 area 0mpls label protocol ldpmpls ldp router-id Loopback0 force
```

* * * * *

7\. Xconnect med VLAN subinterface
==================================

Dette er meget brugt, når du vil transportere et bestemt VLAN.

PE1
---

```
interface GigabitEthernet0/1.100 description XCONNECT-CUSTOMER-A-VLAN100-TO-PE2 encapsulation dot1Q 100 no ip address xconnect 2.2.2.2 100 encapsulation mpls
```

PE2
---

```
interface GigabitEthernet0/1.100 description XCONNECT-CUSTOMER-A-VLAN100-TO-PE1 encapsulation dot1Q 100 no ip address xconnect 1.1.1.1 100 encapsulation mpls
```

Vigtigt:

```
VC ID skal være ens i begge ender.Remote peer skal være modpartens PE-loopback.Encapsulation skal være ens.Subinterface VLAN skal passe til kundens VLAN.
```

* * * * *

8\. Xconnect med pseudowire-class
=================================

Det er pænere og mere struktureret at bruge en `pseudowire-class`.

PE1 og PE2
----------

```
pseudowire-class PW-MPLS encapsulation mpls
```

PE1
---

```
interface GigabitEthernet0/1.100 description CUSTOMER-A-EPL-VLAN100 encapsulation dot1Q 100 no ip address xconnect 2.2.2.2 100 pw-class PW-MPLS
```

PE2
---

```
interface GigabitEthernet0/1.100 description CUSTOMER-A-EPL-VLAN100 encapsulation dot1Q 100 no ip address xconnect 1.1.1.1 100 pw-class PW-MPLS
```

Fordel:

```
Man kan genbruge pseudowire-class til flere xconnects.Det gør konfigurationen mere ensartet.Det er lettere at tilføje features senere.
```

* * * * *

9\. Port mode vs VLAN mode
==========================

Port mode
---------

Hele interfacet transporteres som én Layer 2-forbindelse.

```
interface GigabitEthernet0/1 description PORT-MODE-XCONNECT no ip address xconnect 2.2.2.2 100 encapsulation mpls
```

Bruges når hele porten skal være én pseudowire.

VLAN mode
---------

Kun ét bestemt VLAN transporteres.

```
interface GigabitEthernet0/1.100 encapsulation dot1Q 100 no ip address xconnect 2.2.2.2 100 encapsulation mpls
```

Bruges når flere kunder/VLANs deler samme fysiske trunk.

Cisco skelner mellem EoMPLS port mode og VLAN mode, hvor port mode transporterer hele porten, mens VLAN mode bruges til VLAN-baseret transport.

* * * * *

10\. Flere kunder på samme PE-interface
=======================================

Eksempel:

```
Kunde A bruger VLAN 100Kunde B bruger VLAN 200Kunde C bruger VLAN 300
```

PE1
---

```
interface GigabitEthernet0/1.100 description CUSTOMER-A-EPL encapsulation dot1Q 100 no ip address xconnect 2.2.2.2 100 encapsulation mplsinterface GigabitEthernet0/1.200 description CUSTOMER-B-EPL encapsulation dot1Q 200 no ip address xconnect 2.2.2.2 200 encapsulation mplsinterface GigabitEthernet0/1.300 description CUSTOMER-C-EPL encapsulation dot1Q 300 no ip address xconnect 2.2.2.2 300 encapsulation mpls
```

PE2
---

```
interface GigabitEthernet0/1.100 description CUSTOMER-A-EPL encapsulation dot1Q 100 no ip address xconnect 1.1.1.1 100 encapsulation mplsinterface GigabitEthernet0/1.200 description CUSTOMER-B-EPL encapsulation dot1Q 200 no ip address xconnect 1.1.1.1 200 encapsulation mplsinterface GigabitEthernet0/1.300 description CUSTOMER-C-EPL encapsulation dot1Q 300 no ip address xconnect 1.1.1.1 300 encapsulation mpls
```

Noteregel:

```
VLAN ID og VC ID behøver teknisk set ikke være ens,men det er god praksis i lab og drift at holde dem ens.
```

* * * * *

11\. Verifikation
=================

Se alle xconnects
-----------------

```
show xconnect all
```

Eksempel på godt output:

```
XC ST  Segment 1                         S1 Segment 2                         S2UP     ac Gi0/1.100:100                  UP mpls 2.2.2.2:100                  UP
```

Det vigtige er:

```
XC ST = UPS1 = UPS2 = UP
```

Hvis xconnect er nede:

```
XC ST = DNS1 = DN eller UPS2 = DN eller UP
```

Se detaljer
-----------

```
show xconnect all detail
```

Bruges til at se:

```
Peer addressVC IDLocal circuitRemote circuitLabel statusStateUptime
```

Se MPLS LDP neighbor
--------------------

```
show mpls ldp neighbor
```

Hvis LDP ikke er oppe, virker EoMPLS normalt ikke.

Se MPLS forwarding
------------------

```
show mpls forwarding-table
```

Bruges til at se labels gennem core.

Se route til remote PE-loopback
-------------------------------

```
show ip route 2.2.2.2show ip cef 2.2.2.2
```

På PE1 skal du kunne nå PE2's loopback.

Ping remote PE-loopback
-----------------------

```
ping 2.2.2.2 source Loopback0
```

Hvis denne fejler, skal du ikke fejlfinde xconnect først. Så er problemet i core-reachability.

* * * * *

12\. Typisk fejlfindings-flow
=============================

Trin 1: Er kundevendt interface oppe?
-------------------------------------

```
show ip interface briefshow interfaces GigabitEthernet0/1.100
```

Hvis interface er down/down:

```
Fysisk link er nede.Tjek kabel, SFP, switchport, shutdown.
```

Hvis interface er up/down eller administratively down:

```
Tjek no shutdown.Tjek encapsulation.Tjek modpartens port.
```

* * * * *

Trin 2: Kan PE-routerne nå hinanden?
------------------------------------

På PE1:

```
ping 2.2.2.2 source Loopback0
```

På PE2:

```
ping 1.1.1.1 source Loopback0
```

Hvis det ikke virker:

```
Tjek OSPF/ISIS i core.Tjek route til loopbacks.Tjek MPLS på core-links.
```

* * * * *

Trin 3: Er LDP oppe?
--------------------

```
show mpls ldp neighbor
```

Hvis LDP ikke er oppe:

```
Tjek at mpls ip er aktiveret på core-interfaces.Tjek at loopbacks annonceres i IGP.Tjek at PE/P routere kan nå hinandens LDP router-id.
```

* * * * *

Trin 4: Er xconnect up?
-----------------------

```
show xconnect allshow xconnect all detail
```

Hvis Segment 1 er down:

```
Problemet er lokalt attachment circuit.Tjek kundevendt interface/subinterface/VLAN.
```

Hvis Segment 2 er down:

```
Problemet er pseudowire/MPLS/remote PE.Tjek peer IP, VC ID, LDP og remote config.
```

* * * * *

13\. Typiske fejl
=================

Fejl 1: VC ID matcher ikke
--------------------------

PE1:

```
xconnect 2.2.2.2 100 encapsulation mpls
```

PE2:

```
xconnect 1.1.1.1 200 encapsulation mpls
```

Problem:

```
PE1 forventer VC ID 100.PE2 forventer VC ID 200.Pseudowire kommer ikke korrekt op.
```

Fix:

```
xconnect 1.1.1.1 100 encapsulation mpls
```

* * * * *

Fejl 2: Remote peer er forkert
------------------------------

PE1:

```
xconnect 10.0.23.2 100 encapsulation mpls
```

Bedre:

```
xconnect 2.2.2.2 100 encapsulation mpls
```

Man bør normalt bruge remote PE's loopback, ikke et fysisk interface.

* * * * *

Fejl 3: Mangler MPLS på core-link
---------------------------------

```
interface GigabitEthernet0/0 ip address 10.0.12.1 255.255.255.252 no mpls ip
```

Fix:

```
interface GigabitEthernet0/0 mpls ip
```

* * * * *

Fejl 4: Kundevendt interface har IP-adresse
-------------------------------------------

Forkert:

```
interface GigabitEthernet0/1.100 encapsulation dot1Q 100 ip address 192.168.100.1 255.255.255.252 xconnect 2.2.2.2 100 encapsulation mpls
```

Rigtigt:

```
interface GigabitEthernet0/1.100 encapsulation dot1Q 100 no ip address xconnect 2.2.2.2 100 encapsulation mpls
```

Xconnect attachment circuit er Layer 2 og skal normalt ikke routes lokalt.

* * * * *

Fejl 5: VLAN mismatch
---------------------

CE sender VLAN 100, men PE forventer VLAN 200:

```
interface GigabitEthernet0/1.200 encapsulation dot1Q 200 xconnect 2.2.2.2 100 encapsulation mpls
```

Hvis CE reelt sender VLAN 100, skal PE bruge:

```
interface GigabitEthernet0/1.100 encapsulation dot1Q 100 xconnect 2.2.2.2 100 encapsulation mpls
```

* * * * *

14\. Hvad sker der med MAC-adresser?
====================================

Xconnect transporterer Layer 2, men PE-routeren fungerer ikke som en normal switch i samme forstand som en bridge-domain.

Ved simpel point-to-point xconnect:

```
Trafik ind på attachment circuit bliver sendt gennem pseudowire.Trafik ind fra pseudowire bliver sendt ud på attachment circuit.
```

Det er ikke klassisk routing.

Det er heller ikke almindelig Ethernet switching med mange porte, medmindre du bruger bridge-domain/VPLS/EVPN-lignende design.

* * * * *

15\. Xconnect vs VRF
====================

Xconnect
--------

```
Layer 2 transportKunden styrer selv IP-routingISP ser ikke kundens IP-netBruges til EPL/Layer 2 service
```

VRF/MPLS L3VPN
--------------

```
Layer 3 routingISP PE router med kundenISP har kundens routes i en VRFBruges til routed MPLS VPN
```

Kort sagt:

```
Xconnect = ISP leverer kabelVRF = ISP leverer routing
```

* * * * *

16\. Xconnect vs VPLS
=====================

Xconnect / VPWS
---------------

```
Point-to-point Layer 2Én forbindelse mellem to sitesSimpelt EPL-design
```

VPLS
----

```
Multipoint Layer 2Flere sites i samme Layer 2-domæneMinder mere om en distribueret switch
```

Hvis du kun skal forbinde Site A til Site B, giver xconnect bedst mening.

Hvis du skal have mange sites i samme Layer 2-domæne, kigger man typisk på VPLS eller EVPN.

* * * * *

17\. Eksempel med din type lab: CORE og sites
=============================================

Eksempel:

```
CORE-PE1 ---- MPLS CORE ---- PE2 ---- Site A
```

Hvis Site A skal have en EPL til CORE på VLAN 100:

CORE-PE1
--------

```
interface GigabitEthernet0/1.100 description EPL-SITE-A-VLAN100 encapsulation dot1Q 100 no ip address xconnect 2.2.2.2 100 encapsulation mpls
```

PE2
---

```
interface GigabitEthernet0/1.100 description EPL-SITE-A-VLAN100 encapsulation dot1Q 100 no ip address xconnect 1.1.1.1 100 encapsulation mpls
```

På CE-routerne kan du så køre IP-routing ovenpå forbindelsen:

```
CORE-CE Gi0/0.100: 10.10.0.1/30SITE-CE Gi0/0.100: 10.10.0.2/30
```

Så kan CE'erne køre:

```
BGPOSPFStatic routing
```

Det er vigtigt at forstå:

```
Xconnect transporterer kun Layer 2.BGP/OSPF kører mellem CE-routerne ovenpå den virtuelle Layer 2-forbindelse.
```

* * * * *

18\. Brug af nyere L2VPN CLI
============================

Nyere IOS XE-platforme kan bruge en mere moderne L2VPN-struktur i stedet for den klassiske interface-baserede `xconnect`. Cisco beskriver, at nyere L2VPN protocol-based CLI blev introduceret for mere ensartet funktionalitet på tværs af Cisco-platforme, og at ældre kommandoer kan være deprecated i senere releases.

Eksempel:

```
l2vpn xconnect context CUSTOMER-A member GigabitEthernet0/1 service-instance 100 member Pseudowire100
```

Og pseudowire:

```
interface Pseudowire100 encapsulation mpls neighbor 2.2.2.2 100
```

Den klassiske `xconnect`-syntaks er dog stadig meget brugt i labs:

```
interface GigabitEthernet0/1.100 encapsulation dot1Q 100 no ip address xconnect 2.2.2.2 100 encapsulation mpls
```

* * * * *

19\. Hurtig huskeliste
======================

```
Xconnect = Layer 2 pseudowireVC ID skal matche i begge enderRemote peer er typisk remote PE loopbackPE loopbacks skal kunne nå hinandenMPLS og LDP skal virke i coreKundevendt xconnect-interface har normalt no ip addressshow xconnect all er den vigtigste verifikationskommandoSegment 1 = lokal attachment circuitSegment 2 = pseudowire/MPLS-siden
```

* * * * *

20\. De vigtigste kommandoer
============================

```
show xconnect allshow xconnect all detailshow mpls ldp neighborshow mpls forwarding-tableshow ip route <remote-pe-loopback>show ip cef <remote-pe-loopback>ping <remote-pe-loopback> source Loopback0show interfaces <interface>show run interface <interface>
```

* * * * *

21\. Mini-konfiguration til eksamen/lab
=======================================

PE1
---

```
interface Loopback0 ip address 1.1.1.1 255.255.255.255mpls ldp router-id Loopback0 forceinterface GigabitEthernet0/0 description TO-CORE ip address 10.0.12.1 255.255.255.252 mpls ip no shutdowninterface GigabitEthernet0/1.100 description XCONNECT-TO-PE2-VLAN100 encapsulation dot1Q 100 no ip address xconnect 2.2.2.2 100 encapsulation mpls
```

PE2
---

```
interface Loopback0 ip address 2.2.2.2 255.255.255.255mpls ldp router-id Loopback0 forceinterface GigabitEthernet0/0 description TO-CORE ip address 10.0.23.2 255.255.255.252 mpls ip no shutdowninterface GigabitEthernet0/1.100 description XCONNECT-TO-PE1-VLAN100 encapsulation dot1Q 100 no ip address xconnect 1.1.1.1 100 encapsulation mpls
```
