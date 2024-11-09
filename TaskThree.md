## 1.Создал таблицу с текстовым полем и заполнил сгенерированными данным в размере 1 млн строк
```
CREATE TABLE test_table (
    id SERIAL PRIMARY KEY,
    text_field TEXT
INSERT INTO test_table (text_field)
SELECT md5(random()::text)
FROM generate_series(1, 1000000);
```
 ## Посмотрел размер файла с таблицей
```
SELECT pg_size_pretty( pg_database_size( 'test3' ) );

 pg_size_pretty
----------------
 95 MB
(1 row)
 SELECT pg_size_pretty( pg_total_relation_size( 'test_table' ) );
 pg_size_pretty
----------------
 87 MB
(1 row)
```
 ## 5 раз обновил все строчки и добавил к каждой строчке любой символ
 ```
UPDATE test_table
SET text_field = text_field || '$'

UPDATE test_table
SET id = id || '$'
 ```
 ## Посмотрел количество мертвых строчек в таблице и когда последний раз приходил автовакуум
 ```
SELECT
    relname AS teat_table,
    n_dead_tup AS dead_tuples
FROM
    pg_stat_user_tables
WHERE
    relname = 'your_table_name';
 teat_table | dead_tuples
------------+-------------
(0 rows)
SELECT
    relname AS table_name,
    last_vacuum,
    last_autovacuum,
    n_dead_tup AS dead_tuples
FROM
    pg_stat_user_tables
WHERE
    relname = 'test_table';
 table_name | last_vacuum |        last_autovacuum        | dead_tuples
------------+-------------+-------------------------------+-------------
 test_table |             | 2024-10-13 14:57:58.362969+03 |           0
(1 row)
Подождал некоторое время, проверил автовакуум

```
## 5 раз обновил все строчки и добавил к каждой строчке любой символ
```
UPDATE test_table
SET id = id || '!'
test3-# UPDATE test_table
SET id = id || '2'
test3-# UPDATE test_table
SET id = id || 'h'
test3-# UPDATE test_table
SET id = id || '-'
test3-# UPDATE test_table
SET id = id || '='
```
## Посмотрел размер файла с таблицей
```
test3=# SELECT pg_size_pretty( pg_database_size( 'test3' ) );
 pg_size_pretty
----------------
 95 MB
(1 row)
test3=# SELECT pg_size_pretty( pg_total_relation_size( 'test_table' ) );
 pg_size_pretty
----------------
 87 MB
(1 row)
```
## Отключил автовакуум на конкретной таблице
```
test3=# ALTER TABLE test_table SET (autovacuum_enabled = false);
ALTER TABLE
```
## Несколько раз обновил все строчки и добавил к каждой строчке рандомный  символ


##Посмотрел размер файла с таблицей.
```
 pg_size_pretty
----------------
 87 MB
(1 row)

test3=# SELECT pg_size_pretty( pg_database_size( 'test3' ) );
 pg_size_pretty
----------------
 95 MB
(1 row)
```
