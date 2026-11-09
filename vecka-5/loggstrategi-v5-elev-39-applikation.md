# Loggstrategi v5 – elev-39 (applikationszon, Alexander Schön)

Zon: applikationszon · Server: elev-39 (Rocky Linux 9, eth1 172.20.1.139, eth2 10.40.8.65)
Loggserver: log.sec.chas-lab.dev (172.20.1.58) · Transport: TLS, TCP 6514 (RFC 5425) · Format: RFC 5424 (`RSYSLOG_SyslogProtocol23Format`)
Urval: `*.info;mail.none;cron.none` via `/etc/rsyslog.d/70-central-tls.conf`

> Status: utkast, klart för korsgranskning.

## 1. Loggkällekarta

| Källa | Var / hur den läses | Exempelrad (från servern) | Vilken fråga den besvarar | Rotation | Hur långt bakåt | Tidsformat |
|---|---|---|---|---|---|---|
| Systemlogg | `/var/log/messages` (rsyslog) | `Nov  4 07:32:47 elev-39 systemd-logind[651]: New session 105 of user student.` | Vem/vad har loggat in på servern och när? (fråga 1) | weekly, rotate 4 (`/etc/logrotate.conf`) | ca 4 veckor (äldsta fil `messages-20261011`) | `Nov  4 07:32:47`, utan år och tidszon |
| Autentiseringslogg | `/var/log/secure` | `Nov  4 07:32:47 elev-39 sshd-session[85970]: Accepted password for student from 10.40.8.65 port 53002 ssh2` | Vem loggade in via SSH, från vilken adress, lyckades det eller misslyckades det, och när? (fråga 1) | weekly, rotate 4 | ca 4 veckor | som systemloggen |
| Paketfiltrets logg | kernel-loggen i `messages` (prefix `ZON-`) | `Nov  1 02:38:45 elev-39 kernel: ZON-UT-DROP IN= OUT=eth0 SRC=178.105.124.241 DST=146.75.122.132 LEN=60 … PROTO=TCP SPT=46732 DPT=443 … SYN` | Vilka anslutningar blockerades, i vilken riktning, från vilken adress till vilken adress och port, och när? (fråga 4) | som messages | ca 4 veckor | som systemloggen |
| Journal (journald) | `journalctl`, lagras i minnet (`/run/log/journal`; `/var/log/journal` saknas, `Storage=auto`) | `Nov 05 07:56:13 elev-39 systemd-logind[651]: Removed session 109.` | Vad har hänt på servern sedan senaste start? (fråga 1, 4). Begränsning: sparas bara i minnet och försvinner vid omstart. | ingen, nollställs vid omstart | sedan senaste start (4 okt 15:46) | `Nov 05 07:56:13`, utan år och tidszon (dagen nollfylld) |
| Audit (auditd) | `/var/log/audit/audit.log` (bara root) | `type=USER_START msg=audit(1793777826.873:4874): pid=86037 uid=1000 auid=1000 ses=105 … acct="root" exe="/usr/bin/sudo" … res=success` | Vilka root-kommandon kördes och vem (inloggat konto, auid) låg bakom, och när? (fråga 2) | storleksbaserad: `max_log_file = 8` MB, `num_logs = 5`, `ROTATE` | beror på mängden loggat, inte på tid (högst ca 40 MB) | epoch-sekunder + serienummer: `msg=audit(1793777826.873:4874)` |
| Cron | `/var/log/cron` | `Nov  4 07:01:01 elev-39 CROND[85919]: (root) CMDEND (run-parts /etc/cron.hourly)` | Ingen av fråga 1–5 (struken) | weekly, rotate 4 | ca 4 veckor | `Nov  4 07:01:01` |
| Paketinstallation (dnf) | `/var/log/dnf.log` | `2026-11-04T06:16:44+0000 DDEBUG Cleaning up.` | Ingen av fråga 1–5 (struken) | dnf egen rotation (`dnf.log.1`) | oklart (`dnf.log.1` är ca 1 MB) | ISO 8601 med tidszon (`+0000`) |
| Webbserver | `/var/log/httpd/access_log`, `error_log` | `[Sun Nov 01 00:00:08.719193 2026] [mpm_event:notice] [pid 17245:tid 17245] AH00489: Apache/2.4.62 (Rocky Linux) configured -- resuming normal operations` (error_log) | Ingen av fråga 1–5 (struken) | weekly, rotate 4 | ca 4 veckor | `[Sun Nov 01 00:00:08.719193 2026]`: veckodag, mikrosekunder och år, ingen tidszon |

