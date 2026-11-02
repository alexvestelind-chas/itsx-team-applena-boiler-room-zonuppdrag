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

**Allt som inte står ovan är nekat, inkommande och utgående.**

### Noteringar

- Rad-id:n (F1–F3) är fristående från era driftflöden (D1–D7), som varje server dessutom behöver oavsett zon (SSH från jump/ctf, DNS, NTP, logg, DHCP-förnyelse – inklusive DHCP-förnyelse för eth2 mot 10.40.8.1).
- Kedjan går i en riktning (Klient → DMZ → App → Data) – återtrafiken (svaren) hanteras automatiskt av `ct state established,related accept`, ingen egen rad behövs för den.
- Eftersom molnbrandväggen bara ser `eth0`, filtreras samtliga tre rader uteslutande på respektive värd (`zon.nft`, bundet till `eth2`).
- Regeln för varje rad skrivs på **mottagande** zons server (t.ex. F1 skrivs i Alex O:s `zon.nft`, inte i Mouhamads), i linje med hur `nftables`-filtrering fungerar per värd.
