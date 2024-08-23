<p>  En static route er en manuel indtastning af en rute i en router eller switch, der fortæller enheden, hvilken vej en pakke skal tage for at nå et bestemt netværk. I modsætning til dynamiske routing-protokoller, som automatisk finder og opdaterer ruter, forbliver statiske ruter konstant, indtil de manuelt ændres eller slettes. Static routes er nyttige i mindre netværk eller til specifikke opgaver, hvor ruten skal være forudsigelig og ikke ændre sig dynamisk. </p>

<h1> Grundlæggende konfiguration af en static route </h1>

<p> Lad os sige, at du har to netværk: </p>
<ol></ol>
 <li>Netværk A: 192.168.1.0/24</li>
 <li>Netværk B: 192.168.2.0/24 </li>
</ol>

<p></p>

<p> Routeren, der forbinder de to netværk, har to interfaces: </p>

<p> Interface FastEthernet 0/0 (Fa0/0) er tilsluttet Netværk A med IP-adressen 192.168.1.1 </p>

<p> Interface FastEthernet 0/1 (Fa0/1) er tilsluttet Netværk B med IP-adressen 192.168.2.1 </p>

<h2> Opsætning af en static route </h2>

<p> For at sikre, at routeren ved, hvordan man sender trafik fra Netværk A til Netværk B, kan du konfigurere en static route. </p>

<p> Eksempel på Cisco kode: </p>

```
Router(config)# ip route 192.168.2.0 255.255.255.0 192.168.1.2

````
<p> Forklaring: </p>

<ol> 
<li> ip route: Dette er kommandoen til at definere en static route. </li>
<li> 192.168.2.0: Dette er destinationsnetværket. </li>
<li> 255.255.255.0: Dette er subnetmasken for destinationsnetværket. </li>
<li> 192.168.1.2: Dette er next-hop IP-adressen, dvs. IP-adressen på den næste router, der skal modtage pakken for at nå destinationsnetværket.
 </li>
</ol>

<h2> Flere eksempler: </h2>

<h3> Standard Route (Default Route): </h3>

<p> En standardroute bruges til at sende pakker, hvis der ikke er nogen specifik rute defineret for destinationsnetværket. Dette er nyttigt til at videresende trafik til en gateway, som kan håndtere videre routing. </p>

```
Router(config)# ip route 0.0.0.0 0.0.0.0 192.168.1.254
```

<p> Her angiver 0.0.0.0 0.0.0.0, at enhver destination, der ikke allerede har en specifik rute, skal  </p>
<p> sendes til 192.168.1.254, som normalt vil være en gateway eller en anden router. </p>

<h3> Rute via Interface: </h3>

<p> I stedet for at angive en next-hop IP-adresse, kan du også pege på et direkte tilsluttet interface: </p>

```
Router(config)# ip route 192.168.3.0 255.255.255.0 FastEthernet0/1

```
<p> Dette fortæller routeren at sende trafik til 192.168.3.0/24 direkte ud af interface FastEthernet0/1. </p>

<h2> Kontrollér konfigurationen </h2>

<p> Du kan kontrollere, om dine static routes er konfigureret korrekt med kommandoen: </p>

```
Router# show ip route

```
<p> Dette vil vise alle de ruter, routeren kender, inklusive de statiske ruter, du har konfigureret. </p>

<p> </p>
<h></h>

