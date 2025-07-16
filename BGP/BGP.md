<h1> BGP står for Border Gateway Protocol og er en standardiseret gateway-protokol, der bruges til at udveksle routing-information mellem forskellige autonome systemer (AS) på internettet. Her er en kort forklaring: </h1>

<p> Hvad er BGP?
Formål: BGP bruges til at finde den bedste vej for data at rejse over forskellige netværk på internettet. Det er grundlaget for internettets routing-infrastruktur.

Autonome Systemer (AS): Et autonomt system er et samling af IP-netværk og routere under en fælles administrativ kontrol og med en fælles routingstrategi. BGP håndterer routing mellem disse autonome systemer.

Hvordan det fungerer:

BGP-routere udveksler information om hvilke IP-adresser de kan nå, og hvilken vej data skal tage for at nå disse adresser.
BGP bruger en række attributter (f.eks. AS-stier, next-hop) til at vælge den bedste vej blandt flere tilgængelige ruter.
Typer af BGP:

eBGP (External BGP): Bruges til at udveksle routing-information mellem forskellige autonome systemer.
iBGP (Internal BGP): Bruges inden for et enkelt autonomt system for at sikre konsistent routing-information mellem alle routere i systemet.
Stabilitet og sikkerhed: BGP er designet til at være robust og stabil, men det kan være sårbart over for visse typer af angreb og fejlkonfigurationer, hvilket kan føre til routingproblemer som f.eks. route hijacking eller blackholing.
</p>

<H2> Opsummering </H2>
<p> BGP er afgørende for internettets funktion, da det sikrer, at data kan rejse mellem forskellige netværk på den mest effektive måde. Det håndterer kompleks routing mellem store netværk og er en grundpille i internettets arkitektur.
</p>

R1: AS 65001, IP 192.168.12.1 (mod R2) og 192.168.13.1 (mod R3)

R2: AS 65002, IP 192.168.12.2 (mod R1) og 192.168.23.2 (mod R3)

R3: AS 65003, IP 192.168.13.3 (mod R1) og 192.168.23.3 (mod R2)

```
router bgp 65001
 bgp log-neighbor-changes
 neighbor 192.168.12.2 remote-as 65002
 neighbor 192.168.13.3 remote-as 65003

 ! Annoncering af loopback
 network 1.1.1.1 mask 255.255.255.255
```
```
router bgp 65002
 bgp log-neighbor-changes
 neighbor 192.168.12.1 remote-as 65001
 neighbor 192.168.23.3 remote-as 65003

 ! Annoncering af loopback
 network 2.2.2.2 mask 255.255.255.255
```
```
router bgp 65003
 bgp log-neighbor-changes
 neighbor 192.168.13.1 remote-as 65001
 neighbor 192.168.23.2 remote-as 65002

 ! Annoncering af loopback
 network 3.3.3.3 mask 255.255.255.255

```
```
interface Loopback0
 ip address x.x.x.x 255.255.255.255

```
