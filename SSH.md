# SSH

##### SSH (Secure Shell) er en netværksprotokol, der muliggør sikker fjernlogning og datatransmission mellem computere på et usikkert netværk, som internettet. Den bruges primært til at tilgå shell-konti på Unix-lignende systemer, men kan også anvendes til at sikre enhver form for netværkskommunikation, der kræver sikkerhed. SSH tilbyder en sikker kanal over et usikkert netværk i en klient-server arkitektur, hvilket gør det muligt for brugere at logge ind på en fjernserver, udføre kommandoer og flytte filer.

##### SSH bruger stærk kryptering for at beskytte data mod aflytning, ændring eller kompromittering under overførslen. Det erstatter ældre, mindre sikre protokoller som Telnet og FTP og bruges i en bred vifte af netværksfunktioner, inklusiv systemadministration, filoverførsler (via SCP eller SFTP) og porttunneling.

##### Kort sagt er SSH en vital teknologi for sikker kommunikation over ikke-sikrede netværk, der beskytter mod en række sikkerhedstrusler og gør det muligt at administrere servere og overføre filer sikkert.

```.Cisco
conf t
ip domain-name virkom.local
crypto key generate rsa

line vty 0 4
transport input ssh
login local
exit
enable secret Password1
username Admin password Password1
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 2
exit
write memory
```
