# Playbook: T1110 SSH brute force – elev-39 (applikationszon)

**Författare:** Alexander Schön (elev-39, applikationszonen)
**Status:** utkast, ännu inte korstestad av någon annan
**Version:** 17 november 2026
**MITRE:** T1110 Brute Force (Credential Access)

Den här playbooken ska kunna följas av någon som inte var med när larmet kom. Den bygger på NIST SP 800-61 Rev. 3 (CSF 2.0) med fyra faser: Förberedelse, Upptäckt och analys, Begränsning/undanröjande/återställning, Efterarbete.

---

## Detektion

| | |
|---|---|
| Monitor | "T1110 SSH brute force elev-39" (Wazuh Alerting) |
| Index | `wazuh-alerts-*` |
| Körning | varje minut, tittar 10 minuter bakåt (`{{period_end}}\|\|-10m`) |
| Villkor | `rule.id` 5710 eller 5712, och `location` `/var/log/remote/elev-39/syslog.log` |
| Gruppering | per `data.srcip`, minst 5 träffar (`min_doc_count` 5) |
| Larm | om minst en källadress når 5 träffar, severity 1 |

Poster utan `data.srcip` kan inte ge larm.

Regler: 5710 = inloggningsförsök med användare som inte finns (nivå 5). 5712 = 8 misslyckade försök på 2 minuter (nivå 10). 5715 = lyckad inloggning (nivå 3). 40112 = flera misslyckade följt av lyckad (nivå 12). 5760 = fel lösenord på ett befintligt konto.

## Brus (mätt mot normaltrafik)

- Mätfönster: 11 november 2026 08:06–08:37 (svensk tid), ca **0,52 timmar**.
- Regel 5710 gav **240 träffar** (godkänt av `kontrollera U6.3`) och regel 5712 gav 5, totalt 245.
- Träffarna kommer från **25 olika källadresser** (10.40.x.x). Varje adress gör en skur på **8–12 försök under 0,1 sekund**, med användarnamn ur en lista på 12 vanliga namn (admin, ubuntu, pi, oracle, postgres, ftp, git, deploy, guest, support, test, user), utan siffror.
- Monitorn larmar när en källadress når minst 5 träffar på 10 minuter, så varje skur ger ett larm: ca **25 larm på 0,52 timmar, ungefär 48 per timme**. Enbart den egna normaltrafiken (`inmatat/elev-39`) ger 4 larm på samma tid, ca 8 per timme.
- Normaltrafiken ligger under andra `location` än levande loggar: `/var/log/remote/normaltrafik/syslog.log` och `/var/log/remote/inmatat/elev-NN/normal.log`. Monitorns platsfilter matchar dem inte, så brus mäts **utan** platsfilter eller med normaltrafikens egen `location`.

## Falsklarmsrisk

**Hög.** Antalet försök kan inte skilja normaltrafiken från ett riktigt angrepp. De 12 verifieringsförsöken (från 172.20.1.10, 12 november, ca 7 sekunder) ryms inom normalskurarnas 8–12 träffar, så en högre tröskel hjälper inte. En analytiker skulle få ca 48 larm i timmen där inget är ett verkligt angrepp.

Tre skillnader syns i datan men är inte verifierade som detektionsgrund:
- källadress (172.20.1.10 mot 10.40.x.x)
- hastighet (ca 7 sekunder mot under 0,1 sekund)
- användarnamn (ord plus siffror, t.ex. `test1`, `support63`, mot 12 vanliga namn utan siffror)

De kan vara artefakter av hur övningen genererar data.

## Föreslagen förbättring (inte verifierad)

Komplettera med en korrelationsdetektion på regel **40112** (flera misslyckade följt av lyckad inloggning), eftersom en lyckad inloggning efter brute force är det som kräver åtgärd.

- I normaltrafiken gav 40112 **55 träffar på 0,52 timmar** (ca 106 per timme). Alla 55 hade `data.srcip` 172.20.1.2 (jump-servern), och den träff som öppnades var `Accepted publickey for student from 172.20.1.2`.
- Ett undantag för 172.20.1.2 ger därför 0 träffar i normaltrafiken. Det följer av träfflistan. Sökningen `rule.id:40112 and not data.srcip:"172.20.1.2"` är inte körd.
- **Begränsningar:** ctf-övningen ger bara misslyckade försök, så regeln är inte testad mot ett riktigt intrång, och noll träffar bevisar inte att den fångar något. En angripare som kommer via jump-servern syns inte med ett IP-undantag. Alternativ utan IP-undantag: kräv `Accepted password` i `full_log`.
- Att undanta hela 10.40.x.x är förkastat: ett komprometterat internt system skulle bli osynligt.

---

## 1. Förberedelse (innan något larm)

- Monitorn "T1110 SSH brute force elev-39" är aktiv (severity 1).
- Jag kan logga in i Wazuh och söka i `wazuh-alerts-*`.
- Adressplan och tillgångsförteckning finns att slå upp adresser i (teamets repo: `vecka-2/adressplan.md`, `vecka-4/adressplan-flodestabell-teamnat.md`, `vecka-1/tillgångsförteckning.md`).
- Känd betrodd källa: jump-servern 172.20.1.2.
- Blockering av en adress i nftables och låsning av ett konto på elev-39: **ej övat än**. Öva innan playbooken räknas som klar. Jump-servern (172.20.1.2) får aldrig blockeras.

