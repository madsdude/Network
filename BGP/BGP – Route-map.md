
# 🗺️ Cisco `route-map` Guide

## 📘 Hvad er en `route-map`?

En `route-map` i Cisco IOS bruges til **at matche og manipulere routinginformation**. Det er en slags "if-then"-logik, der ofte anvendes sammen med protokoller som **BGP**, **PBR (Policy-Based Routing)** eller **redistribution**.

---

## 🔧 Grundlæggende syntaks

```cisco
route-map NAVN [permit | deny] SEKVENSNUMMER
  match ...
  set ...
```

- `NAVN`: Navnet på route-map'en
- `permit` / `deny`: Tillad eller afvis, hvad der matches
- `SEKVENSNUMMER`: Angiver rækkefølgen af regler (laveste nummer først)
- `match`: Betingelser for at matche routes
- `set`: Hvad der skal ændres, hvis der er match

---

## 🧪 Eksempel 1: BGP – AS Path Prepending til sekundær linje

👉 Formål: Gør den sekundære internetforbindelse mindre attraktiv ved at bruge AS path prepending.

```cisco
route-map TO-ISP1-PRIMARY permit 10
 set local-preference 200

route-map TO-ISP2-SECONDARY permit 10
 set as-path prepend 65001 65001 65001

router bgp 65001
 address-family ipv4
  network 93.42.139.0 mask 255.255.255.248
  neighbor 93.42.139.2 remote-as 64501
  neighbor 93.42.139.2 route-map TO-ISP1-PRIMARY out
  neighbor 80.91.57.1 remote-as 64502
  neighbor 80.91.57.1 route-map TO-ISP2-SECONDARY out
```

📌 Resultat: ISP1 (primær linje) foretrækkes pga. høj local-preference, mens ISP2 (sekundær linje) nedprioriteres via AS path prepending.

---

## 🧪 Eksempel 2: Policy-Based Routing (PBR) til at sende bestemt trafik ud via primær og sekundær linje

👉 Formål: Intern trafik opdeles – nogle hoste sendes ud via primær linje, andre via sekundær.

```cisco
access-list 101 permit ip 10.0.0.0 0.0.0.255 any
access-list 102 permit ip 10.0.1.0 0.0.0.255 any

route-map PBR-PRIMARY permit 10
 match ip address 101
 set ip next-hop 10.0.0.2

route-map PBR-SECONDARY permit 10
 match ip address 102
 set ip next-hop 10.0.0.3

interface GigabitEthernet0/1
 ip policy route-map PBR-PRIMARY

interface GigabitEthernet0/2
 ip policy route-map PBR-SECONDARY
```

📌 Resultat: Netværk 10.0.0.0/24 går via primær gateway, og 10.0.1.0/24 via sekundær.

---

## 🧪 Eksempel 3: Redistribution med filtrering

👉 Formål: Kun redistribuere specifikke netværk til BGP fra OSPF.

```cisco
access-list 20 permit 192.168.0.0 0.0.255.255

route-map REDIST-OSPF permit 10
 match ip address 20

router bgp 65001
 redistribute ospf 1 route-map REDIST-OSPF
```

📌 Resultat: Kun netværk i `192.168.0.0/16` redistribueres fra OSPF til BGP.

---

## 🧩 Almindelige `match`-kommandoer

| Match-type | Eksempel |
|------------|----------|
| IP-adresse | `match ip address 100` |
| Prefix-list | `match ip address prefix-list MY-PREFIX` |
| AS-path     | `match as-path 10` |
| Community   | `match community 20` |
| Route-type  | `match route-type external` |

---

## 🛠 Almindelige `set`-kommandoer

| Set-type | Eksempel |
|----------|----------|
| Next-hop IP | `set ip next-hop 192.0.2.1` |
| Local-preference | `set local-preference 200` |
| Metric | `set metric 20` |
| AS-path prepend | `set as-path prepend 65001 65001` |
| Community | `set community 65001:100 additive` |

---

## 🔄 Flere sektioner

Du kan lave flere sektioner med forskellige `match`-regler:

```cisco
route-map CUSTOM-OUT permit 10
 match ip address 101
 set local-preference 150

route-map CUSTOM-OUT permit 20
 match ip address 102
 set local-preference 50
```

📌 Lavere sekvensnumre behandles først.

---

## 🚨 Husk

- Uden `match` matcher en `route-map` **alt**.
- Uden `set` ændrer den **intet**, men kan bruges til at tillade/afvise.
- Brug `show route-map` og `debug ip policy` eller `show ip bgp` til fejlsøgning.
