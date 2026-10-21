# Incidentgranskning: komprometterade eMudhra-certifikat

*2026-10-21 · Författare: Alexander Schön · Utomstående granskning baserad på öppna källor · Händelsen pågår*

## Vad hände

Den 17 oktober 2026 rapporterades att de privata nycklarna till åtta mellanliggande CA-certifikat hos den indiska certifikatutfärdaren eMudhra går att återskapa direkt ur certifikaten. Med dem kan en angripare utfärda HTTPS-certifikat för valfri domän som webbläsare och operativsystem litar på. Sex av certifikaten gäller TLS och två kodsignering.

## Tidslinje (2026, UTC)

- **Apr 2025:** nycklarna skapas felaktigt.
- **17 okt:** Chrome får rapporten 18:05 och blockerar 19:45. Mozilla får den 20:06 och har blockerat 22:59.
- **19 okt:** Apple blockerar. Go släpper 1.27.3 och 1.26.10.
- **20 okt:** Microsoft blockerar. Mozillas Dennis Jackson publicerar en incidentsida.

## Orsak

Enligt Jackson genererades nycklarna felaktigt. Enligt BleepingComputer var värdena under 256, så 256 försök räcker för att hitta en nyckel. Förklaringen att ett skriptfel gav en andra, svagare nyckeluppsättning är eMudhras egen och inte oberoende kontrollerad.

## Påverkan

Ingen källa visar missbruk. Enligt Chrome hade inget TLS-certifikat under de sex TLS-nycklarna loggats i Certificate Transparency före blockeringen. Klienter utan CT-krav eller revokeringskontroll, till exempel Go-program och macOS inbyggda `/usr/bin/curl`, kan ha varit exponerade tills de uppdaterats.

## Vår bedömning (tolkning)

- **Fungerade:** snabb och samordnad blockering, under två timmar till den första. Beviset kunde verifieras med OpenSSL.
- **Brast:** svag nyckelgenerering upptäcktes inte på ca 18 månader. Skyddet hänger på att klienten kontrollerar revokering. eMudhra hade den 20 oktober inte svarat offentligt, och uppgifterna om omfattning går isär (över 60 miljoner utfärdade certifikat mot att mellancertifikaten aldrig användes i produktion).
- **Zero Trust:** implicit tillit till en certifikatkedja räckte för att förfalska identitet. Tillit måste kunna dras tillbaka dynamiskt, och klienter måste kontrollera den löpande.

## Egen kontroll

**Ubuntu 26.04.1 LTS:** `ca-certificates` 20260816ubuntu0.26.04.1, den version Jackson listar som rättad. `curl` mot testadressen gav fel 60 (klienten litar inte på kedjan), `curl` mot example.com gav HTTP/2 200.

**Labbserver, Rocky Linux 9.8:** `ca-certificates` 2025.2.80 och Rocky saknas i Jacksons statustabell. `curl` mot testadressen lyckades, alltså litade servern på de komprometterade certifikaten. `dnf check-update ca-certificates` visade ingen rättning (21 okt 07:07 UTC). Vi spärrade rötterna Comodo AAA Services root och emSign ECC Root CA – G3 enligt kursens guide (`blocklist` + `update-ca-trust`). Efteråt står båda som `distrusted` och `curl` mot testadressen ger fel 60, medan example.com ger HTTP/2 200. Antalet certifikat i TLS-bundlen sjönk från 146 till 145, inte med två som väntat. Skillnaden är inte utredd. Spärren ligger i `/etc/` och består efter paketuppdateringar. Ångra genom att ta bort de två `.pem`-filerna ur `blocklist` och köra `update-ca-trust`.

## Källor

- [Dennis Jacksons incidentsida](https://emudhra.we-have-a-problem.com/) (primärkälla)
- [BleepingComputer](https://www.bleepingcomputer.com/news/security/mozilla-google-apple-block-compromised-emudhra-ca-certificates/)
- [The Register](https://www.theregister.com/security/2026/10/20/indian-certification-authority-emudhras-private-keys-compromised-one-of-them-was-the-number-4/5304312)
- [Dark Reading](https://www.darkreading.com/cyber-risk/tech-giants-emudhra-certificates-compromised-keys)
- [Cyber Kendra](https://www.cyberkendra.com/2026/10/go-blocks-compromised-emudhra-ca-keys-in-emergency-update.html)
- Ars Technica kunde inte öppnas och är inte använd som källa.

*AI-stöd: Claude användes för att söka, läsa och sammanfatta källorna och för utkastet. Kontrollen på den egna maskinen kördes av Alexander Schön.*
