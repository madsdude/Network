
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

## 🧪 Eksempel 1: BGP – AS Path Prepending

👉 Formål: Gør en rute mindre attraktiv for indgående trafik.

```cisco
route-map PREPEND-ISP2 permit 10
 set as-path prepend 65001 65001 65001

router bgp 65001
 address-family ipv4
  neighbor 1.1.1.1 route-map PREPEND-ISP2 out
```

📌 Resultat: Når vi annoncerer routes til ISP2, tilføjes 3 ekstra AS-numre, hvilket gør ruten mindre attraktiv.

---

## 🧪 Eksempel 2: Policy-Based Routing (PBR)

👉 Formål: Tving bestemt trafik ud gennem en alternativ gateway.

```cisco
access-list 101 permit ip 10.0.0.100 0.0.0.0 any

route-map PBR-TO-R2 permit 10
 match ip address 101
 set ip next-hop 10.0.0.2

interface GigabitEthernet0/1
 ip policy route-map PBR-TO-R2
```

📌 Resultat: Pakker fra 10.0.0.100 sendes ud via gateway `10.0.0.2` uanset default route.

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
