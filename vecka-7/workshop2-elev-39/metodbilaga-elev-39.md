# Metodbilaga – elev-39 (Alexander Schön)

ITSX26 Kurs 2 · vecka 7 · Workshop 2: Dokumentera konfiguration med AI-stöd · 2026-11-27

## Så använde jag AI under veckan

- **Handledning:** Claude användes som handledare och parprogrammerare på U7.1–U7.6 och Ö1–Ö10. Jag körde alla kommandon på min server själv och skrev mina egna svar på övningarna. Claude ställde följdfrågor och jämförde med lösningsförslaget.
- **Härdningslistan:** Claude skrev ett utkast utifrån det vi gjort under veckan. Jag har granskat raderna mot servern.
- **Verifieringsskriptet:** Claude föreslog koden till kontroll 5. Jag granskade den (`diff`), installerade den och körde den.
- **Rättelseloggen:** Claude föreslog domarna. Kommandona (`belagg.sh`) skrevs av Claude, jag körde dem, och jag har granskat varje rad.
- **AI-utkastet till härdningsrapport** (workshopens uppgift) gjordes i en egen, ren chatt, se rubrik 2–3. Den här bilagan har utkast av Claude.
- **Planeringen av övningen** fick stöd av en annan AI-chatt (ett föreslaget upplägg) och av Claudes kommentarer på det.

## 1. Underlag som matades in

Underlaget insamlades på `elev-39` måndag 2026-11-23 10:33 och matades in i **försök 2** (försök 1 hade platshållare i stället för underlag, se rubrik 2).

| Del | Innehåll |
|---|---|
| Härdningslistan | `hardningslista-elev-39.md` (fyra rader), i den version som låg i `vecka-7/` |
| `sestatus` | hela utskriften |
| `semodule -l` | utdrag, bara raden `zonvakt` |
| `ps -eZ` | utdrag, bara raden för `zonvakt` (`zonvakt_t`, PID 4126) |
| Verifieringsskriptets utdata | de fem `GÄLLER`-raderna |

**Maskerat:** inget. Underlaget innehöll inga lösenord, flaggor eller icke-offentliga adresser. Maskeringen bestämdes inte i par, som studieguiden föreslår. Jag bedömde det själv, och det har jag inte hunnit stämma av med en parkamrat.

AI-assistent och modell för utkastet: Claude Sonnet 5.5, i en ren, temporär (inkognito) chatt utan tidigare kontext. Försök 1: datum och klockslag sparades inte. Försök 2: fredag 2026-11-27 på morgonen, exakt klockslag sparades inte eftersom den temporära chatten stängdes.

## 2. Prompt, ordagrant

Prompterna nedan är exakt de som skickades, utan ändringar.

### Försök 1 (platshållarna ersattes inte)

````text
Skriv ett utkast till en härdningsrapport för en Linux-server med SELinux.
Använd bara underlaget nedan. För varje härdningsåtgärd: beskriv effekten och hur den verifierades.
Underlaget är utdrag.

HÄRDNINGSLISTA:
[klistra in hardningslista-elev-39.md]

SESTATUS:
[klistra in sestatus-utskriften]

SEMODULE -L (utdrag):
[klistra in raden "zonvakt"]

PS -EZ (utdrag):
[klistra in raden med zonvakt_t]

VERIFIERINGSSKRIPTETS UTDATA:
[klistra in de fem GÄLLER-raderna]
````

### Försök 2

````text
Skriv ett utkast till en härdningsrapport för en Linux-server med SELinux.
Använd bara underlaget nedan. För varje härdningsåtgärd: beskriv effekten och hur den verifierades.
Underlaget är utdrag.

HÄRDNINGSLISTA:
# Härdningslista – elev-39 (applikationszonen)

Alexander Schön · ITSX26 Kurs 2 · vecka 7 · Workshop 1, del 1