Obs (audit): före U5.3 gav `sudo auditctl -l` `No rules`, så audit loggade bara standardhändelser (inloggning, sudo-autentisering), inte kommandon (EXECVE). Sedan 8 nov 22:27 UTC finns regeln i `/etc/audit/rules.d/rootcmd.rules` (`-a always,exit -F arch=b64 -S execve -F euid=0 -k rootcmd`, plus b32), och posterna skickas centralt via audit-pluginet (`audispd-plugins`, `active = yes` i `/etc/audit/plugins.d/syslog.conf`).

## 2. Fem frågor och urval

| Fråga (tidsspann, subjekt, vad som ska fastställas) | Källor som krävs | Går att besvara idag? |
|---|---|---|
| 1. Vem loggade in via SSH på elev-39 den 4 nov omkring 07:32, från vilka adresser, och var det en eller flera sessioner? | `secure` (`session opened` 07:32:17, `Accepted password for student from 10.40.8.65` 07:32:47), `wtmp` (`last`: `pts/1` från 172.20.1.2 och `pts/3` från 10.40.8.65), `messages` (`systemd-logind: New session 105 of user student`) | **Ja.** Båda inloggningarna syns: första sessionen via jump (172.20.1.2), andra från elev-39:s egen teamnätsadress (10.40.8.65). Tidszon kontrollerad: servern går på UTC och klockan är synkad (`timedatectl`: Etc/UTC, synchronized: yes), samma som loggservern. |
| 2. Vilka kommandon körde samma session (session 105) som root, och vilket konto stod bakom (`auid`)? | `audit.log` (SYSCALL/EXECVE, `ses=105`, `auid=1000`), `secure` (sudo-rader) | **Delvis för session 105, ja framåt.** Session 105 (4 nov) kom före audit-regeln, så där visar audit bara sudo-händelser (`ses=105 auid=1000`), och själva kommandona syns som sudo-rader i `secure`. Sedan 8 nov 22:27 UTC loggar regeln alla root-kommandon (EXECVE med `auid`, nyckel `rootcmd`) och de når loggservern. |
| 3. Vilka kommandon körde användare som root (via sudo) på elev-39 den 4 nov mellan 07:00 och 08:00, vilket konto stod bakom varje kommando och vilket kommando var det? | `secure` (sudo-rader med `COMMAND=`), `audit.log` (SYSCALL/EXECVE med `auid`) | **Ja.** Sudo-raderna i `secure` visar konto, terminal och kommando: `Nov  2 09:43:13 elev-39 sudo[84460]: student : TTY=pts/3 ; PWD=/home/student ; USER=root ; COMMAND=/bin/nano zon.nft`. Fullständig EXECVE-spårning med `auid` finns sedan 8 nov (audit-regeln från U5.3), men inte bakåt i tiden. |
| 4. Blockerades något utgående mot port 443 från elev-39 den 1 nov, när, och till vilken adress? | `ZON-UT-DROP` i `messages` (kernel) eller `journalctl -k -g ZON-` | **Ja.** Exempelrad finns (1 nov 02:38:45, mål 146.75.122.132:443). Förbehåll: loggregeln har `limit rate`, så alla paket loggas inte, men *att* det blockerades syns. |
| 5. Slutade någon server skicka loggar under dygnet 3 nov, vilken, och när var den sista raden per server? | loggservern (`loggsök` på jump): senaste tidsstämpel per avsändare | **Delvis.** Min egen server skickar nu över TLS, så jag kan se när min senaste rad kom (`loggsök`). Jag kan inte jämföra andra servrar, eftersom jag bara ser mina egna rader. |

### Strukna källor och skäl

