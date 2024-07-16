# Анализ сервиса вопросов и ответов по программированию

## Спринт

Продвинутый SQL
## Ключевые слова проекта

обработка данных, выгрузка данных, SQL
## Описание проекта

Написаны все сложные SQL-запросы для подсчёта требуемых значений и метрик.

## Навыки и инструменты

* SQL
* PostgreSQL
## Задания

## Часть 1

1. Найдите количество вопросов, которые набрали больше 300 очков или как минимум 100 раз были добавлены в «Закладки».
   ```SQL
   SELECT COUNT(id)
   FROM stackoverflow.posts
   WHERE post_type_id = 1 AND (favorites_count>=100 OR score>300)
   ```
2. Сколько в среднем в день задавали вопросов с 1 по 18 ноября 2008 включительно? Результат округлите до целого числа.
   ```SQL
   WITH s as (SELECT *
   FROM stackoverflow.posts
   WHERE DATE_TRUNC('day', creation_date)::date>='2008-11-01' and DATE_TRUNC('day', creation_date)::date<='2008-11-18'),
   d as (SELECT COUNT(s.id) as cnt
   FROM s
   LEFT JOIN stackoverflow.post_types pt on s.post_type_id=pt.id
   WHERE pt.type='Question'
   GROUP BY( DATE_TRUNC('day', creation_date)::date))
   SELECT ROUND(AVG(cnt))
   FROM d
   ```
3. Сколько пользователей получили значки сразу в день регистрации? Выведите количество уникальных пользователей.

   ```SQL
   SELECT COUNT(DISTINCT u.id)
   FROM stackoverflow.users u
   LEFT JOIN stackoverflow.badges b on b.user_id=u.id
   WHERE u.creation_date::date=b.creation_date::date
   ```
4. Сколько уникальных постов пользователя с именем Joel Coehoorn получили хотя бы один голос?
   ```SQL
   SELECT COUNT(DISTINCT p.id)
   FROM stackoverflow.users u 
   JOIN stackoverflow.posts p on p.user_id = u.id
   RIGHT JOIN stackoverflow.votes v on v.post_id=p.id
   WHERE display_name='Joel Coehoorn'
   ```
5. Выгрузите все поля таблицы vote_types. Добавьте к таблице поле rank, в которое войдут номера записей в обратном порядке. Таблица должна быть отсортирована по полю id.
   ```SQL
   SELECT *, ROW_NUMBER() OVER(ORDER BY id DESC) as rank
   FROM stackoverflow.vote_types
   ORDER BY id
   ```
6. Отберите 10 пользователей, которые поставили больше всего голосов типа Close. Отобразите таблицу из двух полей: идентификатором пользователя и количеством голосов. Отсортируйте данные сначала по убыванию количества голосов, потом по убыванию значения идентификатора пользователя.
   ```SQL
   SELECT u.id, COUNT(v.id)
   FROM stackoverflow.users u 
   JOIN stackoverflow.votes v on u.id=v.user_id
   JOIN stackoverflow.vote_types vt on vt.id=v.vote_type_id
   WHERE vt.name='Close'
   GROUP BY u.id
   ORDER BY count DESC, u.id DESC
   LIMIT 10
   ```
7. Отберите 10 пользователей по количеству значков, полученных в период с 15 ноября по 15 декабря 2008 года включительно.
Отобразите несколько полей:
идентификатор пользователя;
число значков;
место в рейтинге — чем больше значков, тем выше рейтинг.
Пользователям, которые набрали одинаковое количество значков, присвойте одно и то же место в рейтинге.
Отсортируйте записи по количеству значков по убыванию, а затем по возрастанию значения идентификатора пользователя.
   ```SQL
   SELECT u.id,  COUNT(b.id) as cnt, DENSE_RANK() OVER(ORDER BY COUNT(b.id) DESC)
   FROM stackoverflow.users u
   JOIN stackoverflow.badges b 
   ON u.id=b.user_id
   WHERE b.creation_date::date>='2008-11-15' AND b.creation_date::date<='2008-12-15'
   GROUP BY u.id
   ORDER BY cnt DESC, u.id
   LIMIT 10
   ```
8. Сколько в среднем очков получает пост каждого пользователя?
Сформируйте таблицу из следующих полей:
заголовок поста;
идентификатор пользователя;
число очков поста;
среднее число очков пользователя за пост, округлённое до целого числа.
Не учитывайте посты без заголовка, а также те, что набрали ноль очков.
   ```SQL
   SELECT p.title, p.user_id, p.score, ROUND(AVG(p.score) OVER(PARTITION BY p.user_id))
   FROM stackoverflow.posts p
   WHERE p.title IS NOT NULL and p.score<>0
   ```
