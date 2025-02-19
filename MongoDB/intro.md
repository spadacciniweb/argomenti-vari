# Esercizi su MongoDB

I seguenti esercizi sono eseguiti dalla shell di MongoDB.

## Connessione al DBMS

```
$ mongosh
Current Mongosh Log ID:	67b44b4033985532bee43268
Connecting to:		mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.3.9
Using MongoDB:		8.0.4
Using Mongosh:		2.3.9

For mongosh info see: https://www.mongodb.com/docs/mongodb-shell/

------
   The server generated these startup warnings when booting
   2025-02-13T15:55:07.834+01:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2025-02-13T15:55:08.181+01:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
   2025-02-13T15:55:08.182+01:00: For customers running the current memory allocator, we suggest changing the contents of the following sysfsFile
   2025-02-13T15:55:08.182+01:00: For customers running the current memory allocator, we suggest changing the contents of the following sysfsFile
   2025-02-13T15:55:08.182+01:00: We suggest setting the contents of sysfsFile to 0.
   2025-02-13T15:55:08.182+01:00: We suggest setting swappiness to 0 or 1, as swapping can cause performance problems.
------

test>
```

## Creazione database

Creare il database `esercizi`.
```
test> use esercizi
switched to db esercizi
esercizi>
```

e, per verificarne la creazione:
```
esercizi> show dbs
admin            40.00 KiB
config           48.00 KiB
local            72.00 KiB
esercizi>
```

Come è evidente, non si è creato nessun DB, semplicemente l'attività di gestione si è spostata al potenziale DB che sarà creato solo qualora sarà creato del contenuto.\
Per semplicità, nel seguito il prompt sarà semplificato con il seguente:
```
>
```

## Lista collections

```
> show collections
>
```

In accordo con il paragrafo precedente, se non sono presenti documenti, non ci sarà nessun contenitore, quindi nemmeno nessuna collections. 

## Inserimento documenti

Inserire i seguenti 5 documenti nella `collection` `movies`:

```
title: The Godfather
director:	Francis Ford Coppola
writer: Mario Puzo
year: 1972
actors : [
  Marlon Brando
  Al Pacino
  Robert Duvall
]
```

```
original_title: Star Wars
title: Episode IV – A New Hope
director:	George Lucas
writer: George Lucas
year: 1977
actors : [
  Mark Hamill
  Harrison Ford
  Carrie Fisher
  Alec Guinness
]
```

```
title: Back to the Future
director:	Robert Zemeckis
writer: [
  Robert Zemeckis
  Bob Gale
]
year: 1985
actors : [
  Michael J. Fox
  Christopher Lloyd
  Thomas Francis Wilson Jr.
]
```

```
title: Kill Bill: Volume 1
director:	Quentin Tarantino
writer: Quentin Tarantino
year: 2003
actors : [
  Uma Thurman
  Lucy Liu
  David Carradine
]
```

e

```
title : Avatar
year: 2009
```

L'inserimento dei documenti potrà essere effettuato tramite due modalità:
- inserimento dei singoli documenti;
- inserimento massivo.
Ciascuna query riceverà un `ACK` se andrà a buon fine (quindi la prima otterrà tanti `ACK` quanti saranno i documenti da inserire), la seconda solo un `ACK` complessivo.

### Inserimento singoli documenti
L'inserimento è effettuato come di seguito:

```
> db.movies.insertOne({ title: "The Godfather", director:	"Francis Ford Coppola", writer: "Mario Puzo", year: 1972, actors: ["Marlon Brando", "Al Pacino", "Robert Duvall"] })
{
  acknowledged: true,
  insertedId: ObjectId('67b4588333985532bee4326e')
}
> db.movies.insertOne({ original_title: "Star Wars", title: "Episode IV – A New Hope", director:	"George Lucas", writer: "George Lucas", year: 1977, actors: ["Mark Hamill", "Harrison Ford", "Carrie Fisher", "Alec Guinness"] })
{
  acknowledged: true,
  insertedId: ObjectId('67b4589833985532bee4326f')
}
> db.movies.insertOne({ title: "Back to the Future", director: "Robert Zemeckis", writer: ["Robert Zemeckis", "Bob Gale"], year: 1985, actors: ["Michael J. Fox", "Christopher Lloyd", "Thomas Francis Wilson Jr."] })
{
  acknowledged: true,
  insertedId: ObjectId('67b458b533985532bee43270')
}
> db.movies.insertOne({ title: "Kill Bill: Volume 1", director:	"Quentin Tarantino", writer: "Quentin Tarantino", year: 2003, actors : ["Uma Thurman", "Lucy Liu", "David Carradine"] })
{
  acknowledged: true,
  insertedId: ObjectId('67b458d733985532bee43271')
}
> db.movies.insertOne({ title : "Avatar", year: 2009 })
{
  acknowledged: true,
  insertedId: ObjectId('67b458fc33985532bee43272')
}
```

Si nota quindi che per ogni query andata a buon fine si è ottenuto un `ACK` e relativo indice assegnato.

### Inserimento massivo

In alternativa al precedente inserimento per singolo documento, è possibile effettuare un inserimento unico che avrà maggiore efficienza poiché ci sarà un unico `ACK` finale:

