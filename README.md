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

Automaticky extrahované dátové typy majú triedy: !number, !email, !phone-number, !date, !time

### Manuálne definované entity

Ak vieme identifikovať jednu konkrétnu vec pomocou rôznych patternov
a chceme ju použiť ako špecifickú entitu, ktorá môže mať aj svoje vlastné dáta.

Princíp je podobný, ako pri premapovaní kľúčových slov. Povedzme, že
vieme ako entitu rozpoznať a.i.works. Ako entitu preto, že je to firma,
a ako ostatné firmy má adresu, počet zamestnancov, zameranie, a podobne.
A chceme s ňou pracovať ako s entitou, aby sme mali priamy prístup k týmto dátam.

a.i.works rozpoznáme napr. pomocou patternov:
```
a i works
a.i.works
vasa firma
firma, kde pracuje ten puk
```

Pri skladaní jednoduchých patternov zasa nechceme kombinovať:
```
ceo v a i works
ceo v a.i.works
ceo vo vasej firme
ceo vo firme, kde pracuje ten puk
```

Chceme:
```
ceo v <entity id="ai-works"/>
```


Pri definovaní entity sa používa špeciálny typ kategórie, ktorý vyprodukuje entitu pre rôzne patterny.

**Príklad:**
Atribúty:
* **id** je sémantické ID entity, pre tento typ je povinné
* **type** je sémantické trieda entity

Je možné definovať ľubovoľne veľa **case** prípadov, kvôli vyhodnoteniu
**star** elementov.

Ku každému **case** je možné priradiť dátový objekt, momentálne len
tak idiotským spôsobom, ako je v príklade. Bude to lepšie.

```
<entity id="nasa-firma" type="company">
    <case>
        <pattern># <morph lemma="vasa"/> <morph lemma="firma"/> ^</pattern>
        <pattern># <morph lemma="firma"/> [KDE, V KTOREJ] ^ JE PETER ^</pattern>
        <pattern># <morph lemma="firma"/> ^</pattern>
        <object>
            <object-field name="name">
                nasa firma
            </object-field>
            <object-field name="info">
                <object>
                    <object-field name="address">
                    nasa addresa
                    </object-field>
                    <object-field name="employees">
                    3
                    </object-field>
                </object>
            </object-field>
        </object>
    </case>
</entity>
```

**!!!!!!** Mechanizmus pre definíciu patternov je úplne voľný, je možné použiť ľubovoľné patterny, ale odporučenie je:
** PATTERN MÁ AKO PRVÝ OPERÁTOR VŽDY # A AKO POSLEDNÝ TIEŽ WILDCARD, IDEÁLNE # ALEBO ^ . **
Týmto sa zabezpečí správna produkcia entity.

### Entita v patterne

Element: &lt;entity&gt;
Atribúty:
* *id* - sémantický typ
* *type* - sémantická trieda

Všetky typy entít sú v patterne rozpoznateľné rovnakým spôsobom. Životnosť entity
je len jeden request. Každý request, samozrejme, produkuje iné entity.

**Príklad:**

```
<category>
    <pattern>POZNAS <entity id="nasa-firma"/></pattern>
    <template>
        zas tychto? hej, poznam
    </template>
</category>

<category>
    <pattern>POZNAS <entity type="!number"/></pattern>
    <template>
        hej, to je cislo
    </template>
</category>

Dialog:
U: poznas ai works
B: zas tychto? hej, poznam

U: poznas 4,56
B: hej, to je cislo
```

### Entita v template


Entita je objekt, pracuje sa s ňou zábavnejšie, keďže máme prístup k jej dátovej
časti.

Podobne ako u premenných **get/set**, entita môže mať rovnako scope:
* **var** - platný pre aktuálne vyhodnotenie template
* **name** - entita je uložená do kontextovej pamäte a má platnosť pre celý dialóg (alebo do manuálneho prepísania/zrušenia)

Práca s entitou umožňuje prístup do jej dátovej časti, tá môže byť použitá na generovanie
odpovede, ale rovnako dobre pre rozhodovacie procesy alebo vstup pre externé servisy.

Entita sa vždy nastavuje zo **star** odchyteného patternu, je treba dávať pozor,
aby bolo nastavenie zo správneho **star**.

**Príklad:**

Práca s entitou .. postup:

