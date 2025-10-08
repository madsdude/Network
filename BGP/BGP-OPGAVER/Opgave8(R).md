# Øvelse 8 — **R = Router-ID tie-break** (laveste RID vinder ved fuld lighed)
*Når **alle andre** BGP-attributter er ens (Weight, Local-Pref, ORIGIN, AS_PATH, MED, Path type, IGP cost), afgør **Router-ID** (RID) vinderen. **Laveste RID** vinder.*

---

## 🎯 Mål & idé
- **Mål:** På **R2 (AS65020)** skal præfikset **55.55.55.0/24** annonceres fra **to iBGP-naboer** (R3 og R5) med **identiske** attributter, så **RID** bliver tie-breakeren: **laveste RID** vinder.
- **Hvorfor?** Best-path rækkefølgen slutter (efter en række ekstra tie-breaks) med at vælge **laveste BGP Router-ID** for path’en, når alt andet er ens.
- **Forudsætning:** Der findes iBGP mellem R2–R3; vi tilføjer en ekstra iBGP-peer **R5** i samme AS (65020).

> **Note:** Sørg for **ikke** at have `maximum-paths` aktiveret på R2 (ellers kan den installere begge veje som ECMP og du ser ikke tie-break på RID).

---

## 🧱 Topologi (minimal for øvelsen)
```
        AS65020 (iBGP)
   R3 (RID 3.3.3.3)         R5 (RID 5.5.5.5)
        \                       //
         \                     //
              ---- R2 ----
                    ^ ser to iBGP paths til 55.55.55.0/24
```

**IP-forslag (justér til din lab):**
- R2–R3: 10.0.23.0/30 (R2=10.0.23.2, R3=10.0.23.3)
- R2–R5: 10.0.25.0/30 (R2=10.0.25.2, R5=10.0.25.5)
- Loopbacks: R3=3.3.3.3/32, R5=5.5.5.5/32 (bruges også som BGP RID og som next-hop hvis du vil)

---

## 1) Klargør R3 og R5 (samme præfiks, identiske attributter)

### På **R3 (AS65020)**
```cisco
conf t
interface Loopback55
 ip address 55.55.55.3 255.255.255.0

! Sæt eksplicit router-id for determinisme
router bgp 65020
 bgp router-id 3.3.3.3
 network 55.55.55.0 mask 255.255.255.0
end
```

### På **R5 (AS65020)** — ny iBGP-peer
```cisco
conf t
interface Loopback55
 ip address 55.55.55.5 255.255.255.0

router bgp 65020
 bgp router-id 5.5.5.5
 network 55.55.55.0 mask 255.255.255.0
end
```

> **Vigtigt:** Begge annoncører **samme præfiks** **med `network`**, så ORIGIN = `i` på begge. Ingen MED, ingen communities, ingen LP/Weight-politikker.

---

## 2) iBGP peering til R2 og next-hop reachability

### På **R2 (AS65020)**
```cisco
conf t
router bgp 65020
 ! iBGP til R3
 neighbor 10.0.23.3 remote-as 65020
 neighbor 10.0.23.3 update-source 10.0.23.2
 neighbor 10.0.23.3 next-hop-self

 ! iBGP til R5 (nyt link/peer)
 neighbor 10.0.25.5 remote-as 65020
 neighbor 10.0.25.5 update-source 10.0.25.2
 neighbor 10.0.25.5 next-hop-self

 ! sørg for at ECMP ikke maskerer tie-break
 no maximum-paths
end
```

### IGP/OSPF for transport (hvis nødvendigt)
```cisco
! På R2, R3, R5 – sørg for at links mellem dem er i IGP
router ospf 1
 network 10.0.23.0 0.0.0.3 area 0
 network 10.0.25.0 0.0.0.3 area 0
```

> **Note:** `next-hop-self` på R2 gør, at ****R2** ser begge paths med **nexthop = R2**, hvilket udligner IGP-cost-til-next-hop (en anden tie-break længere oppe).

---

## 3) Verificér at R2 ser **to** fuldstændigt ens paths
```cisco
R2# show ip bgp 55.55.55.0
```
**Forvent:**
- To entries for 55.55.55.0/24 — én med *neighbor* = **10.0.23.3** (RID 3.3.3.3) og én med *neighbor* = **10.0.25.5** (RID 5.5.5.5).
- Samme Weight (0), samme LocalPref (100), samme ORIGIN (`i`), samme AS_PATH (tom/iBGP), samme MED (—), Path type = iBGP.

> Hvis du ser forskel i attributter, fx Local-Pref eller MED, så fjern politik/route-maps på R2/R3/R5 eller sæt dem identisk (ingen sætninger).

---

## 4) Bekræft tie-break på **Router-ID**
```cisco
R2# show ip bgp 55.55.55.0
R2# show ip route 55.55.55.0
```
**Forvent:** Best (`*>`) bør være **path via R3** (fordi **RID 3.3.3.3** < **5.5.5.5**). RIB peger mod den BGP-nabo hvis path vandt.

> Hvis best ikke bliver R3: Tjek at **RID’erne er sat som angivet** og at **ingen andre attributter** divergerer. Tjek også at `maximum-paths` **ikke** er aktiv.

---

## 5) (Valgfrit) Bevis ved at bytte RID-orden
- Skift R5’s RID til **2.2.2.2** (lavere end 3.3.3.3), og genstart BGP-processen på R5 for at RID træder i kraft:
```cisco
R5(config)# router bgp 65020
R5(config-router)# bgp router-id 2.2.2.2
! RID ændres først ved BGP reset
R5# clear ip bgp *  (eller fjern/tilføj naboen igen)
```
- Se nu at **R5** vinder (laveste RID).

---

## 6) Fejlsøgning
- **Ser du kun én path?**  
  - Mangler iBGP til R5? (`show ip bgp summary`)  
  - Lærer R5 overhovedet 55.55.55.0/24? (`show ip bgp 55.55.55.0` på R5)  
  - Blokerer filtrering på R2/R5?
- **Attributter ikke identiske:**  
  - Fjern `weight`, `local-preference`, `metric` (MED), `as-path prepend` policies.  
  - Bekræft ORIGIN = `i` på begge (`network`-metoden).
- **ECMP i stedet for tie-break:**  
  - `show run | i maximum-paths` på R2 — fjern eller sæt `maximum-paths 1` (default).
- **RID ændres ikke:**  
  - RID låses når BGP starter. Sæt `bgp router-id` og **reset** BGP-processen på den router for at aktivere.

---

## 7) Oprydning (valgfrit)
```cisco
! På R2
conf t
router bgp 65020
 no neighbor 10.0.25.5 remote-as 65020
 no neighbor 10.0.25.5
 no neighbor 10.0.23.3 next-hop-self   ! hvis du vil tilbage til basis
end

! På R5
conf t
no interface Loopback55
no router bgp 65020
end
```

---

## 8) Ekstra lege-ideer
- Slå **iBGP multipath** til på R2 (`maximum-paths 2`) og se at begge veje nu kommer i RIB (ECMP) — ingen RID tie-break.
- Introducér en **lille MED-forskel** via route-map på R3 eller R5, og observer at MED evalueres **før** RID (gør den laveste MED til vinder).  
- Sæt **Weight** på R2 for at tvinge en vinder (Weight evalueres allerførst).

