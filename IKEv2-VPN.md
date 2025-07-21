
# Cisco IKEv2 VPN Konfigurationsguide

## Oversigt
Denne guide beskriver, hvordan man konfigurerer IKEv2 VPN på en Cisco-router med:

- IKEv2 proposal
- IKEv2 policy
- Keyring med pre-shared keys
- IKEv2 profile
- Crypto map til IPSec tunnel

---

## 1. IKEv2 Proposal

Definerer kryptering, integritet og DH-gruppe.

```plaintext
crypto ikev2 proposal DC-VPN-PROPOSAL 
 encryption aes-cbc-256
 integrity sha256
 group 14
```

---

## 2. IKEv2 Policy

Tilknyt proposal og matcher VRF.

```plaintext
crypto ikev2 policy DC-VPN-POLICY 
 match fvrf any
 proposal DC-VPN-PROPOSAL
```

---

## 3. Keyring

Definerer peer og pre-shared key.

```plaintext
crypto ikev2 keyring VPN-KEYS
 peer Core-DK-R4
  address 12.34.56.78 
  pre-shared-key test.test 
```

---

## 4. IKEv2 Profile

Matcher interface/VRF, peer IP og sætter autentificering.

```plaintext
crypto ikev2 profile DC-VPN
 match fvrf Internet
 match identity remote address 12.34.56.78 255.255.255.255
 identity local fqdn PUD-R2.dccat.dk
 authentication remote pre-share
 authentication local pre-share
 keyring local VPN-KEYS
 dpd 10 3 periodic
```

---

## 5. Crypto Map (IPSec)

Bind IKEv2 profilen til en crypto map og anvend på interface.

```plaintext
crypto ipsec transform-set TRANSFORM esp-aes 256 esp-sha256-hmac 
 mode tunnel

crypto map VPN-MAP 10 ipsec-isakmp 
 set peer 12.34.56.78
 set transform-set TRANSFORM
 set ikev2-profile DC-VPN
 match address VPN-ACL

interface GigabitEthernet0/1
 crypto map VPN-MAP

ip access-list extended VPN-ACL
 permit ip 10.0.0.0 0.0.0.255 10.1.0.0 0.0.0.255
```

---

## 6. Forklaring

- **Proposal**: Kryptering og nøgleudveksling for IKEv2.
- **Policy**: Samler proposal og definerer anvendelsesområde (VRF).
- **Keyring**: Peer-identiteter og pre-shared keys.
- **Profile**: Matcher peers, lokal ID, autentificering, DPD.
- **Crypto Map**: IPSec tunnel konfiguration og binding til interface.

---

## 7. Tips

- Sørg for at IP-adresser og navne matcher jeres setup.
- Pre-shared keys skal være sikre og ens på begge sider.
- DPD hjælper med at opdage og rydde op i døde peers.
- Test forbindelsen efter konfiguration med `show crypto session`.

---

## 8. Ressourcer

- [Cisco IKEv2 Configuration Guide](https://www.cisco.com/c/en/us/support/docs/security-vpn/ipsec-negotiation-ike-protocols/116091-config-ikev2-00.html)
- Cisco ASA og router dokumentation for avancerede VPN-opsætninger.

