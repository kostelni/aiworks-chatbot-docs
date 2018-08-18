# Changelog by version

## In next version
* simplified external service to be used from java
* updated log file format to be chewable on windows
    windows can not handle doubledot in file name
* datatype extractor now recognizes also negative number
* pattern combinator extended to handle also permutations
* updated to newest lucene version, after update, all corpuses and indexes must be reindexed
* redesigned rule stemmer
* updated default NER indexer to store root entity document with data and each alternative in separate doc
  with reference to root entity document,
  updated default NER extractor to use stemmed wildcarded complex phrases instead of simple and inefficient term occurence