| Struken källa | Varför ingen fråga behöver den |
|---|---|
| `/var/log/maillog` | 0 byte. Ingen mejltjänst körs på servern. |
| `/var/log/spooler` | 0 byte. Ingen tjänst skriver dit. |
| `/var/log/cron` | Ingen av fråga 1–5 handlar om schemalagda jobb. *Argument för att behålla:* den kan visa att en angripare lagt in ett schemalagt jobb för att få kvar åtkomst (persistens). |
| `/var/log/dnf.log` | Ingen av fråga 1–5 handlar om paketändringar. *Argument för att behålla:* den visar vilka paket som installerats och när. |
| `/var/log/httpd/` | Ingen av fråga 1–5 handlar om anrop till webbservern. *Argument för att behålla:* som applikationszon är webbservern en tjänst i vår zon, och en fråga om anrop och fel vore naturlig. |

## 3. Verifieringsanteckning

| Identifierare | Ungefärlig tid (UTC) | Värdnamn | Källa | Söksträng | Hittad centralt? | Avvikelse? |
|---|---|---|---|---|---|---|
| spar588dcf35 (U5.1) | 2026-11-05 09:48 | elev-39 | logger (user.notice, tagg sparbarhet) | `loggsök spar588dcf35` | Ja, kanal TLS | Nej |
| efter-stangning-514 | 2026-11-05 10:16 | elev-39 | logger (user.notice, tagg sparbarhet) | `loggsök efter-stangning-514` | Ja, kanal TLS | Nej. Klartextflödet (514) var då avstängt. |
| spar06c2a727 (U5.2) | 2026-11-05 10:51:54 | elev-39 | logger (tagg kotest) | `loggsök spar06c2a727` | Ja, kanal TLS | Ja. Kom 280 s efter att den skapades (avsiktligt avbrott). Tidsstämpeln är den ursprungliga. |
| U5-omstart-efter3 (omstartstest) | 2026-11-09 07:48 | elev-39 | logger (user.notice, tagg sparbarhet) | `loggsök U5-omstart-efter3` | Ja, kanal TLS | Ja. Markörerna `efter` (07:39) och `efter2` (07:43) kom inte fram: rsyslogs `imjournal` kastade dem när gränsen på 20 000 meddelanden per 600 s nåddes. Markören `U5-omstart-fore` (07:34:44, före omstarten) kom fram. |
| spare5efe8f8 (U5.3) | 2026-11-08 22:28 | elev-39 | auditd (`sudo /usr/bin/touch /tmp/spare5efe8f8`, via audit-pluginet) | `loggsök spare5efe8f8` | Ja, kontrollen godkändes och visade `auid=student` | Nej |

## 4. Skyddsbeskrivning

Kedjan: händelse på elev-39 → rsyslog (gtls, x509/name, kö på disk) → TLS 6514 → loggservern (sparar per avsändaradress) → `loggsök` på jump.

### 4.1 Vem kan ändra eller radera loggen lokalt?

Loggfilerna är skyddade mot vanliga användare: `messages` och `secure` är `-rw------- root root`, `/var/log/audit` är `drwx------ root` och som `student` fick jag `Permission denied` på `audit.log`. Bara root kan radera filer, eftersom `/var/log` är `drwxr-xr-x root root`. Skyddet gäller däremot inte mot den som har sudo: mitt konto är med i gruppen wheel (`id`: `groups=1000(student),10(wheel)`), så jag eller en angripare som tagit över kontot kan läsa, ändra och radera alla lokala loggar. Journalen ligger dessutom i minnet (`/run/log/journal`) och försvinner vid omstart.

### 4.2 Vem kan ändra eller radera loggen centralt?

Som elev kan jag inte det. Jag har inget konto på loggservern (guiden), och på jump finns inte ens katalogen med loggarna (`ls -ld /var/log/remote` gav `No such file or directory`; `id` på jump visar bara grupperna `elev-39` och `elever`). Det enda jag kan göra är att läsa mina egna rader med `loggsök`. Loggservern sorterar raderna efter avsändarens adress (172.20.1.139) och inte efter värdnamnet i meddelandet, så ingen annan kan skriva i min logg och jag kan inte skriva i andras. En angripare med root på elev-39 kan alltså inte ändra eller radera det som redan skickats. Hen kan bara sluta skicka, eller skicka falska rader från min adress. Varje rad får dessutom en mottagningstid från loggserverns egen klocka (till exempel `mottaget 2026-11-05T10:56:35Z`), som en angripare på min server inte kan påverka.

