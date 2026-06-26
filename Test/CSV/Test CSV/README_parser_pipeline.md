# CSV parser pipeline test pack

Estensione che serve per testare il task:

> Implementare il parser CSV per la pipeline di import, guidato dai parametri del `TraceDefinition`.

## Output atteso del parser

Il parser deve produrre:

- `List<Dictionary<string, string>>`: una riga per record CSV valido, con `fieldName -> rawValue`;
- `ParseError[]`: errori di parsing con almeno riga e campo incriminato.

Le righe indicate nei file JSON sono **1-based**. Quando `headerPresence=true`, la riga 1 è l'header.

## Colonne canoniche

```text
customer_id, first_name, last_name, email, country, amount, note
```

## File positivi ereditati dal wizard

Questi file verificano i parametri base di parsing:

| File | Cosa testa |
|---|---|
| `01_comma_utf8_header_doublequote.csv` | delimiter virgola, UTF-8, header, doppi apici |
| `02_semicolon_win1252_header_singlequote.csv` | punto e virgola, Windows-1252, header, apice singolo |
| `03_tab_utf16_header_noqualifier.csv` | tab, UTF-16, header, nessun qualifier |
| `04_pipe_latin1_noheader_doublequote.csv` | pipe, Latin1, no header, mapping manuale |
| `05_custom_caret_utf8_noheader_noqualifier.csv` | delimiter custom `^`, no header, mapping manuale |
| `06_semicolon_utf8_noheader_singlequote.csv` | punto e virgola, no header, mapping manuale, apice singolo |
| `13_positive_escaped_quotes_and_multiline_utf8_header_doublequote.csv` | quote escapate e campo multilinea quotato |

Expected positivi:

- `expected_output_normalized_utf8.csv`: output dei 6 file originali;
- `expected_output_parser_positive_with_multiline_utf8.csv`: output dei 6 file originali + caso 13.

Configurazioni positive:

- `manifest_config_and_expected.json`: manifest originale wizard;
- `parser_positive_cases_config_and_expected.json`: manifest orientato al parser pipeline.

## File negativi per ParseError[]

| File | Errore atteso |
|---|---|
| `07_error_column_count_mismatch_utf8_header_doublequote.csv` | righe con campi mancanti / campi extra |
| `08_error_unclosed_quote_utf8_header_doublequote.csv` | text qualifier aperto e non chiuso |
| `09_error_missing_header_column_utf8.csv` | `TraceColumnDefinition` richiede una colonna assente nell'header |
| `10_error_empty_row_utf8_header_doublequote.csv` | riga completamente vuota |
| `11_error_wrong_encoding_win1252_configured_as_utf8.csv` | file Windows-1252 letto come UTF-8 |
| `12_error_noqualifier_delimiter_inside_field_utf8_noheader.csv` | delimiter dentro un campo quando `textQualifier` è assente |

Configurazioni/errori attesi:

- `parser_negative_cases_config_and_expected.json`

Expected delle sole righe valide presenti nei casi negativi:

- `expected_output_negative_cases_valid_rows_utf8.csv`

## Convenzioni consigliate per `ParseError`

I test usano queste convenzioni:

- `row`: numero riga fisica, 1-based;
- `fieldName`: nome campo canonico quando individuabile;
- `__row`: errore riferito all'intera riga;
- `__file`: errore riferito al file/decoding;
- `__extraColumns`: colonne extra senza mapping;
- `errorCode`: codice stabile, utile per assert automatici;
- `message`: messaggio descrittivo.

## Nota sui datatype

Questo task produce `rawValue` come stringa. Quindi i file CSV non testano conversioni numeriche/date/boolean: quelle dovrebbero essere validate nelle fasi successive (`ValidationPattern`, `ConversionRule`, `ImportBufferVertical`).
