# Sequence LightNet ChatBot API


## Endpoint

URI servisu. Vždy dostupné ako:

```
ENDPOINT/api

napr. pre lokálne bežiaci tool: ENDPOINT = http://localhost:9000/api
pre azure: ENDPOINT = http(s)://azure.wherever.runs/api
```

Keď budem popisovať servisy, pôjdem cez relatívne cesty ..

```
napr: /service
uri: ENDPOINT/service = http(s)://azure.wherever.runs/api/service
```


## Konfigurácia znalostí

Od verzie 0.0.5 vie servis pracovať s viacerými znalostnými bázami súčasne. Dajú sa spravovať
dynamicky, bez reštartu servisu. Create/Update/Delete.


### Aktuálna konfigurácia

Vráti aktuálnu konfiguráciu znalostí. Toto nepotrebujeme automatizovať, momentálne je to len informačný
servis pre manuálnu kontrolu.

```
GET: /config
```



### Inicializácia servisu

Kým to nezautomatizujeme, po štarte servisu je potrebné urobiť základnú konfiguráciu.
Tú stačí urobiť jediný raz. Konfigurák musí povinne obsahovať
len cesty ku indexom (morfologický, NER (ak sa používa)). Povinná je len cesta k morfologickému indexu.

V konfiguráku je možné inicializovať všetky znalostné bázy, s ktorými servis pracuje. Nie je to povinné,
dá sa to nakonfigurovať neskôr.

Povinný prvý init po štarte servisu:
```
POST: /config

encoding: application/json

payload:
{
  "morphology": "cesta/k/adresaru",
  "ner": "cesta/k/adresaru",
  "knowledge-bases": [
    {
      "id": "volitelne id",
      "folders": [
        "cesta/k/adresaru",
        "cesta/k/adresaru"
      ]
    },
    {
      "id": "volitelne id",
      "folders": [
        "cesta/k/adresaru",
        "cesta/k/adresaru"
      ]
    }
  ]
}

ako inicializácia zaberie aj toto (znalostné bázy sa doplnia neskôr):
payload:
{
  "morphology": "cesta/k/adresaru",
}

```

Servis je nakonfigurovaný. Kotrola:
```
GET: /config
```

### Create/Update znalostnej bázy

Za behu je možné urobiť reload existujúcej znalostnej bázy, alebo pridať novú.

```
PUT: /config

encoding: application/json

payload:
{
    "id": "volitelne id",
    "folders": [
        "cesta/k/adresaru",
        "cesta/k/adresaru"
    ]
}

```

Ak znalostná báza s *id* existuje, aktualizuje sa. Ak neexistuje, doplní sa nová. Re/inicializácia
sa urobí za behu, na servis netreba šahať.


### Delete znalostnej bázy

Za behu je možné existujúcu znalostnú bázu oddrbať zo servisu.

```
DELETE: /config/{id}
```

Znalostná báza *id* prestala v servise existovať.


## (Cho)Bot Response


### Od verzie 0.0.5 musí mať JSON payload každého requestu povinné kľúče [session-id] a [kb=id znalostnej bázy]

### Session

Core vstup/výstup. Bot si pre každú konverzáciu drží session kvôli kontextu. Session je identifikovaná
cez automaticky generované UUID. Pri prvom pokuse o dialóg vznikne nová session, konverzácia dostane
UUID, ktoré je pričapené ku každej odpovedi v kľúči **session-id**. Klient je povinný použiť pri
každom requeste pridelené **session-id**. Ak klient nepoužije **session-id**, konverzácia je prekvapivo považovaná
za novú a zakaždým sa vygeneruje nová session.

**session-id** samozrejme nie je povinné pri úplne prvom pokuse klienta o konverzáciu.

### Konverzácia

Vstup od používateľa je heuristicky rozdelený na vety. Vety sú tu chápané ako jednoznačne
oddelené pomocou: **? ! .**. Nie súvetia.

Bot vráti odpovede na všetky otázky položené na vstupe. Pre každú vetu jednu odpoveď, aj keď
sa pre odpoveď nenašiel žiaden intent.

#### Vstup
```
POST: /response

encoding: application/json

payload pri prvom dialógu:
{
    "kb": "id znalostnej bazy",
    "input": "veta 1 ? veta 2 !"
}

bot v odpovedi vráti session-id, ktoré klient musí použiť. v každom ďalšom dialógu potom:
{
    "kb": "id znalostnej bazy",
    "session-id": "uuid",
    "input": "veta 1 ? veta 2 !"
}
```

#### Odpoveď
```
{
    "error": false,
    "session-id": "vygenerovane uuid pre konverzaciu",
    "results": {
        "responses": [
            {
                "human": "veta 1",
                "bot": {
                    "response": "odpoved na vetu 1",
                    "triggers": "kontextové triggre",
                    "intent-id": "intent:matched-id"
                }
            },
            ...
            {
                "human": "vstup pre ktory nebol matchnuty intent",
                "bot": {},
            }
        ],
        "messages": [
            "tu budu spravy, ktory proces kolko trval",
            "ma to len informacnu hodnodu",
            "klient to ma ignorovat"
        ]
    }
}
```

Pre každú odpoveď je k dispozícii:
* **human** : veta zo vstupu
* **response** : odpoveď pre intent s najvyšším skóre
* **intent-id** : id intentu
* **triggers** : kontextové triggre

Momentálne, kým si to celé nevysvetlíme na workshope ťa zaujíma len:
* **human** : veta zo vstupu
* **response** : odpoveď pre intent s najvyšším skóre

Ak nebol trafený žiaden intent, objekt **bot** je prázdny.

Ak by sa niečo v moznovni pre daný request zdrbalo, dostaneš kľúč **error: true** a chybovú správu. This case should never happen.


## Working Memory