| Nr | Åtgärd | Motivering – varför just här (vad händer om vi låter bli?) | Förväntad effekt | Hur den verifierades | Skript? |
|---|---|---|---|---|---|
| 1 | SELinux körs i **Enforcing**, både nu och efter omstart (`getenforce` och `SELINUX=enforcing` i `/etc/selinux/config`). | I Permissive loggas överträdelser men genomförs ändå, så ett angrepp på en tjänst stoppas inte. `setenforce 0` försvinner dessutom vid omstart och döljer att skyddet varit avstängt. Utan Enforcing kan en komprometterad tjänst göra allt som DAC tillåter, till exempel skriva i `/etc`. | Överträdelser stoppas (`permissive=0` i AVC) och läget är detsamma efter omstart. | `getenforce` → `Enforcing`. `grep '^SELINUX=enforcing' /etc/selinux/config`. Omstart (`sudo systemctl reboot`) och körning av skriptet efteråt. | ☑ `verifiera-hardning`, kontroll 1 och 2 |
| 2 | Tjänsten `zonvakt` körs under en egen tvingande policy: modulen `zonvakt` (`.te` och `.fc`) med domänen `zonvakt_t`, egen loggtyp `zonvakt_log_t` och **utan** `permissive zonvakt_t;`. Jag granskade audit2allow-förslagen och tog inte med de för breda eller onödiga raderna (till exempel `files_manage_etc_files`). | Utan egen domän kör tjänsten som `unconfined_service_t`, alltså utan MAC. Då stoppas inget om tjänsten tas över eller gör fel, och `overtrad`-beteendet (skriva i `/etc`) hade lyckats. En permissiv domän loggar bara. | `zonvakt` kör i `zonvakt_t`, får skriva sin logg, men nekas att skapa filer i `/etc` (typen `etc_t`). | `ps -eZ \| grep zonvakt` → `zonvakt_t`. `ausearch -i -m avc` visar nekningen i `etc_t` med `permissive=0`. Samma händelse hittad på loggservern (Wazuh, regel 100200). | ☑ `verifiera-hardning`, kontroll 3 |
| 3 | Brandvägg med nftables: tabell `inet zon`, kedjan `in` med **policy drop**, bara de flöden som behövs tillåts. Reglerna ligger i `/etc/nftables/zon.nft` (börjar med `flush ruleset`) och tjänsten `nftables` är aktiverad vid boot. | Utan brandvägg är alla lyssnande portar nåbara. Med policy drop är allt förbjudet utom det som uttryckligen tillåts, så en ny tjänst blir inte exponerad av misstag. Reglerna i en fil gör dem reproducerbara efter omstart. | Okända inkommande flöden droppas. De behövda flödena fungerar fortfarande (kontrollerna mot httpd på 80 och zonvakt på 7539). | `nft -a list chain inet zon in`. `nft -c -f /etc/nftables/zon.nft` (syntaxkontroll). `systemctl is-enabled nftables` och `is-active`. Verifierat efter omstart. | ◐ `verifiera-hardning`, kontroll 4 (aktiv vid boot, inte policy drop) |
| 4 | `zonvakt` får bara binda sin egen porttyp: ny typ `zonvakt_port_t`, port 7539 märkt med `semanage port -a -t zonvakt_port_t -p tcp 7539`, och raden `corenet_tcp_bind_generic_port(zonvakt_t)` borttagen. | Den raden tillät bindning till tre porttyper, alltså i praktiken vilken oreserverad port som helst. En komprometterad tjänst hade kunnat öppna en godtycklig port, till exempel som bakdörr. | `zonvakt_t` kan bara binda 7539. En annan port nekas. | `semanage port -l \| grep zonvakt_port_t`. `sesearch -A -s zonvakt_t -c tcp_socket -p name_bind` visar bara `zonvakt_port_t`. `seinfo --portcon=7539` visar `zonvakt_port_t`. Provat: port 7540 i `zonvakt.conf` ger `failed` och AVC `name_bind` (`unreserved_port_t`, `permissive=0`). Återställt. | ☑ `verifiera-hardning`, kontroll 5 (portmärkning och att `zonvakt_t` bara får binda `zonvakt_port_t`) |

SESTATUS:
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33

