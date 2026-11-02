# Flödestabell – Team 8

## 1. Syfte

Flödestabellen beskriver vilken trafik som får skickas mellan zonerna i Team 8:s privata nätverk.

Teamets nätverk är `10.40.8.0/24` och är uppdelat i fyra zoner:

- Klientzon: `10.40.8.0/27`
- Serverzon: `10.40.8.32/27`
- Datazon: `10.40.8.64/28`
- Hanteringszon: `10.40.8.80/28`

Målet är att endast tillåta trafik som behövs för verksamheten. All annan trafik ska blockeras enligt principen **default deny**.

## 2. Flödestabell

| Flöde | Källzon | Målzon | Protokoll | Port | Riktning | Motivering | Filtreras var |
|---|---|---|---|---|---|---|---|
| F1 | Klientzon | DMZ (webbserver) | TCP | 443 | In | Klienter ska kunna använda webbapplikationen via HTTPS. | Värd |
| F2 | DMZ (webbserver) | App (applikationsserver) | TCP | 443 | In | Webbservern ska kunna skicka förfrågningar till applikationsservern. | Värd |
| F3 | App (applikationsserver) | Databas | TCP | 3306 | In | Applikationsservern ska kunna läsa och skriva kunddata i databasen. | Värd |

## 3. Driftregler

Utöver F1–F3 behöver följande drifttrafik tillåtas enligt uppgiftens krav:

| Regel | Källzon | Målzon | Protokoll | Port | Funktion |
|---|---|---|---|---|---|
| D1 | Hanteringszon | Servrar | TCP | 22 | SSH-administration från jump-servern |
| D2 | Hanteringszon | Servrar | TCP | 22 | SSH-administration från CTF |
| D3 | Servrar | DNS-servrar | UDP/TCP | 53 | DNS-uppslagning |
| D4 | Servrar | NTP-server | UDP | 123 | Tidssynkronisering |
| D5 | Servrar | Loggserver | TCP | 514 | Skicka loggar |
| D6 | DHCP-server | Servrar | UDP | 67/68 | Tilldelning och förnyelse av IP-adresser |
| D7 | Servrar | Internet | TCP | Kontrollera uppgiftens krav | Paketuppdateringar |

De exakta adresserna och riktningarna för D1–D7 ska följa kursens driftinstruktioner.

## 4. Beskrivning av flödena

- **F1:** Klientzonen får ansluta till webbservern via HTTPS på TCP-port 443.
- **F2:** Webbservern får skicka förfrågningar till applikationsservern.
- **F3:** Applikationsservern får ansluta till databasen på TCP-port 3306.
- **D1–D7:** Driftreglerna behövs för administration, DNS, tidssynkronisering, loggning, DHCP och paketuppdateringar.

## 5. Säkerhetsprinciper

Vi använder principen **default deny**. Det innebär att all trafik som inte uttryckligen tillåts ska blockeras.

Brandväggsreglerna ska dokumenteras med rätt regel-ID så att varje regel kan kopplas till rätt rad i flödestabellen.

Molnplattformens brandvägg filtrerar trafik på det publika gränssnittet. Trafik mellan servrar och interna zoner filtreras med nftables på respektive server.

## 6. Kontroll och verifiering

Vi ska kontrollera att:

1. F1, F2 och F3 fungerar som planerat.
2. D1–D7 är korrekt konfigurerade.
3. Otillåten trafik blockeras.
4. Brandväggsreglerna finns kvar efter omstart.
5. Testresultaten dokumenteras och kan kopplas till rätt regel-ID.