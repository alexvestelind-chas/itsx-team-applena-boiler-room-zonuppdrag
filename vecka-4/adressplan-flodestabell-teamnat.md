# Adressplan och flödestabell – teamnätet (10.40.8.0/24)

**Förutsättningar:** Privat Hetzner-nät, gemensam gateway 10.40.8.1 för hela nätet (fysiskt placerad i det första subnätet). Varje subnäts egen nät- och broadcastadress är reserverad och kan inte delas ut.

## Adressplan

| Prefix | Zon | Ansvarig | Adressintervall (användbart) | Kommentar |
|---|---|---|---|---|
| 10.40.8.0/27 | Klientzon | Mouhamad | 10.40.8.2 – 10.40.8.30 | .0 nätadress, .1 nätets gemensamma gateway, .31 broadcast |
| 10.40.8.32/27 | DMZ | Alex O | 10.40.8.33 – 10.40.8.62 | .32 nätadress, .63 broadcast |
| 10.40.8.64/28 | Applikationszon | Alexander | 10.40.8.65 – 10.40.8.78 | .64 nätadress, .79 broadcast |
| 10.40.8.80/28 | Datazon | Alex V | 10.40.8.81 – 10.40.8.94 | .80 nätadress, .95 broadcast |

## Flödestabell

| Rad | Källzon | Målzon | Protokoll | Port | Riktning | Motivering (verksamhetsfunktion) | Filtreras var |
|---|---|---|---|---|---|---|---|
| F1 | Klientzon (10.40.8.0/27) | DMZ (10.40.8.32/27) | tcp | 443 | Inkommande (till DMZ) | Klienter når webbportalen | Värd (eth2) |
| F2 | DMZ (10.40.8.32/27) | Applikationszon (10.40.8.64/28) | tcp | 8080 | Inkommande (till App) | Portalen hämtar data från applikationslogiken | Värd (eth2) |
| F3 | Applikationszon (10.40.8.64/28) | Datazon (10.40.8.80/28) | tcp | 5432 | Inkommande (till Data) | Appen läser och skriver data | Värd (eth2) |

### Driftflöden (D1–D7) – gäller varje server oavsett zon

| Rad | Källzon | Målzon | Protokoll | Port | Riktning | Motivering | Filtreras var |
|---|---|---|---|---|---|---|---|
| D1 | Jump-server (172.20.1.2) | Egen server (alla zoner) | tcp | 22 | Inkommande | Administration via jump | Värd (eth1) |
| D2 | CTF-server (172.20.1.10) | Egen server (alla zoner) | tcp | 22 | Inkommande | Kursens kontrolltjänst | Värd (eth1) |
| D3 | Egen server (alla zoner) | Hetzners DNS-resolvrar (185.12.64.1, 185.12.64.2) | udp/tcp | 53 | Utgående | Namnuppslag – molnbrandväggen släpper alltid dessa, men krävs även på värden | Värd (eth0) + molnbrandvägg |
| D4 | Egen server (alla zoner) | Internet | udp | 123 | Utgående | Tidssynk (chrony) | Värd (eth0) + molnbrandvägg |
| D5 | Egen server (alla zoner) | Loggserver (172.20.1.58) | tcp | 6514 | Utgående | Loggar till loggservern över TLS (RFC 5425, MCFFS 2026:11 4 kap. 25 §). Ersätter klartext på 514 från vecka 4, som stängs. | Värd (eth1) |
| D6 | Egen server (alla zoner) | DHCP-servern (172.31.1.1 på eth0, 172.20.0.1 på eth1, 10.40.8.1 på eth2) | udp | 68→67 | Utgående | Förnyelse av serverns adresser – utan regeln tappas de när lånet (24h) går ut | Värd (samtliga gränssnitt) |

**Allt som inte står ovan är nekat, inkommande och utgående.**

### Noteringar

- D1–D7 är identiska för alla fyra servrar oavsett zon – skriv dem i varje server (motsvarande `zon.nft`), inte bara en gång för teamet.
- Kedjan går i en riktning (Klient → DMZ → App → Data) – återtrafiken (svaren) hanteras automatiskt av `ct state established,related accept`, ingen egen rad behövs för den.
- Eftersom molnbrandväggen bara ser `eth0`, filtreras samtliga tre rader uteslutande på respektive värd (`zon.nft`, bundet till `eth2`).
- Regeln för varje rad skrivs på **mottagande** zons server (t.ex. F1 skrivs i Alex O:s `zon.nft`, inte i Mouhamads), i linje med hur `nftables`-filtrering fungerar per värd.
