
# 🔁 Cisco BGP `peer-policy` Guide (Simplified)

## 📘 Hvad er en `peer-policy`?

En `peer-policy` er en genanvendelig skabelon i BGP, der bruges til at tildele politikker som route-maps til flere peers.

---

## 🔧 Syntax

```cisco
bgp peer-policy NAVN
  route-map NAVN in|out
```

```cisco
router bgp <ASN>
 address-family ipv4
  neighbor <IP> inherit peer-policy NAVN
```

---

## 🧪 Eksempel: Primær og sekundær linje

👉 Vi ønsker at bruge ISP1 som **primær**, og ISP2 som **backup**.

### 🔧 Peer-policies

```cisco
router bgp 65001
bgp peer-policy PRIMARY
  route-map PREF200 out
  description Primær linje

router bgp 65001
bgp peer-policy BACKUP
  route-map PREPEND3 out
  description Sekundær linje
```

### 🔧 Route-maps

```cisco
route-map PREF200 permit 10
 set local-preference 200

route-map PREPEND3 permit 10
 set as-path prepend 65001 65001 65001
```

### 🔧 BGP konfiguration

```cisco
router bgp 65001
 address-family ipv4
  neighbor 93.42.139.2 remote-as 64501
  neighbor 93.42.139.2 inherit peer-policy PRIMARY

  neighbor 80.91.57.1 remote-as 64502
  neighbor 80.91.57.1 inherit peer-policy BACKUP
```

📌 ISP1 har højere local-preference. ISP2 bliver nedprioriteret via AS-path prepend.

---

## 🛠 Tips

- Brug simple navne: `PRIMARY`, `BACKUP`, `PREF200`, `PREPEND3`
- Ensartet og nem konfiguration
- Virker godt med mange peers

---

## 🔍 Fejlsøgning

```cisco
show bgp ipv4 unicast neighbors <IP> inherited-config
```

Vil du også have en version med EVPN eller flere peers?
