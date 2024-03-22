# DHCP

## Denne kode er en konfiguration for en DHCP-server på en netværksenhed, såsom en router eller en switch, der kører Cisco IOS software. DHCP, som står for Dynamic Host Configuration Protocol, er et netværksprotokol, der bruges til at automatisk tildele IP-adresser og anden netværkskonfigurationsinformation til enheder på et netværk, så brugere ikke manuelt skal konfigurere disse indstillinger på deres enheder.

```.Cisco

conf t

ip dhcp pool Test1
network 192.168.0.0 255.255.255.0
default-router 192.168.0.1
dns-server 192.168.0.1
lease 30

```

### Lad os bryde koden ned:

##### 1. conf t: Dette er en forkortelse for "configure terminal". Det er en kommando, der bringer dig ind i den globale konfigurationstilstand på enheden, hvor du kan foretage ændringer i enhedens konfiguration.

##### 2. ip dhcp pool Test1: Denne kommando opretter en ny DHCP-pool med navnet "Test1". En DHCP-pool indeholder den række af indstillinger, der skal tildeles til klienter, der forbinder til netværket.

##### 3. network 192.168.0.0 255.255.255.0: Denne kommando angiver det IP-adresseområde, som DHCP-serveren kan tildele fra, samt subnetmasken for netværket. I dette tilfælde er IP-adresseområdet 192.168.0.0 til 192.168.0.255 med en subnetmaske på 255.255.255.0 (dette er et typisk opsætning for et lille lokalt netværk).

##### 4. default-router 192.168.0.1: Denne kommando angiver standard gateway for klienterne. Standard gateway er den enhed på netværket, der formidler trafik mellem forskellige netværk. I dette tilfælde er 192.168.0.1 sandsynligvis IP-adressen på routeren, der forbinder det lokale netværk til internettet eller andre netværk.

##### 5. dns-server 192.168.0.1: Denne kommando angiver DNS-serverens adresse, der skal bruges af klienterne. DNS (Domain Name System) er ansvarlig for at oversætte domænenavne til IP-adresser. Her angives det, at DNS-serveren har samme IP-adresse som standard gateway, hvilket ofte er tilfældet i små netværk.

##### 6. lease 30: Denne kommando angiver varigheden af lejemålet for en IP-adresse i dage. Efter denne periode skal enheden forny sin DHCP-lejemål for at fortsætte med at bruge IP-adressen. I dette tilfælde er lejeperioden sat til 30 dage.

##### Samlet set konfigurerer denne kode en DHCP-server til automatisk at tildele IP-adresser, en standard gateway, og en DNS-serveradresse til enheder, der forbinder til netværket, med en IP-adresse lejemål på 30 dage.
