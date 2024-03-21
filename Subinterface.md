# Subinterface

##### Subinterface refererer til et koncept inden for netværkskonfiguration og programmering, hvor et enkelt fysisk interface kan deles op i flere logiske interfaces. Dette gør det muligt for netværksadministratorer og udviklere at anvende forskellige konfigurationer og politikker for forskellige typer trafik på det samme fysiske interface. Et subinterface opfører sig som et uafhængigt interface, hvilket giver fleksibilitet i netværksdesign og -administration.

##### I netværksverdenen bruges subinterfaces ofte i forbindelse med VLAN (Virtual Local Area Network) tagging og routingprotokoller. For eksempel, i et scenario hvor et enkelt fysisk interface på en router forbinder til et switch, kan subinterfaces konfigureres til at høre til forskellige VLAN'er. Dette tillader data fra forskellige VLAN'er at rejse gennem det samme fysiske interface, men stadig forblive logisk adskilt, hvilket effektiviserer brugen af hardware og simplificerer netværksstrukturen.

##### Subinterfaces er også nyttige i MPLS (Multiprotocol Label Switching) netværk, hvor forskellige labels kan blive tildelt til subinterfaces for at styre trafikken mere effektivt. Desuden kan de anvendes i VPN (Virtual Private Network) scenarier for at adskille trafik mellem forskellige kunder eller afdelinger.

##### Sammendraget er, at subinterfaces tilbyder en metode til at øge netværkets fleksibilitet og effektivitet ved at tillade flere logiske netværk at køre over det samme fysiske netværksinterface, hvilket gør dem uundværlige i komplekse netværksdesign og -konfigurationer.








```.cisco
Conf t

interface gigabitEthernet 0/1.10

exit

interface gigabitEthernet 0/1.20

exit

interface gigabitEthernet 0/1.30

exit

interface gigabitEthernet 0/1.40

exit

interface gigabitEthernet 0/1.99

exit

interface gigabitEthernet 0/1.100

exit
```
# Subinterface R1
```.cisco
interface gigabitEthernet 0/1.100
encapsulation dot1Q 100
ip address 172.100.100.3 255.255.255.0
standby 1 ip 172.100.100.1
standby 1 priority 110
ip nat inside

```

# Subinterface R2
```.cisco
interface gigabitEthernet 0/1.100
encapsulation dot1Q 100
ip address 172.100.100.2 255.255.255.0
standby 1 ip 172.100.100.1
standby 1 priority 95
ip nat inside

```
