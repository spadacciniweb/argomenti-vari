# Esempi su MongoDB - Aggregation II

I seguenti esercizi sono eseguiti dalla shell di MongoDB.\
I dati degli esempi sono collezioni di zip codes presi da [qui](https://www.mongodb.com/docs/manual/tutorial/aggregation-zip-code-data-set/) e una copia degli stessi dati è fornita nel file `zips.json.gz`.

## Data Model

Un documento di esempio è il seguente:

```json
{
  "_id": "10280",
  "city": "NEW YORK",
  "state": "NY",
  "pop": 5574,
  "loc": [
    -74.016323,
    40.710537
  ]
}
```

e ciascun documento segue il seguente modello descritto dai seguenti field:

- `_id` è lo zip code in formato stringa;
- `city` è il nome della città. Una città può avere associati più di uno zip code poiché i distretti cittadini sono a loro volta identificati da propri zip code;
- `state` è formato da due lettere che costituiscono l'abbreviazione dello stato;
- `pop` indica la popolazione;
- `loc` mostra la coordinate della località (latitudine e longitudine).

## Inserimento massivo

Per il caricamento del contenuto del file è possibile utilizzare lo strumento `mongoimport`.

```sh
gunzip -c zips.json.gz | mongoimport --db esercizi --collection zipcodes
```

## Mostrare gli Stati con popolazione maggiore o uguale a 10 milioni

La seguente operazione di aggregazione mostra tutti gli Stati con popolazione maggiore o uguale a 10 millioni:

```mongosh
db.zipcodes.aggregate([
   { $group: { _id: "$state", totalPop: { $sum: "$pop" } } },
   { $match: { totalPop: { $gte: 10*1000*1000 } } }
])
```

In quest'esempio la pipeline è composta dallo stage `$group` e successivamente dallo stage `$match`.

Lo stage `$group`:

- raggruppa i documenti attraverso il field `state`;
- del raggruppamento precedente calcola il field `totalPop` composta dalla somma della popolazione di ciscun documento del gruppo.

In altri termini, i documenti di questo stadio sono formati da due field:

- `_id` è la chiave del raggruppamento ed è formata dal nome dello stato;
- `totalPop` è la somma della popolazione dei gruppo identificato nel precedente punto.

Successivamente a tale stage, tipicamente un documento di esempio sarà:

```json
{
  "_id" : "AK",
  "totalPop" : 550043

}
```

Lo stage `$match` filtra i documenti presentati dal precedente raggruppamento imponendo, come condizione per passare allo stadio successivo, che `totalPop` abbia una valore maggiore o uguale a 10 milioni.

L'equivalente SQL della pipeline mostrata è:

```sql
SELECT state, SUM(pop) AS totalPop
FROM zipcodes
GROUP BY state
HAVING totalPop >= (10*1000*1000)
```

L'output mostrato sarà:

```mongosh
[
  { _id: 'TX', totalPop: 16984601 },
  { _id: 'IL', totalPop: 11427576 },
  { _id: 'FL', totalPop: 12686644 },
  { _id: 'CA', totalPop: 29754890 },
  { _id: 'NY', totalPop: 17990402 },
  { _id: 'OH', totalPop: 10846517 },
  { _id: 'PA', totalPop: 11881643 }
]
```

## Visualizzare la popolazione media per Stato

La seguente aggregation pipeline mostra la popolazione media per ciascuno Stato:

```mongosh
db.zipcodes.aggregate([
   { $group: { _id: { state: "$state", city: "$city" }, pop: { $sum: "$pop" } } },
   { $group: { _id: "$_id.state", avgCityPop: { $avg: "$pop" } } }
])
```

Nell'esempio la pipeline è composta da dallo stage `$group` seguita da un ulteriore stage `$group`.

Nel primo `$group`, i documenti sono raggruppati dalla composizione di `$state` e `$city` e si aggiunge il field  `pop` per il totale della popolazione del gruppo determinato.

Successivamente a tale stage, tipicamente un documento di esempio sarà:

```json
{
  "_id" : {
    "state" : "CO",
    "city" : "EDGEWATER"
  },
  "pop" : 13154
}
```

Il secondo `$group` della pipeline:

- raggrupperà i documenti attraverso il field `_id.state` (quindi il subfield `state` del field `_id` dei documenti del precedente stage);
- utilizza l'espressione `$avg` per calcolare la media della popolazione da assegnare al nuovo field `avgCityPop`;
- mostrerà in output un documento per ogni Stato.

L'output mostrato sarà:

```mongosh
[
  { _id: 'AK', avgCityPop: 2976.4918032786886 },
  { _id: 'WV', avgCityPop: 2771.4775888717154 },
  { _id: 'MI', avgCityPop: 12087.512353706112 },
...
]
```

## Per ciascun Stato visualizzare la città più popolosa e meno popolosa

La seguente pipeline mostra la meno popolosa e la più popolosa città per ciascuno Stato:

```mongosh
db.zipcodes.aggregate([
   { $group:
      {
        _id: { state: "$state", city: "$city" },
        pop: { $sum: "$pop" }
      }
   },
   { $sort: { pop: 1 } },
   { $group:
      {
        _id : "$_id.state",
        biggestCity:  { $last: "$_id.city" },
        biggestPop:   { $last: "$pop" },
        smallestCity: { $first: "$_id.city" },
        smallestPop:  { $first: "$pop" }
      }
   },
  // the following $project is optional, and
  // modifies the output format.
  { $project:
    { _id: 0,
      state: "$_id",
      biggestCity:  { name: "$biggestCity",  pop: "$biggestPop" },
      smallestCity: { name: "$smallestCity", pop: "$smallestPop" }
    }
  }
])
```

Nell'esempio l'aggregation pipeline è comosta da `$group`, `$sort` e un ulteriore `$group`a cui si è aggiunto un opzionale $project.

Il primpo `$group` raggruppa i documenti per `city` e `state`, calcola la popolazione totale per ciascuna combinazione, e mostra tali documenti.\
A questo stage della pipeline i documenti appaiono come nell'esempio:

```json
{
  "_id" : {
    "state" : "CO",
    "city" : "EDGEWATER"
  },
  "pop" : 13154
}
```

Il successivo `$sort` ordina i documenti per il valore del field `pop` in ordine crescente.

Il successivo `$group` riceve i documenti ordinati e li raggruppa per `_id.state` (quindi il subfield `state` nel field `_id`) e visualizza un documento per ciascuno Stato.\
In questo stage si calcolano altri 4 field per ciascuno Stato, in particolare utilizzando:

- l'espressione `$last`, l'operatore '$group' crea `biggestCity` e `biggestPop` per la città più popolosa;
- l'espressione `$first`, l'operatore '$group' crea `smallestCity` e `smallestPop` per la città meno popolosa.

A questo stage della pipeline, un documento potrebbe apparire nel seguente modo:

```json
{
  "_id" : "WA",
  "biggestCity" : "SEATTLE",
  "biggestPop" : 520096,
  "smallestCity" : "BENGE",
  "smallestPop" : 2
}
```

Lo stage finale `$peoject` rinomina il field  '_id' in `state` e raggruppa la coppia `biggestCity` e `biggestPop` e la coppia `smallestCity` e `smallestPop` rispettivamente in `biggestCity` e `smallestCity`.

L'output finale sarà come di seguito:

```mongodb
[
  ...
  {
    biggestCity: { name: 'HOUSTON', pop: 2095918 },
    smallestCity: { name: 'FULTON', pop: 0 },
    state: 'TX'
  },
  {
    biggestCity: { name: 'COLUMBIA', pop: 269521 },
    smallestCity: { name: 'QUINBY', pop: 0 },
    state: 'SC'
  },
  {
    biggestCity: { name: 'INDIANAPOLIS', pop: 348868 },
    smallestCity: { name: 'WESTPOINT', pop: 145 },
    state: 'IN'
  },
  ...
]
```
