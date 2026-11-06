# Loggstrategi v1 – elev-07

## 1. Loggkällakarta
| Källa | Var/hur den läses | Exempelrad | Vilken fråga besvarar den? | Rotation | Hur långt bakåt | Tidsformat |
|---|---|---|---|---|---|---|
| Systemlogg | `/var/log/messages` | Systemhändelser | Vad händer på servern? | Veckovis | 4 rotationer | RFC 3164 lokalt |
| Autentiseringslogg | `/var/log/secure` | SSH-inloggning | Vem loggade in? | Veckovis | 4 rotationer | RFC 3164 lokalt |
| Cron | `/var/log/cron` | Schemalagda jobb | Vilka jobb kördes? | Veckovis | 4 rotationer | Kontrollera format |
| Auditlogg | `/var/log/audit/` | Säkerhetshändelser | Vilka säkerhetsändringar skedde? | Kontrollera separat | Kontrollera separat | Kontrollera format |
| Misslyckade inloggningar | `/var/log/btmp` | Misslyckade försök | Har någon försökt logga in utan att lyckas? | Kontrollera separat | Kontrollera separat | Binärt format |
| Inloggningshistorik | `/var/log/wtmp` | Inloggning och utloggning | När loggade användare in och ut? | Kontrollera separat | Kontrollera separat | Binärt format |


## 2. Fem frågor och urval
| Fråga | Källor som krävs | Går att besvara idag? |
|---|---|---|
| Vem loggade in på servern? | `/var/log/secure` | Ja |
| När startades servern om? | `/var/log/messages` | Ja |
| Vilken trafik blockerades? | Brandväggslogg | Ja, om loggningen är aktiverad |
| Vilka ändringar gjordes på servern? | Auditlogg | Ja, om händelsen loggas |
| Skickas loggarna till loggservern? | Rsyslog och central loggserver | Behöver verifieras |
## 3. Strykna källor och skäl
| Stryken källa | Varför ingen fråga behöver den |
|---|---|
| Maillog (`/var/log/maillog`) | Servern används inte för e-post och därför behöver jag inte den loggen just nu. |
| Spooler (`/var/log/spooler`) | Servern har ingen viktig utskriftsfunktion som behöver övervakas. |

## 4. Verifieringsanteckning
| Identifierare | Ungefärlig tid (UTC) | Värdnamn | Källa | Söksträng | Hittad centralt? Avvikelse? |
|---|---|---|---|---|---|
| U5.5 elev-07 test | 2026-11-06 15:05:14 (lokal tid, UTC ej verifierad) | elev-07 | sparbarhet | U5.5 elev-07 test | Hittad lokalt, central kontroll återstår |

## 5. Skyddsbeskrivning

### Lokalt skydd
På min server elev-07 sparas loggar lokalt i /var/log/. Exempel är systemloggen messages och autentiseringsloggen secure. Filerna har begränsade behörigheter så att vanliga användare inte kan läsa eller ändra alla loggar.
### Centralt skydd
Loggarna skickas till en central loggserver med hjälp av TLS. Anslutningen använder TCP och port 6514. TLS skyddar informationen när den skickas mellan servrarna.
### Lagringstid och rotation
Loggarna roteras varje vecka enligt inställningarna i logrotate.conf. Fyra gamla versioner sparas. Datum läggs till på de gamla loggfilerna.
### Om mottagaren är onåbar
Om loggservern inte går att nå försöker rsyslog skicka loggarna igen. Det finns en kö som kan spara loggmeddelanden under tiden. I konfigurationen är även sparning vid avstängning aktiverad.
### Scenario om bevisförstöring
Om någon får administratörsbehörighet på servern kan personen försöka radera eller ändra loggarna lokalt. Därför är det viktigt att loggar också skickas till en central loggserver. Om både den lokala och centrala loggen förstörs kan viktiga bevis försvinna.