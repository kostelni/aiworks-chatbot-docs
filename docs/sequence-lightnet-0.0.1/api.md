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

V tomto momente je potrebné POST-nuť konfiguráciu pri každom štarte servisu (WAR-ka).
Keď prídeme na to, ako pohodlne nastaviť env. premennú s cestou ku konfiguraku, zautomatizujeme to.

### Aktuálna konfigurácia

Vráti aktuálnu konfiguráciu znalostí. Toto nepotrebujeme automatizovať, momentálne je to len informačný
servis pre manuálnu kontrolu.

```
GET: /config
```

Odpoveď pre nenakonfigurované znalosti:

```
{
    "error": "no knowledge"
}
```

Odpoveď pre nakonfigurované znalosti:
```
{
  "bot-config": {
    "morphology": "cesta/k/adresaru",
    "ner": "Some(cesta/k/adresaru)",
    "knowledge": ["cesta/k/adresaru", "cesta/k/adresaru", ...]
  },
  "bot-knowledge": {
    "configs": 2,
    "knowledge": [
      {
        "layers": [
          "phrase",
          "verb",
          "verbs",
          "complex-verb",
          "sentence"
        ],
        "key": "syntax"
      },
      {
        "layers": ["rules"],
        "key": "rules"
      }
    ]
  }
}
```

### (Re)konfigurácia znalostí

Momentálne je tento servis potrebné zavolať pri každom štarte WAR-ka. Nastaví cesty ku
morfologickému a NER indexu, nastaví cesty ku adresárom so znalosťami (XML markup s intentmi)
a reloadne aktuálny znalostný model.

Ak sa obsah znalostí zmení, tento servis je potrebné použiť ako reload. Kompletne refreshne
(cho)bot-ovi mozgovňu. Takže žiadny redeploy WAR-ka. Len tento reload.

```
POST: /config

encoding: application/json

payload:
{
  "morphology": "cesta/k/adresaru",
  "ner": "cesta/k/adresaru",
  "knowledge": [
    "cesta/k/adresaru", "cesta/k/adresaru", ...
  ]
}
```


## (Cho)Bot Response

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
    "input": "veta 1 ? veta 2 !"
}

bot v odpovedi vráti session-id, ktoré klient musí použiť. v každom ďalšom dialógu potom:
{
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




