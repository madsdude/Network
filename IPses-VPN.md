# IPses/VPN

| R1 | Konfiguration | R2 | Konfiguration | R3 | Konfiguration | R4 | Konfiguration |
| ------ | ------------- | ------ | ------------- | ------ | ------------- | ------ | ------------- |
| R1 | 


| R1 Konfiguration | R2 Konfiguration | R3 Konfiguration | R4 Konfiguration |
|------------------|------------------|------------------|------------------|
| **crypto isakmp policy 10** | **crypto isakmp policy 10** | **crypto isakmp policy 10** | **crypto isakmp policy 10** |
| encryption 3des | encryption 3des | encryption 3des | encryption 3des |
| authentication pre-share | authentication pre-share | authentication pre-share | authentication pre-share |
| hash md5 | hash md5 | hash md5 | hash md5 |
| group 5 | group 5 | group 5 | group 5 |
| exit | exit | exit | exit |
| **crypto isakmp key BirkeboTunnel** | **crypto isakmp key BirkeboTunnel** | **crypto isakmp key BirkeboTunnel** | **crypto isakmp key BirkeboTunnel** |
| address 192.168.150.29 | address 192.168.150.29 | address 192.168.150.25 | address 192.168.150.25 |
| address 192.168.150.39 | address 192.168.150.39 | address 192.168.150.27 | address 192.168.150.27 |
| **crypto ipsec transform-set Birkebo** | **crypto ipsec transform-set Birkebo** | **crypto ipsec transform-set Birkebo** | **crypto ipsec transform-set Birkebo** |
| esp-sha-hmac esp-3des | esp-sha-hmac esp-3des | esp-sha-hmac esp-3des | esp-sha-hmac esp-3des |
| mode transport | mode transport | mode transport | mode transport |
| exit | exit | exit | exit |
| **crypto ipsec profile SEC_GRE** | **crypto ipsec profile SEC_GRE** | **crypto ipsec profile SEC_GRE** | **crypto ipsec profile SEC_GRE** |
| set transform-set Birkebo | set transform-set Birkebo | set transform-set Birkebo | set transform-set Birkebo |
| exit | exit | exit | exit |
| **interface tunnel0** | **interface tunnel0** | **interface tunnel0** | **interface tunnel0** |
| no shut | no shut | no shut | no shut |
| ip address 10.10.10.1 255.255.255.252 | ip address 10.10.40.1 255.255.255.252 | ip address 10.10.10.2 255.255.255.252 | ip address 10.10.40.2 255.255.255.252 |
| tunnel source 192.168.150.25 | tunnel source 192.168.150.27 | tunnel source 192.168.150.29 | tunnel source 192.168.150.39 |
| tunnel destination 192.168.150.29 | tunnel destination 192.168.150.39 | tunnel destination 192.168.150.25 | tunnel destination 192.168.150.27 |
| tunnel protection ipsec profile SEC_GRE | tunnel protection ipsec profile SEC_GRE | tunnel protection ipsec profile SEC_GRE | tunnel protection ipsec profile SEC_GRE |
| exit | exit | exit | exit |
| **interface tunnel1** | **interface tunnel1** | **interface tunnel1** | **interface tunnel1** |
| no shut | no shut | no shut | no shut |
| ip address 10.10.20.1 255.255.255.252 | ip address 10.10.30.1 255.255.255.252 | ip address 10.10.30.2 255.255.255.252 | ip address 10.10.20.2 255.255.255.252 |
| tunnel source 192.168.150.25 | tunnel source 192.168.150.27 | tunnel source 192.168.150.29 | tunnel source 192.168.150.39 |
| tunnel destination 192.168.150.39 | tunnel destination 192.168.150.29 | tunnel destination 192.168.150.27 | tunnel destination 192.168.150.25 |
| tunnel protection ipsec profile SEC_GRE | tunnel protection ipsec profile SEC_GRE | tunnel protection ipsec profile SEC_GRE | tunnel protection ipsec profile SEC_GRE |
| exit | exit | exit | exit |
| **router ospf 1**



