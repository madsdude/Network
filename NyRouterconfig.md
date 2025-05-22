Config til en ny router til og gør livet nemmer 

```
conf t
hostname XXX
no logging console
no ip domain-lookup
exit
write memory
```

```
conf t
hostname XXX
no logging console
no ip domain-lookup
no banner login 
no banner exec
no banner incoming 
exit
write memory
```
