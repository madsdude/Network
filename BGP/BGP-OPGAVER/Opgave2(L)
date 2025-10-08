# Øvelse 2 — **L = Local Preference** (Cisco)
*Styr **udgående trafikvalg for hele dit AS** ved at hæve **Local Preference (LP)** på ruter, du vil foretrække. **Højeste Local-Pref vinder**. LP er **intra‑AS** (viderefordeles til iBGP-naboer) og påvirker alle routere i AS'et.*

---

## 🎯 Mål & idé
- **Mål:** I **AS65020** (R2–R3) skal **det samme eksterne prefix** foretrækkes **via R1 (eBGP)** frem for **via R4 (eBGP)** — ved at hæve **Local-Pref** på ruter **modtaget fra R1**.
- **Hvorfor Local-Pref?** LP er global **inde i AS’et** og vinder over fx AS-PATH og MED. Den bruges til **udgående** trafikstyring (valg af egress), i modsætning til Cisco **Weight**, som kun er lokal til én router.
- **Forudsætning:** AS65020 skal modtage **samme prefix** fra **to forskellige eBGP‑udgange** (R1 og R4). Vi laver et testprefix **9.9.9.9/32** som både **R1** og **R4** annoncerer.

---

## 🧱 Topologi-udsnit (relevant for øvelsen)

```
   R1 (AS65010)        R2/R3 (AS65020)         R4 (AS65040)
      |  eBGP               iBGP                  eBGP  |
      +--------- 10.0.12.0/30 --- 10.0.24.0/30 ---------+
               \_______________  AS65020  ______________/
```

*AS65020 lærer **9.9.9.9/32** fra både R1 og R4. Vi hæver **Local-Pref** på R2 **indgående** fra R1, så **hele AS65020** vælger udgang via **R1**.*

---

## 1) Skab **to** eBGP-veje til **samme** prefix (9.9.9.9/32)

### På **R1** (AS65010) — annoncér 9.9.9.9/32
```cisco
conf t
ip route 9.9.9.9 255.255.255.255 Null0
router bgp 65010
 bgp log-neighbor-changes
 ! eBGP-nabo til R2 er allerede sat (10.0.12.2)
 network 9.9.9.9 mask 255.255.255.255
end
```

### På **R4** (AS65040) — annoncér 9.9.9.9/32
```cisco
conf t
ip route 9.9.9.9 255.255.255.255 Null0
router bgp 65040
 bgp log-neighbor-changes
 ! eBGP-nabo til R2 er allerede sat (10.0.24.2)
 network 9.9.9.9 mask 255.255.255.255
end
```

**Verificér på R2 (før vi ændrer LP):**
```cisco
show ip bgp 9.9.9.9
```
Forvent **to paths**: én via **10.0.12.1** (R1) og én via **10.0.24.4** (R4).  
Med alt andet ens kan valget svinge (fx AS-PATH lige lange → næste regler). Vi **tvinger** valget med Local-Pref.

---

## 2) Hæv **Local-Pref** på R2 for ruter fra **R1**

### På **R2** (AS65020)
> Vi sætter **Local-Pref 200** (standard er ofte 100) på **indgående** routes fra **R1**. Du kan vælge at påvirke **alle** ruter fra R1, eller kun **9.9.9.9/32** – begge varianter vises.

**Variant A — hæv LP for *alle* ruter fra R1 (simpelt):**
```cisco
conf t
route-map LP_FROM_R1 permit 10
 set local-preference 200

router bgp 65020
 neighbor 10.0.12.1 route-map LP_FROM_R1 in
end
```

**Variant B — hæv LP *kun* for 9.9.9.9/32 (granulært):**
```cisco
conf t
ip prefix-list PFX_9 permit 9.9.9.9/32

route-map LP_FROM_R1_ONLY9 permit 10
 match ip address prefix-list PFX_9
 set local-preference 200

router bgp 65020
 neighbor 10.0.12.1 route-map LP_FROM_R1_ONLY9 in
end
```

> **Note:** Local-Pref videredistribueres til **R3** via iBGP, så både **R2** og **R3** vil vælge **R1** som udgang til **9.9.9.9/32**.

---

## 3) Verificering

### På **R2** (efter policy):
```cisco
clear ip bgp 10.0.12.1 soft in
show ip bgp 9.9.9.9
show ip route 9.9.9.9
```
**Forvent:** Path via **10.0.12.1** er `*>` (best) og `LocalPref 200` ses i BGP-output. RIB peger mod **10.0.12.1**.

### På **R3** (bevis for intra‑AS effekt):
```cisco
show ip bgp 9.9.9.9
show ip route 9.9.9.9
```
**Forvent:** Også **LocalPref 200** på ruten til 9.9.9.9/32 og next-hop i retning af **R1** via R2 (pga. iBGP‑deling).

---

## 4) Hvad sker der “under motorhjelmen”?
- **LP sammenlignes tidligt** i best‑path (W **L** O AS O M P R). Med LP=200 beat’er R1‑vejen R4‑vejen, selv hvis fx AS‑PATH er identisk.
- **LP er AS‑global** (iBGP‑shared), derfor påvirkes både **R2** og **R3** – i modsætning til **Weight**, som kun er lokal til én router.
- Brug LP til **politik per udgang**: f.eks. “trafik til cloud‑prefixer skal ud mod R1”, mens andet går ud mod R4.

---

## 5) Typiske fejl & hurtige fixes

**A) Ser du stadig kun én path?**  
- Sørg for at **både R1 og R4** annoncerer **9.9.9.9/32** (static + `network` matcher RIB).  
  ```cisco
  R2# show ip bgp 9.9.9.9
  R1# show ip bgp 9.9.9.9
  R4# show ip bgp 9.9.9.9
  ```

**B) LP ændres ikke i tabellen:**  
- Kør `clear ip bgp 10.0.12.1 soft in` efter du binder route‑map.  
- Bekræft at route‑map er bundet **inbound** til **rigtig neighbor** (10.0.12.1).  
- Tjek route‑map hits: `show route-map`.

**C) RIB installerer ikke best‑path:**  
- `show ip bgp rib-failure` — kan pege på en mere foretrukken non‑BGP rute.  
- Tjek next‑hop reachability: `ping/traceroute <nexthop>`.

---

## 6) Oprydning (valgfrit)

**Fjern LP‑policy fra R2:**
```cisco
conf t
router bgp 65020
 no neighbor 10.0.12.1 route-map LP_FROM_R1 in
 no neighbor 10.0.12.1 route-map LP_FROM_R1_ONLY9 in
end
```

**Stop 9.9.9.9/32‑annoncering på R1 og R4:**
```cisco
! R1
conf t
router bgp 65010
 no network 9.9.9.9 mask 255.255.255.255
no ip route 9.9.9.9 255.255.255.255 Null0
end

! R4
conf t
router bgp 65040
 no network 9.9.9.9 mask 255.255.255.255
no ip route 9.9.9.9 255.255.255.255 Null0
end
```

---

## 7) Hurtig tjekliste (copy/paste)

**Før policy:**
```cisco
R2# show ip bgp 9.9.9.9
```

**Efter policy:**
```cisco
R2# clear ip bgp 10.0.12.1 soft in
R2# show ip bgp 9.9.9.9
R2# show ip route 9.9.9.9

R3# show ip bgp 9.9.9.9
R3# show ip route 9.9.9.9
```

**Forvent:** LocalPref = **200** på path via **R1**, valgt som best (`*>`) på både **R2** og **R3**.
