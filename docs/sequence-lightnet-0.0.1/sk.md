# A.I. Works Sequence LightNet ChatBot


## Intent

```
<intent id="">
    <pattern/>
    <pattern/>
    <pattern/>

    <response/> // kontext 1
    ...
    <response/> // kontext x
</intent>
```


## PATTERN: rozpoznanie textu

```
<pattern >
    <lemma/>
    <exact/>
    <stem/>
    <role/>
    <entity/>
    <synset id=".."/>
    <sequence id=".."/>
    <verb id=""/>
    <verb> // anonymne sloveso definovane len v ramci patternu
    *, ^   // wildcardy
</pattern>
</intent>
```

### &lt;exact/&gt;

Reaguje presne na toto slovo. Skratka: v patterne môže byť konkrétne slovo bez XML elementu.

```
<pattern>
    <exact>robot</exact>
</pattern>

je presne to isté, ako:

<pattern>
    robot
</pattern>
```


### &lt;lemma tags=""/&gt;

Reaguje na základný tvar slova a gramatické tagy - osoba, rod, číslo a pád. Ak tagy nie sú definované, zaberie na akýkoľvek tvar slova.


```
<lemma>robot</lemma>
trafí všetko: robot, robota, roboti, robotov, ..

<lemma tags="fall3, singular">robot</lemma>
trafí len: robotovi
```

* **Tagy pre osobu:** person1, person2, person3
* **Tagy pre rod:** gender-man, gender-woman, gender-neuter, gender-generic
* **Tagy pre číslo:** singular, plural
* **Tagy pre pád:** fall1, fall2, ..., fall7


### &lt;stem same-start="true/false" same-end="true/false"/&gt;

Reaguje na koreň slova a rovnaký začiatok alebo rovnaký koniec. Rovnaký začiatok a koniec nie sú povinné.


```
<stem>robot</lemma>
trafí všetko: robot, nerobota, robotom

<stem same-start="true>robot</lemma>
trafí všetko s prefixom: robot, robotxyz
nie: nerobot

<stem same-end="true>robot</lemma>
trafí všetko so sufixom: nerobot, nadrobot
nie: robotovia
```

### &lt;role allowed="r1, r2" tags="t1, t2" rejected="r1, r2"/&gt;

Reaguje na morfologické roly a tagy.

* **allowed:** zoznam gramatických rolí, ktoré slovo musí mať
* **rejected:** zoznam gramatických rolí, ktoré slovo nesmie mať
* **tags:** zoznam morfologických tagov, ktoré slovo musí mať


```
<role allowed="jj" rejected="rb"/>
trafí všetky prídavné mená, čo nie sú príslovky
áno: pekneho, dobreho, pekny
nie: pekne

<role allowed="jj" tags="singular, fall7"/>
trafí všetky prídavné mená v jednotnom čísle, 7. páde
áno: peknym
nie: v podstate čokoľvek iné
```

**Gramatické roly:**
* **verb**: akékoľvek sloveso
* **vb**: sloveso - základný tvar
* **vbd**: sloveso - minulý čas
* **vbg**: sloveso - gerundium
* **nn**: podstatné meno
* **jj**: prídavné meno
* **rb**: príslovka
* **prp**: zámeno
* **$prp**: privlastňovacie zámeno
* **cd**: číslovka
* **dt**: determinant
* **in**: predložka
* **q**: otázka
* **cc**: priraďovacia spojka
* **sc**: podraďovacia spojka
* **comma**: čiarka
* **any**: nerozpoznaná rola


**Tagy:** viď **lemma**

### Wildcardy
Za wildcard sa nahradí akýkoľvek text medzi dvoma matchermi v patterne.
* <b>^</b> : 0+ slov vo vstupe
* <b>*</b> : 1+ slov vo vstupe

Sekvenčný LightNet spracováva syntax komplexných slovies. Text, ktorým sa môže naplniť wildcard, je ohraničený
pozíciou slovesa a pozíciou striktného subordinátu (podraďovacia spojka). Sloveso a podr. spojka reprezentujú
heuristickú hranicu vety. Wildcard teda nemože obsahovať sloveso ani podraďovaciu spojku.

