# HSRP

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
