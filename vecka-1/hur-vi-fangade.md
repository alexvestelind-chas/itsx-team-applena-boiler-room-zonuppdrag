# Hur vi tog fram och analyserade trafiken

## 1. Syfte
Syftet med uppgiften var att bygga en trafikbaslinje för en server i
labbmiljön, alltså en bild av vilken trafik som är normal. Samtidigt skulle vi
hitta det enda flöde som inte hör hemma i baslinjen och förklara varför det
avviker och vad det skulle kunna vara.

## 2. Utgångspunkt
Vi utgick från uppgift U1.4 (workshop 2) som vi arbetade med förra veckan.
Trafiken fångades inte av oss. Vi hämtade en färdig fångst med kommandot
`hämta-fil U1.4` och analyserade den. Vi använde alltså inte tcpdump, utan
arbetade med en befintlig pcap-fil.

## 3. Om fångsten

| Uppgift | Värde |
|---|---|
| Server som trafiken kommer från | elev-45 |
| Privat IP-adress | 172.20.1.145 |
| Längd | ca 12 minuter |
| Filnamn | `U1.4-avvikelsen.pcap` (22 486 byte) |
| Placering | `~/ctf/` på jump-servern |
| Verktyg för analys | Wireshark |
| Analys gjord av | Alex Vesterlind |
| Datum för analysen | FYLL I |

## 4. Så gick analysen till

### Steg 1: Överblick
Vi öppnade fångsten i Wireshark och gick igenom vilka flöden som förekommer
(källa, mål, protokoll och port).

### Steg 2: Baslinje
Vi identifierade de flöden som ligger på vanliga, välkända portar: SSH
(TCP 22), DNS (UDP/TCP 53), NTP (UDP 123) och HTTP (TCP 80). Det är trafik en
server normalt har, så vi bedömde dem som normala och lade dem i baslinjen
(se `baslinje-trafik.md`).

### Steg 3: Hitta avvikelsen
Vi filtrerade bort all trafik på de vanliga portarna med följande
visningsfilter i Wireshark:

```
!(tcp.port == 80) && !(tcp.port == 22) && !(udp.port == 53) && !(tcp.port == 53) && !(udp.port == 123)
```

Det som blev kvar var trafik på ovanliga portar. Vi granskade de flöden som såg
annorlunda ut och hittade det som inte passade in i baslinjen:

- **Avvikande flöde:** FYLL I (källa → mål, protokoll, port)

### Steg 4: Avkodning
Flaggan i det avvikande flödet var kodad med base64. Vi avkodade den och fick
fram flaggan, som vi lämnade in med `lämna-flagga U1.4`.

## 5. Begränsningar
- Fångsten omfattar bara ca 12 minuter, så trafik som sker mer sällan syns inte.
- Vi ser bara trafik från och till denna server, inte övriga nätet.
- Vi har inte bekräftat vem som äger IP-adressen 46.224.108.117 eller
  adresserna 185.12.64.1 och 185.12.64.2. Det vi skriver om dem är därför
  bedömningar och inte bekräftade fakta.
- Adressen 172.20.1.2 är jump-servern enligt vår tillgångsförteckning, men vi
  har inte kontrollerat vem som startade SSH-anslutningen.
