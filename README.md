1. Напишите запрос, который выводит столбцы «Название фильма» (movie_title), 
«Режиссёр» (director), «Сценарист» (screenwriter), «Актёры» (actors). 
Оставьте только те фильмы, у которых:

рейтинг между 8 и 8.5 (включительно) ИЛИ год выхода в прокат до 1990;
есть описание;
название начинается не с буквы 'Т';
название состоит ровно из 12 символов.

Оставьте только топ-7 по рейтингу.

SELECT 
	movie_title,
	director,
	screenwriter,
	actors
FROM sql.kinopoisk
where (rating between 8.0 and 8.5 or year < 1990)
	and overview is not null
	and movie_title not like 'Т%'
    and movie_title like '____________'
order by rating desc
limit 7

2. Напишите запрос, который выведет:

количество покемонов (столбец pokemon_count),
среднюю скорость (столбец avg_speed),
максимальное и минимальное число очков здоровья (столбцы max_hp и min_hp)
для электрических (Electric) покемонов, имеющих дополнительный тип 
и показатели атаки или защиты больше 50.

SELECT
	count (*) as pokemon_count,
	avg(speed) as avg_speed,
	max(hp) as max_hp,
	min(hp) as min_hp
FROM sql.pokemon
where type1 = 'Electric' 
	and type2 is not null
	and (attack > 50 or defense > 50)

3. Напишите запрос, который выведет столбцы с основным типом покемона
и общим количеством покемонов этого типа.
Учитывайте только тех покемонов, у кого или показатель атаки, 
или показатель защиты принимает значение между 50 и 100 включительно.
Оставьте только те типы покемонов, у которых максимальный 
показатель здоровья не больше 125.
Выведите только тот тип, который находится на пятом месте 
по количеству покемонов.

SELECT
	type1,
	count(type1)
FROM sql.pokemon
where (attack between 50 and 100) or (defense between 50 and 100)
group by 1
having max(hp) <= 125
order by 2 desc
offset 4 limit 1

4. Выведите количество матчей между командами Real Madrid CF и FC Barcelona.

select
	count(m.id)
FROM
	sql.matches as m
    join sql.teams as rm
	on m.home_team_api_id = rm.api_id
	join sql.teams as b
	on m.away_team_api_id = b.api_id 
where (rm.long_name = 'Real Madrid CF' and b.long_name = 'FC Barcelona') 
	or (b.long_name = 'Real Madrid CF' and rm.long_name = 'FC Barcelona')
	
5. Напишите запрос, который объединит в себе все почтовые индексы водителей и их телефоны в единый столбец-справочник. 
Также добавьте столбец с именем водителя и столбец с типом контакта (phone или zip в зависимости от типа).
Отсортируйте список по столбцу с контактными данными в порядке возрастания, а затем — по имени водителя.

select 
	zip_code::text,
	first_name,
	'zip'
from 
	driver
union all
select
	phone,
	first_name,
	'phone'
from 
	driver
order by 1, 2

6. Напишите запрос, который выводит два столбца: city_name и shippings_fake. Выведите города, куда совершались доставки.
Пусть первый столбец содержит название города, а второй формируется так:
если в городе было более десяти доставок, вывести количество доставок в этот город как есть;
иначе — вывести количество доставок, увеличенное на пять.
Отсортируйте по убыванию получившегося «нечестного» количества доставок, а затем — по имени в алфавитном порядке.

SELECT
	city_name,
	count(ship_id) as shippings_fake
FROM
    city as c join shipment as s
	on c.city_id = s.city_id
group by 1
having count(ship_id) > 10
union all
SELECT
	city_name,
	count(ship_id)+5 as shippings_fake
FROM
    city as c join shipment as s
	on c.city_id = s.city_id
group by 1
having count(ship_id) <= 10
order by 2 desc, 1

7. Выведите города с максимальным и минимальным весом единичной доставки.

SELECT
	c.city_name,
	s.weight
FROM
    city as c join shipment as s
	on c.city_id = s.city_id
where s.weight = (select min(s.weight) from shipment as s)
union all
SELECT
	c.city_name,
	s.weight
FROM
    city as c join shipment as s
	on c.city_id = s.city_id
where s.weight = (select max(s.weight) from shipment as s)
order by 2 desc

8. Посчитайте среднее количество товаров в чеке за каждый день.
Отсортируйте запрос по столбцу с датой.

