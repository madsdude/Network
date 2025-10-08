# Øvelse 7 — **P = Path Type**: eBGP > iBGP (ved tie)
*Demonstrér at når alle andre attributter (Weight, Local-Pref, ORIGIN, AS_PATH, MED) er ens, så foretrækkes en **eBGP**-lært rute over en **iBGP**-lært rute.*

---

## 🎯 Mål & idé
- **Mål:** På **R2 (AS65020)** skal præfikset **4.4.4.4/32** være lært både via **eBGP (fra R4)** og via **iBGP (fra R3)**, men **best-path** skal blive **eBGP**-vejen.
- **Hvorfor?** I best-path (W L O AS O M **P** R) evalueres Path Type (eBGP vs iBGP) efter MED. Hvis alt andet er ens, vinder **eBGP**.
- **Forudsætning:** Sørg for, at ingen gamle policies (Weight/Local-Pref/MED/Prepend) påvirker valget for 4.4.4.4/32.

---

## 🧱 Relevant topologi
```
R3 (AS65020) ---iBGP--- R2 (AS65020) ---eBGP--- R4 (AS65040)
     \_____________________ 4.4.4.4/32 ____________________/

R4: Ejer/annoncerer 4.4.4.4/32 (via 'network' eller redistribueret static)
R3: Lærer 4.4.4.4/32 fra R4 via R2 og sender det videre i iBGP til R2
R2: Ser to paths til 4.4.4.4/32: eBGP (direkte) og iBGP (via R3)
```

---

## 1) Sørg for at **R4** annoncerer 4.4.4.4/32
> Hvis du allerede har 4.4.4.4 på R4, spring til trin 2.

### På **R4 (AS65040)**
```cisco
conf t
interface Loopback1
 ip address 4.4.4.4 255.255.255.255

router bgp 65040
 bgp log-neighbor-changes
 network 4.4.4.4 mask 255.255.255.255
end
```

**Verificér på R4:**
```cisco
show ip bgp 4.4.4.4
```

---

## 2) Sørg for at **R3** modtager 4.4.4.4/32 (via iBGP-transit gennem R2)
> Du har i forvejen iBGP mellem R2–R3, og R2 har eBGP til R4. R2 bør derfor modtage 4.4.4.4/32 fra R4 og (med standard iBGP-regler + evt. RR) annoncere den til R3. Hvis R2 **ikke** annoncerer iBGP til R3, så overvej midlertidigt at gøre R2 til **route-reflector** for R3, eller opsæt full-mesh.

### (Hvis nødvendigt) På **R2** – gør R3 til RR-klient
```cisco
conf t
router bgp 65020
 neighbor 10.0.23.3 route-reflector-client
end
```
> Dette gør, at R2 må **reflektere** ruter, den har lært fra eBGP (R4), videre til iBGP-klienten (R3).

**Verificér på R3:**
```cisco
show ip bgp 4.4.4.4
```

---

## 3) R2 skal se **to** veje til 4.4.4.4/32
**På R2:**
```cisco
show ip bgp 4.4.4.4
```
**Forvent:** 
- Én path via **10.0.24.4** (eBGP fra R4)
- Én path via **10.0.23.3** (iBGP fra R3)

*Hvis du kun ser den ene: tjek at R3 faktisk modtager præfikset fra R2 og sender det i iBGP tilbage til R2 (RR/full-mesh), eller at ingen `next-hop`-problemer stopper udvekslingen.*

---

## 4) Hold **alle andre attributter ens**
- **Weight:** Ingen special-politikker på R2 mod 10.0.23.3/10.0.24.4
- **Local-Pref:** Lad standard (100) gælde for begge paths
- **ORIGIN:** Sørg for samme (typisk `i` via `network` på R4). Hvis R3 sender videre med samme ORIGIN (typisk uændret), er de ens
- **AS_PATH:** Begge bør være ét hop set fra R2 (eBGP fra 65040 vs iBGP fra 65020 uden ekstern AS i path)
- **MED:** Ingen sætning (eller ens), og irrelevant ved forskellig nabo-AS i denne sammenhæng

---

## 5) Verificér at **eBGP** bliver **best-path** på R2
**På R2:**
```cisco
show ip bgp 4.4.4.4
show ip route 4.4.4.4
```
**Forventet:**
- I `show ip bgp 4.4.4.4` er **vejen via 10.0.24.4** markeret `*>` (best).  
- I `show ip route 4.4.4.4` peger next-hop mod **10.0.24.4**.

---

## 6) Fejlsøgning
- **Ser du ikke to paths?**  
  - R3 ser måske ikke 4.4.4.4/32 (tjek på R3).  
  - R2 reflekterer måske ikke eBGP→iBGP (sæt midlertidigt RR som vist i trin 2).  
  - `next-hop-self` på R2 overfor R3 er normalt nødvendigt for reachability.
- **Best bliver iBGP alligevel:**  
  - Tjek for skjulte politikker: `show run | sec route-map|neighbor|local-preference|weight|metric`.  
  - Fjern/fallback policies fra forrige øvelser for dette prefix.  
  - Se attributter side om side: `show ip bgp 4.4.4.4`.
- **RIB installerer ikke BGP-ruten:**  
  - `show ip bgp rib-failure` for årsag (bedre non-BGP rute?).  
  - Tjek next-hop reachability (ping/traceroute).

---

## 7) Oprydning (valgfrit)
```cisco
! Fjern RR-klient (hvis du tilføjede det)
conf t
router bgp 65020
 no neighbor 10.0.23.3 route-reflector-client
end
```

---

## 8) Ekstra lege-ideer
- Vend testen om: forsøg at få iBGP til at vinde ved at give iBGP-vejen **højere Local-Pref**, og se at LP slår **Path Type**.  
- Tilføj **Weight** på R2 for iBGP-vejen og observer at Weight (lokal) slår Path Type.  
- Indfør **MED** (samme nabo-AS, eller `always-compare-med`) og se hvor i rækkefølgen Path Type står.
