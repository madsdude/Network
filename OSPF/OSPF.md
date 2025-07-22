# OSPF

### OSPF (Open Shortest Path First) er en routing-protokol brugt i IP-netværk for at finde den korteste vej for data mellem routere. Den er en af de vigtigste protokoller inden for internettets backbone og anvendes bredt i store netværksinstallationer. OSPF tilhører gruppen af link-state routing-protokoller og adskiller sig fra distance-vector routing-protokoller ved at have en fuld oversigt over netværkets topologi, hvilket gør det muligt at beregne den optimale rute meget effektivt.

## Grundlæggende principper for OSPF

### 1. Autonomous Systems (AS): OSPF opererer inden for et enkelt autonomt system, som er en samling af netværk under en fælles administrationsdomæne.

### 2. Area: For at skabe skalerbarhed opdeles et OSPF-netværk i områder (areas), med Area 0 (backbone area) som det centrale område, hvortil alle andre områder skal forbindes direkte eller indirekte.

### 3. Router ID (RID): Hver router i et OSPF-netværk identificeres ved et unikt Router ID, som normalt er den højeste IP-adresse på routerens interfaces eller kan konfigureres manuelt.

### 4. Link-State Advertisements (LSAs): OSPF bruger LSAs til at udveksle information om netværkets topologi mellem routere. Dette inkluderer information om tilgængelige links, deres status og tilhørende metrik.

### 5. Shortest Path First (SPF): OSPF anvender Dijkstra's SPF-algoritme til at beregne den korteste vej gennem netværket baseret på LSAs.

### 6. Cost: OSPF tildeler "cost" til hvert link, som er invers proportional med linkets båndbredde. Routen med lavest samlet cost vælges som den bedste vej.

## Konfigurationseksempel

### Nedenstående er et simpelt eksempel på OSPF-konfiguration på en Cisco-router. Det viser, hvordan man aktiverer OSPF, definerer et OSPF-område og konfigurerer interfaces til at deltage i OSPF-processen.

```
router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.255.255.255 area 0
 network 192.168.1.0 0.0.0.255 area 1

interface GigabitEthernet0/0
 ip ospf cost 10
 ip ospf priority 1

interface GigabitEthernet0/1
 ip ospf cost 20
 ip ospf priority 0
```

### ¤ router ospf 1: Starter OSPF-processen og tildeler den et proces-ID

### ¤ router-id 1.1.1.1: Sætter en statisk router-ID.

### ¤ network 10.0.0.0 0.255.255.255 area 0: Definerer hvilke interfaces der deltager i OSPF-område 0 baseret på deres IP-adresser.

### ¤ network 192.168.1.0 0.0.0.255 area 1: Ligner ovenstående, men for OSPF-område 1.

### ¤ ip ospf cost: Sætter omkostningen (cost) for interfacet, hvilket påvirker valget af den bedste rute.

### ¤ ip ospf priority: Bestemmer routerens valgbarhed som Designated Router (DR) på et multi-access netværk.
