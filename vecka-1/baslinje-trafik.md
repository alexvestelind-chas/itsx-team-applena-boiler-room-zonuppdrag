## Vad en baslinje är
En baslinje beskriver hur normal trafik ser ut när allt fungerar som det ska.
Den används som jämförelse för att upptäcka avvikelser, bland annat i
incidentutredningen i vecka 8.

| Källa | Mål | Protokoll | Port | Syfte | Frekvens / mönster |
|---|---|---|---|---|---|
| 172.20.1.145 | 172.20.1.2 | TCP | 22 | SSH – administration via jump-servern | ~24 paket, utspridda över nästan hela fångsten |
| 172.20.1.145 | 185.12.64.1 | UDP | 53 | DNS – namnuppslag | ~10 paket, utspridda över fångsten |
| 172.20.1.145 | 185.12.64.2 | UDP | 123 | NTP – tidssynkronisering | ~20 paket (10+10), jämnt utspridda över hela fångsten |
| 172.20.1.145 | 46.224.108.117 | TCP | 80 | HTTP – vanlig webbtrafik | ~60 paket, utspridda över fångsten |
