# Test pack parser TXT_POSITIONAL pipeline

Questo file aggiunge i file TXT posizionali già usati per il wizard e aggiunge configurazioni/expected specifici per il parser della pipeline di import.

## Task coperto

Input:

- file TXT come `MemoryStream`
- `TraceDefinition.ParsingParameters` tipo `TXT_POSITIONAL`:
  - `encoding`
  - `lineTerminator`
- `TraceColumnDefinition[]`:
  - `startIndex`
  - `length`
  - `paddingChar`
  - `alignment`
  - `trimOnRead`

Output:

- `List<Dictionary<string, string>>`
- `ParseError[]` per righe con lunghezza insufficiente o formato non valido

## Convenzioni usate

- `startIndex` è **0-based**.
- Offset e lunghezze sono calcolati in **caratteri**, non in byte.
- Ogni record valido ha lunghezza **92 caratteri**.
- L'output del parser è raw string: non vengono fatte conversioni `Int`, `Decimal`, `DateTime` o `Boolean`.
- `trimOnRead=true` rimuove il padding dal lato coerente con `alignment`.
- `trimOnRead=false` conserva il campo completo estratto.

## File positivi consigliati per il parser pipeline

| File | encoding | lineTerminator | Cosa testa |
|---|---|---|---|
| `01_utf8_lf_char_offsets.txt` | UTF-8 | LF | offset in caratteri con accenti/emoji |
| `02_latin1_crlf_accents.txt` | Latin1 | CRLF | encoding Latin1 |
| `03_windows1252_crlf_euro_quotes.txt` | Windows-1252 | CRLF | encoding Windows-1252, euro e virgolette smart |
| `04_utf8_crlf_padding_trim.txt` | UTF-8 | CRLF | padding, alignment e `trimOnRead=false` |

Expected:

- `parser_positive_cases_config_and_expected.json`
- `expected_output_parser_positive_raw_utf8.csv`

## File con header

Sono inclusi anche:

- `05_utf8_lf_with_header.txt`
- `06_latin1_crlf_with_header.txt`
- `07_windows1252_crlf_with_header.txt`

Attenzione: il task pipeline indicato **non prevede** `headerPresence` o `skipRows`.
Questi file vanno usati solo se la pipeline dispone di una logica esterna per saltare la prima riga.
Se non esiste questa logica, non usarli come casi positivi del parser pipeline puro.

## File negativi aggiunti

| File | Cosa testa |
|---|---|
| `08_error_insufficient_length_utf8_lf.txt` | righe più corte del layout richiesto |
| `09_error_blank_and_control_chars_utf8_crlf.txt` | riga vuota, riga composta solo da spazi, carattere di controllo |
| `10_error_wrong_line_terminator_expected_crlf_but_lf.txt` | mismatch tra `lineTerminator` configurato e terminatori presenti nel file |
| `11_error_invalid_padding_alignment_utf8_lf.txt` | padding/alignment non coerenti, utile per parser strict |

Expected error:

- `parser_negative_cases_config_and_expected.json`

## Nota su datatype da verificare

I file wizard originali contengono anche `datatype` nel manifest, ma il task pipeline TXT_POSITIONAL che stai testando non lo prevede nella `TraceColumnDefinition[]`.
Per questo, nei file `parser_*_config_and_expected.json` vengono usati solo:

```json
{
  "startIndex": 0,
  "length": 6,
  "paddingChar": "0",
  "alignment": "right",
  "trimOnRead": true
}
```