```
<category>
    <pattern>POZNAS <entity id="nasa-firma"/></pattern>
    <template>

        1. nastavenie entity do premennej

        do kontextu:
        <set-entity name="x"><star index="1"/></set-entity>

        lokalne pre tento template:
        <set-entity var="y"><star index="1"/></set-entity>

        2. praca s entitou
           - entita je vzdy adresovana cez premennu
           - pre pristup k datam sa nezmyselne pouziva
             atribut "data"

        zakladna semantika:
        ID: <entity name="x" data="id"/>
        TYPE: <entity name="x" data="type"/>

        kompletny dump dat:
        JSON: <entity name="x" data="json"/>

        pre pristup ku konkretnej datovej casti
        sa pouziva JSONPath standard
        NAME: <entity name="x" data="$.name"/>
        ADDRESS: <entity var="x" data="$.info.address"/>

    </template>
</category>
```

## KONTEXT

Rozhodovanie v kontexte. Jednoducho povedané, rovnaká otázka si za rôznych
okolností vyžaduje rôzne odpovede.

Sú k dispozícii tri typy kontextových mechanizmov.

### Topic

Nastavenie témy. Systémový kontext, kategórie je možne organizovať podľa
témy. Rovnaký pattern bude mať pre rôzne tématické oblasti rôznu odpoveď. Ak je téma
nastavená, platí, kým nie je manuálne zmenená alebo odstránená.

**Priklad:**
```
// nastavenie temy na ai works:
<category>
    <pattern>POZNAS A.I.WORKS?</pattern>
    <template>
        <set name="topic">aiworks</set>

        co furt s tymito?
    </template>
</category>

// otazka mimo topic:
<category>
    <pattern>KOLKO MAJU ZAMESTNANCOV?</pattern>
    <template>
        kolko ma zamestnancov kto?
    </template>
</category>

// otazky v ramci topic:
<topic name="aiworks">
    <category>
        <pattern>KOLKO MAJU ZAMESTNANCOV?</pattern>
        <template>
            ai works nema zamestnancov .. su oni vobec?
        </template>
    </category>
</topic>

Dialog:
U: kolko maju zamesntancov?
B: kolko ma zamestnancov kto?

U: poznas ai works?
B: co furt s tymito?

U: kolko maju zamesntancov?
B: ai works nema zamestnancov .. su oni vobec?
```

### Triggers

Okamžité kontextové triggre so životnosťou pre nasledujúci request,
potom zmiznú. Sú vhodné pre bezprostredné riadenie jednoduchého dialógu.

Kategória produkuje okamžitý kontext pomocou atribútu **context="ciarkou oddeleny zoznam produkovanych triggrov"**.

Kategória vyhodnocuje kontext pomocou elementu **context/triggers**,
obsah je čiarkou oddelený zoznam okamžitých triggrov, ktoré musia platiť, aby
platila kategória.

```
<category>
    <pattern>...</pattern>
    <context>
        <triggers></triggers>
    </context>
    <template>
    </template>
</category>
```

**Priklad:**

```
<category context="viac-o-aiworks">
    <pattern>POZNAS A.I.WORKS?</pattern>
    <template>
        co furt s tymito?
        chces o nich vediet viac?
    </template>
</category>

// otazka mimo kontext
<category>
    <pattern>ANO</pattern>
    <template>
        co ano?
    </template>
</category>

// otazka v kontexte
<category>
    <pattern>ANO</pattern>
    <context>
        <triggers>viac-o-aiworks</triggers>
    </context>
    <template>
        ai works su bla bla
    </template>
</category>

Dialog:
U: ano?
B: co ano?

U: poznas ai works?
B: co furt s tymito?
   chces o nich vediet viac?

U: ano
B: co furt s tymito?
   ai works su bla bla

U: ano?
B: co ano?
```

**Priklad:**
Kombinácia topic a triggrov.

