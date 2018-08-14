# A.I. Works ChatBot working docs

Jadro je postavené na jazyku <a href="https://www.pandorabots.com/docs/" target="_blank">AIML</a>.
Základ jazyka bol rozšírený o nové operátory, hlavne pre morfológiu. Niektoré pôvodné operátory boli
zmenené a prispôsobené.

NLP je riešené ako sekvenčný klasifikátor jazykových vzorov. Vyhodnocovací mechanizmus hľadá
vždy najdlhší známy vzor.

## Kategória/Intent

Báza znalostí je modelovaná pomocou tzv. kategórií, jedna kategória reprezentuje
jeden intent.

```
<category>
  <template>
  </template>
</category>
```
