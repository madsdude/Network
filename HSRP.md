# 💡 HSRP – Hot Standby Router Protocol (Cisco)

HSRP (Hot Standby Router Protocol) er en Cisco-proprietær protokol, der skaber **gateway-redundans** i et netværk. Det betyder, at flere routere kan arbejde sammen om én fælles gateway (en virtuel IP-adresse), så enheder i netværket altid har adgang – også hvis én router går ned.

---

## 🧠 Grundprincip

- **Virtuel IP-adresse:** Den IP, som klienterne sætter som deres gateway. Routerne i HSRP-gruppen bliver enige om, hvem der skal håndtere trafikken.
- **Active Router:** Den router, som pt. videresender al trafik via den virtuelle IP.
- **Standby Router:** Næste i rækken til at tage over, hvis den aktive fejler.
- **Preempt:** Gør det muligt for en router med højere prioritet at overtage rollen som "active", hvis den kommer online igen.
- **Priority:** Bruges til at bestemme, hvem der skal være aktiv – højere værdi vinder.
- **Tracking:** Hvis f.eks. en vigtig interface fejler, kan routerens prioritet automatisk sænkes, hvilket kan udløse et failover.

---

## ⚙️ Eksempelkonfiguration

### 🖥️ Router A (Aktiv)

```cisco
interface GigabitEthernet0/1
 description LAN-side
 ip address 192.168.1.2 255.255.255.0

 standby 1 ip 192.168.1.1
 standby 1 priority 110
 standby 1 preempt
 standby 1 track 1 decrement 20
```
### 🖥️ Router B (Standby)
```
interface GigabitEthernet0/1
 description LAN-side
 ip address 192.168.1.3 255.255.255.0

 standby 1 ip 192.168.1.1
 standby 1 priority 100
 standby 1 preempt
```
### ✅ Test HSRP – virker det?
1. Vis status på routerne
Kommando:
```
show standby
```
Du skal kunne se:

Hvem der er aktiv

Hvem der er standby

Virtuel IP og MAC

2. Ping gateway fra en klient
```
ping 192.168.1.1
```
Trafikken går til den aktive router.

3. Simuler fejl på Router A
Sluk interfacet på Router A:
```
interface GigabitEthernet0/1
 shutdown
```
Eller træk kablet.

4. Tjek failover
Router B burde nu blive aktiv.

Trafikken til 192.168.1.1 fortsætter uden nedetid.
```
show standby
```