```
    // pomocne kategorie
    <category>
        <pattern>unset</pattern>
        <template>
            unsetting topic : <get name="topic"/>
            <unset name="topic"/>
        </template>
    </category>

    <category><pattern>set *</pattern>
        <template>
            <set name="topic"><star/></set>
            topic set to: <star/>
        </template>
    </category>

    <category context="ctx">
        <pattern>^ ctx ^</pattern>
        <template>
            OK .. CONTEXT WAS SET
        </template>
    </category>

    // vyznamove kategorie
    <topic name="topic one">
        <category name="topic-1-hi">
            <pattern>hi</pattern>
            <template>TOPIC ONE HI!</template>
        </category>
    </topic>

    <topic name="topic two">
        <category name="topic-2-hi">
            <pattern>hi</pattern>
            <template>TOPIC TWO HI!</template>
        </category>
        <category name="topic-2-ctx-hi">
            <pattern>hi</pattern>
            <context><triggers>ctx</triggers></context>
            <template>TOPIC TWO + CONTEXT HI!</template>
        </category>
    </topic>

    <category name="ctx-hi">
        <pattern>hi</pattern>
        <context><triggers>ctx</triggers></context>
        <template>CONTEXT HI!</template>
    </category>

    <category name="default-hi">
        <pattern>hi</pattern>
        <template>DEFAULT HI!</template>
    </category>

    <category name="default">
        <pattern>*</pattern>
        <template>DEFAULT</template>
    </category>

Dialog:
U: hi
B: DEFAULT HI

U: set topic one
B: topic set to: topic one

U: hi
B: TOPIC ONE HI!

U: ctx
B: OK .. CONTEXT WAS SET

U: hi
B: TOPIC ONE HI!

U: set topic two
B: topic set to: topic two

U: hi
B: TOPIC TWO HI!

U: ctx
B: OK .. CONTEXT WAS SET

U: hi
B: TOPIC TWO + CONTEXT HI!

U: hi
B: TOPIC TWO HI!

U: unset
B: unsetting topic : topic two

U: hi
B: DEFAULT HI!
```

### Kontextové ohraničenia

Blok ohraničení, ktoré určujú okolnosti za akých je kategória platná.
Ohraničenia sú vždy vyhodnocované ako logický **AND**.

Kategória vyhodnocuje kontext pomocou elementu **context/triggers**
a zároveň zoznamu dodatočných ohraničení, ktoré sú definované v elemente **context**.

Element **context** nesmie byť prázdny, musí obsahovať aspoň **triggers** a/alebo
iné ohraničenia. Samozrejme, blok **triggers** nie je povinný, ak sú definované viaceré ohraničenia.

**Príklad:**

```
<category>
    <context>
        // ---------------------------------
        // EXISTENCIA V KONTEXTOVEJ PAMATI
        // ---------------------------------
        // v kontextovej pamati existuje premenna X
        <property name="x"/>

        // v kontextovej pamati existuje entita X
        <entity name="x"/>


        // ---------------------------------
        // EXISTENCIA V KONTEXTOVEJ PAMATI
        // A TEST HODNOTY
        // ---------------------------------
        // v kontextovej pamati je premenna X
        // a jej hodnota je 5
        <property name="x">5</property>

        // v kontextovej pamäti existuje entita X
        // a jej typ je "type"
        <entity name="x" data="type">type</entity>

        // v kontextovej pamäti existuje entita X
        // a jej id je "instance"
        <entity name="x" data="id">instance</entity>

        // v kontextovej pamäti existuje entita X
        // a jej adresovana datova zlozka ma hodnotu 5
        <entity name="x" data="$.info.price">5</entity>

    </context>
</category>
```

Pre testovanie hodnoty je možné použiť zložitejší logický výraz.
* Premenná v logickom výraze sa zapisuje ako **${meno-premennej}**
* Operátory: **==** , **!=** , **>** , **<** , **>=** , **<=**
* Logické operátory: **and** , **or**
* Logická formula môže obsahovať časti zabalené do zátvoriek
* Ak bola do kontextovej pamäti uložená premenná s menom **premenna**,
do ${variable} sa dosadí hodnota tejto premennej

**Príklad:**
```
(
    ( (${x}>=5 and ${x}<=10) or ${y}!=7 )
    and
    (${z}>=5 and ${z}<=10)
)
```

Logické výrazy je možné použiť pre vyhodnotenie konkrétnej premennej:
```
<category>
    <context>
        // TEST HODNOTY PREMENNEJ
        // premenna, ktora obsahuje hodnotu premennej X
        // je v logickej klauzule dostupna v premennej ${value}
        <property name="x">
            <![CDATA[
            ${value}<=20
            ]]>
        </property>

        // TEST DATOVEJ ZLOZKY ENTITY
        // premenna, ktora obsahuje hodnotu datovej zlozky "$.info.price"
        // je v logickej klauzule dostupna v premennej ${value}
        <entity name="x" data="$.info.price">
            <![CDATA[
            ${value}>=20 and ${value}<=30
            ]]>
        </property>
    </context>
</category>
```

