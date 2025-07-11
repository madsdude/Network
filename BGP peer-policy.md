
# 🔁 Cisco BGP `peer-policy` Guide

## 📘 Hvad er en `peer-policy`?

En `peer-policy` er en **template-baseret konfiguration** i BGP, der gør det nemmere at genbruge politikker på flere naboer. Det bruges især i setups med **mange BGP-peers**, som fx i datacentre, EVPN eller MPLS.

Med `peer-policy` kan du centralt definere BGP politikker (som route-maps, as-path prepend osv.) og anvende dem via `inherit` på dine neighbors.

---

## 🧩 Syntax

```cisco
bgp peer-policy NAVN
  route-map NAVN in|out
  set ...
```

På nabo:

```cisco
router bgp <ASN>
 address-family ipv4
  neighbor <IP> inherit peer-policy NAVN
```

---

## 🧪 Eksempel: Primær og sekundær linje med `peer-policy`

👉 Formål: Gør ISP1 til **primær linje**, ISP2 til **sekundær linje**, ved at manipulere med AS path og local-preference via peer-policies.

### 🔧 Peer-policies

```cisco
bgp peer-policy DC_POLICY_PRIMARY
  route-map SET-HIGH-LOCALPREF out
  description Primær linje

bgp peer-policy DC_POLICY_SECONDARY
  route-map PREPEND-AS-PATH out
  description Sekundær linje
```

### 🔧 Route-maps

```cisco
route-map SET-HIGH-LOCALPREF permit 10
 set local-preference 200

route-map PREPEND-AS-PATH permit 10
 set as-path prepend 65001 65001 65001
```

### 🔧 BGP konfiguration med `inherit`

```cisco
router bgp 65001
 address-family ipv4
  neighbor 93.42.139.2 remote-as 64501
  neighbor 93.42.139.2 inherit peer-policy DC_POLICY_PRIMARY

  neighbor 80.91.57.1 remote-as 64502
  neighbor 80.91.57.1 inherit peer-policy DC_POLICY_SECONDARY
```

📌 Resultat:
- **ISP1** får høj `local-preference` → foretrækkes udgående
- **ISP2** har prependet AS-path → nedprioriteres indgående

---

## 🛠 Fordele ved peer-policy

| Fordel | Beskrivelse |
|--------|-------------|
| Skalerbarhed | Tilføj politik én gang, brug på mange peers |
| Overskuelighed | Mindre fejlrisiko ved gentagelser |
| Konsistens | Ens konfiguration på tværs af peers |

---

## 🔄 Kombinér med `peer-session`

Du kan også kombinere `peer-policy` med `peer-session`, som konfigurerer neighbor-parametre (AS, timers, passwords osv.):

```cisco
bgp peer-session MY-SESS
  remote-as 64501
  update-source Loopback0

router bgp 65001
 neighbor 93.42.139.2 inherit peer-session MY-SESS
 neighbor 93.42.139.2 inherit peer-policy DC_POLICY_PRIMARY
```

---

## ✅ Opsummering

- Brug `peer-policy` til at strukturere route-maps og policies
- Brug det til at differentiere mellem **primær og sekundær** ISP
- Let at vedligeholde og ændre i stor skala

---

## 🧪 Test og fejlsøgning

```cisco
show bgp ipv4 unicast neighbors 93.42.139.2 inherited-config
show run | section bgp
```

Vil du også have en version med EVPN eller i en cloud/fabric kontekst?
