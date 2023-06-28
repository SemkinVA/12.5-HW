# 12.5-HW
### Домашнее задание к занятию 12.5 - "Индексы" - Семкин Вячеслав
***
### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

**Ответ:**
```sql
select 	count(table_name) as "Количество таблиц",
		sum(data_length) as "Общий размер таблиц",
		sum(index_length) as "Общий размер индексов",
		concat(round(sum(index_length)/sum(data_length)*100), ' %') as "Процентное соотношение"
FROM INFORMATION_SCHEMA.tables
WHERE TABLE_SCHEMA = 'sakila';
```
***
### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

**Ответ:**

Впринципе будет достаточно убрать подключение к таблице film. Скорость обработки упадёт с 3.400s до 6-8ms. Запрос будет выглядеть  следующим образом:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
	from payment p, rental r, customer c, inventory i
	where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id;
```

Так же можно отказаться от distinct и over (partition by c.customer_id), но скорость обработки не изменить (останется 6-8ms). В итоге получим:
```sql
select concat(c.last_name, ' ', c.first_name) as ФИО, 
	   sum(p.amount) as "Общий платеж"
from payment p, rental r, customer c, inventory i
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
group by ФИО;
```

![2-1](https://github.com/SemkinVA/12.5-HW/blob/main/2-1.png)
***
