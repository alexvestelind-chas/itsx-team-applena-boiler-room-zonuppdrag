# Playbook: Beaconing (T1071.001) – elev-39 (applikationszonen)

**Författare:** Alexander Schön (elev-39, applikationszonen)
**Status:** utkast, ännu inte korstestad av någon annan
**Version:** 18 november 2026
**MITRE:** T1071.001 Web Protocols (under T1071 Application Layer Protocol, taktik Command and Control). T1029 Scheduled Transfer (taktik Exfiltration) är en möjlig andra läsning, eftersom loggraderna anger `action=upload`.

Den här playbooken ska kunna följas av någon som inte var med när larmet kom. Den bygger på NIST SP 800-61 Rev. 3 (CSF 2.0) med fyra faser: Förberedelse, Upptäckt och analys, Begränsning/undanröjande/återställning, Efterarbete.

---

## Detektion

| | |
|---|---|
| Monitor | "T1071.001 Beaconing upload elev-39" (Wazuh Alerting), trigger "Beaconing upload elev-39", severity 1 |
| Index | `wazuh-archives-*` (inte `wazuh-alerts-*`: beaconing matchar ingen Wazuh-regel, så raderna finns bara i archives) |
| Körning | varje minut, tittar 10 minuter bakåt (`{{period_end}}\|\|-10m`) |
| Villkor | `full_log` innehåller frasen `action=upload` och `location` är `/var/log/remote/elev-39/syslog.log` |
| Gruppering | per `predecoder.program_name`, minst 10 träffar (`min_doc_count` 10) |
| Larm | om något program når minst 10 träffar på 10 minuter: `ctx.results[0].aggregations.per_program.buckets.length > 0` |

Larmet kommer ungefär 100 sekunder in i ett beacon med 10 sekunders intervall, eftersom 10 rader då ligger i fönstret.

## Brus (mätt mot normaltrafik)

- Mätfönster: 11 november 2026 08:06–08:36 (svensk tid), ca **0,5 timmar**, i `wazuh-archives-*` med `location` `/var/log/remote/normaltrafik/syslog.log`.
- Normaltrafiken innehåller **384 rader**: `sshd` 253, `CROND` 86, `sudo` 43, `normaltrafik` 2. Frågan `action=upload` ger **0 träffar**.
- Kontroll att noll är meningsfullt: platsfiltret ensamt ger 384 träffar, och `action=upload` ensamt ger träffar på min egen server. Båda delarna hittar alltså något var för sig.
- Normaltrafiken har **ingen utgående nätverkstrafik alls** (inga URL:er, ingen proxy eller brandvägg). Bruset för den här detektionen är därför 0 i den här datan, men det betyder inte att det är 0 i en riktig miljö.
- Det närmaste regelbundna i normaltrafiken är `CROND`. Mellanrummen mellan skurar är 16–78 sekunder (median ca 22 s) enligt mina beräkningar med 1 sekund som skurgräns, alltså inte jämna som ett beacon.
- Min egen server, 14 dagar: **19 träffar**, varav 18 från `syncagent` och 1 från `systemd` (raden som startade körningen och som innehåller kommandotexten). Gruppering per program med minst 10 träffar tar bort `systemd`-raden.

## Falsklarmsrisk

**Låg i den uppmätta datan, men oprövad mot riktig legitim trafik.** Normaltrafiken saknar sådant som riktiga miljöer har och som också ringer ut med jämna mellanrum: säkerhetskopior, uppdateringar och övervakningsagenter. De kan ge falsklarm, och risken är inte mätt.

Begränsningar:
- **Smal på innehåll:** villkoret `action=upload` bygger på just det här loggformatet. Skadlig kod väljer själv programnamn, URL och loggformat.
- **Missar långsamma beacons:** ett beacon som ringer var femte minut ger 2 träffar på 10 minuter och larmar inte. Ett längre fönster fångar fler men kan inte provas i övningen, som bara varar tre minuter.
- **Loggraden är programmets eget påstående.** I övningen skapas raderna av `logger` från ett systemd-startat skal-skript (se bilagan), och `status=200` och `user=svc-backup` är bara text. Inget lämnar servern. I en riktig utredning måste något annat än programmets egen rad styrka att trafik gått ut (se steg 2.5).

## Föreslagen förbättring (inte byggd)

1. **Mät regelbundenheten.** Beaconing känns igen på jämna mellanrum, inte på ett visst programnamn eller en viss URL. Det går i princip att göra med ett Painless-skript i triggervillkoret som räknar mellanrummen i händelsetiden (`predecoder.timestamp`, som är text och måste tolkas, inte Wazuhs `timestamp` som är inläsningstid med 2–3 sekunders skakning). Det kräver att man kan gruppera per konto och mål, och att tröskeln för tillåten variation (jitter) ställs in mot data med både beacons och legitim regelbunden trafik. Det har jag inte här.
2. **Nätverkstelemetri i stället för programmets egen logg.** Bygg detektionen på destination och antal anslutningar från brandvägg, proxy eller flödesdata.
3. **Tillåtlista** över kända program och konton som får ringa ut regelbundet, så att legitim trafik inte larmar.

