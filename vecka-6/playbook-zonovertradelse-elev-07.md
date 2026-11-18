# Playbook – Zonöverträdelse med SELinux
**Team:** 8 – Boiler Room  
**Vecka:** 7 – Härdning med Mandatory Access Control  
**Ansvarig:** elev-07  
**Tjänst:** Zonvakt  
**Port:** 7507  
**Status:** Utkast för granskning

---

## 1. Syfte

Syftet med denna playbook är att beskriva hur teamet ska upptäcka, undersöka och hantera en zonöverträdelse med hjälp av SELinux.

En zonöverträdelse inträffar i detta scenario när tjänsten Zonvakt försöker utföra en åtgärd som den inte har tillstånd till enligt sin säkerhetspolicy.

Målet är att förhindra att tjänsten utför otillåtna åtgärder och samtidigt se till att tjänsten fortfarande fungerar som den ska.

## 2. Bakgrund – Zonvakten

Zonvakt är en liten Python-tjänst som lyssnar på en TCP-port, svarar med en statusrad och skriver information till sin logg.

För elev-07 används port 7507.

Tjänsten har två olika testfall:

- `hej` – ett normalt kommando som ska fungera.
- `overtrad` – framkallar ett avsiktligt säkerhetstest där tjänsten försöker skriva till `/etc/zonvakt-otillaten`.

Tjänsten ska inte ha rätt att skriva till den skyddade platsen `/etc`.

SELinux ska därför stoppa den otillåtna åtgärden.

## 3. Detektion – hur upptäcks överträdelsen?

Överträdelsen upptäcks genom en AVC-post i SELinux-loggen.

AVC betyder Access Vector Cache. Loggen visar när SELinux nekar en åtkomst.

Följande kommandon används för att undersöka händelsen:

```bash
sudo ausearch -m avc -c zonvakt -i
```

För att undersöka vad som nekades kan följande kommando användas:

```bash
sudo ausearch -m avc -c zonvakt --raw | audit2allow -R
```

För att kontrollera vilken säkerhetsdomän tjänsten kör i:

```bash
ps -eZ | grep zonvakt
```

Tjänsten ska köras i domänen `zonvakt_t`.

## 4. Fas 1 – Förberedelse

1. Kontrollera att SELinux är aktiverat.
2. Kontrollera att SELinux kör i läget Enforcing.
3. Kontrollera att Zonvakt är installerad och startad.
4. Kontrollera att tjänsten använder rätt port, 7507.
5. Kontrollera att den egna SELinux-policyn är installerad.

Kommandon:

```bash
getenforce
sestatus
sudo systemctl status zonvakt
sudo semodule -l | grep zonvakt
```

## 5. Fas 2 – Identifiering och analys

När en AVC-överträdelse upptäcks ska följande undersökas:

1. Vilken process orsakade händelsen?
2. Vilken fil eller resurs försökte processen komma åt?
3. Vilken åtgärd nekades?
4. Vilken SELinux-domän körde processen i?
5. Var åtgärden tillåten eller stoppades den?

Kontrollera även loggen:

```bash
sudo ausearch -m avc -c zonvakt -i
```

Det är viktigt att skilja mellan en överträdelse som bara loggas och en överträdelse som faktiskt stoppas.

I Enforcing-läget ska en otillåten åtgärd stoppas.

## 6. Fas 3 – Begränsning och åtgärd

Om Zonvakt försöker skriva till `/etc/zonvakt-otillaten` ska SELinux neka åtkomsten.

Gör följande:

1. Kontrollera AVC-loggen.
2. Kontrollera att nekandet gäller den otillåtna skrivningen.
3. Kontrollera att tjänsten kör i `zonvakt_t`.
4. Granska den aktuella SELinux-policyn.
5. Lägg inte till behörigheter som tjänsten inte behöver.

Använd inte `audit2allow` automatiskt för att tillåta alla förslag. Varje föreslagen behörighet måste granskas.

Om tjänsten inte behöver skriva till `/etc` ska den behörigheten inte läggas till i policyn.

**Viktigt:** SELinux ska inte stängas av eller sättas i permissive-läge för hela systemet för att bli av med överträdelsen.

## 7. Fas 4 – Återställning och verifiering

Efter att policyn har granskats ska tjänsten testas igen.

Testa normal funktion:

```bash
echo hej | nc -w2 127.0.0.1 7507
```

Förväntat resultat:

```text
zonvakt elev-07 ok
```

Testa sedan den otillåtna åtgärden:

```bash
echo overtrad | nc -w2 127.0.0.1 7507
```

Förväntat resultat är att SELinux stoppar den otillåtna skrivningen.

Kontrollera efter testet:

```bash
getenforce
ps -eZ | grep zonvakt
sudo ausearch -m avc -c zonvakt -i
```

Följande ska gälla:

- SELinux ska vara i läget Enforcing.
- Zonvakt ska köras i domänen `zonvakt_t`.
- Policyn ska vara aktiv och domänen ska inte vara permissiv.
- Normal kommunikation ska fungera.
- Den otillåtna skrivningen ska nekas.

## 8. Risk för falsklarm

En AVC-post betyder inte automatiskt att någon angriper servern.

Det kan också vara en tjänst som försöker utföra en åtgärd som inte ingår i dess tillåtna funktion.

Därför måste händelsen undersökas innan man bestämmer vilken åtgärd som ska göras.

Det är viktigt att kontrollera vilken process, fil och operation som berörs.

## 9. Dokumentation av händelsen

Följande information ska dokumenteras vid en zonöverträdelse:

- Datum och tid:
- Berörd tjänst:
- Processens SELinux-domän:
- Fil eller resurs som berördes:
- Vilken åtgärd nekades:
- AVC-resultat:
- Var åtgärden stoppades eller endast loggades:
- Vilken åtgärd genomfördes:
- Resultat efter verifiering:

## 10. Slutsats

SELinux används för att skydda servern genom att begränsa vad en tjänst får göra.

I Zonvakt-scenariot ska normal kommunikation tillåtas medan en otillåten skrivning till `/etc` stoppas.

Genom att undersöka AVC-loggen kan teamet se vad som nekades och avgöra om SELinux-policyn eller tjänstens beteende behöver justeras.

**Status:** Utkast. Resultatet ska verifieras på servern och playbooken ska testas av en annan teammedlem.