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
