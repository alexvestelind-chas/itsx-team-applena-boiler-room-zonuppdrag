# Vecka 6 – SOC-miljö och detektion
## Team 8 – Boiler Room

**Ansvarig:** elev-07  
**Ämne:** Detektioner, larmhantering och SSH Brute Force  
**Vecka:** 6

---

## 1. Syfte

Syftet med arbetet är att upptäcka misstänkta säkerhetshändelser med hjälp av Wazuh, undersöka larmen och dokumentera hur de ska hanteras.

Gruppens mål enligt milstolpe 6 är minst fyra detektioner i Wazuh. Varje detektion ska ha en mätning av träffar i normaltrafik, en bedömning av risken för falsklarm och en tillhörande playbook.

## 2. Resultat från övningarna

### U6.1 – Hitta händelsen

Totalt fanns 201 larm i exportfilen.

| Regel-ID | Antal |
|---|---:|
| 550 | 61 |
| 5715 | 50 |
| 5402 | 46 |
| 2501 | 43 |
| 100600 | 1 |
| **Totalt** | **201** |

Regel 100600 hade endast en träff och var den avvikande händelsen.

### U6.2 – SSH Brute Force

- Monitor: `brute-force-ssh`
- Antal misslyckade inloggningar: 12
- Resultat: Monitorn larmade.

Testet visade att monitorn reagerade på de upprepade misslyckade inloggningarna.

### U6.3 – Brusmätning

- Regel-ID: 5760
- Antal larm: 27
- Datum: 11 november
- Tidsperiod: 08:05–08:37

### U6.4 – Triage

Observerade händelser:

- 31 från `127.0.0.1`
- 4 från `10.40.2.136`
- Resterande händelser hade en träff vardera.

Totalt angavs 36 larm i beskrivningen.

### U6.5 – Egen detektion

**Status: Godkänd**

Resultat:

- 72 träffar i en sökning, huvudsakligen brandväggsbrus.
- 195 träffar efter filtrering.
- Monitorn hittade 51 `syncagent`-poster på 10 minuter.

Övningen är godkänd.

---

## 3. Playbook – SSH Brute Force

### Vad är SSH Brute Force?

SSH Brute Force innebär att någon försöker logga in på en server upprepade gånger genom att testa olika lösenord.

Syftet kan vara att få obehörig åtkomst till servern.

### Detektion

I U6.2 användes Wazuh-monitorn `brute-force-ssh`.

Testet gav 12 misslyckade inloggningar och monitorn larmade.

---

## 4. Fas 1 – Förberedelse

1. Kontrollera att Wazuh fungerar.
2. Öppna det aktuella larmet.
3. Anteckna datum och tid.
4. Identifiera vilken server som berörs.
5. Anteckna källans IP-adress om den finns i loggen.

## 5. Fas 2 – Identifiering och analys

1. Kontrollera antalet misslyckade inloggningar.
2. Undersök vilken IP-adress försöken kommer från.
3. Kontrollera vilket användarkonto som har berörts.
4. Undersök om någon inloggning lyckades.
5. Dokumentera observationerna.

Ett larm betyder inte automatiskt att servern har blivit hackad. Händelsen måste undersökas innan en slutsats dras.

## 6. Fas 3 – Begränsning och åtgärd

Om undersökningen visar ett misstänkt angrepp:

1. Informera ansvarig person i teamet.
2. Följ teamets godkända rutin för att begränsa åtkomsten från den misstänkta källan.
3. Kontrollera om berörda användarkonton behöver skyddas.
4. Dokumentera vilka åtgärder som genomförts.

Ändringar på servern ska godkännas av ansvarig person.

## 7. Fas 4 – Återställning och uppföljning

1. Kontrollera att servern fungerar normalt.
2. Kontrollera om nya misstänkta inloggningsförsök förekommer.
3. Kontrollera att Wazuh fortfarande tar emot loggar.
4. Dokumentera resultatet efter åtgärderna.
5. Informera teamet om resultatet.

---

## 8. Risk för falsklarm

Ett falsklarm kan uppstå om en riktig användare skriver fel lösenord flera gånger.

Därför behöver larmet undersökas innan man bestämmer att det är ett angrepp.

## 9. Dokumentation av en händelse

Vid varje nytt SSH-larm ska följande information sparas:

- Datum och tid:
- Berörd server:
- Källans IP-adress:
- Användarkonto:
- Antal misslyckade inloggningar:
- Lyckades någon logga in?
-