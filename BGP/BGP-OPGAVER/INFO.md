# Her får du en samling hands-on BGP-opgaver du kan køre i GNS3/EVE-NG/IOU/IOSv. Hver øvelse fokuserer på ét trin i best-path (“W L O AS O M P R”) + lidt ekstra. Jeg giver dig en lille grund-topologi, en basis-konfig, og derefter opgaver hvor du kun ændrer det nødvendige (som du foretrækker).
## Grund-topologi (bruges i de fleste øvelser)

<img width="838" height="510" alt="image" src="https://github.com/user-attachments/assets/d01b31e7-b210-421a-8a8b-f76fa097a8f2" />

# IP-plan 

R1–R2: 10.0.12.0/30 (R1=10.0.12.1, R2=10.0.12.2)

R2–R3: 10.0.23.0/30 (R2=10.0.23.2, R3=10.0.23.3) (bruges som IGP transport)

R2–R4: 10.0.24.0/30 (R2=10.0.24.2, R4=10.0.24.4)

Loopbacks: R1=1.1.1.1/32, R2=2.2.2.2/32, R3=3.3.3.3/32, R4=4.4.4.4/32

Prefix du kan annoncere:

R1: 1.1.1.0/24

R3: 3.3.3.0/24

R4: 4.4.4.0/24


# R1

```
conf t

hostname R1

interface GigabitEthernet0/0
ip address 10.0.12.1 255.255.255.252
no shutdown 

interface loopback 1 
ip address 1.1.1.1 255.255.255.255

router bgp 65010
 bgp log-neighbor-changes
 neighbor 10.0.12.2 remote-as 65020
 network 1.1.1.1 mask 255.255.255.255
```

# R2 

```
conf t

hostname R2

interface GigabitEthernet0/0
ip address 10.0.12.2 255.255.255.252
no shutdown 

interface GigabitEthernet0/1
ip address 10.0.23.2 255.255.255.248
no shutdown 

interface GigabitEthernet0/2
ip address 10.0.24.2 255.255.255.248
no shutdown 

interface loopback 1 
ip address 2.2.2.2 255.255.255.255

router bgp 65020
 bgp log-neighbor-changes
 neighbor 10.0.12.1 remote-as 65010
 neighbor 10.0.24.4 remote-as 65040
 neighbor 10.0.23.3 remote-as 65020
 neighbor 10.0.23.3 update-source 10.0.23.2
 neighbor 10.0.23.3 next-hop-self
 
router ospf 1
 network 10.0.23.0 0.0.0.3 area 0

```

# R3 
```
conf t

hostname R3

interface GigabitEthernet0/1
ip address 10.0.23.3 255.255.255.248
no shutdown 

interface loopback 1 
ip address 3.3.3.3 255.255.255.255

router bgp 65020
 bgp log-neighbor-changes
 neighbor 10.0.23.2 remote-as 65020
 neighbor 10.0.23.2 update-source 10.0.23.3
 network 3.3.3.3 mask 255.255.255.255
 

router ospf 1
 network 10.0.23.0 0.0.0.3 area 0
```
# R4
```
conf t

hostname R4

interface GigabitEthernet0/2
ip address 10.0.24.4 255.255.255.248
no shutdown 

interface loopback1
 ip address 4.4.4.4 255.255.255.255

router bgp 65040
 bgp log-neighbor-changes
 neighbor 10.0.24.2 remote-as 65020
 network 4.4.4.4 mask 255.255.255.255

```

