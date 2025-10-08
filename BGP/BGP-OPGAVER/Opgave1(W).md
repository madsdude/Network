# Øvelse 1 — **W = Weight** (Cisco)
*Styr den udgående **best path** på **R2** for præfikset **3.3.3.3/32** ved at give den vej, du ønsker, **højere Weight**. Weight er **kun lokal** til routeren, og **højeste vinder**.*

---

## 🎯 Mål & idé
- **Mål:** På **R2 (AS65020)** skal **3.3.3.3/32** foretrækkes **via R3 (iBGP)** i stedet for **via R4 (eBGP)** – **kun** ved at bruge **Weight**.  
- **Hvorfor Weight?** Weight er den **øverste** beslutningsfaktor på Cisco (W L O AS O M P R). Sætter du Weight højere på en indkommende rute, vinder den **altid** på **den router** – uden at påvirke andre routere.
- **Forudsætning:** R2 skal **se to veje** til **3.3.3.3/32**:
  1) via **R3 (iBGP)** — den “rigtige” kilde (R3 ejer 3.3.3.3/32)  
  2) via **R4 (eBGP)** — vi “faker” samme præfiks på R4, så R2 har et valg

---

## 🧱 Topologi-udsnit (relevant for øvelsen)

```
R3 (AS65020, iBGP) --- R2 (AS65020) --- R4 (AS65040, eBGP)
     10.0.23.3         10.0.23.2        10.0.24.2 --- 10.0.24.4
        |                   |                    |
     Lo1=3.3.3.3        (Weight sættes her)   [faker 3.3.3.3/32]
```

---

## 1) Skab **to** konkurrerende veje til 3.3.3.3/32

### På **R4** (AS65040) – “falsk” annoncering af 3.3.3.3/32
> Formål: Gør så **R2** også modtager 3.3.3.3/32 fra **R4** (eBGP), så der er **to** paths at vælge imellem.

```cisco
conf t
! Lav en local “kilde” til præfikset, så BGPs network-kommando kan matche RIB
ip route 3.3.3.3 255.255.255.255 Null0

router bgp 65040
 bgp log-neighbor-changes
 ! Du har allerede eBGP-naboskab til R2 (10.0.24.2)
 network 3.3.3.3 mask 255.255.255.255
end
```

**Tjek på R2 (efter et par sekunder / eller kør soft reset):**
```cisco
show ip bgp 3.3.3.3
```
Du bør nu se **to** paths: én via **10.0.23.3** (R3/iBGP) og én via **10.0.24.4** (R4/eBGP).  
Som udgangspunkt vinder ofte **eBGP**-vejen, hvis alt andet er ens.

---

## 2) Sæt **Weight** på R2 for at favorisere iBGP-vejen (R3)

### På **R2** (AS65020)
> Vi matcher præcis **3.3.3.3/32** og giver **høj Weight** på **indgående** updates fra iBGP-naboen **10.0.23.3**.

```cisco
conf t
ip prefix-list NET_33 permit 3.3.3.3/32

route-map PREFER_R3_WEIGHT permit 10
 match ip address prefix-list NET_33
 set weight 500

router bgp 65020
 neighbor 10.0.23.3 route-map PREFER_R3_WEIGHT in
end
```

> **Bemærk:** Weight anvendes **kun** på R2. R1 eller R3 påvirkes ikke.

---

## 3) Verificering (på R2)

```cisco
! Tving genanvendelse af indgående politik (valgfrit, men rart)
clear ip bgp 10.0.23.3 soft in

! Se BGP-beslutningen
show ip bgp 3.3.3.3

! Se hvilken vej der kom i RIB (routing-tabellen)
show ip route 3.3.3.3
```

### Forventet “essens” af output
- `show ip bgp 3.3.3.3` viser **to** paths. Den via **10.0.23.3** (R3) har **Weight 500** og er markeret `*>` (valid+best).  
- `show ip route 3.3.3.3` viser next-hop **10.0.23.3** (i stedet for 10.0.24.4).

---

## 4) Hvad sker der “under motorhjelmen”?
- **Før policy:** R2 ser to paths → eBGP vs iBGP. Alt andet ens ⇒ eBGP vinder (reglen “P” i W-L-O-AS-O-M-P-R).  
- **Efter Weight:** R2 sætter **Weight=500** på iBGP-vejen. Weight sammenlignes **før** alt andet ⇒ iBGP-vejen **vinder** lokalt på R2.  
- **Scope:** Weight er **lokal** (ikke annonceret videre). Det er perfekt til **lokal udgående trafikstyring**.

---

## 5) Typiske fejl & hurtige fixes

**A) Ser du kun én path på R2?**  
- Tjek at R4 **reelt** annoncerer 3.3.3.3/32:
  ```cisco
  R4# show ip bgp 3.3.3.3
  R4# show ip route 3.3.3.3
  ```
  Skal indeholde `S 3.3.3.3/32 is directly connected, Null0` og `*> 3.3.3.3/32 ...` i BGP.  
- Tjek eBGP-naboskab R4↔R2:  
  ```cisco
  R2# show ip bgp summary
  R2# show ip bgp 3.3.3.3
  ```

**B) BGP viser to paths, men RIB installerer ikke din “vinder”:**  
- Kør:
  ```cisco
  R2# show ip bgp rib-failure
  ```
  typiske årsager:
  - Der ligger en **connected/static/IGP** til samme prefix med lavere AD.  
  - Next-hop **ikke nåbar** (især på iBGP – men du har `next-hop-self` på R2→R3, så det bør være fint).  
  - Forkert mask på `network`-kommandoen i forhold til RIB.

**C) Politikken bider ikke (ingen ændring):**  
- Tjek at route-map er bundet **inbound** til **den rigtige neighbor** (10.0.23.3).  
- Brug `clear ip bgp 10.0.23.3 soft in` efter du har tilføjet policy.

---

## 6) Oprydning (valgfrit, når du er færdig)

**På R2 (fjern Weight-policy):**
```cisco
conf t
router bgp 65020
 no neighbor 10.0.23.3 route-map PREFER_R3_WEIGHT in
end
```

**På R4 (stop “falsk” annoncering):**
```cisco
conf t
router bgp 65040
 no network 3.3.3.3 mask 255.255.255.255
no ip route 3.3.3.3 255.255.255.255 Null0
end
```

---

## 7) Hurtig tjekliste (copy/paste)
**Før du sætter Weight:**
```cisco
R2# show ip bgp 3.3.3.3
```
> Ser du **to** paths (via 10.0.23.3 og 10.0.24.4)?

**Efter du sætter Weight + soft clear:**
```cisco
R2# clear ip bgp 10.0.23.3 soft in
R2# show ip bgp 3.3.3.3
R2# show ip route 3.3.3.3
```
> Vinder den via **10.0.23.3** nu (Weight 500, `*>`)?

---

## 8) Hvad kan du lege videre med?
- Sæt **Weight per neighbor** (uden prefix-match), fx:
  ```cisco
  router bgp 65020
   neighbor 10.0.23.3 weight 400
  ```
  (Gælder **alle** ruter fra den nabo — pas på sideeffekter!)
- Kombinér med **Local-Preference** (næste øvelse) for at styre hele **AS65020** i stedet for kun R2.
- Prøv at få eBGP-vejen til at vinde igen ved at sænke Weight eller hæve Weight på R4-path’en.
