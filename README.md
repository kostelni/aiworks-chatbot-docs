# A.I. Works ChatBot working docs - ultra fast intro

Jadro modelovacieho jazyka je postavené na <a href="https://www.pandorabots.com/docs/" target="_blank">AIML</a> XML markup-e.

Základ jazyka bol rozšírený o nové operátory, hlavne pre morfológiu. Niektoré pôvodné operátory boli
zmenené a prispôsobené.

NLP je riešené ako sekvenčný klasifikátor jazykových vzorov. Vyhodnocovací mechanizmus hľadá
vždy najdlhší známy vzor. Keďže sú jazykové vzory reprezentované ako sekvencia, vyhráva prvý nájdený vzor.

## Kategória/Intent

Báza znalostí je modelovaná pomocou tzv. kategórií, jedna kategória reprezentuje
jeden intent.

Kategória reprezentuje reakciu na jazykový vzor. Má dve základné časti:
* **PATTERN**: Jazykový vzor, odchytený od používateľa, na ktorý táto kategória reaguje. Pre jednu kategóriu je možné definovať ľubovoľne veľa patternov.
Pattern môže obsahovať rôzne operátory, ktoré vedia zovšeobecniť jazykový vzor. Napr. synonymá, morfologické kategórie, asociatóvne mapy, wildcardy, apod.
* **TEMPLATE**: Samotná reakcia na pattern. Vďaka XML markup-u môže byť reakcia v template celkom komplexná záležitosť.
Napr. použitie premenných, rozhodovacie procesy, rôzne renderery, zneužitie externých servisov, apod.

```
<category>
    <pattern>CO ROBIS</pattern>
    <pattern>CO TU CHCES</pattern>
    <template>
    A co ja mozem za to, ze ma tu dali?
    </template>
</category>
```

## Pattern

Presnejšie, prehľad operátorov v pattern-och.

### Wildcards

Najjednoduchšie zovšeobecnenie patternu. Za wildcard sa môže strčiť ľubovoľná sekvencia
vstupu, ktorú klasifikátor nevedel vyhodnotiť lepšie. Sú štyri:
* <b>#</b> -
* **_** -
* <b>*</b> -
* **^** -