9. Отобразите заголовки постов, которые были написаны пользователями, получившими более 1000 значков. Посты без заголовков не должны попасть в список.
   ```SQL
   SELECT DISTINCT(p.title)
   FROM stackoverflow.posts p
   JOIN stackoverflow.users u on p.user_id=u.id
   JOIN stackoverflow.badges b on b.user_id=u.id
   WHERE p.title IS NOT NULL AND u.id IN(
   SELECT user_id
   FROM stackoverflow.badges b
   GROUP BY user_id
   HAVING COUNT(b.id)>1000)
   ```
10. Напишите запрос, который выгрузит данные о пользователях из Канады (англ. Canada). Разделите пользователей на три группы в зависимости от количества просмотров их профилей:
пользователям с числом просмотров больше либо равным 350 присвойте группу 1;
пользователям с числом просмотров меньше 350, но больше либо равно 100 — группу 2;
пользователям с числом просмотров меньше 100 — группу 3.
Отобразите в итоговой таблице идентификатор пользователя, количество просмотров профиля и группу. Пользователи с количеством просмотров меньше либо равным нулю не должны войти в итоговую таблицу.
   ```SQL
   SELECT DISTINCT id, views,
   CASE WHEN views>=350 THEN 1
   WHEN views>=100 THEN 2
   WHEN views<100 THEN 3
   END as group
   FROM stackoverflow.users 
   WHERE views>0 AND location LIKE '%Canada%'
   ```
11. Дополните предыдущий запрос. Отобразите лидеров каждой группы — пользователей, которые набрали максимальное число просмотров в своей группе. Выведите поля с идентификатором пользователя, группой и количеством просмотров. Отсортируйте таблицу по убыванию просмотров, а затем по возрастанию значения идентификатора.
   ```SQL
   WITH s AS(SELECT DISTINCT id, views,
   CASE WHEN views>=350 THEN 1
   WHEN views>=100 THEN 2
   WHEN views<100 THEN 3
   END as groups
   FROM stackoverflow.users 
   WHERE views>0 AND location LIKE '%Canada%')
   SELECT *
   FROM s
   WHERE views IN( SELECT MAX(views) OVER(PARTITION BY groups ORDER BY views DESC) 
               FROM s)
   ORDER BY views DESC, id
   ```
12.  Посчитайте ежедневный прирост новых пользователей в ноябре 2008 года. Сформируйте таблицу с полями:
номер дня;
число пользователей, зарегистрированных в этот день;
сумму пользователей с накоплением.
   ```SQL
   WITH s AS(SELECT EXTRACT('day' from creation_date::date) as day, 
   COUNT(id) AS cnt
   FROM stackoverflow.users
   WHERE DATE_TRUNC('month', creation_date)::date>='2008-11-01'
   AND DATE_TRUNC('month', creation_date)::date<='2008-11-30'
   GROUP BY day)
   SELECT *, SUM(cnt) OVER ( ORDER BY day) AS cum_cnt
   FROM s
   ```
13. Для каждого пользователя, который написал хотя бы один пост, найдите интервал между регистрацией и временем создания первого поста. Отобразите:
идентификатор пользователя;
разницу во времени между регистрацией и первым постом.
   ```SQL
   WITH s AS 
   (SELECT user_id, 
       creation_date,
       RANK() OVER (PARTITION BY user_id ORDER BY creation_date) AS first_pub
       FROM stackoverflow.posts )

   SELECT user_id,
       s.creation_date - u.creation_date AS diff
   FROM s
   JOIN stackoverflow.users u on s.user_id = u.id
   WHERE first_pub = 1
   ```

## Часть 2

1. Выведите общую сумму просмотров у постов, опубликованных в каждый месяц 2008 года. Если данных за какой-либо месяц в базе нет, такой месяц можно пропустить. Результат отсортируйте по убыванию общего количества просмотров.
   ```SQL
   SELECT SUM(views_count), DATE_TRUNC('month', creation_date)::date as mnth
   FROM stackoverflow.posts p
   GROUP BY mnth
   ORDER BY sum DESC
   ```
