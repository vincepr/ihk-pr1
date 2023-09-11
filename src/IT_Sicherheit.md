# IT Sicherheit

## CIA
**Vertraulichkeit**

**Verfügbarkeit**

**Integrität**

*Authentizität*

*Vertraulichkeit*

## Datenschutz
**Datenminimierung** - Nur dem Zweck angemessen Personenbezogene Daten sammeln/verarbeiten

**Zweckbindung** - Personenbezogene Daten nur für festgelegte, eindeutige Zwecke erheben/nutzen

**Erlaubnisvorbehalt** - Grundsätzlich Verboten. Nur nach Expliziter Erlaubniss dürfen Daten erhoben werden

**Datenrichtigkeit** - Daten müssen sowhol sachlich/inhaltlich korrekt und aktuell sein

**Rechtmäßigkeit und Transparenz** - Daten müssen nachvollziehbar und rechtmäßig verarbeitet werden

**Speicherbegrenzung** - Nur solange speichern, wie notwendig für ihren Zweck

Datenintegrität, Datenvertraulichkeit, Datensicherheit - Durch Techn. Maßnahmen ist sicherzustellen das diese gewährleistet sind

## Maßnahmen zum erhöhen der Sicherheit
**Orga Maßnahmen:** - IT-Sicherheitsbeauftragter, Passwortrichtilinen, Abteilungen Schafften (QM/Sicherheit)

**Technische Maßnahmen:** - Virenschutzsysteme, Firewall, Anti-Spam

** Infrastrukturelle Maßnahmen:** - Abschirmung des Rechenzentrums

**Personelle Maßnahmen: ** - Schulungen

# Datensicherung
## RAID
**RAID 0:**
- Keine Datenredundanz. Also bei Ausfall Verlust
- Vorteile: hohe Transferrate / paralleles Schreiben
**RAID 1:**
- volle Redundanz (Spiegeln)
- Nachteil: Speicher Halbiert
- Vorteil: Hohe Lesegeschwindigkeit
**RAID 5:**
- Paritätsinformationen auf 1 Platte aufgeteilt. Ausfallsicherheit 1 Platte
- hohe Lesegeschwindigkeit, Schreiben leicht verringert
- Mind 3 Platten benötigt
**RAID 6:**
- 2 Paritätsplatten. Ausfallsicherheit 2 Platten

## Backup Strategien
**Vollbackup:**
- Alle Daten werden ausnahmslos gesichert
- Das Achivbit wird zurückgesetzt (muss nicht ausgelesen werden da eh alles gesichert wird)
- es wird dementsprechend viel Speicher benötigt

**Differentielles Backup:**
- Alle Daten die Seit dem letzten Vollbackup verändert wurden werden gesichert
- Archivbit bleibt unverändert
- relativ viel Speicherbedarf

**Inkrementelles Backup:**
- Daten die seit dem letzten Vollbackup oder Inkrementellen Backup verändert wurden werden gesichert
- Archivbit wird zurückgesetzt
- Die Sicherungen bauen aufeinander auf
- Größe der Sicherung bleibt bei gleichmäßigem Betrieb in etwa konstant