V patterne nesmú byť dva wildcardy po sebe, Na začiatku a konci patternu je dovolený len +1 wildcard <b>*</b>.


### &lt;synset id=""/&gt;

Synset je zoznam synoným slova. Musí mať ID. Jediná výnimka je, ak je synset definovaný v slovese (viď nižšie).

```
<synset id="">
    <exact>word, word, word</exact>
    <stem same-start="true/false" same-end="true/false">word, word, </stem>
    <role allowed="r1, r2" tags="t1, t2" rejected="r1, r2"/>
    <lemma tags="t1, t2">word, word, word</lemma>
</synset>
```

### Slovesá

Sekvenčný LightNet je gramatika, ktorá rozpoznáva tvary slovies. V slovenčine je to ultimátne záchranné koleso,
pretože tvary slovies sú pre syntaktický analyzátor na pozvracanie.

Príklad .. bez diakritiky: **chcel by si ma mat**.
Jedno sloveso, že hej? No ale v skutočnosti, z pohľadu morfológie, tam máme slovesá tri: **chcel, si, má** + neurčitok **mať**, ktorý
je zároveň neurčitok a zároveň môže byť podstatné meno **mať - matka**. Tfuj.

Gramatika heuristicky hľadá komplexné slovesá, ktoré prevádza do tvaru:
* **aux**: rozšírenie
* **verb**: nosné sloveso
* **inf**: neurčitok

Slovesá majú viac rôznych významov, podľa toho, či sú modálne a plno/neplnovýznamové. LightNet generuje
všetky možnosti, na ktoré príde.

Ak sa v slovese vyskutujú neplnovýznamové slovesá, ich význam sa prenáša.

```
môžeš/musíš/chceš/máš/... vidieť = vidíš
vidíš pracovať != pracuješ .. vidíš je plnovýznamové, neprenáša význam
```

Momentálne LightNet prenáša význam pre neplnovýznamové/modálne slovesá:
môcť, smieť, musieť, byť, vedieť, chcieť, mať. Je to záležitosť slovníka, ak treba, stačí doplniť.

Príklady.

```
sloveso: môžeš(neplnoýznamove) vidieť

1. aux(môžeš) verb(vidieť)
2. verb(môžeš) inf(vidieť)
interpretácia: možeš vidieť, vidíš

sloveso: môžeš(neplnoýznamove) chcieť(neplnovýznamové) vidieť

1. aux(môžeš) verb(chcieť) inf(vidieť)
interpretácia: možeš chcieť vidieť, chceš vidieť
```


**Verb matching:**
```
<pattern>
    <verb>
        // HLAVNE VYZNAMOVE SLOVESO
        <synset>
            <lemma>vidiet</lemma>
        </synset>
        // ROZSIRENIE
        <aux>
            <synset>
                <lemma>moct</lemma>
            </synset>
        </aux>
        // NEURCITOK
        <inf>
            <synset>
                <lemma>robit</lemma>
            </synset>
        </inf>
    </verb>
</pattern>
```

Ak sloveso chceme recyklovať, definuje sa samostatne a je ready na použitie vo viacerých patternoch.
```
<verb id="maj-verb">
    <synset>
        <lemma>vidiet</lemma>
    </synset>
</verb>

<pattern>
    <verb id="maj-verb"/>
</pattern>
```

**Príklady:**
```
<verb id="maj-verb">
    <synset>
        <lemma>vidiet</lemma>
    </synset>
</verb>

trafí: všetko, kde je nosné sloveso vidieť
    možeš/chceš/musíš/ .. vidieť
    možeš/chceš/musíš/ .. vidieť robiť
    vidíš robiť
    vidíš
```

```
<verb id="maj-verb">
    <synset>
        <lemma>vidiet</lemma>
    </synset>
    <aux>
        <synset>
            <lemma>moct</lemma>
        </synset>
    </aux>
</verb>

povinný aux .. trafí: všetko, kde je nosné sloveso a aux
áno:
    možeš vidieť
    možeš vidieť vidieť robiť
nie:
    chceš vidieť
```

