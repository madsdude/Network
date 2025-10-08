# Øvelse 4 — **AS = AS_PATH & Prepending** (Cisco)
*Styr **indgående** trafikvalg hos en fjern-AS ved at **forlænge** din annoncerede AS-sti (AS-path prepending). Korteste AS_PATH vinder ved tie på Weight/Local-Pref/Origin.*

---

## 🎯 Mål & idé
- **Mål:** Få **R1 (AS65010)** til at **foretrække vejen via R2 (AS65020)** til præfikset **3.3.3.3/32**, i stedet for vejen via **R4 (AS65040)** — ved at lave **AS-path prepending** på **R4**.
- **Hvorfor?** Når en ekstern AS (R1) har to lige gode eBGP-veje til samme præfiks (fra R2 og R4), vil den normalt vælge den med **kortest AS_PATH**. Hvis vi **forlænger** stien fra R4, vil R1 vælge **R2** som “bedre”/kortere vej.
- **Forudsætning:** R1 skal se **to eBGP-veje** til **3.3.3.3/32** — én via R2 og én via R4.

---

## 🧱 Topologi-udsnit (relevant for øvelsen)
```
R3 (AS65020) ---iBGP--- R2 (AS65020) ---eBGP--- R1 (AS65010)

R3 (ejer 3.3.3.3/32)
                         \---eBGP--- R4 (AS65040) ---/
```

*R1 lærer 3.3.3.3/32 både via **R2** (AS-sti: `65020`) og via **R4** (AS-sti: `65040 65020` eller `65040`). Vi gør stien via **R4** **længere** med prepends, så **R1** vælger vejen via **R2**.*

---

## 1) Sørg for **to** veje til 3.3.3.3/32 på **R1**
- **R3** annoncerer 3.3.3.3/32 (allerede sat i dine tidligere øvelser via `network 3.3.3.3/32`).
- **R2** videresender ruten til **R1** (eBGP).
- **R4** skal også annoncere **3.3.3.3/32** til **R1**. Vi kan “fake” det ved at lære ruten via R2/R3 (normalt sker det automatisk hvis R4 lærer ruten fra R2). Hvis ikke, kan du midlertidigt annoncere 3.3.3.3/32 fra R4 (Null0) for at skabe to veje på R1.

**(Hvis nødvendigt på R4 – ikke altid påkrævet i din lab):**
```cisco
conf t
ip route 3.3.3.3 255.255.255.255 Null0
router bgp 65040
 network 3.3.3.3 mask 255.255.255.255
end
```

**Verificér på R1:**
```cisco
show ip bgp 3.3.3.3
show ip route 3.3.3.3
```
> Du bør se **to** paths: én via **10.0.12.2 (R2)** og én via **10.0.24.4 (R4)**. Hvis kun én path ses, tjek reachability/policy på R2/R4.

---

## 2) Lav **AS-path prepending** på **R4** (ud mod R1)
> Vi forlænger stien **kun for præfikset 3.3.3.3/32** for at undgå sideeffekter.

### På **R4**
```cisco
conf t
ip prefix-list NET33 permit 3.3.3.3/32

route-map PREPEND_OUT permit 10
 match ip address prefix-list NET33
 set as-path prepend 65040 65040 65040

router bgp 65040
 neighbor 10.0.24.2 route-map PREPEND_OUT out
end
```

> **Forklaring:** `set as-path prepend <AS> <AS> <AS>` gentager **dit eget AS** i stien. Tre prepends er et typisk udgangspunkt (kan justeres).

**Anvend ændringen:**
```cisco
clear ip bgp 10.0.24.2 soft out
```

---

## 3) Verificér på **R1**
```cisco
show ip bgp 3.3.3.3
show ip route 3.3.3.3
```
**Forvent:** 
- AS-sti via **R4** er nu længere, fx `65040 65040 65040 65040 ...`  
- **Best (`*>`)** bør være vejen via **R2** (kortere AS_PATH).  
- `show ip route 3.3.3.3` skal nu pege mod **10.0.12.2** (R2).

> Hvis den ikke skifter: Tjek om andre attributter (Local-Pref på R1, MED, eBGP/iBGP forskel, etc.) påvirker. AS_PATH evalueres **efter** Weight og Local-Pref, men **før** Origin/MED i standard-reglen (W L O **AS** O M P R).

---

## 4) Variant: Prepending på **R2** i stedet
- Hvis du vil have R1 til at foretrække **R4** i stedet, kan du **prepend’e ud fra R2** på præfikset **3.3.3.3/32** (mod R1):
```cisco
! På R2 (AS65020) – gør vejen via R2 “længere”
conf t
ip prefix-list NET33 permit 3.3.3.3/32
route-map PREPEND_R2_OUT permit 10
 match ip address prefix-list NET33
 set as-path prepend 65020 65020 65020

router bgp 65020
 neighbor 10.0.12.1 route-map PREPEND_R2_OUT out
end
```
> **Advarsel:** Det påvirker **alle** der modtager ruterne fra R2 (her R1). Brug altid **prefix-lister** for granularitet.

---

## 5) Fejlsøgning
- **R1 ser kun én path:**  
  - Tjek at både R2 og R4 **annoncerer** 3.3.3.3/32 til R1.  
  - `show ip bgp summary` på R1 (sessions Established?).  
  - `show ip bgp 3.3.3.3` på R2/R4 (bliver det annonceret? `advertised-routes` kræver evt. `soft-reconfiguration inbound`).

- **Prepends ændrer intet:**  
  - Tjek om **Local-Pref** på R1 favorisere en vej (Local-Pref vinder over AS_PATH).  
  - Tjek **MED** (sammenlignes kun hvis nabo-AS er samme).  
  - Husk at **eBGP > iBGP** (P-reglen). Begge veje mod R1 er eBGP i denne øvelse, så fair.

- **RIB installerer ikke best-path:**  
  - `show ip bgp rib-failure` for at se årsag (kan være mere “attraktiv” non-BGP rute).  
  - Tjek next-hop reachability (ping/traceroute nexthop).

- **Policy bider ikke:**  
  - Er route-map bundet på **korrekt neighbor** i **rigtig retning** (`out`)?  
  - Kør `clear ip bgp <neighbor> soft out` efter ændringer.

---

## 6) Oprydning (valgfrit)
```cisco
! På R4 – fjern prepending
conf t
router bgp 65040
 no neighbor 10.0.24.2 route-map PREPEND_OUT out
no route-map PREPEND_OUT permit 10
no ip prefix-list NET33
end

! Hvis du “fakede” 3.3.3.3/32 på R4:
conf t
router bgp 65040
 no network 3.3.3.3 mask 255.255.255.255
no ip route 3.3.3.3 255.255.255.255 Null0
end
```

---

## 7) Hurtig tjekliste (copy/paste)

**Før prepends:**
```cisco
R1# show ip bgp 3.3.3.3
R1# show ip route 3.3.3.3
```

**Efter prepends (på R4→R2):**
```cisco
R4# clear ip bgp 10.0.24.2 soft out
R1# show ip bgp 3.3.3.3
R1# show ip route 3.3.3.3
! Forvent best via R2 (kortere AS_PATH)
```

---

## 8) Ekstra lege-ideer
- Varier antallet af prepends (1–6) og se hvornår R1 skifter valg.  
- Kombinér med **MED** fra R2/R4 og se hvilke regler der “vinder” i best-path.  
- Brug **community** på R3 (f.eks. `no-export`) og se hvordan det påvirker udbredelsen.  
- Annoncér et nyt testprefix (fx 33.33.33.0/24) og lav forskellig politik per prefix.
