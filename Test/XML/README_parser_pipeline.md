# XML parser pipeline test pack

Questo file estende i test del wizard XML per il task di implementazione del parser nella pipeline di import.

## Contenuto

- `01_*.xml` ... `07_*.xml`: file XML positivi già utili per verificare XPath, namespace, mapping elementi e attributi.
- `manifest_config_and_expected.json`: configurazioni positive e righe attese.
- `expected_output_normalized_utf8.csv`: output cumulativo positivo atteso.
- `parser_negative_cases_config_and_expected.json`: configurazioni negative pensate per validare `ParseError[]`.

## Copertura positiva

I test positivi coprono:

- input XML letto come bytes/MemoryStream;
- `xPathSelector` su nodi semplici e annidati;
- mapping da attributi con `attributeMapping`;
- mapping da elementi figli con `nodeMapping` relativo;
- namespace prefissati;
- default namespace mappato tramite prefisso artificiale;
- più namespace nello stesso documento;
- XPath selector con predicato;
- nodi opzionali mancanti o vuoti.

## Copertura negativa / ParseError[]

Il file `parser_negative_cases_config_and_expected.json` aggiunge casi per:

- XPath selector non valido;
- prefisso namespace non dichiarato;
- selector valido ma nessun nodo trovato;
- nodeMapping relativo mancante;
- attributeMapping mancante;
- nodeMapping XPath non valido;
- uso di prefissi con `namespaceSupport=false`;
- namespaceMap con URI errato.

## Nota su error handling

Per mapping mancanti su una singola colonna, il comportamento raccomandato è produrre comunque la riga, valorizzare la colonna mancante con stringa vuota/null e aggiungere un `ParseError` riferito a riga e colonna.
Se la pipeline è progettata come fail-fast, lo sviluppatore può adattare l'asserzione, ma questi casi servono comunque a validare la gestione degli errori.