*Begränsning:* min rsyslog har inget klientcertifikat (varningen `certificate file / key file is not set` vid `rsyslogd -N1`). Jag kontrollerar alltså loggservern (CA + namn), men loggservern identifierar mig på adress och inte på certifikat.

### 4.3 Lagringstid och rotation

**Lokalt:**
- `messages`, `secure`, `cron`: `/etc/logrotate.d/rsyslog` har inga egna rotationsvärden, så de ärver `weekly` och `rotate 4` från `/etc/logrotate.conf`, alltså ca 4 veckor.
- `audit.log`: `max_log_file = 8`, `num_logs = 5`, `max_log_file_action = ROTATE` (`/etc/audit/auditd.conf`). Det blir högst ca 40 MB, och tiden beror på hur mycket som loggas.
- Journalen ligger i minnet och finns bara sedan senaste start.

**Centralt:** okänt. Guiden anger ingen lagringstid, och jag har inget konto på loggservern där jag kan se rotationen.

### 4.4 Vad händer när mottagaren är onåbar?

Konfigurationen i `/etc/rsyslog.d/70-central-tls.conf`:

```
action.resumeRetryCount="-1"
queue.type="LinkedList"
queue.filename="fwd_log"
queue.maxDiskSpace="100m"
queue.saveOnShutdown="on"
```

- **Kö och buffert:** `queue.filename` gör att kön kan ligga på disk. `resumeRetryCount="-1"` ger oändligt antal omförsök, och `saveOnShutdown="on"` sparar kön när rsyslog stängs av.
- **Belägg (U5.2):** när jag blockerade utgående 6514 och startade om rsyslog två gånger fanns köfilerna `fwd_log.00000001` och `fwd_log.qi` i `/var/lib/rsyslog/` (`sudo ls -l`). Efter att jag tagit bort blocket kom händelsen fram 280 s senare med sin ursprungliga tidsstämpel.
- **Gräns:** `queue.maxDiskSpace="100m"`. Enligt rsyslogs dokumentation (queue_parameters) kastas ett meddelande om det inte kan köas inom `queue.timeoutEnqueue` (2000 ms som standard). Dokumentationen anger inte uttryckligen vad som händer när `maxDiskSpace` nås, så det är inte verifierat.
- **Belägg (omstart, 9 nov):** efter en omstart kunde rsyslog inte nå loggservern på ca 2 minuter (`cannot connect to 172.20.1.58:6514 (took 131.90 seconds): Connection timed out`, 07:38:05), men åtgärden återupptogs (`action ... resumed`) och kön levererade. Anslutningen var `ESTAB` efteråt (`ss -tn dst 172.20.1.58`).
- **Utan diskkö:** en ren minneskö (utan `queue.filename`) hade förlorat händelsen vid omstarten.

### 4.5 Scenario: en angripare med root på elev-39

**Hen raderar eller ändrar de lokala loggarna** (`messages`, `secure`, `audit.log`). Det går, eftersom root äger filerna. Men raderna har redan skickats till loggservern, eftersom vidarebefordran sker i realtid: i U5.1 skapades händelsen 09:48:04.536 och loggservern tog emot den 09:48:04.636. Ett test med `sudo /usr/bin/echo sudo-test-1106` gav samma bild (skapad 14:42:02.069, mottagen 14:42:02.227). Det som hände fram till raderingen finns kvar centralt, och jag kan inte själv ändra det där.

**Hen stoppar loggningen** (`systemctl stop rsyslog`) **eller blockerar 6514 i nftables.** Då blir det tyst. Men kommandona kommer själva med i `secure` som sudo-rader med `COMMAND=`, och de skickas i realtid (se testet ovan) innan loggningen stängs av. Tystnaden syns också centralt som att inga nya rader kommer (fråga 5). Blockerar hen bara flödet sparar kön händelserna på disk (U5.2), men root kan radera köfilerna i `/var/lib/rsyslog`, så en lucka kan uppstå.