---

## 1. Förberedelse (innan något larm)

- Monitorn "T1071.001 Beaconing upload elev-39" är aktiv (Enabled). Den skapades som en ny monitor, aldrig genom att redigera en annan.
- Jag kan logga in i Wazuh och söka i **`wazuh-archives-*`**. Indexmönstret finns under Dashboards Management → Index patterns (tidsfält `timestamp`) om det saknas.
- Vilka utgående flöden som är förväntade: enligt servens nftables (kedjan `ut`, `policy drop`) är bara DNS (D3), NTP (D4), loggar till loggservern över TLS 6514 (U5), DHCP (D6) och app till databas port 5432 (F4) tillåtna vid skrivande stund. Teamets flödestabell är den som gäller. Allt annat utgående till internet är avvikande.
- Brandväggslogg: stoppade utgående paket loggas med prefixet `ZON-UT-DROP` (`sudo journalctl -k | grep ZON-UT-DROP`).
- Stoppa en process eller tjänst och låsa ett konto på elev-39 (`systemctl stop`, `usermod -L`): **ej övat**. Vid kontroll den 18 nov var körningen redan klar (ingen enhet kvar att stoppa) och kontot `svc-backup` finns inte på servern (`getent passwd svc-backup` gav ingen träff), så det fanns inget att stoppa eller låsa i labbet. Stegen i avsnitt 3 är därför beskrivna men inte provade. Jump-servern (172.20.1.2) och loggservern får aldrig blockeras.

## 2. Upptäckt och analys

**Tid:** servern (`journalctl`, `date`) går på UTC, och svensk tid är UTC+1 på vintern. Jämför aldrig klockslag utan att skriva vilken tidszon de gäller.

1. Öppna larmet i Alerting → Alerts. Notera trigger, monitor och **klockslaget** (svensk tid).
2. I Discover: välj `wazuh-archives-*`, klicka **New**, kontrollera att inga filterpiller finns under sökfältet och att tidsfönstret täcker larmet (svensk tid, med både början och slut). Raderna i archives har ingen `rule.id`.
3. Sök `location:"/var/log/remote/elev-39/syslog.log" and full_log:"action=upload"`. Klicka på `predecoder.program_name` för att se vilket program som står för träffarna. Lägg till kolumnerna `timestamp` och `full_log`.
4. Läs raderna: vilket konto (`user=`), vilket mål (`url=`), antal, och **mellanrum mellan händelserna**. Mät mellanrummet med tiden i raden (`predecoder.timestamp`), inte kolumnen `Time` i Wazuh. Notera om storleken (`bytes`) varierar och räkna ihop summan.
5. **Kontrollera med något annat än programmets egen rad.** Sök `ZON-UT-DROP` i brandväggsloggen för samma tidsrymd. Finns destinationen i flödestabellen? En lyckad utgående anslutning till en adress som inte är tillåten är i sig ett larmtecken, och en stoppad är ett försök som brandväggen hindrade.
6. Ta reda på vad som startade det: sök `systemd` och `CROND` kring den första raden. I övningen finns en `systemd`-rad ("Started /bin/bash -c ...") ca 25 millisekunder före första anropet. Notera enhetens namn och kommandot.
7. Slå upp mål och domän. Okänt mål behandlas som misstänkt tills någon kan säga motsatsen.
8. Skriv en post i triageloggen med observation och slutsats i skilda fält (se bilagan).
9. Beslut:
   - Känt och godkänt: dokumentera, lägg till i tillåtlistan och justera detektionen. Gå till steg 4.
   - Okänt eller oklart: gå till steg 3.

## 3. Begränsning, undanröjande och återställning

1. **Säkra bevis först** (innan något stoppas): spara de relevanta loggraderna, `ps`-utskrift och enhetens definition.
2. Begränsa:
   - Stoppa programmet eller enheten (`systemctl stop <enhet>`) och avaktivera den om den är permanent.
   - Blockera målet utgående. Kontrollera först att ingen `accept`-regel i `ut`-kedjan redan släpper ut det. Öppna aldrig en utgående regel för att få något att "lyckas".
   - Lås kontot som programmet kör som (`sudo usermod -L <konto>`) och kontrollera att det inte längre kan logga in.
3. Undersök omfattning: vilka filer kontot kunde läsa, hur mycket som skickades (summan av `bytes` är en övre gräns i loggen men inte bevis), och persistens: cron, systemd-enheter, `authorized_keys`, nya användare.
4. Återställ: ta bort persistensen, byt lösenord och nycklar för kontot, öppna det först efter kontroll. Verifiera att trafiken har upphört och att monitorn fortfarande larmar vid ett nytt test.
5. Informera teamet.

