# Øvelse 5 — **O = ORIGIN-kode** (IGP > EGP > Incomplete)
*Sammenlign to annonceringer af **det samme prefix** med forskellig **ORIGIN** og se, at **IGP** (`i`) vinder over **incomplete** (`?`) når alle andre attributter er ens.*

---

## 🎯 Mål & idé
- **Mål:** Få **R2 (AS65020)** til at modtage **1.1.1.0/24** fra **både R1 og R4**, men vælge **R1’s** vej, fordi den har **ORIGIN = IGP** (`i`), mens **R4’s** vej har **ORIGIN = incomplete** (`?`).
- **Hvorfor?** I best-path-ordenen (W L O AS O M P R) slår **ORIGIN** (`IGP` > `EGP` > `incomplete`) igennem, når Weight, Local-Pref og AS_PATH er ens.
- **Forudsætning:** Sørg for at der **ikke** ligger rester fra tidligere øvelser, fx Local-Pref- eller Weight-politikker, som kan skævvride valget.

> Bemærk: Vi holder **AS_PATH-længden ens** (begge veje er én hop væk fra R2: via R1 = `65010`, via R4 = `65040`) og vi rører ikke MED/communities her.

---

## 🧱 Topologi-udsnit (relevant for øvelsen)
```
R1 (AS65010) ---eBGP--- R2 (AS65020) ---iBGP--- R3 (AS65020)
                                   \
                                    \---eBGP--- R4 (AS65040)

Testprefix: 1.1.1.0/24 (samme prefix annonceres fra R1 og R4)
R1: ORIGIN = IGP (via 'network')
R4: ORIGIN = incomplete (via 'redistribute static')
```

---

## 1) R1 annoncerer **1.1.1.0/24** med **ORIGIN = IGP**
> `network` kræver **RIB-match** på **præcis** prefix/mask.

### På **R1 (AS65010)**
```cisco
conf t
! Skab RIB-match til 1.1.1.0/24
ip route 1.1.1.0 255.255.255.0 Null0
!
router bgp 65010
 bgp log-neighbor-changes
 ! eBGP-nabo til R2 (10.0.12.2) findes allerede i dit setup
 network 1.1.1.0 mask 255.255.255.0
end
```

**Verificér (R1):**
```cisco
show ip route 1.1.1.0
show ip bgp 1.1.1.0
```
> Du bør se ruten i BGP med `i` som ORIGIN.

---

## 2) R4 annoncerer **samme prefix** via **redistribute static** (ORIGIN = `?`)
> Vi redistribuerer **kun** 1.1.1.0/24 (med route-map), for at undgå at lække andre statics.

### På **R4 (AS65040)**
```cisco
conf t
! Statisk route (RIB-match) – nødvendig for at kunne redistribuere
ip route 1.1.1.0 255.255.255.0 Null0

! Begrænsning til kun 1.1.1.0/24
ip prefix-list ONLY_11 permit 1.1.1.0/24
route-map REDIST_ONLY_11 permit 10
 match ip address prefix-list ONLY_11

router bgp 65040
 bgp log-neighbor-changes
 ! eBGP-nabo til R2 (10.0.24.2) findes allerede i dit setup
 redistribute static route-map REDIST_ONLY_11
end
```

**Verificér (R4):**
```cisco
show ip bgp 1.1.1.0
```
> ORIGIN skal vises som `?` (incomplete).

---

## 3) Verificér på **R2**
```cisco
! (valgfrit) genanvend politikker uden at flap'e sessioner
clear ip bgp 10.0.12.1 soft in
clear ip bgp 10.0.24.4 soft in

show ip bgp 1.1.1.0
show ip route 1.1.1.0
```

**Forventet:**
- `show ip bgp 1.1.1.0` viser **to** paths (via R1 og via R4).  
  - Vejen via **R1** har **ORIGIN `i`**.  
  - Vejen via **R4** har **ORIGIN `?`**.  
- Best-path (`*>`) er **R1**.  
- I RIB: `show ip route 1.1.1.0` peger mod **10.0.12.1** (R1).

> Hvis best-path **ikke** bliver via R1, ligger der sandsynligvis **Weight/Local-Pref** fra en tidligere øvelse. Fjern disse først (se “Fejlsøgning”).

---

## 4) Hvad sker der “under motorhjelmen”?
- **Best-path rækkefølge:** Weight → Local-Pref → **ORIGIN** → AS_PATH → MED → (eBGP/iBGP) → RID.  
- Når **Weight** (lokal) og **Local-Pref** (AS-intern) er ens, sammenlignes **ORIGIN**, hvor **IGP (i)** slår **incomplete (?)**.  
- Derfor vælges R1’s vej, selv om AS_PATH-længden er den samme (begge = 1).

---

## 5) Fejlsøgning & almindelige faldgruber
- **Ser du kun én path på R2?**  
  - Tjek at **både R1 og R4** faktisk annoncerer 1.1.1.0/24 (`show ip bgp 1.1.1.0` på dem).  
  - Tjek sessions: `show ip bgp summary` på R2 (Established?).

- **Best-path går stadig via R4:**  
  - **Fjern gamle policies** på R2 fra tidligere øvelser:  
    ```cisco
    conf t
    router bgp 65020
     no neighbor 10.0.23.3 route-map PREFER_R3_WEIGHT in
     no neighbor 10.0.12.1 route-map LP_FROM_R1 in
     no neighbor 10.0.12.1 route-map LP_FROM_R1_ONLY9 in
    end
    clear ip bgp * soft in
    ```
  - Tjek AS_PATH-længder — er de **ens**? (begge ét hop).  
  - MED sammenlignes kun hvis **samme nabo-AS**; her er de **forskellige** (65010 vs 65040), så MED bør være irrelevant.

- **BGP-ruten kom ikke i RIB (intet i `show ip route`)**  
  - `show ip bgp rib-failure` kan afsløre en mere attraktiv non-BGP rute.  
  - Tjek next-hop reachability (ping/traceroute).

---

## 6) Oprydning (valgfrit)
```cisco
! På R1 – stop annoncering
conf t
router bgp 65010
 no network 1.1.1.0 mask 255.255.255.0
no ip route 1.1.1.0 255.255.255.0 Null0
end

! På R4 – stop redistribuering
conf t
router bgp 65040
 no redistribute static route-map REDIST_ONLY_11
no route-map REDIST_ONLY_11 permit 10
no ip prefix-list ONLY_11
no ip route 1.1.1.0 255.255.255.0 Null0
end
```

---

## 7) Hurtig tjekliste (copy/paste)
**Før test:**
```cisco
R2# show ip bgp summary
```

**Under/efter test:**
```cisco
R1# show ip bgp 1.1.1.0
R4# show ip bgp 1.1.1.0

R2# clear ip bgp 10.0.12.1 soft in
R2# clear ip bgp 10.0.24.4 soft in
R2# show ip bgp 1.1.1.0
R2# show ip route 1.1.1.0
! Forvent: best via R1 (ORIGIN i)
```

---

## 8) Ekstra lege-ideer
- Skift R4 til også at bruge `network 1.1.1.0/24` (ORIGIN `i`) og se, at valget **ikke længere** afgøres af ORIGIN — så tager BGP næste kriterium (**AS_PATH**).  
- Lad R1 prepend’e AS_PATH og se om R2 så vælger R4 **trods** ORIGIN `i` på R1 (fordi **AS_PATH** kommer efter ORIGIN i beslutningsrækkefølgen).
