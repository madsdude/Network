# 🖧 Subnetting Guide

## 📌 Introduktion
Subnetting er en metode til at opdele et stort netværk i mindre netværk (subnets) for at forbedre effektiviteten af IP-adresser og netværksadministration.

## 📖 Grundlæggende IP-adressering
IPv4-adresser består af 32 bits, opdelt i fire oktetter, f.eks.:


Hver oktet repræsenterer 8 bits, hvilket betyder, at hele adressen har **4 x 8 = 32 bits**.

| Bitlængde | IP-adresse |
|-----------|-------------|
| 8 bits    | 0 - 255    |
| 16 bits   | 0 - 65535  |
| 24 bits   | 0 - 16777215 |
| 32 bits   | 0 - 4294967295 |

---

## 🎯 Hvad er subnetting?
Subnetting er processen med at opdele et netværk i mindre subnetværk. Dette gøres ved at ændre subnetmasken, hvilket definerer, hvor mange bits der bruges til netværksadressen, og hvor mange der er til hosts.

### 📌 Standard Subnetmasker

| Klasse | Standard Subnetmaske | Netværks-Bits | Host-Bits |
|--------|----------------------|--------------|----------|
| A      | 255.0.0.0 (/8)       | 8            | 24       |
| B      | 255.255.0.0 (/16)    | 16           | 16       |
| C      | 255.255.255.0 (/24)  | 24           | 8        |

---

## 📌 Eksempel: Subnetting af en /24-adresse
Lad os subnette **192.168.1.0/24** i **fire subnetværk**.

1. Vi har en **/24 (255.255.255.0)**, hvilket betyder, at vi har **8 host-bits**.
2. Vi låner **2 ekstra bits**, så vi har en **/26 subnetmaske (255.255.255.192)**.
3. **Antal subnetværk** = 2² = **4 subnetværk**.
4. **Antal hosts pr. subnet** = 2^6 - 2 = **62 hosts**.

| Subnet | Netværksadresse | Første Host | Sidste Host | Broadcast |
|--------|---------------|------------|------------|----------|
| 1      | 192.168.1.0/26  | 192.168.1.1 | 192.168.1.62 | 192.168.1.63 |
| 2      | 192.168.1.64/26 | 192.168.1.65 | 192.168.1.126 | 192.168.1.127 |
| 3      | 192.168.1.128/26 | 192.168.1.129 | 192.168.1.190 | 192.168.1.191 |
| 4      | 192.168.1.192/26 | 192.168.1.193 | 192.168.1.254 | 192.168.1.255 |

---

## 🔍 VLSM (Variable Length Subnet Masking)
VLSM giver mulighed for at bruge **forskellige subnetmasker** inden for samme netværk for at optimere brugen af IP-adresser.

Eksempel på VLSM-subnetting:
- **192.168.1.0/26** (store netværk, 62 hosts)
- **192.168.1.64/27** (mellemstore netværk, 30 hosts)
- **192.168.1.96/28** (små netværk, 14 hosts)

---

```bash
Router# show ip interface brief
Router# show ip route
```
