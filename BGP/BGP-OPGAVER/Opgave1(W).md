# Øvelse 1 — W = Weight (Cisco)

## Styr den udgående best path på R2 for præfikset 3.3.3.3/32 ved at give den vej, du ønsker, højere Weight. Weight er kun lokal til routeren, og højeste vinder.

## 🎯 Mål & idé

* Mål: På R2 (AS65020) skal 3.3.3.3/32 foretrækkes via R3 (iBGP) i stedet for via R4 (eBGP) – kun ved at bruge Weight.

* Hvorfor Weight? Weight er den øverste beslutningsfaktor på Cisco (W L O AS O M P R). Sætter du Weight højere på en indkommende rute, vinder den altid på den router – uden at påvirke andre routere.

* Forudsætning: R2 skal se to veje til 3.3.3.3/32:

* via R3 (iBGP) — den “rigtige” kilde (R3 ejer 3.3.3.3/32)

* via R4 (eBGP) — vi “faker” samme præfiks på R4, så R2 har et valg


# Skab to konkurrerende veje til 3.3.3.3/32

## På R4 (AS65040) – “falsk” annoncering af 3.3.3.3/32
Formål: Gør så R2 også modtager 3.3.3.3/32 fra R4 (eBGP), så der er to paths at vælge imellem.

```
conf t
! Lav en local “kilde” til præfikset, så BGPs network-kommando kan matche RIB
ip route 3.3.3.3 255.255.255.255 Null0
```

Tjek på R2 (efter et par sekunder / eller kør soft reset):

```
show ip bgp 3.3.3.3
```
Du bør nu se to paths: én via 10.0.23.3 (R3/iBGP) og én via 10.0.24.4 (R4/eBGP).
Som udgangspunkt vinder ofte eBGP-vejen, hvis alt andet er ens

```
R4#show ip bgp 3.3.3.3
BGP routing table entry for 3.3.3.3/32, version 5
Paths: (1 available, best #1, table default, RIB-failure(17))
  Not advertised to any peer
  Refresh Epoch 1
  65020
    10.0.24.2 from 10.0.24.2 (2.2.2.2)
      Origin IGP, localpref 100, valid, external, best
      rx pathid: 0, tx pathid: 0x0
R4#
```


