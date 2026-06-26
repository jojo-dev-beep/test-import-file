# JSON Parser Pipeline Test Pack

Pacchetto per il task:

> Implementare il parser JSON per la pipeline di import, con estrazione path-driven: il parser naviga la struttura JSON tramite rootPath per trovare l'array da iterare e mappa le proprietà di ogni oggetto sulle colonne del tracciato.

## Input/Output attesi

Input:

- file JSON come `MemoryStream`;
- `TraceDefinition.ParsingParameters` tipo JSON: `rootPath`, `nodeSelector`, `arraySelector`;
- `TraceColumnDefinition[]`: per ogni colonna, `propertyMapping` relativo alla proprietà.

Output:

- `List<Dictionary<string, string>>`, una riga per elemento estratto;
- `ParseError[]` per proprietà non trovata, tipo non atteso, `rootPath` non valido.

## Semantica ufficiale dei parametri

```text
rootPath -> nodeSelector -> arraySelector opzionale -> propertyMapping
```

- `rootPath`: JSONPath valutato sul documento completo. Definisce il contesto iniziale.
- `nodeSelector`: JSONPath valutato relativamente al risultato di `rootPath`; identifica i nodi/oggetti da considerare record o contenitori di record.
- `arraySelector`: opzionale. Se presente, viene valutato dentro ogni nodo selezionato da `nodeSelector`; deve restituire un array da aprire/flattenare.
- `propertyMapping`: JSONPath relativo all'oggetto finale che diventa riga.

Assunzione dialetto JSONPath: sintassi tipo Jayway/Newtonsoft (`$`, dot notation, `[*]`, predicati `?(@.field == true)`).

## File positivi già inclusi

Sono gli stessi del pacchetto wizard JSON v2:

| File | Cosa testa |
|---|---|
| `01_data_array_simple.json` | array diretto sotto `$.data` |
| `02_response_items_nested_rootpath.json` | rootPath annidato |
| `03_payload_with_arrayselector.json` | wrapper con array interno `employees` |
| `04_departments_nested_arrays.json` | `nodeSelector` sui reparti + `arraySelector` su `employees` |
| `05_node_selector_filter_active.json` | filtro JSONPath nel `nodeSelector` |
| `06_missing_null_optional_properties.json` | proprietà mancanti/null trattate come vuoto nei casi wizard/happy path permissivi |
| `07_root_selector_nonstandard_property_names.json` | struttura non standard mappata sulle colonne canoniche |

Expected positivo cumulativo: `expected_output_normalized_utf8.csv`.

Configurazioni positive per parser: `parser_positive_cases_config_and_expected.json`.

## File negativi aggiunti per ParseError[]

| File | Cosa testa |
|---|---|
| `08_parser_errors_property_and_type_mismatch.json` | proprietà mancante, propertyMapping che restituisce object, propertyMapping che restituisce array |
| `09_parser_errors_root_and_selector_type_mismatch.json` | `rootPath` che punta a un oggetto/stringa invece che a un array/nodeset iterabile |
| `10_parser_errors_arrayselector_not_array.json` | `arraySelector` che restituisce object o non trova l'array |
| `11_parser_errors_malformed_json.json` | JSON malformato, caso extra di robustezza |

Configurazioni negative ed expected error: `parser_negative_cases_config_and_expected.json`.

## Note sui ParseError

Il file `parser_negative_cases_config_and_expected.json` usa nomi errore descrittivi:

- `PropertyNotFound`;
- `UnexpectedType`;
- `InvalidRootPath`;
- `RootPathNotFound`;
- `InvalidJson`.

Se nel progetto avete enum diversi, mappare i nomi mantenendo invariati questi elementi minimi:

- path o mapping incriminato;
- `fieldName`, quando l'errore riguarda una colonna;
- indice riga logico, quando applicabile;
- messaggio diagnostico leggibile.

## Attenzione su proprietà mancanti

Nel pacchetto wizard il file `06_missing_null_optional_properties.json` è positivo e tratta proprietà mancanti/null come valore vuoto. Nel parser pipeline, se la regola di progetto richiede sempre un `ParseError` per proprietà non trovata, usare i casi `NEG01` e `NEG07` nel file negativo. In alternativa, distinguere colonne obbligatorie/opzionali nel `TraceColumnDefinition[]`.
