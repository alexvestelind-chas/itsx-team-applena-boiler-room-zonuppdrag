# eMudhra-incidenten – konsekvenser för vår labbmiljö

*Rapport · ITSX26 Kurs 2 · 21 oktober 2026*
*Skapad av Claude AI*

## Sammanfattning

Den 20 oktober 2026 offentliggjorde Mozilla att privata nycklar till åtta mellanliggande CA-certifikat utfärdade av den indiska certifikatutfärdaren eMudhra (under roten emSign) hade komprometterats. Nycklarna visade sig vara kritiskt svaga – i vissa fall bestod de effektivt av endast ett fåtal bitar – vilket gör dem triviala att gissa eller återskapa, snarare än korrekt slumpmässigt genererade. Felet spåras till en nyckelgenereringsceremoni (KGC) eMudhra genomförde i april 2025.

Sex av de åtta certifikaten användes för att utfärda TLS-certifikat till webbplatser, och två användes för kodsignering (litade av Microsoft). Google, Mozilla, Apple, Microsoft, Cloudflare och Go-projektet blockerade samtliga åtta certifikat inom ungefär tre dagar efter upptäckten.

## Teknisk bakgrund

Hela webbens TLS-förtroendemodell (WebPKI) bygger på en kedja av tillit: en webbläsare litar på ett fåtal rotcertifikat, som i sin tur signerar mellanliggande certifikat, som i sin tur signerar certifikat för enskilda webbplatser. Om en privat nyckel någonstans i den kedjan är svag eller läckt kan en angripare utfärda falska, men till synes giltiga, certifikat för vilken domän som helst – och därmed utge sig för en webbplats eller tjänst för att avlyssna eller manipulera trafik (man-in-the-middle).

## Tidslinje

| Datum | Händelse |
| --- | --- |
| 17 okt, ca 17:00–20:00 UTC | Cloudflare och Mozilla får kännedom om de komprometterade nycklarna |
| 17 okt | Mozilla blockerar certifikaten i Firefox via OneCRL inom cirka 3 timmar |
| 18–19 okt | eMudhra återkallar certifikaten; Apple och Sectigo blockerar/återkallar |
| 20 okt | Microsoft blockerar certifikaten; Mozilla släpper Firefox 158.0.1 och uppdaterar NSS; Go släpper en nödversion som avvisar certifikatkedjor med de komprometterade nycklarna |

## Relevans för vår labbmiljö

Vår labbserver (elev-NN, Rocky Linux 9) är inte direkt måltavla, men är inte heller helt oberörd:

- **Utgående HTTPS-anrop.** Allt som kör `curl`, `dnf`/paketuppdateringar eller egna script mot externa HTTPS-tjänster litar på samma systemomfattande trust store (`ca-certificates`/NSS) som en webbläsare. Om en angripare skulle presentera ett förfalskat certifikat signerat med en av de komprometterade nycklarna, skulle servern i teorin acceptera det som giltigt.
- **Egna TLS-tjänster.** I en skarp miljö (inte vår labbmiljö, som använder interna/självsignerade certifikat) skulle en organisation vars eget certifikat utfärdats via eMudhra/emSign riskera att bli blockerad av uppdaterade klienter – ett driftsavbrott snarare än ett intrång.
- **Automatiserade pipelines.** Skript och automatiserade processer som validerar TLS utan mänsklig granskning är mer sårbara än en människa som surfar, eftersom en webbläsare ofta varnar synligt för ett ogiltigt certifikat medan ett skript bara fortsätter.

### Kontroll på vår server

En kontroll av den lokala trust-storen visade att roten **emSign ECC Root CA - G3** – den rot som signerade de komprometterade mellanliggande certifikaten – fortfarande fanns listad som litad:

```
trust list | grep -i emsign
```

Detta bekräftar att Rockys paketrepo vid kontrolltillfället ännu inte hunnit distribuera den uppdaterade NSS-trustlistan som Mozilla publicerade samma dag.

## Rekommenderade åtgärder

1. Uppdatera trust-store och NSS: `sudo dnf update nss ca-certificates -y`
2. Verifiera att roten försvunnit eller flyttats till blocklistan: `trust list --filter=blacklist`
3. Om uppdateringen inte innehåller fixen ännu: lägga till roten manuellt i den lokala blocklistan (`/etc/pki/ca-trust/source/blacklist/`) och köra `update-ca-trust`
4. Uppdatera eventuella installerade webbläsare och Go-verktygskedjan på servern
5. Granska om någon egen tjänst eller något beroende refererar till ett certifikat utfärdat via eMudhra/emSign

## Koppling till kursens Zero Trust-principer

Incidenten är ett konkret exempel på varför **implicit förtroende** är farligt (NIST SP 800-207, princip 2: all kommunikation ska säkras oavsett källa, inte bara litas på för att den kommer från en "känd" part). Ett certifikat som är tekniskt giltigt enligt kedjan visade sig ändå vara opålitligt, eftersom själva kedjans svagaste länk – nyckelgenereringen hos en enskild utfärdare – brast. Samma princip som gjorde att klienten i U3.1 kunde nå databasen bara för att den var "innanför" det interna nätet gäller här: ett certifikat ska inte litas på enbart för att det formellt verifieras mot en känd rot, utan kräver kontinuerlig övervakning och snabb reaktionsförmåga när förtroendet visar sig vara felplacerat.


