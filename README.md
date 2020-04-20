### TM1637
TM1637 je integrovaný obvod, který umožňuje po specifickém komunikačním rozhraní ovládat až 6 jednodigitových sedmisegmentových displejů.
Komunikační rozhraní je založeno na bázi dvouvodičové sběrnice I2C. Rozdíl je především ve způsobu komunikace.

#### I2C - zjednodušený popis
Je multimastrová počítačová sběnice užívaná v nízkorychlostních systémech, např. připojení periferií k základní desce, vestavěnému systému
nebo mobilnímu telefonu. Rozhraní umožňuje připojit až 128 zařízení na obousměrnou dvouvodičovou datovou linku. Rozpoznáváme zařízení 
*master* nebo *slave*. Vodiče nazýváme *SDA* (Synchronous data) a *SCL* (Synchronous clock). Oba jsou připojeny pull-up rezistorem k vysoké úrovni - HIGH v klidovém stavu. Změna úrovně *SDA* je možna pouze když je úroveň *SCL* nízká. To neplatí v případě odesílání tzv. *STOP bitu* a *START bitu*, kterými komunikace počíná a končí. Zřízení, které chce poslat data (master), vyšle start bit - SCL: HIGH a SDA: sestupná hrana - čímž se všechna zařízení na lince připraví k příjmu. Master odešle sedmibitovou adresu a jednobitový požadavek na přenos *R/W* (čtení/zápis). Příjemce (slave) ztotožněný s vyslanou adresou vyšle potvrzovací bit, tzv. *ACK*. Následuje samotná datová komunikace, kdy kažý byte je následován ACK bitem, směr komunikace určen R/W bitem, obsaženým ve zprávě, a konec komunikace zakončen STOP bitem.
<img src = "I2C_wiring.png">
<img src = "I2C_comunication.png">

*Obrázky: www.zavavov.cz.*     
**Důležité je pořadí přenosu bitů. Jak je znázorněno na obrázku, komunikace začíná MSB bitem zprávy a končí LSB bitem zprávy.**

#### Komunikační linka TM1637
Zapojení je opět dvouvodičové (vodiče *CLK* a *DIO*; jiné pojmenování oproti I2C kvůli licenčním důvodům, funkce je však stejná) s pull-up rezistory. Jelikož u tohoto typu komunikace neexistuje adresování (základní rozdíl od I2C), lze na linku připojit pouze jeden SLAVE obvod (nebo více, ale všechna SLAVE zařízení budou přijímat stejná data). Přenos dat se pak děje stejně jako u I2C  tím rozdílem, že jako **první z datového slova odesíláme LSB bit**.

O nastavení TM1637 a jeho dislejů rozhodují 3 datová slova, kterým budeme říkat příkazy. První příkaz určuje zda chceme data číst či zapisovat, zda chceme adresu digitu nastavit fixně či inkrementovat od zvolené hodnoty, atd... (viz. tabulka). V naší impelemntaci užíváme slovo *01000000*. Jeho význam lze slovně popsat jako: nastavení displeje pro zápis do normálního módu s automatickou inkrementací adresy od počáteční adresy nastavenou dalším příkazem.

|B7|B6|B5|B4|B3|B2|B1|B0|Význam|
|----|----|----|----|----|----|----|----|-----------------|
|0|1|0|0|||0|0|zápis dat do registru|
|0|1|0|0|||1|0|čtení, data z klávesnice|
|0|1|0|0||0|||automatická inkrementace adresy digitu|
|0|1|0|0||1|||nastavení adresy digitu fixně|
|0|1|0|0|0||||normální mód|
|0|1|0|0|1||||testovací mód|

Druhý příkaz určuje adresu prvního digitu, od které bude inkrementováno. V našem případě kdy užíváme 4 digitový displej, jsou adresy vyšší jak 0xC3 ignorovány.

|B7 |   B6 |   B5|    B4 |   B3 |   B2 |   B1 |   B0|Význam|
|----|----|----|----|----|----|----|----|-----------------|
|1|1|0|0|0|0|0|0|0xC0 - 1. digit od leva|
|1|1|0|0|0|0|1|0|0xC1 - 2. digit od leva|
|1|1|0|0|0|0|1|0|0xC2 - 3. digit od leva|
|1|1|0|0|0|0|1|1|0xC3 - 4. digit od leva|
|1|1|0|0|0|1|0|0|0xC4 - 5. digit od leva|
|1|1|0|0|0|1|0|1|0xC5 - 6. digit od leva|

Třetí příkaz udává zda budou displeje indikovat a také jak velkým jasem

|B7 |   B6 |   B5|    B4 |   B3 |   B2 |   B1 |   B0|Význam|
|----|----|----|----|----|----|----|----|-----------------|
|1|0|0|0||0|0|0| PWM 1/16|
|1|0|0|0||0|0|1| PWM 2/16|
|1|0|0|0||0|1|0| PWM 4/16|
|1|0|0|0||0|1|1| PWM 10/16|
|1|0|0|0||1|0|0| PWM 11/16|
|1|0|0|0||1|0|1| PWM 12/16|
|1|0|0|0||1|1|0| PWM 13/16|
|1|0|0|0||1|1|1| PWM 14/16|
|1|0|0|0|0|||| display OFF|
|1|0|0|0|1|||| display ON|

**Popis komunikace v naší implementaci**    
Začínáme odesláním start bitu a hned za ním odesláním prvního příkazu. Po MSB nastavíme na MASTERU výstup na vysokou impedanci, protože SLAVE (TM1637) s dalším taktem signálu CLK odešle ACK bit. Ve skutečnosti tento bit vůbec nečteme, proto nastavujeme vysokou impedanci. Po ACK následuje STOP bit a další START bit, který zahájí přenos druhého příkazu. Opět po odeslání nastavíme DIO na vysokou impedanci s následným odesláním STOP bitu. Další START bit zahajuje přenos dat pro první (námi nastavený) digit. Po odeslání posledního bitu nastavujeme DIO na vysokou impedanci - "čtení ACK". Přenos těchto dat není ukončován STOP bitem, stejnětak jako počátek přensou dat dalšího bitu není uvozován START bitem. Přenos dat pro digity je zkrátka uvozen jedním START bitem, potom po každých osmi bitech následuje ACK bit a přenos je zakončen po posledních osmi bitech STOP bitem. No a nakonec odesíláme příkaz třetí, rozhodující o jasu dipleje. Přehlednější znázornění datové komunikace je na dalším obrázku.    
<img src = "TM1637_data.PNG">
*Obrázek: katalogový list TM1637; Titan micro electronics*      
