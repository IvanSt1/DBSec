# Лабораторная работа №2-1: «Пользователи. Роли. Привилегии»

## Структура базы данных

```
-- Таблица с информацией о книгах.
CREATE TABLE books (
id SERIAL PRIMARY KEY,
title VARCHAR(100),
author VARCHAR(100),
isbn VARCHAR(13),
price DECIMAL,
quantity INT);

--Таблица с информацией о поставщиках книг.
CREATE TABLE suppliers (
id SERIAL PRIMARY KEY,
name VARCHAR(100),
contact_info VARCHAR(100));
);

--Таблица с информацией о заказах 
CREATE TABLE orders (
id SERIAL PRIMARY KEY,
order_date DATE,
customer_id INT,
total_cost DECIMAL
);

--Таблица с информацией о клиентах
CREATE TABLE customers (
id SERIAL PRIMARY KEY,
name VARCHAR(100),
contact_info VARCHAR(100)
);
```

## Определить, в какой схеме находятся таблицы Вашей базы данных. Следует ли изменить схему?

Запрос для определения схемы

```
SELECT table_schema, table_name FROM information_schema.tables WHERE table_schema NOT LIKE 'pg_%' and table_schema != 'information_schema';
```

Результат:
```
 table_schema | table_name 
--------------+------------
 public       | books
 public       | suppliers
 public       | orders
 public       | customers
(4 rows)
```

## Следует ли изменить схему?

Использование схемы "public" для всех объектов в базе данных не всегда является неправильным и может быть удобным в разработке простых приложений. Тем не менее, в более сложных приложениях обычно рекомендуется избегать этого подхода по нескольким причинам:

Безопасность: Чтобы уменьшить риск несанкционированного доступа или вмешательства.
Управление именами объектов: Для предотвращения конфликтов имен в масштабных или сложных базах данных.
Изоляция данных: Для обеспечения чёткого разделения и управления данными.

## Следует ли создать несколько отдельных схем для выбранной предметной области? Почему?

Существует несколько важных причин для создания разных схем в базе данных:

Логическое разграничение данных: Разные схемы позволяют четко разделять различные аспекты системы. Например, можно использовать одну схему для хранения данных о продуктах, другую для управления информацией о пользователях, и третью для финансовой отчетности.

Повышение уровня безопасности: Разделение данных на схемы улучшает управление безопасностью. Установка разных прав доступа к разным схемам для отдельных пользователей или групп пользователей позволяет детализировать контроль доступа.

```
-- Создание схемы для отслеживания запасов книг
CREATE SCHEMA inventory;
-- Создание схемы для отслеживания продаж
CREATE SCHEMA sales;

-- Добавление таблиц в соответсвующие схемы
ALTER TABLE books SET SCHEMA inventory;
ALTER TABLE suppliers SET SCHEMA inventory;
ALTER TABLE orders SET SCHEMA sales;
ALTER TABLE customers SET SCHEMA sales;
```

## Определить, какие роли нужны для нормального функционирования Вашей базы данных. Какие системные и объектные привилегии потребуются каждой роли? Понадобятся ли вложенные роли?

1. Системные роли:
  + superuser: Роль полными административными правами для управления всей базой данных.
2. Объектные роли:
  + Роли для управления продуктами и инвентаризацией:
    + inventory_manager: Роль для управления запасами. Нужен доступ на чтение и запись в схему inventory.
    + sales_manager: Роль для управления продажами. Нужен доступ на чтение и запись в схему sales.
  + Роли для безопасности:
    + report_viewer: Роль для просмотра отчетов. Только чтение из обеих схем.
