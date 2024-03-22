# HSRP

#### HSRP står for Hot Standby Router Protocol og er et Cisco-specifikt redundansprotokol designet til at tillade flere routere at arbejde sammen i en gruppe, hvilket giver fejltolerance og høj tilgængelighed i IP-netværk. Det sikrer, at brugertrafik øjeblikkeligt og transparent kan omdirigeres til en backup-router, hvis den primære router fejler eller går ned. Dette opnås ved at tildele gruppen af routere en enkelt virtuel IP-adresse og MAC-adresse. Den virtuelle IP-adresse bruges som gateway-adresse for enheder på netværket. Inden for HSRP-gruppen er der en primær (aktiv) router, der håndterer al trafik for den virtuelle IP-adresse, mens de andre routere står i standby og overtager trafikbehandlingen, hvis den aktive router fejler.


```.cisco

interface GigabitEthernet0/1
 encapsulation dot1Q 100
 ip address 192.168.1.2 255.255.255.0
 ip nat inside
 ip virtual-reassembly in
 standby 1 ip 192.168.1.1
 standby 1 priority 110
 standby 1 preempt
 standby 1 track 1 decrement 20

```
