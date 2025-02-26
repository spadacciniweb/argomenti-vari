# Esercizi su MongoDB

I seguenti esercizi sono eseguiti dalla shell di MongoDB.

## step 0 - inserimento massivo

```mongosh
> db.orders.insertMany([
   { _id: 0, name: "Margherita", size: "small", price: 4,
     quantity: 5, date: ISODate( "2025-02-27T08:14:30Z" ) },
   { _id: 1, name: "Margherita", size: "medium", price: 7,
     quantity: 10, date : ISODate( "2025-02-27T08:23:25Z" ) },
   { _id: 2, name: "Margherita", size: "large", price: 11,
     quantity: 12, date : ISODate( "2025-02-27T08:55:51Z" ) },
   { _id: 3, name: "Capricciosa", size: "small", price: 5,
     quantity: 18, date : ISODate( "2025-02-27T21:18:40.654Z" ) },
   { _id: 4, name: "Capricciosa", size: "medium", price: 8,
     quantity: 20, date : ISODate( "2025-02-27T21:19:22.370Z" ) },
   { _id: 5, name: "Margherita", size: "small", price: 4,
     quantity: 2, date: ISODate( "2025-02-27T21:19:35Z" ) },
   { _id: 6, name: "Capricciosa", size: "large", price: 12,
     quantity: 6, date : ISODate( "2025-02-27T21:20:17Z" ) },
   { _id: 7, name: "Vegana", size: "small", price: 7,
     quantity: 2, date : ISODate( "2025-02-28T09:12:19Z" ) },
   { _id: 8, name: "Margherita", size: "medium", price: 7,
     quantity: 3, date : ISODate( "2025-02-28T09:13:27Z" ) },
   { _id: 9, name: "Vegana", size: "medium", price: 10,
     quantity: 1, date : ISODate( "2025-02-28T09:14:14Z" ) }
])
```

## Mostrare field tramite aggregazione

Di seguito una semplice aggregazione la quale, tramite l'aggregatore `$projects`, restituisce solo il field `name` per ciascun documento:

```mongosh
db.orders.aggregate([
  { $project: { _id: 0, name: 1 } }
])
```

L'output mostrato sarà:

```mongosh
[
  { name: 'Margherita' },
  { name: 'Margherita' },
  { name: 'Margherita' },
  { name: 'Capricciosa' },
  { name: 'Capricciosa' },
  { name: 'Margherita' },
  { name: 'Capricciosa' },
  { name: 'Vegana' },
  { name: 'Margherita' },
  { name: 'Vegana' }
]
```

`Nota:` Nel seguente caso semplificato di query semplice con pipeline formata da un unico stage con funzione di aggregazione `$project`, la query standard espressa tramite il metodo `find()` ha performance migliori.

## Normalizzare valori e ordinarli

Di seguito l'elenco precedente sarà ampliato con la `size` riporatata tramite stringhe maiuscole e l'ordinamento sarà alfabetico per `name` and `size`:

```mongosh
db.orders.aggregate([
  { $project: { _id: 0, name: 1, size: { $toUpper: "$size" }  } },
  { $sort: { name: 1, size: -1 } }
])
```

I documenti che formano la collection `orders` passano attraverso la pipeline la quale è formata dalle seguenti operazioni:

Lo stage dell'operatore `$project`:

- crea un nuovo field con nome `size` il cui valore sarà formato dal `$toUpper` del valore del field `$size` del documento originale;
- sopprime i field originali ad esclusione di `$name`.

Lo stage dell'operatore `$sort` effettua l'ordinamento ascendente del field `name`, discendente di `size`.

L'output mostrato sarà quindi:

```mongosh
[
  { name: 'Capricciosa', size: 'SMALL' },
  { name: 'Capricciosa', size: 'MEDIUM' },
  { name: 'Capricciosa', size: 'LARGE' },
  { name: 'Margherita', size: 'SMALL' },
  { name: 'Margherita', size: 'SMALL' },
  { name: 'Margherita', size: 'MEDIUM' },
  { name: 'Margherita', size: 'MEDIUM' },
  { name: 'Margherita', size: 'LARGE' },
  { name: 'Vegana', size: 'SMALL' },
  { name: 'Vegana', size: 'MEDIUM' }
]
```

## Filtro su `size` e calcolo della quantità totale ordinata per singola tipologia

La seguente pipeline è formata da due step e ritorna la quatità di pizza di misura media raggruppate per nome:

