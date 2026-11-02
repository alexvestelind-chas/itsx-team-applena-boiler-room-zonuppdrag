| Rad | Källzon          | Målzon              | Protokoll | Port | Riktning | Motivering (verksamhetsfunktion)                              | Filtreras var |
|-----|------------------|---------------------|----------|------|-----------|--------------------------------------------------------------|---------------|
| 1   | Internet         | DMZ (Webbserver)    | TCP      | 443  | In        | Användare ska kunna nå webbapplikationen via HTTPS.         | Molnplattform |
| 2   | Klientzon        | DMZ (Webbserver)    | TCP      | 443  | Ut        | Anställda ska kunna använda webbapplikationen.              | Värd          |
| 3   | DMZ (Webbserver) | Applikationszon     | TCP      | 443  | Ut        | Webbservern skickar förfrågningar till app-servern.         | Värd          |
| 4   | Applikationszon  | Datazon             | TCP      | 3306 | Ut        | App-servern behöver läsa och skriva kunddata i databasen.   | Värd          |
| 5   | Hanteringszon    | DMZ (Webbserver)    | TCP      | 22   | Ut        | Administratörer behöver hantera webbservern via SSH.        | Värd          |
| 6   | Hanteringszon    | Applikationszon     | TCP      | 22   | Ut        | Administratörer behöver hantera app-servern via SSH.        | Värd          |
| 7   | Hanteringszon    | Datazon             | TCP      | 22   | Ut        | Administratörer behöver hantera databasservern via SSH.     | Värd          |
| 8   | DMZ/App/Datazon  | Hanteringszon       | TCP/UDP  | 514  | Ut        | Servrar skickar loggar till loggservern.                    | Värd          |
| 9   | Gästzon          | Internet            | TCP/UDP  | Alla | Ut        | Gäster ska endast kunna nå internet.                        | Molnplattform |
| 10  | Alla zoner       | Alla zoner          | -        | -    | In/Ut     | Allt som inte uttryckligen tillåts ovan är nekat.           | Molnplattform + Värd |

## Beskrivning av flödestabellen
- Flödestabellen beskriver vilken trafik som är tillåten mellan de olika zonerna i nätverket. Syftet är att endast tillåta den kommunikation som behövs för att verksamheten ska fungera, samtidigt som all onödig trafik blockeras enligt principen "default deny".

   * Rad 1 tillåter trafik från internet till webbservern i DMZ-zonen på port 443 (HTTPS). Detta behövs för att användare ska kunna besöka företagets webbplats på ett säkert sätt. Trafiken kommer från internet och kan därför filtreras i molnplattformens brandvägg.
   * Rad 2 tillåter klientzonen att ansluta till webbservern i DMZ-zonen via HTTPS. Detta gör att anställda kan använda webbapplikationen. Eftersom trafiken sker mellan interna zoner filtreras den på värden.
   * Rad 3 tillåter webbservern i DMZ-zonen att kommunicera med applikationsservern i applikationszonen. Webbservern tar emot användarnas förfrågningar och skickar dem vidare till applikationen för behandling.
   * Rad 4 tillåter applikationsservern att ansluta till databasen i datazonen på port 3306. Detta behövs för att applikationen ska kunna läsa och lagra kundinformation i databasen.
   * Rad 5, 6 och 7 tillåter administratörer i hanteringszonen att ansluta till servrarna via SSH på port 22. Detta krävs för administration, uppdateringar, felsökning och underhåll av systemen.
   * Rad 8 tillåter servrarna att skicka loggar till hanteringszonen. Loggning är viktig för övervakning, felsökning och att kunna upptäcka säkerhetsincidenter.
   * Rad 9 tillåter gästzonen att komma åt internet. Gäster ska endast kunna använda internet och får inte kommunicera med några interna system eller servrar.

- Den sista raden innebär att all trafik som inte uttryckligen har tillåtits i tabellen nekas, både inkommande och utgående. Detta följer principen "default deny", vilket minskar attackytan och gör nätverket säkrare eftersom endast nödvändig kommunikation tillåts mellan zonerna.