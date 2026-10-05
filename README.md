Лабораторна робота 2. Створення складних SQL запитів
Загальна інформація
Здобувач освіти: [Чаркін Дмитро Олександрович] Група: [ІПЗ-33] Обраний рівень складності: [1/2]

Виконання завдань
Рівень 1
1. З'єднання таблиць
Завдання 1.1: INNER JOIN - список товарів з категоріями та постачальниками

SELECT p.product_name, c.category_name, s.company_name AS supplier_name, p.unit_price FROM products p INNER JOIN categories c ON p.category_id = c.category_id INNER JOIN suppliers s ON p.supplier_id = s.supplier_id ORDER BY c.category_name, p.product_name;

Результат виконання:
<img width="973" height="883" alt="image" src="https://github.com/user-attachments/assets/e8dbfcba-e4f4-482f-a29c-ba1320b057fc" />

Пояснення: Запит здійснює внутрішнє з'єднання трьох таблиць: основної products, довідника категорій categories (за первинним ключем category_id) та довідника контрагентів suppliers (за supplier_id). INNER JOIN гарантує, що в підсумкову вибірку потрапляють виключно ті позиції, для яких одночасно визначені і категорія, і постачальник (перетин множин без NULL у ключових полях).

Завдання 1.2: LEFT JOIN - клієнти з кількістю замовлень

SELECT c.contact_name, c.customer_type, r.region_name, COUNT(o.order_id) AS order_count FROM customers c LEFT JOIN orders o ON c.customer_id = o.customer_id LEFT JOIN regions r ON c.region_id = r.region_id GROUP BY c.customer_id, c.contact_name, c.customer_type, r.region_name ORDER BY order_count DESC;

Результат виконання:
<img width="734" height="896" alt="image" src="https://github.com/user-attachments/assets/08f8629a-2460-4554-a537-3a56b52e355e" />

Пояснення: На відміну від INNER JOIN, оператор LEFT JOIN зберігає всі записи з лівої таблиці (customers), навіть якщо клієнт ще не зробив жодного замовлення. У такому разі поля правої таблиці orders заповнюються значенням NULL.Агрегатна функція COUNT(o.order_id) повертає 0 для таких клієнтів (оскільки COUNT(column) ігнорує NULL), тоді як INNER JOIN просто викинув би цих покупців зі звіту.

Завдання 1.3: Множинне з'єднання - детальна інформація про замовлення

SELECT o.order_id, o.order_date, cu.contact_name AS customer_name, p.product_name, cat.category_name, oi.quantity, oi.unit_price, ROUND(oi.quantity * oi.unit_price * (1 - oi.discount), 2) AS line_total FROM orders o JOIN customers cu ON o.customer_id = cu.customer_id JOIN order_items oi ON o.order_id = oi.order_id JOIN products p ON oi.product_id = p.product_id JOIN categories cat ON p.category_id = cat.category_id ORDER BY o.order_date DESC, o.order_id;

Результат виконання:
<img width="1307" height="901" alt="image" src="https://github.com/user-attachments/assets/b058e6ca-b830-479f-acf3-069753c50187" />

Аналіз складності: Ми просто розгортаємо "чеки" магазину. Починаємо із замовлення 
→
 дивимося, хто його зробив (клієнт) 
→
 дивимося, які рядки в чеку (order_items) 
→
 дізнаємося точну назву товару 
→
 і в яку категорію він входить. Це як зібрати пазл з 5 різних коробок в одну зрозумілу таблицю.

2. Агрегатні функції
Завдання 2.1: Статистика товарів за категоріями

SELECT c.category_name, COUNT(p.product_id) AS product_count, ROUND(AVG(p.unit_price), 2) AS avg_price, MIN(p.unit_price) AS min_price, MAX(p.unit_price) AS max_price FROM categories c LEFT JOIN products p ON c.category_id = p.category_id GROUP BY c.category_id, c.category_name ORDER BY product_count DESC;

Результат виконання:
<img width="663" height="769" alt="image" src="https://github.com/user-attachments/assets/f893e99b-8b19-4e88-83ee-970867e48ede" />

Завдання 2.2: Продажі за регіонами з використанням HAVING

