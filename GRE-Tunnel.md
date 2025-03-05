# **Guide: Opsætning af GRE Tunnel på Cisco Routere**

## **Hvad er en GRE Tunnel?**
GRE (Generic Routing Encapsulation) er en tunneling-protokol, der kan indkapsle forskellige protokoller og sende dem via en IP-tunnel. Dette bruges ofte til at forbinde netværk over internettet eller en anden IP-baseret infrastruktur.

---

## **1. Netværksscenarie**
Vi opsætter en GRE-tunnel mellem to Cisco-routere:

- **Router 1 (R1)**
  - WAN-interface: `GigabitEthernet0/0`, IP: `192.168.1.1`
  - LAN-interface: `GigabitEthernet0/1`, IP: `10.1.1.1/24`
  - Tunnel-interface: `Tunnel0`, IP: `10.10.10.1/30`

- **Router 2 (R2)**
  - WAN-interface: `GigabitEthernet0/0`, IP: `192.168.2.1`
  - LAN-interface: `GigabitEthernet0/1`, IP: `10.2.2.1/24`
  - Tunnel-interface: `Tunnel0`, IP: `10.10.10.2/30`

---

## **2. Opsætning af GRE Tunnel på R1**

På **Router 1**, konfigurer følgende:

```cisco
conf t
! Opret tunnel-interface
interface Tunnel0
 ip address 10.10.10.1 255.255.255.252
 tunnel source 192.168.1.1
 tunnel destination 192.168.2.1
 exit

! Statisk route til R2’s LAN via tunnel
ip route 10.2.2.0 255.255.255.0 10.10.10.2
exit
```

---

## **3. Opsætning af GRE Tunnel på R2**

På **Router 2**, konfigurer følgende:

```cisco
conf t
! Opret tunnel-interface
interface Tunnel0
 ip address 10.10.10.2 255.255.255.252
 tunnel source 192.168.2.1
 tunnel destination 192.168.1.1
 exit

! Statisk route til R1’s LAN via tunnel
ip route 10.1.1.0 255.255.255.0 10.10.10.1
exit
```

---

## **4. Test af Tunnel**

### **4.1. Ping tunnel IP**
Test forbindelsen mellem tunnel IP'erne:
```cisco
ping 10.10.10.2
```
Hvis pingen går igennem, betyder det, at tunnelen er oppe.

### **4.2. Test LAN-kommunikation**
Fra R1 skal du kunne pinge en enhed på **R2's LAN**:
```cisco
ping 10.2.2.1 source 10.1.1.1
```

Omvendt kan du fra R2 pinge **R1's LAN**:
```cisco
ping 10.1.1.1 source 10.2.2.1
```

---

## **5. Fejlfinding**
Hvis tunnelen ikke fungerer, kan du bruge følgende kommandoer:

- **Se tunnelstatus:**
  ```cisco
  show interface Tunnel0
  ```
  Tunnelen skal være **up/up**.

- **Se routing-tabel:**
  ```cisco
  show ip route
  ```
  Sørg for, at ruterne er korrekte.

- **Se tunnel-konfiguration:**
  ```cisco
  show running-config | section Tunnel0
  ```

- **Debugging:**
  ```cisco
  debug ip icmp
  debug tunnel
  ```

---

## **6. Valgfri: Beskyttelse med IPsec**
For at sikre GRE-tunnelen kan du tilføje IPsec kryptering. Her er et eksempel på at sikre tunnelen med IPsec:

### **Step 1: Definer adgangsliste**
```cisco
access-list 100 permit gre host 192.168.1.1 host 192.168.2.1
```

### **Step 2: Konfigurer IPsec-politik**
```cisco
crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 86400
exit
```

### **Step 3: Definer nøgle**
```cisco
crypto isakmp key GRE-TUNNEL-KEY address 192.168.2.1
```

### **Step 4: Konfigurer IPsec transform-set**
```cisco
crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
```

### **Step 5: Konfigurer IPsec tunnel**
```cisco
crypto map GRE-MAP 10 ipsec-isakmp
 set peer 192.168.2.1
 set transform-set TSET
 match address 100
exit
```

### **Step 6: Anvend IPsec på WAN-interface**
```cisco
interface GigabitEthernet0/0
 crypto map GRE-MAP
```
