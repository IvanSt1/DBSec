### Лабораторная работа №2-2: «Транзакции. Изоляция транзакций»
## Создать две сессии в Вашей базе данных. Начать транзакцию на уровне изоляции READ COMMITTED в одной из сессий. Изменить и добавить какие-либо данные. Пронаблюдать за изменениями из обеих сессий. Объяснить полученные результаты. Завершить транзакцию инструкцией ROLLBACK. Пронаблюдать за изменениями из обеих сессий. Объяснить полученные результаты.
Создаем две сессии:
Сессия 1:
```
bookstore=*# INSERT INTO inventory.books (title, author, isbn, price, quantity) VALUES ('New Book', 'Author A', '1234567890', 25.99, 10);
INSERT 0 1
bookstore=*# SELECT * FROM inventory.books;
 id |  title   |  author  |    isbn    | price | quantity 
----+----------+----------+------------+-------+----------
  1 | New Book | Author A | 1234567890 | 25.99 |       10
(1 row)
```
Сессия 2:
```
bookstore=# SELECT * FROM inventory.books;
 id | title | author | isbn | price | quantity 
----+-------+--------+------+-------+----------
(0 rows)
```
Транзакция не завершилась, поэтому во второй сессии нет данных добавленных в первой

Сессия 1:
```
postgres=*> ROLLBACK;
ROLLBACK
postgres=> select * from sneakershop.inventory;
 inventory_id | sneaker_id | size_value | quantity
--------------+------------+------------+----------
(0 rows)
```
Сессия 2:
```
bookstore=# SELECT * FROM inventory.books;
 id | title | author | isbn | price | quantity 
----+-------+--------+------+-------+----------
(0 rows)
```
Транзакция была завершена с инструкцией ROLLBACK, данные не были сохранены и вернулись в состояние перед началом транзакции.

## Внести изменения ещё раз и завершить транзакцию инструкцией COMMIT. Пронаблюдать за изменениями из обеих сессий. Объяснить полученные результаты.

Сессия 1:
```
bookstore=# START TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION
bookstore=*# INSERT INTO inventory.books (title, author, isbn, price, quantity) VALUES ('New Book', 'Author A', '1234567890', 25.99, 10);
INSERT 0 1
bookstore=*# SELECT * FROM inventory.books;
 id |  title   |  author  |    isbn    | price | quantity 
----+----------+----------+------------+-------+----------
  2 | New Book | Author A | 1234567890 | 25.99 |       10
(1 row)

bookstore=*# COMMIT;
COMMIT
bookstore=# SELECT * FROM inventory.books;
 id |  title   |  author  |    isbn    | price | quantity 
----+----------+----------+------------+-------+----------
  2 | New Book | Author A | 1234567890 | 25.99 |       10
(1 row)
```
Сессия 2:
```
bookstore=# SELECT * FROM inventory.books;
 id |  title   |  author  |    isbn    | price | quantity 
----+----------+----------+------------+-------+----------
  2 | New Book | Author A | 1234567890 | 25.99 |       10
(1 row)
```
Изменения появились в обоих сессиях, так как завершение транзакции с инструкцией COMMIT изменяет данные в базе данных.

## Выполнить те же операции для уровней изоляции REPEATABLE READ и SERIALIZABLE. Объяснить различия.
## REPEATABLE READ:
Сессия 1:
```
bookstore=# START TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION
bookstore=*# SELECT * FROM inventory.books;
 id |  title   |  author  |    isbn    | price | quantity 
----+----------+----------+------------+-------+----------
  2 | New Book | Author A | 1234567890 | 25.99 |       10
(1 row)
```
Сессия 2:
```
bookstore=# INSERT INTO inventory.books (title, author, isbn, price, quantity) VALUES ('Another Book', 'Author B', '0987654321', 19.99, 5);
INSERT 0 1

bookstore=# SELECT * FROM inventory.books;
 id |    title     |  author  |    isbn    | price | quantity 
----+--------------+----------+------------+-------+----------
  2 | New Book     | Author A | 1234567890 | 25.99 |       10
  3 | Another Book | Author B | 0987654321 | 19.99 |        5
(2 rows)
```
Сессия 1:
```
bookstore=*# COMMIT;
COMMIT
bookstore=# SELECT * FROM inventory.books;
 id |    title     |  author  |    isbn    | price | quantity 
----+--------------+----------+------------+-------+----------
  2 | New Book     | Author A | 1234567890 | 25.99 |       10
  3 | Another Book | Author B | 0987654321 | 19.99 |        5
(2 rows)
```
REPEATABLE READ в сесси2 аидит данные только в состояние в котором они были перед началом транзакции, поэтому только после COMMIT данные добавленные во второй сессии появились в первой сессии.
