1. Opret templaten (Definition)
Cisco CLI

template VLAN10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 description Access-Port-VLAN10
exit
2. Brug templaten på interfaces (Application)
Når templaten er oprettet, skal den "knyttes" til de fysiske porte. Det gør du sådan her:

Cisco CLI

interface range GigabitEthernet1/0/1-48
 source template VLAN10
exit
Forklaring af kommandoerne
template [navn]: Starter template-mode. Alt du skriver herunder gemmes som en makro.

source template [navn]: Dette er "triggeren". Den tager alle kommandoerne fra din template og skyder dem ind på de valgte interfaces.

Fordelen: Hvis du senere ændrer i template VLAN10 (f.eks. tilføjer spanning-tree bpduguard enable), så slår det igennem på alle porte, der bruger templaten (afhængig af IOS version og source konfigurationen).
