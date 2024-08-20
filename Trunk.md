<h1> Hvad er en Cisco Trunk? </h1>

<p> En trunk er en type forbindelse mellem switches i et netværk, der tillader, at flere VLANs (Virtual LANs) passerer gennem én enkelt fysisk forbindelse. Trunking er nyttigt, når du har flere VLANs, som du vil have til at kommunikere mellem switches uden at skulle bruge separate forbindelser for hvert VLAN. </p>

<h2> Hvordan virker Trunking? </h2>

<p> Når en trunk-forbindelse er oprettet, bruger den et protokol som IEEE 802.1Q til at tagge de dataframes, så switchene ved, hvilket VLAN de tilhører. Dette gør det muligt for en switch at sende trafik fra flere VLANs over en enkelt fysisk forbindelse til en anden switch, som også kan forstå VLAN-tagget og sende det til det rette VLAN. </p>

<h3> Eksempel på Opsætning af en Cisco Trunk </h3>

<p> Lad os antage, at du har to Cisco switches, og du vil konfigurere en trunk-forbindelse mellem dem. Her er en grundlæggende opsætning: </p>

```.cisco

Switch1# configure terminal
Switch1(config)# interface gigabitEthernet 0/1
Switch1(config-if)# switchport mode trunk
Switch1(config-if)# switchport trunk encapsulation dot1q
Switch1(config-if)# end

```

<ol>  
<li> interface gigabitEthernet 0/1: Vælg den port, du vil bruge som trunk. </li>
<li> switchport mode trunk: Sæt porten til trunk-mode. </li>
<li> switchport trunk encapsulation dot1q: Angiv, at du bruger IEEE 802.1Q til tagging af VLANs. </li>
</ol>