```
<verb id="maj-verb">
    <synset>
        <lemma>vidiet</lemma>
    </synset>
    <aux>
        <synset>
            <lemma>moct</lemma>
        </synset>
    </aux>
    <inf>
        <synset>
            <lemma>robit</lemma>
        </synset>
    </inf>
</verb>

povinný aux aj inf .. trafí:
áno:
    možeš vidieť robiť
nie:
    možeš vidieť písať
    možeš vidieť
    chceš vidieť robiť
```

```
<verb id="maj-verb">
    <synset>
        <lemma>vidiet</lemma>
    </synset>
    <inf>
        <synset>
            <lemma>robit</lemma>
        </synset>
    </inf>
</verb>

nepovinný aux, povinný inf .. trafí:
    možeš/... vidieť robiť
    vidíš robiť
```

**Transformácia: komplexné sloveso - pattern**

Komplexné sloveso môže obsahovať frázy.
Napríklad: **mohlo by** xolution **ma** zajtra **vidieť**.

Aby tieto frázy bolo možné trafiť v patterne, musia sa dostať zo slovesa von.
Preto LightNet vyšmarí tieto frázy vždy za sloveso. Transformácia potom vyzerá takto:
```
tak mohlo by xolution ma zajtra vidieť na koňovi

sa pretransformuje na:

tak [sloveso: mohlo by vidieť] xolution ma zajtra na koňovi
```



Zložité. Viem. Ale to len na prvé poučutie. Užitočnosť to vyváží :)

### Sekvencie

Pattern je možné vybudovať zdola-nahor pomocou sekvencií, ktoré je možné
do seba skladať. Sekvencia môže obsahovať viac rôznych patternov. Sekvencie môžu byť povnárané
do akejkoľvek hĺbky. Puzzle.

```
    <sequence id="xolution">
        <pattern>
            <stem same-start="true">xolution</stem>
        </pattern>
        <pattern>
            <lemma>nas</lemma>
            <lemma>firma</lemma>
        </pattern>
    </sequence>

    <synset id="blbec">
        <stem same-start="true">blbec</stem>
        <stem same-start="true">blbco</stem>
    </synset>
    <sequence id="bot">
        <pattern><stem>bot</stem></pattern>
        <pattern>
            <synset id="blbec"/>
        </pattern>
    </sequence>

    <verb id="pracuje3">
        <synset>
            <lemma tags="person3">pracovat, robit</lemma>
        </synset>
    </verb>

    <sequence id="xolution-pracuje">
        <pattern>
            <sequence id="xolution"/>
            ^
            <verb id="pracuje3"/>
        </pattern>
        <pattern>
            <verb id="pracuje3"/>
            ^
            <sequence id="xolution"/>
        </pattern>
    </sequence>

    <intent id="i">
        <pattern>
            <sequence id="xolution-pracuje"/>
            ^
            <sequence id="bot"/>
        </pattern>
        <response>
            trafil
        </response>
    </intent>

Zaberie na:
pracuje xolution s tym blbcom
xolution moze pracovat s tym blbcom
mohlo by xolution pracovat s tym blbcom
vedelo by xolution s tym blbcom pracovat
...
```

## RESPONSE: odpoveď

```
<response>
    <context/>

    text ...

    <wildcard/>
    <set-property>
    <property/>
    <unset-property/>
    <template/>
    <random>
        <item/>
    </random>
    <service/>
    <cond/>
</response>
```

Intent môže mať lubovoľne veľa rôznych odpovedí, každú pre rôzny &lt;context&gt;. Defaultná odpoveď nemá
&lt;context&gt;. Ku kontextu sa dostaneme neskôr.

### &lt;wildcard index="int"&gt;

Vytiahne wildcard s indexom z patternu. Ak sú v patterne povnárane sekvencie, záleží na indexe vo výslednom zloženom patterne.

```
    <intent id="x">
        <pattern>x ^ y ^ z</pattern>
        <response>
            wildcards:

            1: [<wildcard index="1"/>]

            2: [<wildcard index="2"/>]
        </response>
    </intent>

U: x a y a z
B: wildcards: 1: [a] 2: [b]
```

### Set/Get Property

