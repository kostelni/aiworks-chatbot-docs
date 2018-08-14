# A.I. Works ChatBot working docs - ultra fast intro

Jadro modelovacieho jazyka je postavené na <a href="https://www.pandorabots.com/docs/" target="_blank">AIML</a> XML markup-e.

Základ jazyka bol rozšírený o nové operátory, hlavne pre morfológiu. Niektoré pôvodné operátory boli
zmenené a prispôsobené.

NLP je riešené ako sekvenčný klasifikátor jazykových vzorov.

## Kategória/Intent

Báza znalostí je modelovaná pomocou tzv. kategórií, jedna kategória reprezentuje
jeden intent.

Kategória reprezentuje reakciu na jazykový vzor. Má tri základné časti:
* **PATTERN**: Jazykový vzor, odchytený od používateľa, na ktorý táto kategória reaguje. Pre jednu kategóriu je možné definovať ľubovoľne veľa patternov.
Pattern môže obsahovať rôzne operátory, ktoré vedia zovšeobecniť jazykový vzor. Napr. synonymá, morfologické kategórie, asociatóvne mapy, wildcardy, apod.
* **KONTEXT**: Dialóg má svoju históriu, vyvíja sa kontext. **kontext** určuje, za akých okolností
daný intent platí. Rovnaká otázka potrebuje v závislosti od kontextu rôzne odpovede.
* **TEMPLATE**: Samotná reakcia na pattern. Vďaka XML markup-u môže byť reakcia v template celkom komplexná záležitosť.
Napr. symbolická redukcia, reťazenie templajtov, použitie premenných, rozhodovacie procesy, rôzne renderery, zneužitie externých servisov, apod.

```
<category>
    <pattern>CO ROBIS</pattern>
    <pattern>CO TU CHCES</pattern>
    <context></context>
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

### Defaulty

Je dobré vedieť reagovať aj vtedy, ak nevieme, ako reagovať. To je účel defaultných
kategórií.

**Príklad:**
```
Informovany intent:
<category>
    <pattern>POZNAS A.I.WORKS</pattern>
    <template>
        jasne, ze poznam .. id mi s nimi het
    </template>
</category>

Defaultny intent:
<category>
    <pattern>POZNAS *</pattern>
    <template>
        nepoznam [<star index="1"/>], nikto mi take nepovedal
    </template>
</category>

Ultimatny defaultny intent, zaberie na uplne vsetko, na co neexistuje ziaden pattern:
<category>
    <pattern>*</pattern>
    <template>
        sorac, netusim, co s tym
    </template>
</category>

Dialog:
U: poznas a.i.works?
B: jasne, ze poznam .. id s nimi het

U: poznas daco?
B: nepoznam daco, nikto mi take nepovedal

U: bla ble blu
B: sorac, netusim, co s tym
```

## PATTERN

Presnejšie, prehľad operátorov v pattern-och.

### Slovo

Proste platí, ak sa na vstupe objaví úplne konkrétne slovo.

```
<pattern>POZNAS *</pattern>
```

### Stem

Element: **&lt;stem&gt;**

Definuje koreň slova. Nepoužíva stemmer, je to triviálna heuristika, ktorá
platí, ak je operátor koreňom slova na vstupe. Namapuje sa do **star**.

```
<category>
    <pattern>^ <stem>POZNA</stem> ^</pattern>
    <template>
        slovo, pre ktore koren zabral: [<star index="2"/>]
        koren, ktory zabral: [<star index="2" matched-item="true"/>]
    </template>
</category>

U: nepoznas
B: slovo, pre ktore koren zabral: [nepoznas]
   koren, ktory zabral: [pozna]

```

Vyhodnotenie koreňa je možné obmedziť pre slová s rovnakým začiatkom/koncom:

```
<pattern>^ <stem same-start="true">POZNA</stem>^ </pattern> // poznala, poznas
<pattern>^ <stem same-end="true">POZNA</stem>^ </pattern>  // nepozna
```

### Morph

Element: **&lt;morph&gt;**

Definuje morfologické ohraničenia. Namapuje sa do **star**.

Atribúty:
* **lemma** : obmedzenie pre základný tvar slova
* **tags** : čiarkou oddelený zoznam morfologických rolí alebo tagov, platí, ak slovo musí mať aspoň jednu z rolí alebo tagov
* **not-tags** : čiarkou oddelený zoznam morfologických rolí alebo tagov, nesmie mať žiadnu z rolí alebo tagov

Morfologické roly (podstatné meno, sloveso) alebo tagy (plurál, tretia osoba) závisia
od aktuálne používaného korpusu. Každý korpus, podľa potreby, môže používať iné značky pre roly a tagy.

```
<pattern><morph lemma="poznat"/></pattern>
ano: poznas, poznali, poznate