SELECT r.region_name, COUNT(DISTINCT o.order_id) AS delivered_orders_count, ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)), 2) AS total_revenue FROM orders o JOIN order_items oi ON o.order_id = oi.order_id JOIN regions r ON o.ship_region_id = r.region_id WHERE o.order_status = 'delivered' GROUP BY r.region_id, r.region_name HAVING SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) > 50000 ORDER BY total_revenue DESC;

Результат виконання:
<img width="662" height="566" alt="image" src="https://github.com/user-attachments/assets/bfe5a224-1dd7-4d25-a7e0-2e39958bef93" />

Завдання 2.3: Постачальники з кількістю товарів більше 2

SELECT s.company_name, s.city, COUNT(p.product_id) AS products_supplied FROM suppliers s JOIN products p ON s.supplier_id = p.supplier_id GROUP BY s.supplier_id, s.company_name, s.city HAVING COUNT(p.product_id) > 2 ORDER BY products_supplied DESC;

Результат виконання:
<img width="512" height="532" alt="image" src="https://github.com/user-attachments/assets/448ca763-0536-4037-aed8-ec331cbc53ad" />

3. Базові підзапити
Завдання 3.1: Товари з ціною вище середньої по категорії

SELECT p.product_name, p.unit_price, c.category_name FROM products p INNER JOIN categories c ON p.category_id = c.category_id WHERE p.unit_price > ( SELECT AVG(p2.unit_price) FROM products p2 WHERE p2.category_id = p.category_id ) ORDER BY c.category_name, p.unit_price DESC;

Результат виконання:
<img width="768" height="860" alt="image" src="https://github.com/user-attachments/assets/41df5e7c-d8c0-4896-8e0d-c231a3ee9262" />

Завдання 3.2: Клієнти з замовленнями у 2024 році

