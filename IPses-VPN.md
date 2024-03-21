# IPses/VPN

| Kommando | Detalje |
|----------|---------|
| **R1 Konfiguration** | |
| `crypto isakmp policy 10` | encryption 3des<br>authentication pre-share<br>hash md5<br>group 5<br>exit |
| `crypto isakmp key` | BirkeboTunnel address 192.168.150.29<br>BirkeboTunnel address 192.168.150.39 |
| `crypto ipsec transform-set Birkebo` | esp-sha-hmac esp-3des<br>mode transport<br>exit |
| `crypto ipsec profile SEC_GRE` | set transform-set Birkebo<br>exit |
| `interface tunnel0` | no shut<br>ip address 10.10.10.1 255.255.255.252<br>tunnel source 192.168.150.25<br>tunnel destination 192.168.150.29<br>tunnel protection ipsec profile SEC_GRE<br>exit |
| `interface tunnel1` | no shut<br>ip address 10.10.20.1 255.255.255.252<br>tunnel source 192.168.150.25<br>tunnel destination 192.168.150.39<br>tunnel protection ipsec profile SEC_GRE<br>exit |
| `router ospf 1` | network 10.10.10.0 0.0.0.3 area 0<br>network 10.10.20.0 0.0.0.3 area 0 |

| **R2 Konfiguration** | |
| `crypto isakmp policy 10` | ... |
| `interface tunnel0` | ... |
| `interface tunnel1` | ... |
| `router ospf 1` | ... |

| **R3 Konfiguration** | |
| `crypto isakmp policy 10` | ... |
| `interface tunnel0` | ... |
| `interface tunnel1` | ... |
| `router ospf 1` | ... |

| **R4 Konfiguration** | |
| `crypto isakmp policy 10` | ... |
| `interface tunnel0` | ... |
| `interface tunnel1` | ... |
| `router ospf 1` | ... |

