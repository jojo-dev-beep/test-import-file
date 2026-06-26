# CSV wizard examples

Questi file servono a testare il wizard di configurazione per tracciati CSV.
Ogni file copre una combinazione diversa di delimiter, textQualifier, encoding, headerPresence e columnOrder.

- `manifest_config_and_expected.json`: parametri da impostare nel wizard e risultato atteso per ogni file.
- `expected_output_normalized_utf8.csv`: output cumulativo atteso dopo parsing e mapping verso le colonne canoniche.

Colonne canoniche attese:

```text
customer_id, first_name, last_name, email, country, amount, note
```
