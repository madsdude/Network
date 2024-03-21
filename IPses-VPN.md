# IPses/VPN

| R1 Konfiguration | R2 Konfiguration | R3 Konfiguration | R4 Konfiguration |
|------------------|------------------|------------------|------------------|
| R1 |
crypto isakmp policy 10<br>encryption 3des<br>authentication pre-share<br>hash md5<br>group 5<br>exit<br>crypto isakmp key BirkeboTunnel address 192.168.150.29<br>crypto isakmp key BirkeboTunnel address 192.168.150.39<br>crypto ipsec transform-set Birkebo esp-sha-hmac esp-3des<br>mode transport<br>exit<br>crypto ipsec profile SEC_GRE<br>set transform-set Birkebo<br>exit<br>interface tunnel0<br>no shut<br>ip address 10.10.10.1 255.255.255.252<br>tunnel source 192.168.150.25<br>tunnel destination 192.168.150.29<br>tunnel protection ipsec profile SEC_GRE<br>exit<br>interface tunnel1<br>no shut<br>ip address 10.10.20.1 255.255.255.252<br>tunnel source 192.168.150.25<br>tunnel destination 192.168.150.39<br>tunnel protection ipsec profile SEC_GRE<br>exit<br>router ospf 1<br>network 10.10.10.0 0.0.0.3 area 0<br>network 10.10.20.0 0.0.0.3 area 0
| R2 |
crypto isakmp policy 10<br>encryption 3des<br>authentication pre-share<br>hash md5<br>group 5<br>exit<br>crypto isakmp key BirkeboTunnel address 192.168.150.29<br>crypto isakmp key BirkeboTunnel address 192.168.150.39<br>crypto ipsec transform-set Birkebo esp-sha-hmac esp-3des<br>mode transport<br>exit<br>crypto ipsec profile SEC_GRE<br>set transform-set Birkebo<br>exit<br>interface tunnel0<br>no shut<br>ip address 10.10.40.1 255.255.255.252<br>tunnel source 192.168.150.27<br>tunnel destination 192.168.150.39<br>tunnel protection ipsec profile SEC_GRE<br>exit<br>interface tunnel1<br>no shut<br>ip address 10.10.30.1 255.255.255.252<br>tunnel source 192.168.150.27<br>tunnel destination 192.168.150.29<br>tunnel protection ipsec profile SEC_GRE<br>exit<br>router ospf 1<br>network 10.10.30.0 0.0.0.3 area 0<br>network 10.10.40.0 0.0.0.3 area 0 





