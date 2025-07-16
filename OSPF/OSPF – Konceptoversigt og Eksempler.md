
# 📡 OSPF – Konceptoversigt og Eksempler

Denne dokumentation dækker centrale OSPF-emner med fokus på:
- LSA-typer
- ABR og ASBR-rolle
- Type 3 LSA-regler
- Route filtering
- OSPF timers
- Summarization og ECMP

---

## 🔢 LSA-Typer (Link State Advertisements)

| Type | Navn             | Beskrivelse                                                                 |
|------|------------------|------------------------------------------------------------------------------|
| 1    | Router LSA       | Skabt af hver router; beskriver links, interfaces og naboer indenfor **ét område** |
| 2    | Network LSA      | Skabt af DR på broadcast-netværk; beskriver alle routere på segmentet       |
| 3    | Summary LSA      | Skabt af **ABR** for at annoncere netværk til **andre områder**              |
| 4    | ASBR Summary LSA | Identificerer, hvor ASBR er placeret (for ekstern rute-opløsning)           |
| 5    | External LSA     | Skabt af **ASBR** for at annoncere **eksterne ruter** (f.eks. BGP/static)    |
| 7    | NSSA External    | Som Type 5, men bruges i **Not-So-Stubby Areas**                             |

---

## 🧭 ABR og ASBR

- **ABR (Area Border Router)**  
  - Har interfaces i **flere OSPF-områder**
  - Skaber **Type 3 og Type 4 LSAs**
  - Anvender `area x range` for summarization

- **ASBR (Autonomous System Boundary Router)**  
  - Importerer ruter fra **eksterne protokoller** (BGP, static, osv.)
  - Skaber **Type 5 (eller Type 7 i NSSA)** LSAs

---

## 🧩 Type 3 LSA – Regler

En ABR skaber Type 3 LSAs ifølge disse regler:

1. Type 1 LSAs modtaget fra et område → skaber Type 3 LSAs i backbone og andre områder.
2. Type 3 LSAs modtaget fra Area 0 → skabes videre i nonbackbone areas.
3. Type 3 LSAs modtaget fra **nonbackbone areas** → **bliver kun lagt i LSDB**, men **ikke gensendt**.

Eksempel:
```bash
area 1 range 10.1.0.0 255.255.252.0
```
> ABR’en opsummerer netværk 10.1.0.0–10.1.3.255 som én Type 3 LSA.

---

## 🛑 Route Filtering på ABR

### 🎯 For at filtrere Type 3 LSAs på ABR:

| Metode         | Beskrivelse |
|----------------|-------------|
| `distribute-list` | ✅ Den **korrekte metode** til at **filtrere Type 3 LSAs** på ABR |
| `prefix-list`     | ❌ Brugt ved redistribution, **ikke på interarea LSA filtering** |
| `route-map`       | ❌ Understøttes ikke af OSPF ABR |
| `summarization`   | Samler ruter, **filtrerer ikke** |

Eksempel:
```bash
router ospf 1
 distribute-list prefix DENY_LSA in
```

---

## ⏱️ OSPF Timers

- **LSA Refresh timer**: 1800 sekunder
- **LSA MaxAge (purge)**: **3600 sekunder**
  - Efter 3600 sekunder uden opdatering fjernes LSA fra LSDB

---

## 🧮 Equal-Cost Multipath (ECMP)

OSPF tillader routing over flere lige gode veje.

| Indstilling       | Standardværdi |
|-------------------|----------------|
| `maximum-paths`   | **4** (kan øges til 16) |

Eksempel:
```bash
router ospf 1
 maximum-paths 4
```

---

## 📦 Eksempel på Samlet Opsummering

```bash
router ospf 1
 network 192.168.0.0 0.0.255.255 area 0
 area 1 range 192.168.32.0 255.255.252.0
 distribute-list prefix BLOCK_ROUTES in
```

---

## ✅ Hurtige FAQ

- **Hvordan bliver en router ABR?**
  → Når den har interfaces i **flere områder**, fx Area 0 og Area 1.

- **Hvilken LSA bruges til ekstern rute?**
  → **Type 5** (eller Type 7 i NSSA)

- **Hvordan filtrerer man Type 3 LSAs på ABR?**
  → Med en **distribute-list**.

- **Hvornår slettes en LSA fra LSDB?**
  → Når den når **3600 sekunder** uden opdatering.

---
