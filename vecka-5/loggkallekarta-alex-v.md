# Vecka 5 – Centralloggning: loggkällekarta, urval och skyddsbeskrivning

## Loggkällekarta (Workshop 1, del 1)

| Källa | Var/hur den läses | Exempel rad (klistrad) | Vilken fråga den besvarar | Rotation | Hur långt bakåt | Tidsformat |
|---|---|---|---|---|---|---|
| Systemlogg (journal/messages) | `journalctl` (volatile, RAM) eller `/var/log/messages` (fil) | `Nov  5 08:57:49 elev-45 audisp-syslog[101587]: type=CWD msg=audit(1793869069.952:10925): cwd="/home/student"` | Vad har hänt på systemnivå – tjänster, fel, varningar | Journal: ingen rotation, rensas vid omstart (volatile). `/var/log/messages`: logrotate, veckovis, 4 veckors backlogg, ej komprimerad | Journal: försvinner vid omstart. `/var/log/messages`: upp till ~4 veckor | Klassiskt syslog-format (`Nov 4 10:15:02`), ingen tidszon/år |
| Autentiseringslogg (secure) | `/var/log/secure` eller `journalctl -u sshd` | `Nov 5 08:35:39 elev-45 sshd-session[103084]: Accepted publickey for student from 172.20.1.2 port 44480 ssh2: ED25519 SHA256:rvhrmsA8UmYmuUL1OQEzf+hBC3PUVuD8xizPDehL5oc` | Vem loggade in/försökte logga in, lyckades eller misslyckades, varifrån, med vilken metod | Delar `/etc/logrotate.d/rsyslog`, veckovis, 4 veckor, ej komprimerad | Upp till ~4 veckor | Klassiskt syslog-format (`Nov 5 08:35:39`) |
| Paketfiltrets logg (vecka 4) | `journalctl -k -g ZON-` | `Nov 05 08:45:24 elev-45 kernel: ZON-UT-DROP IN= OUT=eth1 SRC=172.20.1.145 DST=172.20.1.58 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=22673 DF PROTO=TCP SPT=44750 DPT=514 WINDOW=64860 RES=0x00 SYN URGP=0` | Vilken trafik nekades av brandväggen – källa, mål, port, skede av anslutningen | Delar journalens rotation (volatile, försvinner vid omstart) | Samma som systemlogg | Syslog-format (`Nov 05 08:45:24`) |
| Auditd – root-kommandon | `sudo ausearch -k root_command -i` | `type=SYSCALL msg=audit(11/05/2026 08:50:21.323:10921): arch=x86_64 syscall=execve success=yes exit=0 ... auid=student uid=root euid=root ... comm=ausearch exe=/usr/sbin/ausearch key=root_command` | Vem körde vilket kommando med root-behörighet – auid (ursprunglig inloggad användare), inte bara uid/euid | `cat /etc/audit/auditd.conf`, leta efter max_log_file/num_logs]* | *[samma fil]* | MM/DD/ÅÅÅÅ HH:MM:SS.mmm – skiljer sig från syslog-formatet i övriga källor |
| rsyslogs egen status | `journalctl -u rsyslog` |  `Nov 05 08:47:54 elev-45 rsyslogd[103204]: warning: key file is not set [v8.2510.0-2.el9_8.2 try https://www.rsyslog.com/e/2331 ]` | Om loggöverföringen till loggservern faktiskt fungerar | Samma som systemlogg (volatile) | Samma som systemlogg | Samma som systemlogg |
| Nätverkshändelser (NetworkManager/DHCP) | `journalctl -u NetworkManager` | `Nov 05 03:46:04 elev-45 NetworkManager[683]: <info>  [1793850364.9473] dhcp4 (eth0): state changed new lease, address=2.28.103.167` | Nätverksförändringar – adresstilldelning, gränssnitt upp/ner | Samma som systemlogg (volatile) | Samma som systemlogg | Samma som systemlogg |

