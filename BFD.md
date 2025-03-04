
bfd-template single-hop DC-Locations
interval min-tx 750 min-rx 750 multiplier 3

interface gig 0/0/0 


# Bidirectional Forwarding Detection (BFD) Guide

## Hvad er BFD?
Bidirectional Forwarding Detection (BFD) er en hurtig og letvægts protokol til at detektere link- og routingsvigt mellem to netværksenheder. BFD fungerer uafhængigt af routingprotokoller som OSPF, BGP og IS-IS og kan reducere failover-tider markant.

## Fordele ved BFD
- Hurtig detektion af linksvigt
- Uafhængig af routingprotokoller
- Lav overhead
- Kan anvendes til både fysiske og logiske links

---

## Konfiguration af BFD

### 1. **BFD Konfiguration på Cisco IOS**
```cisco
interface GigabitEthernet0/1
 ip address 192.168.1.1 255.255.255.0
 bfd interval 50 min_rx 50 multiplier 3
!
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 neighbor 192.168.1.2 bfd
```
**Forklaring:**
- `bfd interval 50 min_rx 50 multiplier 3` – Sender en BFD-pakke hver 50ms og forventer at modtage én inden 50ms. Hvis 3 pakker mistes, anses forbindelsen for nede.
- `neighbor 192.168.1.2 bfd` – Aktiverer BFD for naboen.

### 2. **BFD Konfiguration på Juniper JunOS**
```junos
set interfaces ge-0/0/1 unit 0 family inet address 192.168.1.1/24
set protocols ospf area 0.0.0.0 interface ge-0/0/1 bfd-liveness-detection minimum-interval 50 multiplier 3
```
**Forklaring:**
- `bfd-liveness-detection minimum-interval 50 multiplier 3` – Samme princip som i Cisco-konfigurationen.

### 3. **BFD Konfiguration på MikroTik RouterOS**
```mikrotik
/routing bfd interface add interface=ether1 min-tx=50ms min-rx=50ms multiplier=3
```
**Forklaring:**
- `min-tx=50ms` – Sender en BFD-pakke hver 50ms.
- `multiplier=3` – Hvis tre pakker mistes, betragtes forbindelsen som nede.

---

## Fejlfinding af BFD

### Cisco
```cisco
show bfd neighbors detail
show bfd sessions
show ip ospf neighbor
```

### Juniper
```junos
show bfd session
show ospf neighbor
```

### MikroTik
```mikrotik
/routing bfd print
```

## Konklusion
BFD er en hurtig metode til at detektere fejl i netværket og forbedre failover-tider. Det kan integreres med forskellige routingprotokoller for at forbedre netværksstabiliteten.
