# Härdningsrapport – Linux-server med SELinux (rättad)

Alexander Schön · ITSX26 Kurs 2 · vecka 7 · Workshop 1, del 1 · elev-39 (applikationszonen)

*Rättad version av AI-utkastet (`rasvar-forsok2.md`, som är oredigerat). Alla påståenden har dömts mot servern, se avsnitt 7 och `rattelselogg-elev-39.md`. Ändringar är markerade med [rättat: Px]. Belägg insamlade 2026-11-27.*

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

**Reservation.** Servern startades om senast 2026-11-19 05:27 (`last reboot`), och omstarten med skriptkörningen efteråt gäller kontroll 1–4. Porttypen (installerad 22/11) och kontroll 5 (tillagd 23/11) har inte provats över en omstart. [rättat: P4]

### Åtgärd 2 – `zonvakt` körs under en egen tvingande policy

**Åtgärd.** Tjänsten har en egen policymodul `zonvakt` (`.te` och `.fc`) med domänen `zonvakt_t` och den egna loggtypen `zonvakt_log_t`. Modulen innehåller inte `permissive zonvakt_t;`. Förslagen från audit2allow granskades, och de för breda eller onödiga raderna (till exempel `files_manage_etc_files`) togs inte med.

**Motivering.** Utan egen domän kör tjänsten enligt studieguiden (vecka 7) som `unconfined_service_t`, alltså utan MAC. Det är inte provat på min server. [rättat: P8, ogrundat] Då stoppas inget om tjänsten tas över eller gör fel, och beteendet att skriva i `/etc` (`overtrad`) hade lyckats. En permissiv domän loggar bara.

**Effekt.** `zonvakt` körs i `zonvakt_t`, får skriva sin logg men nekas att skapa filer i `/etc` (typen `etc_t`).

**Verifiering.**

- `ps -eZ | grep zonvakt` visar `zonvakt_t`. Utdraget bekräftar detta (`system_u:system_r:zonvakt_t:s0`, PID 4126).
- `semodule -l` visar modulen `zonvakt`.
- Skriptets kontroll 3 rapporterar GÄLLER för domänen.
- Enligt listan visar `ausearch -i -m avc` nekningen i `etc_t` med `permissive=0`, och samma händelse hittades på loggservern (Wazuh, regel 100200).

**Belägg.** `sudo ausearch -m avc -i -c zonvakt | grep etc_t` visar `{ write }` mot `etc_t` med `permissive=0`: 18 nov 10:27:55 UTC och 22 nov 07:01:35 UTC. Wazuh-träffen 18 nov 11:27:58 svensk tid (regel 100200) motsvarar händelsen 10:27:55 UTC (P7). Att domänen inte är permissiv syns i `permissive=0` och i att `.te` saknar `permissive zonvakt_t;` (P5). Loggskrivningen syns i `/var/log/zonvakt/zonvakt.log` med typen `zonvakt_log_t` (P6).

### Åtgärd 3 – Brandvägg med nftables

**Åtgärd.** Tabellen `inet zon` har kedjan `in` med policy drop, och bara de flöden som behövs tillåts. Reglerna ligger i `/etc/nftables/zon.nft`, som börjar med `flush ruleset`. Tjänsten `nftables` är aktiverad vid boot.

**Motivering.** Utan brandvägg är alla lyssnande portar nåbara. Med policy drop är allt förbjudet utom det som uttryckligen tillåts, så en ny tjänst blir inte exponerad av misstag. Reglerna i en fil gör dem reproducerbara efter omstart.

**Effekt.** Okända inkommande flöden droppas, medan de behövda flödena fungerar fortfarande från ctf-servern (httpd på port 80 och `zonvakt` på port 7539). [rättat: P11]

**Verifiering.**

- `nft -a list chain inet zon in` granskades.
- `nft -c -f /etc/nftables/zon.nft` gjordes som syntaxkontroll.
- `systemctl is-enabled nftables` och `is-active` kontrollerades.
- Enligt listan är detta verifierat efter omstart.
- Skriptets kontroll 4 rapporterar GÄLLER för att `nftables` är aktiverad vid boot och kör.

**Reservation.** Skriptet täcker bara att tjänsten är aktiv vid boot, inte att kedjans policy är drop (markerat ◐ i listan). Policy drop och de tillåtna flödena är därmed verifierade manuellt. Belägg: `sudo nft list chain inet zon in` visar `type filter hook input priority filter; policy drop;` (P9), `zon.nft` börjar med `flush ruleset` och `nftables` är `enabled` och `active` (P10). `httpd` är `active` och port 80 och 7539 lyssnar (P11).

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

**Belägg.** `seinfo --portcon=7539` och `semanage port -l` visar `zonvakt_port_t` på 7539 (P13). `.te` saknar `generic_port`, och `sesearch -A -s zonvakt_t -c tcp_socket -p name_bind` visar bara `zonvakt_port_t` (P14). Testet med port 7540 gav tre AVC (`name_bind`, `unreserved_port_t`, `permissive=0`) 22 nov 07:00:15–16 UTC, eftersom tjänsten startade om sig själv (P15).

## 5. Verifieringsskriptets utdata

Skriptet `verifiera-hardning` rapporterar GÄLLER för följande fem kontroller:

1. SELinux är i läge Enforcing nu (`getenforce`).
2. `/etc/selinux/config` sätter `SELINUX=enforcing`.
3. `zonvakt` kör i domänen `zonvakt_t` (`ps -eZ`).
4. `nftables` är aktiverad vid boot och kör (`systemctl`).
5. Port 7539 är märkt `zonvakt_port_t` och `zonvakt_t` får bara binda den (`seinfo`, `sesearch`).

## 6. Status på de öppna punkterna från utkastet

- Utdata från `ausearch` och Wazuh (åtgärd 2): bifogad, se åtgärd 2 (P7).
- Domänen inte permissiv och loggskrivning fungerar: visat (P5, P6, P7).
- Policy drop i kedjan `in`: visat med `nft list` (P9). Skriptet kontrollerar fortfarande inte det.
- De behövda flödena efter omstart: flödena fungerar (P11), men omstarten gäller bara kontroll 1–4 (P4). Inte provat efter att porttypen lades till.
- `deny_unknown status: allowed`: stämmer (P17). Ingen åtgärd gjord, och det ingår inte i listan.

## 7. Belägg per påstående

Hela tabellen med domar, kommandon och utfall finns i `rattelselogg-elev-39.md` (P1–P17). Sammanfattning: P1–P3, P5–P7, P9, P10, P12–P17 stämmer. P4 och P11 stämmer med precisering. P8 var ogrundat och är omskrivet.