<pattern><morph lemma="poznat" tags="inf"/></pattern>
ano: poznat
nie: poznali, poznate, ..

<pattern><morph tags="inf"/></pattern>
ano: poznat, robit, ..
nie: poznali, robite, ..

<pattern><morph not-tags="inf"/></pattern>
ano: poznali, robite, ..
nie: poznat, robit, ..

<pattern><morph lemma="poznat" tags="person2"/></pattern>
ano: poznas, poznate
nie: poznam, pozname, pozna
```

### Set

Element: **&lt;set name="nazov-setu"&gt;**

Definuje množinu výrazov. Vhodný pre synonymické slovníky alebo
pre malé skupiny fráz so spoločným významom (farby, mestá, ..).
Namapuje sa do **star**.

Pri veľkých zoznamoch je lepšie použiť NER.

Platí, ak je v jazykovom vzore fráza, zo **set-u**.

**Set** obsahuje len jednoduché frázy, bez wildcardov. Frázy môžu obsahovať
jednoduché slová a operátory **&lt;stem&gt;** alebo **&lt;morph&gt;**.
Frázy môžu byť viacslovné.

**Príklad:**

```
Definicia:
<set name="robit-firma">
    <pattern><morph lemma="robit" tags="person2, plural"/></pattern>
    <pattern><morph lemma="vyrabat" tags="person2, plural"/></pattern>
    <pattern><stem same-start="true">vyvijat</stem></pattern>
    <pattern>SKUSATE KAZIT</pattern>
</set>

Pouzitie:
<pattern>CO ^ <set name="robit-firma"/> ^</pattern>

zaberie na:
co robite
co vyrabate
co vyvijate
co skusate kazit
```

Keď sa skladajú väčšie synonymické slovníky, je celkom šikovné skladať rôzne **set-y**
do väčších celkov. To je možné urobiť pri definícii setov.

Element: **&lt;multi-set name="nazov-setu"&gt;**
**Multiset** môže zahŕňať iné **sety** alebo **multi-sety** vymenovaním ich názvov, čiarkou oddelene.
Umožňuje tiež pridať vlastné špecifické patterny.


**Príklad:**
```
Definicia
<set name="robite">
    <pattern>ROBITE</pattern>
    <pattern>VYRABATE</pattern>
</set>
<set name="chystate">
    <pattern>CHYSTATE</pattern>
    <pattern>PLANUJETE</pattern>
</set>

<multi-set name="cinite">
    robite, chystate
    <pattern>CINITE</pattern>
</multi-set>


pouzitie"
<pattern>^ <set name="cinite"/> ^</pattern>
plati pre vsetky prvky embednutych setov
```

### Map

Element: **&lt;map name="nazov-setu"&gt;**

Definuje jednoduchú asociatívnu mapu. Funguje podobne ako **set**, obsahuje
množinu rôznych fráz, kde ku každej je priradený zoznam hodnôt.

Platí, ak je v jazykovom vzore fráza, z **mapy-y**.
Pre frázy platia rovnaké pravidlá ako u **set-u**.

Rovnako, pre veľké mapy je lepšie použiť NER.

**Príklad:**
```
Definicia:
<map name="stat-mesto">
    <item>
        <pattern><stem same-start="true">slovensk</stem></pattern>
        <value>bratislava</value>
    </item>
    <item>
        <pattern><stem same-start="true">madarsk</stem></pattern>
        <value>budapest</value>
    </item>
</map>

<category>
    <pattern>^ HLAVNE MESTO <map name="stat-mesto"/> ^</pattern>
    <template>
        HLAVNE MESTO [<star index="2"/>] JE [<star index="2" matched-value="true"/>]
    </template>
</category>

<category>
    <pattern>^ HLAVNE MESTO *</pattern>
    <template>
        netusim, kamo, <star index="2"/>? o takom mi nepovedali
    </template>
</category>

Dialog:
U: ake je hlavne mesto slovenska
B: HLAVNE MESTO [slovenska] JE [bratislava]

U: ake je hlavne mesto madarska
B: HLAVNE MESTO [madarska] JE [budapest]

