PostgresSQL
===========

UPDATE avec jointures
---------------------

Il est souvent utile de devoir mettre à jour des données d'une table t1 avec des valeurs issue d'une table t2

```SQL
UPDATE t1
SET t1.c1 = new_value
FROM t2
WHERE t1.c2 = t2.c2;
```