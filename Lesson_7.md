## Задача 10.
Выясните, есть ли в таблице courier_actions такие заказы, которые были приняты курьерами, но не были созданы пользователями. Посчитайте количество таких заказов.
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
