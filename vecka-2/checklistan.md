| Princip | Kontrollfr├Ñga | bevis f├Âr ja | vem eller vad i v├Ñr milj├Â kan visa beviset? |
|---------|---------------|--------------|-------------------------------------------|
| Allt ├ñr en resurs | ├är tillg├Ñngarna nedskrivna i tillg├Ñngsf├Ârteckningen? | Den ├ñr nedskriven i tillg├Ñngsf├Ârteckningen | Tillg├Ñngsf├Ârteckningen i repot |
| All kommunikation skyddas, oavsett n├ñtverksplats | Hanteras intern trafik p├Ñ samma s├ñtt som extern trafik i milj├Ân om man t├ñnker p├Ñ s├ñkerheten? | Brandv├ñggsregler inom n├ñtverket | klassificering-och-zoner dokumentet |
| ├àtkomst per session | Pr├Âvas en anslutning varje g├Ñng? | T.ex U3.2 d├ñr vi anv├ñnde en tokenbaserad autentisering | Om man testar att ansluta utan token s├Ñ blir man nekad |
| Dynamisk policy | Pr├Âvas andra variabler f├Ârutom ip och n├ñt n├ñr en anslutning pr├Âvas? | Eventuell policyregel som man har | H├ñndelser som flaggas som avvikande i loggarna |
| ├ûvervaka tillst├Ñnd hos alla tillg├Ñngar | Kontrolleras tillg├Ñngen efter s├Ñrbarheter innan ├Ñtkomst ges? | Tillg├Ñngen ska vara kontrollerad innan den ansluter | - |
| Dynamisk autetisering och auktorisering, strikt i f├Ârv├ñg | Ompr├Âvas anslutningen under sessionen och inte bara vid inloggning? | sessionen ├ñr tidsbegr├ñnad och kr├ñver inloggning p├Ñ nytt | time to live p├Ñ token eller session |
| Samla information f├Âr att f├Ârb├ñttra | Loggas relevanta h├ñndelser idag? | Loggar skickas till loggservern med information som beh├Âvs | Loggarna p├Ñ loggservern |
