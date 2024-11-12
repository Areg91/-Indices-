                                                                              Домашнее задание к занятию «Индексы» 
Инструкция по выполнению домашнего задания
Сделайте fork репозитория c шаблоном решения к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
Выполните клонирование этого репозитория к себе на ПК с помощью команды git clone.
Выполните домашнее задание и заполните у себя локально этот файл README.md:
впишите вверху название занятия и ваши фамилию и имя;
в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
для корректного добавления скриншотов воспользуйтесь инструкцией «Как вставить скриншот в шаблон с решением»;
при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в инструкции по MarkDown.
После завершения работы над домашним заданием сделайте коммит (git commit -m "comment") и отправьте его на Github (git push origin).
Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.
Желаем успехов в выполнении домашнего задания.

Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

Решение 1
sql
WITH table_sizes AS (
    SELECT relname AS table_name,
           pg_relation_size(relname::regclass) AS table_size
    FROM pg_class
    WHERE relkind = 'r' -- Только таблицы ('r' означает обычные таблицы)
),
index_sizes AS (
    SELECT indexrelid::regclass::text AS index_name,
           pg_relation_size(indexrelid) AS index_size
    FROM pg_index
    JOIN pg_class ON indexrelid = pg_class.oid
)
SELECT SUM(table_size) AS total_table_size,
       SUM(index_size) AS total_index_size,
       ROUND((SUM(index_size)::numeric / NULLIF(SUM(table_size), 0)) * 100, 2) AS percentage
FROM table_sizes
CROSS JOIN index_sizes;

Задание 2
Выполните explain analyze следующего запроса:

select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
перечислите узкие места;
оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.
Решение 2

1) 
CREATE INDEX idx_payment_date ON payment(payment_date);

SELECT
    CONCAT(c.last_name, ' ', c.first_name) AS full_name,
    SUM(p.amount) AS total_amount
FROM
    payment p
JOIN
    rental r ON p.payment_date = r.rental_date
JOIN
    customer c ON r.customer_id = c.customer_id
JOIN
    inventory i ON r.inventory_id = i.inventory_id
WHERE
    p.payment_date >= '2005-07-30' AND p.payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
GROUP BY
    c.customer_id, c.last_name, c.first_name;


   
2)
EXPLAIN ANALYZE
SELECT
    CONCAT(c.last_name, ' ', c.first_name) AS full_name,
    SUM(p.amount) AS total_amount
FROM
    payment p
JOIN
    rental r ON p.payment_date = r.rental_date
JOIN
    customer c ON r.customer_id = c.customer_id
JOIN
    inventory i ON r.inventory_id = i.inventory_id
WHERE
    p.payment_date >= '2005-07-30' AND p.payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
GROUP BY
    c.customer_id, c.last_name, c.first_name;