```mongosh
db.orders.aggregate([
   // stage 1: filtra gli ordini per size "medium"
   {
      $match: { size: "medium" }
   },
   // stage 2: raggruppa i rimanenti documenti per nome della pizza e calcolo della quantità totale
   {
      $group: { _id: "$name", totalQuantity: { $sum: "$quantity" } }
   }
])
```

Lo stage `$match`:

- filtra gli ordini delle pizze per `size` con valore `medium`;
- passa i documenti rimanenti allo step successivo.

Lo stage `$group`:

- raggruppa gli ordini per il field `name` della pizza;
- utilizza `$sum` per calcolare la quantità degli ordini totale per ogni nome della pizza. Il totale è memorizzato nel field `totalQuantity` che sarà mostrato dalla pipeline aggregata.

L'output mostrato sarà quindi:

```mongosh
[
  { _id: 'Margherita', totalQuantity: 13 },
  { _id: 'Capricciosa', totalQuantity: 20 },
  { _id: 'Vegana', totalQuantity: 1 }
]
```

## Mostrare la quantità aggregata degli ordini per name e size, limitandosi alla visualizzazione ordinata per quantità descrescenti di soli 3 documenti

```mongosh
db.orders.aggregate([
  {
    $group: { _id: { name: "$name", size: "$size" }, totalQuantity: { $sum: "$quantity" } }
  },
  {
    $sort: { totalQuantity: -1 }
  },
  {
    $limit: 3
  }
])
```
Lo stage `$group`:

- raggruppa i documenti per `$name` e `$size`;
- per ciascun gruppo identificato:
  - crea l'accumulatore `totalQuantity` sommando tramite la funzione `$sum` ciascuna `$quantity`;
  - passa i documenti rimanenti allo step successivo.

Lo stage `$sort` mostrerà i documenti riportati nello stage precedente ordinandoli tramite il field `totalQuantity` in ordine discendente.

Lo stage `$limit` limiterà il report ai soli primi 3 documenti dello stage precedente.

L'output mostrato sarà:

```mongosh
[
  { _id: { name: 'Capricciosa', size: 'medium' }, totalQuantity: 20 },
  { _id: { name: 'Capricciosa', size: 'small' }, totalQuantity: 18 },
  { _id: { name: 'Margherita', size: 'medium' }, totalQuantity: 13 }
]
```

## Calcolo del valore totale e quantità media degli ordini in una finestra temporale stabilita

Di seguito sarà calcolato il valore totale della pizza ordinata e la quantità media ordinata in una finestra temporale:

```mongosh
db.orders.aggregate([
   // stage 1: filtro i documenti per intervallo di data
   {
      $match:
      {
         "date": { $gte: new ISODate( "2025-02-27" ), $lt: new ISODate( "2025-02-29" ) }
      }
   },
   // stage 2: ragguppo i documenti rimanenti per il calcolo
   {
      $group:
      {
         _id: { $dateToString: { format: "%Y-%m-%d", date: "$date" } },
         totalOrderValue: { $sum: { $multiply: [ "$price", "$quantity" ] } },
         averageOrderQuantity: { $avg: "$quantity" }
      }
   },
   // stage 3: ordinamento dei documenti per il field totalOrderValue in ordine discendente
   {
      $sort: { totalOrderValue: -1 }
   }
 ])
 ```

Lo stage `$match`:

- filtra gli ordini per la finestra temporale specificata ( `2025-02-27 <= t < 2025-02-29`);
- passa i documenti rimanenti allo step successivo.

Lo stage `$group`:

- raggruppa i documenti per data tramite la funzione `$dateToString`;
- per ciascun gruppo identificato:
  - calcola il valore totale dell'ordine sommando i valori restituiti dal prodotto `$price` e `$quantity`, tramite le funzioni rispettivamente `$sum` e `$multiply`;
  . calcola la quantità media dell'ordine tramite la funzione `$avg`;
  - passa i documenti rimanenti allo step successivo.

Lo stage `$sort` mostrerà i documenti riportati nello stage precedente ordinandoli tramite il field `totalOrderValue` in ordine discendente.

L'output mostrato sarà:

```mongosh
[
  {
    _id: '2025-02-27',
    totalOrderValue: 552,
    averageOrderQuantity: 10.428571428571429
  },
  { _id: '2025-02-28', totalOrderValue: 45, averageOrderQuantity: 2 }
]
```
