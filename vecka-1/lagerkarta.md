# Lagerkarta – Zonuppdraget

Lagerkartan beskriver trafiken lager för lager enligt TCP/IP-modellens fyra
lager. Underlaget är fångsten U1.4 från elev-45 (172.20.1.145), ca 12 minuter
trafik, analyserad i Wireshark. Flödena är sammanställda i `baslinje-trafik.md`.

## 1. Applikationslagret
Uppgift: Lagret beskriver vad kommunikationen används till.

- SSH: säker fjärrinloggning och administration av en server.
- DNS: översätter domännamn till IP-adresser.
- NTP: synkroniserar tiden.
- HTTP: kommunikation mellan webbläsare och webbserver.

Belägg: I fångsten fanns fyra typer av applikationstrafik från elev-45:
SSH mot 172.20.1.2 (ca 24 paket), DNS mot 185.12.64.1 (ca 10 paket), NTP mot
185.12.64.2 (ca 20 paket) och HTTP mot 46.224.108.117 (ca 60 paket).

## 2. Transportlagret
Uppgift: Lagret transporterar data mellan två värdar. Portnumren visar
vilken tjänst trafiken hör till.

- TCP: används för SSH (port 22) och HTTP (port 80).
- UDP: används för DNS (port 53) och NTP (port 123).

Belägg: Flödena gick på TCP 22, TCP 80, UDP 53 och UDP 123. Vi filtrerade
bort dessa portar i Wireshark med
`!(tcp.port == 80) && !(tcp.port == 22) && !(udp.port == 53) && !(tcp.port == 53) && !(udp.port == 123)`
för att hitta trafik som inte hörde till baslinjen. Portar som blev kvar:
FYLL I.

## 3. Internetlagret
Uppgift: Lagret adresserar källa och mål med IP-adresser så att paketen hittar
rätt mellan nät.

- IPv4: adresserar avsändare och mottagare.

- Belägg: 172.20.1.145 (elev-45) var källa i alla flöden i baslinjen. Målen var
172.20.1.2 (jump-servern enligt vår tillgångsförteckning), 185.12.64.1 (DNS),
185.12.64.2 (NTP) och 46.224.108.117 (HTTP). Vi har inte bekräftat vem som äger
de tre externa adresserna.

## 4. Nätverksåtkomstlagret
Uppgift: Lagret flyttar data över den lokala länken i Ethernet-ramar med
MAC-adresser. ARP kopplar en IP-adress till en MAC-adress på det lokala nätet.

- Ethernet: transporterar data i ramar mellan enheter på samma nät.
- ARP: hittar MAC-adressen som hör till en IPv4-adress.

-Belägg: Ett paket från 172.20.1.145 har käll-MAC FYLL I och mål-MAC FYLL I
(raden *Ethernet II* i Wireshark).

## Exempel: SSH genom de fyra lagren
Flödet är elev-45 (172.20.1.145) mot jump-servern (172.20.1.2), ca 24 paket.

| Lager | Koppling till trafiken |
| Applikation | SSH används för fjärrinloggning och administration |
| Transport | TCP, destinationsport 22 |
| Internet | IPv4 med källa 172.20.1.145 och mål 172.20.1.2 |
| Nätverksåtkomst | Ethernet-ram med käll-MAC FYLL I och mål-MAC FYLL I |

Exemplet visar att lagren har olika uppgifter men samverkar när data skickas.

## Källa och begränsningar
- Underlag: `baslinje-trafik.md` och fångsten U1.4.
- Fångsten är bara ca 12 minuter, så trafik som sker mer sällan syns inte.
- Vi har inte kontrollerat vem som startade SSH-anslutningen.
- Vi har inte bekräftat ägaren till 46.224.108.117, 185.12.64.1 och 185.12.64.2.

## Sammanfattning
Vi har beskrivit trafiken i fångsten lager för lager och kopplat varje protokoll
till rätt TCP/IP-lager. Där vi inte har kunnat fastställa något har vi bara
skrivit det vi faktiskt observerade.
