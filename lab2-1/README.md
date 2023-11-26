## Лабораторная работа №2-1: «Пользователи. Роли. Привилегии»

### Структура базы данных

```sql
-- Таблица брендов кроссовок
CREATE TABLE sneakers (
    sneaker_id SERIAL PRIMARY KEY,
    model VARCHAR(100) NOT NULL,
    brand VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);

-- Таблица запасов
CREATE TABLE inventory (
    inventory_id SERIAL PRIMARY KEY,
    sneaker_id INT REFERENCES sneakers(sneaker_id),
    size_value VARCHAR(10) NOT NULL,
    quantity INT NOT NULL
);

-- Таблица продаж
CREATE TABLE sales (
    sale_id SERIAL PRIMARY KEY,
    sneaker_id INT REFERENCES sneakers(sneaker_id),
    sale_date DATE NOT NULL,
    quantity INT NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL
);
```

### Определить, в какой схеме находятся таблицы Вашей базы данных. Следует ли изменить схему?

Для определения схемы используем данный запрос

```sql
SELECT table_schema, table_name FROM information_schema.tables WHERE table_schema NOT LIKE 'pg_%' and table_schema != 'information_schema';
```

Результат:

```sql
 table_schema | table_name
--------------+------------
 public       | sneakers
 public       | inventory
 public       | sales
```



### Следует ли изменить схему? 

Оставление всех объектов в схеме "public" не является запрещенным, и в некоторых случаях это может быть даже удобным, особенно при разработке простых приложений. Однако есть несколько причин, почему часто рекомендуется избегать использования схемы "public" для всех объектов в более сложных приложениях:

1. Безопасность
2. Управление именами объектов(избегание конфликтов имен в сложных базах данных)
3. Изоляция данных


### Следует ли создать несколько отдельных схем для выбранной предметной области? Почему?

Для создания нескольких отдельных схем есть несколько причин:

1. Логическое разделение данных: Использование схемы может обеспечить логическое разделение различных компонентов вашей системы. Например, можно иметь схему для управления продуктами, другую для управления пользователями и третью для учета финансов.

2. Улучшенная безопасность: Разделение данных по схемам может помочь в управлении безопасностью. Мы можем устанавливать различные права доступа к схемам для разных пользователей или ролей, что обеспечивает более гранулированный контроль доступа.

```sql
-- Создание схемы sneakershop
CREATE SCHEMA sneakershop;

-- Изменение схемы таблицы sales на sneakershop
ALTER TABLE sales SET SCHEMA sneakershop;

-- Изменение схемы таблицы sneakers на sneakershop
ALTER TABLE sneakers SET SCHEMA sneakershop;

-- Изменение схемы таблицы inventory на sneakershop
ALTER TABLE inventory SET SCHEMA sneakershop;
```

Теперь вывод команды из пункта выше выглядит так
```sql
 table_schema | table_name
--------------+------------
 sneakershop  | sales
 sneakershop  | sneakers
 sneakershop  | inventory
```

---

### Определить, какие роли нужны для нормального функционирования Вашей базы данных. Какие системные и объектные привилегии потребуются каждой роли? Понадобятся ли вложенные роли?

1. **Системные роли:**
   - `superuser`: Возможно, нам потребуются одна или несколько ролей с полными административными правами для управления всей базой данных, особенно при создании и обновлении схемы.

2. **Объектные роли:**
   - **Роли для управления продуктами и инвентаризацией:**
     - `product_manager`: Роль для управления продуктами (таблица sneakers).
     - `inventory_manager`: Роль для управления инвентаризацией (таблица inventory).
   - **Роли для безопасности:**
     - `public`: Это системная роль, к которой по умолчанию у всех пользователей есть доступ. Однако, **нам**, возможно, придется уточнить права доступа для конкретных объектов, чтобы обеспечить безопасность.
     - `read_only_user`: Роль, которая может выполнять только операции чтения.
   - **Вложенная роль:**
     - `admin`: Роль, включающая в себя `product_manager`, `inventory_manager` и другие необходимые роли ().

