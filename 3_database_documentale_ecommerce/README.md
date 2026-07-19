
## 3. Database NoSQL Documentale

Il terzo database rappresenta il catalogo prodotti di un e-commerce, implementato su un database documentale (key-document).

Si tratta di sistemi NoSQL che memorizzano i dati sotto forma di documenti, generalmente in formato JSON/BSON, costituiti da coppie chiave-valore e identificati tramite un codice univoco. A differenza dei database relazionali che memorizzano i dati in tabelle predefinite (spesso richiedendo che un oggetto sia suddiviso su più tabelle), queste tipologie di database archiviano tutte le informazioni di un determinato oggetto in un singolo documento, il quale presenta potenzialmente una struttura unica. Un altro vantaggio risiede nel fatto che i database documentali non richiedono campi uniformi tra i diversi documenti, e questo consente di aggiungere nuove informazioni senza influire sulla struttura dei record già presenti. 

Non utilizzano normalmente operazioni di JOIN come i database relazionali, bensì adottano un approccio denormalizzato in cui i dati correlati vengono archiviati insieme all’interno dello stesso documento, anche se questo può comportare la duplicazione dei dati. Questa scelta consente migliori prestazioni di lettura e una maggiore semplicità in termini di recupero delle informazioni. Nel caso di un e-commerce, ad esempio, dati quali i dettagli del prodotto, la categoria, le immagini e le caratteristiche possono essere memorizzati nello stesso documento per velocizzare le operazioni di consultazione.

Questi database presentano uno schema flessibile, ovvero non richiedono uno schema predefinito o fisso; ciò li rende ideali per strutture dati dinamiche che si evolvono nel tempo senza richiedere modifiche strutturali massive, come quelle tipiche di un catalogo e-commerce.

È stato creato un indice in Elasticsearch denominato `catalogo_prodotti`, dove ogni documento rappresenta un prodotto, identificato univocamente tramite un ID proprio, contenente informazioni di varia natura:

| Campo | Tipo | Descrizione |
|---|---|---|
| `prodotto_id` | keyword | Identificatore univoco del prodotto |
| `modello` | keyword | Modello commerciale |
| `descrizione` | text | Descrizione testuale estesa del prodotto |
| `marca` | keyword | Marca |
| `categoria` | keyword | Categoria del prodotto |
| `prezzo` | float | Prezzo di vendita |
| `quantità_magazzino` | integer | Quantità disponibile a magazzino | 
| `disponibilità` | boolean | Stato di disponibilità |
| `tags` | keyword | Parole chiave per l'indicizzazione |
| `specifiche_tecniche` | text | Caratteristiche tecniche del prodotto |
| `recensioni` | text | Contenuto testuale delle recensioni |
| `punteggio_soddisfazione`| float | Punteggio assegnato dagli utenti |
| `valutazione_media` | float | Media delle valutazioni |
| `data_acquisto` | date | Data dell'acquisto |
| `data_recensione` | date | Data della recensione |

### Le query

*   È stata effettuata una ricerca full-text attraverso la clausola **match** sul campo `descrizione`. Elasticsearch analizza il contenuto testuale e restituisce i documenti nei quali compare il termine ricercato.
*   È stata utilizzata una query booleana per combinare due condizioni: una ricerca sulla categoria del prodotto e un filtro numerico sul prezzo. La clausola **must** contribuisce al punteggio di rilevanza (scoring), mentre la clausola **filter** applica un vincolo stringente senza influenzare il punteggio, ottimizzando le prestazioni.

In questo modo è possibile combinare efficacemente la ricerca testuale non strutturata e i filtri strutturati tipici di un catalogo e-commerce.

#### Visualizzazioni console su ElasticSearch (Cloud)

#### 1. Creazione dell'Indice e Mapping (visualizzazione soltanto della parte iniziale)
![Creazione Indice](./immagini/indice%20mapping.jpg)

#### 2. Struttura di un Documento JSON
![Esempio Documento](./immagini/esempio%20documento.jpg)

#### 3. Query e Output
![Screenshot Elastic](./immagini/query%201.jpg)
![Screenshot Elastic](./immagini/query%202.jpg)

