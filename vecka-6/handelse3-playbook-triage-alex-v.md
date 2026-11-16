# Händelse 3: Root-kommando (auditd) – Playbook och triagelogg

## Playbook: Root-kommando upptäckt

### Fas 1 – Preparation (förberedelse)
- [ ] `audispd-plugins` installerat, `active = yes` satt i `/etc/audit/plugins.d/syslog.conf`
- [ ] Auditd-regeln aktiv: `sudo auditctl -l` visar `-F key=root_command`
- [ ] Alerting-monitor byggd och verifierad (larmar vid testkommando)
- [ ] Känt, förväntat mönster dokumenterat: ctf-servrarens egna kontroller loggar in som root med nyckel från 172.20.1.10 (nämnt explicit i studieguiden, korrelerar till regel 40112 – "Multiple authentication failures followed by a success")

### Fas 2 – Detection & Analysis (upptäckt och analys)
När larmet går, samla in – **observation, inte tolkning**:
1. `data.auid` – vem var ursprungligen inloggad?
2. `data.exe` / `data.command` – vilket kommando kördes?
3. Källadress för den inloggning som föregick kommandot (sök `predecoder.program_name: sshd*` runt samma tid)
4. Tidskorrelation – ligger det nära en nyligen lyckad inloggning (`Accepted publickey`/`Accepted password`)?
5. Serienummer (`msg=audit(TID:SERIENUMMER)`) – binder ihop EXECVE och SYSCALL som samma händelse

### Fas 3 – Containment, Eradication & Recovery (begränsning, utrotning, återställning)
**Om larmet bedöms verkligt (eskalerat, se kriterier nedan):**
- [ ] Blockera källadressen i `zon.nft` (`chain in`, `drop` före ev. befintlig accept-regel)
- [ ] Avsluta aktiva sessioner från den adressen: `who`, `sudo pkill -KILL -u <konto>` vid behov
- [ ] Kontrollera persistens: nya `~/.ssh/authorized_keys`-rader, nya cron-jobb, nya användarkonton (`last`, `cat /etc/passwd`)
- [ ] Rotera ev. komprometterade nycklar/lösenord

**Om larmet avfärdas** (se kriterier nedan): ingen åtgärd i den här fasen, gå till Fas 4.

### Fas 4 – Post-Incident Activity (efterarbete)
- [ ] Logga beslutet i triageloggen (observation + slutsats, se mall nedan)
- [ ] Om falsklarm: överväg att justera monitorns villkor (t.ex. undanta känt källmönster) – men ändra aldrig regeln förrän händelsen är bekräftad, enligt kursens egen varning
- [ ] Om verkligt larm: uppdatera lessons-learned, dela med teamet

---

## Eskaleringskriterier (när larmet ska tas vidare)

Eskalera om **minst ett** gäller:
- `data.auid` pekar på ett konto som **inte** är en känd administratör eller den dokumenterade ctf-källan
- Källadressen för föregående inloggning är **okänd** (inte jump 172.20.1.2, inte ctf 172.20.1.10, inte en känd lagkamrat)
- Kommandot (`data.exe`/`data.command`) matchar ett misstänkt mönster: nya användare skapas, `authorized_keys` ändras, nedladdning/körning av okänd binär, ändring i sudoers
- Flera larm från **olika** källor inom kort tid (tyder på bredare aktivitet, inte ett enstaka administrativt kommando)

## Avfärdandekriterier (när larmet kan stängas utan åtgärd)

Avfärda **bara** med ett **granskbart** skäl (ett konkret belägg – inte en känsla):
- `data.auid` matchar en känd administratör, **och** tidpunkten matchar ett faktiskt planerat arbete (styrkt t.ex. av egen anteckning/kalender)
- Källan är **ctf-servern (172.20.1.10)**, föregående rad är `Accepted publickey`, kontot är `root` – det dokumenterade, förväntade mönstret för kursens egna kontroller
- Kommandot är ett **känt, rutinmässigt** administrativt kommando (t.ex. `whoami`, en paketuppdatering ni själva precis körde) och ni kan peka på **er egen, samtidiga terminalsession** som belägg

**Exempel på INTE ett giltigt avfärdande:** "Det var nog bara jag" – utan att faktiskt kontrollera tidsstämpel, `auid` eller källadress mot vad ni faktiskt gjorde.

---

## Triagelogg – mall (en post per larm)

| Larm-id/tid | Observation (enbart fakta) | Slutsats (bedömning + beslut) |
|---|---|---|
| *[larm-tid]* | *[t.ex.: auid=student, uid=0, exe=/usr/bin/whoami, föregående rad: Accepted publickey för root från 172.20.1.10 kl samma minut]* | *[t.ex.: Matchar det dokumenterade ctf-mönstret (regel 40112). Avfärdas – belägg: källa + kontonamn + tidskorrelation stämmer exakt med studieguidens beskrivna rutin.]* |
| *[larm-tid]* | *[fyll i från ett verkligt larm ni får idag]* | *[eskalera eller avfärda, med belägg]* |




