# DuckDB overview

Ci sono svariati sistemi di gestione dei database (DBMS), nessuno con caratteristiche omnicomprensive, ma ciascuno con differenti trade-offs per i diversi casi d'uso.\
DuckDB è un RDBMS che supporta lo Structured Query Language (SQL).

Per la sua semplicità è accostabile a `SQLite`, il più diffuso DBMS.
Per entrambi non ci sono DBMS server software da installare e manutenere, ma sono embeddati nel processi che si interfacciano tramite l'utilizzo di librerie.
Inoltre entrambi possono lavorare sia in-memory sia in maniera persistente memorizzando il database su file il quale includerà tutte le tabelle, le viste, gli indici, le macro, ....

Il formato di memorizzazione di DuckDB è costituito da una `compressed columnar representation`, la quale unisce compattezza ad efficienti operazioni bulk. Questa caratteristica permette di operare agevolmente per casistiche analitiche con importanti velocità di trasferimento tra il processo e il database.\
Questa scelta progettuale la rende valida per supportare carichi di lavoro per operazioni `online analytical processing`* (OLAP).

Nota:
[*] -> il tipico workload delle OLAP è caratterizzato da complesse query che hanno relativamente un tempo di processamento lungo le quali solitamente esaminano significativi porzioni di tabelle o join tra tabelle. Eventuali modifiche su dati sono spesso su larga scala, con operazioni su ampie porzioni di tabelle.

## Teoria

- [Installation](http://duckdb.org/docs/installation)
- Connect:
  - [Overview](https://duckdb.org/docs/stable/connect/overview)
  - [Concurrency](https://duckdb.org/docs/stable/connect/concurrency)
- Data Import:
  - [Overview](https://duckdb.org/docs/stable/data/overview)
  - [Data Sources](https://duckdb.org/docs/stable/data/data_sources)
  - CSV files:
    - [Overview](https://duckdb.org/docs/stable/data/csv/overview)
    - [Auto Detection](https://duckdb.org/docs/stable/data/csv/auto_detection)
    - [Reading Faulty CSV Files](https://duckdb.org/docs/stable/data/csv/reading_faulty_csv_files)
    - [Tips](https://duckdb.org/docs/stable/data/csv/tips)
  - JSON files:
    - [Overview](https://duckdb.org/docs/stable/data/json/overview)
    - [Loading](https://duckdb.org/docs/stable/data/json/loading_json)
    - [SQL to/from JSON](https://duckdb.org/docs/stable/data/json/sql_to_and_from_json)
  - Parquet files:
    - [Overview](https://duckdb.org/docs/stable/data/parquet/overview)
    - [Metadata](https://duckdb.org/docs/stable/data/parquet/metadata)
  - Client API:
    - [Overview](https://duckdb.org/docs/stable/clients/overview)
  - SQL
    - [Introduction](https://duckdb.org/docs/stable/sql/introduction)
    - [Statements](https://duckdb.org/docs/stable/sql/statements/overview)
    - [Data types](https://duckdb.org/docs/stable/sql/data_types/overview)
  - [Guides](https://duckdb.org/docs/stable/guides/overview)

## Esempi
  
- [intro](0_intro.md)

## Approfondimenti

- [window functions](https://duckdb.org/2025/02/10/window-catchup.html)
- [CSV Reader and the Pollock Robustness Benchmark](https://duckdb.org/2025/04/16/duckdb-csv-pollock-benchmark.html)
- [Temporal Analysis with Stream Windowing Functions](https://duckdb.org/2025/05/02/stream-windowing-functions.html)

## Tricks

- [part 1](https://duckdb.org/2024/08/19/duckdb-tricks-part-1.html)
- [part 2](https://duckdb.org/2024/10/11/duckdb-tricks-part-2.html)
- [part 3](https://duckdb.org/2024/11/29/duckdb-tricks-part-3.html)
