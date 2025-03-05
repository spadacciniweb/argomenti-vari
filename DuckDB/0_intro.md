# Esempi su DuckDB - Introduzione

I seguenti esercizi sono eseguiti dalla shell di DuckDB.

## Connessione al DBMS

```ducksh
$ duckdb
v1.2.0 5f5512b827
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
D
```

## Creazione e cancellazione di tabellle

Creare la tabella `weather`.

```ducksh
D CREATE TABLE weather (
    city    VARCHAR,
    temp_lo INTEGER, -- minimum temperature on a day
    temp_hi INTEGER, -- maximum temperature on a day
    prcp    FLOAT,
    date    DATE
);
```

e per verificare:

```ducksh
D show tables;
┌─────────┐
│  name   │
│ varchar │
├─────────┤
│ weather │
└─────────┘
```

Qualora si volesse cancellare tale tabella:

```ducksh
DROP TABLE weather;
```

e per verificare:

```ducksh
D show tables;
┌─────────┐
│  name   │
│ varchar │
├─────────┤
│ 0 rows  │
└─────────┘
```

## Inserimento righe

Inserire il seguente record nella tabella `weather`:

```ducksh
D INSERT INTO weather (city, temp_lo, temp_hi, prcp, date)
  VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');
```

Nonostante non sia una buona pratica, qualora si inseriscano valori per tutte le colonne mantenendo l'ordine, è possibile non esplicitarle.
Quindi la precedente `INSERT` è equivalente alla seguente:

```ducksh
D INSERT INTO weather
  VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');
```

Per il seguito si inseriranno ulteriori record:

```ducksh
D INSERT INTO weather VALUES
  ('San Francisco', 43, 57, 0.0, '1994-11-29'),
  ('Hayward', 37, 54, NULL, '1994-11-29');
```

## Visualizzazione record

Per visualizzare i record è possibile utilizzare la comune `SELECT`:

```ducksh
D SELECT * FROM weather;
┌───────────────┬─────────┬─────────┬───────┬────────────┐
│     city      │ temp_lo │ temp_hi │ prcp  │    date    │
│    varchar    │  int32  │  int32  │ float │    date    │
├───────────────┼─────────┼─────────┼───────┼────────────┤
│ San Francisco │      46 │      50 │  0.25 │ 1994-11-27 │
│ San Francisco │      43 │      57 │   0.0 │ 1994-11-29 │
│ Hayward       │      37 │      54 │  NULL │ 1994-11-29 │
└───────────────┴─────────┴─────────┴───────┴────────────┘
```

Per rimuovere le `city` duplicate è possibile utilizzare la `DISTINCT`:

```ducksh
SELECT DISTINCT city
FROM weather;
┌───────────────┐
│     city      │
│    varchar    │
├───────────────┤
│ Hayward       │
│ San Francisco │
└───────────────┘
```

Chiaramente è possibile utilizzare una `SELECT` più complessa:

```ducksh
D SELECT city, (temp_hi + temp_lo) / 2 AS temp_avg, date
FROM weather;
┌───────────────┬──────────┬────────────┐
│     city      │ temp_avg │    date    │
│    varchar    │  double  │    date    │
├───────────────┼──────────┼────────────┤
│ San Francisco │     48.0 │ 1994-11-27 │
│ San Francisco │     50.0 │ 1994-11-29 │
│ Hayward       │     45.5 │ 1994-11-29 │
└───────────────┴──────────┴────────────┘
```

con filtraggio tramite la  'WHERE', utilizzo di `NULL` e ordinamento:

```ducksh
D SELECT *
  FROM weather
  WHERE city = 'San Francisco'
    AND prcp IS NOT NULL
 ORDER BY city;
┌───────────────┬─────────┬─────────┬───────┬────────────┐
│     city      │ temp_lo │ temp_hi │ prcp  │    date    │
│    varchar    │  int32  │  int32  │ float │    date    │
├───────────────┼─────────┼─────────┼───────┼────────────┤
│ San Francisco │      46 │      50 │  0.25 │ 1994-11-27 │
│ San Francisco │      43 │      57 │   0.0 │ 1994-11-29 │
└───────────────┴─────────┴─────────┴───────┴────────────┘
```

## Aggiornare record

