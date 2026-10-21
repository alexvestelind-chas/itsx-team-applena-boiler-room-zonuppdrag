| Princip | Kontrollfråga | bevis för ja | vem eller vad i vår miljö kan visa beviset? |
|---------|---------------|--------------|-------------------------------------------|
| Allt är en resurs | Är tillgångarna nedskrivna i tillgångsförteckningen? | Den är nedskriven i tillgångsförteckningen | Tillgångsförteckningen i repot |
| All kommunikation skyddas, oavsett nätverksplats | Hanteras intern trafik på samma sätt som extern trafik i miljön om man tänker på säkerheten? | Brandväggsregler inom nätverket | klassificering-och-zoner dokumentet |
| Åtkomst per session | Prövas en anslutning varje gång? | T.ex U3.2 där vi använde en tokenbaserad autentisering | Om man testar att ansluta utan token så blir man nekad |
| Dynamisk policy | Prövas andra variabler förutom ip och nät när en anslutning prövas? | Eventuell policyregel som man har | Händelser som flaggas som avvikande i loggarna |
| Övervaka tillstånd hos alla tillgångar | Kontrolleras tillgången efter sårbarheter innan åtkomst ges? | Tillgången ska vara kontrollerad innan den ansluter | - |
| Dynamisk autetisering och auktorisering, strikt i förväg | Omprövas anslutningen under sessionen och inte bara vid inloggning? | sessionen är tidsbegränad och kräver inloggning på nytt | time to live på token eller session |
| Samla information för att förbättra | Loggas relevanta händelser idag? | Loggar skickas till loggservern med information som behövs | Loggarna på loggservern |