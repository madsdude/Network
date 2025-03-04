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

## ✏️ Hvordan Subnetter Man?
### Trin 1: Bestem antallet af subnets eller hosts pr. subnet
1. **Formel til subnets**:  