Funguje presne ako v AIML.

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
<set-property name="x">hodnota</set>

ulozenie do kontextovej pamate
<set-property name="y">hodnota</set>
```

Hodnota premennej môže byť akýkoľvek XML markup.

Získanie hodnoty premennej:

```
plati len pre aktualne vyhodnotenie template
<property var="x"/>

kontextova pamat: premenna moze byt ziskana kedykolvek a
kdekolvek pocas dialogu
<property name="y"/>
```

Likvidácia premennej:
```
<unset-property name="y"/>
```

### &lt;cond&gt;

Klasický switch/case:
```
<cond name/var="x">
    <case value="to compare">
        response
    </case>
    <case>
        <value>to compare</value>
        response
    </case>
    <case value="*">
        any value
    </case>
    <case>
        default
    </case>
</cond>
```
Hodnota vo &lt;value&gt; môže byť akýkoľvek XML markup.

### &lt;random&gt;

Náhodne vyberá ako odpoveď jeden z prvkov v zozname. Interpreter si pamatá, ktoré
prvky boli zodpovedané, takže pri viacerých odpovediach vyberá vždy ešte nepoužitú možnosť,
kým sa nepoužijú všetky. Protom sa cirkus začne znova náhodne opakovať.

```
<random>
    <item>response 1</item>
    <item>response 2</item>
    <item>response 3</item>
</random>
```

Hodnota vo &lt;item&gt; môže byť akýkoľvek XML markup.

### &lt;service&gt;

Do odpovede je možné injektovať ľubovoľnú externú logiku. Tým pádom nie je skoro
žiadne obmedzenie na integráciu s ľubovoľným externým systémom.

Je potrebné vytvoriť servisnú triedu, ktorá implementuje interface:
```
scala:
eu.aiworks.lightnet.engine.rules.service.ExternalService

java:
eu.aiworks.lightnet.engine.rules.service.ExternalServiceWrapper
```

Tento interface predpisuje metódu:
```
def process(input: String, method: String, brain: BrainData): scala.Option[String]
```

Argumenty:
* **input**: hodnota odoslaná do servisu, nemá žiadne obmedzenia
* **method**: meno funkcie, ktorú má servis exekjutnuť. servis je dobré chápať ako dispatcher rôznych funkcií
* **brain**: kompletný prístup k statickej a kontextovej pamäti

Táto metóda urobí, čo treba a vráti výstup ako *string* späť do response.

Je to celé napísané v scale, ale scala je prikompilovaná v engine, takže
s tým nie je problém.

**Príklad:**
```
1. deklaracia servisu:
<service id="demo" class="whatewer.you.say.Service"/>

2. pouzitie servisu:
    <service id="default" class=" eu.aiworks.lightnet.bot.service.TestService "/>
    <intent id="more">
        <pattern>* viac ako *</pattern>
        <response>
            <set-property var="is-more">
                <service id="default" function="more">
                    {"n1": <wildcard index="1"/>, "n2": <wildcard index="2"/>}
                </service>
            </set-property>

            <wildcard index="1"/> je viac ako <wildcard index="2"/>: [<property var="is-more"/>]

        </response>
    </intent>

3. vysledok:

U: 2 viac ako 1
B: 2 je viac ako 1: [true]
```

### &lt;template id=""&gt;

Často potrebujeme použiť v rôznych intentoch a rôznych kontextoch rovnakú odpoveď, len s inými "parametrami". Presne na to je template.
Variabilný text v template sa dá vytvoriť pomocou premenných. Navyše, templejt môže mať znova koľkokoľvek
rôznych odpovedí v závislosti od kontextu. Templejt je teda šablóna - parametrizovaná odpoveď
recyklovateľná z rôznych intentov a kontextov.
```
<template id="">
    <response>
        no context: default response!
    </response>

    <response>
        <context>...</context>
        context response..
    </response>

    <response>
        <context>...</context>
        context response..
    </response>

</template>
```

**Príklad:**
```
    <template id="t">
        <response>
            Ja som templejt [<property var="parameter"/>]
        </response>
    </template>
    <intent id="template">
        <pattern>go *</pattern>
        <response>
            Template this [<wildcard index="1"/>]
            <set-property var="parameter">
                <wildcard index="1"/>
            </set-property>
            <template id="t"/>

        </response>
    </intent>

