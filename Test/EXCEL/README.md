# Excel .xlsx wizard examples

Cartella di test per implementare il wizard di configurazione dei tracciati Excel, inclusi:

- range semplici letti da `A1`;
- range semplici con `startRow`;
- range senza header e mapping manuale;
- listObject / tabelle strutturate Excel;
- listObject posizionati lontano da `A1`;
- più listObject nello stesso foglio;
- sheet non rilevanti da ignorare.

## Convenzioni

- `startRow` è 1-based, cioè usa la numerazione righe di Excel.
- Se `listObjectName` è valorizzato, il parser deve usare i bounds della tabella strutturata Excel.
- Se `listObjectName` è valorizzato, `startRow` non serve.
- Se `listObjectName` è assente, il parser legge da `A1`, oppure da `A{startRow}` se `startRow` è valorizzato.
- Con `headerPresence=true`, la prima riga del range/tabella è usata come header.
- Con `headerPresence=false`, il mapping è manuale e va usato `columnOrder`.
- Per i listObject con `headerPresence=false`, la tabella Excel ha comunque una header row tecnica; il parser deve leggere il data body e mappare per posizione, non per nome header.

## Output atteso

- Record totali attesi: 12
- Colonne canoniche attese: `customer_id, first_name, last_name, email, country, amount, signup_date, active`

Confronta il risultato del parser con:

- `expected_output_normalized_utf8.csv`
- `expected_output_normalized.xlsx`

Il CSV include anche `test_case` e `source_file` per rendere più semplice capire da quale file proviene ogni record.
