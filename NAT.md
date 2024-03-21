# Nat

# NAT (Network Address Translation) er en teknologi, der anvendes i netværk til at oversætte IP-adresser. Denne teknologi er især nyttig i situationer, hvor der er et begrænset antal offentlige IP-adresser tilgængelige. Med NAT kan et netværk med mange enheder, der hver især har en privat IP-adresse, dele en enkelt offentlig IP-adresse. Dette gør det muligt for enhederne at kommunikere med internettet, selvom de ikke hver især har en unik offentlig IP-adresse.

```.cisco

interface gigabitEthernet 0/0 

ip address dhcp

ip nat outside

interface gigabitEthernet 0/1

ip nat inside

ip nat inside source list 1 interface GigabitEthernet0/0 overload

access-list 1 permit ip any any

access-list 2 permit any

```