U: go whatever
B: Template this [whatever]
   Ja som templejt [whatever]
```

## ENTITY
Entita je vo všeobecnosti samostatná
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
definovať pomocou patternov

Každá entita obsahuje:
* triedu: sémantická trieda, napr. farba, mesto, osoba
* id: sémantický identifikátor, napr. konkrétna farba, mesto, osoba
* dáta: JSON, ktorý obsahuje dáta patriace entite, napr. pre mesto to môže byť počet obyvateľov, štát, geo dáta apod. Dáta sú dostupné pre kontextové ohraničenia a tiež pre generovanie odpovedí.

Automaticky extrahované dátové typy majú triedy: !number, !email, !phone-number, !date, !time

### Manuálne definované entity
Ak vieme identifikovať jednu konkrétnu vec pomocou rôznych patternov
a chceme ju použiť ako špecifickú entitu, ktorá môže mať aj svoje vlastné dáta.

Povedzme, že
vieme ako entitu rozpoznať xolution. Ako entitu preto, že je to firma,
a ako ostatné firmy má adresu, počet zamestnancov, zameranie, a podobne.
A chceme s ňou pracovať ako s entitou, aby sme mali priamy prístup k týmto dátam.

rozpoznáme napr. pomocou patternov:
```
xolution
vasa firma
```

Pri skladaní jednoduchých patternov zasa nechceme kombinovať:
```
ceo v xolution
ceo vo vasej firme
```

Chceme:
```
ceo v <entity id="xolution"/>

alebo

ceo v <entity class="company"/>
```

Entitu vyrobíme takto:
```
    <entity id="" class="">
        <pattern/>
        <pattern/>
        <pattern/>

        <data>
            {json object}
        </data>
    </entity>

Napr(d):

    <entity id="xolution" class="company">
        <pattern>xolution</pattern>
        <pattern><lemma>vas</lemma> <lemma>firma</lemma></pattern>

        <data>
            {"name" : "xolution", "address": "dzeska na tomasikovej"}
        </data>
    </entity>

```

### Entita v patterne

Element: &lt;entity&gt;
Atribúty:
* *id* - sémantický typ
* *type* - sémantická trieda

Všetky typy entít sú v patterne rozpoznateľné rovnakým spôsobom. Životnosť entity
je len jeden request. Každý request, samozrejme, produkuje iné entity.

**Príklad:**
```
<intent id="e">
    <pattern>ceo [v,vo] <entity class="company"/></pattern>
    <response>
        ceo rules
    </response>
</intent>
U: ceo vo vasej firme
U: ceo v xolution
B: ceo rules

<intent id="dt1">
    <pattern>co je <entity class="!number"/></pattern>
    <response>
        cislo
    </response>
</intent>
<intent id="dt2">
    <pattern>co je <entity class="!date"/></pattern>
    <response>
        datum
    </response>
</intent>

U: co je 4
B: cislo

U: co je 12.11.2019
B: datum
```

### Entita v response

Entita je objekt, pracuje sa s ňou zábavnejšie, keďže máme prístup k jej dátovej
časti.

Podobne ako u get/set properties, entita môže mať rovnako scope:
* **var** - platný pre aktuálne vyhodnotenie response
* **name** - entita je uložená do kontextovej pamäte a má platnosť pre celý dialóg (alebo do manuálneho prepísania/zrušenia)

Práca s entitou umožňuje prístup do jej dátovej časti, tá môže byť použitá na generovanie
odpovede, ale rovnako dobre pre rozhodovacie procesy alebo vstup pre externé servisy.

**&lt;entity index/var/name&gt;** prístup ku entite
možnosti:
* **var/name** - entita, ktorá už je uložená v premennej
* **index** - entita z patternu, rovnaký princíp, ako pri wildcarde. Entity majú samostatný index. Index je teda podarové číslo entity, nie je zmiešaný s wildcardami alebo čímkoľvek iným.

**&lt;set-entity var/name&gt;** uloženie entity do premennej. Vždy sa nastavuje z **&lt;entity&gt;** elementu.
```
    <intent id="e">
        <pattern>ceo [v,vo] <entity class="company"/></pattern>
        <response>
            <set-entity name="cmp"><entity index="1"/></set-entity>
            ok .. ceo vo firme : <entity name="cmp"/>
        </response>
    </intent>
