# Her får du en samling hands-on BGP-opgaver du kan køre i GNS3/EVE-NG/IOU/IOSv. Hver øvelse fokuserer på ét trin i best-path (“W L O AS O M P R”) + lidt ekstra. Jeg giver dig en lille grund-topologi, en basis-konfig, og derefter opgaver hvor du kun ændrer det nødvendige (som du foretrækker).
## Grund-topologi (bruges i de fleste øvelser)

<img width="838" height="510" alt="image" src="https://github.com/user-attachments/assets/d01b31e7-b210-421a-8a8b-f76fa097a8f2" />

# IP-plan (forslag)

R1–R2: 10.0.12.0/30 (R1=10.0.12.1, R2=10.0.12.2)

R2–R3: 10.0.23.0/30 (R2=10.0.23.2, R3=10.0.23.3) (bruges som IGP transport)

R2–R4: 10.0.24.0/30 (R2=10.0.24.2, R4=10.0.24.4)

Loopbacks: R1=1.1.1.1/32, R2=2.2.2.2/32, R3=3.3.3.3/32, R4=4.4.4.4/32

Prefix du kan annoncere:

R1: 1.1.1.0/24

R3: 3.3.3.0/24

R4: 4.4.4.0/24

