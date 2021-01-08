Link al corso https://app.pluralsight.com/library/courses/angular-ngrx-getting-started

Introduzione
	Intro
		NgRx implementa il pattern Redux x gestire lo stato di una app 
		Che cos's' lo stato
			- info da mostrare nella view
			- puo' essere info dell'utente loggato
			- entity data, per es. dati di prodotti recuperati dal BE e manipolati a FE
			- filtri impostati dall'utente a livello dell'app
			- tutti altri dati che monitora' app
		NgRx consente centralizzare lo stato in cosi detto state container, gestire il suo aggiornamento e comunicare gli aggiornamenti dello stato
	Che cos'e' NgRx?
		libreria di angular che implementa il pattern Redux
		su github https://github.com/ngrx/platform
		Redux pattern
			aiuta a gestire lo stato dell'app fornendo un one-way data flow in tutta l'app
			come funziona?
				-> View invia il controllo al relativo Component su un evento dell'utente -> Component eseguie il dispatch dell'evento verso relativa action
				(il meccanismo di dispatching invia i dati giusti alla action in base all'evento ricevuto dalla View, es. codice del prodotto / prodotto da aggiungere al carrello)
				-> la action si interfaccia con il Reducer che recupera lo stato attuale dell'oggetto dallo Store, aggiorna oggetto, e persiste oggetto aggiornato 
				all'interno di Store.
				Store - state container, e' un singleton
				Reducer - crea lo stato nuovo, NON modifica quello esistente! Quindi lo stato rimane sempre immutabile.
				Ogni Component si sottoscrive alle modifiche nello Store usando Selector, Selector contiene la logica di recupero dello stato dallo Store 
				Quando oggetto all'interno dello Store cambia, il Selector viene notificato, il Component percepisce la modifica e la View si aggiorna usando il meccanismo
					di data binding di angular
	Cosa risolve NgRx?
		la gestione dello stato di una app
		abbiamo il punto centrallizzato dove viene salvato lo stato
		senza usare NgRx, il Component che ha bisogno di dati, ogni volta che viene creato, dovrebbe usare il servizio che effettua la chiamata al BE di recupero dati,
			se i dati non cambiamo frequentemente, abbiamo N chiamate al BE che ritornano gli stessi dati
		disacoppia la logica di aggiornamento dello stato tra Component e State, il component non aggiorna mai direttamente i dati nello Store, idem per aggiornamento della View,
			il component rimane in ascolto sulle modifiche dello State usando i Selector
		esiste il plugin x Redux x Chrome, x fare il debugging di aggiornamento dello stato (time traveling)
		quando usare NgRx
			- quando dobbiamo gestire lo stato a livello dell'app
			- quando abbiamo tante chiamate pesanti al BE, possiamo usare client-side cache fornita dallo Store
			- quando dobbiamo notificare gli aggiornamenti dello stato verso piu' Component
		quando non usare NgRx
			- quando l'app e' semplice
			- quando i dev non sono famigliati con Angular e RxJS
	Some tips
		https://github.com/DeborahK/Angular-NgRx-GettingStarted/blob/master/CHANGELOG.md
		vedi demo app 
Redux pattern
	Intro
		Redux e' il modo x implementare cosi detto 'predictable state container' per le app JS
		E' il modo x organizzare la gestione dello stato in una app SPA
		Si basa sui 3 concetti fondamentali
			- Esiste solo un punto dove viene mantenuto lo stato, lo Store
			- lo stato e' in sola lettura, x modificarlo e' necessario fare il dispatching di una action
			- lo stato viene modificato usando le funzioni Reducer
	Store
		e' un oggetto JS che contiene lo stato
		cosa da non mettere nello store
			- lo stato non condiviso con piu' componenti
			- le view, gestite gia' dal framework
			- stato non serializzabile (es. routing)
	Actions
		usate per inviare la richiesta di aggiornamento stato (es. login, side menu action, recupero dei dati che aggiornato piu' componenti dell'app)
		l'aggiornamento dei dati avviene sostituendo completamento oggetto nello Store
		una Action ha 
			- il tipo (identifica il tipo dell'azione)
			- payload (facoltativo)
	Reducer
		rappresente in che modo cambia lo stato in base alla action
		riceve action e payload contenente i dati utili all'aggiornamento dello stato
		NOTA: qualche action puo' avere 'side effect', quindi NON passa da Reducer per aggiornare lo stato
			in angular esiste NgRx Effects library x gestire questi 'side effects'
		Reducer e' una funzione JS pura (NON modifica niente al di fuori della fn stessa!), che accetta lo stato attuale e action 
			in base alla action esegue un aggiornamento opportuno dello stato
	Vantaggi del pattern Redux
		- centalizza l'immutabilita' dello stato
		- piu' performante (es. in angular, i componenti si aggiornano usando il meccanismo OnPush)
		- testabilita' piu' semplice
		- tooling (debugging)
		- in angular e' possibile iniettare lo stati nei componenti dove serve
Primo contatto con NgRx (NgRx in action)
	Preparazione dell'app 
		repo github, vedi prima
		NOTA: app non ha il proprio BE, viene usata la libreria di angular 'in-memory-web-api'
	Installiamo lo Store
		NgRx e' composto da vari package, il package @NgRx/Store e' quello obbligatorio
		NOTA: lo Store viene mantenuto solo durante utilizzo dell'app (Local Storage)
		e' meglio organizzare lo stato in una struttura logica che rispecchia i requisiti di business dell'app
			(oggetto con proprieta' sensati, raggruppati in modo opportuno)
			si puo' dividere lo stato x funzionalita' dell'app
			possiamo avere delle proprieta' condivise a livello di tutta app (es. app, user, products) + proprieta' x funzionalita' (che si aggiungono nel momento di caricamento di tali, 
				es. in angular, se dividiamo l'app x moduli, ogni modulo puo' aggiugere allo stato la sua porzione)
				le porzioni dello stato sono chiamati SLICES
			il paragone puo' essere fatto con il DB, il modo in cui organizziamo nostri dati nelle tabelle
		installiamo il pacchetto digitando 'npm i @ngrx/store'
	Inizializziamo lo Store
		inizializzazione dello Store avviene usando il Reducer
		AppModule importa StoreModule, e mettiamo StoreModule.forRoot(reducer)  nell'array imports di @NgModule
		possiamo avere un singolo Store e un singolo Reducer x tutta l'app, ma e' meglio evitare
		MEGLIO organizzare il nostro stato x funzionalita' che rispecchia anche i moduli funzionali della nostra app (moduli angular)
		quindi, x creare lo stato, definiamo N Reducers, uno x ogni pezzo di funzionalita' (slice)
			es. RootReducer, ProductsReducer, UsersReducer, CustomersReducer
		in questo modo ogni Reducer e' piu' piccolo, che facilita la manutenzione, modifiche e testing
		in piu', lo stato non viene inizializzato x il modulo che non viene mai caricato
		x organizzare lo stato modulare usiamo la tecnica 'Feature Module State Composition'
			praticamento, in ogni definizione del modulo, usiamo StoreModule x configurare il reducer di riferimento
			il modulo root chiamata StoreModule.forRoot(reducer), il modulo di una funzionalita' chiama StoreModule.forFeature('nome funzionalita'/slice', reducer)
		NOTA: possiamo splittare ulteriormente lo stato anche all'interno di un modulo, es.
			StoreModule.forFeature('products', 
				{
					productList: listReducer,
					productData: dataReducer
				} 
			oggetto dello stato finale sarebbe: { 
				products: {
					productList: { ...}
					productData: { ...}
				}
	Demo - inizializzazione dello stato (vedi repo git)
	Definizione dello stato e action
		NOTA: e' importante prendere il tempo x raggionare sullo stato che vogliamo tracciare e le action che lo possono cambiare
	Costruzione del Reducer
		ricordiamo che un Reducer riceve in input lo stato precedente, e ritorna in output lo stato (la porzione dello stato associato al Reducer) nuovo
		in questo caso il cambiamento dello stato e' piu' esplicito
		Reducer es. 
			reducer(state, action) {
				switch (action.type) {
					case 'ACTION_1':
						...new state...
						break;
				}
				return new state
			}
		NOTA: Reducer deve rimanere un 'pure function', senza side effects! Non modifica niente al di fuori della stessa.
	Demo - costruzione di Reducer x processare una action
		definiamo una cartella 'state' a livello del modulo, conterra' le action e reducer
	Dispatching di action
		iniettiamo Store nel nostro componente 
		dopo chiamiamo this.store.dispatch nel metodo di component
	Subscribing allo Store x riceve gli aggiornamenti
		x leggere lo stato possiamo usare la fn this.store.select('slice'), oppure this.store.pipe(select('products')) x avere un Observable
Dev tools e debugging
	1. installare Chrome Redux Store Dev Tools
	2. installare npm i @ngrx/store-devtools (serve x esporre lo stato al plugin di chrome)
	3. inizializzare il modulo di store-devtools nell'app (AppModule, vedi demo, repo git)
		es.  StoreDevtoolsModule.instrument({
				  name: 'APM Demo Dev Tools',
				  maxAge: 25, -- numero di action salvate nella history
				  logOnly: environment.production
				})
Tipizzazione strong dello stato
	Intro
		'right type in the right time'
		la fn di reducer ha i parametri in input di tipo any
		idem quando facciamo subscribe allo store, la fn di callback ha il parametro in input di tipo any
		tipizzare lo stato ci fa risparmiare il tempo in bedugging
		useremo interfacce x tipizzare lo stato e action
	Disegnare interfacce x i slices dello state 
		mettiamo il suffissio State nel nome di interfaccia
		per ogni slice creiamo il suo interface
		per lo stato globale creiamo interface che aggrega tutti slice 
		mettiamo la definizione dell'interfaccia nello stesso file di reducer, interfaccia del corrispondente slice dello stato
	Fix lazy loading boundaris
		NOTA: quando disegnamo i tipi dello stato dobbiamo considerare il sistema modulare della nostra app, NON dobbiamo rompere la regola mantenendo i tipi dello stato locali al modulo
		vedi demo, git repo x un esempio
	Tipizzare lo stato
		tipizziamo i parametri nel reducer e a livello di component quando iniettiamo nel costruttore lo State (NOTA: importiamo lo stato globale ridefinito a livello del modulo,
			importazione stile typescript usando la sintassi 'as {namespace}' che ci consente di specificare il namespace per tipi importati, e' mandatorio, altrimenti andiamo in 
			conflitto con lo stato globale dell'app, che non contiene le slice della nostra funzionalita' (modulo)
	Impostare i valori iniziali dello stato
		inizialmente lo stato e' undefined (quando parte l'app)
		possiamo definire una costante di tipo initialState a livello di reducer
		successivamente modifichiamo il reducer mettendo il valore di default x il parametro state
		vedi demo
	Costruzione di selectors
		AS IS abbiamo  this.store.pipe(select('products')).subscribe
			- costante cablata a livello di component
			- diamo x scontato che la struttura dello state non cambia, se cambia, dobbiamo fare find e replace in N file
			- ci stiamo sottoscrivendo a tutte le modifiche dello slice products, anche se quello che ci interessa non cambia
		selector - sono delle query riutilizzabili verso il nostro Store
		benifici di utilizzare i selector
			- fornisce un API strongly typed
			- disaccoppia lo Store dal Componente
			- puo' incapsulare la trasformazione di dati, in modo che il Componente riceve gia' i dati in formato desiderato
			- sono riutilizzabili (piu' component possano accedere allo State nello stesso modo)
			- i valori ritornati da selector sono Memoized (cached), quindi sono in cache e non cambiano se non cambia lo Store 
		selector - e' una funzione, ci sono due tipi di selector forniti da NxRx
			1. createFeatureSelector, consente di ottenere la parte dello stato relativa alla funzionalita' desiderata, specificata con il nome (nome del modulo)
			2. createSelector, consente di comporre piu' selector in modo da ottenere la parte dello stato desiderata, qui siamo in grado di mettersi in ascolto sulla modifica solo di 
				una porzione dello stato di una funziona'/modulo
		NOTA: i selector devono rimanere delle fn JS pure, SENZA side effects 
		demo
	Comporre i selector
		usando createSelector di NgRx, ultimo parametro e' cosi detta projection function, che riceve in input i parametri che corrispondono ai selector definiti prima, ordine di parametri
			rispecchia l'ordine di definizione di selector
		vedi demo
		NOTA: cmq e' da preferire sempre l'utilizzo di selector x accedere allo Store, in modo di essere piu' indipendenti dalle modifiche dello Store, oppure di avere un refactoring 
			piu' 'piccolo'
Tipizzazione di action con Action Creators
	Intro
		ogni action contiene type e payload
		AS IS abbiamo payload non tipizzato, e il nome di action una costante ripetuta in piu' posti del codice
	Costruzione di Action Creators 
		benifici
			- prevenzione dagli errori
			- migliore utilizzzo di tool di sviluppo
			- documentazione di un insieme delle action valide
		per tipizzare le action dobbiamo
			1. definire i tipi di action come un set di costanti
			2. costruire action creators
			3. definire 'union type' x action creators
		es. possiamo avere queste action: 1. Toggle Product Code, 2. Set Current Product, 3. Clear Current Product, 4. Initialize Current Product
		usiamo enum x le costanti che rappresentano i tipi di action
		x avere una convenzione, i tipi di action possano avere i seguenti prefissi TOGGLE, SET, CLEAR, INITIALIZE, nei nomi di costanti,
			invece il valore puo' iniziare con il nome di funzionalita', per es. [Product] / [Product List Page] / [Product API] ...
		un action creator invece e' una classe, con due props, type e payload
			classe implementa Action di NgRx
			definiamo il type come readonly e la inizializziamo con la costante che rappresenta il tipi di azione
			definiamo il constructor, con parametro public payload: di tipo opportuno alla action (es. boolean), in questo momento la nostra action e' strongly typed
		come ultima cosa definiamo Union Type x le nostre Action Creators
			es. export type ProductActions = ToggleProductCode | SetCurrentProduct | ClearCurrentProduct | InitializeCurrentProduct
	Demo
	Utilizzo di actions e selectors x una comunicazione tra i component
		usiamo le action e selectors x far comunicare tra piu' component 
		vedi la demo, non c'e' niente di nuovo, solo piu' action dispatching e subscribing
	Definizione action x operazioni piu' complesse
		es. chiamate ai servizi di BE, servono una catena di action 
		il flusso nel codice e' il seguente: componente viene caricato con dati assenti -> dispatch di action LOAD -> reducer gestisce la action chiamando il servizio di comunicazione con il BE 
			-> il servizio esegue la chiamata, nel momento di arrivo della risposta esegue il dispatch della action gestita dal component che di nuovo fa il fispatch della action LOAD_SUCCESS,
			gestita dal reducer che aggiorna lo stato -> i dati arrivano al componenente tramite il selector 
		vedi la demo
Lavorare con side effects
	Intro
		di solito e' da preferire il codice relativo a NgRx che non ha side effects
		ma non e' sempre possibile, per esempio quando dobbiamo chiamare API REST, per recuperare i dati
			in questo caso abbiamo side effects by design
		x gestire i side effects NgRx mette a disposizione la libreria @ngrx/effects 
	Cosa sono gli effetti?
		operazioni che interagiscono con stato esterno (es. iterazioni con i servizi che chiamano le API REST)
		e' meglio prevedere una gestione controllata di questi effetti esterni allo stato dell'app
		in questo modo abbiamo i nostri componenti piu' puliti
		NOTA: potevamo mettere le chiamate ai servizi sempre nel componente o reducer
			se mettiamo nel reducer, una chiamata assincrona non funzionera', i reducer sono funzioni pure by design!
		da questi considerazioni deriva la strada piu' corretta di gestire i side effects -> utilizzare effects 
			un effects prende in input la action, esegue l'operazione desiderata, e fa il dispatch di un'altra action sia nel caso OK che KO
		il flusso del codice sarebbe: component fa il dispatching di una action (es. Load) -> Reducer gestirsce la action e usa Effects x eseguire la chiamata all'API REST usando 
			un servizio angular opportuno -> il servizio esegue una chiamata assincrona all'API -> alla ricezione della response passa i dati all'Effects -> 
			Effects esegue il dispatching di un'altra action (es. LoadSuccess) passando i dati al Component -> component puo' esseguire un'altro dispatching gia' i dati x 
			aggiornare lo stato dell'app -> che una volta aggiornato lo stato, la modifica arriva anche al Component tramite il relativo Selector
	Perche' implementare una gestione di side effects
		1. Continuiamo ad avere il componente puro, con le logiche relative al FE (UI)
		2. Isoliamo side effects in un punto centralizzato
		3. E' piu' facile testare side effects, indipendentemente dal componente utilizzatore
	Definire un Effect
		un effect viene visto come un servizio angular 
			la classe con un suffisso Effects (es. ProductEffects)
			il costruttore inietta Actions di NgRx, che e' un Observable, quindi siamo in grado di rimanere in ascolto su tutte le richieste di dispatching nella nostra app
			il costruttore inietta anche il servizio responsabile delle chiamate alle API di BE
			viene definita una variabile del servizio con decoratore @Effect() - serve x registrare l'effetto nella libreria di NgRx/effects
				tale variabile ha il suffisso $ x far capire a noi dev che si tratta di un Observable
				tale variabile viene valorizzata in questo modo, es.
					@Effect()
					loadProducts$ = this.actions$.pipe(
						ofType(ProductActionTypes.Load),		-- serve x filtrare solo le action che ci interessano
						mergeMap(action => {					-- fa il merge di tutti Observable ritornati dalla chiamata al servizio di BE in un unico stream 
							this.productService.getProducts().pipe(
								map(products => (new LoadSuccess(products)))	-- ritorniamo la action con i dati ricevuti dal BE (aka dispatching di una nuova action)
							)
						})
					);
	Demo
		vedi repo git
	SwitchMap versus MergeMap
		il nostro servizio di BE ritorna un Observable
		quando viene fatto il dispatching della action verso lo stesso Effect, possiamo avere l'esigenza di annullare la chiamata al servizio fatta prima, e che e' ancora in corso
		mergeMap mergia Observable della Action con Observable del servizio nello stesso stream, e di solito puo' andare bene, ma non sempre
			possiamo avere il bisogno di usare l'operatore switchMap (NOTA: scelta di un operatore sbagliato puo' provocare le race conditions!!!)
			vediamo gli operatori che possiamo usare
				- switchMap: usato piu' raramente, cancella la subscription/richiesta corrente, se e' stato emmesso un nuovo valore, e puo' causare race conditions
					es: se e' in corso il salvataggio del prodotto e arriva un'altra richiesta di salvataggio, la prima viene abortita, cancellando il relativo salvataggio, 
						rischio di perdere i dati
					NOTA: da usare di solito con le richieste GET o le richieste cancellabili, come per esempio le ricerche!
				- concatMap: esegue le subscription/richieste in ordine ed e' meno performante, pero e' piu' sicuro
					NOTA: da usare con le richieste GET/POST/PUT dove l'ordine e' importante
				- mergeMap: esegue le subscription/richieste in parallelo
					NOTA: da usare dove l'ordine non ha importanza
				- exhaustMap: ignore tutte le richieste successive finche non viene completata quella corrente
					NOTA: da usare x esempio nella login, quando tutte le richieste successive non hanno importanza finche non viene procesata quella iniziale
	Registrazione di un Effect
		la registrazione e' simile a quella dello Store, ma al posto di passare il reducer, si passa array di effects
		idem per la registrazione a livello di modulo, si usa il metodo forFeature di EffectsModule
		demo
	Utilizzo di effects
		come scritto prima, facciamo il dispatching della action in modo che il reducer si interfaccia con effects per eseguire l'operazione desiderata
		aggiornamento al componente arriva tramite il selector dello store
		demo
	Unsubscribing dallo store
		ci sono due strade
			1. eseguire unsubscribe nel metodo ngOnDestroy, oppure definire una variabile che traccia se il component e' attivo o no, e successivamente usare l'operator takeWhile(componentIsActive)
				nel momento di utilizzo di Observable dello store
			2. usare aync pipe ( | async) a livello di template, in questo caso angular gestisce in autonomia le operazioni di subscribing e unsubscribing
		il primo approccio e' da preferire se a livello di componente dobbiamo manipolare in qualche modo la subscription, altrimenti usiamo async pipe
	Gestione delle eccezioni in effects
		viene aggiunta una nuova prop di tipo errore nello state
		viene creato il relativo selector x poter rimanere in ascolto sugli errori a livello di componente
		nell'effect, quando mappiamo il risultato della chiamata di nostro servizio di BE, in parallelo viene usato anche operatore catchError di RxJs
			NOTA: questo operatore NON ritorna un Observable, quindi, x ritornare la action contenente l'errore, usiamo operatore of()
		lato componente si mette in ascolto sulla action di errore usando il selettore dedicato
Esecuzione operazioni di update
	iter e' simile a quello di GET
	x ogni operazione async, abbiamo 3 action, es. Update, UpdateSuccess, UpdateError
	NOTA: il codice bolerplate che dobbiamo scrivere x ogni operazione FE-BE (es. Load, LoadSuccess, LoadFail) puo' essere ridotto, vedi sotto
	NOTA: se dobbiamo aggiornare un elemento all'interno della lista salvata nello state, dobbiamo costruire un array nuovo contenente item aggiornato.
		  si usa la fn map(), per mappare elementi in un array nuovo, mettendo al posto dell'item aggiornata, il valore nuovo (es. ritornato dal BE, presente nel payload della action)
		  vedi demo
	ricordiamo che il reducer e' una fn pura, senza side effects, state passato nel parametro e' IMMUTABILE!
Considerazioni architetturali
	Obiettivo e' di avere una lettura, prestazioni e risoluzione di bug migliori nel momento di crescita dell'app e il team di lavoro
	Organizzazione di cartelle per funzionalita' o funzione
		da preferire l'organizzazione per funzionalita' (piu' facile identificare le funzionalita')
	Container presentational component pattern
		NgRx porta la logica fuori dal componente (in reducer e servizi angular)
		possiamo classificare i componenti in due categorie
			1. Presentational component
				il focus su come le cose sono visualizzate (HTML e CSS)
				zero dipendenze (il costruttore non inietta niente, nessun servizio o store)
				usa @Input() per ricevere i dati e @Output() per emettere degli eventi
				puo' contenere altri component
				es: menu, barra di navigazione
			2. Container component
				e' concentrato su come le cose funzionino 
				poco o niente HTML e CSS
				ci sono dipendenze iniettate nel costruttore
				e' statefull, specifichiamo in che modo i dati sono caricati e in che modo cambiano
				spesso cono i componenti caricati durante la navigazione, associati alle rotte 
				puo' contenere altri component 
		vantaggi di presentational/container components
			- prestazioni, usando la strategia di aggiornamento OnPush si puo' evitare aggiornamento automatico di component se i dati marcati con @Input() non cambiano (vedi sotto)
			- riutilizzo di componenti (combinazine di componenti)
			- piu' facili da testare
		a livello della struttura di cartelle possiamo creare la cartella COMPONENTS contenente i componenti di presentazione e cartella CONTAINERS
			contenente i componenti container (struttura simile e' usata anche internamente a NgRx)
	Demo1
		spostiamo la logica dal componente product-list (lo rendiamo un presentational componente) nel componente product-shell (lo rendiamo container component)
		le proprieta nel product-shell mettiamo tutti di tipo Observable (questi dati passiamo al componente lista, e usiamo async pipe x gestire unsubscribe automaticamente)
		vedi il repo, il componente container praticamente usa solo lo store per fare il dispatch di eventi e rimanere in ascolto sull'aggiornamento di dati
		le proprieta' di tipo Observable passiamo al componente di presentazione, insieme ai gestori eventi 
	Demo2
		cambiamo il componente di presentazione aggiungendo le proprieta' marcati con @Input() e @Output()
		metodi per fare emit di eventi 
		togliamo async da html referenziando le proprieta che non sono piu' observable lato il componente di presentazione
		vedi repo 
	Change detection OnPush
		il pattern 'componenti container e di presentazione' consente sfruttare piu' facilmente la strategia di aggiornamenot OnPush di nostri componenti 
			questa strategia megliore le prestazione 
			in questo caso la modalita' di determinazione delle modifiche e' impostata a CheckOnce
			questo vuol dire che il componente si aggiorna solo se cambia il riferimento delle proprieta' @Input() o se viene cattura un evento di tipo onclick
				NOTA: deve cambiare proprio il riferimento dell'oggetto e NON il suo contenuto (es. una delle proprieta')
				NOTA: usando NgRx che applicato il concetto di immutable abbiamo una 'convivenza' di default
			di default la strategia e' di aggiornare tutti i componenti su eventi DOM / eventi HTTP / promisses etc..
				NOTA: se abbiamo una gerarchia di componenti 'profonda' e l'evento per esempio onclick avviene su un componente piu' interno nella gerarchia (il figlio senza figli),
					angular prende in considerazione tutto l'albero di componenti, partendo dal componente root e esegue il check per capire se deve aggiornare il componente per ogni
					singolo componente nell'albero (vedi slide)
				NOTA: se un componenente dell'albero ha la strategia di aggiornamento OnPush, nel caso descritto prima, angular skippa tale componente e tutti suoi figli nel 
					momento di check di aggiornamento componenti
	Creazione di Barrel usando i file Index.ts
		e' il modo per centralizzare le export in un singolo modulo 
		la convenzione e' di usare il file index.ts per definire barrel
		e' comune vedere il file index.ts nelle app NgRx, viene collocato nella cartella state 
			in questo file vanno a finire le interfacce dello state e selector
		benefici usare index.ts a livello di state
			diventa un API pubblica per lo stato di un modulo
			reducer rimane pulito, il file contiene solo la logica relativa all'aggiornamento dello state
			codice piu' pulito
Ultime note
	libreria angular ngrx-data - zero ngrx boilerplate 
	@ngrx/entity - aiuta a gestire le operazioni CRUD sulle entita'
		funzioni helper per gestire le collection di entita'
		permette di costruire action, selectors e reducer minimizzando il codice boilerplate
	@ngrx/schematics - per aggiungere a CLI le funzionalita' di NgRx (possiamo generare lo store, action, reducer e effects + feature, container, entity)
	@ngrx/router-store - per connettere lo store al routing
	ngrx-data, usando convenzioni e qualche configurazione, abbiamo le operazioni per le nostre entitia' senza creare esplicitamente action, reducers, selectors 
	
				
			
		
	