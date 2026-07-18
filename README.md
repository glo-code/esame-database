# esame-database

Il progetto prevede la creazione di tre diversi paradigmi di database (Relazionale, NO SQL key document e NO SQL graph) per tre diverse tipologie di situazioni del mondo reale.

Il primo database rappresenta un noleggio di biciclette per un’azienda e tale mini-mondo è stato modellato con un database relazionale, in quanto si tratta di dati strutturati ed era necessario rispettare i vincoli di integrità per evitare inconsistenze nel database.
Senza il rispetto di tali vincoli, si potrebbero verificare situazioni che alterano lo stato valido del database, come ad esempio un noleggio associato a un cliente inesistente o una bicicletta non presente nel sistema..
Infatti tali tipologie di database garantiscono l’affidabilità assoluta delle transazioni, grazie allo standard ACID, oltre ad eliminare la ridondanza per via della proprietà di normalizzazione del database, organizzando i dati in più tabelle collegate da chiavi esterne.

La prima fase è stata la progettazione dello schema entità-relazione in base ai requisiti espressi dall’azienda.
Il modello concettuale, rappresentazione astratta e di alto livello dei dati, permette di definire le entità e le loro relazioni in modo indipendente dalla loro implementazione tecnica.
L’obiettivo di questa fase è rappresentare in maniera chiara e coerente i dati del mondo reale, le loro proprietà e le relazioni tra di essi.
Le entità sono:

bicicletta;
cliente;
punti di ritiro e riconsegna;
noleggio.

Noleggio è stato modellato come entità, in quanto oggetto reale con un’esistenza indipendente e con attributi specifici ed anche perché si tratta di un evento che si ripete nel tempo.
Noleggio non è stato modellato come entità debole perché possiede un identificatore proprio (artificiale) ID_Noleggio, che consente di identificarlo autonomamente e le relazioni previste sono state suddivise in relazioni binarie di tipo 1:N.
I punti di ritiro e riconsegna sono stati uniti in un’unica entità Punti di gestione per evitare di duplicare la stessa entità e tale entità riveste due ruoli (ritiro e riconsegna).
Anche cliente e bicicletta sono stati modellati come entità forti.

Gli attributi sono:

CLIENTE: id_cliente come identificatore univoco artificiale; nome e cognome attributi semplici obbligatori; telefono e email attributi facoltativi e semplici;
NOLEGGIO: id_noleggio come identificatore univoco artificiale; data e ora di inizio attributi semplici obbligatori affinchè il noleggio sia valido e data e ora di fine attributi semplici facoltativi (perchè il noleggio potrebbe essere ancora in corso) e importo totale è un attributo derivato da tariffa oraria presente nell’entità BICICLETTA e le date del noleggio;
PUNTO DI GESTIONE: id_stazione come identificatore univoco artificiale; nome attributo semplice obbligatorio; indirizzo attributo composto in sue sottocomponenti semplici obbligatori (Via, Civico e Cap, Città) per motivi di interrogazioni specifiche tramite query SQL;
BICICLETTA: ID_Bici come identificatore univoco artificiale; modello, tariffa oraria, stato di utilizzo e tipologia attributi semplici obbligatori.

E’ stata effettuata una specializzazione dell’entità BICICLETTA (superclasse) in quanto le entità seppur possiedono attributi comuni, alcune hanno caratteristiche uniche per cui bisogna definire un insieme di sottoclassi di un tipo di entità.
E’ un approccio dall’alto verso il basso in cui l’entità di livello superiore è specializzata in due o più entità di livello inferiore e questo consente una rappresentazione più specifica delle entità del mondo reale.
È una specializzazione basata sull’attributo tipologia ed è totale, in quanto ogni istanza della superclasse Bicicletta deve appartenere necessariamente ad almeno un sottotipo. 
È inoltre disgiunta, perché un’istanza del super-tipo può appartenere al massimo ad un solo sottotipo e non può essere contemporaneamente elettrica e muscolare.
Ogni bicicletta del noleggio deve essere per forza inserita in una delle due categoria e non esistono bici sia muscolari che elettriche.
Le classi figlie per via dell’ereditarietà, erediteranno le relazioni e gli attributi dell’entità padre.
Gli attributi delle classi figlie sono:

