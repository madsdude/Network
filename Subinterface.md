# Subinterface

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