select 
	tr_date,
	avg(su)
from (select
	tr_date,
	tr_id,
	sum(quantity) as su
from sql.coffeeshop_sales as s
group by 1,2
) s
group by 1
order by 1

9. Выведите данные по средней сумме чека за каждый день.
Отсортируйте запрос по столбцу с датой.

SELECT
	tr_date,
	AVG (daily_revenue) avg_rev_daily
FROM 
(
SELECT
	cs.tr_id,
	cs.tr_date,
	SUM(cs.quantity * cp.price) daily_revenue
FROM 
	sql.coffeeshop_sales cs
	JOIN sql.coffeeshop_products cp ON cs.product_id = cp.product_id
GROUP BY 1,2
) avg_revenue_by_day
GROUP BY 1
ORDER BY 1

10. Для товарного исследования необходимо предоставить данные о сумме продаж по напиткам (Drinks) 
и выпечке (Foods) в формате: категория продукта, выручка.
Напишите запрос для вывода необходимых данных, отсортируйте результат по сумме в порядке убывания.
Напитки: product_type содержит chocolate / barista / brewed / drip / drink / syrup.
Выпечка: product_type содержит pastry / scone / biscotti.

WITH product_type AS 
(
SELECT
	'Drinks' new_category, 
	cp.product_id,
	cp.price
FROM 
	sql.coffeeshop_products cp
WHERE
	cp.product_type ILIKE '%chocolate%'
	OR cp.product_type ILIKE '%barista%'
	OR cp.product_type ILIKE '%brewed%'
	OR cp.product_type ILIKE '%drip%'
	OR cp.product_type ILIKE '%drink%'
	OR cp.product_type ILIKE '%syrup%'
UNION ALL
SELECT 
	'Foods' new_category,
	cp.product_id,
	cp.price
FROM
	sql.coffeeshop_products cp 
WHERE
	cp.product_type ILIKE '%pastry%'
	OR cp.product_type ILIKE '%scone%'
	OR cp.product_type ILIKE '%biscotti%')
SELECT
	pt.new_category,
	SUM(cs.quantity * pt.price)
FROM
	sql.coffeeshop_sales cs
	JOIN product_type pt ON cs.product_id = pt.product_id
GROUP BY 1
order by 2 desc

11. Для анализа портрета покупателя необходимо предоставить данные по клиентам, 
которые сначала совершили покупку онлайн, а потом пришли в кафе лично. Формате вывода:
номер клиента, имя клиента.
Отсортируйте запрос по id пользователя в порядке возрастания.

SELECT
    DISTINCT
    cc.cust_id,
    cc.cust_name
FROM
    (SELECT
        cs.cust_id,
        MIN(cs.tr_date) online_date
    FROM
        sql.coffeeshop_sales cs
    WHERE
        instore_flg = 'N'
    GROUP BY 1
    ) online
    JOIN sql.coffeeshop_sales cs ON cs.cust_id = online.cust_id
    JOIN sql.coffeeshop_custs cc ON online.cust_id = cc.cust_id
WHERE
    cs.instore_flg = 'Y'
    AND online.online_date < cs.tr_date
ORDER BY 1

12. Напишите запрос, который выведет год, месяц и количество доставок.
Отсортируйте по году и по месяцу в порядке возрастания.
Столбцы в выдаче: year_n (номер года), month_n (номер месяца), qty (количество доставок).

SELECT 
	EXTRACT(YEAR FROM ship_date) as year_n,
	EXTRACT(MONTH FROM ship_date) as month_n,
	COUNT(ship_id)
from shipment
group by 1,2
order by 1,2 

13. Напишите запрос, который выведет дату доставки, округлённую до квартала, и общую массу доставок.
Отсортируйте по кварталу в порядке возрастания.
Столбцы в выдаче: q (начало квартала, тип date), total_weight (сумма масс доставок за квартал)

select
	date_trunc('quarter', ship_date)::date as q,
	sum(weight) as total_weight 
from
	shipment
group by 1
order by 1

14. Напишите запрос, который выведет разницу между последним и первым днём доставки по каждому городу.
Отсортируйте по первому и второму столбцам.
Столбцы в выдаче: city_name (название города) и days_active (время от первой до последней доставки в днях).

select
	city_name,
	max(ship_date)- min(ship_date) as days_active
from
	city as c join shipment as s
	on c.city_id = s.city_id
group by 1
order by 1,2