---

## Fem frågor och urval (Workshop 1, del 2)

| Fråga (tidsspann, subjekt, vad som ska fastställas) | Källor som krävs | Går att besvara idag? |
|---|---|---|
| Vilka källadresser försökte nå en nekad port på min server under det senaste dygnet? | Paketfiltrets logg | Ja |
| Loggade någon in som root på min server mellan kl 02 och 04 natten till idag? | Autentiseringslogg | Ja |
| Vilket konto körde `sudo`-kommandon på min server den senaste veckan, och vilka kommandon? | Auditd (root_command, auid) | Ja |
| Startade eller stoppade någon systemtjänst oväntat under natten? | Systemlogg | Delvis – bara om servern inte startats om sedan dess (journal är volatile) |
| Avbröts loggöverföringen till loggservern någon gång den senaste veckan, och i så fall hur länge? | rsyslogs egen status + paketfiltrets logg | Delvis – kan bara verifieras lokalt, inte vad som faktiskt kom fram centralt |

---

## Strukna källor och skäl

| Struken källa | Varför ingen fråga behöver den |
|---|---|
| Webbserverns applikationslogg | Ingen webbtjänst körs på den här specifika servern |
| DNS-serverns frågelogg | Hanteras av Hetzners resolvrar, ingen egen åtkomst |

---

## Observationer gjorda under arbetet (värt att ha med i redovisningen)

1. **Journalen är volatile** – `/var/log/journal` saknas, så all lokal journal-historik (inklusive paketfiltrets ZON-loggar) försvinner vid omstart. `/var/log/messages` och `/var/log/secure` är mer pålitliga lokalt eftersom de skrivs till disk och roteras separat.
2. **Kvarvarande gammal rsyslog-konfiguration hittad och åtgärdad.** `/etc/rsyslog.d/50-gammal-brandvagg.conf` försökte fortfarande skicka till den gamla klartextporten 514 (vecka 4), vilket orsakade dubbletter och extra ZON-UT-DROP-brus efter att 514 stängdes i brandväggen. Filen döptes om/inaktiverades.
3. **SSH-reglerna i `zon.nft` saknar `iifname`-bindning** – de matchar port 22 oavsett gränssnitt. Värt att nämna som en möjlig förbättring (binda till `eth1` specifikt) i Skyddsbeskrivningen, även om inget faktiskt missbruk observerats.
4. **Lösenordsautentisering är aktiverad** vid sidan av nyckelbaserad inloggning (sågs i secure-loggen: "Accepted password" via 10.40.8.81) – värt att kontrollera om det är avsiktligt.


## Skyddsbeskrivning av loggkedjan (Workshop 2, del 2)

### 1. Vem kan ändra eller radera loggen lokalt?
Root, eller vem som helst som uppnår root via `sudo` (bekräftat: `student` har sudo-rättighet, se secure-loggen). Belägg: `ls -l /var/log/secure /var/log/messages`, `sudo -l`.

### 2. Vem kan det centralt?
Ingen av oss – "Du har inget konto på loggservern" (studieguiden). Det är i sig en del av skyddet: en angripare som tar sig in på vår server kan inte radera det som redan nått fram centralt.

### 3. Lagringstid och rotation
Lokalt: se tabellen ovan (journal volatile, filer ~4 veckor). Centralt: okänt utan åtkomst till loggservern – explicit begränsning att nämna.

### 4. Vad händer när mottagaren är onåbar?


### Avslutande scenario
En angripare med root kan radera lokala loggar och stänga av rsyslog för att stoppa framtida leverans – men allt som redan skickats och tagits emot centralt INNAN ingreppet ligger kvar, oåtkomligt för angriparen. TLS skyddar bara transporten (RFC 5425 §6.3), inte redan lagrad lokal data. Ett plötsligt gap i den centrala loggströmmen (ingen trafik från vår adress) vore i sig en avvikelse värd att larma på.
