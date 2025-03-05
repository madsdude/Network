<h1>Static Route Forklaring med Eksempler</h1>
  <p>En <strong>static route</strong> er en manuel indtastning af en rute i en router eller switch, der fortæller enheden, hvilken vej en pakke skal tage for at nå et bestemt netværk. I modsætning til dynamiske routing-protokoller, som automatisk finder og opdaterer ruter, forbliver statiske ruter konstant, indtil de manuelt ændres eller slettes. Static routes er nyttige i mindre netværk eller til specifikke opgaver, hvor ruten skal være forudsigelig og ikke ændre sig dynamisk.</p>
    
  <h2>Grundlæggende Konfiguration af en Static Route</h2>
    <p>Lad os sige, at du har to netværk:</p>
    <ul>
        <li>Netværk A: <code>192.168.1.0/24</code></li>
        <li>Netværk B: <code>192.168.2.0/24</code></li>
    </ul>
    <p>Routeren, der forbinder de to netværk, har to interfaces:</p>
    <ul>
        <li>Interface FastEthernet 0/0 (Fa0/0) er tilsluttet Netværk A med IP-adressen <code>192.168.1.1</code>.</li>
        <li>Interface FastEthernet 0/1 (Fa0/1) er tilsluttet Netværk B med IP-adressen <code>192.168.2.1</code>.</li>
    </ul>

  <h2>Opsætning af en Static Route</h2>
    <p>For at sikre, at routeren ved, hvordan man sender trafik fra Netværk A til Netværk B, kan du konfigurere en static route.</p>

  <h3>Eksempel på Cisco kode:</h3>
    <pre><code>Router(config)# ip route 192.168.2.0 255.255.255.0 192.168.1.2</code></pre>
    
   <h4>Forklaring:</h4>
    <ul>
        <li><strong>ip route</strong>: Dette er kommandoen til at definere en static route.</li>
        <li><strong>192.168.2.0</strong>: Dette er destinationsnetværket.</li>
        <li><strong>255.255.255.0</strong>: Dette er subnetmasken for destinationsnetværket.</li>
        <li><strong>192.168.1.2</strong>: Dette er next-hop IP-adressen, dvs. IP-adressen på den næste router, der skal modtage pakken for at nå destinationsnetværket.</li>
    </ul>

   <h3>Flere Eksempler:</h3>
    <h4>1. Standard Route (Default Route):</h4>
    <p>En standardroute bruges til at sende pakker, hvis der ikke er nogen specifik rute defineret for destinationsnetværket. Dette er nyttigt til at videresende trafik til en gateway, som kan håndtere videre routing.</p>
    <pre><code>Router(config)# ip route 0.0.0.0 0.0.0.0 192.168.1.254</code></pre>
    <p>Her angiver <code>0.0.0.0 0.0.0.0</code>, at enhver destination, der ikke allerede har en specifik rute, skal sendes til <code>192.168.1.254</code>, som normalt vil være en gateway eller en anden router.</p>

  <h4>2. Rute via Interface:</h4>
    <p>I stedet for at angive en next-hop IP-adresse, kan du også pege på et direkte tilsluttet interface:</p>
    <pre><code>Router(config)# ip route 192.168.3.0 255.255.255.0 FastEthernet0/1</code></pre>
    <p>Dette fortæller routeren at sende trafik til 192.168.3.0/24 direkte ud af interface <code>FastEthernet0/1</code>.</p>

   <h2>Kontrollér Konfigurationen</h2>
    <p>Du kan kontrollere, om dine static routes er konfigureret korrekt med kommandoen:</p>
    <pre><code>Router# show ip route</code></pre>
    <p>Dette vil vise alle de ruter, routeren kender, inklusive de statiske ruter, du har konfigureret.</p>

   <h2>Fordele og Ulemper ved Static Routes</h2>
    <ul>
        <li><strong>Fordele:</strong>
            <ul>
                <li><strong>Forudsigelighed:</strong> Ruterne ændrer sig ikke, medmindre du manuelt ændrer dem.</li>
                <li><strong>Lav belastning:</strong> Static routes kræver ingen beregninger, så de belaster ikke routerens CPU.</li>
            </ul>
        </li>
        <li><strong>Ulemper:</strong>
            <ul>
                <li><strong>Mangel på skalerbarhed:</strong> Det er upraktisk i større netværk, hvor dynamiske routing-protokoller er mere effektive.</li>
                <li><strong>Manglende failover:</strong> Hvis en link går ned, skal ruten manuelt opdateres, medmindre du bruger flere ruter til redundans.</li>
            </ul>
        </li>
    </ul>
    <p>Static routes er nyttige i mindre netværk, til simple routing-scenarier, eller som backup-ruter i mere komplekse netværk. De giver kontrol og stabilitet, men kræver manuel opsætning og vedligeholdelse.</p>
</body>
</html>

# 📌 Forstå IP Route Kommandoen
Når du ser en kommando som:

ip route 10.16.0.0 255.255.0.0 10.10.10.2

yaml
Kopiér
Rediger

Så betyder det følgende:

## 🔹 Struktur:
| Parameter       | Beskrivelse |
|----------------|------------|
| `ip route`     | Opretter en statisk route |
| `10.16.0.0`    | **Destination subnet** (hvilket netværk der routes til) |
| `255.255.0.0`  | **Subnetmaske** (bestemmer netværkets størrelse) |
| `10.10.10.2`   | **Next-hop IP** (hvor trafikken sendes hen) |

---

## 🖥️ Diagram over rutingen

[Router] --- (10.10.10.2) ---> [Next-hop Router]
--- (10.16.0.0/16 netværk)

yaml
Kopiér
Rediger

### 📖 Forklaring:
- **Routeren** lærer, at alle pakker til `10.16.0.0/16` skal sendes til `10.10.10.2`.
- Subnetmasken `255.255.0.0` betyder, at **alle IP-adresser fra `10.16.0.0` til `10.16.255.255`** er dækket.
- `10.10.10.2` er **næste hop**, som routeren sender trafikken videre til.

---

## ✅ Eksempel på anvendelse
Hvis en enhed sender en pakke til `10.16.5.25`, vil routeren:
1. Matche `10.16.5.25` til `10.16.0.0/16`.
2. Sende den videre til `10.10.10.2` for yderligere routing.

---

## 📌 Huskeregler
- **Destination IP + Subnetmaske** bestemmer, hvilket netværk der dækkes.
- **Next-hop IP** er routerens guide til, hvor trafikken skal hen.
- Hvis der ikke er en specifik route, bruges standardruten (`0.0.0.0/0`).

---


