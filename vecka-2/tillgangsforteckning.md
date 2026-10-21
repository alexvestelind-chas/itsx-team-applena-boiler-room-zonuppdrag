# Tillgångsförteckning

Samordnad med adressplanen (10.40.0.0/16) och zonmodellen från Workshop 1–2.

| Tillgång | Zon | Adress | Roll | Klassificering | Kommentar |
|---|---|---|---|---|---|
| Webbserver | DMZ | 10.40.1.129 | Tar emot inkommande anrop från internet och klienter, visar portalen | Medel | Enda tillgång som är nåbar direkt från internet |
| Appserver | Applikationszon | 10.40.1.65 | Applikationslogik, hämtar/skriver data via databasen | Medel–Hög | Nås bara från DMZ, inte direkt från klienter |
| Databas (kunddata) | Datazon | 10.40.1.113 | Lagrar kunddata med personuppgifter | Hög | Nås bara från applikationszonen, aldrig direkt från DMZ eller klienter |
| Loggserver | Hanteringszon | 10.40.1.121 | Central loggning (syslog/tcp 6514) från alla zoner | Hög | Kritisk för att upptäcka och utreda incidenter |
| Adminstation / jump | Hanteringszon | 10.40.1.97 | Enda vägen in för administration (tcp/22) av övriga servrar | Hög | Åtkomst ska motiveras och begränsas per mål, inte generellt öppen |
| DNS | Hanteringszon | 10.40.1.122 | Namnuppslag (udp/53) för alla interna tjänster | Medel | Bekräfta zontillhörighet mot egen tillgångsförteckning – kan ligga i egen infrastrukturzon |
| Klienter | Klientzon | 10.40.0.0/24 | 30+ arbetsplatser, normal användning av portalen | Låg–Medel | Vanlig ingångsväg (nätfiske), många enheter |
| Gäst och besök | Gästzon | 10.40.1.0/26 | Besökares enheter, endast internetåtkomst | Låg | Helt isolerad från övriga interna zoner |

**Reserverat utrymme:** se adressplanen (`10.40.0.0/16`) för fullständig lista över lediga block för framtida tillgångar.

