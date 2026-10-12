# Hur vi tog fram och analyserade trafiken

## 1. Syfte
Syftet med uppgiften var att bygga en trafikbaslinje för en server i labbmiljön,
alltså en bild av vilken trafik som är normal. Samtidigt skulle vi hitta det enda
flöde som inte hör hemma i baslinjen och förklara varför det avviker.

## 2. Utgångspunkt
Vi utgick från uppgift U1.4 som vi jobbade med förra veckan för att fånga trafiken, eftersom vi redan gjort det. Vi hämtade alltså en färdig fångst med kommandot `hämta-fil U1.4`. Vi använde alltså inte tcpdump själva, utan arbetade med att analysera en befintlig pcap-fil.

## 3. Om fångsten
| Uppgift | Värde |
| Server som trafiken kommer från | elev-45 |
| Privat IP-adress | 172.20.1.145 |
| Längd | ca 12 minuter |
| Filnamn | U1.4 |
| Analys gjord av | Alex Vesterlind |
| Datum för analysen | 2026-10-12 |


Analysen gjordes i förra veckans uppgift U1.4. Vi hämtade fångstfilen med
`hämta-fil U1.4` (filen sparades som `~/ctf/U1.4-avvikelsen.pcap`, ca 12
minuters trafik från servern 172.20.1.145) och använde den som underlag för
baslinjen i denna uppgift.

Vi arbetade så här:

1. **Överblick:** Vi gick igenom fångsten och sammanställde vilka flöden som
   förekommer (källa, mål, protokoll, port). Verktyg: FYLL I
2. **Baslinje:** Vi bedömde syftet med varje flöde och hur ofta det
   förekom. Resultatet finns i tabellen i `baslinje-trafik.md`.
3. **Avvikelsen:** Vi identifierade det flöde som inte hörde till baslinjen:
   FYLL I (vilket flöde och varför).
4. **Avkodning:** FYLL I (hur flaggan hittades och avkodades, eller skriv
   "se U1.4-inlämningen").
## 4. Så gick analysen till

Analysen gjordes i uppgift U1.4. Vi hämtade fångstfilen med `hämta-fil U1.4`
och öppnade den i Wireshark.

### Steg 1: Överblick
Vi gick igenom fångsten för att se vilka flöden som förekommer (källa, mål,
protokoll och port).

### Steg 2: Baslinje
Vi identifierade de flöden som ligger på vanliga, välkända portar:
SSH (TCP 22), DNS (UDP/TCP 53), NTP (UDP 123) och HTTP (TCP 80). Det är
trafik en server normalt har, så vi bedömde dem som normala och lade dem i
baslinjen (se `baslinje-trafik.md`).

### Steg 3: Hitta avvikelsen
Vi filtrerade bort all trafik på de vanliga portarna med följande
visningsfilter i Wireshark:
!(tcp.port == 80) && !(tcp.port == 22) && !(udp.port == 53) && !(tcp.port == 53) && !(udp.port == 123)
Det som blev kvar var trafik på ovanliga portar. Vi granskade de flöden som såg
annorlunda ut och hittade det som inte passade in i baslinjen.

### Steg 4: Avkodning
Flaggan i det avvikande flödet var kodad med base64. Vi avkodade den och fick
fram flaggan.

## 5. Begränsningar
- Fångsten omfattar bara ca 12 minuter, så trafik som sker mer sällan syns inte.
- Vi ser bara trafik från och till denna server, inte övriga nätet.
- Vi har inte bekräftat vem som äger IP-adressen 46.224.108.117 eller
  adresserna 172.20.1.2, 185.12.64.1 och 185.12.64.2. Det vi skriver om dem är
  därför bedömningar och inte bekräftade fakta.