BICI_ELETTRICHE: la potenza del motore, stato della carica e tipi di pneumatici come attributi semplici obbligatori;
BICI_MUSCOLARI: il numero di rapporti, tipo di pneumatici come attributi semplici obbligatori.

Le relazioni sono:

CLIENTE effettua NOLEGGIO, con cardinalità 1-N, in particolare un cliente può effettuare da 0 a N noleggi nella sua vita e un noleggio appartiene a un solo e unico cliente (1,1).
BICICLETTA usata in NOLEGGIO, con cardinalità 1-N, in particolare una bicicletta può essere usata in 0 noleggi (se appena comprata) o in N noleggi nel tempo.
Un noleggio coinvolge sempre una sola e unica specifica bicicletta (1,1).
NOLEGGIO parte da RITIRO: un noleggio inizia necessariamente da un solo e unico punto di gestione mentre un punto di gestione può essere origine di 0 o N noleggi;
Noleggio finisce in RICONSEGNA: un noleggio termina in una sola e unica punto di gestione e un punto di gestione può ricevere la riconsegna di 0 o N noleggi;







Nella seconda fase, è stato effettuato il mapping e realizzato lo schema logico.
Il modello relazionale rappresenta i dati in tabelle o relazioni: ogni tabella è composta da righe (dette tuple o record) che rappresentano le singole istanze, e da colonne (dette attributi) che ne descrivono le proprietà.
Le entità del modello concettuale sono rappresentate sotto forma di tabelle e i nomi delle entità sono i nomi delle tabelle.
Le associazioni tra le tabelle vengono stabilite tramite l'uso condiviso di valori, implementando i vincoli di chiave primaria (Primary Key) e chiave esterna (Foreign Key), vincolo di integrità referenziale secondo il quale un valore inserito come chiave esterna deve necessariamente esistere nella tabella referenziata.

In questa fase, sono state identificate le chiavi primarie e la PRIMARY KEY garantisce automaticamente unicità e obbligatorietà del valore, impedendo duplicati e valori NULL.
Dell’attributo composto indirizzo di PUNTI DI GESTIONE sono state salvate soltanto le sue sottocomponenti semplici mentre l’attributo derivato importo totale in NOLEGGIO, si è scelto di memorizzarlo per via di cambiamenti possibili della tariffa oraria nel tempo e per lo storico dei noleggi e di impostarlo come facoltativo perchè calcolato al termine del noleggio, accettando una ridondanza contenuta.
La specializzazione si è scelto di non eliminare l’entità padre e tenere i figli in tabelle separate che presentano la chiave primaria del padre come chiave esterna, che funge contemporaneamente da chiave locale e da chiave esterna verso il padre.
Significa che una chiave può esistere nella tabella figlio solo se esiste già nella tabella padre.
 Per ricostruire tutti i dati di un figlio si fa un'operazione di EQUIJOIN tra il padre e il figlio.
Questa scelta permette di mantenere separati gli attributi specifici delle sottoclassi evitando colonne inutilizzate con valori NULL nella tabella padre, mantenendo inoltre una rappresentazione aderente al modello E-R.

Le relazioni essendo tutte binarie di cardinalità 1:N, si traducono aggiungendo la chiave primaria della tabella "lato 1" come chiave esterna nella tabella "lato n". 
Non sono necessarie tabelle di giunzione intermedie.
Nella terza fisica, durante la creazione dello schema fisico, sono stati aggiunti:

su alcuni campi dei vincoli UNIQUE per evitare valori duplicati;
 NOT NULL su campi considerati obbligatori per evitare inserimenti nulli;
CHECK per impedire valori non validi (es. sulle date del noleggio per garantire coerenza temporale);
 le politiche per la gestione dei vincoli di integrità.

Tali politiche sono:

ON DELETE CASCADE per tabelle Bicicletta_elettrica e Bicicletta_muscolare, questo permette la cancellazione automatica delle tuple dipendenti. 
Una bicicletta eliminata dalla tabella padre Biciclette viene automaticamente eliminata anche dalla tabella figlia corrispondente.
ON DELETE NO ACTION utilizzata nelle tabelle principali dello storico per impedire di eliminare informazioni necessarie per ricostruire lo storico dei noleggi.
ON UPDATE CASCADE utilizzata nelle chiavi esterne perchè permette di propagare automaticamente la modifica di una chiave primaria alle chiavi esterne collegate.


