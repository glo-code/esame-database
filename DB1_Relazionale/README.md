
## 1. Database Relazionale

Il primo database rappresenta un servizio di noleggio di biciclette per un’azienda. 
Tale mini-mondo è stato modellato con un database relazionale, in quanto si tratta di dati strutturati ed era necessario rispettare i vincoli di integrità per evitare inconsistenze nel database.

Senza il rispetto di tali vincoli, si potrebbero verificare situazioni in grado di alterare lo stato valido del database, come ad esempio un noleggio associato a un cliente inesistente o una bicicletta non presente nel sistema. 
Questa tipologia di database garantisce l’affidabilità assoluta delle transazioni grazie allo standard ACID, oltre a eliminare la ridondanza per via della proprietà di normalizzazione, organizzando i dati in più tabelle collegate da chiavi esterne.

La prima fase ha riguardato la progettazione dello schema Entità-Relazione (E-R) in base ai requisiti espressi dall’azienda. 
Il modello concettuale, inteso come rappresentazione astratta e di alto livello dei dati, permette di definire le entità e le loro relazioni in modo indipendente dalla loro implementazione tecnica. L’obiettivo di questa fase è rappresentare in maniera chiara e coerente i dati del mondo reale, le loro proprietà e le relazioni tra di essi.

### Entità
Le entità individuate sono:
*   Bicicletta
*   Cliente
*   Punti di gestione
*   Noleggio

**Noleggio** è stato modellato come entità in quanto oggetto reale con un’esistenza indipendente, dotato di attributi specifici e basato su un evento che si ripete nel tempo. 
Noleggio non è stato modellato come entità debole poiché possiede un identificatore proprio (artificiale), `ID_Noleggio`, che consente di identificarlo autonomamente; inoltre, i legami previsti sono stati suddivisi in relazioni binarie di tipo 1:N.

I punti di ritiro e riconsegna sono stati uniti in un’unica entità, denominata **Punti di gestione**, per evitare di duplicare lo stesso oggetto; tale entità riveste quindi due ruoli distinti (ritiro e riconsegna).
Anche cliente e bicicletta sono stati modellati come entità forti.

### Attributi
Gli attributi sono così definiti:
*   **CLIENTE**: `id_cliente` come identificatore univoco artificiale; nome e cognome come attributi semplici obbligatori; telefono e email come attributi facoltativi e semplici.
*   **NOLEGGIO**: `id_noleggio` come identificatore univoco artificiale; data e ora di inizio come attributi semplici obbligatori affinché il noleggio sia valido; data e ora di fine come attributi semplici facoltativi (poiché il noleggio potrebbe essere ancora in corso); importo totale come attributo derivato, calcolato in base alla tariffa oraria presente nell’entità BICICLETTA e alle date del noleggio.
*   **PUNTO DI GESTIONE**: `id_stazione` come identificatore univoco artificiale; nome come attributo semplice obbligatorio; indirizzo come attributo composto dalle sue sottocomponenti semplici obbligatorie (Via, Civico, CAP e Città) per consentire interrogazioni specifiche tramite query SQL.
*   **BICICLETTA**: `ID_Bici` come identificatore univoco artificiale; modello, tariffa oraria, stato di utilizzo e tipologia come attributi semplici obbligatori.

### Specializzazione dell'entità BICICLETTA
È stata effettuata una specializzazione dell’entità BICICLETTA (superclasse) poiché, sebbene le entità possiedano attributi comuni, alcune presentano caratteristiche uniche per le quali risulta necessario definire un insieme di sottoclassi. 
Si tratta di un approccio dall’alto verso il basso (top-down), in cui l’entità di livello superiore viene specializzata in due o più entità di livello inferiore, consentendo una rappresentazione più specifica del mondo reale.

È una specializzazione basata sull’attributo tipologia ed è di tipo totale, in quanto ogni istanza della superclasse Bicicletta deve appartenere necessariamente ad almeno un sottotipo. 
È inoltre disgiunta, perché un’istanza del super-tipo può appartenere al massimo a un solo sottotipo e non può essere contemporaneamente elettrica e muscolare. 
Ogni bicicletta del noleggio deve essere per forza inserita in una delle due categorie, non esistendo mezzi sia muscolari che elettrici. 
Le classi figlie, per via dell’ereditarietà, ereditano le relazioni e gli attributi dell’entità padre.

Gli attributi delle classi figlie sono:
*   **BICI_ELETTRICHE**: la potenza del motore, lo stato della carica e i tipi di pneumatici come attributi semplici obbligatori.
*   **BICI_MUSCOLARI**: il numero di rapporti e il tipo di pneumatici come attributi semplici obbligatori.

### Relazioni
Le relazioni individuate sono:
*   **CLIENTE effettua NOLEGGIO ($1:N$)**: un cliente può effettuare da 0 a N noleggi nella sua vita, mentre un noleggio appartiene a un solo e unico cliente (1,1).
*   **BICICLETTA usata in NOLEGGIO ($1:N$)**: una bicicletta può essere usata in 0 noleggi (se appena acquistata) o in N noleggi nel tempo; un noleggio coinvolge sempre una sola e specifica bicicletta (1,1).
*   **NOLEGGIO parte da RITIRO**: un noleggio inizia necessariamente da un solo e unico punto di gestione, mentre un punto di gestione può essere origine di 0 o N noleggi.
*   **NOLEGGIO finisce in RICONSEGNA**: un noleggio termina in un solo e unico punto di gestione, mentre un punto di gestione può ricevere la riconsegna di 0 o N noleggi.

![Schema E-R](./DB1_Relazionale/immagini/schema%20E-R%20DEFINITIVO.png)

