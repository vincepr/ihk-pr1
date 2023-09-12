# IT Sicherheit


|Datensicherheit | Datenschutz |
|---|---|
|alle Daten die in einem Unternehmen anfallen | nur Personenbezogene Daten|
|CIA zum minimieren von Risiken durch z.B. Verlust| Daten mit denen ein Mensch identifiziert werden kann|

## CIA
**Vertraulichkeit** - kann nur die berechtigte Person die Daten einsehen?
- einrichten von Zugriffsberechtigungen und Nutzer mit entsprechenden Berechtigungen

**Integrität** - Daten sind Korrekt und Vollständig
- Logging von Veränderungen, Versionskontrolle

**Verfügbarkeit** - größtmögliche Verfügbarkeit, wenige Ausfälle wie möglich
- einrichten USV, Backups

*Authentizität* - Veränderungen müssen klar einem Author zugeordnet werden können.

## DSGVO - Verarbeitung PErsonenbezogener Daten
Diese muss das Unternehmen einhalten

**Datenminimierung** - Nur dem Zweck angemessen Personenbezogene Daten sammeln/verarbeiten

**Zweckbindung** - Personenbezogene Daten nur für festgelegte, eindeutige Zwecke erheben/nutzen

**Rechenschaftspflicht/Nachweispflicht** - Es gibt eine benannte Person die Verantwortlich ist. Diese muss für Einhaltung sorgen und trägt Rechenschaft.

**Erlaubnisvorbehalt** - Grundsätzlich Verboten. Nur nach Expliziter Erlaubniss dürfen Daten erhoben werden

**Datenrichtigkeit** - Daten müssen sowhol sachlich/inhaltlich korrekt und aktuell sein

**Rechtmäßigkeit und Transparenz** - Daten müssen nachvollziehbar und rechtmäßig verarbeitet werden

**Speicherbegrenzung** - Nur solange speichern, wie notwendig für ihren Zweck

Integrität, Veraulichkeit, Sicherheit - Durch Techn. Maßnahmen ist sicherzustellen das diese gewährleistet sind

## DSGVO - Rechte betroffener Personen
Diese Rechte hat jede Person im Umgang mit ihren Daten

- Auskunft
- Löschung
- Berichtigung
- Einschränkung der Verarbeitung
- Wiederruf

## Maßnahmen zum erhöhen der Sicherheit
**Orga Maßnahmen:** - IT-Sicherheitsbeauftragter, Passwortrichtilinen, Abteilungen Schafften (QM/Sicherheit)

**Technische Maßnahmen:** - Virenschutzsysteme, Firewall, Anti-Spam

** Infrastrukturelle Maßnahmen:** - Abschirmung des Rechenzentrums

**Personelle Maßnahmen: ** - Schulungen

## Verschlüsselung
#### Symmetrische Verschlüsselung
- Ein einziger Schlüssel wird zum ver- und entschlüsseln benutzt
- Nachteil: - Dieser muss über das ungesicherte Internet geteilt werden
- Vorteil: - Sehr schnell (im vgl zu assym)

#### Asymmetrische Verschlüsselung
Erzeugen eines Key-Pairs. Der Schlüsselträger behät private-key geheim und kann öffentlichen Schlüssel ohne bedenken teilen

**Öffentlicher Schlüssel:**
- jeder kann Daten für Schlüsselträger mit diesen verschlüsseln

**Privater Schlüssel:**
- Daten die für Schlüsselträger verschlüsselt werden lassen sich mit diesem entschlüsseln

#### Hybride Verschlüsselung
- bei z.B. SSL-TLS oder VPN genutzt
- bestes von beiden Arten:
    - erst Assymetrisch für Sicherheit
    - auf dieser Verbindung wird dann der Symmetrische Key geteilt, für eine Schnellere Verbindung
    
