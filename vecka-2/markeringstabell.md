| Plats i zonförslaget | Kontrollfråga som faller | Var den implicita tilliten ligger | Föreslagen ändring | Vad ändringen kostar i drift |
|---|---|---|---|---|
| Klientzon → DMZ (tcp/443) | Princip 2/4: väger beslutet in mer än källadress? | Webbservern litar på att källan tillhör klientzonens subnät – ingen användarautentisering på nätverksnivå | Applikationens egen inloggning + kort sessionslivslängd | Extra inloggningssteg för användare, viss ökad latens, hantering av token-förnyelse |
| DMZ → Applikationszon (tcp/8080) | Princip 2/4: ömsesidig autentisering? | Appservern litar på att anropet kommer från rätt IP (webbservern), ingen ömsesidig verifiering | mTLS mellan webb- och appserver | Certifikathantering/rotation, ökad felsökningskomplexitet, viss prestandakostnad vid TLS-handskakning |
| Applikationszon → Datazon (tcp/5432) | Princip 3/4: per-session, dynamisk policy? | Databasen litar på käll-IP, inget tjänstekonto med kort livslängd | Tjänstekonto med kortlivade databaslösenord/token | Secrets-hantering och rotation, driftrisk om rotationen missköts |
| Hanteringszon → alla zoner (tcp/22) | Princip 3/6: omprövas förtroendet per session? | Alla servrar litar på att inloggning från adminstationens IP är legitim, ingen MFA/token | Kräv MFA/engångstoken per inloggning (som U3.2) | Extra steg för administratörer, beroende av att policymotorn fungerar (risk: enskild felpunkt) |
| Alla zoner → Hanteringszon (tcp/6514, logg) | Princip 4/7: vet vi vem som faktiskt skickar? | Loggservern litar på att allt som kommer på port 6514 är en legitim tjänst | Klientcertifikat per tjänst mot loggservern | Certifikatutfärdande/förnyelse per tjänst, ökad konfigurationsbörda |


