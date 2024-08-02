<h1> BGP står for Border Gateway Protocol og er en standardiseret gateway-protokol, der bruges til at udveksle routing-information mellem forskellige autonome systemer (AS) på internettet. Her er en kort forklaring: </h1>

<p> Hvad er BGP?
Formål: BGP bruges til at finde den bedste vej for data at rejse over forskellige netværk på internettet. Det er grundlaget for internettets routing-infrastruktur.

Autonome Systemer (AS): Et autonomt system er et samling af IP-netværk og routere under en fælles administrativ kontrol og med en fælles routingstrategi. BGP håndterer routing mellem disse autonome systemer.

Hvordan det fungerer:

BGP-routere udveksler information om hvilke IP-adresser de kan nå, og hvilken vej data skal tage for at nå disse adresser.
BGP bruger en række attributter (f.eks. AS-stier, next-hop) til at vælge den bedste vej blandt flere tilgængelige ruter.
Typer af BGP:

eBGP (External BGP): Bruges til at udveksle routing-information mellem forskellige autonome systemer.
iBGP (Internal BGP): Bruges inden for et enkelt autonomt system for at sikre konsistent routing-information mellem alle routere i systemet.
Stabilitet og sikkerhed: BGP er designet til at være robust og stabil, men det kan være sårbart over for visse typer af angreb og fejlkonfigurationer, hvilket kan føre til routingproblemer som f.eks. route hijacking eller blackholing.
</p>