Per quanto riguarda invece le due query:

nella prima sono state effettuate operazioni di INNER JOIN per ritrovare i riferimenti tra la tabella NOLEGGIO e le tabelle CLIENTI e BICICLETTE tramite le chiavi esterne.
Tramite istruzione SELECT sono state mostrati nei risultati gli attributi nome e cognome della tabella CLIENTI per identificare i clienti e gli attributi modello della tabella BICICLETTE.
           Oltre a questo, tramite un’espressione di calcolo temporale è stata calcolata la durata 
           del noleggio ed infine è stato filtrato il risultato su un cliente fittizio tramite ID.

per la seconda sono state effettuate operazioni di INNER JOIN per ritrovare i riferimenti tra la tabella NOLEGGIO e la tabella PUNTI DI GESTIONE ma solo nel ruolo di ritiro.

E’ stata effettuata anche un’operazione di raggruppamento dividendo i dati in partizioni in base all’attributo ID_stazione di PUNTI DI GESTIONE e sapendo che in tal caso, tutte le colonne presenti nella clausola SELECT devono essere incluse nel GROUP BY oppure essere elaborate da una funzione di aggregazione, nel SELECT è stato messo ID_stazione e nome_stazione di PUNTI DI GESTIONE e la funzione di aggregazione COUNT che conta quante volte un punto di ritiro è stato utilizzato per un noleggio.
E’ stato incluso nome_stazione di PUNTI DI GESTIONE anche in GROUP BY per poterlo utilizzare anche in SELECT.

Il secondo database rappresenta un grafo della conoscenza su corsi online.
I nodi o oggetti discreti del dominio sono: 

gli studenti;
i corsi;
i docenti;
gli argomenti;
i certificati.

I nodi sono identificati tramite etichette (labels) che raggruppano le entità dello stesso tipo in insiemi differenti, per poter essere distinti e interrogate in modo efficiente dal database.
Le proprietà, informazioni aggiuntive sui nodi sono presenti soltanto nei nodi corsi, che sono descritti dal titolo, livello, durata e categoria. 

I tipi di relazione sono:

TIENE	Il docente tiene un corso
ISCRITTO_A	Lo studente è iscritto a un corso
TRATTA	Il corso tratta un argomento
PROPEDEUTICO_A	Un argomento è prerequisito di un altro
HA_CONSEGUITO	Lo studente ottiene un certificato
ACCREDITA               Il certificato accredita un corso

E’ stato utilizzato un DB a grafo, in quanto il contesto dei corsi online può essere un mini mondo altamente interconnesso e complesso.
Il database a grafo tratta le relazioni tra gli elementi dei dati come dati di prima classe, ovvero considerano le connessioni tra le entità importanti quanto le entità stesse e le relazioni vengono memorizzate in modo esplicito. 
I database relazionali invece forzano le relazioni in chiavi esterne e tabelle di JOIN, operazioni costose quando bisogna unire più tabelle per query indirette.
Di conseguenza nei database a grafo, le operazioni che richiederebbero più JOIN vengono  eseguite attraversando direttamente le relazioni che collegano i nodi, in quanto già memorizzate nel grafo.
Questo consente di accedere a milioni di connessioni al secondo. 
Un altro motivo è il fatto che i database a grafo permettono una maggiore flessibilità nel modello, non avendo un modello rigido sottostante.

In base alle relazioni e al tipo di grafi, bisogna dividere il database in diverse parti:

da una parte i nodi argomenti che sono prerequisiti di altri nei corsi (relazione argomento PROPEDEUTICO A argomento), che sono grafi aciclici diretti (DAG), ovvero grafi i cui bordi scorrono in una sola direzione, in cui non è possibile tornare al nodo di partenza seguendo il verso delle frecce e quindi non possono verificarsi cicli infiniti.
Questo perché la loro struttura è importante nella gestione di dipendenze circolari.
dall’altra parte gli studenti, corsi e argomenti che insieme rappresentano un grafo k-partito (considerando esclusivamente le relazioni di iscrizione e trattazione degli argomenti) , in quanto si tratta di un grafo i cui nodi possono essere divisi in tre sottoinsiemi disgiunti e indipendenti in modo tale che nessun arco unisce due vertici appartenenti allo stesso insieme.
i nodi docenti e certificati rappresentano altre tipologie di nodi del grafo di conoscenza.
 I docenti sono collegati ai corsi tramite la relazione TIENE, mentre i certificati rappresentano il completamento di un corso da parte di uno studente attraverso le relazioni HA_CONSEGUITO e ACCREDITA.
