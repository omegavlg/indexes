# Домашнее задание к занятию "`Индексы`" - `Дедюрин Денис`

---
## Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Ответ:
```
SELECT
    (SUM(INDEX_LENGTH) / SUM(DATA_LENGTH + INDEX_LENGTH)) * 100 AS index_to_table_ratio
FROM
    information_schema.TABLES
WHERE
    TABLE_SCHEMA = 'sakila';
```
<img src = "img/01.png" width = 100%>

---
## Задание 2
Выполните explain analyze следующего запроса:

```
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Ответ:
Выполняем **explain analyze** следующего запроса:
```
EXPLAIN ANALYZE
SELECT DISTINCT 
    CONCAT(c.last_name, ' ', c.first_name), 
    SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM 
    payment p
    JOIN rental r ON p.payment_date = r.rental_date
    JOIN customer c ON r.customer_id = c.customer_id
    JOIN inventory i ON r.inventory_id = i.inventory_id
    JOIN film f ON i.film_id = f.film_id
WHERE 
    DATE(p.payment_date) = '2005-07-30';
```


<img src = "img/02.png" width = 100%>

**Возможные узкие места:**

**Фильтрация по дате:** Использование функции DATE на колонке payment_date может привести к полному сканированию таблицы.

**JOIN операции:** Множество операций соединения могут быть медленными, если нет подходящих индексов.

**DISTINCT** и **WINDOW FUNCTION**: Могут быть затратными по ресурсам при работе с большими объемами данных.

**Для оптимизации на мой взгляд можно выполнить следующее:**

**Использование индексов:**

Добавление индексов на ключи, используемые в условиях соединения и фильтрации.

**Оптимизация фильтрации по дате:**

Избежать использования функции DATE, чтобы позволить использование индексов.

**Добавим индексы:**
```
CREATE INDEX idx_payment_date ON payment(payment_date);
CREATE INDEX idx_rental_date ON rental(rental_date);
CREATE INDEX idx_rental_customer ON rental(customer_id);
CREATE INDEX idx_inventory_id ON inventory(inventory_id);
CREATE INDEX idx_film_id ON film(film_id);
```

**Видоизменим сам запрос следующим образом:**
```
EXPLAIN ANALYZE
SELECT DISTINCT 
    CONCAT(c.last_name, ' ', c.first_name) AS customer_name, 
    SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title) AS total_amount
FROM 
    payment p
    JOIN rental r ON p.payment_date = r.rental_date
    JOIN customer c ON r.customer_id = c.customer_id
    JOIN inventory i ON r.inventory_id = i.inventory_id
    JOIN film f ON i.film_id = f.film_id
WHERE 
    p.payment_date BETWEEN '2005-07-30 00:00:00' AND '2005-07-30 23:59:59';
```

---
## Задание 3
Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

Приведите ответ в свободной форме.

### Ответ:

<img src = "img/03.png" width = 100%>