SELECT customer_id, contact_name, city, customer_type FROM customers WHERE customer_id IN ( SELECT customer_id FROM orders WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31' ) ORDER BY contact_name;

Результат виконання:
<img width="675" height="893" alt="image" src="https://github.com/user-attachments/assets/ef6d8b03-5faa-4f6d-9fcc-21d8b08bfae2" />

Завдання 3.3: Товари з загальною кількістю продажів

SELECT p.product_id, p.product_name, p.unit_price, COALESCE(( SELECT SUM(oi.quantity) FROM order_items oi JOIN orders o ON oi.order_id = o.order_id WHERE oi.product_id = p.product_id AND o.order_status = 'delivered' ), 0) AS total_units_sold FROM products p ORDER BY total_units_sold DESC;

Результат виконання:

<img width="814" height="894" alt="image" src="https://github.com/user-attachments/assets/07e8f480-c025-4b39-ba15-f5a84b4446d4" />

Рівень 2
4. Складні з'єднання
Завдання 4.1: RIGHT JOIN - аналіз категорій та товарів

SELECT c.category_name, COUNT(p.product_id) AS products_count, COALESCE(ROUND(AVG(p.unit_price), 2), 0) AS avg_price FROM products p RIGHT JOIN categories c ON p.category_id = c.category_id GROUP BY c.category_id, c.category_name ORDER BY products_count DESC;

Результат виконання:
<img width="590" height="743" alt="image" src="https://github.com/user-attachments/assets/533f9a74-a5a9-4a71-8585-69eb046ac0a9" />

Завдання 4.2: Self-join - співробітники та керівники

SELECT e1.first_name || ' ' || e1.last_name AS employee, e1.title AS employee_title, COALESCE(e2.first_name || ' ' || e2.last_name, 'Немає (Керівник компанії)') AS manager, COALESCE(e2.title, '—') AS manager_title FROM employees e1 LEFT JOIN employees e2 ON e1.reports_to = e2.employee_id ORDER BY e2.last_name NULLS FIRST, e1.last_name;

Результат виконання:
<img width="792" height="760" alt="image" src="https://github.com/user-attachments/assets/a99dfd51-137c-4b91-b162-fa774d851b88" />

5. Віконні функції
Завдання 5.1: Ранжування товарів за ціною в категоріях

SELECT c.category_name, p.product_name, p.unit_price, ROW_NUMBER() OVER (PARTITION BY c.category_name ORDER BY p.unit_price DESC) AS row_num, RANK() OVER (PARTITION BY c.category_name ORDER BY p.unit_price DESC) AS price_rank, DENSE_RANK() OVER (PARTITION BY c.category_name ORDER BY p.unit_price DESC) AS price_dense_rank FROM products p JOIN categories c ON p.category_id = c.category_id ORDER BY c.category_name, p.unit_price DESC;

Результат виконання:
<img width="1017" height="863" alt="image" src="https://github.com/user-attachments/assets/ec23f6a5-044b-4579-988f-8a394cb95ab3" />

Завдання 5.2: Порівняння замовлень з попередніми датами

SELECT customer_id, order_id, order_date, freight, LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_order_date, LEAD(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS next_order_date, order_date - LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS days_since_last_order FROM orders ORDER BY customer_id, order_date;

Результат виконання:
<img width="959" height="864" alt="image" src="https://github.com/user-attachments/assets/0ceb7354-3ec6-47e3-9b2d-90699806b32a" />

Аналіз продуктивності
Дослідження планів виконання
Найповільніший запит:

EXPLAIN ANALYZE SELECT p.product_name, p.unit_price, c.category_name FROM products p INNER JOIN categories c ON p.category_id = c.category_id WHERE p.unit_price > ( SELECT AVG(p2.unit_price) FROM products p2 WHERE p2.category_id = p.category_id );

План виконання (EXPLAIN ANALYZE):

<img width="599" height="829" alt="image" src="https://github.com/user-attachments/assets/b7b11dbd-9b8b-4fd9-8a8d-04f4af363dbe" />

Створені індекси
Індекс 1:

CREATE INDEX idx_products_cat_price ON products(category_id, unit_price);

Обґрунтування: Прискорює фільтрацію та агрегацію товарів у розрізі категорій, дозволяючи СУБД використовувати швидкий Index Only Scan без звернення до сторінок даних самої таблиці.

Індекс 2:

CREATE INDEX idx_orders_customer_status ON orders(customer_id, order_status);

Обґрунтування: Оптимізує вибірки замовлень конкретного клієнта з фільтрацією за статусом (наприклад, тільки delivered), що часто зустрічається в кабінеті користувача та аналітичних звітах.

 Порівняльний аналіз

 Ефективність різних підходів

Завдання: визначити 5 найдорожчих товарів у кожній категорії.

Для виконання завдання було використано два різні підходи.

Підхід 1 — використання віконної функції `DENSE_RANK`:**

```sql
WITH ranked_products AS (
    SELECT product_name, category_id, unit_price,
           DENSE_RANK() OVER (
               PARTITION BY category_id
               ORDER BY unit_price DESC
           ) AS rnk
    FROM products
)
SELECT product_name, category_id, unit_price
FROM ranked_products
WHERE rnk <= 5;
```

Підхід 2 — використання корельованого підзапиту:**

```sql
SELECT p1.product_name, p1.category_id, p1.unit_price
FROM products p1
WHERE (
    SELECT COUNT(DISTINCT p2.unit_price)
    FROM products p2
    WHERE p2.category_id = p1.category_id
      AND p2.unit_price >= p1.unit_price
) <= 5
ORDER BY p1.category_id, p1.unit_price DESC;
```

Результати порівняння

За результатами вимірювання часу виконання було отримано такі показники:

* **Віконна функція:** приблизно 0,660 мс;
* **Корельований підзапит:** приблизно 0,395 мс.

Отримані результати показують, що на використаному невеликому наборі даних корельований підзапит фактично виконався швидше. Водночас віконні функції є більш зручним та масштабованим рішенням для подібних аналітичних задач, оскільки дозволяють ранжувати записи в межах кожної категорії без виконання окремого підзапиту для кожного товару.

Таким чином, вибір конкретного підходу залежить не лише від виміряного часу виконання на невеликій вибірці, а й від структури запиту, обсягу даних та особливостей плану виконання PostgreSQL.

 Висновки

Самооцінка: 4

Обґрунтування: У повному обсязі виконано завдання Рівня 1 та Рівня 2. Під час роботи було продемонстровано навички використання різних типів з'єднань (`INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `Self-join`), агрегатних та віконних функцій (`RANK`, `LAG`, `LEAD`). Також було використано сучасні можливості PostgreSQL, зокрема `WITH RECURSIVE` та `MATERIALIZED VIEW`.

Окрім цього, було проведено порівняння різних способів виконання SQL-запитів за допомогою `EXPLAIN ANALYZE`. На основі отриманих результатів здійснено аналіз ефективності запитів та обґрунтовано вибір найбільш доцільних методів їх реалізації й оптимізації.
