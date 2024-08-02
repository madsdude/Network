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

<p> Indstil VTP-domæne:  </p>
<p> Dette er navnet på dit VTP-domæne. Alle switches i samme domæne vil dele VLAN-information. </p>

```
vtp domain [Domenenavn]
```

<p> Indstil VTP-tilstand </p>