---
### Создать роли и выдать им необходимые объектные и системные привилегии


Пример создания ролей:

```sql
-- Создание системных ролей
CREATE ROLE superuser_role SUPERUSER;

-- Создание объектных ролей
CREATE ROLE product_manager;
CREATE ROLE inventory_manager;
CREATE ROLE read_only_user;

-- Создание иерархии ролей
CREATE ROLE admin;
GRANT product_manager TO admin;
GRANT inventory_manager TO admin;
GRANT read_only_user TO admin;
```

Пример выдачи прав:

```sql
-- Предоставление администратору доступа к таблице продаж
GRANT INSERT, UPDATE, DELETE, SELECT ON TABLE sneakershop.sales TO admin;

-- Предоставление прав менеджеру продаж на управление данными только в таблице sneakers
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA sneakershop TO product_manager;
GRANT INSERT, UPDATE, DELETE, SELECT ON TABLE sneakershop.sneakers TO product_manager;

-- Предоставление прав менеджеру инвентаризации на управление данными только в таблице inventory
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA sneakershop TO inventory_manager;
GRANT INSERT, UPDATE, DELETE, SELECT ON TABLE sneakershop.inventory TO inventory_manager;

-- Предоставление прав читателю на чтение данных из всех таблиц
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA sneakershop TO read_only_user;
GRANT SELECT ON ALL TABLES IN SCHEMA sneakershop TO read_only_user;
```
---

###  Проверить по представлению системного каталога pg_catalog.pg roles, что все нужные роли были созданы и обладают корректным набором привилегий;

Команда:
```sql
SELECT grantee, privilege_type, table_name
FROM information_schema.table_privileges
WHERE table_schema = 'sneakershop' 
ORDER BY grantee;
```

Вывод:
```sql
      grantee      | privilege_type | table_name
-------------------+----------------+------------
 admin             | TRIGGER        | inventory
 admin             | INSERT         | sales
 admin             | SELECT         | sales
 admin             | UPDATE         | sales
 admin             | DELETE         | sales
 admin             | TRUNCATE       | sales
 admin             | REFERENCES     | sales
 admin             | TRIGGER        | sales
 admin             | UPDATE         | sneakers
 admin             | DELETE         | sneakers
 admin             | TRUNCATE       | sneakers
 admin             | REFERENCES     | sneakers
 admin             | TRIGGER        | sneakers
 admin             | INSERT         | inventory
 admin             | SELECT         | inventory
 admin             | UPDATE         | inventory
 admin             | DELETE         | inventory
 admin             | TRUNCATE       | inventory
 admin             | REFERENCES     | inventory
 admin             | INSERT         | sneakers
 admin             | SELECT         | sneakers
 inventory_manager | UPDATE         | inventory
 inventory_manager | DELETE         | inventory
 inventory_manager | INSERT         | inventory
 inventory_manager | SELECT         | inventory
 product_manager   | SELECT         | sneakers
 product_manager   | UPDATE         | sneakers
 product_manager   | INSERT         | sneakers
 product_manager   | DELETE         | sneakers
 read_only_user    | SELECT         | inventory
 read_only_user    | SELECT         | sneakers
 read_only_user    | SELECT         | sales
```
---

### Попробовать подключиться от лица каждой роли (из тех, которым разрешено подключение к серверу БД). Убедиться, что роль имеет доступ к разрешённым данным и не имеет доступа ко всем остальным.

Разрешаем вход
```sql
ALTER ROLE inventory_manager WITH PASSWORD 'secret';
ALTER ROLE inventory_manager LOGIN;
```

Проверка прав для inventory_manager:

```sql
postgres=> select * from sneakershop.inventory;
 inventory_id | sneaker_id | size_value | quantity
--------------+------------+------------+----------
(0 rows)

postgres=> select * from sneakershop.sales;
ERROR:  permission denied for table sales

postgres=> select * from sneakershop.sneakers;
ERROR:  permission denied for table sneakers
```
---