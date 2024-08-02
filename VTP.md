<h1> Guide på VTP </h1>

<h2> Trin 1: Forberedelse </h2>
<p> Før du begynder, skal du sørge for, at du har følgende: </p>

<p> 1. En Cisco-switch med administrativ adgang. </p>
<p> 2. Enhedens IP-adresse og netværksadgang. </p>
<p> 3. Kendskab til det VLAN-id, du vil konfigurere. </p>
<p> 4. Konsolkabel og terminalemuleringssoftware (som PuTTY).</p>


<h2> Trin 2: Indtast global konfigurationstilstand </h2>

<p> Indtast følgende kommandoer for at komme ind i global konfigurationstilstand: </p>

```
enable
configure terminal
```

<h2> Trin 3: Konfigurer VTP </h2>

<p> For at konfigurere VTP skal du bruge følgende trin: </p>

<h3> Indstil VTP-domæne:  </h3>
<p> Dette er navnet på dit VTP-domæne. Alle switches i samme domæne vil dele VLAN-information. </p>

```
vtp domain [Domenenavn]
```

<h3> Indstil VTP-version </h3>

<p> Sørg for, at VTP-versionen på klienten er den samme som på serveren. Standardversionen er normalt version 2, men du kan også indstille den til version 1 eller 3, afhængig af din netværksopsætning. </p>

```
vtp version 1
```
<p> Version 1 er det nemmest til at start med </p>

<h3> Indstil VTP-tilstand </h3>

<p> Der er tre VTP-tilstande: server, client, og transparent. Vælg den tilstand, der passer til dit netværk. </p>

<p> Server-tilstand (standardtilstand, tillader oprettelse og ændring af VLANs):  </p>

```
vtp mode server
```

<p> Client-tilstand (modtager og anvender VLAN-oplysninger fra en VTP-server): </p>

```
vtp mode client
```

<p> Transparent-tilstand (videresender VTP-oplysninger uden at ændre dem og opretholder lokale VLANs): </p>

```
vtp mode transparent
```

<h3> Indstil VTP-adgangskode (valgfrit): </h3>

<p> Hvis du vil bruge en adgangskode til at sikre VTP-oplysninger, kan du indstille en sådan: </p>

```
vtp password [Adgangskode]
```






