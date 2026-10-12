# Våra servrar – Zonuppdraget

Här ska vi skriva vilka servrar som finns i vår labbmiljö, vad de har för IP-adress och vad de används till.
# Tillgångsförteckning – Zonuppdraget

Här listar vi servrarna i labbmiljön, deras IP-adresser och vad de används till.

| Server | DNS-namn | IP-adress | Vad den gör |
|---|---|---|---|
| Jump-servern | jump.sec.chas-lab.dev | 172.20.1.2 | Används för att ansluta till andra servrar. |
| Loggservern | log.sec.chas-lab.dev | 172.20.1.58 | Samlar loggar och har en testtjänst via HTTP. |
| CTF-servern | ctf.sec.chas-lab.dev | 172.20.1.10 | Tar emot flaggor från CTF-uppgifter. |
| Vår server | elev-NN.sec.chas-lab.dev | 172.20.1.(100+NN) | Rocky Linux 9, webbserver och verktyg. |

## Kommentar

Uppgifterna kommer från gruppens adressplan. Vi behöver ta reda på vårt eget servernummer (NN) för att kunna skriva in rätt IP-adress för vår server.
## Vad behöver vi ta reda på?

- Vilka servrar finns?
- Vilka IP-adresser har servrarna?
- Vad gör servrarna?
- Vilka tjänster använder de?

Vi fyller i informationen när vi har undersökt labbmiljön tillsammans i gruppen.