il modello rappresenta un grafo sparso, in cui manca la maggior parte dei possibili bordi nel grafico, rendendo il grafico relativamente vuoto in termini di connessioni di bordo.
La maggior parte delle reti del mondo reale come i grafi sociali sono sparse perché non tutte le persone sono connesse a tutte le altre persone. 
Motivo per cui una matrice di adiacenza, nel caso di grafi sparsi, rappresenterebbe un sacco di bordi inesistenti, mentre un elenco di adiacenza, cattura succintamente le relazioni tra i vertici, rendendo la memorizzazione ottimale.

Si sono posti due problemi:

trovare tutti i corsi collegati ad un certo argomento, direttamente o tramite prerequisiti;
individuare studenti simili perché hanno seguito corsi che condividono gli stessi argomenti.

Nel primo caso, per la risoluzione si è scelto l’algoritmo di ricerca in profondità (DFS), che visita sistematicamente tutti i vertici di un grafo o di un albero.
Tale algoritmo parte da un nodo radice arbitrario e ancora prima di aver visitato i nodi del primi livelli, può ritrovarsi a visitare vertici lontani dalla radice andando cosi in profondità e continua questo processo ricorsivamente fino a raggiungere un vicolo cieco.
Incontrato tale tipo di nodo, torna indietro al nodo inesplorato più vicino e ripete il processo finchè tutti i nodi non sono stati visitati.
E’ presente anche il set visitato, poiché i grafi possono contenere cicli un DFS deve ricordare quali nodi ha già esplorato e senza questo meccanismo, l’algoritmo potrebbe entrare in un ciclo infinito visitando ripetutamente gli stessi vertici, oltre a uno stack per memorizzare i nodi da visitare.

A differenza della BFS, che esplora i nodi di livello per livello, DFS dà priorità alla profondità rispetto all’ampiezza.
Il vantaggio di DFS è il basso sovraccarico di memoria rispetto a BFS, in quanto memorizza solo il percorso che sta attualmente esplorando cioè i nodi a livello di percorso corrente e non al livello attuale.
Tuttavia non garantisce il percorso più breve tra i nodi ma può essere utilizzata con i DAG, per eseguire l’ordinamento topologico.
L’ordinamento topologico ordina i vertici di un DAG in base alle relazioni di dipendenza in una sequenza lineare e DFS consente di calcolare questo ordine in modo efficiente (in post ordine inverso, l’opposto dell’elenco dei vertici nell’ordine in cui sono stati visitati l’ultima volta dall’algoritmo).
Ogni attività viene eseguita solo dopo che le cose da cui dipende sono completate e questo evita dipendenze contrastanti, garantendo un’esecuzione finita e prevedibile. 

Nel caso dei nostri dati fittizi, DFS ha riportato i seguenti risultati:



L’ordine prevede di intraprendere prima i corsi con argomenti di Matematica di base e di Python e successivamente Algebra è un prerequisito sia di Data Science che di Calcolo.
Machine learning ha come prerequisito l’argomento Data Science. 
Questo è il possibile ordine in cui gli argomenti dovrebbero essere studiati, rispettando tutte le relazioni di propedeuticità definite nel grafo

Nel secondo caso, si è optato per la categoria di algoritmi Community Detection, che aiuta a rivelare relazioni nascoste tra i nodi della rete e comunità.
Prima di applicare l'algoritmo di community detection, è stata costruita una proiezione monopartita dei soli nodi studente. 
Due studenti vengono collegati quando hanno frequentato corsi che condividono gli stessi argomenti. 
Su questo nuovo grafo è quindi possibile individuare gruppi di studenti con interessi simili.



Solo questi 4 studenti seguono corsi con gli stessi argomenti mentre non è presente uno dei 5 studenti presente nel database.

