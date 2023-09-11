# IT Grundlagen
## Physik
- Zusammenhang Leistung
```
P  =  I  *  U 
[W] = [A] * [V]
Leistung = Stromstärke * Spannung
```
- Umrechnung Decimal vs Binär
```
8 Bit = 1 Byte
1 MiB = 1/1024 GiB
1MB = 1/1000 GB
```

## Aufbau eines Rechners

### BIOS vs UEFI

**UEFI** - Unified Extensible Firmware Interface
- Schnittstellensoftware zwischen Mainboard Firmware und OS.
Vorteile zum Vorgänger (**BIOS** : Basic Input Output System)
- Grafisches Interface
- höhere Bildschirmauflösung
- Booten über Netzwerk
- leichter aktualisierbar

### Teile einer CPU
- IDU - Instruction Decode Unit
- ALU - Arithmetic Logic Unit
- FPU - Floating Point Unit
- EXU - Execution Unit
- COL - Controll Logic
- DC - Data Cache
- CC - Code Cache

#### Cache
L1 Cache - First Level:
- Häufigst benutzten Daten
- z.B. 80 KB groß

L2 Cache - Second Level:
- Arbeisspeicher zwischenspeichern
- Größer aber langsamer (als L1)
- z.B. 2BM groß

L3 Cache - Smart Cache:
- jeder Kern besitzt eigen L1 & L2
- L3 wird jedoch von allen Kernen geteilt
- hält zwischen Kernen die Daten konsistent
- z.B. 30 MB groß

### RISC
Reduced Instruction Set Computing.
- RISC Prozessoren verfügen über eingeschränkten Befehlssatz
- Entsprechende Programmierung vorrausgesetzt bessere Leistung als reines CISC
- Mitlerweile verschmelzung CISC und RISC zu modernen x86/x64 Prozessoren

### CPU nach Von-Neumann-Arkchitektur
gemeinsame Speichernutzung von Programmen und Daten, wordurch Speichernutzung optimiert ist. (Vorläufer z.B. Harvard-Architektur hatte extra Datenspeicher und extra Codespeicher. Hier jedoch erhöhte Framentierung des Speichers)

![Darstellung Von NeumannArchitektur](./img/VonNeumannArchitektur.svg)

### Bussystem
Verbindet Komponenten (Prozessor, Controller, RAM, Eingabe...) miteinander elektrisch
- Realisiert als Bündel elektrischer Leitungen, parallel angeschlossen
- Datenbus
    - Busbreite 64 Bit - also 64 Bit gleichzeitig = 8 Byte gleichzeitig
    - Taktfrequenz neben Busbreite das andere Leistungskriterium

![Bussystem](./img/Bussystem.excalidraw.svg)

### ECC - Error Correcting Code
- 1 Bit Fehler im RAM werden automatisch verbessert.
- 2 Bit Fehler automatisch erkannt.

## Support
**Ticket System:**
- Störungen und notwendige Infos werden gebündelt
- priorisieren von wichtigen Aufgaben
- Da Daten über z.B. unternommenen Schritte beisammen, kann einfach von anderer Personan/an anderem Tag weitergearbeitet werden

**First Level Support:**
- Erste anlaufstelle für Störungen (Ticketsystem, Mail, Telefon)
- Direkter Kundenkontakt
- z.B. Passwordreset

**Second Level Support:**
- Wenn 1st Level nicht ausreicht
- Oft IT-Admins
- z.B. austausch Hardware nötig

**Third Level Support:**
- Hochste Stufe der Störungseskalation
- Spezialistenwissen. Oft erfahrene Experten die lange im Unternehmen arbeiten
- Erstellen oft die Schemen/Checklisten nach denen 1-2nd level arbeiten
- Berechtigt zum ändern von Konfigurationen / Sourcecode sollte dies nötig sein

## Verkabelung
![Alt text](./img/Verkabelung.svg)
- Bandbreite: Übertragungsrate in Datenverbindungen
- Datenrate: aktuelle Übertragungsrate
- Bitfehlerrate: Verfälschte-Bit-Segmente / Gesamtanzahl
- Reflexion:
    - in elektrischen Wellen, Leitungen, Lichtwellenleitern
    - das selbe Signal wird 2 mal zeitlich verzögert empfangen
    - häufig an Leitungsenden
- Rauschen: unerwartete Störung eines Signal
    - thermisch-
    - elektromagnetisch-
    - intermodulationsrauschen
- Nebensprechen/Crosstalk: Feld eines Leiters überträgt(koppelt) ungewollt in Neben-Kabel