Informácia o aktuálnom stave kontextovej pamäte. Servis je určený len pre manuálnu kontrolu, nemám case,
kedy by sa toto malo akokoľvek automatizovať.

```
GET: /working-memory/{session-id}
```

Pre neexistujúce **session-id**:
```
{
    "error": true,
    "reason": "bot session for uuid [session-id] does not exist!"
}
```

Pre existujúce **session-id**:
```
{
    "error": false,
    "results": {
        "working-memory": {
            "entities": {
                "entity-name": "entity.toString"
            },
            "properties": {
                "property-name": "property value"
            }
        }
    }
}
```

### Preprocessor pre morfológiu a NER
Tool, ktorý vráti výstup morfologickej analýzy a NER (Named Entity Recognition). Znova, je to tool,
nie je určený pre automatizáciu.

```
POST: /preprocessor

encoding: application/json

payload:
{
    "kb": "id znalostnej bazy",
    "session-id": "uuid",
    "input": "slovo slovo slovo"
}
```

* **kb** : id znalostnej bázy
* **session-id** : používaj existujúce **session-id**, inak bude bot zbytočne vytvárať nové sessions.
    Pretože morfologický analyzátor a NER sú súčasťou existujúceho bot-a.
* **input** : medzerou oddelený zoznam slov na analýzu.

Príklad:
```
VSTUP:
{
    "kb": "id znalostnej bazy",
    "session-id": "uuid",
    "input": "robot si mala ja@tu.nie.som 26.02.2019"
}

VÝSTUP:
{
    "error": false,
    "session-id": "uuid",
    "results": {
        "input": "robot si mala ja@tu.nie.som 26.02.2019",
        "entities": [
            {
                "entity": "[EmailEntity (3 - 3) [classes: List(!email)][ja@tu.nie.som] : [ja@tu.nie.som]]"
            },
            {
                "entity": "[DateEntity (4 - 8) [classes: List(!date)][2019-02-26] : 26 . 02 . 2019]"
            }
        ],
        "words": [
            {
                "annotations": [
                    "[MorphAnnotation [role: nn] (lemma: Some(robot)): nn, singular, fall1, gender-man]",
                    "[MorphAnnotation [role: nn] (lemma: Some(robot)): nn, singular, fall5, gender-man]",
                    "[MorphAnnotation [role: nn] (lemma: Some(robot)): nn, singular, fall4, gender-man]",
                    "[MorphAnnotation [role: nn] (lemma: Some(robota)): nn, plural, fall2, gender-woman]"
                ],
                "word": "robot"
            },
            {
                "annotations": [
                    "[MorphAnnotation [role: prp] (lemma: Some(si)): reflexive]",
                    "[MorphAnnotation [role: vb] (lemma: Some(byt)): person2, vb, singular, affirmation, verb]",
                    "[MorphAnnotation [role: prp] (lemma: Some(byt)): reflexive]",
                    "[MorphAnnotation [role: vb] (lemma: Some(sit)): person2, vb, singular, affirmation, verb]"
                ],
                "word": "si"
            },
            {
                "annotations": [
                    "[MorphAnnotation [role: nn] (lemma: Some(malo)): nn, plural, fall4, gender-neuter]",
                    "[MorphAnnotation [role: jj] (lemma: Some(maly)): jj, singular, fall1, gender-woman]",
                    "[MorphAnnotation [role: nn] (lemma: Some(malo)): nn, plural, fall5, gender-neuter]",
                    "[MorphAnnotation [role: vbd] (lemma: Some(mat)): person3, singular, affirmation, gender-woman, verb, vbd]",
                    "[MorphAnnotation [role: nn] (lemma: Some(mala)): nn, singular, fall1, gender-woman]",
                    "[MorphAnnotation [role: jj] (lemma: Some(mala)): jj, singular, fall1, gender-woman]",
                    "[MorphAnnotation [role: nn] (lemma: Some(mala)): nn, singular, fall5, gender-woman]",
                    "[MorphAnnotation [role: nn] (lemma: Some(malo)): nn, plural, fall1, gender-neuter]",
                    "[MorphAnnotation [role: jj] (lemma: Some(maly)): jj, singular, fall5, gender-woman]",
                    "[MorphAnnotation [role: rb] (lemma: Some(mala)): ]",
                    "[MorphAnnotation [role: jj] (lemma: Some(mala)): jj, singular, fall5, gender-woman]",
                    "[MorphAnnotation [role: vbd] (lemma: Some(mat)): person2, singular, affirmation, gender-woman, verb, vbd]",
                    "[MorphAnnotation [role: vbd] (lemma: Some(mat)): singular, affirmation, gender-woman, verb, person1, vbd]",
                    "[MorphAnnotation [role: nn] (lemma: Some(malo)): nn, singular, fall2, gender-neuter]"
                ],
                "word": "mala"
            },
            {
                "annotations": [
                    "[MorphAnnotation [role: any] (lemma: Some(ja@tu.nie.som)): any]"
                ],
                "word": "ja@tu.nie.som"
            },
            {
                "annotations": [
                    "[MorphAnnotation [role: any] (lemma: Some(26)): any]"
                ],
                "word": "26"
            },
            {
                "annotations": [
                    "[MorphAnnotation [role: any] (lemma: Some(.)): any]"
                ],
                "word": "."
            },
            {
                "annotations": [
                    "[MorphAnnotation [role: any] (lemma: Some(02)): any]"
                ],
                "word": "02"
            },
            {
                "annotations": [
                    "[MorphAnnotation [role: any] (lemma: Some(.)): any]"
                ],
                "word": "."
            },
            {
                "annotations": [
                    "[MorphAnnotation [role: any] (lemma: Some(2019)): any]"
                ],
                "word": "2019"
            }
        ]
    }
}
```