2. Выведите имена самых активных пользователей, которые в первый месяц после регистрации (включая день регистрации) дали больше 100 ответов. Вопросы, которые задавали пользователи, не учитывайте. Для каждого имени пользователя выведите количество уникальных значений user_id. Отсортируйте результат по полю с именами в лексикографическом порядке.
   ```SQL
   SELECT u.display_name, COUNT(DISTINCT p.user_id)
   FROM stackoverflow.users u
   JOIN stackoverflow.posts p on u.id=p.user_id
   JOIN stackoverflow.post_types pt on pt.id=p.post_type_id
   WHERE pt.type='Answer' AND p.creation_date::date BETWEEN u.creation_date::date AND (u.creation_date::date + INTERVAL '1 month') 
   GROUP BY u.display_name
   HAVING COUNT(p.id)>100
   ORDER BY 1
   ```
3. Выведите количество постов за 2008 год по месяцам. Отберите посты от пользователей, которые зарегистрировались в сентябре 2008 года и сделали хотя бы один пост в декабре того же года. Отсортируйте таблицу по значению месяца по убыванию.
   ```SQL
   SELECT COUNT(p.id),
   DATE_TRUNC('month', p.creation_date)::date as mnth
   FROM stackoverflow.posts p
   JOIN stackoverflow.users u on p.user_id=u.id
   WHERE EXTRACT('year' from p.creation_date)=2008 AND 
   u.id IN (
       SELECT u.id
       FROM stackoverflow.users u 
       JOIN stackoverflow.posts p on p.user_id=u.id
       WHERE EXTRACT('month' from u.creation_date)=9 
       AND EXTRACT('month' from p.creation_date)=12)
   GROUP BY 2
   ORDER BY 2 DESC
   ```
4. Используя данные о постах, выведите несколько полей:
идентификатор пользователя, который написал пост;
дата создания поста;
количество просмотров у текущего поста;
сумма просмотров постов автора с накоплением.
Данные в таблице должны быть отсортированы по возрастанию идентификаторов пользователей, а данные об одном и том же пользователе — по возрастанию даты создания поста.
   ```SQL
   SELECT p.user_id, p.creation_date, p.views_count, 
   SUM(views_count) OVER(PARTITION BY user_id ORDER BY creation_date)
   FROM stackoverflow.posts p
   ORDER BY user_id
   ```
5. Сколько в среднем дней в период с 1 по 7 декабря 2008 года включительно пользователи взаимодействовали с платформой? Для каждого пользователя отберите дни, в которые он или она опубликовали хотя бы один пост. Нужно получить одно целое число — не забудьте округлить результат.
   ```SQL
   WITH s AS (SELECT (COUNT(DISTINCT DATE_TRUNC('day', creation_date)::date)) as cnt
   FROM stackoverflow.posts p
   WHERE creation_date::date BETWEEN '2008-12-01' AND '2008-12-07'
   GROUP BY user_id)
   SELECT ROUND(AVG(cnt))
   FROM s
   ```
6. На сколько процентов менялось количество постов ежемесячно с 1 сентября по 31 декабря 2008 года? Отобразите таблицу со следующими полями:
Номер месяца.
Количество постов за месяц.
Процент, который показывает, насколько изменилось количество постов в текущем месяце по сравнению с предыдущим.
Если постов стало меньше, значение процента должно быть отрицательным, если больше — положительным. Округлите значение процента до двух знаков после запятой.
   ```SQL
   WITH s AS(SELECT EXTRACT('month' from creation_date) as mnth, 
          COUNT(id)
   FROM stackoverflow.posts p
   WHERE creation_date::date between '2008-09-01' AND '2008-12-31'
   GROUP BY mnth)
   SELECT *, 
   ROUND(((count-LAG(count, 1) OVER ())/LAG(count, 1) OVER ()::numeric)*100, 2)
   FROM s
   ```
7. Найдите пользователя, который опубликовал больше всего постов за всё время с момента регистрации. Выведите данные его активности за октябрь 2008 года в таком виде:
номер недели;
дата и время последнего поста, опубликованного на этой неделе.
   ```SQL
   WITH s AS(SELECT user_id, COUNT(id)
   FROM stackoverflow.posts p
   GROUP BY user_id
   ORDER BY 2 DESC
   LIMIT 1)

   SELECT EXTRACT('week' from creation_date) as nweek,
   MAX(creation_date)
   FROM s
   JOIN stackoverflow.posts p on p.user_id=s.user_id
   WHERE DATE_TRUNC('month', creation_date)='2008-10-01'
   GROUP BY nweek
   ```
