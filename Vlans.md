# Vlan

##### VLANs (Virtual Local Area Networks) bruges til at oprette isolerede netværk inden for det samme fysiske netværk. De bruges til at forbedre netværkets ydeevne ved at reducere størrelsen af broadcast-domæner og forbedre sikkerheden ved at isolere forskellige netværkssegmenter. De kan også bruges til at gruppere logiske enheder sammen, uanset deres fysiske placering i netværket.

```.cisco

config terminal
vlan 10
name HR

config terminal
vlan 20
name IT

"Switch config"
interface FastEthernet0/1
switchport access vlan 20
switchport mode access

interface FastEthernet0/23

switchport trunk allowed vlan 10,20,30,40,99,100
switchport mode trunk

"Router config"

Router>enable
Router#config terminal

Router(config)#int fa0/0
Router(config-if)#no shutdown

Router(config-if)#int fa0/0.10
Router(config-subif)#encapsulation dot1q 10
Router(config-subif)#ip add 192.168.1.1 255.255.255.0
Router(config-subif)#

Router(config-subif)#int fa0/0.20
Router(config-subif)#encapsulation dot1q 20
Router(config-subif)#ip add 192.168.2.1 255.255.255.0

```