## 4. Efterarbete

- Dokumentera i triageloggen vad som hände och vad som gjordes.
- Justera tröskel, fönster och tillåtlista, och mät bruset igen.
- Genomför förbättringarna ovan (regelbundenhet, nätverkstelemetri).
- Uppdatera playbooken med det som hände.
- Fällor att ta med:
  - Rätt index: archives för det som inte matchar en regel, alerts bara för regelträffar. Noll träffar i fel index betyder ingenting.
  - Visualize har ett eget tidsfönster och ger lätt tomma resultat.
  - Nollställ Discover med **New**, kontrollera tidsfönstret och ta bort ett villkor i taget när en sökning ger noll.
  - Redigera aldrig en befintlig monitor när du ska bygga en ny: skapa en ny med **Create monitor** och kontrollera sidans rubrik och monitorns namn före **Save**.
  - Ett larm som redan gått när övningen startades räknas inte: vänta tills det är Completed och starta om.

---

## Bilaga: triageloggpost (testkörning U6.5, 18 november 2026)

| Fält | Innehåll |
|---|---|
| **Larm** | Monitorn "T1071.001 Beaconing upload elev-39", trigger "Beaconing upload elev-39", severity 1. Larmtid i Alerting → Alerts: **18 nov 2026 09:18** (svensk tid, ur larmlistan). |
| **Observation** | Discover, `wazuh-archives-*`, fråga `location:"/var/log/remote/elev-39/syslog.log" and full_log:"action=upload"`, Last 14 days: 19 träffar. 18 med `predecoder.program_name` `syncagent` och 1 med `systemd`. De 18 raderna (första körningen, 18 nov 08:29:54–08:32:44 svensk tid) har samma text `user=svc-backup action=upload url=https://sync.telemetri-app.net/v1/beacon status=200`. Mellanrum enligt tiden i raden: 10,005–10,012 sekunder (17 mellanrum). `bytes` varierar mellan 2 252 och 96 557, summa 889 524. `systemd`-raden (08:29:54) lyder "Started /bin/bash -c for i in $(seq 1 18); do logger -t syncagent ... sleep 10; done", och en rad kl. 08:32:56 anger att enheten `ctf-u65-…service` avslutats. Normaltrafik (11 nov 08:06–08:36): 0 träffar på `action=upload` av 384 rader. **Andra körningen** (den som utlöste larmet): serverns journal (`sudo journalctl -t syncagent --since today`) visar 36 rader med `action=upload` den 18 nov, alltså två körningar à 18 rader. Den andra gick **09:16:25–09:19:15 svensk tid** (08:16:25–08:19:15 UTC), 170 sekunder mellan första och sista raden, vilket ger 10 sekunder i snitt. Den tionde raden kom ca 09:17:55 (beräknat), och larmet kom 09:18. Den första körningen gick innan monitorn fanns och gav därför inget larm. **Brandväggsloggen:** `sudo journalctl -k \| grep ZON-UT-DROP \| tail` visar bara tio SYN-försök mot port 443 från 05:46–05:47 UTC (dnf-försök, före första beacon-raden), inga under någon av beacon-körningarna. **Ej kontrollerat:** DNS, och mellanrummen i andra körningen rad för rad. |
| **Slutsats** | Regelbundna, uppladdningsliknande anrop var 10:e sekund under ca tre minuter från programmet `syncagent` under kontot `svc-backup` mot en extern URL, startade av ett systemd-kommando. Mönstret stämmer med beaconing (T1071.001). Raderna är programmets egna uppgifter. Brandväggsloggen visar ingen utgående trafik under körningarna, vilket stämmer med att raderna skapas av `logger` och att ingen trafik har lämnat servern. I en riktig miljö skulle det här sakna stöd i nätverksdatan och måste utredas vidare. |
| **Berört** | Server elev-39 (applikationszonen). Inget konto bedöms vara komprometterat. Mängden som skickades enligt loggen: 889 524 byte (ej verifierat). |
| **Åtgärd** | Testkörning från ctf-servern (U6.5), ingen åtgärd. I ett skarpt läge: säkra loggar, kontrollera mot brandväggsloggen, stoppa enheten och lås kontot (se steg 2 och 3). |

---

## Metod

Mätningarna och triagen är gjorda av mig i Wazuh Discover. Claude hjälpte mig tolka resultaten, kontrollerade ATT&CK-id mot MITRE:s webbplats, föreslog formuleringar och JSON-frågan till monitorn och skrev utkastet till den här texten, som jag har granskat. Kontrollerna på servern (brandväggsloggen, antal rader, tider, om kontot finns) körde jag själv, efter förslag från Claude. Jag byggde och testade monitorn själv, kontrollerade att den inte hittade något i normaltrafiken och körde `kontrollera U6.5`, som godkände den.
