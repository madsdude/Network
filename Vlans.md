# Vlan

##### VLANs (Virtual Local Area Networks) bruges til at oprette isolerede netværk inden for det samme fysiske netværk. De bruges til at forbedre netværkets ydeevne ved at reducere størrelsen af broadcast-domæner og forbedre sikkerheden ved at isolere forskellige netværkssegmenter. De kan også bruges til at gruppere logiske enheder sammen, uanset deres fysiske placering i netværket.

```.cisco

interface FastEthernet0/1
switchport access vlan 20
switchport mode access

interface FastEthernet0/23

switchport trunk allowed vlan 10,20,30,40,99,100
switchport mode trunk

```
