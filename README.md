# A.I. Works ChatBot working docs

Jadro modelovacieho jazyka je postavené na <a href="https://www.pandorabots.com/docs/" target="_blank">AIML</a> XML markup-e.

Základ jazyka bol rozšírený o nové operátory, hlavne pre morfológiu. Niektoré pôvodné operátory boli
zmenené a prispôsobené.

NLP je riešené ako sekvenčný klasifikátor jazykových vzorov. Vyhodnocovací mechanizmus hľadá
vždy najdlhší známy vzor.

## Kategória/Intent

Báza znalostí je modelovaná pomocou tzv. kategórií, jedna kategória reprezentuje
jeden intent.

Kategória reprezentuje reakciu na jazykový vzor. Má dve základné časti:
* **PATTERN**: Jazykový vzor, na ktorý táto kategória reaguje
* **TEMPLATE**: Samotná reakcia na pattern. Reakcia v template môže byť
komplexný XML kód.

```
<category>
  <template>
  </template>
</category>
```
