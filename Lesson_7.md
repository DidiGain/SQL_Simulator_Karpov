## Задача 10.
Выясните, есть ли в таблице courier_actions такие заказы, которые были приняты курьерами, но не были созданы пользователями. 

Посчитайте количество таких заказов.
Колонку с числом заказов назовите orders_count.
Поле в результирующей таблице: orders_count

### Solution
<pre>
<b>WITH</b>
  created_t <b>AS</b> (<b>SELECT</b> order_id <b>FROM</b> user_actions <b>WHERE</b> action = 'create_order')

<b>SELECT</b> <b>COUNT</b>(order_id) <b>AS</b> orders_count
<b>FROM</b> courier_actions
<b>WHERE</b> action = 'accept_order' <b>AND</b> order_id <b>NOT IN</b> (<b>SELECT</b> order_id <b>FROM</b> created_t)
</pre>

## Задача 19.
Для каждого заказа, в котором больше 5 товаров, рассчитайте время, затраченное на его доставку. 

В результат включите id заказа, время принятия заказа курьером, время доставки заказа и время, затраченное на доставку. Новые колонки назовите соответственно time_accepted, time_delivered и delivery_time.

В расчётах учитывайте только неотменённые заказы. Время, затраченное на доставку, выразите в минутах, округлив значения до целого числа. Результат отсортируйте по возрастанию id заказа.

Поля в результирующей таблице: order_id, time_accepted, time_delivered и delivery_time

### Solution
<pre>WITH
    cancelled_orders_t AS (
        SELECT order_id
        FROM user_actions
        WHERE action = 'cancel_order'
    )

SELECT
  order_id,
  MIN(CASE WHEN action = 'accept_order' THEN time END) AS time_accepted,
  MAX(CASE WHEN action = 'deliver_order' THEN time END) AS time_delivered,
  (EXTRACT(EPOCH FROM MAX(time) - MIN(time)) / 60) :: integer AS delivery_time
FROM courier_actions
WHERE 
    order_id IN (SELECT order_id FROM orders WHERE array_length(product_ids, 1) > 5)
    AND order_id NOT IN (SELECT order_id FROM cancelled_orders_t)
GROUP BY
    order_id
ORDER BY order_id</pre>

## Задача 20.
Для каждой даты в таблице user_actions посчитайте количество первых заказов, совершённых пользователями.

Первыми заказами будем считать заказы, которые пользователи сделали в нашем сервисе впервые. В расчётах учитывайте только неотменённые заказы.

В результат включите две колонки: дату и количество первых заказов в эту дату. Колонку с датами назовите date, а колонку с первыми заказами — first_orders.

Результат отсортируйте по возрастанию даты.

Поля в результирующей таблице: date, first_orders

### Solution
<pre>WITH
    cancelled_orders_t AS (
        SELECT order_id
        FROM user_actions
        WHERE action = 'cancel_order'
    ),
    first_orders_t AS (
        SELECT MIN(time)::DATE as date, user_id
        FROM user_actions
        WHERE order_id NOT IN (SELECT order_id FROM cancelled_orders_t)
        GROUP BY
            user_id
    )

SELECT
  date,
  COUNT(user_id) as first_orders
FROM first_orders_t
GROUP BY
    date
ORDER BY date</pre>

## Задача 22.
Используя функцию unnest, определите 10 самых популярных товаров в таблице orders.

Самыми популярными товарами будем считать те, которые встречались в заказах чаще всего. Если товар встречается в одном заказе несколько раз (когда было куплено несколько единиц товара), это тоже учитывается при подсчёте. Учитывайте только неотменённые заказы.

Выведите id товаров и то, сколько раз они встречались в заказах (то есть сколько раз были куплены). Новую колонку с количеством покупок товаров назовите times_purchased.

Результат отсортируйте по возрастанию id товара.

Поля в результирующей таблице: product_id, times_purchased

### Solution
<pre>WITH
    cancelled_orders_t AS (
        SELECT order_id
        FROM user_actions
        WHERE action = 'cancel_order'
    ),
    unnest_t AS (
        SELECT
          unnest(product_ids) as product_id
        FROM orders
        WHERE order_id NOT IN (SELECT order_id FROM cancelled_orders_t)
    ),
    count_t AS (
        SELECT 
          product_id,
          COUNT(product_id) as times_purchased
        FROM unnest_t
        GROUP BY product_id
        ORDER BY
          times_purchased DESC
        LIMIT 10
    )

SELECT product_id, times_purchased
FROM count_t
ORDER BY product_id</pre>

<pre>WITH
    cancelled_orders_t AS (
        SELECT order_id
        FROM user_actions
        WHERE action = 'cancel_order'
    ),
    unnest_t AS (
        SELECT
          unnest(product_ids) as product_id,
          count(*) as times_purchased
        FROM orders
        WHERE order_id NOT IN (SELECT order_id FROM cancelled_orders_t)
        GROUP BY product_id
        ORDER BY
          times_purchased DESC
        LIMIT 10
    )

SELECT product_id, times_purchased
FROM unnest_t
ORDER BY product_id</pre>
