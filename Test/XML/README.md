# XML Wizard Examples

Questa cartella contiene XML di esempio per testare un wizard di configurazione tracciati XML.

## Colonne canoniche attese

employee_id, first_name, last_name, email, department, country, salary, hire_date, active

## File inclusi

| File | namespaceSupport | xPathSelector | Record attesi | Cosa testa |
|---|---:|---|---:|---|
| `01_no_namespace_elements_and_attributes.xml` | `false` | `/Employees/Employee` | 2 | XML semplice senza namespace: selector diretto, mapping misto da attributi del nodo Employee e da elementi figli/annidati. |
| `02_nested_relative_xpath_and_attributes.xml` | `false` | `//Company/Employees/Employee` | 2 | XML senza namespace con elementi annidati: test dei nodeMapping XPath relativi tipo Name/First e Contact/Email. |
| `03_prefixed_namespace.xml` | `true` | `//hr:Employees/hr:Employee` | 2 | XML con namespace prefissato hr: namespaceSupport=true e namespaceMap obbligatorio per XPath selector e nodeMapping. |
| `04_default_namespace_requires_prefix_in_xpath.xml` | `true` | `//emp:Employees/emp:Employee` | 2 | XML con default namespace: nel file non c’è prefisso, ma l’XPath deve usare un prefisso dichiarato nel namespaceMap. |
| `05_multiple_namespaces.xml` | `true` | `//hr:Employee` | 2 | XML con due namespace: dati HR in hr: e data assunzione in meta:. Test namespaceMap multiplo e nodeMapping cross-namespace. |
| `06_xpath_selector_with_predicate.xml` | `false` | `//Department[@code="IT"]/Employee` | 2 | XML senza namespace con XPath selector che filtra il nodeset usando un predicato: deve leggere solo Department[@code="IT"]. |
| `07_missing_optional_nodes.xml` | `false` | `/Employees/Employee` | 2 | XML senza namespace con campi mancanti o vuoti: serve a verificare che il parser produca stringa vuota/null senza fallire. |


## Regole di parsing attese

- `xPathSelector` seleziona il nodeset da iterare.
- `nodeMapping` è un XPath relativo rispetto al nodo selezionato.
- `attributeMapping` legge un attributo XML del nodo selezionato.
- Se `namespaceSupport=true`, il parser deve valorizzare e usare `namespaceMap`.
- In XML con default namespace, l’XPath non può usare tag “nudi”: deve usare un prefisso dichiarato nel `namespaceMap`.
- I record non selezionati dal selector non devono comparire nell’output finale.
- I campi mancanti o vuoti sono rappresentati come cella vuota nel CSV atteso.

## Output atteso

- `expected_output_normalized_utf8.csv`: output cumulativo di tutti i file, con colonna `source_file`.
- `expected_<nome_file>.csv`: output atteso del singolo XML, senza colonna `source_file`.
- `manifest_config_and_expected.json`: configurazione da impostare nel wizard e record attesi per ciascun file.
