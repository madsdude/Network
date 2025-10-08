# Øvelse 6 — **M = MED (Multi-Exit Discriminator)** (lavest vinder)
*Styr **indgående trafik** fra en nabo-AS ved at annoncere en **lavere MED** på de præfikser, du ønsker at de skal tage ind via en bestemt egress. **Lavest MED vinder**.*

> **Vigtigt:** Standard BGP sammenligner **kun MED når ruterne er modtaget fra samme nabo-AS**. I vores lab modtager **R1** ruter både fra **AS65020 (R2)** og **AS65040 (R4)** — **forskellige AS**. Derfor bruger vi to varianter:
> - **Variant A (nem):** Slå **`bgp always-compare-med`** til på **R1** (tving sammenligning af MED mellem forskellige nabo-AS).
> - **Variant B (ren BGP-standard):** Lad **begge veje komme fra **samme** AS (fx to eBGP-naboer fra AS65020). Det kræver en ekstra router (R5) eller at du midlertidigt sætter R4 i AS65020.

---

## 🎯 Mål & idé
- **Mål:** Få **R1 (AS65010)** til at vælge **vejen via R2 (AS65020)** til **3.3.3.3/32** ved at annoncere en lavere **MED** fra **R2** end fra **R4**.
- **Forudsætning:** R1 skal se **to eBGP-veje** til **3.3.3.3/32** (via R2 og R4).

---

## 🧱 Topologi-udsnit (relevant for øvelsen)
```
R3 (ejer 3.3.3.3/32)
   | iBGP
   v
R2 (AS65020) --- eBGP --- R1 (AS65010) --- eBGP --- R4 (AS65040)
              MED=50                          MED=150
```

---

## Variant A — Brug `bgp always-compare-med` på **R1** (hurtigst i lab)

### 1) Sæt MED-værdier “ud” fra R2 og R4 (kun for 3.3.3.3/32)

**På R2 (AS65020):**
```cisco
conf t
ip prefix-list NET33 permit 3.3.3.3/32

route-map MED_TO_R1 permit 10
 match ip address prefix-list NET33
 set metric 50

router bgp 65020
 neighbor 10.0.12.1 route-map MED_TO_R1 out
end
```

**På R4 (AS65040):**
```cisco
conf t
ip prefix-list NET33 permit 3.3.3.3/32

route-map MED_TO_R1_HIGH permit 10
 match ip address prefix-list NET33
 set metric 150

router bgp 65040
 neighbor 10.0.24.2 route-map MED_TO_R1_HIGH out
end
```

> `set metric` i BGP = **MED**. Laveste værdi er “bedst” set fra modtagerens perspektiv.

### 2) Lad R1 sammenligne MED på tværs af forskellig nabo-AS
**På R1 (AS65010):**
```cisco
conf t
router bgp 65010
 bgp always-compare-med
end
```

### 3) Verificér på **R1**
```cisco
clear ip bgp 10.0.12.2 soft in
clear ip bgp 10.0.24.4 soft in

show ip bgp 3.3.3.3
show ip route 3.3.3.3
```
**Forvent:** Vejen via **R2 (MED 50)** er **best (`*>`)**, og routing peger mod **10.0.12.2**.  
I `show ip bgp 3.3.3.3` kan du se MED i attributlisten for begge paths.

---

## Variant B — Samme nabo-AS (uden `always-compare-med`)
> **Idé:** Sørg for at **R1** modtager **begge** veje til 3.3.3.3/32 fra **samme** AS (fx AS65020). Det kan du gøre ved at tilføje en ekstra eBGP-nabo **R5** i **AS65020** på den anden side af R1, eller midlertidigt sætte **R4** i **AS65020** til denne øvelse.

**Eksempel (R4 midlertidigt i AS65020):**
```cisco
! På R4 – skift AS (KUN til testen)
conf t
no router bgp 65040
router bgp 65020
 neighbor 10.0.24.2 remote-as 65010   ! eBGP mod R1 (AS65010)
 ! annoncer 3.3.3.3/32 med høj MED
ip prefix-list NET33 permit 3.3.3.3/32
route-map MED_HIGH permit 10
 match ip address prefix-list NET33
 set metric 150
router bgp 65020
 neighbor 10.0.24.2 route-map MED_HIGH out
end
```

**R2 beholder MED=50 som i Variant A.**  
**R1 behøver ikke `always-compare-med` i denne variant**, da begge veje nu kommer fra **samme** nabo-AS (65020).

**Verificér på R1 igen:**
```cisco
clear ip bgp 10.0.12.2 soft in
clear ip bgp 10.0.24.4 soft in
show ip bgp 3.3.3.3
show ip route 3.3.3.3
```
**Forvent:** R1 vælger vejen med **lavest MED** (via R2).

> **Husk** at sætte R4 tilbage til AS65040 bagefter.

---

## Fejlsøgning
- **R1 skifter ikke best-path:**  
  - Tjek at `bgp always-compare-med` er slået til (Variant A), eller at begge veje kommer fra **samme AS** (Variant B).  
  - Sørg for at **andre attributter** ikke dominerer (Weight/Local-Pref/AS_PATH).  
  - Se attributter: `show ip bgp 3.3.3.3` og sammenlign linje for MED/LocalPref/AS_PATH/Origin.
- **Ser du kun én path på R1?**  
  - Tjek at både **R2** og **R4** annoncerer 3.3.3.3/32 til R1 (prefix-lister/route-maps OK?).  
  - `show ip bgp summary` på R1 for sessionstatus.
- **Policy ikke anvendt:**  
  - Kør `clear ip bgp <neighbor> soft out` på R2/R4 efter du binder route-map **out**.
- **RIB tager ikke BGP-ruten:**  
  - `show ip bgp rib-failure` for forklaring.  
  - Tjek next-hop reachability (ping/traceroute).

---

## Oprydning (valgfrit)

**Variant A (med always-compare-med):**
```cisco
! På R2
conf t
router bgp 65020
 no neighbor 10.0.12.1 route-map MED_TO_R1 out
no route-map MED_TO_R1 permit 10
no ip prefix-list NET33
end

! På R4
conf t
router bgp 65040
 no neighbor 10.0.24.2 route-map MED_TO_R1_HIGH out
no route-map MED_TO_R1_HIGH permit 10
no ip prefix-list NET33
end

! På R1
conf t
router bgp 65010
 no bgp always-compare-med
end
```

**Variant B (samme AS på begge veje):**
```cisco
! Sæt R4 tilbage til sit oprindelige AS
conf t
no router bgp 65020
router bgp 65040
 neighbor 10.0.24.2 remote-as 65020
end
```

---

## Hurtig tjekliste (copy/paste)

**Efter du har sat MED og (evt.) always-compare-med:**
```cisco
R2# show run | sec route-map|ip prefix-list|router bgp
R4# show run | sec route-map|ip prefix-list|router bgp

R1# clear ip bgp 10.0.12.2 soft in
R1# clear ip bgp 10.0.24.4 soft in
R1# show ip bgp 3.3.3.3
R1# show ip route 3.3.3.3
! Forvent: best via R2 (lavest MED)
```

---

## Ekstra lege-ideer
- Test **per-prefix** MED: lav **lav MED** for nogle præfikser, **høj MED** for andre.  
- Kombinér MED med **AS-path prepending** og se hvilken regel der “vinder” når medtageren bruger standard vs. `always-compare-med`.  
- Brug **community**-mærker og politik på R2/R4 til at styre hvilke præfikser der får hvilken MED.
