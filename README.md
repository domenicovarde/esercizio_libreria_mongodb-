
Progettare un database **NoSQL (MongoDB)** per una **libreria** con relazioni **molti-a-molti** richiede scelte intelligenti tra **embedding** e **referencing**, bilanciando prestazioni, leggibilità e coerenza.

dopo un'analisi del lavoro è stato elaborata e pensata l'ideazione di un database NoSQL che presenta le seguenti collezioni con i seguenti campi :

### ==1) collezione autori==  

in questa collezione andremo a inserire tutti gli autori dei libri presenti nella biblioteca
identificandoli con un id_autore univoco e inserendo i seguenti campi per ogni autore:

##### ==campi:==

	- id_autore
	- nome_autore 
	- data_nascita
	- nazionalita
	

==codice di esempio :==

	`{`
	  `"id_autore": ObjectId("..."),`
	  `"nome_autore": "Italo Calvino",`
	  `"data_nascita": "1923-10-15",`
	  `"nazionalità": "Italiana"`
	`}`


### ==2) collezione libri:==

in questa collezione andremmo semplicemente a inserire i libri presenti nella biblioteca che avranno un collegamento con gli utenti e gli autori, generando in entrambi i casi una relazione molti a molti tra le rispettive collezioni 

##### ==campi:==

	- id_libro
	- id_autore 
	- titolo 
	- anno_uscita
	- genere
	- id_utente
	- utenti_che_hanno_preso_in_prestito 
	{	   
		  "utente_id": ObjectId("..."),
	      "data_prestito": "2024-10-15",
	      "data_restituzione": "2024-11-15"
    }
	

==codice di esempio :==

	{
	  "id_libro": ObjectId("..."),
	   "id_autore": [ObjectId("..."),
	  "titolo": "Il barone rampante",
	  "anno": 1957,
	  "genere": "Romanzo"
	  "utenti_che_hanno_preso_in_prestito": [
	    {
	      "utente_id": ObjectId("..."),
	      "data_prestito": "2024-10-15",
	      "data_restituzione": "2024-11-15"
    }
	]
	}



### ==3) collezione utenti:==

in questa collezione andremmo a registrare  gli utenti  presenti nella biblioteca ovvero i cosi detti soci, andando a inserire un id univoco e delle ulteriori informazioni identificative per poi tenere traccia dei libri prestati.

##### ==campi:==

	- id_utente
	- id_utente 
	- libro_in_prestito
	}
		libro_id
		data_prestito
		data_restituzione
	}
	- email 
	- telefono
	- data_registrazione
	- libri_prestati:
	}
		   "utente_id": ObjectId("..."),
	      "data_prestito": "2024-10-15",
	      "data_restituzione": "2024-11-15"
    }
	

==codice di esempio :==

	`{`
	  `"_id": ObjectId("666abc1234567890abcdef01"),           // id_utente`
	  `"email": "mario.rossi@example.com",`
	  `"telefono": "+39 3456789012",`
	  `"data_registrazione": ISODate("2024-09-01T10:30:00Z"),`
	  
  `"libri_prestati": [`
    `{`
      `"libro_id": ObjectId("777def9876543210abcdef12"),   // riferimento al libro`
      `"data_prestito": ISODate("2024-10-15T00:00:00Z"),`
      `"data_restituzione": ISODate("2024-11-15T00:00:00Z")`
    `},`
    `{`
      `"libro_id": ObjectId("888ghi543210fedcba432109"),`
      `"data_prestito": ISODate("2025-01-10T00:00:00Z"),`
      `"data_restituzione": ISODate("2025-02-10T00:00:00Z")`
    `}`
  `]`
`}`

##  ==Relazioni==

###  ==Autori <-> Libri (**molti a molti**)==

- Un autore può aver scritto più libri.
    
- Un libro può avere più autori.
    

**Soluzione**: nella collezione `Libri`, usare un array di ID (`autori`) per riferirsi agli autori.  
Oppure, per query inverse, mantenere anche in `Autori`:

`"libri_scritti": [ObjectId("..."), ObjectId("...")]`



---

###  ==Utenti <-> Libri (**molti a molti**)==

- Un utente può aver preso in prestito più libri.
    
- Un libro può essere stato prestato a più utenti.
    

**Soluzione consigliata**: tracking doppio, sia in `Libri` che in `Utenti`, per facilitare query in entrambe le direzioni.

Ogni prestito è rappresentato da:

`{   "utente_id": ...,    "data_prestito": ...,    "data_restituzione": ... }`



# ==possibili indici da inserire==  

Per accelerare la maggior parte delle operazioni sulla collezione `utenti`, ecco gli indici da considerare:

### ==1. **Indice su `email`**==

`db.utenti.createIndex({ email: 1 }, { unique: true });`

-  Ottimo per recuperare rapidamente un utente tramite login/scambio email
    
-  Blocca inserimenti duplicati
    
-  Buona pratica quando interroghi spesso `find({ email: "..." })` 
    

### ==2. **Indice su `telefono`**==

`db.utenti.createIndex({ telefono: 1 }, { unique: true });`

-  Utile se il telefono serve come identificatore alternativo o chiave di ricerca
    

### ==3. **Indice su `data_registrazione==`**


`db.utenti.createIndex({ data_registrazione: 1 });`

-  Veloce per operazioni come “utenti registrati in un certo periodo”
    

### ==4. **Indice multikey su `libri_prestati.libro_id==`**


`db.utenti.createIndex({ "libri_prestati.libro_id": 1 });`

-  Consente di trovare velocemente utenti che hanno preso in prestito uno specifico libro
    
-  Indice _multikey_, come previsto per campi array [mongodb.com+1percona.com+1](https://www.mongodb.com/docs/manual/core/indexes/index-types/index-multikey/create-multikey-index-embedded/?utm_source=chatgpt.com)
    

### ==5. (Opzionale) **Indice su `libri_prestati.data_prestito==`**

`db.utenti.createIndex({ "libri_prestati.data_prestito": -1 });`

-  Utile per query su prestiti recenti, ordinati per data