SEMODULE -L (utdrag, bara raden för zonvakt):
zonvakt

PS -EZ (utdrag, bara raden för zonvakt):
system_u:system_r:zonvakt_t:s0     4126 ?        00:00:00 zonvakt

VERIFIERINGSSKRIPTETS UTDATA:
GÄLLER: SELinux är i läge Enforcing nu (getenforce)
GÄLLER: /etc/selinux/config sätter SELINUX=enforcing
GÄLLER: zonvakt kör i domänen zonvakt_t (ps -eZ)
GÄLLER: nftables är aktiverad vid boot och kör (systemctl)
GÄLLER: port 7539 är märkt zonvakt_port_t och zonvakt_t får bara binda den (seinfo, sesearch)
````

## 3. Råsvar, oredigerat

### Försök 1

AI:n fick inget underlag och vägrade gissa. Den bad om underlaget och beskrev vad den skulle göra. Det gav inget utkast.

````markdown
Det ser ut som att mallen skickades med platshållarna kvar. Jag ser bara texten "[klistra in …]" och ingen faktisk data. Ingen fil har heller bifogats, så jag har inget underlag att skriva rapporten utifrån. Eftersom rapporten bara ska bygga på underlaget vill jag inte gissa mig till härdningsåtgärder, SELinux-lägen eller verifieringsresultat.
Klistra in följande som text i nästa meddelande:

1. Innehållet i `hardningslista-elev-39.md`
2. Utskriften från `sestatus`
3. Raden med `zonvakt` från `semodule -l`
4. Raden med `zonvakt_t` från `ps -eZ`
5. De fem GÄLLER-raderna från verifieringsskriptet

När jag har dem gör jag ett utkast med den här strukturen:

* Sammanfattning: serverns SELinux-status enligt `sestatus` (läge, policy)
* Åtgärder: en post per härdningsåtgärd med Åtgärd, Effekt och Verifiering
* SELinux-modul och processdomän: vad `zonvakt` och `zonvakt_t` visar
* Luckor och osäkerheter: sådant som underlaget inte täcker, eftersom det bara är utdrag

Åtgärder som saknar stöd i underlaget markerar jag som "ej verifierat" i stället för att fylla i något själv.
````

### Försök 2

Råsvaret nedan är hela svaret, kopierat orört från chatten.

