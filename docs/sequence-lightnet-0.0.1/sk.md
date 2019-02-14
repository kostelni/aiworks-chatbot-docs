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


## Pattern: rozpoznanie textu

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
trafí všetko s prefixom: robot, nerobota
nie: robota

<stem same-end="true>robot</lemma>
trafí všetko so sufixom: robotovi, robote
nie: nerobota
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

Sekvenčný LightNet spracováva syntax komplexných slovies. Hranice pre text, ktorým sa môže naplniť wildcard je ohraničený
pozíciou slovesa a pozíciou striktného subordinátu (podraďovacia spojka). Sloveso a podr. spojka reprezentujú
heuristickú hranicu vety. Wildcard teda nemože obsahovať sloveso ani podraďovaciu spojku.

V patterne nesmú byť dva wildcardy po sebe, Na začiatku a konci patternu je dovolený len +1 wildcard <b>*</b>.


### &lt;synset id=""/&gt;

Synset je zoznam synoným slova.

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

Príklad: **chcel by si ma mať**.
Jedno sloveso, že hej? No ale v skutočnosti tam máme slovesá tri: **chcel, si, má** + neurčitok **mať**, ktorý
je zároveň neurčitok a zároveň môže byť podstatné meno **mať - matka**. Divočina.

Gramatika heuristicky hľadá komplexné slovesá, ktoré prevádza do tvaru:
* **aux**: rozšírenie
* **verb**: nosné sloveso
* **inf**: neurčitok

Slovesá majú viac rôznych významov, podľa toho, či sú modálne a plno/neplnovýznamové. LightNet generuje
všetky možnosti, na ktoré príde. Príklady.

```
sloveso: môžeš(neplnoýznamove) vidieť

1. aux(môžeš) verb(vidieť)
2. verb(môžeš) inf(vidieť)
interpretácia: možeš vidieť, vidíš

sloveso: môžeš(neplnoýznamove) chcieť(neplnovýznamové) vidieť

1. aux(môžeš) verb(chcieť) inf(vidieť)
interpretácia: možeš chcieť vidieť, chceš vidieť
```

Ak sa v slovese vyskutujú neplnovýznamové slovesá, ich význam sa prenáša.

```
môžeš/musíš/chceš/máš/... vidieť = vidíš
vidíš pracovať != pracuješ .. vidíš je plnovýznamové, neprenáša význam
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

trafí:
    možeš/chceš/musíš/ .. vidieť
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

povinný aux .. trafí:
    možeš vidieť
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
    možeš vidieť robiť
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

Zložité. Viem. Ale to len na prvé poučutie. Užitočnosť to vyváží :)

### Sekvencie

Pattern je možné vybudovať zdola-nahor pomocou sekvencií, ktoré je možné
do seba skladať. Sekvencia môže obsahovať viac rôznych patternov.

```
<sequence id="xolution">
</sequence>
```