**Hen ändrar klockan eller skickar falska rader.** Meddelandets tidsstämpel kommer från min servers klocka och kan manipuleras, men loggservern sätter egen mottagningstid (`mottaget ...Z`), så en avvikelse blir synlig. Falska rader kan hen skicka från min adress, men hen kan inte ta bort de riktiga.

**Vad kedjan inte fångar:** händelser mellan att hen tar över kontot och att något loggas, samt allt som sker medan loggningen är avstängd. Audit-posterna skickas sedan U5.3 vidare via audit-pluginet, men bara för händelser efter 8 nov 22:27 UTC. Och en svaghet jag hittade vid omstartstestet: `imjournal` har en gräns på 20 000 meddelanden per 600 s (`4089 messages lost due to rate-limiting`, 9 nov 07:46), och audit-regeln för root-kommandon genererar många poster. En angripare med root kan därför översvämma loggen och få senare händelser kastade innan de når kön. Och en angripare med root kan stoppa `auditd` eller ändra reglerna (`auditctl -D`), vilket syns som tystnad centralt. Och en admin på loggservern kan ändra där, vilket jag som elev inte kan verifiera.

## 5. Flödestabell, ändringar för loggflödet

| Rad | Källa | Mål | Protokoll | Port | Riktning | Motivering | Filtreras var |
|---|---|---|---|---|---|---|---|
| D5 (borttagen) | elev-39 | loggserver 172.20.1.58 | tcp | 514 | Utgående | Klartext från vecka 4. Stängd 5 nov, ersatt av TLS. Regeln (handle 17) och `60-brandvagg.conf` är borttagna. | Värd (nftables) |
| U5 (ny) | elev-39 | loggserver 172.20.1.58 | tcp | 6514 | Utgående | Loggar över TLS (MCFFS 2026:11 4 kap. 25 §) | Värd (nftables). Molnbrandväggen ser inte privat trafik. |
| (tillfällig) | elev-39 | Rocky-speglar | tcp | 80, 443 | Utgående | Installera `rsyslog-gnutls`. Öppnades i både värdens och molnets brandvägg och togs bort direkt efter installationen. | Värd och moln |

Rad-numren följer inte teamets numrering ännu. Sätt rätt nummer när flödestabellen samordnas.

## 6. Öppna punkter

- Audit skickas centralt sedan U5.3 (regel + `audispd-plugins`). Verifierat med `kontrollera U5.3` och `ausearch --input-logs -k rootcmd`.
- Omstartstest gjort 9 nov: brandväggsregler, audit-regler och tjänsterna (nftables, rsyslog, auditd) kom upp av sig själva, och loggningen fungerade igen efter ca 2 minuter.
- `imjournal` kastar meddelanden när gränsen på 20 000 per 600 s nås (audit-regeln ger många poster). Beslut behövs: höja gränsen (`Ratelimit.Burst`) eller minska bruset från audit-regeln, till exempel genom att filtrera bort rutinkommandon.
- Webbserverloggen, cron och dnf samlas inte in (strukna enligt avsnitt 2).
- Journalen ligger bara i minnet och försvinner vid omstart.
- Central lagringstid är okänd. Fråga Peter.
- Min rsyslog saknar klientcertifikat, så loggservern identifierar mig på adress och inte på certifikat.
- Teamets flödestabell behöver uppdateras (514 → 6514).

## AI-not

Alla utskrifter och exempelrader kommer från min egen server (elev-39) och från jump, där jag körde kommandona själv. Claude hjälpte till med struktur, förklarade kedjan, föreslog kommandon och tester (sudo-testet och sökningen efter audit-rader) och skrev utkast till texten under 4.1, 4.2, 4.4 och scenariot samt till rader i avsnitt 5. Claude guidade mig också genom U5.3 (audit-regel och audit-plugin), där jag körde alla kommandon själv och kontrollerade utskrifterna. Frågekolumnen i loggkällekartan skrev jag själv, med små språkliga justeringar, och fråga 3–5 samt formuleringarna under *Strukna källor* föreslogs av Claude och granskades av mig. Rubrikstrukturen följer ett kamratexempel (elev-32:s fil), men innehållet är mitt eget. Uppgiften om rsyslogs gräns (`timeoutEnqueue`) kommer från rsyslogs dokumentation och är inte testad.
