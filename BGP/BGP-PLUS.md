
# BGP Guide

## 1. Grundlæggende BGP opsætning

### Eksempel: Simpel eBGP peering mellem to routere

```bash
router bgp 65001
 bgp router-id 1.1.1.1
 neighbor 192.0.2.2 remote-as 65002
 !
 network 10.10.10.0 mask 255.255.255.0
```

- `router bgp <AS>`: Starter BGP proces med eget AS-nummer  
- `bgp router-id`: Unikt ID, ofte loopback IP  
- `neighbor`: Angiver BGP-peer og deres AS  
- `network`: Annoncerer et internt netværk i BGP  

---

## 2. BGP med flere peers (iTransit, iBGP og eBGP)

### Eksempel: iBGP og eBGP peers

```bash
router bgp 65001
 bgp router-id 1.1.1.1

 ! eBGP peering mod ISP
 neighbor 203.0.113.2 remote-as 65002
 !
 ! iBGP peering internt i AS
 neighbor 10.1.1.2 remote-as 65001
```

- eBGP: Peering mellem forskellige AS (fx mod ISP)  
- iBGP: Peering inden for samme AS (interne routere)  

---

## 3. BGP Multi-homed setup med to ISP'er og load balancing

### Topologi:

```
ISP1 (AS65002)
    |
R1 (AS65001) ---- LAN
    |
ISP2 (AS65003)
```

### Eksempel konfiguration:

```bash
router bgp 65001
 bgp router-id 1.1.1.1

 ! eBGP mod ISP1
 neighbor 192.0.2.2 remote-as 65002
 !
 ! eBGP mod ISP2
 neighbor 198.51.100.2 remote-as 65003

 ! Annoncer LAN netværk
 network 10.0.0.0 mask 255.255.255.0

 ! Load balancing via BGP multipath
 bgp bestpath as-path multipath-relax
```

- Multipath-relax tillader load balancing på tværs af flere AS-paths  

---

## 4. Route Reflector (RR)

### Problem:

iBGP kræver full mesh. Med mange routere bliver det uoverskueligt.

### Løsning:

En Route Reflector reflekterer BGP updates til andre iBGP peers.

### Eksempel på RR konfiguration:

På RR-routeren:

```bash
router bgp 65001
 bgp router-id 1.1.1.1
 !
 neighbor 10.1.1.2 remote-as 65001
 neighbor 10.1.1.2 route-reflector-client
 neighbor 10.1.1.3 remote-as 65001
 neighbor 10.1.1.3 route-reflector-client
```

- RR-klient markeres med `route-reflector-client`  
- RR sender opdateringer til alle klienter uden full mesh  

---

*Guide lavet af Madsdude*
