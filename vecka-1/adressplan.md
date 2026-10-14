## Adressplan – 10.40.0.0/16

**Förutsättningar:** Uppdragsgivarens adressutrymme är 10.40.0.0/16 (RFC 1918). Adressplanen omfattar IPv4. IPv6 är avstängt på labbservrarna och ingår inte i planen. 25 % tillväxt är inräknad på samtliga grupper utom länknät.

### Subnet cheat-sheet

128   64   32   16   8   4   2   1   Group Size
128   192  224  240 248 252 254 255  Subnet Mask
/25   /26  /27  /28 /29 /30 /31 /32  CIDR
/17   /18  /19  /20 /21 /22 /23 /24  3rd octet
/9    /10  /11  /12 /13 /14 /15 /16  2nd octet
/1    /2   /3   /4  /5  /6  /7  /8   1st octet

Dela upp nätet på:
Grupp                     Idag   Med 25 % tillväxt
Klienter                   120   150
Gäst och besök              40    50
Servrar                     14    18
Drift och administration     6     8
Databaser                    4     5
Logg och övervakning         3     4
DMZ                          4     5
Länknät mellan routrar       3 st à 2 adresser


10.40.0.0/16
32-16=16
2^16=65536

Klientnätverk:
10.40.0.0/24 (254 tillgängliga adresser)
10.40.0.1 - 10.40.0.254

Gäst & besök:
10.40.1.0/26 (62 tillgängliga)
10.40.1.1 - 10.40.1.62

Servrar:
10.40.1.64/27 (30)
10.40.1.65 - 10.40.1.94

Drift & admin:
10.40.1.96/28 (14)
10.40.1.97 - 10.40.1.110

Databaser:
10.40.1.112/29 (6)
10.40.1.113 - 10.40.1.118

Logg & övervakning:
10.40.1.120/29 (6)
10.40.1.121 - 10.40.1.126

DMZ:
10.40.1.128/29 (6)
10.40.1.129 - 10.40.1.134

Länk 1:
10.40.1.136/30 (2)
10.40.1.137 - 10.40.1.138

Länk 2:
10.40.1.140/30 (2)
10.40.1.141 - 10.40.1.142

Länk 3:
10.40.1.144/30 (2)
10.40.1.145 - 10.40.1.146



| Prefix | Syfte | Adressintervall | Ansvarig tjänst | Reserverat utrymme | Kommentar |
|---|---|---|---|---|---|
| 10.40.0.0/24 | Klienter | 10.40.0.1 – 10.40.0.254 | IT-drift | - | 150 av 254 i bruk, plats för vidare tillväxt |
| 10.40.1.0/26 | Gäst och besök | 10.40.1.1 – 10.40.1.62 | IT-drift | — | Isolerat, bara internetåtkomst |
| 10.40.1.64/27 | Servrar | 10.40.1.65 – 10.40.1.94 | Drift | — | Applikations- och filservrar |
| 10.40.1.96/28 | Drift och administration | 10.40.1.97 – 10.40.1.110 | Drift | — | Jump-server, adminverktyg |
| 10.40.1.112/29 | Databaser | 10.40.1.113 – 10.40.1.118 | Drift | — | Kunddatabas, personuppgifter – högst skyddad |
| 10.40.1.120/29 | Logg och övervakning | 10.40.1.121 – 10.40.1.126 | Drift | — | Central loggning (syslog) |
| 10.40.1.128/29 | DMZ | 10.40.1.129 – 10.40.1.134 | Drift | — | Publik webbserver |
| 10.40.1.136/30 | Länknät 1 | 10.40.1.137 – 10.40.1.138 | Nätverk | — | Punkt-till-punkt mellan routrar |
| 10.40.1.140/30 | Länknät 2 | 10.40.1.141 – 10.40.1.142 | Nätverk | — | Punkt-till-punkt mellan routrar |
| 10.40.1.144/30 | Länknät 3 | 10.40.1.145 – 10.40.1.146 | Nätverk | — | Punkt-till-punkt mellan routrar |

**Reserverat utrymme:** 10.40.1.148 – 10.40.255.255 ledigt för framtida grupper/tillväxt (merparten av /16 oanvänt).