1.Напишите запрос, который выводит столбцы «Название фильма» (movie_title), 
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

2.Напишите запрос, который выведет:

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