```ducksh
D UPDATE weather
  SET prcp = NULL WHERE prcp = 0;
```

Verificando:

```ducksh
D SELECT city, prcp, date FROM weather WHERE city = 'San Francisco';
┌───────────────┬───────┬────────────┐
│     city      │ prcp  │    date    │
│    varchar    │ float │    date    │
├───────────────┼───────┼────────────┤
│ San Francisco │  0.25 │ 1994-11-27 │
│ San Francisco │  NULL │ 1994-11-29 │
└───────────────┴───────┴────────────┘
```

## Cancellazione record

Per la cancellazione dei record si ricorrerrà al `DELETE`:

```ducksh
D DELETE FROM weather;
```

## Eseguire JOIN

Per eseguire la `JOIN` si inserisca la nuova tabella di seguito

```ducksh
D CREATE TABLE cities (
    name VARCHAR,
    lat  DECIMAL,
    lon  DECIMAL
);
```

e la si popola tramite:

```ducksh
D INSERT INTO cities
VALUES ('San Francisco', -194.0, 53.0);
```

Una semplice `JOIN` è la seguente:

```ducksh
D SELECT weather.city, weather.temp_lo, weather.temp_hi,
       weather.prcp, weather.date, cities.lon, cities.lat
FROM weather, cities
WHERE cities.name = weather.city;
┌───────────────┬─────────┬─────────┬───────┬────────────┬───────────────┬───────────────┐
│     city      │ temp_lo │ temp_hi │ prcp  │    date    │      lon      │      lat      │
│    varchar    │  int32  │  int32  │ float │    date    │ decimal(18,3) │ decimal(18,3) │
├───────────────┼─────────┼─────────┼───────┼────────────┼───────────────┼───────────────┤
│ San Francisco │      46 │      50 │  0.25 │ 1994-11-27 │        53.000 │      -194.000 │
│ San Francisco │      43 │      57 │  NULL │ 1994-11-29 │        53.000 │      -194.000 │
└───────────────┴─────────┴─────────┴───────┴────────────┴───────────────┴───────────────┘
```

in cui non sono visualizzate row con la città `Hayward` poiché essa non è presente nella tabella `cities`.\
La query precedente è equivalente a quella di seguito:

```ducksh
D SELECT weather.city, weather.temp_lo, weather.temp_hi,
       weather.prcp, weather.date, cities.lon, cities.lat
  FROM weather
  INNER JOIN cities ON weather.city = cities.name;
```

quindi, per ovviare al problema di non visualizzazione precedente, non si utilizzerà la `INNER JOIN` ma la `OUTER JOIN` come di seguito:

```ducksh
SELECT *
FROM weather
LEFT OUTER JOIN cities ON weather.city = cities.name;
```

in cui si è optato per la `LEFT OUTER JOIN` per quanto di seguito:

- `INNER JOIN` (default) -> combina le row delle tabelle basandosi sulla condizione di match, considerando solo le row che hanno un match in entrambe le tabelle (in un diagramma di Venn, ritorna solo l'intersezione tra insiemi);
- `LEFT OUTER JOIN` -> combina tutte le row della tabella menzionata per prima e solo le row della seconda tabella che rispondono alla condizione di match. Se non ci sono match nella seconda tabella, le relative colonne saranno visualizzate con `NULL`;
- `RIGHT OUTER JOIN` -> combina tutte le row della tabella menzionata per seconda e solo le row della prima tabella che rispondono alla condizione di match. Se non ci sono match nella prima tabella, le relative colonne saranno visualizzate  con `NULL`;
- `FULL OUTER JOIN` -> anche denominata `FULL JOIN` mostra tutti i record da entrambe le tabelle: quando ci sono match mostrerà i dati, nei casi residui visualizzera il valore `NULL`;
- `CROSS JOIN` -> anche denominata `CARTESIAN JOIN` crea un prodotto cartesiano tra row, quindi creerà un nuovo insieme in cui combinerà ogni row dalla prima tabelle con ogni row della seconda tabella. In altri termini, sarà creato ogni possibile conmbinazione senza effettuare verifiche di match.

Dall'elenco precedente sono trascurate le metodologie di `JOIN` più specifiche quali le 'ASOF Join' e le join su file e subqueries.
