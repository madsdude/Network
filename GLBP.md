# GLBP (Gateway Load Balancing Protocol) - Guide og Konfiguration

## Hvad er GLBP?

**GLBP (Gateway Load Balancing Protocol)** er en Cisco-proprietær protokol, der både tilbyder *redundans* og *load balancing* for gateways i et netværk. Hvor protokoller som HSRP og VRRP kun tillader én aktiv gateway ad gangen, tillader GLBP, at flere gateways håndterer trafik samtidig.

## Funktioner

- **Redundans**: Hvis én router fejler, overtager en anden automatisk.
- **Load balancing**: Flere routere kan deles om trafikbelastningen.
- **Virtuel gateway og virtuelle MAC-adresser**: En gruppe af routere optræder som én virtuel router med én IP-adresse og flere MAC-adresser.

## Grundlæggende Roller i GLBP

- **AVG (Active Virtual Gateway)**: Ansvarlig for at tildele virtuelle MAC-adresser til andre routere.
- **AVF (Active Virtual Forwarder)**: Routere som rent faktisk videresender trafik.
- Hver AVF får tildelt en virtuel MAC, som klienter kan bruge.

---

## Typisk Netværksscenarie

```text
  [PC1]----|
           |---(SW)---[R1] <--- GLBP Router (AVG + AVF)
           |          [R2] <--- GLBP Router (AVF)
  [PC2]----|
```
GLBP Konfigurationseksempel
Antag følgende setup:

Virtuel IP: 192.168.1.1

R1: Prioritet 120 (foretrukken AVG)

R2: Prioritet 100

Konfiguration på R1:
```
interface GigabitEthernet0/0
 ip address 192.168.1.2 255.255.255.0
 glbp 1 ip 192.168.1.1
 glbp 1 priority 120
 glbp 1 preempt
 glbp 1 load-balancing round-robin
```
Konfiguration på R2:
```
interface GigabitEthernet0/0
 ip address 192.168.1.3 255.255.255.0
 glbp 1 ip 192.168.1.1
 glbp 1 priority 100
 glbp 1 preempt
 glbp 1 load-balancing round-robin
```

Verifikation

Vis status:
```
show glbp
show glbp brief
```
Eksempeloutput:
```
Interface   Grp  Fwd Pri State    Address         Active router
Gi0/0       1    1   120  Active   192.168.1.2     Local
Gi0/0       1    2   100  Listen   192.168.1.3     192.168.1.2
```