`````markdown
# Härdningsrapport – Linux-server med SELinux (utkast)

Alexander Schön · ITSX26 Kurs 2 · vecka 7 · Workshop 1, del 1 · elev-39 (applikationszonen)

## 1. Syfte och underlag

Rapporten redovisar fyra härdningsåtgärder på servern i applikationszonen. För varje åtgärd beskrivs vad som gjorts, vilken effekt den har och hur den verifierades. Underlaget är härdningslistan, `sestatus`, utdrag ur `semodule -l` och `ps -eZ` samt utdata från verifieringsskriptet `verifiera-hardning`. Allt är utdrag. Där underlaget bara påstår att en kontroll gjorts, men inte visar kommandots utdata, anges det under respektive åtgärd.

## 2. Systemstatus

`sestatus` visar att SELinux är aktiverat med policyn `targeted`, att nuvarande läge är `enforcing` och att läget enligt konfigurationsfilen också är `enforcing`. MLS-status är `enabled`. Tjänsten `zonvakt` körs som process 4126 med kontexten `system_u:system_r:zonvakt_t:s0`, och modulen `zonvakt` finns bland de laddade modulerna.

## 3. Sammanfattning

- **Åtgärd 1, SELinux i Enforcing:** genomförd och verifierad, även med skript.
- **Åtgärd 2, egen tvingande policy för `zonvakt`:** genomförd. Skriptet verifierar domänen. Nekningen i `etc_t` och loggservern redovisas i listan men utan bifogad utdata.
- **Åtgärd 3, brandvägg med nftables:** genomförd. Skriptet täcker bara att tjänsten är aktiv vid boot, inte policy drop.
- **Åtgärd 4, begränsad portbindning:** genomförd och verifierad, även med skript och ett negativt test.

## 4. Åtgärder

### Åtgärd 1 – SELinux körs i Enforcing, nu och efter omstart

**Åtgärd.** SELinux är satt till Enforcing både i drift (`getenforce`) och i `/etc/selinux/config` (`SELINUX=enforcing`).

**Motivering.** I Permissive loggas överträdelser men genomförs ändå, så ett angrepp på en tjänst stoppas inte. `setenforce 0` försvinner vid omstart och döljer att skyddet varit avstängt. Utan Enforcing kan en komprometterad tjänst göra allt som DAC tillåter, till exempel skriva i `/etc`.

**Effekt.** Överträdelser stoppas (`permissive=0` i AVC), och läget är detsamma efter omstart.

**Verifiering.**

- `getenforce` gav `Enforcing`.
- `grep '^SELINUX=enforcing' /etc/selinux/config` kontrollerades.
- Servern startades om (`sudo systemctl reboot`) och verifieringsskriptet kördes efteråt.
- Skriptet (kontroll 1 och 2) rapporterar GÄLLER för både aktuellt läge och konfigurationsfilen. `sestatus` visar `Current mode: enforcing` och `Mode from config file: enforcing`.

**Reservation.** Utdata från körningen efter omstart redovisas inte separat i underlaget.

### Åtgärd 2 – `zonvakt` körs under en egen tvingande policy

**Åtgärd.** Tjänsten har en egen policymodul `zonvakt` (`.te` och `.fc`) med domänen `zonvakt_t` och den egna loggtypen `zonvakt_log_t`. Modulen innehåller inte `permissive zonvakt_t;`. Förslagen från audit2allow granskades, och de för breda eller onödiga raderna (till exempel `files_manage_etc_files`) togs inte med.

**Motivering.** Utan egen domän kör tjänsten som `unconfined_service_t`, alltså utan MAC. Då stoppas inget om tjänsten tas över eller gör fel, och beteendet att skriva i `/etc` (`overtrad`) hade lyckats. En permissiv domän loggar bara.

**Effekt.** `zonvakt` körs i `zonvakt_t`, får skriva sin logg men nekas att skapa filer i `/etc` (typen `etc_t`).

**Verifiering.**

- `ps -eZ | grep zonvakt` visar `zonvakt_t`. Utdraget bekräftar detta (`system_u:system_r:zonvakt_t:s0`, PID 4126).
- `semodule -l` visar modulen `zonvakt`.
- Skriptets kontroll 3 rapporterar GÄLLER för domänen.
- Enligt listan visar `ausearch -i -m avc` nekningen i `etc_t` med `permissive=0`, och samma händelse hittades på loggservern (Wazuh, regel 100200).

**Reservation.** Utdata från `ausearch` och från Wazuh finns inte i underlaget. Inget underlag visar heller direkt att domänen inte är permissiv, eller att skrivning till `zonvakt_log_t` fungerar.

### Åtgärd 3 – Brandvägg med nftables

**Åtgärd.** Tabellen `inet zon` har kedjan `in` med policy drop, och bara de flöden som behövs tillåts. Reglerna ligger i `/etc/nftables/zon.nft`, som börjar med `flush ruleset`. Tjänsten `nftables` är aktiverad vid boot.

**Motivering.** Utan brandvägg är alla lyssnande portar nåbara. Med policy drop är allt förbjudet utom det som uttryckligen tillåts, så en ny tjänst blir inte exponerad av misstag. Reglerna i en fil gör dem reproducerbara efter omstart.

**Effekt.** Okända inkommande flöden droppas, medan de behövda flödena fungerar fortfarande (httpd på port 80 och `zonvakt` på port 7539).

**Verifiering.**

- `nft -a list chain inet zon in` granskades.
- `nft -c -f /etc/nftables/zon.nft` gjordes som syntaxkontroll.
- `systemctl is-enabled nftables` och `is-active` kontrollerades.
- Enligt listan är detta verifierat efter omstart.
- Skriptets kontroll 4 rapporterar GÄLLER för att `nftables` är aktiverad vid boot och kör.

**Reservation.** Skriptet täcker bara att tjänsten är aktiv vid boot, inte att kedjans policy är drop (markerat ◐ i listan). Policy drop och de tillåtna flödena är därmed bara verifierade manuellt, och utdata från `nft`-kommandona finns inte i underlaget.

### Åtgärd 4 – `zonvakt` får bara binda sin egen porttyp

**Åtgärd.** En ny typ `zonvakt_port_t` skapades och port 7539 märktes med `semanage port -a -t zonvakt_port_t -p tcp 7539`. Raden `corenet_tcp_bind_generic_port(zonvakt_t)` togs bort ur policyn.

**Motivering.** Den raden tillät bindning till tre porttyper, alltså i praktiken vilken oreserverad port som helst. En komprometterad tjänst hade kunnat öppna en godtycklig port, till exempel som bakdörr.

**Effekt.** `zonvakt_t` kan bara binda port 7539, och en annan port nekas.

**Verifiering.**

- `semanage port -l | grep zonvakt_port_t` kontrollerades.
- `sesearch -A -s zonvakt_t -c tcp_socket -p name_bind` visar bara `zonvakt_port_t`.
- `seinfo --portcon=7539` visar `zonvakt_port_t`.
- Negativt test: port 7540 sattes i `zonvakt.conf`. Starten gav `failed` och en AVC för `name_bind` (`unreserved_port_t`, `permissive=0`). Inställningen återställdes sedan.
- Skriptets kontroll 5 rapporterar GÄLLER för både portmärkningen och att `zonvakt_t` bara får binda `zonvakt_port_t`.

**Reservation.** Utdata från kommandona och från testet finns inte i underlaget. Skriptets rad bekräftar dock resultatet.

## 5. Verifieringsskriptets utdata

Skriptet `verifiera-hardning` rapporterar GÄLLER för följande fem kontroller:

1. SELinux är i läge Enforcing nu (`getenforce`).
2. `/etc/selinux/config` sätter `SELINUX=enforcing`.
3. `zonvakt` kör i domänen `zonvakt_t` (`ps -eZ`).
4. `nftables` är aktiverad vid boot och kör (`systemctl`).
5. Port 7539 är märkt `zonvakt_port_t` och `zonvakt_t` får bara binda den (`seinfo`, `sesearch`).

## 6. Öppna punkter inför inlämning

- Bifoga utdata från `ausearch` och Wazuh-händelsen (regel 100200) för åtgärd 2.
- Visa att domänen `zonvakt_t` inte är permissiv och att loggskrivning till `zonvakt_log_t` fungerar.
- Utöka skriptet, eller bifoga utdata från `nft -a list chain inet zon in`, så att policy drop i kedjan `in` verifieras (åtgärd 3).
- Visa att de behövda flödena (port 80 och 7539) fungerar efter omstart.
- `sestatus` visar `Policy deny_unknown status: allowed`. Underlaget tar inte upp detta i någon åtgärd.
`````

