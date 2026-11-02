# Flödestabell – Team 8

## 1. Syfte

Flödestabellen beskriver vilken trafik som får skickas mellan zonerna i Team 8:s privata nätverk.

Nätverket är `10.40.8.0/24` och är uppdelat i fyra zoner:

- Klientzon: `10.40.8.0/27`
- Serverzon: `10.40.8.32/27`
- Datazon: `10.40.8.64/28`
- Hanteringszon: `10.40.8.80/28`

Målet är att endast tillåta trafik som behövs för verksamheten. All annan trafik ska blockeras enligt principen **default deny**.

## 2. Flödestabell

| Rad | Källzon | Målzon | Protokoll | Port | Riktning | Motivering | Filtreras var |
|---|---|---|---|---:|---|---|---|
| 1 | Internet | Serverzon (webbserver) | TCP | 443 | In | Användare ska kunna nå webbapplikationen via HTTPS. | Molnplattform |
| 2 | Klientzon | Serverzon (webbserver) | TCP | 443 | Ut | Anställda ska kunna använda webbapplikationen. | Värd |
| 3 | Serverzon (webbserver) | Serverzon (applikationsserver) | TCP | 443 | Ut | Webbservern skickar förfrågningar till applikationsservern. | Värd |
| 4 | Serverzon (applikationsserver) | Datazon | TCP | 3306 | Ut | Applikationsservern behöver läsa och skriva kunddata i databasen. | Värd |
| 5 | Hanteringszon | Serverzon (webbserver) | TCP | 22 | Ut | Administratörer behöver hantera webbservern via SSH. | Värd |
| 6 | Hanteringszon | Serverzon (applikationsserver) | TCP | 22 | Ut | Administratörer behöver hantera applikationsservern via SSH. | Värd |
| 7 | Hanteringszon | Datazon | TCP | 22 | Ut | Administratörer behöver hantera databasservern via SSH. | Värd |
| 8 | Serverzon och Datazon | Hanteringszon | TCP/UDP | 514 | Ut | Servrar skickar loggar till loggmottagningen. | Värd |
| 9 | Alla zoner | Alla zoner | – | – | In/Ut | All trafik som inte uttryckligen tillåts ska nekas. | Molnplattform och värd |

## 3. Beskrivning av flödena

- **Rad 1:** Tillåter HTTPS-trafik från internet till webbservern på TCP-port 443.
- **Rad 2:** Tillåter klienter att använda webbapplikationen via HTTPS.
- **Rad 3:** Tillåter webbservern att kommunicera med applikationsservern.
- **Rad 4:** Tillåter applikationsservern att ansluta till databasen på TCP-port 3306.
- **Rad 5–7:** Tillåter administratörer i hanteringszonen att använda SSH för att sköta servrar och databas.
- **Rad 8:** Tillåter servrar och databaser att skicka loggar till hanteringszonen.
- **Rad 9:** All trafik som inte uttryckligen tillåts ska blockeras.

## 4. Säkerhetsprinciper

Vi använder principen **default deny**. Det innebär att trafik som inte uttryckligen har tillåtits ska blockeras.

Brandväggsreglerna ska dokumenteras så att varje regel kan kopplas till rätt rad i flödestabellen.

## 5. Saker att kontrollera med gruppen

1. Ska webbservern och applikationsservern vara två separata servrar i Serverzon?
2. Ska gäster ingå i kurslabben, eller ska Gästzon inte användas?
3. Ska loggservern finnas i Hanteringszon?
4. Behöver webbservern vara nåbar från internet på TCP-port 443?
5. Vilka exakta serveradresser ska användas för respektive flöde?