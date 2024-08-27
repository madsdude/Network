# LACP

## LACP står for Link Aggregation Control Protocol og er en del af IEEE 802.3ad-standarden, som senere blev absorberet i 802.1AX-standarden. LACP tillader flere fysiske netværksforbindelser (links) at blive bundtet til én logisk forbindelse, ofte refereret til som en "link aggregation group" (LAG) eller "port channel." Dette har flere formål:

### 1. 
#### Øget båndbredde: Ved at kombinere flere fysiske forbindelser i én logisk forbindelse, kan LACP øge den samlede båndbredde tilgængelig mellem to enheder, såsom switches, routere eller servere.
---
### 2. 
#### Belastningsfordeling: LACP kan fordele netværkstrafikken jævnt over de tilgængelige fysiske links i en LAG. Dette sikrer en mere effektiv udnyttelse af de samlede ressourcer.
---
### 3. 
#### Fejltolerance: Hvis et af de fysiske links i en LAG fejler, vil LACP automatisk omfordele trafikken til de resterende sunde links, hvilket forbedrer netværkets robusthed og pålidelighed.
---
#### LACP kræver, at begge ender af forbindelsen (f.eks., to switches) understøtter protokollen og er konfigureret til at tillade link-aggregering. Det er værd at bemærke, at selvom LACP kan øge båndbredden og forbedre netværkssikkerheden, så afhænger dets effektivitet af korrekt konfiguration og understøttelse fra netværksudstyret.

#### Sammenfatningsvis er LACP en nyttig protokol til at optimere netværksydeevne og pålidelighed ved at tillade aggregering af flere netværksforbindelser til en enkelt logisk forbindelse, hvilket giver fordele som øget båndbredde, forbedret belastningsfordeling og fejltolerance.
---

```.cisco 

conf t

interface Port-channel 1

interface range FastEthernet 1/0/17-20
switchport trunk encapsulation dot1q
switchport mode trunk
channel-group 1 mode active

```
