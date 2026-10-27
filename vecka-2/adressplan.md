# Adressplan – Team 8

## 1. Förutsättningar

I den gamla adressplanen använde vi nätverket `10.40.0.0/16`. För vårt arbete i Team 8 har vi fått nätverket `10.40.8.0/24`, så vi behöver anpassa adressplanen till det mindre nätverket.

Vi utgår från den gamla adressplanen som redan är godkänd och använder samma zoner och trafikprinciper som grund. Däremot behöver vi kontrollera hur mycket adressutrymme varje zon kan få.

Vi använder privata IPv4-adresser.

## 2. Behov av adresser

Vi utgår från följande behov i den gamla adressplanen:

| Zon | Nuvarande behov | Behov inklusive tillväxt |
|---|---:|---:|
| Klienter | 120 | 150 |
| Gäster | 40 | 50 |
| Servrar | 14 | 18 |
| Drift och administration | 6 | 8 |
| Databaser | 4 | 5 |
| Loggar och övervakning | 3 | 4 |
| DMZ | 4 | 5 |
| Routerlänkar | 3 länkar | 2 adresser per länk |

## 3. Teamets nätverk och subnät

Teamets nätverk är `10.40.8.0/24`. Det innehåller totalt 256 adresser, men alla kan inte användas till servrar eftersom nätverksadresser, broadcastadresser och eventuellt reserverade adresser måste räknas bort.

Vi behöver dela upp nätverket i mindre subnät för de zoner som ska användas i labben. Eftersom det nya nätverket är mindre än det gamla behöver vi kontrollera vilka zoner som krävs och hur många adresser varje zon behöver.

De slutliga subnäten och adressintervallen ska bestämmas när vi har kontrollerat uppgiftens krav och gruppens flödestabell. Alla subnät ska ligga inom `10.40.8.0/24` och får inte överlappa varandra.

## 4. Trafik mellan zonerna

Vi behåller trafikprinciperna från den gamla adressplanen:

- Klienter får ansluta till DMZ på de portar som behövs.
- DMZ får ansluta till servrar på godkända portar.
- Servrar får ansluta till databaser när det behövs.
- Drift och administration ska ha åtkomst till de zoner som behövs för att sköta systemen.
- Zonerna ska kunna skicka relevanta loggar till loggning och övervakning.
- Gäster ska endast ha tillgång till internet och inte till interna zoner.
- Klienter ska inte ha direkt åtkomst till databaser.
- DMZ ska inte ha direkt åtkomst till databaser.

Brandväggsreglerna ska följa gruppens flödestabell. Trafik som inte uttryckligen tillåts ska blockeras.

## 5. Kontroll och återstående arbete

För att färdigställa adressplanen behöver vi:

1. Kontrollera vilka zoner som ska ingå i kurslabben.
2. Bestämma hur många adresser varje zon behöver i labben.
3. Dela upp `10.40.8.0/24` i passande subnät.
4. Kontrollera att subnäten inte överlappar varandra.
5. Bestämma vilka IP-adresser servrarna ska använda.
6. Jämföra adressplanen med flödestabellen.
7. Skapa och testa nätverket när gruppen har bekräftat planen.

## 6. Ändringar från den gamla adressplanen

Den gamla adressplanen använde `10.40.0.0/16`. Den nya planen använder `10.40.8.0/24`, som är ett betydligt mindre nätverk.

Vi behåller samma grundläggande zonindelning och trafikprinciper, men adressfördelningen behöver anpassas till kurslabben. Därför behöver vi kontrollera vilka zoner och adressbehov som ska prioriteras innan vi fastställer de slutliga subnäten.