U: ake je hlavne mesto burundi burundi
B: netusim, kamo, burundi burundi? o takom mi nepovedali
```

### Morfologický wildcard

Element: **&lt;morph-wildcard tags="..." not-tags="..." allow-empty="true/false"&gt;**

Heuristický wildcard, ktorý zachytí sekvenciu pre ktorú platí morfologické ohraničenie.
* Atribúty **tags/not-tags** fungujú rovnako ako pri operátore **&lt;morph/&gt;**.
* Atribút **allow-empty="true"** dovolí aj prázdnu sekvenciu.

Bol vymyslený hlavne pre prípady, keď potrebujeme zamedziť v sekvekcii výskytu nejakého
slovného druhu. Najčastejšie slovesa.

**Príklad:**

```
<pattern>
    ^
    a.i.works
    ^
    <morph lemma="robit" tags="person3, inf"/>
    ^
    <stem same-start="true">chatbot</stem> ^
</pattern>
```

Tento pattern zaberie napr. na:
```
U: nejaky a.i.works mozno robi dalsieho blbeho chatbota so zelenym sosakom
```

Ale rovnako dobre zaberie napr. na:
```
U: kym a.i.works sedi na riti, niekto robi poriadneho chatbota
```

So štandardným sekvenčným klasifikátorom, ktorý nerobí plnú syntaktickú analýzu, je tento prípad proste súčasť života.
Morfologický wildcard tu ale môže pomôcť s obmedzením sekvencií, kde sa môže a kde
nemôže objaviť sloveso.

**Príklad:**

```
<pattern>
    ^
    a.i.works
    <morph-wildcard not-tags="verb" allow-empty="true"/>
    <morph lemma="robit" tags="person3, inf"/>
    <morph-wildcard not-tags="verb" allow-empty="true"/>
    <stem same-start="true">chatbot</stem> ^
</pattern>
```

V tomto prípade sa nemôže objaviť sloveso medzi operátormi:
* **a.i.works** a **robi** a ani
* **robi** a **chatbot**

Pattern teda chytí omnoho korektnejšie sekvencie na vstupe.

## TEMPLATE

Presnejšie, možnosti pre generovanie odpovedí.

### set/get

Nastavenie premennej a získanie hodnoty premennej.
Premenné je možné nastaviť v rôznom scope:
* **var="meno premennej"** premenná žije len pre aktuálny template, keď sa template vyhodnotí, premenná zaniká
* **name="meno premennej"** premenná sa uloží do kontextovej pamäte a existuje, kým nie je manuálne odstránená/zmenená

**Príklad:**

Nastavenie premennej:

Hodnota premennej môže byť konkrétny **string**, ale môže to byť ľubovoľný
výraz, ktorý je vyhodnotiteľný v rámci template. Môžu sa teda použiť
všetky operátory, ktoré sú platné pre template.

```
plati len pre aktualne vyhodnotenie template
<set var="x">hodnota</set>
<set var="x"><star index="i"/></set>

ulozenie do kontextovej pamate
<set name="y">hodnota</set>
<set name="x"><star index="i"/></set>
```

Získanie hodnoty premennej:

```
plati len pre aktualne vyhodnotenie template
<get var="x"/>

kontextova pamat: premenna moze byt ziskana kedykolvek a
kdekolvek pocas dialogu
<get name="y"/>
```

Likvidácia premennej:
```
<unset var="x"/>
<unset name="y"/>
```

### Conditional

Element: &lt;condition name/var="premenna"/&gt;

Jednoduchý **case**, ktorý dostane na vstup premennú a reaguje podľa jej hodnoty.
Premenná sa berie zo scopu **var/name**.

Každý case je definovaný ako element &lt;item/&gt; a obsahuje hodnotu pre porovnanie.
Dve možnosti:

Priamo definovaná hodnota pre porovnanie:
```
<item value="hodnota"> // value="*" plati pre akukolvek hodnotu
    reakcia
</item>
```

Hodnota pre porovnanie je odvodena:
```
<item>
    <value>
        template elementy pre vyhodnotenie,
        posledny vyhodnoteny vyraz bude hodnota
        pre porovnanie
        <set var="y">5</set>
        <get var="y"/> // toto je hodnota pre porovnanie
    </value>
    reakcia
</item>
```

**item** bez hodnoty je default.

Reakcia moze byt znova lubovolna sekvencia template elementov.

**Príklad:**

```
pomocna kategoria pre nastavenie porovnavacej premennej
<category>
    <pattern>SET *</pattern>
    <template>
        <set name="x"><star index="1"/></set>
    </template>
