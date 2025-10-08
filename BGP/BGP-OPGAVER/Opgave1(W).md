# Øvelse 1 — W = Weight (Cisco)

## Styr den udgående best path på R2 for præfikset 3.3.3.3/32 ved at give den vej, du ønsker, højere Weight. Weight er kun lokal til routeren, og højeste vinder.

## 🎯 Mål & idé

* Mål: På R2 (AS65020) skal 3.3.3.3/32 foretrækkes via R3 (iBGP) i stedet for via R4 (eBGP) – kun ved at bruge Weight.

* Hvorfor Weight? Weight er den øverste beslutningsfaktor på Cisco (W L O AS O M P R). Sætter du Weight højere på en indkommende rute, vinder den altid på den router – uden at påvirke andre routere.

* Forudsætning: R2 skal se to veje til 3.3.3.3/32:

* via R3 (iBGP) — den “rigtige” kilde (R3 ejer 3.3.3.3/32)

* via R4 (eBGP) — vi “faker” samme præfiks på R4, så R2 har et valg
