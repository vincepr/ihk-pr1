# IT Sicherheit


|Datensicherheit | Datenschutz |
|---|---|
|alle Daten die in einem Unternehmen anfallen | nur Personenbezogene Daten|
|CIA zum minimieren von Risiken durch z.B. Verlust| Daten mit denen ein Mensch identifiziert werden kann|

**IT Grundschutz (BSI)** 
- typische Gefahren zusammengefasst
- mit Lösungsvorschlägen
- Ziel systematische Risikominimierung

**ISO 27001**
- Informationssicherheitsmanagement
- Prozessorientiert mit knappenBeschreibungen der Regelungen

**Security by Design:**
- Sicherheit als Anforderung im Entwicklungsprozess
- einbezogen in Planung/Entwicklung/Tests
- stetige Analyse of Sicherheitsrisiken und wie zu beheben

**Security by Default:**
- Software standardmäßig so voreingestellt, dass am sichersten
- Standartisierte Sicherheitsbausteine (z.B. mindestens Firewall, privates Netz...)

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

**Integrität, Veraulichkeit, Sicherheit** - Durch Techn. Maßnahmen ist sicherzustellen das diese gewährleistet sind

## DSGVO - Rechte betroffener Personen
Diese Rechte hat jede Person im Umgang mit ihren Daten

- Auskunft
- Löschung
- Berichtigung
- Einschränkung der Verarbeitung
- Wiederruf

## Maßnahmen zum erhöhen der Sicherheit
**Orga Maßnahmen:** - IT-Sicherheits-Beauftragter, Passwortrichtilinen, Abteilungen Schafften (QM/Sicherheit)

**Technische Maßnahmen:** - Virenschutzsysteme, Firewall, Anti-Spam

**Infrastrukturelle Maßnahmen:** - Abschirmung des Rechenzentrums

**Personelle Maßnahmen:** - Schulungen

## Verschlüsselung
#### Symmetrische Verschlüsselung
- Ein einziger Schlüssel wird zum ver- und entschlüsseln benutzt
- *Nachteil*: - Dieser muss über das ungesicherte Internet geteilt werden
- *Vorteil*: - Sehr schnell (im vgl zu assym)

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

### Bausteine IT Grundschutz nach BSI
|Application|It-Systeme|
|---|---|
|Office Produkte|allgemeiner Server|
|Webbrowser|Windows Server|
|Mobile Apps|Linus/Unix Server|
|Webserver|Virtualisierung|
|Fileserver|Windows Clients|
|DNS Server|Linux Clients|
|Datenbanksysteme|MacOS Clients|
|Kubernetes|IOS / Android|
|Email Client/Server|Wechseldatenträger|

## Firewall Arten
Von Niedrigen Schichten nach höheren Schichten

### Packet Filter Firewall
in OSI 3

- Quell IP, Ziel IP
- Quell Port, Ziel Port
- Protokolle

### Stateful Packet Inspection
in OSI 4

- TCP Handshakes, Session fulfilment rules

### Application Gateway Firewall
in OSI 7. Proxy Server ist ein beispiel für eine Application Firewall

- advanced features against threats like SQL-injections, cross-site-scripting, cookie-tempering
