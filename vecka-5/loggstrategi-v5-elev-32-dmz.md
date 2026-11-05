# Loggstrategi v1 – elev-32 (DMZ, Alex O)

Zon: DMZ (10.40.8.32/27) · Loggserver: log.sec.chas-lab.dev (172.20.1.58)
Transport: TLS, TCP 6514 (RFC 5425) · Format: RFC 5424
Allt skickas (`*.*`) via /etc/rsyslog.d/50-central.conf.

## 1. Loggkällekarta

| Källa | Var / hur den läses | Exempelrad (från servern) | Vilken fråga den besvarar | Rotation | Hur långt bakåt | Tidsformat |
|---|---|---|---|---|---|---|
| Systemlogg | /var/log/messages | `Nov  5 09:05:37 elev-32 audisp-syslog[97690]: type=EOE msg=audit(1793869537.076:9383):` | Vad hände på servern? | weekly, rotate 4 | ca 5 veckor | RFC 3164 lokalt (inget år, ingen tidszon) |
| Autentiseringslogg | /var/log/secure | `Nov  5 08:44:57 elev-32 sshd-session[98354]: pam_unix(sshd:session): session opened for user student(uid=1000) by student(uid=0)` | Vem loggade in, och när? | weekly, rotate 4 | ca 5 veckor | RFC 3164 lokalt |
| Journal | journalctl -u sshd | `Nov 05 08:44:57 elev-32 sshd-session[98354]: pam_unix(sshd:session): session opened for user student(uid=1000)…` | Samma händelser som secure, med fler fält | Bara i minnet (/var/log/journal saknas) | Tills omstart (41,2 MB nu) | Kort format utan år |
| Brandväggslogg | journalctl -k, prefix ZON-IN-DROP / ZON-UT-DROP | `2026-11-05T00:17:25+00:00 172.20.1.132 elev-32 kernel: ZON-IN-DROP IN=eth0 OUT= … SRC=216.25.89.122 … PROTO=TCP SPT=49991 DPT=80 … SYN` | Vilken trafik blockerades, från vilken adress och mot vilken port? | som messages | ca 5 veckor | Sekunder, +00:00 |
| Auditd | sudo ausearch -k rootcmd -i | `type=USER_CMD msg=audit(11/02/2026 09:46:40.893:6758) : pid=82249 uid=student auid=student ses=173 … cmd=nano /etc/nftables/zon.nft exe=/usr/bin/sudo … res=success` | Vem (auid) körde kommandon som root? | max_log_file = 8 MB, num_logs = 5, ROTATE | Max 5 filer à 8 MB | Epoch i msg=audit(TID:SERIENUMMER) |
| Webbserverlogg | /var/log/httpd/access_log | Ingen rad: filen är tom (0 byte) sedan rotationen 1 nov | Vilka klienter nådde webbportalen? | weekly, rotate 4 | ca 4 veckor (access_log-20261011 är äldsta) | Apache-format |
| Cron | /var/log/cron | `Nov  5 09:01:01 elev-32 CROND[98411]: (root) CMDEND (run-parts /etc/cron.hourly)` | Vilka schemalagda jobb körde? | weekly, rotate 4 | ca 5 veckor | RFC 3164 |

## 2. Fem frågor och urval

| Fråga (tidsspann, subjekt, vad som ska fastställas) | Källor som krävs | Går att besvara idag? |
|---|---|---|
| Vilka konton loggade in eller misslyckades via ssh på elev-32 de senaste 24 timmarna, och från vilka adresser? | Autentiseringslogg (secure) | Ja, lokalt och centralt |
| Vilka externa adresser blockerades mot port 80 på eth0 de senaste 24 timmarna, och hur ofta? | Brandväggslogg | Ja, centralt (loggsök ZON-) |
| Vilken person (auid) körde kommandon som root på elev-32 den senaste veckan? | Auditd | Ja (U5.3) |
| Vilka klienter nådde webbportalen på port 443 den senaste timmen? | Webbserverlogg | Nej: access_log är tom och skickas inte centralt (ingen imfile-rad) |
| Gick loggflödet från elev-32 till loggservern utan avbrott det senaste dygnet? | Systemlogg, journalctl -u rsyslog | Ja |

### Strukna källor

| Struken källa | Varför ingen fråga behöver den |
|---|---|
| Mail (maillog) | Filerna är tomma och ingen fråga rör e-post. |
| Spooler | Filerna är tomma och ingen fråga rör utskrifter. |
| Cron | Ingen av de fem frågorna rör schemalagda jobb. |

## 3. Verifieringsanteckning

