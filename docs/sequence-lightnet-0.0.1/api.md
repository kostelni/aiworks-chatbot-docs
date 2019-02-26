# Sequence LightNet ChatBot API


## Endpoint

URI servisu. Vždy dostupné ako:

```
endpoint/api

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


