### Домашнее задание к занятию "`12.5 "Реляционные базы данных: Индексы`" - `Спиривак Юрий`


### Задание 1

Напишите запрос к учебной базе данных, который вернет процентное отношение общего размера всех индексов к общему размеру всех таблиц.

```sql
select round((sum(INDEX_LENGTH) / sum(DATA_LENGTH) * 100), 1) "%"
FROM information_schema.TABLES WHERE TABLE_SCHEMA = 'sakila'
```

---

### Задание 2

Выполните explain analyze следующего запроса:

```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
```sql
Table scan on <temporary>  (cost=2.50..2.50 rows=0) (actual time=10431.463..10431.555 rows=391 loops=1)
    -> Temporary table with deduplication  (cost=0.00..0.00 rows=0) (actual time=10431.460..10431.460 rows=391 loops=1)
        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=5760.129..10033.973 rows=642000 loops=1)
            -> Sort: c.customer_id, f.title  (actual time=5760.088..5909.066 rows=642000 loops=1)
                -> Stream results  (cost=10650522.24 rows=16705435) (actual time=0.655..4684.080 rows=642000 loops=1)
                    -> Nested loop inner join  (cost=10650522.24 rows=16705435) (actual time=0.648..4093.772 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=8975802.39 rows=16705435) (actual time=0.643..3704.622 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=7301082.55 rows=16705435) (actual time=0.635..3226.519 rows=642000 loops=1)
                                -> Inner hash join (no condition)  (cost=1608774.80 rows=16086000) (actual time=0.618..151.249 rows=634000 loops=1)
                                    -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.68 rows=16086) (actual time=0.051..11.007 rows=634 loops=1)
                                        -> Table scan on p  (cost=1.68 rows=16086) (actual time=0.032..8.097 rows=16044 loops=1)
                                    -> Hash
                                        -> Covering index scan on f using idx_title  (cost=103.00 rows=1000) (actual time=0.063..0.387 rows=1000 loops=1)
                                -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.25 rows=1) (actual time=0.003..0.005 rows=1 loops=634000)
                            -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.00 rows=1) (actual time=0.000..0.000 rows=1 loops=642000)
                        -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.00 rows=1) (actual time=0.000..0.000 rows=1 loops=642000)
```

перечислите узкие места:
```sql
Inner hash join (no condition)  (cost=1608774.80 rows=16086000) (actual time=0.618..151.249 rows=634000 loops=1)
Nested loop inner join  (cost=7301082.55 rows=16705435) (actual time=0.635..3226.519 rows=642000 loops=1)
Nested loop inner join  (cost=8975802.39 rows=16705435) (actual time=0.643..3704.622 rows=642000 loops=1)
Nested loop inner join  (cost=10650522.24 rows=16705435) (actual time=0.648..4093.772 rows=642000 loops=1)
Stream results  (cost=10650522.24 rows=16705435) (actual time=0.655..4684.080 rows=642000 loops=1)
```
оптимизируйте запрос (внесите корректировки по использованию операторов, при необходимости добавьте индексы)
```sql
select distinct concat(c.last_name, ' ', c.first_name) "customer",
	   sum(p.amount) over (partition by c.customer_id, f.title) "sum",
	   f.title
from payment p
	join rental r on r.rental_id = p.rental_id
	join customer c on c.customer_id = r.customer_id
	join inventory i on i.inventory_id = r.inventory_id 
	join film f on f.film_id = i.film_id 
where date(p.payment_date) = '2005-07-30'
order by 1
```