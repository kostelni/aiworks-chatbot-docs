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

## Wildcards / Stars

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
Pomocou **&lt;star/&gt;** elementu.

Každý wildcard má index podľa poradia, ako sa vyskytol v patterne. Prvý wildcard má **index=1**.

**Príklad:**
Prvý wildcard musí obsahovať aspoň jedno slovo, druhý môže byť prázdny.

```
<category>
    <pattern>PRECO * ROBI ^</pattern>
    <template>
        Netusim preco <star index="1"/> robi <star index="2"/>
    </template>
</category>

Dialóg:
U: preco napr. xolution robi chatboty?
B: Netusim preco napr. xolution robi chatboty

U: preco mi to xolution robi?
B: Netusim preco xolution robi
```

Toto je úplne nazákladneší koncept. V skutočnosti sa skoro všetky operátory mapujú
na konkrétny **&lt;star/&gt;**.
Ďalšie príklady sa objavia po ceste.

## Základné koncepty

### Symbolická redukcia

Základný spôsob zjednodušenia pre rôzne jazykové vzory, na ktoré je potrebné
reagovať rovnako. Pre zachytenie jedného intentu je potrebné
definovať Y rôznych patternov. Odpoveď pre všetky tieto vzory ale chceme definovať len
raz. Presne na toto je symbolická redukcia.

Pre symbolickú redukciu je k dispozícii element &lt;srai&gt;. Umožňuje
definovať špecifický pattern, ktorý sa následne znova preženie klasifikátorom.
V príklade nižšie máme dve rôzne kategórie pre rovnaký intent, ktoré
pri vyhodnotení zavolajú rovnakú kategóriu. &lt;srai&gt; pre túto kategóriu je v podstate možné chápať
ako volanie funkcie s parametrami.

Sledujte indexy v **star**. Ak chceme pre jednu kategóriu definovať viac patternov, ale wildcardy, ktoré potrebujeme použiť
majú rôzne indexy, potrebujeme pre každý takýto pattern vytvoriť novú kategóriu.

**Príklad:**
```
<category>
    <pattern>^ VIES CI * ROBI *</pattern>
    <template>
        Volam jednodnu kategoriu pre tento intent..
        <srai>_ROBI <star index="2"/> _OBJECT <star index="3"/></srai>
    </template>
</category>

<category>
    <pattern>* ROBI *</pattern>
    <template>
        Volam jednodnu kategoriu pre tento intent..
        <srai>_ROBI <star index="1"/> _OBJECT <star index="2"/></srai>
    </template>
</category>

<category>
    <pattern>_ROBI * _OBJECT *</pattern>
    <template>
        Ja som jednotna kategoria pre intent, ci X robi Y.
        A pytam sa gugla, ci <star index="1"/> robi <star index="2"/>.
    </template>
</category>

```

### Premapovanie kľúčových slov

Ďalšie užitočné využitie symbolickej redukcie. Wildcardy s najvyššou
prioritou sa využijú na vyhľadanie sekvencií, ktoré sa premapujú
na preddefinované kľúče.

**Príklad:**
Cheme identifikovať rôzne frázy, pod ktorými je adresovaná **naša firma** a
použiť pre ne jednotný kľúč **#NASA_FIRMA#**.
Naša firma sa v texte môže objaviť ako:
* vaša firma
* a.i.works
* tvoja firma
* ...

Ak by sme definovali pattern týkajúci sa našej firmy, museli by sme kombinovať
všetky tieto frázy:

```
co robi vasa firma
co robi a.i.works
co robi tvoja firma
```

To nechceme. Chceme tieto frázy zjednotiť pod unikátny kľúč, ktorý sa bude
v patternoch o našej firme vyskytovať.

```
co robi #NASA_FIRMA#
```

Takto:

```
<category>
    <pattern># VASA FIRMA #</pattern>
    <pattern># A.I. WORKS #</pattern>
    <pattern># TVOJA FIRMA #</pattern>
    <template>
        <srai><star index="1"/> #NASA_FIRMA# <star index="2"/></srai>
    </template>
</category>

U: co robi vasa firma teraz
B: co robi #NASA_FIRMA# teraz

pridame cielovy intent

<category>
    <pattern>CO ROBI #NASA_FIRMA#</pattern>
    <template>
        Robime ultratechno!
    </template>
</category>

U: co robi vasa firma teraz
B: Robime ultratechno!
```


## Pattern

Presnejšie, prehľad operátorov v pattern-och.







