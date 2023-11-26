## Лабораторная работа №2-2: «Транзакции. Изоляция транзакций»

### Создать две сессии в Вашей базе данных. Начать транзакцию на уровне изоляции READ COMMITTED в одной из сессий. Изменить и добавить какие-либо данные. Пронаблюдать за изменениями из обеих сессий. Объяснить полученные результаты. Завершить транзакцию инструкцией ROLLBACK. Пронаблюдать за изменениями из обеих сессий. Объяснить полученные результаты.

Создаем две сессии:

Сессия 1:
```sql
-- Начало транзакции с уровнем изоляции READ COMMITTED
START TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- Теперь сессия 1 находится в режиме транзакции с уровнем изоляции READ COMMITTED


postgres=*> INSERT INTO sneakershop.inventory VALUES (13, 37, 'big size', 150);
INSERT 0 1
postgres=*> select * from sneakershop.inventory;
 inventory_id | sneaker_id | size_value | quantity
--------------+------------+------------+----------
           13 |         37 | big size   |      150
```

Сессия 2:
```sql
postgres=# select * from sneakershop.inventory;
 inventory_id | sneaker_id | size_value | quantity
--------------+------------+------------+----------
(0 rows)
```
Изменений не произошло, так как транзакция не завершилась

Сессия 1:

```sql
postgres=*> ROLLBACK;
ROLLBACK
postgres=> select * from sneakershop.inventory;
 inventory_id | sneaker_id | size_value | quantity
--------------+------------+------------+----------
(0 rows)
```
Изменения не были сохранены в базу данных, так как была введена команда ROLLBACK (изменения откатились в начальное состояние).

---
### Внести изменения ещё раз и завершить транзакцию инструкцией COMMIT. Пронаблюдать за изменениями из обеих сессий. Объяснить полученные результаты.

Сессия 1:

```sql
postgres=> START TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION
postgres=*> INSERT INTO sneakershop.inventory VALUES (13, 37, 'big size', 150);
INSERT 0 1
postgres=*> COMMIT;
COMMIT
postgres=> select * from sneakershop.inventory;
 inventory_id | sneaker_id | size_value | quantity
--------------+------------+------------+----------
           13 |         37 | big size   |      150
(1 row)
```

Сессия 2:
```sql
postgres=# select * from sneakershop.inventory;
 inventory_id | sneaker_id | size_value | quantity
--------------+------------+------------+----------
           13 |         37 | big size   |      150
(1 row)
```

Изменения появились и во второй сессии, так как команда COMMIT зафиксировала изменения в базе данных.

---
###  Выполнить те же операции для уровней изоляции REPEATABLE READ и SERIALIZABLE. Объяснить различия.

Сессия 1:
```sql
postgres=> START TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION
postgres=*> select * from sneakershop.inventory;
 inventory_id | sneaker_id | size_value | quantity
--------------+------------+------------+----------
           13 |         37 | big size   |      150
(1 row)
```

Сессия 2:
```sql
postgres=# UPDATE sneakershop.inventory SET quantity = 300 WHERE size_value ='big size';
UPDATE 1
```

Сессия 1:
```sql
postgres=*> select * from sneakershop.inventory;
 inventory_id | sneaker_id | size_value | quantity
--------------+------------+------------+----------
           13 |         37 | big size   |      150
(1 row)

postgres=*> COMMIT;
COMMIT
postgres=> select * from sneakershop.inventory;
 inventory_id | sneaker_id | size_value | quantity
--------------+------------+------------+----------
           13 |         37 | big size   |      300
(1 row)
```

Изменения внесенные в Сессии 2 появляются в Сессии 1, только после COMMIT. Так как в `REPEAATABLE READ` сессия 1 видит данные в том состоянии, в котором они были на момент начала транзакции.

### SERIALIZABLE

Сессия 1:
```sql
START TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION
postgres=*> select * from sneakershop.inventory;
 inventory_id | sneaker_id | size_value | quantity
--------------+------------+------------+----------
           13 |         37 | big size   |      150
(1 row)
postgres=*>  UPDATE sneakershop.inventory SET inventory_id = 25 WHERE size_value ='big size';
UPDATE 1
```

Сессия 2:
```sql
START TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION
postgres=# UPDATE sneakershop.inventory SET quantity = 300 WHERE size_value ='big size';
UPDATE 1
ERROR:  could not serialize access due to concurrent update
```

Уровень изоляции Serializable не дает одновременно изменить одну и ту же строку в разных транзакциях.


## Результаты:
Было произведено знакомство с тремя уровнями изоляции транзакций.

1. **READ COMMITTED:**
   - Низкий уровень изоляции.
   - Изменения, внесенные другими транзакциями, видны после их фиксации.
   - Не предотвращает изменения данных внутри текущей транзакции.

2. **REPEATABLE READ:**
   - Более высокий уровень изоляции, чем `READ COMMITTED`.
   - Гарантирует стабильность данных в рамках транзакции.
   - Предотвращает чтение новых данных, соответствующих условиям запроса, в течение транзакции.

3. **SERIALIZABLE:**
   - Самый высокий уровень изоляции.
   - Гарантирует стабильность данных и предотвращает изменения данных другими транзакциями в рамках текущей транзакции.
   - Видимость данных соответствует их состоянию на момент начала транзакции.
