# JSON Wizard Examples

Pacchetto di test per il task:

> Implementare il wizard di configurazione per tracciati di tipo JSON, con path-driven extraction per identificare il nodo array da iterare e mappare le proprietà sulle colonne del tracciato.

## Colonne canoniche attese

```text
employee_id, first_name, last_name, email, department, city, country, salary, hire_date, active, source_system
```

## Semantica ufficiale dei parametri

L’ordine logico da implementare nel wizard/parser è:

```text
rootPath -> nodeSelector -> arraySelector opzionale -> propertyMapping
```

- `rootPath`: JSONPath valutato sul documento JSON completo. Risponde a: “Da dove parto nel JSON?”.
- `nodeSelector`: JSONPath valutato relativamente al risultato di `rootPath`. Risponde a: “Quali elementi devo considerare come record/righe di primo livello?”.
- `arraySelector`: opzionale. Se valorizzato, viene valutato dentro ciascun nodo selezionato da `nodeSelector`. Risponde a: “Se dentro ogni nodo c’è un array annidato, quale array devo aprire?”. Ogni elemento dell’array aperto diventa la riga finale. Se vengono trovati più array, il risultato atteso è il flatten/concatenamento degli elementi.
- `propertyMapping`: JSONPath relativo all’oggetto finale che diventa riga. Se `arraySelector` è assente, l’oggetto finale è quello selezionato da `nodeSelector`; se `arraySelector` è presente, l’oggetto finale è ogni elemento dell’array aperto.

Assunzione di dialetto JSONPath: sintassi tipo Jayway/Newtonsoft (`$`, dot notation, `[*]`, predicati `?(@.field == value)`). Se la libreria scelta usa un dialetto differente, adattare le espressioni mantenendo invariato l’output atteso.

## File inclusi

| File | rootPath | nodeSelector | arraySelector | Record attesi | Cosa testa |
|---|---|---|---|---:|---|
| `01_data_array_simple.json` | `$.data` | `$[*]` | - | 2 | array diretto sotto `data` |
| `02_response_items_nested_rootpath.json` | `$.response.items` | `$[*]` | - | 2 | rootPath annidato |
| `03_payload_with_arrayselector.json` | `$.payload` | `$` | `$.employees` | 2 | oggetto wrapper: apre l’array `employees` dentro `payload` |
| `04_departments_nested_arrays.json` | `$.departments` | `$[*]` | `$.employees` | 3 | itera i reparti e apre l’array `employees` di ogni reparto |
| `05_node_selector_filter_active.json` | `$.employees` | `$[?(@.active == true)]` | - | 2 | filtro su nodeSelector |
| `06_missing_null_optional_properties.json` | `$.data` | `$[*]` | - | 2 | proprietà mancanti/null/vuote |
| `07_root_selector_nonstandard_property_names.json` | `$` | `$.result.records[*]` | - | 2 | mapping da struttura non standard |

Totale record attesi: **15**.

## Esempio di caso nested

Per `04_departments_nested_arrays.json` la configurazione corretta è:

```json
{
  "rootPath": "$.departments",
  "nodeSelector": "$[*]",
  "arraySelector": "$.employees"
}
```

Interpretazione:

```text
1. Parto da $.departments.
2. Itero ogni department con $[*].
3. Dentro ogni department apro $.employees.
4. Ogni employee diventa una riga.
5. propertyMapping viene applicato al singolo employee.
```

## File di supporto

- `manifest_config_and_expected.json`: configurazione completa + expected rows per ogni file.
- `configs_only.json`: solo parametri e mapping da usare nel wizard.
- `expected_output_normalized_utf8.csv`: output cumulativo atteso.
- `expected_*.csv`: output atteso per singolo file.

## Risultato atteso

Il wizard/parser configurato con i parametri del manifest deve produrre righe normalizzate come dizionari `fieldName -> rawValue` sulle colonne canoniche.

Controlli importanti:

- il file `05_node_selector_filter_active.json` deve escludere il record `IGNORED001`;
- il file `04_departments_nested_arrays.json` deve restituire 3 righe, flattenando gli array `employees` dei reparti;
- il file `06_missing_null_optional_properties.json` non deve fallire su proprietà mancanti o `null`;
- il file `07_root_selector_nonstandard_property_names.json` deve dimostrare che il mapping proprietà può rimappare strutture diverse sulle stesse colonne.