**Príklad:**

Máme chatota, ktorý odpovedá otázky o hrách. Hry
sú reprezentované ako pomenované entity. Každá obsahuje dátovú časť v tvare:
```
{
    "name": "meno hry",
    "price": cena,
    "genre": [strielacka, strategia, ...]
}
```

Cheme dialóg:
```
U: poznas csko?
B: jasne, poznam counter strike

U: kolko stoji
B: counter strike stoji 30EUR

U: poznas takeho nit
B: takeho nit? take nepoznam

U: kolko stoji?
B: kolko stoji co?

U: prince
B: prince stoji 10EUR
```

Ak sa používateľ nesmelo spýta na hru, chabot, ak hru pozná, uloží si ju do
kontextovej pamäte, táto hra bude náš aktuálny kontext.
Pri každej kontextovej otázke chatbot musí overiť, či je kontext nastavený
na hru, ak áno, vie pracovať s hrou, ktorá je nastavená ako kontextová téma.

```
<?xml version="1.0" encoding="UTF-8"?>

<aiml version="1.0">

    // POZNAS HRU
    <category>
        <pattern>POZNAS ^ <entity type="game"/> ^</pattern>
        <template>
            // NASTAVIME ENTITU DO KONTEXTU
            <set-entity name="game"><star index="2"/></set-entity>

            jasne, poznam hru [<entity name="game" data="$.name"/>]
        </template>
    </category>

    // POZNAS DEFAULT
    <category>
        <pattern>POZNAS *</pattern>
        <template>
            // VYSMARIME ENTITU Z KONTEXTU
            <unset name="game"/>
            <star index="1"/>? take nepoznam
        </template>
    </category>

    // KOLKO STOJI HRA O KTOREJ SA BAVIME
    <category>
        <pattern>KOLKO STOJI</pattern>
        <context>
            <entity name="game"/>
        </context>
        <template>
            <entity name="game" data="$.name"/> stoji <entity name="game" data="$.price"/>
        </template>
    </category>

    // KOLKO STOJI DEFAULT
    // NASTAVI KONTEXTOVY TRIGGER, AK V DALSEJ ODPOVEDI
    // PRIDE HRA, ODPOVIE NA OTAZKU: KOLKO STOJI HRA
    <category context="kolko-stoji-hra">
        <pattern>KOLKO STOJI</pattern>
        <template>
            kolko stoji co?
        </template>
    </category>


    // GENERICKY SPUSTAC: HRA
    <category>
        <pattern>^ <entity type="game"/> ^</pattern>
        <template>
            <set-entity name="game"><star index="2"/></set-entity>
            ok .. bavime sa o hre [<entity name="game" data="$.name"/>]
        </template>
    </category>

    // KONTEXTOVY SPUSTAC: HRA
    <category>
        <pattern>^ <entity type="game"/> ^</pattern>
        <context>
            <triggers>kolko-stoji-hra</triggers>
        </context>
        <template>
            <set-entity name="game"><star index="2"/></set-entity>
            <srai>KOLKO STOJI</srai>
        </template>
    </category>

</aiml>
```

### Kategórie v kontexte

Kontext znamená na rovnakú otázku odpovedať inak podľa aktuálnych okolností.
Zapisovať vždy novú kategóriu pre každý nový kontext je dláždenie si cesty do pekla.
Totiž, ak znalostná báza dosiahne rozmery úctyhodného charakteru,
nie je nič jednoduchšie, ako stratiť sa v nej. Aj keď je zosnovaná vlastnoručne.

Preto je možné dať dohromady reakcie na rovnaký pattern (alebo skupinu patternov).
Jeden pattern, defaultný template, a potom reakcie pre rôzne kontexty. Ten
zápis vyzerá takto:
```
<category>
    <pattern>^ <entity type="game"/> ^</pattern>

    // KONTEXT 1
    <context>
        <triggers></triggers>
        <property/>
        .. ohranicenia

        <template>
            TEMPLATE PRE KONTEXT 1
        </template>
    </context>


    // KONTEXT 2
    <context>
        <triggers></triggers>
        <property/>
        .. ohranicenia

        <template>
            TEMPLATE PRE KONTEXT 2
        </template>
    </context>

    .. dalsie kontexty

    <template>
        DEFAULTNY TEMPLATE
    </template>
    </category>
```

