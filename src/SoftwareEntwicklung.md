# Software Entwicklung



## Unternehmens Software 
**CRM - Customer Relationship Management**
- Strategischer Ansatz zur vollständigen Planung, Steuerung und Durchführung aller interaktiven Prozesse mit dem Kunden
- Software dient zur Dokumentation und Verwaltung von Kundenbeziehungen über einen langen Zeitraum

**BI - Business Intelligence**
- Systematische Analyse des eigenen Unternehments
- Sammlung, Auswertung, Darstellung großer Datenmengen

**ERP - Enterprise Resource Planing**
- Umfasst Kernprozesse, die zur Führung eines Unternehmens erforderlich sind
- Umfasst Prozesse und Hilfsmittel für z.B. Finanzen, Personal, Beschaffung, Logistik

### Ebenen im Unternehmen
|Ebene|System|
|---|---|
|Management Ebene|Entscheidungsunterstützungssysteme|
|Management Ebene|Management Informationssysteme|
|Strategische Ebene|Führungsunterstützungssysteme|
|Operative Ebene|Operative Systeme / Transactions Processing Systems|

## Anforderungen

### Funktionale Anforderungen
**Funktionale Anforderungen:**
- legen Fest was das Produkt tun soll
- z.B. ist Shop-Website. benötigt Warenkorb, kauf abschließen

**NICHT Funktionale Anforderungen:**
- gehen über Funktionale Anforderungen hinaus
- beschreiben *wie gut* ein Produkt die Funktionen erfüllt
- z.B.
    - wie schnell die Seite lädt
    - Lesbarkeit und Kommentare des Quellcodes
    - Konfigurierbarkeit

### Lastenheft
beschreibt die Anforderungen (vom **Kunden** erstellt)

- Enthält Spezifikationen des Kunden
- Grundlage für Angebot des Dienstleisters 
- Basis für Pflichtenheft

### Pflichtenheft
beschreibt wie die Anforderungen umgesetzt werden. (vom **Anbieter** erstellt)

- zeigt Umsetzungsmöglichkeiten
- nach Ausschlussprinzip keine Konkreten Beispiele

## Compiler, Interpreter etc.
### Linker
fügt alle vom Compiler erzeugten Objektdateien zu Binary zusammen
- Zugriff zwischen Objektdateien wird aufgelöst
- greift hierfür auch auf vorcompilierte `std` Libraries zu

## Dateiformate
3 Beispiele für gängige Human Readable Textformate: 


**JSON** - Javascript Object Notation:
- Key-Value-Pairs
- Values können Js-Objekte, Arrays, Variablen etc. halten

**CSV** - Comma Separated Values:
- Textadatei ohne Steuerzeichen
- Mit Kommas getrennt
- Geeignet für strukturierte Daten (Tabellen)

**XML** - Extensible Markup Language:
- Textdatei in hierarchischer Struktur
- Elemente durch Schlüsselworte (Tags) getrennt

## Datenbanken
### Arten von DBMs - Datenbankmanagement Modelle
- Hierarchisches DBM
- Netzwerk DBM
- Relationales DBM
- Objektorientiertes DBM

### Referentielle Integrität
Verweisende Attribute z.B. Fremdschlüssel, müssen auf existierende Werte zeigen.
- in SQL könne oft Änderungen/Löschen an die Referenz weitergegeben werden. z.B. `... ON DELETE CASCADE`

### Normalformen
#### Erste Normalform
- Attribute atomar
- Frei von Wiederholungsgruppen
#### Zweite Normalform
- Jedes Nichtschlüssel Attribut von jedem Schlüsselkanidaten voll abhängig
#### Dritte Normalform
- Kein Nichtschlüssel Attribut von einem anderen Nichtschlüssel abhängig

### Anforderung an DB nach Codd
|||
|---|---|
|Datenintegration | einheitliche Verwaltung|
|Datenoperationen |Daten suchen, ändern, speichern|
|Datenkatalog|Relationen zwischen Tabellen einsehbar|
|Benutzeransichten||
|Konsistenzüberwachung||
|Zugriffskontrolle|kein unberechtigter Zugriff|
|Transaktionen|zusammenfassen und (aufeinmal) ausführen|
|Synchronisation|gleichzeitiges Benutzen muss synchronisiert werden. Also keine Beeinflussung untereinander|
|Datensicherung|Wiederherstellung von Daten nach Systemabsturz|
