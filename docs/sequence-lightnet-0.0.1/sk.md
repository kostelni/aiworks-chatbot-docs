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

### &lt;lemma tags=""/&gt;

Reaguje na základný tvar slova a gramatické tagy - rod, číslo a pád. Ak tagy nie sú definované, zaberie na akýkoľvek tvar slova.


```
<lemma>robot</lemma>
trafí všetko: robot, robota, roboti, robotov, ..

<lemma tags="fall3, singular">robot</lemma>
trafí len: robotovi


```

**Tagy pre rod:** person1, person2, person3
**Tagy pre číslo:** singular, plural
**Tagy pre pád:** fall1, fall2, ..., fall7


### Sekvencie


Sequence LightNet je gramatika. Pattern je možné vybudovať zdola-nahor pomocou sekvencií, ktoré je možné
do seba skladať.