```


**&lt;unset-entity var/name&gt;** likvidácia entity


**Prístup k dátam v entite**

Dáta sú vždy JSON a na prístup k nim sa vždy používa [JSONPath štandard](https://goessner.net/articles/JsonPath/).

```
    <intent id="e">
        <pattern>ceo [v,vo] <entity class="company"/></pattern>
        <response>
            ceo v o firme

            meno: <entity index="1" data="$.name"/>

            adresa: <entity index="1" data="$.address"/>
        </response>
    </intent>

U: ceo vo vasej firme
B: ceo v o firme
   meno: xolution
   adresa: dzeska na tomasikovej
```

## KONTEXT

Rozhodovanie v kontexte. Jednoducho povedané, rovnaká otázka si za rôznych okolností vyžaduje rôzne odpovede.

Sú k dispozícii tri typy kontextových mechanizmov: kontextové triggre a ohraničenia.

### Kontextové triggre

Okamžité kontextové triggre so životnosťou pre nasledujúci request,
potom zmiznú. Sú vhodné pre bezprostredné riadenie jednoduchého dialógu.
Prečo majú triggre životnosť jedného requestu? Pretože trigger si vyžaduje
kontext bezprostredne poslednej odpovede.

**Príklad:**
```
U: kolko je hodin?
B: je 5:34

// toto si vyzaduje kontext bezprostredne poslednej odpovede
U: preco?
B: pretoze tolko mam na hodinkach

// toto si znova vyzaduje kontext bezprostredne
// poslednej odpovede
U: preco?
B: neviem preco. preto.
```

Každý intent, ktorý je odpoveďou automaticky vloží ako trigger svoje vlastne **id**. Dodatočné triggre je možné
do intentu doplniť. Takto:
```
<response>
    <set-context triggers="t1, t2, .., tx"/>
</response>
```

Príklad: použijem automaticky generovaný trigger z **id** intentu. Rovnako dobre to ale funguje aj pre manuálne
doplnený trigger.
```
    <intent id="c1">
        <pattern>
            <verb>
                <synset>
                    <lemma tags="person2">poznat</lemma>
                </synset>
            </verb>
            xolution
        </pattern>
        <response>
            poznam xolution .. chces vediet viac?
        </response>
    </intent>

    <intent id="c2">
        <pattern>
            ano
        </pattern>
        <response>
            co ano?
        </response>
        <response>
            <context triggers="c1"/>
            tu poviem viac o xolution
        </response>
    </intent>

U: ano
B: co ano?

U: poznas xolution?
B: poznam xolution .. chces vediet viac?

U: ano
B: tu poviem viac o xolution

U: ano
B: co ano?
```

### Kontextové ohraničenia

Blok ohraničení, ktoré určujú okolnosti za akých je intent platný.
Ohraničenia sú vždy vyhodnocované ako logický **AND**.

Element **context** nesmie byť prázdny, musí obsahovať aspoň atribút **triggers** a/alebo
aspoň jedno ohraničenie. Samozrejme, atribút **triggers** nie je povinný, ak sú definované viaceré ohraničenia.

Ako na to:
```
<response>
    <context>

        // PREMENNA EXISTUJE V KONTEXTOVEJ PAMATI:
        <property name="premenna"/>

        // PREMENNA EXISTUJE V KONTEXTOVEJ PAMATI A MA KONKRETNU HODNOTU:
        <property name="premenna" value="hodnota"/>

        // ENTITA EXISTUJE V KONTEXTOVEJ PAMATI:
        <entity name="premenna"/>

        // ENTITA EXISTUJE V KONTEXTOVEJ PAMATI A MA ID/CLASS:
        <entity name="premenna" id="entity-id"/>
        <entity name="premenna" class="entity-class"/>

    </context>
