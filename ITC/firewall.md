# FIREWALL

<img src="http://www.grouppolicy.biz/wp-content/uploads/2010/07/Firewall.png" width="700px">

- **zabezpečuje síť**
- různé **podoby**
    - zařízení
    - služba na serveru
    - součást routeru
    - součást OS (Windows Firewall)
- firewall stojí na rozhraní mezi zabezpečenou a nezabezpečenou sítí (**mezi LAN a WAN**)
- firewall je **účinný** za těchto podmínek
    - musí jím procházet veškerá komunikace
    - povoluje pouze komunikaci splňující pravidla
    - sám účinný proti napadení
- dva základní způsoby
    1. komunikaci **blokuje**
    2. komunikaci **povoluje**
- **kategorie** firewallů
    - paketové filtry
    - aplikační brány (proxy server)
    - stavové paketové filtry
    - stavové paketové filtry s kontrolou protokolů
- firewall poskytuje ochranu na základě **pravidel**
 - **příchozí pravidla**
   - pravidla pro komunikaci směřující do naší sítě
 - **odchozí pravidla**
    - pravidla pro komunikaci směřující z naší sítě

## Čísla portů
- každý packet obsahuje v záhlaví
  - zdrojovou a cílovou **IP**
  - zdrojové a cílové **číslo portu** (16b číslo ... $2^{16}$ ... 0-65535)
- IP:PORT = **socket**
- firewall provádí svá rozhodnutí většinou na základě informací o IP a číslech portů
  - dobře známé porty/**well-known** (0 - 1023)
    - http - 80
    - https - 443
    - http://gpoa.cz:80

| port number | service | protocol|
|:---:|:---:|:---:|
|20, 21| FTP | TCP|
|22 | SSH | TCP|
|23 | Telnet| TCP|
|25 | SMTP | TCP|
|53 | DNS | UDP, TCP|
| 80 | HTTP | TCP|
|110| POP3| TCP|
|143 | IMAP | TCP|
| 443 | HTTPS | TCP |

- cílový port je známý, zdrojový port je přidělován OS (1024 - 65 535)

---

## Paketový filter
- **nejjednodušší a nejstarší** forma firewallu
- firewall pracuje na základě informací obsažených v záhlaví paketů (IP adresy, čísla portů)
- podle těchto informací a podle tabulky pravidel firewall rozhodne, zda
	- paket příjme a pošle dál k cíli
	- paket odmítne
	- popř. se zeptá uživatele, co má s paketem udělat

### příklad
- 1 webový a 1 poštovní server jsou přístupné z internetu
- Vzdálená správa těchto serverů bude povolena prostřednictvím protokolu SSH (22)
- Z internetu však bude správa omezena pouze na zdrojovou síť 128.5.6.0/24

<img src="https://raw.githubusercontent.com/black05burn/pokus/master/firewall-priklad.png">

|Pravidla |Zdrojová IP| Cílová IP| Protokol | Zdrojový port | Cílový port | Akce |
| --- | --- | --- | --- | --- | --- | --- |
| 1	|	128.5.6.0/24 | 129.1.5.245 (WWW server) | TCP | >1023 | 22 | Povolit|
| 2 | 128.5.6.0/24 | 129.1.5.255 (MAIL server) | TCP | >1023 | 22 | Povolit |
| 3 | jakákoliv | 129.1.2.254 | TCP | >1023 | 80 | Povolit |
| 4 | jakákoliv | 129.1.5.255 | TCP | >1023 | 25 | Povolit |
| 5 | jakákoliv | jakákoliv | jakýkoliv | jakýkoliv | jakýkoliv| Zakázat|

- pravidlo č. **1,2**
	- povoluje ze sítě 128.5.6.0/24 odeslat pakety na webový nebo poštovní server ze zdrojovým portem větším než 1023 a cílovým portem 22 (SSH). Pokud paket nevyhovuje jde na další krok
- pravidlo č. **3**
	- povoluje jakékoliv síti odesílat pakety na port 80 (HTTP), zdrojový port víc jak 1023
- pravidlo č. **4**
	- obdoba č. 3, pouze definuje porty na poštovní server (SMTP)
- pravidlo č. **5**
	- musí být nakonfigurováno tak, aby došlo k zahození všech paketů, které nebyly ustanoveny žádným z předchozích pravidel

## Paketový filter
- zásady **vytváření pravidel**
	- od nejkonkrétnějšího po obecnější
	- nejčastěji používaná pravidla na začátku seznamu
- **výhody**
	- filtrování je prováděno ve vysokých rychlostech
	- finančně nenáročné (často součást OS)
	- nevyžaduje změnu chování uživatele
- **nevýhody**
	- paketové filtry mohou být napadeny viry
	- nízká úroveň kontroly procházejících spojení
- představitelé
	- ACL ve starších verzích Cisco zařízení.

---

## Brány aplikací (Proxy Server)
- na rozdíl od paketového filtru (pracuje na 3. síťové a 4. transportní vrstvě OSI modelu) pracují brány aplikací na všech 7 vrstvách OSI modelu
- brána aplikací **může**
	- být součástí brány firewall
	- pracovat ve spojení s firewallem
- **výhody**
	- je znalá aplikací, tj. ví, která aplikace si data vyžádala
- **nevýhody**
	- nízký výkon
	- uživatel musí komunikaci s proxy severem nastavit

---

## Stavové paketové filtry
-	provádějí kontrolu stejně jako jednoduché paketové filtry, navíc si však ukládají informace o povolených spojeních, které pak mohou využít při rozhodování, zda procházející pakety patří do již povoleného spojení a mohou být propuštěny, nebo zda musí znovu projít rozhodovacím procesem
-	**výhody**
	-	urychluje zpracování paketů již povolených spojení
	-	v pravidlech pro firewall lze uvádět jen směr navázání spojení a firewall bude samostatně schopen povolit i odpovědní pakety a u známých protokolů i další spojení, které daný protokol používá
	-	vysoká rychlost
	-	mnohonásobně snazší konfigurace
	-	nižší pravděpodobnost chybného nastavení pravidel
-	**nevýhody**
	-	nižší bezpečnost (než aplikační brány)

---

## Stavové paketové filtry s kontrolou protokolů a IDS
-	kromě informací o stavu spojení a schopnosti dynamicky otevírat porty pro různá řídící a datová spojení implementují tzv. Deep Inspection
	-	jsou schopny kontrolovat procházející spojení až na úroveň korektnosti procházejících dat a známých protokolů i aplikací
-	**IDS - Intrusion Detection System** (systém pro detekci útoků)
	-	je schopný odhalit vzorce útoků ve zdánlivě nesouvisejících pokusech o spojení (např. skenování IP adres, portů)
-	**výhody**
	-	vysoká úroveň bezpečnosti kontroly
	-	snadná konfigurace
	-	vysoká rychlost kontroly v porovnání s aplikačními branami
-	**nevýhody**
	-	nižší rychlost (o třetinu až polovinu) proti stavovým paketovým  filtrům
	-	velmi složité systémy -> vysoká pravděpodobnost, že v jejich kódu bude zneužitelná chyba
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzg3ODY4NTksMTY2OTkzOTA0OF19
-->
