# SSH
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
