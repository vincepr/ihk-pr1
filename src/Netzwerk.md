# Netzwerk

## Begriffe
Domain Controller:
- Server zur zentralen Verwaltung von Benutzerrechten und Authentifizierung in einem Netzwerk

## IPv4
**Localhost IP Adressen:**
```
localhost IPv4  127.0.0.1
localhost IPv6  ::1
```

**Private IP Bereiche:**
|A|10.0.0.0/8|
|B|172.16.0.0/12|
|C|192.168.0.0/16|

### NAT
#### NAT - Network Adress Translation
- wenn viele Zugänge ins Internet nur eine/wenige öffentliche IP benutzen/teilen.
- NAT64 für kombination aus IPv4 und IPv6

#### PAT  - Port Adress Translation
- wie bei NAT, nur werden Ports gezielt genutzt um sie Verbindungen/lokalen-IPs zuzuordnen
- wenn heutzutage (nicht IHK) von NAT geredet wird, wird NAT mit PAT impliziert

### WLAN
|Name|Vorteile|Nachteile|geeignet|
|---|---|---|---|
|WPA|einfach|Unsicher da bei vielen Nutzern schnell geleaktes PW| für kleine Unternehmen|
|WPA-Enterprise mit RADIUS|sicherer|hohe kosten, aufwändiges Einrichten|große Unternehmen|