# Øvelse 3 — **O = Originate**: `network` vs. `redistribute` (Cisco)
*Lær forskellen på at annoncere præfikser med `network` (ORIGIN=**IGP**) og `redistribute` (ORIGIN=**incomplete** `?`). Forstå kravet om **RIB-match** for `network`.*

---

## 🎯 Mål & idé
- **Mål:** På **R2 (AS65020)** skal du annoncere et nyt præfiks **2.2.2.0/24** til resten af netværket på to måder og sammenligne:
  1) `network 2.2.2.0 mask 255.255.255.0` → ORIGIN = **IGP** (`i`)
  2) `redistribute static` af en route til 2.2.2.0/24 → ORIGIN = **incomplete** (`?`)
- **Forstå:** `network X mask Y` kræver, at **samme præfiks** ligger i **RIB** (routing-tabellen) med **samme mask**. Ellers annonceres det **ikke**.
- **Bonus:** Begræns `redistribute static` med **route-map + prefix-list**, så du ikke oversvømmer BGP.

---

## 🧱 Topologi-udsnit (relevant for øvelsen)
```
R1 (AS65010) —— eBGP —— R2 (AS65020) —— iBGP —— R3 (AS65020) —— eBGP —— R4 (AS65040)
          \______________________  annoncerer 2.2.2.0/24 fra R2  _____________________/
```
*R1 og R3 skal kunne se forskellen på ORIGIN-koden for præfikset 2.2.2.0/24.*

---

## 1) Metode A — Annoncér via `network` (ORIGIN = IGP)
> Vi skaber en **rigtig** route i RIB til 2.2.2.0/24 og bruger `network`-kommandoen.

### På **R2**
```cisco
conf t
! a) Skab RIB-match (vælg én af varianterne)

! Variant A1 — brug loopback med /24
interface Loopback2
 ip address 2.2.2.2 255.255.255.0

! eller Variant A2 — brug en statisk route til Null0
! ip route 2.2.2.0 255.255.255.0 Null0

! b) Annoncér i BGP med netværkskommando
router bgp 65020
 bgp log-neighbor-changes
 network 2.2.2.0 mask 255.255.255.0
end
```

**Verificér på R2:**
```cisco
show ip route 2.2.2.0
show ip bgp 2.2.2.0
```
Forvent at se `*> 2.2.2.0/24 ... i` (`i` = ORIGIN **IGP**).

**Verificér på R1 og R3:**
```cisco
show ip bgp 2.2.2.0
```
De bør se ruten fra R2 med ORIGIN **IGP** (`i`).

---

## 2) Metode B — Annoncér via `redistribute static` (ORIGIN = incomplete)
> Nu annoncerer vi **samme** præfiks som **statisk route** redistribueret i BGP. Her bliver ORIGIN **incomplete** (`?`).

### På **R2**
```cisco
conf t
! a) Lav en statisk route (hvis ikke allerede lavet i A2)
ip route 2.2.2.0 255.255.255.0 Null0

! b) (bedste praksis) Begræns redistribute til kun 2.2.2.0/24
ip prefix-list ONLY_22 permit 2.2.2.0/24

route-map REDIST_ONLY_22 permit 10
 match ip address prefix-list ONLY_22

router bgp 65020
 ! Aktiver redistribuering (vælg kontrolleret variant med route-map)
 redistribute static route-map REDIST_ONLY_22
end
```

**Verificér på R2, R1, R3:**
```cisco
show ip bgp 2.2.2.0
```
Forvent at se samme præfiks **med ORIGIN `?` (incomplete)** fra BGP.  
> Hvis både `network` **og** `redistribute` er aktive samtidig for samme præfiks, kan den ene “overskygge” den anden. Til test: slå den ene fra ad gangen, eller kig i `show ip bgp 2.2.2.0` på attributterne og hvilken der blev valgt som best.

---

## 3) Sammenlign forskelle (det her skal du observere)
- **ORIGIN-kode:**  
  - `network` → **IGP** (`i`)  
  - `redistribute` → **incomplete** (`?`)
- **Kontrol & sikkerhed:**  
  - `network`-metoden er **deterministisk** (annoncerer kun det, du eksplicit beder om, og **kun** hvis der er RIB-match).  
  - `redistribute static` kan være **farlig** uden filtrering: du kan komme til at annoncere **alle** statiske routes.
- **Best-path effekt:** ORIGIN **IGP** foretrækkes over **incomplete** ved tie med alle andre attributter.

---

## 4) Fejlsøgning (hvis du ikke ser præfikset)
- **`network` annoncerer ikke noget?**  
  - Tjek RIB-match: `show ip route 2.2.2.0` (masken **skal** matche præcist).  
  - Korrekt maske i `network`-kommandoen?
- **`redistribute static` oversvømmer for meget?**  
  - Bekræft at du bruger `route-map` + `ip prefix-list` som ovenfor.  
  - Tjek hits: `show route-map`, `show ip prefix-list`.
- **Naboer ser intet?**  
  - Tjek `show ip bgp summary` (neighbors Established?).  
  - `clear ip bgp * soft out` efter ændringer kan hjælpe.
- **Bedste rute blev ikke installeret i RIB?**  
  - `show ip bgp rib-failure` — kan pege på en bedre (AD) non-BGP route.  
  - Tjek next-hop reachability: `ping/traceroute <nexthop>`.

---

## 5) Oprydning (valgfrit)
**Hvis du vil tilbage til “clean slate”:**

```cisco
conf t
! Fjern network-linjen
router bgp 65020
 no network 2.2.2.0 mask 255.255.255.0
 no redistribute static route-map REDIST_ONLY_22
exit

! Fjern route-map og prefix-list
no route-map REDIST_ONLY_22 permit 10
no ip prefix-list ONLY_22

! Fjern loopback/route (hvis du lavede dem)
no interface Loopback2
no ip route 2.2.2.0 255.255.255.0 Null0
end
```

---

## 6) Hurtig tjekliste (copy/paste)

**Metode A — `network`:**
```cisco
R2# show ip route 2.2.2.0
R2# show ip bgp 2.2.2.0
R1# show ip bgp 2.2.2.0
R3# show ip bgp 2.2.2.0
! Forvent ORIGIN = i
```

**Metode B — `redistribute static`:**
```cisco
R2# show ip bgp 2.2.2.0
R1# show ip bgp 2.2.2.0
R3# show ip bgp 2.2.2.0
! Forvent ORIGIN = ?
```

---

## 7) Ekstra lege-ideer
- Annoncér **både** `network` og `redistribute static` for samme præfiks og se hvilken **best-path** vælges (se attributter).  
- Tilføj **MED** eller **AS-path prepend** på den ene metode og se hvordan best-path skifter.  
- Brug `aggregate-address` til at samle fx `2.2.2.0/25` og `2.2.2.128/25` til **2.2.2.0/24** og test ORIGIN på aggregatet.
