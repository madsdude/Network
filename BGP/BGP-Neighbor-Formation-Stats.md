# BGP Neighbor Formation and Stats

## 🔍 Hvad er BGP?

**BGP (Border Gateway Protocol)** er et path-vector routing-protokol, der bruges mellem forskellige autonome systemer (AS) på internettet. Den bruges også internt i store netværk (iBGP).

---

## 🧱 BGP Neighbor Formation (Finite State Machine)

BGP-dannelsen foregår i 6 trin:

| State         | Beskrivelse |
|---------------|-------------|
| **Idle**      | BGP-processen starter. Ingen TCP-forbindelse forsøges endnu. |
| **Connect**   | BGP prøver at etablere en TCP-session til naboen (port 179). |
| **Active**    | TCP-forbindelsen mislykkedes; forsøger igen. |
| **OpenSent**  | TCP er oppe, og BGP har sendt en OPEN-meddelelse. |
| **OpenConfirm** | BGP har modtaget en OPEN fra naboen og sender KEEPALIVE. |
| **Established** | Naboer har nu en aktiv session og kan udveksle routing information. |

---

## 📡 Wireshark: Eksempel på Neighbor Formation

![Wireshark BGP](b7e45e29-6aa8-4c8c-9c54-8848b89e39e2.png)

### Trin-for-trin fra billedet:

1. **TCP 3-Way Handshake** (linie 719–721):
   - `SYN`, `SYN-ACK`, `ACK` på TCP port 179 — vigtigt for at starte BGP-sessionen.

2. **BGP OPEN Message** (linie 722–723):
   - Sender konfigurationsparametre (AS-nummer, hold time, BGP version osv.).

3. **BGP KEEPALIVE Message** (linie 724, 726, 728, 730):
   - Bekræfter modtagelsen af OPEN-meddelelse og bruges til at holde forbindelsen i live.

4. **BGP UPDATE Message** (linie 725, 727):
   - Sender routingoplysninger.

5. **Established**:
   - Når OPEN og KEEPALIVE er sendt og modtaget, går forbindelsen i `Established`-tilstand, hvor UPDATEs nu kan sendes regelmæssigt.

---

## 📊 BGP Neighbor Stats (på en Cisco-router)

Brug kommandoen:

```bash
show ip bgp summary
```

Eksempeloutput:

```
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
198.51.100.2    4 65001     210     215        5    0    0 02:14:23       34
```

| Felt          | Betydning |
|---------------|-----------|
| Neighbor      | IP-adresse på BGP-naboen |
| V             | BGP version |
| AS            | Naboens AS-nummer |
| MsgRcvd       | Antal BGP-meddelelser modtaget |
| MsgSent       | Antal BGP-meddelelser sendt |
| TblVer        | Routing tabel version |
| InQ/OutQ      | Meddelelser i kø |
| Up/Down       | Hvor længe sessionen har været oppe |
| State/PfxRcd  | Status eller antal modtagne prefixes |

---

## 📌 Tjekliste for BGP Neighbor Troubleshooting

- [ ] Kan du nå naboen med ping?
- [ ] Er TCP port 179 åben i begge retninger?
- [ ] Stemmer AS-numre og BGP version?
- [ ] Har du modtaget en OPEN meddelelse?
- [ ] Er HOLD timer og KEEPALIVE korrekt sat?
