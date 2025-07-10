# 🧠 OSPF DR og BDR – Simpel Forklaring

## Hvad er OSPF?
**OSPF (Open Shortest Path First)** er en routingprotokol, som routere bruger til at finde den hurtigste vej gennem et netværk.

---

## 🔸 Hvad er DR og BDR?

### 🏆 DR (Designated Router)
- Den "chef"-router, som styrer OSPF-kommunikationen på netværket.
- Alle andre routere sender opdateringer til DR’en.
- DR’en videresender oplysninger til resten.

### 🧍‍♂️ BDR (Backup Designated Router)
- En backup-router, som **overtager**, hvis DR går ned.
- Den lytter med, men bruger kun sin rolle hvis nødvendigt.

---

## 🔁 Hvorfor har man DR og BDR?

På netværk som f.eks. Ethernet (multi-access), kan der være mange routere forbundet. Uden DR og BDR ville alle routere tale med hinanden direkte:

**Uden DR/BDR:**
- 4 routere = 6 forbindelser  
  _(fuld mesh: 4 * (4-1)/2 = 6)_

**Med DR/BDR:**
- 4 routere = 3 forbindelser (alle til DR)  
  _(mindre trafik og nemmere at håndtere)_

---

## 📋 Hvordan vælges DR og BDR?

1. Router med **højeste OSPF-prioritet** bliver DR.
2. Næsthøjeste bliver BDR.
3. Ved lige prioritet bruges **Router-ID** (RID).
4. **Prioritet 0** = router **kan ikke** blive DR/BDR.

> 🔧 Standard-prioritet er `1`.

---

## 🧪 Eksempel

Tre routere på samme netværk:

| Router | OSPF-prioritet | Router-ID     |
|--------|----------------|---------------|
| R1     | 1              | 1.1.1.1       |
| R2     | 5              | 2.2.2.2       |
| R3     | 5              | 3.3.3.3       |

➡️ DR = **R3** (samme prioritet som R2, men højere Router-ID)  
➡️ BDR = **R2**  
➡️ R1 bliver "normalt medlem"

---

## 🔌 Hvornår vælges DR/BDR?
Kun på:
- **Broadcast-netværk** (f.eks. Ethernet)
- **NBMA-netværk** (f.eks. Frame Relay)

**IKKE** på:
- Point-to-point links (f.eks. direkte kabel mellem to routere)

---

## 🛠️ Eksempel i Cisco CLI

```bash
R1(config-if)# ip ospf priority 100
```

## 🌐 OSPF Multicast-adresser (Layer 2 og IP)

OSPF bruger IP-multicast og MAC-multicast adresser til at sende beskeder på Layer 2.

| Formål             | IPv4 Multicast | MAC-adresse (Layer 2)      |
|--------------------|----------------|-----------------------------|
| All OSPF Routers   | 224.0.0.5      | 01:00:5E:00:00:05           |
| All DR Routers     | 224.0.0.6      | 01:00:5E:00:00:06           |

> DR og BDR lytter på **224.0.0.6**  
> Alle OSPF-routere lytter på **224.0.0.5**

---

## 🌐 OSPF og IP Protokolnummer

**OSPF (Open Shortest Path First)** bruger et særligt IP-protokolnummer, som identificeres i IP-pakkens header.

### 🔢 Protokolnummer:
- **OSPF = 89**

Dette nummer er tildelt af **IANA (Internet Assigned Numbers Authority)** og bruges til at fortælle netværksenheder, at IP-pakken indeholder en OSPF-meddelelse.

---

### 📦 Eksempel – Almindelige IP-protokolnumre:

| Protokol | Nummer |
|----------|--------|
| ICMP     | 1      |
| TCP      | 6      |
| UDP      | 17     |
| **OSPF** | **89** |

---

### 🧠 Hvor bruges det?
I IP-headerens **Protocol-felt** (8-bit), der afgør hvilken protokol IP-pakken indeholder:

| Version | Header Length | ... | Protocol = 89 | ... |

---

