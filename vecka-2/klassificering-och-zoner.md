# Klassificering och zoner

## Klassificeringslista

| Tillgång | Nivå | Motivering |
|---|---|---|
| Publik webbserver | Medel | Störst attackyta (internetexponerad), men lagrar ingen känslig data |
| Applikationsserver | Medel–Hög | Länk mellan webb och databas – kan användas för att pivotera vidare |
| Kunddatabas (personuppgifter) | Hög | GDPR-data, intrång kostar mest juridiskt och ekonomiskt |
| Loggserver | Hög | Kritisk för att upptäcka/utreda incidenter – manipulerad logg döljer spår |
| Jump-server (admin) | Hög | Enda vägen in för administration – intrång ger kontroll över allt |
| Klienter (30 arbetsplatser) | Låg–Medel | Vanlig ingångsväg (nätfiske), lågt värde per enhet men många och ofta mål |
| Gäst och besök | Låg | Isolerat segment, inget eget värde men potentiell ingångsväg om ej segmenterat |

## Zontabell

| Zon | Innehåller | Nät |
|---|---|---|
| DMZ | Publik webbserver | 10.40.1.128/29 |
| Klientzon | 30 arbetsplatser | 10.40.0.0/24 |
| Gästzon | Gäst och besök | 10.40.1.0/26 |
| Applikationszon | Applikationsserver | 10.40.1.64/27 |
| Datazon | Kunddatabas | 10.40.1.112/29 |
| Hanteringszon | Jump-server, loggserver | 10.40.1.96/28, 10.40.1.120/29 |

## Vilka subnät/zoner ska nå varandra

| Från | Till | Tillåtet/Nekat | Motivering |
|---|---|---|---|
| Klientzon | DMZ | Tillåtet (specifika portar) | Normal webbanvändning |
| DMZ | Applikationszon | Tillåtet (specifika portar) | Webbserver anropar applikationslogik |
| Applikationszon | Datazon | Tillåtet (specifika portar) | Endast applikationslagret pratar med databasen |
| Hanteringszon | Alla zoner | Tillåtet | Jump-server administrerar allt |
| Alla zoner | Hanteringszon (logg) | Tillåtet | Central loggning kräver att alla skickar till loggservern |
| Gästzon | Internet (endast) | Tillåtet | Ingen åtkomst till interna zoner |
| Klientzon | Datazon | **Nekat** | Ingen direkt väg – ska gå via applikationslagret |
| DMZ | Datazon | **Nekat** | Webb och data ska aldrig prata direkt |
| Gästzon | Övriga interna zoner | **Nekat** | Gäster ska vara helt isolerade |
