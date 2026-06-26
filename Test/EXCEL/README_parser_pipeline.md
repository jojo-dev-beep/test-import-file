# Excel parser pipeline test pack

Estensione per testare il parser Excel della pipeline di import.

## Cosa coprono i file già presenti

- lettura da `sheetName`;
- lettura da range semplice da `A1`;
- `startRow` 1-based;
- `headerPresence=true`;
- `headerPresence=false` con mapping manuale/`columnOrder`;
- lettura da `listObjectName`;
- listObject lontano da `A1`;
- più listObject nello stesso foglio e scelta della tabella corretta.

## Cosa aggiungono i file nuovi

- `07_parser_errors_bad_cells_empty_rows.xlsx`: range con riga vuota e celle non convertibili.
- `08_parser_errors_listobject_empty_and_bad_cells.xlsx`: listObject con riga vuota e valore decimal non convertibile.
- `parser_negative_cases_config_and_expected.json`: configurazioni ed errori attesi per `ParseError[]`, inclusi foglio non trovato, righe vuote e celle non convertibili.

## Nota su celle non convertibili

Il task dichiara output `List<Dictionary<string,string>>`, ma richiede anche `ParseError[]: celle non convertibili`. Questo ha senso se `TraceColumnDefinition[]` include un datatype o una fase di conversione/validazione. Nel manifest negativo ho quindi indicato datatype per colonna.

## Expected positivo

Per i casi positivi confrontare con:

- `expected_output_normalized_utf8.csv`
- `expected_output_normalized.xlsx`

Record positivi attesi dal pacchetto originale: 12.


## Criterio aggiuntivo: celle formula risolte correttamente

Inclusi anche `09_formula_cells_resolved.xlsx` e `parser_formula_cases_config_and_expected.json`.

Questi casi verificano che il parser Excel legga il **valore calcolato** della cella formula e non il testo della formula. Per esempio:

- `=100+25.5` deve produrre `125.50`;
- `=ROUND(10/4,2)` deve produrre `2.50`;
- `=LOWER(...)` deve produrre l'email già calcolata;
- `=TRUE()` / `=FALSE()` devono produrre `true` / `false`;
- le formule devono essere risolte sia in un range semplice (`FormulaRange`) sia dentro una tabella strutturata Excel (`FormulaCustomersTable`).

Output atteso specifico: `expected_output_formula_cases_only_utf8.csv`.
Output atteso cumulativo con i casi formula: `expected_output_normalized_with_formula_cases_utf8.csv`.