## 2. Upptäckt och analys

1. Öppna larmet i Alerting → Alerts och notera klockslaget.
2. I Discover: klicka **New**, kontrollera att inga filterpiller syns under sökfältet och att tidsfönstret är rätt (svensk tid, med både början och slut).
3. Sök `rule.id:(5710 or 5712) and data.srcip:"<IP>"`. Notera antal, tidsintervall, `rule.id` och `data.srcuser`.
4. Slå upp adressen i adressplanen. En okänd adress behandlas som misstänkt tills någon kan säga motsatsen.
5. Sök lyckad inloggning från samma adress: `rule.id:(5715 or 40112) and data.srcip:"<IP>"`, i larmfönstret och senare. Kontrollera även om försöken träffade befintliga konton (5760). Noll träffar är bara en observation: kontrollera att sökningen kan hitta lyckade inloggningar, till exempel `rule.id:5715 and location:"/var/log/remote/elev-39/syslog.log"` med Last 7 days.
6. Skriv en post i triageloggen, med observation och slutsats i skilda fält (se bilagan).
7. Beslut:
   - Ingen lyckad inloggning: fortsätt övervaka och överväg blockering. Gå till steg 4.
   - Lyckad inloggning: gå till steg 3.

## 3. Begränsning, undanröjande och återställning (om någon har kommit in)

1. Ta reda på hur: vilken metod (lösenord eller nyckel), vilket konto och vilken tid, via `full_log` på träffen.
2. Begränsa:
   - Blockera adressen inkommande med nftables på elev-39.
   - Avsluta pågående sessioner från adressen (en blockering stoppar bara nya anslutningar).
   - Lås kontot och kontrollera att det inte längre kan logga in (även med nyckel).
3. Undersök vad kontot har rört: kommandon, ändrade filer, sudo (regel 5402) och nya konton (5902). Kontrollera `authorized_keys`, cron och nya användare.
4. Återställ: byt lösenord och nycklar, ta bort ändringar, öppna kontot först efter kontroll. Verifiera att tjänsten fungerar och att monitorn fortfarande larmar.
5. Informera teamet.

## 4. Efterarbete

- Dokumentera i triageloggen vad som hände och vad som gjordes.
- Justera tröskel eller undantag och mät bruset igen.
- Uppdatera playbooken med det som hände.
- Fällor att ta med: nollställ Discover med **New** före varje sökning, kontrollera tidsfönstret, och ta bort ett villkor i taget när en sökning ger noll träffar.

---

## Bilaga: triageloggpost (verifieringen 12 november 2026)

| Fält | Innehåll |
|---|---|
| **Larm** | Monitorn "T1110 SSH brute force elev-39", severity 1. Larmtid: FYLL I från Alerting → Alerts. |
| **Observation** | Discover, `wazuh-alerts-*`, 12 nov 2026 09:25–09:35 (svensk tid), fråga `rule.id:(5710 or 5712)`: 12 träffar, alla med `data.srcip` 172.20.1.10 och `location` `/var/log/remote/elev-39/syslog.log`. Första träffen 09:29:54.982, sista 09:30:01.762 (ca 7 sekunder). 11 träffar på regel 5710 och 1 på regel 5712 (09:29:59.726, det åttonde försöket). `data.srcuser`: 12 olika namn, alla ett ord plus siffror (t.ex. `test1`, `support63`, `admin24`, `oracle67`). Sökningen `rule.id:(5715 or 40112) and data.srcip:"172.20.1.10"` i samma fönster gav 0 träffar. Kontroll: `rule.id:5715 and location:"/var/log/remote/elev-39/syslog.log"` med Last 7 days gav 8 träffar, bl.a. från jump (172.20.1.2, användaren `student`). |
| **Slutsats** | Ett automatiserat försök att gissa användarkonton över SSH mot elev-39 (T1110) från 172.20.1.10. Alla 12 försök gäller användarnamn som inte finns (regel 5710), och ingen lyckad inloggning från adressen syns i loggen. Inget tyder på att någon har kommit in. |
| **Berört** | Server elev-39 (applikationszonen), tjänsten sshd. Inget konto bedöms vara komprometterat. |
| **Åtgärd** | Testkörning från ctf-servern, ingen åtgärd. I ett skarpt läge: kontrollera om adressen är känd (adressplanen), kontrollera lyckade inloggningar från den och blockera den vid behov (se steg 2 och 3). |

---

## Metod

Mätningarna och triagen är gjorda av mig i Wazuh Discover. Claude hjälpte mig tolka resultaten, föreslog formuleringar och skrev utkasten till texterna, som jag har granskat.

## Kvar innan playbooken är klar

- Larmtid i triageloggposten.
- Öva blockering i nftables och kontolåsning på elev-39 (se steg 1 i Förberedelse).
- Korstest: någon annan följer playbooken på ett nytt larm.
- Köra `rule.id:40112 and not data.srcip:"172.20.1.2"` och bekräfta 0 träffar.
