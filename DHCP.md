# DHCP

```.Cisco

conf t

ip dhcp pool Test1
network 192.168.0.0 255.255.255.0
default-router 192.168.0.1
dns-server 192.168.0.1
lease 30

```
