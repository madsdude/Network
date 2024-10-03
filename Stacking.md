
Stacking af Cisco-switches refererer til processen med at forbinde flere switches sammen, så de fungerer som en enkelt logisk switch. Dette gør det lettere at administrere og udvide netværket, da alle de stakkede enheder kan styres via en enkelt IP-adresse og som en samlet enhed, selvom de fysisk er separate. Dette øger også fejltolerancen og kapaciteten i netværket.

### Fordele ved stacking
- **Forenklet administration**: Da hele stakken ses som én switch, reduceres antallet af IP-adresser, der skal administreres.
- **Forbedret ydeevne**: Stackede switches deler båndbredde og kan videresende trafik mellem hinanden med høj hastighed.
- **Redundans og fejlbeskyttelse**: Hvis en switch i stakken fejler, vil de resterende switches fortsætte med at fungere normalt.

### Eksempel på stacking
Lad os sige, du har flere Cisco Catalyst-switches, og du ønsker at stacke dem. Her er nogle typiske trin og kommandoer for at oprette og administrere en stack:

#### 1. Kabling
Stacking-kabler forbinder de enkelte switches, typisk i en ringkonfiguration, hvilket sikrer redundans. Hvis et kabel fejler, vil stakken stadig være funktionel.

#### 2. Identifikation af master-switch
Når en switch-stak starter op, vælger den en master-switch, som administrerer hele stakken. Dette kan verificeres med følgende kommando:

```bash
show switch
```

Output viser alle switches i stakken, deres switch-nummer, rolle (master, medlem, standby), status og prioriteter.

#### 3. Konfiguration af stack-prioritet
Du kan konfigurere prioriteter for at bestemme, hvilken switch der skal blive master, hvis stakken starter op. Jo højere prioritet, desto større sandsynlighed har switchen for at blive master.

```bash
switch [switch-nummer] priority [1-15]
```
Eksempel:
```bash
switch 1 priority 15
```
Her sættes switch 1 til at have højeste prioritet i stakken.

#### 4. Tildeling af switch-numre
Hvis en ny switch tilføjes til stakken, kan det være nødvendigt at tildele et switch-nummer. Dette gøres med:

```bash
switch [current-switch-number] renumber [new-switch-number]
```
Eksempel:
```bash
switch 2 renumber 3
```
Dette ændrer switch nummer 2 til at blive nummer 3 i stakken.

#### 5. Visning af stack-status
Du kan kontrollere status og information om hele stakken med følgende kommando:

```bash
show switch stack-ports
```

#### 6. Opgradering af software i stakken
Hele stakkens switches kan opgraderes på én gang ved hjælp af én switch (typisk masteren), hvilket gør vedligeholdelsen af stakken lettere. Dette kan gøres med:

```bash
archive download-sw /overwrite /reload tftp://[server-ip]/[image-name]
```

### Bemærkninger
- Det er vigtigt at planlægge for redundans og kabel-layout, så stakken stadig er funktionsdygtig, hvis en forbindelse skulle fejle.
- Stacking er typisk tilgængelig på specifikke modeller af Cisco-switches, så det er godt at kontrollere, om din model understøtter dette.