</response>
```

Príklad:

```
// MAME 2 ENTITY TYPU "company"

    <entity id="ai-works" class="company">
        <pattern>aiw</pattern>
        <data/>
    </entity>

    <entity id="xolution" class="company">
        <pattern>xolution</pattern>
        <data/>
    </entity>

// PRAVIDLO PRE NASTAVENIE/VYSMARENIE PREMENNEJ A ENTITY DO/Z KONTEXTOVEJ PAMATE

    <intent id="ctx-init">
        <pattern>
            set * <entity class="any"/>
        </pattern>
        <pattern>
            set *
        </pattern>
        <pattern>
            set <entity class="any"/>
        </pattern>
        <response>
            <set-property name="p"><wildcard index="1"/></set-property>
            <set-entity name="e"><entity index="1"/></set-entity>

            inicializujem kontext

            premenna [p]=[<property name="p"/>]

            entita [e]=[<entity name="e"/>]
        </response>
    </intent>
    <intent id="ctx-unset">
        <pattern>
            unset
        </pattern>
        <response>
            <unset-property name="p"/>
            <unset-entity name="e"/>

            mazem kontext
        </response>
    </intent>

// TEST NA PREMENNU

    <intent id="ctx-p">
        <pattern>
            p
        </pattern>
        <response>
            <context>
                <property name="p" value="x"/>
            </context>
            prop [p] is set to [x]
        </response>
        <response>
            <context>
                <property name="p"/>
            </context>
            prop [p] exists
        </response>
        <response>
            default: property [p] is not set
        </response>
    </intent>

// TEST NA ENTITU

    <intent id="ctx-e">
        <pattern>
            e
        </pattern>
        <response>
            <context>
                <entity name="e" id="xolution"/>
            </context>
            entity [e] is set to [xolution]
        </response>
        <response>
            <context>
                <entity name="e" class="company"/>
            </context>
            entity [e] is set to [company]
        </response>
        <response>
            <context>
                <property name="e"/>
            </context>
            entity [e] exists
        </response>
        <response>
            default: property [p] is not set
        </response>
    </intent>

U: p
B: default: property [p] is not set

U: set whatever
B: inicializujem kontext
   premenna [p]=[whatever]
   entita [e]=[]

U: p
B: prop [p] exists

U: set x
B: inicializujem kontext
   premenna [p]=[x]
   entita [e]=[]

U: p
B: prop [p] is set to [x]

U: set x@y.z
B: inicializujem kontext
   premenna [p]=[]
   entita [e]=[set x@y.z]

U: e
B: entity [e] exists

U: set aiw
B: inicializujem kontext
   premenna [p]=[]
   entita [e]=[aiw]

U: e
B: entity [e] is set to [company]

U: set xolution
B: inicializujem kontext
   premenna [p]=[]
   entita [e]=[xolution]

U: e
B: entity [e] is set to [xolution]
```

**Špeciálne kontextové ohraničenia**

Sekvenčný LightNet chytí intent podľa definovaného pravidla, ale nerozpoznáva kontext v rámci celého vstupu.
Táto iterácia LightNetu nemá kontext. Funguje v podsate ako gramatika. Rozpoznáva stále len sekvenčné patterny.

Napr. ak chceme chytiť intent pre jednoduché **áno**, vyzeralo by to takto:
```
<intent id="ano">
    <pattern>ano</pattern>
    <response>
        ano!
    </response>
</intent>

U: ano
B: ano!

U: ano je nove nie
B: ano!

U: bol som vcera pre rohliky a ano, mali nejake
B: ano!
```

Aby bolo možné rozpoznať jednoduchý kontext vstupu, existujú heuristické ohraničenia, ktoré
zistia, či intent je jednoduchá fráza (žiadne sloveso na vstupe) alebo jednoduchá veta (žiadne iné vety).
```
<response>
    <context>
        <input-is-single-sentence/>
    </context>
    vstup je len jedna jedina veta, jedno nosne sloveso
</response>
<response>
    <context>
        <input-is-phrase/>
    </context>
    vstup je len jedna jednoducha fraza, vstup nema ziadne slovesa
</response>
```