> 📝 **Nota sullo Schema E-R:**
> In una successiva fase di revisione del modello concettuale, è emerso che la relazione `STAZIONA IN` (inizialmente prevista tra le biciclette e i punti di gestione) risultava ridondante, per cui è stata eliminata nello schema logico e fisico. 

---

### Schema Logico

Nella seconda fase è stato effettuato il mapping ed è stato realizzato lo schema logico.
Il modello relazionale rappresenta i dati in tabelle (o relazioni): ogni tabella è composta da righe (dette tuple o record) che rappresentano le singole istanze, e da colonne (dette attributi) che ne descrivono le proprietà. 
Le entità del modello concettuale sono rappresentate sotto forma di tabelle e i nomi delle entità diventano i nomi delle tabelle stesse.

Le associazioni tra le tabelle vengono stabilite tramite l'uso condiviso di valori, implementando i vincoli di chiave primaria (Primary Key) e chiave esterna (Foreign Key). 
Quest'ultima esprime un vincolo di integrità referenziale secondo il quale un valore inserito come chiave esterna deve necessariamente esistere nella tabella referenziata.

In questa fase sono state identificate le chiavi primarie; la PRIMARY KEY garantisce automaticamente l'unicità e l'obbligatorietà del valore, impedendo duplicati e valori NULL. 
Dell’attributo composto indirizzo di PUNTI DI GESTIONE sono state salvate soltanto le sue sottocomponenti semplici. 
L’attributo derivato importo totale in NOLEGGIO si è scelto invece di memorizzarlo fisicamente, accettando una ridondanza contenuta, sia per far fronte a possibili cambiamenti della tariffa oraria nel tempo sia per preservare lo storico dei noleggi; è stato impostato come facoltativo perché calcolato solo al termine del noleggio.

Per quanto riguarda la specializzazione, si è scelto di non eliminare l’entità padre e di mantenere i figli in tabelle separate. 
Queste ultime presentano la chiave primaria del padre come chiave esterna, la quale funge contemporaneamente da chiave locale e da chiave esterna verso il padre. 
Ciò significa che una chiave può esistere nella tabella figlio solo se esiste già nella tabella padre. 
Per ricostruire tutti i dati di un figlio si rende necessaria un'operazione di EQUIJOIN tra il padre e il figlio. 
Questa scelta permette di mantenere separati gli attributi specifici delle sottoclassi, evitando colonne inutilizzate con valori NULL nella tabella padre e mantenendo una rappresentazione aderente al modello E-R.

Le relazioni, essendo tutte binarie di cardinalità 1:N, si traducono aggiungendo la chiave primaria della tabella "lato 1" come chiave esterna nella tabella "lato N", senza la necessità di tabelle di giunzione intermedie.

![Schema Logico](./immagini/Schema_logico_DEFINITIVO.png)

---

### Creazione Schema Fisico e Query SQL

Nella terza fase (progettazione fisica), durante la creazione dello schema fisico, sono stati aggiunti:
*   Vincoli UNIQUE su alcuni campi per evitare valori duplicati;
*   Vincoli NOT NULL sui campi considerati obbligatori per evitare inserimenti nulli;
*   Vincoli CHECK per impedire valori non validi (es. sulle date del noleggio per garantire la coerenza temporale);
*   Le politiche per la gestione dei vincoli di integrità.

Tali politiche sono:
*   **ON DELETE CASCADE** per le tabelle `Bicicletta_elettrica` e `Bicicletta_muscolare`, scelta che permette la cancellazione automatica delle tuple dipendenti. Una bicicletta eliminata dalla tabella padre `Biciclette` viene automaticamente eliminata anche dalla corrispondente tabella figlia.
*   **ON DELETE NO ACTION** utilizzata nelle tabelle principali dello storico per impedire l'eliminazione di informazioni necessarie a ricostruire lo storico dei noleggi.
*   **ON UPDATE CASCADE** utilizzata nelle chiavi esterne per propagare automaticamente la modifica di una chiave primaria alle chiavi esterne collegate.

Per quanto riguarda le due query:
1.  **Prima Query**: sono state effettuate operazioni di INNER JOIN per ritrovare i riferimenti tra la tabella `NOLEGGIO` e le tabelle `CLIENTI` e `BICICLETTE` tramite le chiavi esterne.
Tramite l'istruzione SELECT sono stati mostrati nei risultati gli attributi nome e cognome della tabella `CLIENTI`, per identificare i clienti, e l'attributo modello della tabella `BICICLETTE`.
Oltre a questo, tramite un’espressione di calcolo temporale è stata calcolata la durata del noleggio e, infine, è stato filtrato il risultato su un cliente fittizio tramite ID.
3.  **Seconda Query**: sono state effettuate operazioni di INNER JOIN per ritrovare i riferimenti tra la tabella `NOLEGGIO` e la tabella `PUNTI DI GESTIONE`, ma limitatamente al ruolo di punto di ritiro.
E' stata eseguita anche un’operazione di raggruppamento dividendo i dati in partizioni in base all’attributo ID_stazione di `PUNTI DI GESTIONE`.
Sapendo che in questo caso tutte le colonne presenti nella clausola SELECT devono essere incluse nel GROUP BY oppure essere elaborate da una funzione di aggregazione, nella SELECT sono stati inseriti ID_stazione, nome_stazione di `PUNTI DI GESTIONE` e la funzione di aggregazione COUNT, che conta quante volte un punto di ritiro è stato utilizzato per un noleggio.
È stato incluso nome_stazione di `PUNTI DI GESTIONE` anche nel GROUP BY per poterlo utilizzare nella SELECT.