## Создать роли и выдать им необходимые объектные и системные привилегии
Создание ролей:
```
CREATE ROLE superuser WITH LOGIN SUPERUSER PASSWORD 'secure_password';
CREATE ROLE inventory_manager;
CREATE ROLE sales_manager;
CREATE ROLE report_viewer;

```
Выдача прав для ролей:
```
GRANT USAGE ON SCHEMA inventory TO inventory_manager;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA inventory TO inventory_manager;

GRANT USAGE ON SCHEMA sales TO sales_manager;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA sales TO sales_manager;

GRANT USAGE ON SCHEMA inventory TO report_viewer;
GRANT USAGE ON SCHEMA sales TO report_viewer;
GRANT SELECT ON ALL TABLES IN SCHEMA inventory TO report_viewer;
GRANT SELECT ON ALL TABLES IN SCHEMA sales TO report_viewer;

```
## Проверить по представлению системного каталога pg_catalog.pg roles, что все нужные роли были созданы и обладают корректным набором привилегий;
Команда:
```
SELECT grantee, privilege_type, table_name
FROM information_schema.table_privileges
WHERE table_schema = 'inventory' 
ORDER BY grantee;
```
Вывод:
```
  grantee      | privilege_type | table_name 
-------------------+----------------+------------
 inventory_manager | UPDATE         | books
 inventory_manager | DELETE         | books
 inventory_manager | DELETE         | suppliers
 inventory_manager | INSERT         | books
 inventory_manager | SELECT         | books
 inventory_manager | INSERT         | suppliers
 inventory_manager | SELECT         | suppliers
 inventory_manager | UPDATE         | suppliers
 postgres          | SELECT         | suppliers
 postgres          | UPDATE         | suppliers
 postgres          | DELETE         | suppliers
 postgres          | TRUNCATE       | suppliers
 postgres          | REFERENCES     | suppliers
 postgres          | TRIGGER        | suppliers
 postgres          | INSERT         | books
 postgres          | SELECT         | books
 postgres          | UPDATE         | books
 postgres          | DELETE         | books
 postgres          | TRUNCATE       | books
 postgres          | REFERENCES     | books
 postgres          | TRIGGER        | books
 postgres          | INSERT         | suppliers
 report_viewer     | SELECT         | books
 report_viewer     | SELECT         | suppliers
```
Команда:
```
SELECT grantee, privilege_type, table_name
FROM information_schema.table_privileges
WHERE table_schema = 'sales' 
ORDER BY grantee;
```
Вывод:
```
    grantee    | privilege_type | table_name 
---------------+----------------+------------
 postgres      | INSERT         | customers
 postgres      | SELECT         | customers
 postgres      | UPDATE         | customers
 postgres      | DELETE         | customers
 postgres      | TRUNCATE       | customers
 postgres      | REFERENCES     | customers
 postgres      | TRIGGER        | customers
 postgres      | INSERT         | orders
 postgres      | SELECT         | orders
 postgres      | UPDATE         | orders
 postgres      | DELETE         | orders
 postgres      | TRUNCATE       | orders
 postgres      | REFERENCES     | orders
 postgres      | TRIGGER        | orders
 report_viewer | SELECT         | customers
 report_viewer | SELECT         | orders
 sales_manager | UPDATE         | orders
 sales_manager | INSERT         | customers
 sales_manager | SELECT         | customers
 sales_manager | INSERT         | orders
 sales_manager | DELETE         | customers
 sales_manager | DELETE         | orders
 sales_manager | UPDATE         | customers
 sales_manager | SELECT         | orders
(24 rows)

```
## Попробовать подключиться от лица каждой роли (из тех, которым разрешено подключение к серверу БД). Убедиться, что роль имеет доступ к разрешённым данным и не имеет доступа ко всем остальным.
```
bookstore=# SET ROLE inventory_manager;
SET
bookstore=> SELECT * FROM inventory.suppliers;
 id | name | contact_info 
----+------+--------------
(0 rows)

bookstore=> SELECT * FROM inventory.books;
 id | title | author | isbn | price | quantity 
----+-------+--------+------+-------+----------
(0 rows)

bookstore=> SELECT * FROM sales.customers;
ERROR:  permission denied for schema sales
LINE 1: SELECT * FROM sales.customers;
                      ^
bookstore=> SELECT * FROM sales.orders;
ERROR:  permission denied for schema sales
LINE 1: SELECT * FROM sales.orders;
                      ^
```

### Результаты:
В результате данной лабораторной работы было произведено ознакомление со схемами и ролями в PostgreSQL. Для тестовой базы данных магазина книг были созданы схема и несколько ролей, которым были выданы необходимые привилегии для работы с определенными таблицами.
