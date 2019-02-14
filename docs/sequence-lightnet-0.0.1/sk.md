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



```
```

### Sekvencie


Sequence LightNet je gramatika. Pattern je možné vybudovať zdola-nahor pomocou sekvencií, ktoré je možné
do seba skladať.
