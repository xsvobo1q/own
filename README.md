### TM1637
TM1637 je integrovaný obvod, který umožňuje po specifickém komunikačním rozhraní ovládat až 8 jednodigitových sedmisegmentových displejů.
Komunikační rozhraní je založeno na bázi dvouvodičové sběrnice I2C. Rodíl je především ve způsobu komunikace.
#### I2C - zjednodušený popis
Je multimastrová počítačová sběnice užívaná v nízkorychlostních systémech, např. připojení periferií k základní desce, vestavěnému systému
nebo mobilnímu telefonu. Rozhraní umožňuje připojit až 128 zařízení na obousměrnou dvouvodičovou datovou linku. Rozpoznáváme zařízení 
*master* nebo *slave*. Vodiče nazýváme *SDA* (Synchronous data) a *SCL* (Synchronous clock). Oba jsou připojeny pull-up rezistorem k vysoké úrovni - HIGH
v klidovém stavu. Změna úrovně *SDA* je možna pouze když je úroveň *SCL* nízká. To neplatí v případě odesílání tzv. *STOP bitu* a
*START bitu*, kterými komunikace počíná a končí. Zřízení, které chce poslat data (master), vyšle start bit - SCL: HIGH a SDA: sestupná hrana - čímž se
všechna zařízení na lince připraví k příjmu. Master odešle sedmibitovou adresu a jednobitový požadavek na přenos *R/W* (čtení/zápis).
Příjemce (slave) ztotožněný s vyslanou adresou vyšle potvrzovací bit, tzv. *ACK*. Následuje samotná datová komunikace, kdy kažý byte je 
následován ACK bitem, směr komunikace určen R/W bitem, obsaženým ve zprávě, a konec komunikace zakončen STOP bitem.
