# Esempio di transazione

Si abbiano 2 sessioni, la prima gestirà la transazione, la seconda esterna alla transazione.


Si esegua per entrambe una `SELECT` significativa (di seguito con `id = n`).
|session 1|session 2|
|:---:|:---:|
|SELECT * from bank_movement WHERE id = n\G|SELECT * from bank_movement WHERE id = n\G|

che saranno evidentemente equivalenti.

Ora si crei la transazione con `START TRANSACTION` (o equivalentemente `BEGIN`):

|session 1|session 2|
|:---:|:---:|
|START TRANSACTION;||
|UPDATE bank_movement SET processed = 1 WHERE id = n;|
|SELECT * from bank_movement WHERE id = n\G|SELECT * from bank_movement WHERE id = n\G|

in cui sarà evidente la differenza.

Successivamente per annullare la transazione non conclusa:
|session 1|session 2|
|:---:|:---:|
|ROLLBACK;||
|SELECT * from bank_movement WHERE id = n\G|SELECT * from bank_movement WHERE id = n\G|

in cui la prima sessione sarà tornata convergente all'originale poiché è stato eseguito un  `ROLLBACK`.

Di seguito una transazione conclusa positivamente:
|session 1|session 2|
|:---:|:---:|
|START TRANSACTION;||
|UPDATE bank_movement SET processed = 1 WHERE id = n;|
|COMMIT||
|SELECT * from bank_movement WHERE id = n\G|SELECT * from bank_movement WHERE id = n\G|

in cui ora la prima sessione è stata terminata con il `COMMIT`, per cui avrà modificato l'originale.