| Identifierare | Ungefärlig tid (UTC) | Värdnamn | Källa | Söksträng | Hittad centralt? | Avvikelse? |
|---|---|---|---|---|---|---|
| spar9f73ca71 (U5.1) | 2026-11-03 (exakt tid ej noterad) | elev-32 | sparbarhet | spar9f73ca71 | Godkänd i kontrollera U5.1 (LÖST 2026-11-03) | loggsök hittar den inte 5 nov. Orsak ej utredd |
| spardcb79368 (U5.2) | skapad 2026-11-04 07:30:18, mottagen 07:32:41 | elev-32 | sparbarhet | spardcb79368 | Ja, TLS, ca 2 min 23 s efter skapandet | Första försöket kom fram på 1 s eftersom 50-central.conf saknades och ingen kö fanns |
| spar2589b80e (U5.5) | 2026-11-05 07:35:34 | elev-32 | sparbarhet | spar2589b80e | Ja, TLS | Kom två gånger (RFC 5424 och RFC 3164) eftersom 50-tls.conf hade en egen action utan template. Efter att filen flyttats till /root/50-tls.conf.bak kom en rad (07:44:04) |

## 4. Skyddsbeskrivning

### 4.1 Vem kan ändra eller radera loggen lokalt?
Root och användare med sudo. `sudo -l` visar att student har `(ALL) ALL`. Filerna messages och secure är `-rw------- root root`, så vanliga användare kan inte ens läsa dem. En angripare med root kan radera eller ändra dem.

### 4.2 Vem kan det centralt?
Ingen från vår server. Vi har inget konto på loggservern, och den sorterar raderna efter avsändaradress, så ingen kan skriva i någon annans loggar. Vi söker bara via loggsök på jump. Begränsning (RFC 5425 §5.3, §6.1): loggservern identifierar oss på adress, inte på certifikat.

### 4.3 Lagringstid och rotation
Lokalt: /etc/logrotate.conf har `weekly` och `rotate 4`, alltså ca 5 veckor. Filerna messages-20261011 till messages-20261101 visar det. Auditd sparar max 5 filer à 8 MB. Journalen ligger bara i minnet (`#Storage=auto`, /var/log/journal saknas, /run/log/journal finns) och försvinner vid omstart.
Centralt: ej verifierat, vi har ingen åtkomst till loggservern.

### 4.4 Vad händer när mottagaren är onåbar?
Belägg i /etc/rsyslog.d/50-central.conf:
- `queue.type="LinkedList"` och `queue.filename="fwd_queue"`: kön ligger på disk i /var/lib/rsyslog
- `queue.saveOnShutdown="on"`: kön sparas vid nedstängning
- `action.resumeRetryCount="-1"`: rsyslog försöker för alltid

Verifierat i U5.2: raden skapades 07:30:18, port 6514 var blockerad, rsyslog startades om två gånger, och raden kom fram 07:32:41 när blockeringen togs bort. Med en ren minneskö hade raden försvunnit vid första omstarten.
Gräns: ingen storlek är satt i vår konfiguration. Vad som händer när disken blir full är **[ej verifierat]**.

### 4.5 Scenario: angripare med root
Angriparen raderar /var/log/secure och /var/log/messages, rensar journalen och stoppar rsyslog.
Kedjan fångar ändå allt som redan skickats över TLS, och auditposterna med auid, som visar vem som loggade in även efter sudo.
Kedjan fångar inte rader som ändrades innan de skickades, eller händelser medan rsyslog är stoppad. TLS skyddar bara transporten.

## 5. Flödestabell, ändringar för loggflödet

| Rad | Källa | Mål | Protokoll | Port | Riktning | Motivering | Filtreras var |
|---|---|---|---|---|---|---|---|
| D5 | Egen server | Loggserver 172.20.1.58 | tcp | 514 | Utgående | Brandväggslogg i klartext (vecka 4, 60-brandvagg.conf). Bör bytas till TLS eller tas bort | Värd (eth1) |
| D8 (ny) | Egen server | Loggserver 172.20.1.58 | tcp | 6514 | Utgående | Loggar över TLS (MCFFS 2026:11 4 kap. 25 §) | Värd (eth1) |
| D7 (ny) | Egen server | Internet | tcp | 80, 443 | Utgående | Paketuppdateringar (dnf). Bred regel, ta bort när den inte behövs | Värd (eth0) |

## 6. Öppna punkter
- Brandväggsloggen skickas även i klartext (port 514) via 60-brandvagg.conf. Teamet ska besluta om den byts till TLS eller tas bort.
- Webbloggen skickas inte centralt (ingen imfile-rad).
- Journalen ligger bara i minnet.