</category>

<category>
    <pattern>TEST</pattern>
    <template>
        porovnavam .. X=[<get name="x"/>]
        <condition name="x" >
            <item>
                <value>
                    <set var="y">5</set>
                    <get var="y"/>
                </value>
                x = y
            </item>
            <item value="2">
                x = 2
            </item>
            <item>
                default value
            </item>
        </condition>
    </template>
</category>

Dialog:
U: set 5
U: test
B: porovnavam .. X=[5]
   x = y

U: set 2
U: test
B: porovnavam .. X=[2]
   x = 2

U: set x
U: test
B: porovnavam .. X=[x]
   default value
```

### Random

Element &lt;random/&gt;

Zoznam rôznych odpovedí, to keď nechceme, aby chatbot odpovedal na otázku
stále rovnako.

**Príklad:**

```
<category>
    <pattern>CO VIES</pattern>
    <template>
        <random>
            <item>nic</item>
            <item>neviem, co viem</item>
            <item>nepoviem</item>
        </random>
    </template>
</category>

Dialog:
U: co vies
B: nic

U: co vies
B: nepoviem

U: co vies
B: nic

U: co vies
B: neviem, co viem
```

### String

Element: &lt;string delimiter="del"/&gt;

Jednoduchá normalizácia textu. Ak je odpoveď formálne rozhodená na viac riadkov,
toto ju jednoducho znormalizuje. Delimiter môže byť akýkoľvek *string*.

**Príklad:**

```
<string delimiter="">
    xx       xx
    yy
    zz
</string>

vrati: xxxxyyzz

<string delimiter=" ">
    xx       xx
    yy
    zz
</string>

vrati: xx xx yy zz
```

### Externé servisy

Elementy: &lt;sraix/&gt; a &lt;serivce/&gt;

Do templejtu je možné injektovať ľubovoľnú externú logiku. Tým pádom nie je skoro
žiadne obmedzenie na integráciu s ľubovoľným externým systémom.

Je potrebné vytvoriť servisnú triedu, ktorá implementuje interface:
```
scala:
eu.aiworks.chatbot.service.ExternalService

java:
eu.aiworks.chatbot.service.ExternalServiceWrapper
```

Tento interface predpisuje metódu:
```
def process(content: String, brain: BotBrain, indent: Int): Option[String]
```

Argumenty:
* **content**: hodnota odoslaná do servisu z template, nemá žiadne obmedzenia, môže to byť string, json, xml, čokoľvek
* **brain**: kompletný prístup k statickej a kontextovej pamäti
* **indent**: zabudnutý debugovací parameter pre intentáciu textu

Táto metóda urobí, čo treba a vráti výstup ako *string* späť do template.

Je to celé napísané v scale, pre javu to bude treba prispôsobiť. Hneď, ako bude treba.

**Príklad:**
```
1. deklaracia servisu:
<service name="demo" class="whatewer.you.say.Service"/>

2. pouzitie servisu:
<category>
    <pattern>TIME</pattern>
    <template>
        <set var="date-time">
            <sraix service="demo">
                {
                "action": "datetime"
                }
            </sraix>
        </set>
        je presne [<string delimiter=" "><get var="date-time"/></string>]
    </template>
</category>

3. v service:
content argument bude mat hodnotu:
{
"action": "datetime"
}
servis sparsuje json, pozrie na akciu, podla toho
sa bude chovat, kym nieco flusne spat do template
```

## ENTITY

Entity sú nový, experimentálny koncept. Entita je vo všeobecnosti samostatná
jednotka, ktorá môže byť rozpoznaná z frázy. Momentálne sú k dispozícii tri typy
entít:
* **automaticky extrahovaný dátový typ**:
    * číslo: 4, 45.3 , 34,3
    * email: x@y.z
    * telefónne číslo: +421 902 123 356
    * dátum: 23.5.1987
    * čas: 23:44
* **NER entita** rozpoznaná NER klasifikátorom
* **PATTERN entita**, špeciálny experimentálny koncept, kde entitu je možné
definovať cez špeciálnu kategóriu

Každá entita obsahuje:
* triedu: sémantická trieda, napr. farba, mesto, osoba
* dáta: JSON, ktorý obsahuje dáta patriace entite, napr. pre mesto to môže byť počet obyvateľov, štát, geo dáta apod. Dáta sú dostupné pre kontextové ohraničenia a tiež pre generovanie odpovedí.
* id: sémantický identifikátor, napr. konkrétna farba, mesto, osoba
