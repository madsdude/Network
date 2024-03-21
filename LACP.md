# LACP

```.cisco 

conf t

interface Port-channel 1

interface FastEthernet 1/0/17-20
switchport trunk encapsulation dot1q
switchport mode trunk
channel-group 1 mode active

```
