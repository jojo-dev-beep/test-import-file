# Esempi TXT posizionali / fixed-width

Questa cartella serve per testare il wizard di configurazione per tracciati TXT posizionali.
Ogni record ha lunghezza fissa di **92 caratteri**. Gli offset sono **0-based** e misurati in **caratteri**, non in byte.

## File inclusi

| File | encoding | lineTerminator | record |
|---|---|---:|---:|
| `01_utf8_lf_char_offsets.txt` | UTF-8 | LF | 3 |
| `02_latin1_crlf_accents.txt` | Latin1 | CRLF | 2 |
| `03_windows1252_crlf_euro_quotes.txt` | Windows-1252 | CRLF | 2 |
| `04_utf8_crlf_padding_trim.txt` | UTF-8 | CRLF | 2 |



## File aggiunti con header row

Questi file contengono una prima riga di intestazione fixed-width, lunga anch'essa **92 caratteri**. La riga header usa gli stessi offset delle righe dati, ma non deve essere convertita nei datatype.

Configurazione consigliata per questi file:

```json
{
  "headerPresence": true,
  "skipRows": 1
}
```

Nota: nel fixed-width l'header serve solo per preview/lettura umana. Il mapping resta basato su `startIndex` e `length`.

| File | encoding | lineTerminator | righe header | record dati attesi |
|---|---|---:|---:|---:|
| `05_utf8_lf_with_header.txt` | UTF-8 | LF | 1 | 2 |
| `06_latin1_crlf_with_header.txt` | Latin1 | CRLF | 1 | 2 |
| `07_windows1252_crlf_with_header.txt` | Windows-1252 | CRLF | 1 | 2 |

Per validare solo questi casi, usare `expected_output_header_files_only_utf8.csv`.
Per validare tutti i file del pacchetto, usare `expected_output_normalized_utf8.csv`.

## Schema colonne

| Campo | startIndex | length | datatype | paddingChar | alignment | trimOnRead |
|---|---:|---:|---|---|---|---:|
| `customer_id` | 0 | 6 | Int | 0 | right | true |
| `first_name` | 6 | 12 | String | spazio | left | true |
| `last_name` | 18 | 15 | String | spazio | left | true |
| `amount` | 33 | 10 | Decimal | 0 | right | true |
| `signup_at` | 43 | 10 | DateTime | spazio | left | true |
| `active` | 53 | 1 | Boolean | spazio | left | true |
| `account_code` | 54 | 8 | String | 0 | right | false |
| `region_code` | 62 | 5 | String | spazio | right | true |
| `note` | 67 | 25 | String | spazio | left | true |

## Risultato atteso

Dopo parsing, trimming e conversione datatype, confrontare il risultato con:

- `expected_output_normalized_utf8.csv`

Note importanti:

- `account_code` ha `trimOnRead=false`, quindi gli zeri iniziali devono rimanere nel valore finale.
- `region_code` è allineato a destra con spazi a sinistra: con `trimOnRead=true` deve diventare, per esempio, `IT`, non `   IT`.
- `customer_id` e `amount` sono allineati a destra con zero-padding: gli zeri sono padding e vanno rimossi prima della conversione.
- `01_utf8_lf_char_offsets.txt` contiene caratteri multibyte UTF-8. Se il parser usa offset in byte invece che in caratteri, i campi successivi verranno letti male.
- `03_windows1252_crlf_euro_quotes.txt` contiene caratteri specifici Windows-1252 come `€` e virgolette tipografiche.
