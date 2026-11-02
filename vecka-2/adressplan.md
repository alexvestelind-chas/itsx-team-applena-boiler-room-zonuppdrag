# Adressplan – Team 8

## 1. Förutsättningar

Team 8 har fått nätverket `10.40.8.0/24` för arbetet i kurslabben.

Vi använder privata IPv4-adresser och delar upp nätverket i fyra subnät. Varje subnät representerar en egen zon med en särskild funktion.

## 2. Teamets nätverk och subnät

Teamets privata nätverk är `10.40.8.0/24`.

Vi har skapat följande fyra subnät:

| Zon | Subnät | Antal adresser totalt |
|---|---|---:|
| Klientzon | `10.40.8.0/27` | 32 |
| Serverzon | `10.40.8.32/27` | 32 |
| Datazon | `10.40.8.64/28` | 16 |
| Hanteringszon | `10.40.8.80/28` | 16 |

Alla fyra subnät ligger inom `10.40.8.0/24` och överlappar inte varandra.

## 3. Zonernas funktion

- **Klientzon:** För klienter och användarnas datorer.
- **Serverzon:** För webbserver och applikationsserver.
- **Datazon:** För databaser och lagring av verksamhetsdata.
- **Hanteringszon:** För administration, drift och hantering av servrar.

Loggning och övervakning hanteras inom den planerade hanteringsfunktionen. Den exakta placeringen behöver stämmas av med gruppens lösning.

## 4. Trafik mellan zonerna

Vi använder principen att endast nödvändig trafik ska tillåtas.

- Klientzon får ansluta till tjänster i Serverzon på godkända portar.
- Serverzon får kommunicera med Datazon när applikationen behöver det.
- Hanteringszon får administrera servrar och databaser via godkända anslutningar.
- Servrar ska kunna skicka relevanta loggar till loggmottagningen.
- Klienter ska inte ha direkt åtkomst till databaser.
- Trafik som inte uttryckligen tillåts ska blockeras.

## 5. Kontroll och verifiering

Vi ska kontrollera att:

1. Alla fyra subnäten är skapade.
2. Subnäten ligger inom `10.40.8.0/24`.
3. Subnäten inte överlappar varandra.
4. Servrarna är anslutna till rätt subnät.
5. Flödestabellen stämmer med zonindelningen.
6. Brandväggsreglerna följer flödestabellen.
7. Trafiken testas och dokumenteras.

## 6. Ändringar från den gamla adressplanen

Den gamla adressplanen använde `10.40.0.0/16`. Teamets nya nätverk är `10.40.8.0/24`.

Vi har anpassat adressplanen till fyra zoner: Klientzon, Serverzon, Datazon och Hanteringszon.

Den tidigare zonindelningen med bland annat DMZ, applikationszon, gästzon och loggzon har ersatts av den nya fyrzonsindelningen. Funktionerna behöver hanteras inom den nya strukturen enligt gruppens beslut.