## 4. Rättelselogg

Hela loggen med 17 påståenden (P1–P17) finns i `rattelselogg-elev-39.md`. 14 påståenden stämde rakt av. Tre fick en rättelse eller precisering:

| Påst. nr | Påstående (kort) | Dom | Kommando som avgjorde | Faktiskt utfall | Feltyp | Ändrat till |
|---|---|---|---|---|---|---|
| P4 | Åtgärd 1: servern startades om och skriptet kördes efteråt | stämmer, med begränsning | `last reboot`, `uptime -s` | Senaste omstart 19 nov 05:27. Porttypen (22/11) och kontroll 5 (23/11) tillkom efter den och är inte provade över en omstart. | ofullständigt | Omstarten gäller kontroll 1–4. Kontroll 5 och porttypen har inte provats över omstart. |
| P8 | Utan egen domän kör tjänsten som unconfined_service_t | ogrundat | – (inget kommando visar det på min server) | Ingen utdata. Påståendet kommer från studieguiden (vecka 7), inte från min server. | ogrundat | Utan egen domän kör tjänsten enligt studieguiden som unconfined_service_t (ej provat på min server). |
| P11 | Flödena httpd:80 och zonvakt:7539 är tillåtna och fungerar | stämmer, med begränsning | `sudo nft list chain inet zon in \| grep -E 'dport (80\|7539)'`, `systemctl is-active httpd`, `ss -tln \| grep -E ':(80\|7539) '` | Reglerna finns, men bara från ctf-adressen (`<IP>`). `httpd` är `active` och port 80 och 7539 lyssnar. | för vidsträckt | Flödena är tillåtna från ctf-servern till httpd:80 och zonvakt:7539. |