Sulla base dei risultati ottenuti con la proiezione, si è applicato l’algoritmo di Louvain Modularity, che divide il grafo in comunità o cluster nel modo più significativo.
Una divisione significativa è definita come quella in cui le connessioni tra i nodi all’interno di ciascun cluster sono significativamente più forti o più dense delle connessioni tra nodi in cluster diversi.
Un modo per valutare questo è il punteggio di modularità.
L'algoritmo tenta di trovare un partizionamento del grafico che massimizzi questo valore.
Un ampio punteggio di modularità indica che la proporzione di connessione osservata è significativamente più alta della probabilità di connessione casuale suggerendo che le connessioni all’interno della comunità sono forti e improbabili che si verifichino per caso.
Questo indica quindi la presenza di forti connessioni interne e di connessioni sparse tra le comunità.
L’algoritmo produce una struttura gerarchica di comunità ed è efficiente computazionalmente.
Questa efficienza si ottiene attraverso aggiornamenti locali piuttosto che attraverso l'ottimizzazione globale, il che sarebbe computazionalmente costoso.

L’algoritmo ha prodotto i seguenti risultati:





Nei nostri dati fittizi, la maggior parte dei nodi appartiene ad una comunità mentre un solo nodo appartiene ad un’altra comunità.

Il terzo database rappresenta un e-commerce di prodotti implementato su un database documentale (key-document).
Si tratta di sistemi NOSQL che memorizzano i dati sotto forma di documenti, generalmente in formato JSON/BSON, costituiti da coppie chiave-valore e identificati tramite un identificatore univoco.
A differenza dei database relazionali che memorizzano i dati in tabelle predefinite, spesso richiedendo che un oggetto sia suddiviso su più tabelle, tali tipologie di database memorizzano tutte le informazioni per un determinato oggetto in un singolo documento che ha potenzialmente una struttura unica.
Un altro vantaggio è che i database documentali non richiedono campi uniformi tra i documenti e questo consente di aggiungere nuove informazioni senza influire sulla struttura dei documenti già presenti.
Non utilizzano normalmente operazioni di JOIN come i database relazionali ma utilizzano un approccio denormalizzato in cui i dati correlati sono spesso archiviati insieme all’interno dello stesso documento, anche se questo può portare alla duplicazione dei dati.
Questo consente una migliore prestazione di lettura e semplicità in termini di recupero dei dati.
Nel caso di un e-commerce, ad esempio, informazioni come dettagli del prodotto, categoria, immagini e caratteristiche possono essere memorizzate nello stesso documento per velocizzare le operazioni di consultazione.

Sono schema flessibile cioè non richiedono a differenza dei database relazionali uno schema predefinito o fisso e questo rende tali database ideali per strutture dati dinamiche che si evolvono nel tempo senza richiedere modifiche allo schema del database come quelli di un e-commerce.
E’ stato creato un indice in ElasticSearch, catalogo_prodotti dove ogni documento rappresenta un prodotto del catalogo, ognuno identificato univocamente tramite un identificatore proprio e contiene informazioni di varia natura sui prodotti:

prodotto_id (keyword):identificatore univoco prodotto;
modello (keyword): modello commerciale;
descrizione (text): descrizione del prodotto;
marca(keyword):marca;
categoria(keyword):categoria prodotto;
prezzo (float):	prezzo di vendita;
quantità_magazzino (integer):quantità disponibile; 
disponibilità (boolean): stato disponibilità;
tags (keyword): parole chiave;
specifiche_tecniche(text) :caratteristiche tecniche del prodotto;
recensioni (text): contenuto delle recensioni;
punteggio_soddisfazione (float): punteggio assegnato dagli utenti;
valutazione_media (float): media delle valutazioni;
data_acquisto (date);
data_recensione (date).


Le query:

È stata effettuata una ricerca full-text attraverso la clausola match sul campo descrizione.
Elasticsearch analizza il contenuto testuale e restituisce i documenti nei quali compare il termine ricercato.
È stata utilizzata una query booleana per combinare due condizioni: una ricerca sulla categoria del prodotto e un filtro numerico sul prezzo. 
La clausola must contribuisce al punteggio di rilevanza, mentre filter applica un vincolo senza influenzare lo scoring.

In questo modo è possibile combinare ricerca testuale e filtri strutturati tipici di un catalogo e-commerce.
