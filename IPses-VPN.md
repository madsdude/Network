# IPses/VPN

| R1 | Konfiguration | R2 | Konfiguration | R3 | Konfiguration | R4 | Konfiguration |
| ------ | ------------- | ------ | ------------- | ------ | ------------- | ------ | ------------- |
| R1 | 


| R1 Konfiguration | R2 Konfiguration | R3 Konfiguration | R4 Konfiguration |
|------------------|------------------|------------------|------------------|
| ```
<br> crypto isakmp policy 10
<br> encryption 3des
<br> authentication pre-share
<br> hash md5
<br> group 5
<br> exit

crypto isakmp key BirkeboTunnel address 192.168.150.29
crypto isakmp key BirkeboTunnel address 192.168.150.39
crypto ipsec transform-set Birkebo esp-sha-hmac esp-3des
mode transport
exit
crypto ipsec profile SEC_GRE
set transform-set Birkebo
exit

interface tunnel0
no shut
ip address 10.10.10.1 255.255.255.252
tunnel source 192.168.150.25
tunnel destination 192.168.150.29
tunnel protection ipsec profile SEC_GRE
exit

interface tunnel1
no shut
ip address 10.10.20.1 255.255.255.252
tunnel source 192.168.150.25
tunnel destination 192.168.150.39
tunnel protection ipsec profile SEC_GRE
exit

router ospf 1
network 10.10.10.0 0.0.0.3 area 0
network 10.10.20.0 0.0.0.3 area 0 ``` | 