### Var AI hade fel under veckan, och hur det upptäcktes

| # | Vad AI:n hade fel i | Hur det upptäcktes |
|---|---|---|
| 1 | Claudes tabell för U7.3 bedömde `corecmd_exec_bin`, `corecmd_mmap_bin_files` och `auth_read_passwd_file` på ett sätt som inte stämde med lösningsförslaget (Ö9). | Jämförelse mot lösningsförslaget. |
| 2 | Claude föreslog `semanage port -l` i verifieringsskriptet, men kommandot kräver root. | Jag körde det utan `sudo`: `SELinux policy is not managed`. Ersattes med `seinfo --portcon`. |
| 3 | Claude lät `semodule -i` stå efter `make` utan `&&`, så den gamla modulen installerades efter ett misslyckat bygge. | `semanage` svarade `must be a port type` och `sesearch` visade fortfarande `unreserved_port_t`. Rättades med `chown` och kedjade kommandon. |
| 4 | `audit2allow` och Claude missade en nekning som var dold av `dontaudit` (`tcp_socket { read write }`). | Tjänsten slutade fungera när `permissive` togs bort. Dolda regler visades med `semodule -DB`. |
| 5 | Claude sa i U6.5 att `bytes` var konstanta, utifrån en enda rad. | Fler rader visade att värdena varierade. |
| 6 | Ö6: Claudes fråga var otydligt formulerad. | Jag frågade hur den skulle läsas. |
| 7 | Claude bedömde att P7 blandade ihop kursens övningsfil och min server, innan beläggen var körda. | `ausearch` på servern visade en egen händelse (18 nov 10:27:55 UTC) som motsvarar Wazuh-träffen 11:27:58 svensk tid. Domen ändrades till *stämmer*. |

## 5. Utelämningslista

Det här matades medvetet **inte** in i AI-utkastet i workshopen, och varför:

| Utelämnat | Skäl |
|---|---|
| Lösenord | Personliga, delas aldrig med en AI-tjänst. |
| CTF-flaggor | Hör inte hemma i en rapport och ska inte delas. |
| Icke-offentliga IP-adresser och nftables-reglerna (`zon.nft`, `nft`-utdata) | Interna adresser, ska inte matas in. |
| Policyfilerna (`.te`, `.fc`, `.if`) | Behövdes inte för utkastet. De finns i repot. |
| Hela `semodule -l` och hela `ps -eZ` | Långa och inte relevanta, endast utdrag matades in. |
| Loggutdata (`ausearch`, Wazuh) | Underlaget var begränsat till det listan krävde. Utkastet markerade själv att de saknades. |

**Observera:** i den löpande handledningen under veckan (Claude, i en annan chatt än utkastet) förekom interna adresser (till exempel jump-servern) och CTF-flaggor i samtalet. Det gäller alltså inte den ordagranna beskrivningen av utkastets underlag ovan, men jag redovisar det här.

## Lärdomar om AI-stödet

- Utkastet höll sig till underlaget och markerade själv vad som saknade belägg ("Reservation") i stället för att gissa.
- Utkastet återgav påståenden från min egen lista som sanna. Utan ett kommando bredvid dem är de ogrundade, och det var min lista, inte utkastet, som saknade belägg.
- Mitt första försök misslyckades eftersom jag skickade mallen med platshållare. AI:n sa ifrån i stället för att hitta på.
- Beläggen ska komma från servern, inte från AI-svaret, och inte heller från en tidigare körning.
