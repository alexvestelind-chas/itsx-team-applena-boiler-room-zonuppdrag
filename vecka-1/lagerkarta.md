# Lagerkarta – Zonuppdraget

## Lager 3 – IP-adresser
Här ser vi vilka datorer eller servrar som skickar och tar emot information.

Exempel från vår trafik:
- `172.20.1.145` skickar information till `172.20.1.2`.
- `172.20.1.145` skickar också information till andra IP-adresser, till exempel `185.12.64.1`.

## Lager 4 – TCP och UDP
Här tittar vi på hur informationen skickas och vilka portar som används.

Vi har sett:
- TCP port 22 – SSH, används för att ansluta till en server.
- UDP port 53 – DNS, används för att hitta rätt IP-adress till ett namn.
- UDP port 123 – NTP, används för att hålla rätt tid.
- TCP port 80 – HTTP, används för vanlig webbtrafik.

## Lager 7 – Tjänster
Här tittar vi på vad trafiken används till.

- SSH används för att ansluta till och hantera en server.
- DNS hjälper datorn att hitta rätt adress.
- NTP ser till att datorns klocka går rätt.
- HTTP används när information skickas mellan en webbläsare och en webbserver.

## Varifrån kommer informationen?
Vi har använt observationerna i `baslinje-trafik.md` som underlag.

## Vad behöver vi göra mer?
Vi behöver kontrollera att uppgifterna stämmer med vår trafikfångst och lägga till fler observationer om det behövs.