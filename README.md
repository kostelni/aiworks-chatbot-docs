# A.I. Works ChatBot working docs - ultra fast intro

Jadro modelovacieho jazyka je postavené na <a href="https://www.pandorabots.com/docs/" target="_blank">AIML</a> XML markup-e.

Základ jazyka bol rozšírený o nové operátory, hlavne pre morfológiu. Niektoré pôvodné operátory boli
zmenené a prispôsobené.

NLP je riešené ako sekvenčný klasifikátor jazykových vzorov.

## Kategória/Intent

Báza znalostí je modelovaná pomocou tzv. kategórií, jedna kategória reprezentuje
jeden intent.

Kategória reprezentuje reakciu na jazykový vzor. Má dve základné časti:
* **PATTERN**: Jazykový vzor, odchytený od používateľa, na ktorý táto kategória reaguje. Pre jednu kategóriu je možné definovať ľubovoľne veľa patternov.
Pattern môže obsahovať rôzne operátory, ktoré vedia zovšeobecniť jazykový vzor. Napr. synonymá, morfologické kategórie, asociatóvne mapy, wildcardy, apod.
* **TEMPLATE**: Samotná reakcia na pattern. Vďaka XML markup-u môže byť reakcia v template celkom komplexná záležitosť.
Napr. symbolická redukcia, reťazenie templajtov, použitie premenných, rozhodovacie procesy, rôzne renderery, zneužitie externých servisov, apod.

```
<category>
    <pattern>CO ROBIS</pattern>
    <pattern>CO TU CHCES</pattern>
    <template>
    A co ja mozem za to, ze ma tu dali?
    </template>
</category>
```

Vyhodnocovací mechanizmus hľadá vždy najdlhší známy vzor. Operátory v patternoch sú
vyhodnocované podľa priority. Vyhráva prvý nájdený vzor s najvyššou nájdenou prioritou operátorov.

## Wildcards, stars a symbolická redukcia

### Wildcards / Stars

Najjednoduchšie zovšeobecnenie patternu. Za wildcard sa môže strčiť ľubovoľná sekvencia
vstupu, ktorú klasifikátor nevedel vyhodnotiť lepšie. Sú štyri a ich priorita,
ako sú vyhodnocované, aj v kontexte ostatných operátorov, je nasledovná:
* <b>#</b> : 0+ slov vo vstupe
* <b>_</b> : 1+ slov vo vstupe
* <b>ľubovoľný iný operátor</b>
* <b>*</b> : 0+ slov vo vstupe
* <b>^</b> : 1+ slov vo vstupe

S operátormi <b>#</b> a <b>_</b> treba nakladať opatrne, majú najvyššiu prioritu
a ak sú použité bezhlavo, môžu napáchať množstvo opičincov. Prekryjú totiž všetky ostatné
patterny. Ukážeme o chvíľu.

V templejte je, samozrejme, možné, dostať sa ku častiam vstupu, ktoré boli namapované na wildcardy.
Pomocou &lt;star/&gt; elementu.


## Pattern

Presnejšie, prehľad operátorov v pattern-och.