```
> db.movies.insertMany([{ title: "The Godfather", director:	"Francis Ford Coppola", writer: "Mario Puzo", year: 1972, actors: ["Marlon Brando", "Al Pacino", "Robert Duvall"], massivo: 1 }, { original_title: "Star Wars", title: "Episode IV – A New Hope", director:	"George Lucas", writer: "George Lucas", year: 1977, actors: ["Mark Hamill", "Harrison Ford", "Carrie Fisher", "Alec Guinness"], massivo: 1 },{ title: "Back to the Future", director: "Robert Zemeckis", writer: ["Robert Zemeckis", "Bob Gale"], year: 1985, actors: ["Michael J. Fox", "Christopher Lloyd", "Thomas Francis Wilson Jr."], massivo: 1 }, { title: "Kill Bill: Volume 1", director:	"Quentin Tarantino", writer: "Quentin Tarantino", year: 2003, actors : ["Uma Thurman", "Lucy Liu", "David Carradine"], massivo: 1 },{ title : "Avatar", year: 2009, massivo: 1 }])
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId('67b45a5533985532bee43273'),
    '1': ObjectId('67b45a5533985532bee43274'),
    '2': ObjectId('67b45a5533985532bee43275'),
    '3': ObjectId('67b45a5533985532bee43276'),
    '4': ObjectId('67b45a5533985532bee43277')
  }
}
```

Si nota quindi che, essendo una query il cui contenuto sono 5 documenti, si otterrà un singolo `ACK` complessivo e relativi indici assegnati.

## Cancellazione documenti

Avendo inserito duplicati, per procedere in maniera pulita si è preferito anticipare il metodo di cancellazione.\
Come l'inserimento, potrà essere effettuato tramite due modalità:
- cancellazione dei singoli documenti tramite `deleteOne()`;
- cancellazione massiva tramite `deleteMany()`.
Ciascuna query riceverà un `ACK` se andrà a buon fine (quindi la prima otterrà tanti `ACK` per ciascun documento cancellato), la seconda solo un `ACK` complessivo.

### Cancellazione di singoli documenti
Si vogliano cancellare i primi due documenti inseriti tramite la modalità massiva precedente.\
Si procede come di seguito:

```
> db.movies.deleteOne({ _id: ObjectId('67b45a5533985532bee43273') })
{ acknowledged: true, deletedCount: 1 }
> db.movies.deleteOne({ massivo: 1 })
{ acknowledged: true, deletedCount: 1 }
```

Chiaramente effettuando la cancellazione tramite l'attributo `_id` necessariamente si avrà un unico documento, ma sarà cancellato un unico documento anche utilizzando il valore di un attributo ripetuto. 

### Cancellazione massiva

Si vogliano cancellare i documenti inseriti tramite la modalità massiva precedente.\
Si può sfuttare il valore di un attributo ripetuto (`massivo: 1`), quindi si procederà come di seguito:

```
> db.movies.deleteMany({ massivo: 1 })
{ acknowledged: true, deletedCount: 3 }
```

in cui saranno cancellati solo 3 dei 5 poiché i mancanti erano stati cancellati precedentemente tramite le `deleteOne()`.

## Ricerca documenti

Per ricercare documenti si utilizzano principalmente i metodi:
- `findOne()`: restituisce un documento nel caso di match, `null` in caso negativo.
- `find()`, restituisce un `cursor object`.

Di seguito alcune query sulla collect `movies` per ottenere:

1. tutti i documenti:

```
> db.movies.find()
```

2. i documento con `writer` "Quentin Tarantino"

```
> db.movies.find({ writer: "Quentin Tarantino" })
```

3. i documenti con `actors` "Christopher Lloyd"

```
> db.movies.find({ actors: "Christopher Lloyd" })
```

4. i documenti rilasciati nel decennio 1970-1980:

```
> db.movies.find({ year: {$gt: 1970, $lt: 1980 } })
```

5. i documenti pubblicati prima dell'anno 1975 o successivamente all'anno 2005:

```
> db.movies.find({ $or: [{ year: { $gt: 2005 }}, { year: { $lt: 1975 } }] })
```

## Aggiornare i documenti

Come l'inserimento e la cancellazione, la modifica potrà essere effettuato tramite due modalità:
- aggiornamento di singoli documenti tramite `updateOne()`, quindi sarà modificato solo il primo documento che avrà match con il filtro;
- aggiornamento massivo tramite `updateMany()`, quindi saranno modificati tutti i documenti che avranno match con il filtro.\
Ciascuna query riceverà un `ACK` se andrà a buon fine (quindi per la prima si otterranno `ACK` per ciascun documento che dovrà essere aggiornato), la seconda solo un `ACK` complessivo.


1. aggiungere director: "James Cameron" e writer: "James Cameron" al film "Avatar":

```
> db.movies.update({ _id: ObjectId("67b46dac33985532bee43281") }, { $set: { director: "James Cameron", writer: "James Cameron" } })
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
```

2. aggiungere actors: "Sam Worthington" al film "Avatar":
```
> db.movies.update({ _id: ObjectId("67b46dac33985532bee43281") }, { $push: { actors: "Sam Worthington" } })
```

## Ricerca di testo

1. ricerca tutti i documenti contenenti nel titolo la stringa "the":

```
> db.movies.find({ title: { $regex: "the" } })
```

2. ricerca tutti i documenti contenenti nel titolo la stringa "the" o la stringa "to":

```
> db.movies.find({ $or: [{ title: { $regex: "the" } }, { title: { $regex: "to" } }] })
```

3. ricerca tutti i documenti contenenti nel titolo la stringa "the" ma non la stringa "back" (quest'ultima insensitive): 

```
> db.movies.find({ $and: [{ title: { $regex: "the" }}, { title: { $not: /back/i }}] })
```
