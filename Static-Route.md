<p>  En static route er en manuel indtastning af en rute i en router eller switch, der fortæller enheden, hvilken vej en pakke skal tage for at nå et bestemt netværk. I modsætning til dynamiske routing-protokoller, som automatisk finder og opdaterer ruter, forbliver statiske ruter konstant, indtil de manuelt ændres eller slettes. Static routes er nyttige i mindre netværk eller til specifikke opgaver, hvor ruten skal være forudsigelig og ikke ændre sig dynamisk. </p>

<h1> Grundlæggende konfiguration af en static route </h1>

<p> Lad os sige, at du har to netværk: </p>
<ol></ol>
 <li>Netværk A: 192.168.1.0/24</li>
 <li>Netværk B: 192.168.2.0/24 </li>
</ol>

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

<p> </p>
<h></h>

