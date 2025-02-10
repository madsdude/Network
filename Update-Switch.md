# **Guide: Opdatering af en Cisco-switch via USB**

Denne guide beskriver, hvordan du opdaterer en Cisco-switch ved hjælp af en USB-stick.

## **Forudsætninger**
- En USB-stick (FAT32-formateret)  
- Den nyeste switch-software (.bin-fil) fra [Cisco's officielle hjemmeside](https://software.cisco.com/download/home)  
- Konsoladgang til switchen via SSH eller en seriel forbindelse (f.eks. via PuTTY)  
- Grundlæggende kendskab til Cisco CLI  

---

## **1. Download og forbered opdateringen**
1. Download den nyeste firmware fra Cisco's hjemmeside (kræver Cisco-konto).  
2. Kopiér `.bin`-filen til en FAT32-formateret USB-stick.  
3. Tilslut USB-sticken til switchens USB-port.  

---

## **2. Tjek at switchen genkender USB-sticken**
1. Log ind på switchen via SSH eller en seriel forbindelse.  
2. Kør følgende kommando for at se, om USB'en er genkendt:  
   ```sh
   dir usbflash0:
   ```
   Hvis USB’en er korrekt monteret, vises filerne på USB’en.

---

## **3. Kopiér firmware til switchens flash-hukommelse**
1. Kopiér `.bin`-filen fra USB til switchens flash:  
   ```sh
   copy usbflash0:<filnavn>.bin flash:
   ```
   **Eksempel:**  
   ```sh
   copy usbflash0:c2960x-universalk9-mz.152-7.E2.bin flash:
   ```

---

## **4. Sæt den nye firmware som boot-image**
1. Tjek de eksisterende filer i flashen:  
   ```sh
   dir flash:
   ```
2. Indstil switchen til at boote på den nye firmware:  
   ```sh
   conf t
   boot system flash:<filnavn>.bin
   exit
   ```
   **Eksempel:**  
   ```sh
   boot system flash:c2960x-universalk9-mz.152-7.E2.bin
   ```

---

## **5. Gem ændringer og genstart switchen**
1. Gem konfigurationen:  
   ```sh
   write memory
   ```
   eller  
   ```sh
   copy running-config startup-config
   ```
2. Genstart switchen:  
   ```sh
   reload
   ```
   Bekræft genstart ved at trykke `y`.  

---

## **6. Verificér opdateringen**
1. Når switchen er startet op, tjekker du versionen:  
   ```sh
   show version
   ```
   Her skal den nye firmwareversion være angivet.  

2. Tjek også boot-variablen for at sikre, at den peger på den rigtige fil:  
   ```sh
   show boot
   ```

---

## **Fejlfinding**
- **Hvis USB’en ikke genkendes:**  
  - Prøv `dir usbflash1:` i stedet for `dir usbflash0:`  
  - Tjek om USB’en er FAT32-formateret  
- **Hvis switchen ikke booter korrekt:**  
  - Boot manuelt fra switchens ROMMON ved at bruge kommandoen:  
    ```sh
    boot flash:<filnavn>.bin
    ```

---
