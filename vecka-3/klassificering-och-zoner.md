# Klassificering och zoner

## Klassificeringslista

| Tillgång | Nivå | Motivering |
|---|---|---|
| Publik webbserver | Medel | Är åtkomlig från internet och har därför en större attackyta, men lagrar ingen känslig data. |
| Applikationsserver | Medel–Hög | Kopplar ihop webbservern och databasen och kan användas för att komma vidare i nätverket vid ett intrång. |
| Kunddatabas | Hög | Innehåller personuppgifter och behöver därför skyddas extra noga. |
| Loggserver | Hög | Behövs för att upptäcka och utreda incidenter. |
| Jump-server | Hög | Används för administration och behöver skyddas eftersom den ger åtkomst till andra system. |
| Klienter | Låg–Medel | Kan vara en ingång för attacker, till exempel genom nätfiske. |
| Gäster | Låg | Ska inte ha åtkomst till interna system och behöver därför hållas åtskilda. |

## Zontabell

Team 8 använder nätverket `10.40.8.0/24`. Vi behöver därför anpassa den gamla adressplanen och använda färre zoner.

| Zon | Innehåller | Nät |
|---|---|---|
| DMZ | Publik webbserver | Bestäms när subnäten planeras |
| Applikations- och klientzon | Applikationsserver och klienter, om uppgiften tillåter att de ligger tillsammans | Bestäms när subnäten planeras |
| Datazon | Kunddatabas | Bestäms när subnäten planeras |
| Hanteringszon | Jump-server och loggserver | Bestäms när subnäten planeras |

De slutliga subnäten ska väljas så att de ryms inom `10.40.8.0/24` och inte överlappar varandra. Vi behöver också kontrollera att molnplattformens krav och uppgiften tillåter den här indelningen.

## Vilka zoner ska nå varandra?

| Från | Till | Tillåtet/Nekat | Motivering |
|---|---|---|---|
| Klienter | DMZ | Tillåtet på specifika portar | Användare behöver kunna nå webbservern. |
| DMZ | Applikationsserver | Tillåtet på specifika portar | Webbservern behöver kunna anropa applikationen. |
| Applikationsserver | Datazon | Tillåtet på specifika portar | Applikationen behöver kunna hämta och spara data. |
| Hanteringszon | Övriga zoner | Tillåtet för nödvändig administration | Jump-servern används för att administrera systemen. |
| Övriga zoner | Loggserver | Tillåtet för loggtrafik | Loggar behöver samlas centralt. |
| Klienter | Datazon | Nekat | Klienter ska inte ha direkt åtkomst till databasen. |
| DMZ | Datazon | Nekat | Webbservern ska inte ansluta direkt till databasen. |
| Gäster | Interna zoner | Nekat | Gäster ska inte komma åt interna system. |

## Kvar att kontrollera

- Bekräfta med gruppen att fyra zoner passar uppgiften.
- Bestäm om gäster ska vara en egen zon eller inte ingå i labben.
- Bestäm och kontrollera de exakta subnäten inom `10.40.8.0/24`.
- Kontrollera att flödestabellen och brandväggsreglerna stämmer med zonerna.
- Testa att tillåten trafik fungerar och att otillåten trafik blockeras.