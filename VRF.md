# Virtual Routing and Forwarding (VRF) og Default Route

## Hvad er VRF?

**VRF** (Virtual Routing and Forwarding) er en teknologi, der gør det muligt at have flere separate, logiske routingtabeller på én fysisk router. Dette giver mulighed for at opdele og isolere netværkstrafik, selvom den kører på samme fysiske infrastruktur.

### Fordele ved VRF

- **Isolation**  
  Hver VRF har sin egen routingtabel, så trafikken i én VRF er adskilt fra trafikken i en anden.
- **Sikkerhed**  
  Adskillelsen af routingtabeller sikrer, at data fra én VRF ikke kan “kigge ind” i en anden.
- **Skalerbarhed**  
  Du kan tilføje flere VRF-instanser efter behov — fx én VRF per kunde i et serviceudbyder-netværk.

---

## Default Route (0.0.0.0/0) i VRF

En **default route** (0.0.0.0/0) er den rute, som al trafik tager, hvis der ikke findes en mere specifik rute i routingtabellen. Hver VRF kan have sin egen default route, der peger på en unik next-hop.

> **Vigtigt**: Når du konfigurerer en default route i en VRF, skal du bruge syntaksen  
> `ip route vrf <VRF-navn> 0.0.0.0 0.0.0.0 <next-hop>`  
> for at gøre denne route gældende **udelukkende** for den specifikke VRF.

---

## Eksempel: VRF “Red” og Default Route på en Cisco-router

```plaintext
!
! 1) Opret VRF "Red"
ip vrf Red
 rd 100:1
 route-target export 100:1
 route-target import 100:1
exit
!
! 2) Tilknyt et Interface til VRF "Red"
interface GigabitEthernet0/0
 ip vrf forwarding Red
 ip address 192.168.1.1 255.255.255.0
exit
!
! 3) Sæt en default route for VRF "Red"
ip route vrf Red 0.0.0.0 0.0.0.0 10.0.0.1
```

### Forklaring

#### Opret VRF “Red”

- Med `ip vrf Red` oprettes en ny VRF-instans ved navn “Red”.
- `rd` (Route Distinguisher) og `route-target` bruges til at adskille og eventuelt dele ruter mellem VRF’er (især i MPLS/VPN-sammenhænge).

#### Tilknyt grænseflade

- Kommandoen `ip vrf forwarding Red` på interfacet `GigabitEthernet0/0` gør, at trafikken herfra fremover benytter VRF “Red”.
- IP-adressen `192.168.1.1` er nu knyttet til VRF “Red” i stedet for routerens globale routing-tabel.

#### Default route i VRF “Red”

- Kommandoen `ip route vrf Red 0.0.0.0 0.0.0.0 10.0.0.1` sørger for, at al trafik, der ikke har en mere specifik rute, sendes til `10.0.0.1` inden for VRF “Red”.
- Uden `vrf Red`-delen ville denne default route ligge i den globale routing-tabel i stedet.

