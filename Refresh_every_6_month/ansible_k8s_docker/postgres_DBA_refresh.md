# Полный практический курс по PostgreSQL

## От новичка до эксперта - DevOps Edition

---

## 🟢 JUNIOR DATABASE USER

### Задача 1: Установка и первое подключение

**Цель:** Установить PostgreSQL и научиться подключаться

**Практика:**

bash

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install postgresql postgresql-contrib

# macOS
brew install postgresql@16
brew services start postgresql@16

# Проверка установки
psql --version

# Подключение к БД
sudo -u postgres psql

# Создать своего пользователя
CREATE USER myuser WITH PASSWORD 'mypassword';
CREATE DATABASE mydb OWNER myuser;

# Выход
\q

# Подключение под своим пользователем
psql -U myuser -d mydb -h localhost

# Основные команды psql
\l          # Список баз данных
\du         # Список пользователей
\dt         # Список таблиц
\d table    # Структура таблицы
\c dbname   # Переключиться на другую БД
\?          # Помощь по командам psql
\h          # Помощь по SQL командам
```

**Упражнение:**

1. Установите PostgreSQL
2. Создайте пользователя и базу данных
3. Подключитесь через psql
4. Изучите основные команды psql

---

### Задача 2: Создание таблиц и типы данных

**Цель:** Научиться создавать таблицы с разными типами данных

**Практика:**

sql

```sql
-- Создание простой таблицы
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Типы данных
CREATE TABLE data_types_demo (
    -- Числовые
    int_col INTEGER,
    bigint_col BIGINT,
    decimal_col DECIMAL(10,2),
    real_col REAL,
    
    -- Текстовые
    varchar_col VARCHAR(100),
    text_col TEXT,
    char_col CHAR(10),
    
    -- Дата и время
    date_col DATE,
    time_col TIME,
    timestamp_col TIMESTAMP,
    timestamptz_col TIMESTAMPTZ,
    
    -- Булевы
    bool_col BOOLEAN,
    
    -- JSON
    json_col JSON,
    jsonb_col JSONB,
    
    -- Массивы
    int_array INTEGER[],
    text_array TEXT[],
    
    -- UUID
    uuid_col UUID DEFAULT gen_random_uuid()
);

-- Таблица с ограничениями
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    price DECIMAL(10,2) CHECK (price > 0),
    stock INTEGER DEFAULT 0 CHECK (stock >= 0),
    category VARCHAR(50),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Таблица с внешним ключом
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER CHECK (quantity > 0),
    total DECIMAL(10,2),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Просмотр структуры таблицы
\d users
\d+ products  -- Детальная информация
```

**Упражнение:**

1. Создайте таблицу users с полями id, username, email, age
2. Создайте таблицу posts с внешним ключом на users
3. Добавьте ограничения (NOT NULL, UNIQUE, CHECK)
4. Изучите структуру созданных таблиц

---

### Задача 3: CRUD операции (Create, Read, Update, Delete)

**Цель:** Освоить базовые операции с данными

**Практика:**

sql

```sql
-- CREATE (Вставка данных)
INSERT INTO users (username, email) 
VALUES ('john_doe', 'john@example.com');

-- Вставка нескольких строк
INSERT INTO users (username, email) VALUES
    ('alice', 'alice@example.com'),
    ('bob', 'bob@example.com'),
    ('charlie', 'charlie@example.com');

-- Вставка с возвратом данных
INSERT INTO users (username, email)
VALUES ('dave', 'dave@example.com')
RETURNING *;

-- READ (Чтение данных)
-- Выбрать все
SELECT * FROM users;

-- Выбрать конкретные колонки
SELECT id, username FROM users;

-- С условием WHERE
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE username LIKE 'a%';

-- Сортировка
SELECT * FROM users ORDER BY created_at DESC;
SELECT * FROM users ORDER BY username ASC;

-- Ограничение количества
SELECT * FROM users LIMIT 5;
SELECT * FROM users LIMIT 5 OFFSET 10;  -- Пропустить первые 10

-- Подсчет
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM users WHERE created_at > '2024-01-01';

-- UPDATE (Обновление данных)
UPDATE users 
SET email = 'newemail@example.com' 
WHERE username = 'john_doe';

-- Обновление нескольких полей
UPDATE products 
SET price = price * 1.1, updated_at = NOW() 
WHERE category = 'electronics';

-- Обновление с RETURNING
UPDATE users 
SET email = 'updated@example.com' 
WHERE id = 1 
RETURNING *;

-- DELETE (Удаление данных)
DELETE FROM users WHERE id = 5;

-- Удаление с условием
DELETE FROM users WHERE created_at < '2023-01-01';

-- Удаление всех записей (ОСТОРОЖНО!)
DELETE FROM users;

-- Удаление с RETURNING
DELETE FROM users 
WHERE username = 'old_user' 
RETURNING *;

-- Безопасное удаление (с подтверждением)
BEGIN;
DELETE FROM users WHERE id = 10;
SELECT * FROM users WHERE id = 10;  -- Проверить
ROLLBACK;  -- Отменить, если что-то не так
-- или COMMIT; -- Подтвердить
```

**Упражнение:**

1. Вставьте 10 пользователей в таблицу users
2. Выберите пользователей с определенным условием
3. Обновите email одного пользователя
4. Удалите пользователя с конкретным id
5. Попрактикуйтесь с LIMIT и OFFSET

---

### Задача 4: Базовые JOIN запросы

**Цель:** Научиться объединять данные из нескольких таблиц

**Практика:**

sql

```sql
-- Подготовка данных
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department_id INTEGER REFERENCES departments(id),
    salary DECIMAL(10,2)
);

INSERT INTO departments (name) VALUES
    ('Engineering'),
    ('Sales'),
    ('HR'),
    ('Marketing');

INSERT INTO employees (name, department_id, salary) VALUES
    ('Alice', 1, 80000),
    ('Bob', 1, 75000),
    ('Charlie', 2, 60000),
    ('Dave', 2, 65000),
    ('Eve', 3, 55000),
    ('Frank', NULL, 50000);  -- Без департамента

-- INNER JOIN (только совпадающие записи)
SELECT 
    employees.name AS employee_name,
    departments.name AS department_name,
    employees.salary
FROM employees
INNER JOIN departments ON employees.department_id = departments.id;

-- LEFT JOIN (все записи из левой таблицы)
SELECT 
    e.name AS employee_name,
    d.name AS department_name,
    e.salary
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;

-- RIGHT JOIN (все записи из правой таблицы)
SELECT 
    e.name AS employee_name,
    d.name AS department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;

-- FULL OUTER JOIN (все записи из обеих таблиц)
SELECT 
    e.name AS employee_name,
    d.name AS department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;

-- Множественные JOIN
CREATE TABLE projects (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department_id INTEGER REFERENCES departments(id)
);

INSERT INTO projects (name, department_id) VALUES
    ('Project A', 1),
    ('Project B', 1),
    ('Project C', 2);

SELECT 
    e.name AS employee,
    d.name AS department,
    p.name AS project
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
INNER JOIN projects p ON d.id = p.department_id;

-- JOIN с агрегацией
SELECT 
    d.name AS department,
    COUNT(e.id) AS employee_count,
    AVG(e.salary) AS avg_salary,
    MAX(e.salary) AS max_salary
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.name
ORDER BY employee_count DESC;
```

**Упражнение:**

1. Создайте таблицы customers, orders, products
2. Заполните их тестовыми данными
3. Напишите запрос с INNER JOIN для получения заказов с именами клиентов
4. Используйте LEFT JOIN для показа всех клиентов, даже без заказов
5. Напишите запрос с множественным JOIN

---

### Задача 5: Агрегатные функции и GROUP BY

**Цель:** Научиться группировать и агрегировать данные

**Практика:**

sql

```sql
-- Подготовка данных
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    product VARCHAR(100),
    category VARCHAR(50),
    amount DECIMAL(10,2),
    quantity INTEGER,
    sale_date DATE
);

INSERT INTO sales (product, category, amount, quantity, sale_date) VALUES
    ('Laptop', 'Electronics', 1200, 5, '2024-01-15'),
    ('Mouse', 'Electronics', 25, 50, '2024-01-15'),
    ('Keyboard', 'Electronics', 75, 30, '2024-01-16'),
    ('Chair', 'Furniture', 200, 10, '2024-01-16'),
    ('Desk', 'Furniture', 400, 5, '2024-01-17'),
    ('Laptop', 'Electronics', 1200, 3, '2024-01-18'),
    ('Monitor', 'Electronics', 300, 8, '2024-01-18');

-- Агрегатные функции
SELECT COUNT(*) FROM sales;                    -- Количество записей
SELECT SUM(amount) FROM sales;                 -- Сумма
SELECT AVG(amount) FROM sales;                 -- Среднее
SELECT MIN(amount) FROM sales;                 -- Минимум
SELECT MAX(amount) FROM sales;                 -- Максимум

-- GROUP BY - группировка по категории
SELECT 
    category,
    COUNT(*) AS total_sales,
    SUM(amount) AS total_revenue,
    AVG(amount) AS avg_sale_price
FROM sales
GROUP BY category;

-- GROUP BY с сортировкой
SELECT 
    product,
    SUM(quantity) AS total_sold,
    SUM(amount) AS total_revenue
FROM sales
GROUP BY product
ORDER BY total_revenue DESC;

-- HAVING - фильтрация после группировки
SELECT 
    category,
    COUNT(*) AS sale_count,
    SUM(amount) AS total_revenue
FROM sales
GROUP BY category
HAVING SUM(amount) > 1000;

-- GROUP BY с несколькими колонками
SELECT 
    category,
    product,
    COUNT(*) AS times_sold,
    SUM(amount) AS revenue
FROM sales
GROUP BY category, product
ORDER BY category, revenue DESC;

-- GROUP BY с датами
SELECT 
    DATE_TRUNC('day', sale_date) AS sale_day,
    COUNT(*) AS transactions,
    SUM(amount) AS daily_revenue
FROM sales
GROUP BY DATE_TRUNC('day', sale_date)
ORDER BY sale_day;

-- Комплексный запрос
SELECT 
    category,
    COUNT(DISTINCT product) AS unique_products,
    COUNT(*) AS total_transactions,
    SUM(quantity) AS total_items_sold,
    SUM(amount) AS total_revenue,
    AVG(amount) AS avg_transaction,
    MIN(amount) AS min_sale,
    MAX(amount) AS max_sale
FROM sales
GROUP BY category
HAVING COUNT(*) > 2
ORDER BY total_revenue DESC;

-- DISTINCT - уникальные значения
SELECT DISTINCT category FROM sales;
SELECT DISTINCT product, category FROM sales;

-- COUNT DISTINCT
SELECT COUNT(DISTINCT category) AS unique_categories FROM sales;
```

**Упражнение:**

1. Создайте таблицу с данными о продажах
2. Найдите общую сумму продаж по категориям
3. Найдите среднюю цену продукта в каждой категории
4. Выберите категории с продажами больше определенной суммы (HAVING)
5. Посчитайте продажи по дням/месяцам

---

### Задача 6: WHERE условия и операторы

**Цель:** Освоить фильтрацию данных

**Практика:**

sql

```sql
-- Операторы сравнения
SELECT * FROM products WHERE price = 100;
SELECT * FROM products WHERE price != 100;
SELECT * FROM products WHERE price > 100;
SELECT * FROM products WHERE price >= 100;
SELECT * FROM products WHERE price < 100;
SELECT * FROM products WHERE price <= 100;

-- BETWEEN - диапазон значений
SELECT * FROM products WHERE price BETWEEN 50 AND 150;
SELECT * FROM sales WHERE sale_date BETWEEN '2024-01-01' AND '2024-01-31';

-- IN - список значений
SELECT * FROM products WHERE category IN ('Electronics', 'Books', 'Toys');
SELECT * FROM users WHERE id IN (1, 5, 10, 15);

-- NOT IN
SELECT * FROM products WHERE category NOT IN ('Electronics');

-- LIKE - поиск по шаблону
SELECT * FROM users WHERE username LIKE 'john%';      -- Начинается с john
SELECT * FROM users WHERE username LIKE '%smith';     -- Заканчивается на smith
SELECT * FROM users WHERE username LIKE '%admin%';    -- Содержит admin
SELECT * FROM users WHERE email LIKE '%@gmail.com';

-- ILIKE - регистронезависимый поиск
SELECT * FROM users WHERE username ILIKE 'JOHN%';

-- NOT LIKE
SELECT * FROM users WHERE email NOT LIKE '%@gmail.com';

-- IS NULL / IS NOT NULL
SELECT * FROM employees WHERE department_id IS NULL;
SELECT * FROM employees WHERE department_id IS NOT NULL;

-- Логические операторы
-- AND
SELECT * FROM products 
WHERE category = 'Electronics' AND price > 100;

-- OR
SELECT * FROM products 
WHERE category = 'Electronics' OR category = 'Books';

-- NOT
SELECT * FROM products 
WHERE NOT category = 'Electronics';

-- Комбинирование условий
SELECT * FROM products 
WHERE (category = 'Electronics' OR category = 'Books')
  AND price BETWEEN 20 AND 100
  AND stock > 0;

-- ANY / ALL с подзапросами
SELECT * FROM products 
WHERE price > ANY (SELECT price FROM products WHERE category = 'Books');

SELECT * FROM products 
WHERE price > ALL (SELECT price FROM products WHERE category = 'Books');

-- EXISTS
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.user_id = u.id
);

-- CASE - условная логика
SELECT 
    name,
    price,
    CASE 
        WHEN price < 50 THEN 'Cheap'
        WHEN price BETWEEN 50 AND 200 THEN 'Medium'
        WHEN price > 200 THEN 'Expensive'
        ELSE 'Unknown'
    END AS price_category
FROM products;

-- COALESCE - первое не NULL значение
SELECT 
    name,
    COALESCE(email, phone, 'No contact') AS contact
FROM users;

-- NULLIF - вернуть NULL если значения равны
SELECT 
    name,
    NULLIF(email, '') AS email  -- NULL если email пустой
FROM users;
```

**Упражнение:**

1. Найдите продукты в ценовом диапазоне
2. Выберите пользователей с email от конкретного домена
3. Найдите записи с NULL значениями
4. Используйте комбинацию AND/OR для сложного условия
5. Примените CASE для категоризации данных

---

### Задача 7: Индексы (основы)

**Цель:** Научиться создавать базовые индексы для ускорения запросов

**Практика:**

sql

```sql
-- Создание тестовой таблицы
CREATE TABLE users_large (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100),
    email VARCHAR(100),
    age INTEGER,
    city VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Заполнение тестовыми данными
INSERT INTO users_large (username, email, age, city)
SELECT 
    'user_' || i,
    'user' || i || '@example.com',
    20 + (i % 50),
    CASE (i % 5)
        WHEN 0 THEN 'New York'
        WHEN 1 THEN 'London'
        WHEN 2 THEN 'Tokyo'
        WHEN 3 THEN 'Paris'
        ELSE 'Berlin'
    END
FROM generate_series(1, 100000) AS i;

-- Запрос БЕЗ индекса (медленный)
EXPLAIN ANALYZE
SELECT * FROM users_large WHERE email = 'user50000@example.com';

-- Создание простого индекса
CREATE INDEX idx_users_email ON users_large(email);

-- Тот же запрос С индексом (быстрый)
EXPLAIN ANALYZE
SELECT * FROM users_large WHERE email = 'user50000@example.com';

-- Индекс на несколько колонок (составной индекс)
CREATE INDEX idx_users_city_age ON users_large(city, age);

-- Использование составного индекса
EXPLAIN ANALYZE
SELECT * FROM users_large WHERE city = 'Tokyo' AND age = 25;

-- UNIQUE индекс
CREATE UNIQUE INDEX idx_users_username ON users_large(username);

-- Частичный индекс (только для части данных)
CREATE INDEX idx_users_active_email 
ON users_large(email) 
WHERE age > 30;

-- Индекс для LIKE запросов (с текстовыми операциями)
CREATE INDEX idx_users_username_pattern 
ON users_large(username text_pattern_ops);

-- Просмотр индексов
\di                                    -- Все индексы
\d users_large                         -- Индексы конкретной таблицы
SELECT * FROM pg_indexes 
WHERE tablename = 'users_large';

-- Размер индексов
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexname::regclass)) AS index_size
FROM pg_indexes
WHERE tablename = 'users_large';

-- Удаление индекса
DROP INDEX idx_users_username_pattern;

-- Пересоздание индекса (если поврежден или для оптимизации)
REINDEX INDEX idx_users_email;
REINDEX TABLE users_large;

-- Когда НЕ нужно создавать индексы:
-- 1. На маленьких таблицах (< 1000 строк)
-- 2. На колонках с частыми UPDATE/INSERT
-- 3. На колонках с низкой селективностью (мало уникальных значений)

-- Проверка использования индекса
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM users_large 
WHERE city = 'Tokyo' AND age BETWEEN 25 AND 35;
```

**Упражнение:**

1. Создайте таблицу с большим количеством записей
2. Выполните запрос и посмотрите EXPLAIN
3. Создайте индекс на часто используемую колонку
4. Сравните производительность до и после
5. Создайте составной индекс для сложного запроса

---

## 🟡 MIDDLE DATABASE USER

### Задача 8: Транзакции и изоляция

**Цель:** Понять ACID принципы и уровни изоляции

**Практика:**

sql

```sql
-- Базовая транзакция
BEGIN;
    INSERT INTO accounts (name, balance) VALUES ('Alice', 1000);
    INSERT INTO accounts (name, balance) VALUES ('Bob', 500);
COMMIT;

-- Транзакция с откатом
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
    UPDATE accounts SET balance = balance + 100 WHERE name = 'Bob';
    -- Что-то пошло не так
ROLLBACK;

-- Savepoint - точки сохранения внутри транзакции
BEGIN;
    INSERT INTO users (username) VALUES ('user1');
    SAVEPOINT sp1;
    
    INSERT INTO users (username) VALUES ('user2');
    SAVEPOINT sp2;
    
    INSERT INTO users (username) VALUES ('user3');
    
    -- Откатить до sp2
    ROLLBACK TO SAVEPOINT sp2;
    
    -- user3 отменен, user1 и user2 остались
COMMIT;

-- Перевод денег (классический пример транзакции)
CREATE TABLE accounts (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    balance DECIMAL(10,2) CHECK (balance >= 0)
);

INSERT INTO accounts (name, balance) VALUES 
    ('Alice', 1000),
    ('Bob', 500);

-- Безопасный перевод
BEGIN;
    -- Списание с отправителя
    UPDATE accounts 
    SET balance = balance - 100 
    WHERE name = 'Alice';
    
    -- Проверка (если баланс < 0, произойдет ошибка из-за CHECK)
    
    -- Зачисление получателю
    UPDATE accounts 
    SET balance = balance + 100 
    WHERE name = 'Bob';
COMMIT;

-- Уровни изоляции транзакций
-- 1. READ UNCOMMITTED (не поддерживается в PostgreSQL)
-- 2. READ COMMITTED (по умолчанию)
-- 3. REPEATABLE READ
-- 4. SERIALIZABLE

-- READ COMMITTED (по умолчанию)
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
    SELECT balance FROM accounts WHERE name = 'Alice';
    -- Другая транзакция может изменить данные
    -- Повторное чтение покажет новое значение
    SELECT balance FROM accounts WHERE name = 'Alice';
COMMIT;

-- REPEATABLE READ
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
    SELECT balance FROM accounts WHERE name = 'Alice';
    -- Даже если другая транзакция изменит данные
    -- В этой транзакции значение останется прежним
    SELECT balance FROM accounts WHERE name = 'Alice';
COMMIT;

-- SERIALIZABLE (самый строгий)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    SELECT SUM(balance) FROM accounts;
    UPDATE accounts SET balance = balance * 1.05;
COMMIT;

-- Демонстрация Dirty Read проблемы (в других СУБД)
-- Сессия 1:
BEGIN;
    UPDATE accounts SET balance = 5000 WHERE name = 'Alice';
    -- НЕ делаем COMMIT

-- Сессия 2:
SELECT balance FROM accounts WHERE name = 'Alice';
-- В READ UNCOMMITTED увидели бы 5000 (dirty read)
-- В PostgreSQL увидим старое значение

-- Сессия 1:
ROLLBACK;

-- Non-repeatable Read
-- Сессия 1:
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
    SELECT balance FROM accounts WHERE name = 'Alice';  -- 1000
    -- Ждем...

-- Сессия 2:
UPDATE accounts SET balance = 1500 WHERE name = 'Alice';

-- Сессия 1 (продолжение):
    SELECT balance FROM accounts WHERE name = 'Alice';  -- 1500 (изменилось!)
COMMIT;

-- Phantom Read
-- Сессия 1:
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
    SELECT COUNT(*) FROM accounts WHERE balance > 500;  -- 1
    -- Ждем...

-- Сессия 2:
INSERT INTO accounts (name, balance) VALUES ('Charlie', 1000);

-- Сессия 1 (продолжение):
    SELECT COUNT(*) FROM accounts WHERE balance > 500;  -- 2 (phantom!)
COMMIT;

-- Блокировки
-- FOR UPDATE - эксклюзивная блокировка
BEGIN;
    SELECT * FROM accounts WHERE name = 'Alice' FOR UPDATE;
    -- Другие транзакции не могут изменить эту строку
    UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
COMMIT;

-- FOR SHARE - разделяемая блокировка
BEGIN;
    SELECT * FROM accounts WHERE name = 'Alice' FOR SHARE;
    -- Другие могут читать, но не изменять
COMMIT;

-- Мониторинг блокировок
SELECT 
    pid,
    usename,
    pg_blocking_pids(pid) as blocked_by,
    query as blocked_query
FROM pg_stat_activity
WHERE cardinality(pg_blocking_pids(pid)) > 0;
```

**Упражнение:**

1. Создайте таблицу accounts с балансами
2. Реализуйте перевод денег в транзакции
3. Попробуйте откатить транзакцию с ROLLBACK
4. Используйте SAVEPOINT для частичного отката
5. Протестируйте разные уровни изоляции в двух сессиях

---

### Задача 9: Подзапросы и CTE

**Цель:** Освоить вложенные запросы и Common Table Expressions

**Практика:**

sql

```sql
-- Подготовка данных
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2),
    manager_id INTEGER REFERENCES employees(id)
);

INSERT INTO employees (name, department, salary, manager_id) VALUES
    ('Alice', 'Engineering', 90000, NULL),
    ('Bob', 'Engineering', 80000, 1),
    ('Charlie', 'Engineering', 75000, 1),
    ('Dave', 'Sales', 70000, NULL),
    ('Eve', 'Sales', 65000, 4),
    ('Frank', 'HR', 60000, NULL);

-- Простой подзапрос в WHERE
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Подзапрос с IN
SELECT name, department
FROM employees
WHERE department IN (
    SELECT department 
    FROM employees 
    GROUP BY department 
    HAVING AVG(salary) > 70000
);

-- Подзапрос в SELECT (скалярный подзапрос)
SELECT 
    name,
    salary,
    (SELECT AVG(salary) FROM employees) AS avg_salary,
    salary - (SELECT AVG(salary) FROM employees) AS diff_from_avg
FROM employees;

-- Подзапрос в FROM (производная таблица)
SELECT 
    dept_stats.department,
    dept_stats.avg_salary,
    dept_stats.employee_count
FROM (
    SELECT 
        department,
        AVG(salary) AS avg_salary,
        COUNT(*) AS employee_count
    FROM employees
    GROUP BY department
) AS dept_stats
WHERE dept_stats.avg_salary > 70000;

-- Коррелированный подзапрос
SELECT 
    e1.name,
    e1.department,
    e1.salary
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e1.department
);

-- EXISTS
SELECT name, department
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM employees m
    WHERE m.manager_id = e.id
);

-- NOT EXISTS
SELECT name
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM employees m
    WHERE m.manager_id = e.id
);

-- CTE (Common Table Expression) - WITH clause
WITH dept_avg AS (
    SELECT 
        department,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT 
    e.name,
    e.department,
    e.salary,
    da.avg_salary,
    e.salary - da.avg_salary AS diff
FROM employees e
JOIN dept_avg da ON e.department = da.department
WHERE e.salary > da.avg_salary;

-- Множественные CTE
WITH 
high_earners AS (
    SELECT * FROM employees WHERE salary > 75000
),
departments_stats AS (
    SELECT 
        department,
        COUNT(*) AS total_employees,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT 
    he.name,
    he.department,
    he.salary,
    ds.avg_salary
FROM high_earners he
JOIN departments_stats ds ON he.department = ds.department;

-- Рекурсивный CTE (иерархия - дерево сотрудников)
WITH RECURSIVE employee_hierarchy AS (
    -- Базовый случай: руководители верхнего уровня
    SELECT 
        id,
        name,
        manager_id,
        salary,
        1 AS level,
        name AS path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Рекурсивный случай: подчиненные
    SELECT 
        e.id,
        e.name,
        e.manager_id,
        e.salary,
        eh.level + 1,
        eh.path || ' -> ' || e.name
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT 
    REPEAT('  ', level - 1) || name AS employee_tree,
    level,
    salary,
    path
FROM employee_hierarchy
ORDER
```

Continue

15 нояб.

sql

```sql
ORDER BY path;

-- Рекурсивный CTE для чисел Fibonacci
WITH RECURSIVE fibonacci(n, fib_n, fib_n_plus_1) AS (
    -- Базовый случай
    SELECT 1, 0::BIGINT, 1::BIGINT
    
    UNION ALL
    
    -- Рекурсивный случай
    SELECT 
        n + 1,
        fib_n_plus_1,
        fib_n + fib_n_plus_1
    FROM fibonacci
    WHERE n < 20
)
SELECT n, fib_n FROM fibonacci;

-- Рекурсивный CTE для генерации дат
WITH RECURSIVE date_series AS (
    SELECT DATE '2024-01-01' AS date
    
    UNION ALL
    
    SELECT date + INTERVAL '1 day'
    FROM date_series
    WHERE date < DATE '2024-01-31'
)
SELECT 
    date,
    EXTRACT(DOW FROM date) AS day_of_week,
    TO_CHAR(date, 'Day') AS day_name
FROM date_series;

-- LATERAL JOIN с подзапросом
SELECT 
    d.department,
    top_earner.name,
    top_earner.salary
FROM (SELECT DISTINCT department FROM employees) d
CROSS JOIN LATERAL (
    SELECT name, salary
    FROM employees e
    WHERE e.department = d.department
    ORDER BY salary DESC
    LIMIT 1
) AS top_earner;

-- Подзапросы с ANY/ALL
-- Найти сотрудников, которые зарабатывают больше хотя бы одного в Sales
SELECT name, salary
FROM employees
WHERE salary > ANY (
    SELECT salary FROM employees WHERE department = 'Sales'
);

-- Найти сотрудников, которые зарабатывают больше всех в Sales
SELECT name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary FROM employees WHERE department = 'Sales'
);

-- Материализованный CTE (MATERIALIZED)
WITH dept_stats AS MATERIALIZED (
    SELECT 
        department,
        COUNT(*) AS emp_count,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT * FROM dept_stats
WHERE avg_salary > 70000;

-- Комплексный пример с несколькими CTE и подзапросами
WITH RECURSIVE 
-- CTE 1: Иерархия сотрудников
org_chart AS (
    SELECT 
        id, name, manager_id, salary, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    SELECT 
        e.id, e.name, e.manager_id, e.salary, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
),
-- CTE 2: Статистика по уровням
level_stats AS (
    SELECT 
        level,
        COUNT(*) AS employees_count,
        AVG(salary) AS avg_salary,
        MIN(salary) AS min_salary,
        MAX(salary) AS max_salary
    FROM org_chart
    GROUP BY level
)
-- Основной запрос
SELECT 
    oc.name,
    oc.level,
    oc.salary,
    ls.avg_salary AS level_avg,
    CASE 
        WHEN oc.salary > ls.avg_salary THEN 'Above average'
        WHEN oc.salary < ls.avg_salary THEN 'Below average'
        ELSE 'Average'
    END AS performance
FROM org_chart oc
JOIN level_stats ls ON oc.level = ls.level
ORDER BY oc.level, oc.salary DESC;
```

**Упражнение:**

1. Найдите сотрудников с зарплатой выше средней используя подзапрос
2. Создайте CTE для расчета статистики по отделам
3. Напишите рекурсивный CTE для иерархии (например, категории товаров)
4. Используйте множественные CTE в одном запросе
5. Сравните производительность подзапросов и CTE

---

### Задача 10: Window Functions (Оконные функции)

**Цель:** Освоить мощные аналитические функции

**Практика:**

sql

```sql
-- Подготовка данных
CREATE TABLE sales_data (
    id SERIAL PRIMARY KEY,
    product VARCHAR(100),
    category VARCHAR(50),
    amount DECIMAL(10,2),
    sale_date DATE,
    region VARCHAR(50)
);

INSERT INTO sales_data (product, category, amount, sale_date, region) VALUES
    ('Laptop', 'Electronics', 1200, '2024-01-15', 'North'),
    ('Mouse', 'Electronics', 25, '2024-01-15', 'North'),
    ('Keyboard', 'Electronics', 75, '2024-01-16', 'South'),
    ('Chair', 'Furniture', 200, '2024-01-16', 'North'),
    ('Desk', 'Furniture', 400, '2024-01-17', 'South'),
    ('Monitor', 'Electronics', 300, '2024-01-18', 'North'),
    ('Lamp', 'Furniture', 50, '2024-01-18', 'South'),
    ('Phone', 'Electronics', 800, '2024-01-19', 'North'),
    ('Tablet', 'Electronics', 600, '2024-01-19', 'South'),
    ('Sofa', 'Furniture', 900, '2024-01-20', 'North');

-- ROW_NUMBER - уникальный номер строки
SELECT 
    product,
    amount,
    ROW_NUMBER() OVER (ORDER BY amount DESC) AS row_num
FROM sales_data;

-- RANK и DENSE_RANK - ранжирование с учетом дубликатов
SELECT 
    product,
    amount,
    RANK() OVER (ORDER BY amount DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY amount DESC) AS dense_rank
FROM sales_data;

-- PARTITION BY - разделение окна на группы
SELECT 
    category,
    product,
    amount,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY amount DESC) AS rank_in_category
FROM sales_data;

-- SUM с окном - накопительная сумма
SELECT 
    sale_date,
    product,
    amount,
    SUM(amount) OVER (ORDER BY sale_date) AS running_total
FROM sales_data;

-- Накопительная сумма по категориям
SELECT 
    category,
    sale_date,
    product,
    amount,
    SUM(amount) OVER (
        PARTITION BY category 
        ORDER BY sale_date
    ) AS category_running_total
FROM sales_data
ORDER BY category, sale_date;

-- AVG с окном - скользящее среднее
SELECT 
    sale_date,
    product,
    amount,
    AVG(amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3
FROM sales_data;

-- LAG и LEAD - доступ к предыдущей и следующей строкам
SELECT 
    sale_date,
    product,
    amount,
    LAG(amount, 1) OVER (ORDER BY sale_date) AS prev_amount,
    LEAD(amount, 1) OVER (ORDER BY sale_date) AS next_amount,
    amount - LAG(amount, 1) OVER (ORDER BY sale_date) AS diff_from_prev
FROM sales_data;

-- FIRST_VALUE и LAST_VALUE
SELECT 
    category,
    product,
    amount,
    FIRST_VALUE(product) OVER (
        PARTITION BY category 
        ORDER BY amount DESC
    ) AS top_product,
    LAST_VALUE(product) OVER (
        PARTITION BY category 
        ORDER BY amount DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS bottom_product
FROM sales_data;

-- NTH_VALUE - N-ое значение в окне
SELECT 
    category,
    product,
    amount,
    NTH_VALUE(product, 2) OVER (
        PARTITION BY category 
        ORDER BY amount DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS second_best
FROM sales_data;

-- NTILE - разделение на N групп (квартили, децили)
SELECT 
    product,
    amount,
    NTILE(4) OVER (ORDER BY amount) AS quartile
FROM sales_data;

-- Процентили с PERCENT_RANK и CUME_DIST
SELECT 
    product,
    amount,
    PERCENT_RANK() OVER (ORDER BY amount) AS percent_rank,
    CUME_DIST() OVER (ORDER BY amount) AS cumulative_dist,
    ROUND(PERCENT_RANK() OVER (ORDER BY amount) * 100, 2) AS percentile
FROM sales_data;

-- Сравнение с агрегатными функциями
SELECT 
    category,
    product,
    amount,
    SUM(amount) OVER (PARTITION BY category) AS category_total,
    AVG(amount) OVER (PARTITION BY category) AS category_avg,
    amount / SUM(amount) OVER (PARTITION BY category) * 100 AS pct_of_category
FROM sales_data;

-- Frame clause (ROWS vs RANGE)
-- ROWS - физические строки
SELECT 
    sale_date,
    amount,
    SUM(amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) AS sum_3_rows
FROM sales_data;

-- RANGE - логический диапазон значений
SELECT 
    sale_date,
    amount,
    SUM(amount) OVER (
        ORDER BY sale_date
        RANGE BETWEEN INTERVAL '1 day' PRECEDING AND INTERVAL '1 day' FOLLOWING
    ) AS sum_3_days
FROM sales_data;

-- Комплексный анализ продаж
WITH sales_analytics AS (
    SELECT 
        category,
        region,
        sale_date,
        product,
        amount,
        -- Ранг по продажам в категории
        RANK() OVER (PARTITION BY category ORDER BY amount DESC) AS category_rank,
        -- Доля от общих продаж категории
        ROUND(
            amount / SUM(amount) OVER (PARTITION BY category) * 100, 
            2
        ) AS pct_of_category,
        -- Накопительная сумма по дате
        SUM(amount) OVER (
            PARTITION BY category 
            ORDER BY sale_date
        ) AS running_total,
        -- Сравнение с предыдущей продажей
        amount - LAG(amount) OVER (
            PARTITION BY category 
            ORDER BY sale_date
        ) AS diff_from_prev,
        -- Скользящее среднее за 3 продажи
        ROUND(
            AVG(amount) OVER (
                PARTITION BY category 
                ORDER BY sale_date
                ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
            ),
            2
        ) AS moving_avg
    FROM sales_data
)
SELECT * FROM sales_analytics
ORDER BY category, sale_date;

-- Топ N в каждой группе
SELECT *
FROM (
    SELECT 
        category,
        product,
        amount,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY amount DESC) AS rn
    FROM sales_data
) ranked
WHERE rn <= 3;

-- Поиск выбросов (outliers)
WITH stats AS (
    SELECT 
        category,
        AVG(amount) AS avg_amount,
        STDDEV(amount) AS stddev_amount
    FROM sales_data
    GROUP BY category
)
SELECT 
    sd.category,
    sd.product,
    sd.amount,
    s.avg_amount,
    s.stddev_amount,
    CASE 
        WHEN sd.amount > s.avg_amount + 2 * s.stddev_amount THEN 'High outlier'
        WHEN sd.amount < s.avg_amount - 2 * s.stddev_amount THEN 'Low outlier'
        ELSE 'Normal'
    END AS outlier_status
FROM sales_data sd
JOIN stats s ON sd.category = s.category;
```

**Упражнение:**

1. Создайте ранжирование продаж по категориям
2. Рассчитайте накопительную сумму продаж
3. Найдите разницу с предыдущей продажей используя LAG
4. Создайте скользящее среднее за последние 3 записи
5. Выберите топ-3 товара в каждой категории

---

### Задача 11: JSON и JSONB

**Цель:** Работа с JSON данными в PostgreSQL

**Практика:**

sql

```sql
-- Создание таблицы с JSON
CREATE TABLE products_json (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    data JSON,
    metadata JSONB  -- JSONB быстрее для запросов
);

-- Вставка JSON данных
INSERT INTO products_json (name, data, metadata) VALUES
    ('Laptop', 
     '{"brand": "Dell", "price": 1200, "specs": {"cpu": "i7", "ram": "16GB"}}',
     '{"brand": "Dell", "price": 1200, "specs": {"cpu": "i7", "ram": "16GB"}}'
    ),
    ('Phone',
     '{"brand": "Apple", "price": 999, "specs": {"storage": "128GB", "color": "black"}}',
     '{"brand": "Apple", "price": 999, "specs": {"storage": "128GB", "color": "black"}}'
    );

-- Чтение JSON полей
-- -> возвращает JSON
-- ->> возвращает текст
SELECT 
    name,
    data->'brand' AS brand_json,
    data->>'brand' AS brand_text,
    data->'price' AS price
FROM products_json;

-- Доступ к вложенным полям
SELECT 
    name,
    data->'specs'->>'cpu' AS cpu,
    data->'specs'->>'ram' AS ram
FROM products_json;

-- Поиск по JSON полям (JSONB поддерживает индексы!)
SELECT * FROM products_json
WHERE metadata->>'brand' = 'Dell';

SELECT * FROM products_json
WHERE (metadata->>'price')::NUMERIC > 1000;

-- Операторы для JSONB
-- @> содержит
SELECT * FROM products_json
WHERE metadata @> '{"brand": "Apple"}';

-- ? ключ существует
SELECT * FROM products_json
WHERE metadata ? 'price';

-- ?| хотя бы один ключ существует
SELECT * FROM products_json
WHERE metadata ?| ARRAY['brand', 'model'];

-- ?& все ключи существуют
SELECT * FROM products_json
WHERE metadata ?& ARRAY['brand', 'price'];

-- Изменение JSON данных
-- jsonb_set - обновление значения
UPDATE products_json
SET metadata = jsonb_set(
    metadata,
    '{price}',
    '1100'
)
WHERE name = 'Laptop';

-- Добавление нового поля
UPDATE products_json
SET metadata = jsonb_set(
    metadata,
    '{warranty}',
    '"2 years"',
    true  -- create_if_missing
)
WHERE name = 'Laptop';

-- Удаление поля
UPDATE products_json
SET metadata = metadata - 'warranty'
WHERE name = 'Laptop';

-- jsonb_insert - вставка нового значения
UPDATE products_json
SET metadata = jsonb_insert(
    metadata,
    '{specs, screen}',
    '"15 inch"'
)
WHERE name = 'Laptop';

-- Агрегация JSON данных
-- json_agg - собрать строки в JSON массив
SELECT 
    data->>'brand' AS brand,
    json_agg(json_build_object(
        'name', name,
        'price', data->'price'
    )) AS products
FROM products_json
GROUP BY data->>'brand';

-- jsonb_object_agg - собрать в JSON объект
SELECT jsonb_object_agg(name, data->'price')
FROM products_json;

-- Разворачивание JSON
-- jsonb_each - развернуть в key-value пары
SELECT * FROM jsonb_each(
    '{"a": 1, "b": 2, "c": 3}'::JSONB
);

-- jsonb_each_text - то же, но значения как текст
SELECT * FROM jsonb_each_text(
    '{"name": "John", "age": "30"}'::JSONB
);

-- jsonb_array_elements - развернуть массив
CREATE TABLE orders_json (
    id SERIAL PRIMARY KEY,
    customer VARCHAR(100),
    items JSONB
);

INSERT INTO orders_json (customer, items) VALUES
    ('John', '[{"product": "Laptop", "qty": 1}, {"product": "Mouse", "qty": 2}]'),
    ('Jane', '[{"product": "Phone", "qty": 1}]');

SELECT 
    customer,
    jsonb_array_elements(items) AS item
FROM orders_json;

-- Извлечение данных из массива
SELECT 
    customer,
    item->>'product' AS product,
    (item->>'qty')::INTEGER AS quantity
FROM orders_json,
     jsonb_array_elements(items) AS item;

-- Индексы для JSONB
-- GIN индекс для быстрого поиска
CREATE INDEX idx_products_metadata ON products_json USING GIN (metadata);

-- Индекс на конкретное поле
CREATE INDEX idx_products_brand ON products_json ((metadata->>'brand'));

-- Поиск по вложенным полям с индексом
CREATE INDEX idx_products_specs ON products_json USING GIN ((metadata->'specs'));

-- JSON функции построения
-- json_build_object
SELECT json_build_object(
    'name', 'John',
    'age', 30,
    'email', 'john@example.com'
);

-- json_build_array
SELECT json_build_array(1, 2, 3, 'four', true);

-- row_to_json - преобразовать строку в JSON
SELECT row_to_json(products_json)
FROM products_json
LIMIT 1;

-- to_jsonb - преобразовать любое значение в JSONB
SELECT to_jsonb(ARRAY[1, 2, 3]);

-- Валидация JSON
-- Проверка корректности JSON (автоматически при вставке)
INSERT INTO products_json (name, data)
VALUES ('Test', '{"valid": "json"}');  -- ОК

-- JSON Schema validation (через расширение)
-- CREATE EXTENSION IF NOT EXISTS jsonschema;

-- Практический пример: логи приложения
CREATE TABLE app_logs (
    id SERIAL PRIMARY KEY,
    timestamp TIMESTAMPTZ DEFAULT NOW(),
    level VARCHAR(20),
    message TEXT,
    context JSONB
);

INSERT INTO app_logs (level, message, context) VALUES
    ('ERROR', 'Database connection failed', 
     '{"host": "db.example.com", "port": 5432, "retry_count": 3}'),
    ('INFO', 'User logged in',
     '{"user_id": 123, "ip": "192.168.1.1", "device": "mobile"}'),
    ('WARNING', 'High memory usage',
     '{"memory_used": "85%", "threshold": "80%", "server": "web-01"}');

-- Запросы по логам
SELECT 
    timestamp,
    level,
    message,
    context->>'user_id' AS user_id
FROM app_logs
WHERE level = 'INFO'
  AND context ? 'user_id';

-- Агрегация логов по серверам
SELECT 
    context->>'server' AS server,
    COUNT(*) AS error_count
FROM app_logs
WHERE level = 'ERROR'
  AND context ? 'server'
GROUP BY context->>'server';
```

**Упражнение:**

1. Создайте таблицу с JSONB колонкой для хранения настроек пользователей
2. Вставьте несколько записей с разными JSON структурами
3. Выполните поиск по JSON полям
4. Обновите вложенное значение в JSON
5. Создайте GIN индекс и сравните производительность

---

### Задача 12: Full-Text Search (Полнотекстовый поиск)

**Цель:** Реализовать мощный поиск по тексту

**Практика:**

sql

```sql
-- Создание таблицы для поиска
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    author VARCHAR(100),
    published_date DATE,
    tags TEXT[]
);

INSERT INTO articles (title, content, author, published_date, tags) VALUES
    ('PostgreSQL Tutorial', 
     'PostgreSQL is a powerful open-source database system with many advanced features.',
     'John Doe',
     '2024-01-15',
     ARRAY['database', 'postgresql', 'tutorial']),
    ('Introduction to SQL',
     'SQL is the standard language for managing relational databases. Learn the basics here.',
     'Jane Smith',
     '2024-01-20',
     ARRAY['sql', 'database', 'basics']),
    ('Advanced PostgreSQL Features',
     'Explore advanced features like full-text search, JSON support, and window functions.',
     'John Doe',
     '2024-02-01',
     ARRAY['postgresql', 'advanced', 'features']);

-- Базовый текстовый поиск с LIKE (медленный)
SELECT * FROM articles
WHERE content LIKE '%PostgreSQL%';

-- Полнотекстовый поиск
-- to_tsvector - преобразовать текст в tsvector (токены для поиска)
-- to_tsquery - создать поисковый запрос

-- Простой поиск
SELECT title, content
FROM articles
WHERE to_tsvector('english', content) @@ to_tsquery('english', 'postgresql');

-- Поиск с операторами
-- & - AND
SELECT * FROM articles
WHERE to_tsvector('english', content) @@ to_tsquery('english', 'postgresql & features');

-- | - OR
SELECT * FROM articles
WHERE to_tsvector('english', content) @@ to_tsquery('english', 'postgresql | mysql');

-- ! - NOT
SELECT * FROM articles
WHERE to_tsvector('english', content) @@ to_tsquery('english', 'database & !mysql');

-- Поиск по нескольким полям
SELECT * FROM articles
WHERE to_tsvector('english', title || ' ' || content) 
      @@ to_tsquery('english', 'tutorial');

-- Добавление весов (A, B, C, D - от более важного к менее)
SELECT 
    title,
    ts_rank(
        setweight(to_tsvector('english', title), 'A') ||
        setweight(to_tsvector('english', content), 'B'),
        to_tsquery('english', 'postgresql')
    ) AS rank
FROM articles
WHERE 
    setweight(to_tsvector('english', title), 'A') ||
    setweight(to_tsvector('english', content), 'B')
    @@ to_tsquery('english', 'postgresql')
ORDER BY rank DESC;

-- Создание колонки для хранения tsvector (оптимизация)
ALTER TABLE articles ADD COLUMN search_vector tsvector;

-- Обновление search_vector
UPDATE articles
SET search_vector = 
    setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
    setweight(to_tsvector('english', coalesce(content, '')), 'B') ||
    setweight(to_tsvector('english', coalesce(author, '')), 'C');

-- Теперь поиск быстрее
SELECT title, content
FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql')
ORDER BY ts_rank(search_vector, to_tsquery('english', 'postgresql')) DESC;

-- Создание GIN индекса для полнотекстового поиска
CREATE INDEX idx_articles_search ON articles USING GIN (search_vector);

-- Триггер для автообновления search_vector
CREATE OR REPLACE FUNCTION articles_search_trigger() RETURNS trigger AS $$
BEGIN
    NEW.search_vector :=
        setweight(to_tsvector('english', coalesce(NEW.title, '')), 'A') ||
        setweight(to_tsvector('english', coalesce(NEW.content, '')), 'B') ||
        setweight(to_tsvector('english', coalesce(NEW.author, '')), 'C');
    RETURN NEW;
END
$$ LANGUAGE plpgsql;

CREATE TRIGGER tsvector_update BEFORE INSERT OR UPDATE
ON articles FOR EACH ROW EXECUTE FUNCTION articles_search_trigger();

-- Тестирование триггера
INSERT INTO articles (title, content, author, published_date) VALUES
    ('New Article', 'This article discusses PostgreSQL indexes', 'Bob Johnson', '2024-02-15');

SELECT title FROM articles
WHERE search_vector @@ to_tsquery('english', 'indexes')
ORDER BY ts_rank(search_vector, to_tsquery('english', 'indexes')) DESC;

-- Поиск с учетом словоформ (stemming)
-- to_tsquery автоматически применяет stemming
SELECT * FROM articles
WHERE search_vector @@ to_tsquery('english', 'features');
-- Найдет и 'feature' и 'features'

-- plainto_tsquery - простой синтаксис (автоматически добавляет &)
SELECT * FROM articles
WHERE search_vector @@ plainto_tsquery('english', 'postgresql database features');

-- phraseto_tsquery - поиск фразы
SELECT * FROM articles
WHERE search_vector @@ phraseto_tsquery('english', 'open source database');

-- websearch_to_tsquery - синтаксис как в веб-поиске
SELECT * FROM articles
WHERE search_vector @@ websearch_to_tsquery('english', '"advanced features" OR tutorial');

-- Подсветка найденных слов
SELECT 
    title,
    ts_headline('english', content, 
                to_tsquery('english', 'postgresql'),
                'StartSel=<b>, StopSel=</b>, MaxWords=50, MinWords=10'
    ) AS highlighted_content
FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql');

-- Статистика поиска
SELECT 
    to_tsvector('english', content) AS tokens,
    ts_stat.word,
    ts_stat.ndoc
FROM articles,
     LATERAL ts_stat('SELECT to_tsvector(''english'', content) FROM articles') AS ts_stat
LIMIT 10;

-- Поиск похожих документов
WITH search_doc AS (
    SELECT id, search_vector
    FROM articles
    WHERE id = 1
)
SELECT 
    a.title,
    a.id,
    ts_rank(a.search_vector, sd.search_vector::tsquery) AS similarity
FROM articles a, search_doc sd
WHERE a.id != sd.id
ORDER BY similarity DESC
LIMIT 5;

-- Многоязычный поиск
-- Для русского языка
CREATE TABLE articles_ru (
    id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT,
    search_vector tsvector
);

INSERT INTO articles_ru (title, content) VALUES
    ('База данных PostgreSQL', 'PostgreSQL является мощной системой управления базами данных');

UPDATE articles_ru
SET search_vector = to_tsvector('russian', coalesce(title, '') || ' ' || coalesce(content, ''));

SELECT * FROM articles_ru
WHERE search_vector @@ to_tsquery('russian', 'база & данных');

-- Настройка конфигурации поиска
-- Посмотреть доступные конфигурации
SELECT cfgname FROM pg_ts_config;

-- Создать свою конфигурацию
CREATE TEXT SEARCH CONFIGURATION my_config (COPY = english);
ALTER TEXT SEARCH CONFIGURATION my_config
    ALTER MAPPING FOR word WITH simple;  -- Без stemming для определенных типов
```

**Упражнение:**

1. Создайте таблицу blog_posts с полнотекстовым поиском
2. Добавьте GIN индекс на tsvector колонку
3. Реализуйте поиск по заголовку и содержимому
4. Добавьте триггер для автоматического обновления search_vector
5. Реализуйте ранжирование результатов поиска

---

### Задача 13: Partitioning (Секционирование)

**Цель:** Разделить большие таблицы на управляемые части

**Практика:**

sql

```sql
-- Партиционирование по диапазону (RANGE)
-- Пример: логи по датам
CREATE TABLE logs (
    id BIGSERIAL,
    log_date DATE NOT NULL,
    level VARCHAR(10),
    message TEXT,
    user_id INTEGER
) PARTITION BY RANGE (log_date);

-- Создание партиций
CREATE TABLE logs_2024_01 PARTITION OF logs
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE logs_2024_02 PARTITION OF logs
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

CREATE TABLE logs_2024_03 PARTITION OF logs
    FOR VALUES FROM ('2024-03-01') TO ('2024-04-01');

-- Партиция по умолчанию (для данных вне диапазонов)
CREATE TABLE logs_default PARTITION OF logs DEFAULT;

-- Вставка данных (автоматически попадает в нужную партицию)
INSERT INTO logs (log_date, level, message, user_id) VALUES
    ('2024-01-15', 'INFO', 'User logged in', 123),
    ('2024-02-10', 'ERROR', 'Connection failed', 456),
    ('2024-03-20', 'WARNING', 'High memory usage', 789);

-- Запрос автоматически использует только нужные партиции
SELECT * FROM logs
WHERE log_date BETWEEN '2024-01-01' AND '2024-01-31';

-- EXPLAIN покажет, что используется только logs_2024_01
EXPLAIN SELECT * FROM logs WHERE log_date = '2024-01-15';

-- Партиционирование по списку (LIST)
CREATE TABLE sales_by_region (
    id BIGSERIAL,
    region VARCHAR(50) NOT NULL,
    product VARCHAR(100),
    amount DECIMAL(10,2),
    sale_date DATE
) PARTITION BY LIST (region);

CREATE TABLE sales_north PARTITION OF sales_by_region
    FOR VALUES IN ('North', 'Northeast');

CREATE TABLE sales_south PARTITION OF sales_by_region
    FOR VALUES IN ('South', 'Southeast');

CREATE TABLE sales_west PARTITION OF sales_by_region
```

Continue

15 нояб.

sql

```sql
    FOR VALUES IN ('West', 'Northwest');

CREATE TABLE sales_east PARTITION OF sales_by_region
    FOR VALUES IN ('East', 'Northeast');

-- Партиционирование по хешу (HASH) - для равномерного распределения
CREATE TABLE orders_hash (
    id BIGSERIAL,
    customer_id INTEGER NOT NULL,
    order_date DATE,
    total DECIMAL(10,2)
) PARTITION BY HASH (customer_id);

-- Создание 4 хеш-партиций
CREATE TABLE orders_hash_0 PARTITION OF orders_hash
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE orders_hash_1 PARTITION OF orders_hash
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);

CREATE TABLE orders_hash_2 PARTITION OF orders_hash
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);

CREATE TABLE orders_hash_3 PARTITION OF orders_hash
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);

-- Многоуровневое партиционирование (sub-partitioning)
CREATE TABLE metrics (
    id BIGSERIAL,
    metric_date DATE NOT NULL,
    region VARCHAR(50) NOT NULL,
    metric_name VARCHAR(100),
    value DECIMAL(10,2)
) PARTITION BY RANGE (metric_date);

-- Партиция по месяцам
CREATE TABLE metrics_2024_01 PARTITION OF metrics
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01')
    PARTITION BY LIST (region);

-- Суб-партиции по регионам
CREATE TABLE metrics_2024_01_north PARTITION OF metrics_2024_01
    FOR VALUES IN ('North');

CREATE TABLE metrics_2024_01_south PARTITION OF metrics_2024_01
    FOR VALUES IN ('South');

-- Индексы на партиционированных таблицах
-- Индекс на главной таблице создаст индексы на всех партициях
CREATE INDEX idx_logs_user_id ON logs (user_id);
CREATE INDEX idx_logs_level ON logs (level);

-- Индекс на конкретной партиции
CREATE INDEX idx_logs_2024_01_message ON logs_2024_01 (message);

-- Уникальные ограничения должны включать ключ партиционирования
CREATE TABLE partitioned_users (
    id BIGSERIAL,
    email VARCHAR(255) NOT NULL,
    signup_date DATE NOT NULL,
    UNIQUE (email, signup_date)  -- signup_date - ключ партиционирования
) PARTITION BY RANGE (signup_date);

-- Управление партициями
-- Отсоединение партиции (для архивации или удаления)
ALTER TABLE logs DETACH PARTITION logs_2024_01;

-- Теперь logs_2024_01 - обычная таблица, можно архивировать
-- pg_dump -t logs_2024_01 mydb > logs_2024_01.sql

-- Удаление старой партиции
DROP TABLE logs_2024_01;

-- Присоединение существующей таблицы как партиции
CREATE TABLE logs_2024_04 (LIKE logs INCLUDING ALL);
ALTER TABLE logs ATTACH PARTITION logs_2024_04
    FOR VALUES FROM ('2024-04-01') TO ('2024-05-01');

-- Автоматическое создание партиций с помощью функции
CREATE OR REPLACE FUNCTION create_monthly_partition(
    base_table TEXT,
    partition_date DATE
) RETURNS VOID AS $$
DECLARE
    partition_name TEXT;
    start_date DATE;
    end_date DATE;
BEGIN
    start_date := DATE_TRUNC('month', partition_date);
    end_date := start_date + INTERVAL '1 month';
    partition_name := base_table || '_' || TO_CHAR(start_date, 'YYYY_MM');
    
    EXECUTE FORMAT(
        'CREATE TABLE IF NOT EXISTS %I PARTITION OF %I
         FOR VALUES FROM (%L) TO (%L)',
        partition_name, base_table, start_date, end_date
    );
    
    RAISE NOTICE 'Created partition: %', partition_name;
END;
$$ LANGUAGE plpgsql;

-- Использование функции
SELECT create_monthly_partition('logs', '2024-05-01');
SELECT create_monthly_partition('logs', '2024-06-01');

-- Триггер для автоматического создания партиций при вставке
CREATE OR REPLACE FUNCTION auto_create_partition()
RETURNS TRIGGER AS $$
BEGIN
    PERFORM create_monthly_partition(TG_TABLE_NAME, NEW.log_date);
    RETURN NEW;
EXCEPTION
    WHEN duplicate_table THEN
        RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Статистика по партициям
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE tablename LIKE 'logs_%'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Просмотр всех партиций таблицы
SELECT
    nmsp_parent.nspname AS parent_schema,
    parent.relname AS parent,
    nmsp_child.nspname AS child_schema,
    child.relname AS child
FROM pg_inherits
    JOIN pg_class parent ON pg_inherits.inhparent = parent.oid
    JOIN pg_class child ON pg_inherits.inhrelid = child.oid
    JOIN pg_namespace nmsp_parent ON nmsp_parent.oid = parent.relnamespace
    JOIN pg_namespace nmsp_child ON nmsp_child.oid = child.relnamespace
WHERE parent.relname = 'logs';

-- Перемещение данных между партициями
-- Иногда нужно при изменении схемы партиционирования
CREATE TABLE logs_new (LIKE logs INCLUDING ALL);

-- Скопировать данные
INSERT INTO logs_new SELECT * FROM logs WHERE log_date < '2024-01-01';

-- Переименовать таблицы
ALTER TABLE logs RENAME TO logs_old;
ALTER TABLE logs_new RENAME TO logs;

-- Преимущества партиционирования:
-- 1. Улучшение производительности запросов (partition pruning)
-- 2. Быстрое удаление старых данных (DROP PARTITION)
-- 3. Параллельная обработка партиций
-- 4. Упрощение архивирования

-- Мониторинг эффективности партиционирования
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM logs 
WHERE log_date BETWEEN '2024-01-01' AND '2024-01-15';
-- Должен показать, что используется только одна партиция

-- Пример: система мониторинга с автоматическим партиционированием
CREATE TABLE system_metrics (
    id BIGSERIAL,
    metric_time TIMESTAMPTZ NOT NULL,
    hostname VARCHAR(100),
    metric_name VARCHAR(100),
    value DOUBLE PRECISION
) PARTITION BY RANGE (metric_time);

-- Функция для создания еженедельных партиций
CREATE OR REPLACE FUNCTION create_weekly_partitions(
    start_date DATE,
    num_weeks INTEGER
) RETURNS VOID AS $$
DECLARE
    week_start DATE;
    week_end DATE;
    partition_name TEXT;
BEGIN
    FOR i IN 0..(num_weeks - 1) LOOP
        week_start := start_date + (i * INTERVAL '1 week');
        week_end := week_start + INTERVAL '1 week';
        partition_name := 'system_metrics_' || TO_CHAR(week_start, 'YYYY_WW');
        
        EXECUTE FORMAT(
            'CREATE TABLE IF NOT EXISTS %I PARTITION OF system_metrics
             FOR VALUES FROM (%L) TO (%L)',
            partition_name, week_start, week_end
        );
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- Создать партиции на 12 недель вперед
SELECT create_weekly_partitions(CURRENT_DATE, 12);

-- Функция очистки старых партиций
CREATE OR REPLACE FUNCTION drop_old_partitions(
    base_table TEXT,
    retention_days INTEGER
) RETURNS VOID AS $$
DECLARE
    partition_record RECORD;
    cutoff_date DATE;
BEGIN
    cutoff_date := CURRENT_DATE - retention_days;
    
    FOR partition_record IN
        SELECT tablename
        FROM pg_tables
        WHERE tablename LIKE base_table || '_%'
        AND schemaname = 'public'
    LOOP
        -- Проверить дату партиции и удалить если старая
        -- (упрощенная логика, в продакшене нужна более точная проверка)
        EXECUTE 'DROP TABLE IF EXISTS ' || partition_record.tablename;
        RAISE NOTICE 'Dropped old partition: %', partition_record.tablename;
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- Запустить очистку старых партиций (старше 90 дней)
-- SELECT drop_old_partitions('logs', 90);
```

**Упражнение:**

1. Создайте партиционированную таблицу для логов по месяцам
2. Создайте несколько партиций на будущие месяцы
3. Вставьте данные и проверьте распределение по партициям
4. Используйте EXPLAIN для проверки partition pruning
5. Создайте функцию для автоматического создания партиций
6. Реализуйте процесс архивации старых партиций

---

## 🔴 SENIOR DATABASE USER

### Задача 14: Репликация и High Availability

**Цель:** Настроить репликацию для отказоустойчивости

**Практика:**

bash

```bash
# НАСТРОЙКА PRIMARY СЕРВЕРА

# 1. Редактировать postgresql.conf
sudo nano /etc/postgresql/16/main/postgresql.conf

# Добавить/изменить:
# wal_level = replica
# max_wal_senders = 10
# max_replication_slots = 10
# hot_standby = on
# archive_mode = on
# archive_command = 'test ! -f /var/lib/postgresql/archive/%f && cp %p /var/lib/postgresql/archive/%f'

# 2. Создать директорию для архивов
sudo mkdir -p /var/lib/postgresql/archive
sudo chown postgres:postgres /var/lib/postgresql/archive

# 3. Настроить pg_hba.conf для репликации
sudo nano /etc/postgresql/16/main/pg_hba.conf

# Добавить строку:
# host    replication     replicator      replica_ip/32       md5

# 4. Создать пользователя для репликации
sudo -u postgres psql
```

sql

```sql
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'repl_password';
```

bash

```bash
# 5. Перезапустить PostgreSQL
sudo systemctl restart postgresql

# НАСТРОЙКА REPLICA СЕРВЕРА

# 1. Остановить PostgreSQL на replica
sudo systemctl stop postgresql

# 2. Очистить data directory
sudo rm -rf /var/lib/postgresql/16/main/*

# 3. Сделать базовый backup с primary
sudo -u postgres pg_basebackup -h primary_ip -D /var/lib/postgresql/16/main \
    -U replicator -P -v -R -X stream -C -S replica_slot

# Флаги:
# -R: создать standby.signal и настроить репликацию
# -X stream: стримить WAL во время backup
# -C: создать replication slot
# -S: имя replication slot

# 4. Запустить replica
sudo systemctl start postgresql

# 5. Проверить статус репликации
```

sql

```sql
-- На PRIMARY
SELECT * FROM pg_stat_replication;

SELECT 
    client_addr,
    state,
    sent_lsn,
    write_lsn,
    flush_lsn,
    replay_lsn,
    sync_state
FROM pg_stat_replication;

-- На REPLICA
SELECT pg_is_in_recovery();  -- Должно вернуть true

SELECT 
    pg_last_wal_receive_lsn(),
    pg_last_wal_replay_lsn(),
    pg_last_xact_replay_timestamp();
```

bash

```bash
# STREAMING REPLICATION с несколькими репликами

# postgresql.conf на primary
# synchronous_standby_names = 'FIRST 1 (replica1, replica2)'
# Хотя бы одна реплика должна подтвердить запись

# Проверка синхронной репликации
```

sql

```sql
SELECT 
    application_name,
    sync_state,
    state
FROM pg_stat_replication;
-- sync_state может быть: sync, potential, async
```

bash

```bash
# АВТОМАТИЧЕСКИЙ FAILOVER с pg_auto_failover

# Установка на coordinator узле
sudo apt install pg-auto-failover-cli

# Инициализация monitor
sudo -u postgres pg_autoctl create monitor \
    --pgdata /var/lib/postgresql/monitor \
    --pgport 5433

# На primary узле
sudo -u postgres pg_autoctl create postgres \
    --pgdata /var/lib/postgresql/16/main \
    --pgport 5432 \
    --hostname primary.example.com \
    --monitor postgres://autoctl_node@monitor.example.com:5433/pg_auto_failover

# На standby узлах
sudo -u postgres pg_autoctl create postgres \
    --pgdata /var/lib/postgresql/16/main \
    --pgport 5432 \
    --hostname standby1.example.com \
    --monitor postgres://autoctl_node@monitor.example.com:5433/pg_auto_failover

# Мониторинг состояния
sudo -u postgres pg_autoctl show state

# Ручной failover
sudo -u postgres pg_autoctl perform failover
```

sql

```sql
-- ЛОГИЧЕСКАЯ РЕПЛИКАЦИЯ (для выборочной репликации таблиц)

-- На PRIMARY
-- 1. Создать publication
CREATE PUBLICATION my_publication FOR TABLE users, orders;

-- Или для всех таблиц
CREATE PUBLICATION all_tables FOR ALL TABLES;

-- 2. Проверить publications
SELECT * FROM pg_publication;
SELECT * FROM pg_publication_tables;

-- На REPLICA
-- 1. Создать таблицы (схема должна совпадать)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER,
    total DECIMAL(10,2)
);

-- 2. Создать subscription
CREATE SUBSCRIPTION my_subscription
    CONNECTION 'host=primary_ip port=5432 dbname=mydb user=replicator password=repl_password'
    PUBLICATION my_publication;

-- 3. Проверить статус
SELECT * FROM pg_subscription;
SELECT * FROM pg_stat_subscription;

-- Управление subscription
ALTER SUBSCRIPTION my_subscription DISABLE;
ALTER SUBSCRIPTION my_subscription ENABLE;
ALTER SUBSCRIPTION my_subscription REFRESH PUBLICATION;

-- Удаление
DROP SUBSCRIPTION my_subscription;

-- На PRIMARY удалить publication
DROP PUBLICATION my_publication;

-- МОНИТОРИНГ РЕПЛИКАЦИИ

-- Задержка репликации
SELECT 
    now() - pg_last_xact_replay_timestamp() AS replication_lag
FROM pg_stat_replication;

-- Размер WAL lag
SELECT 
    client_addr,
    pg_wal_lsn_diff(sent_lsn, replay_lsn) AS lag_bytes
FROM pg_stat_replication;

-- Replication slots
SELECT 
    slot_name,
    slot_type,
    active,
    pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn) AS retained_wal
FROM pg_replication_slots;

-- Удаление неиспользуемого слота
SELECT pg_drop_replication_slot('unused_slot');

-- КОНФЛИКТЫ РЕПЛИКАЦИИ (на replica)
SELECT * FROM pg_stat_database_conflicts;

-- Решение конфликтов
-- Если запросы на replica блокируют репликацию:
-- 1. Увеличить max_standby_streaming_delay
-- 2. Или использовать hot_standby_feedback = on
```

bash

```bash
# BACKUP стратегии с репликацией

# 1. Continuous archiving (Point-in-Time Recovery)
# На primary в postgresql.conf:
# archive_mode = on
# archive_command = 'rsync -a %p backup_server:/archive/%f'

# Создание базового backup
pg_basebackup -D /backup/base -F tar -z -P

# Восстановление на конкретный момент времени
# 1. Восстановить базовый backup
# 2. Создать recovery.conf
# restore_command = 'cp /archive/%f %p'
# recovery_target_time = '2024-01-15 14:30:00'

# 2. Barman (Backup and Recovery Manager)
# Установка
sudo apt install barman

# Настройка
sudo -u barman barman receive-wal primary_server
sudo -u barman barman backup primary_server

# Восстановление
sudo -u barman barman recover primary_server latest /var/lib/postgresql/16/main
```

**Упражнение:**

1. Настройте streaming репликацию между двумя серверами
2. Проверьте статус репликации запросами
3. Протестируйте failover вручную
4. Настройте логическую репликацию для конкретных таблиц
5. Настройте мониторинг задержки репликации

---

### Задача 15: Производительность и оптимизация

**Цель:** Научиться анализировать и оптимизировать запросы

**Практика:**

sql

```sql
-- EXPLAIN и EXPLAIN ANALYZE

-- EXPLAIN показывает план запроса
EXPLAIN 
SELECT * FROM users WHERE email = 'test@example.com';

-- EXPLAIN ANALYZE выполняет запрос и показывает реальное время
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'test@example.com';

-- Детальный анализ с буферами
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT u.username, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.username;

-- Понимание плана выполнения:
-- Seq Scan - последовательное сканирование (медленно на больших таблицах)
-- Index Scan - использование индекса (быстро)
-- Index Only Scan - данные только из индекса (очень быстро)
-- Bitmap Index Scan - для больших результатов
-- Nested Loop - вложенные циклы (для маленьких джойнов)
-- Hash Join - хеш-соединение (для больших джойнов)
-- Merge Join - соединение слиянием (для отсортированных данных)

-- VACUUM и ANALYZE

-- VACUUM - очистка мертвых строк и обновление статистики
VACUUM users;

-- VACUUM FULL - полная очистка с блокировкой таблицы
VACUUM FULL users;

-- VACUUM ANALYZE - с обновлением статистики планировщика
VACUUM ANALYZE users;

-- Автовакуум (настройки в postgresql.conf)
-- autovacuum = on
-- autovacuum_naptime = 1min
-- autovacuum_vacuum_threshold = 50
-- autovacuum_analyze_threshold = 50

-- Проверка последнего VACUUM
SELECT 
    schemaname,
    relname,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables;

-- ANALYZE - обновление статистики для планировщика
ANALYZE users;
ANALYZE;  -- Для всех таблиц

-- Статистика таблиц
SELECT * FROM pg_stats WHERE tablename = 'users';

-- ИНДЕКСЫ для производительности

-- Проблемные запросы без индексов
EXPLAIN ANALYZE
SELECT * FROM users WHERE city = 'New York';  -- Seq Scan

-- Создание индекса
CREATE INDEX idx_users_city ON users(city);

-- Теперь быстрее
EXPLAIN ANALYZE
SELECT * FROM users WHERE city = 'New York';  -- Index Scan

-- Частичный индекс (только для активных пользователей)
CREATE INDEX idx_users_active_email 
ON users(email) 
WHERE is_active = true;

-- Индекс для сортировки
CREATE INDEX idx_users_created_at ON users(created_at DESC);

SELECT * FROM users ORDER BY created_at DESC LIMIT 10;
-- Index Scan Backward

-- Covering index (индекс покрывает все нужные колонки)
CREATE INDEX idx_users_email_username ON users(email) INCLUDE (username);

SELECT username FROM users WHERE email = 'test@example.com';
-- Index Only Scan

-- Мультиколоночный индекс (порядок важен!)
CREATE INDEX idx_users_city_age ON users(city, age);

-- Эффективно:
SELECT * FROM users WHERE city = 'New York' AND age > 25;
SELECT * FROM users WHERE city = 'New York';  -- Использует индекс

-- Неэффективно:
SELECT * FROM users WHERE age > 25;  -- НЕ использует индекс

-- Индекс для LIKE запросов
CREATE INDEX idx_users_username_pattern ON users(username text_pattern_ops);

SELECT * FROM users WHERE username LIKE 'john%';  -- Использует индекс
SELECT * FROM users WHERE username LIKE '%john%';  -- НЕ использует индекс

-- BRIN индекс для больших последовательных данных
CREATE INDEX idx_logs_created_brin ON logs USING BRIN (created_at);
-- Занимает меньше места, подходит для времени/дат в больших таблицах

-- GiST индекс для геоданных, полнотекстового поиска
CREATE INDEX idx_locations_point ON locations USING GIST (coordinates);

-- Неиспользуемые индексы
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;

-- Дублирующиеся индексы
SELECT 
    idx1.indexrelid::regclass AS index1,
    idx2.indexrelid::regclass AS index2,
    idx1.indrelid::regclass AS table
FROM pg_index idx1
JOIN pg_index idx2 ON idx1.indrelid = idx2.indrelid
WHERE idx1.indexrelid < idx2.indexrelid
  AND idx1.indkey::text = idx2.indkey::text;

-- ОПТИМИЗАЦИЯ ЗАПРОСОВ

-- 1. Используйте LIMIT для больших результатов
SELECT * FROM users ORDER BY created_at DESC LIMIT 100;

-- 2. Избегайте SELECT *
-- Плохо:
SELECT * FROM users;
-- Хорошо:
SELECT id, username, email FROM users;

-- 3. Используйте EXISTS вместо IN для подзапросов
-- Медленно:
SELECT * FROM users 
WHERE id IN (SELECT user_id FROM orders WHERE total > 1000);

-- Быстрее:
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.user_id = u.id AND o.total > 1000
);

-- 4. JOIN вместо подзапросов
-- Медленно:
SELECT 
    u.username,
    (SELECT COUNT(*) FROM orders WHERE user_id = u.id) as order_count
FROM users u;

-- Быстрее:
SELECT 
    u.username,
    COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.username;

-- 5. Избегайте функций в WHERE (они блокируют использование индексов)
-- Плохо:
SELECT * FROM users WHERE LOWER(email) = 'test@example.com';

-- Хорошо (создать функциональный индекс):
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
SELECT * FROM users WHERE LOWER(email) = 'test@example.com';

-- Или использовать COLLATE
CREATE INDEX idx_users_email_ci ON users(email COLLATE "en_US");

-- CONNECTION POOLING

-- PgBouncer конфигурация (/etc/pgbouncer/pgbouncer.ini)
# [databases]
# mydb = host=localhost port=5432 dbname=mydb
# 
# [pgbouncer]
# listen_addr = *
# listen_port = 6432
# auth_file = /etc/pgbouncer/userlist.txt
# pool_mode = transaction
# max_client_conn = 1000
# default_pool_size = 25

-- Подключение через PgBouncer
-- psql -h localhost -p 6432 -U myuser mydb

-- НАСТРОЙКА POSTGRESQL.CONF для производительности

-- Память
# shared_buffers = 256MB          # 25% от RAM
# effective_cache_size = 1GB      # 50-75% от RAM
# work_mem = 10MB                 # Для сортировки/JOIN
# maintenance_work_mem = 128MB    # Для VACUUM, INDEX

-- Checkpoint
# checkpoint_completion_target = 0.9
# wal_buffers = 16MB
# max_wal_size = 2GB
# min_wal_size = 1GB

-- Планировщик запросов
# random_page_cost = 1.1          # Для SSD
# effective_io_concurrency = 200  # Для SSD
# cpu_tuple_cost = 0.01

-- Параллельные запросы
# max_worker_processes = 8
# max_parallel_workers_per_gather = 4
# max_parallel_workers = 8
# parallel_tuple_cost = 0.1

-- МОНИТОРИНГ ПРОИЗВОДИТЕЛЬНОСТИ

-- Медленные запросы
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 20;

-- Включить pg_stat_statements
-- В postgresql.conf:
-- shared_preload_libraries = 'pg_stat_statements'

CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Блокировки
SELECT 
    pg_stat_activity.pid,
    pg_stat_activity.usename,
    pg_stat_activity.query,
    pg_stat_activity.state,
    pg_locks.mode,
    pg_locks.granted
FROM pg_stat_activity
JOIN pg_locks ON pg_stat_activity.pid = pg_locks.pid
WHERE NOT pg_locks.granted;

-- Размеры таблиц и индексов
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size,
    pg_size_pretty(pg_indexes_size(schemaname||'.'||tablename)) AS indexes_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Кеш hit ratio (должен быть > 99%)
SELECT 
    sum(heap_blks_read) as heap_read,
    sum(heap_blks_hit)  as heap_hit,
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM pg_statio_user_tables;

-- Активные подключения
SELECT 
    datname,
    count(*) as connections
FROM pg_stat_activity
GROUP BY datname;

-- Длительные запросы
SELECT 
    pid,
    now() - query_start as duration,
    state,
    query
FROM pg_stat_activity
WHERE state != 'idle'
  AND now() - query_start > interval '5 minutes'
ORDER BY duration DESC;

-- Убить долгий запрос
SELECT pg_terminate_backend(pid);

-- Убить все idle in transaction
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE state = 'idle in transaction'
  AND now() - state_change > interval '10 minutes';
```

**Упражнение:**

1. Найдите медленные запросы с помощью EXPLAIN ANALYZE
2. Создайте подходящие индексы для оптимизации
3. Настройте pg_stat_statements для мониторинга
4. Проанализируйте использование индексов
5. Оптимизируйте postgresql.conf для вашего сервера
6. Настройте автовакуум правильно

---

### Задача 16: Backup и Recovery стратегии

**Цель:** Реализовать надежное резервное копирование

**Практика:**

bash

```bash
# ЛОГИЧЕСКОЕ РЕЗЕРВНОЕ КОПИРОВАНИЕ (pg_dump)

# Дамп одной базы данных
pg_dump -U postgres -d mydb -F c -f mydb_backup.dump

# Флаги:
# -F c: custom format (сжатый, может выборочно восстанавливать)
# -F p: plain SQL (текстовый файл)
# -F d: directory format (параллельное восстановление)
# -F t: tar format

# Дамп со сжатием
pg_dump -U postgres -d mydb -F c -Z 9 -f mydb_backup.dump

# Дамп только схемы (без данных)
pg_dump -U postgres -d mydb --schema-only -f schema.sql

# Дамп только данных
pg_dump -U postgres -d mydb --data-only -f data.sql

# Дамп конкретных таблиц
pg_dump -U postgres -d mydb -t users -t orders -f tables_backup.dump

# Исключить таблицы
pg_dump -U postgres -d mydb -T logs -T temp_data -f mydb_no_logs.dump

# Дамп со всеми БД
pg_dumpall -U postgres -f all_databases.sql

# Дамп только ролей и табл пространств
pg_dumpall -U postgres --globals-only -f globals.sql

# ВОССТАНОВЛЕНИЕ из dump

# Восстановление custom
```

Continue

16 нояб.

bash

```bash
# Восстановление custom format
pg_restore -U postgres -d mydb -v mydb_backup.dump

# Восстановление с созданием новой БД
createdb -U postgres mydb_restored
pg_restore -U postgres -d mydb_restored mydb_backup.dump

# Восстановление конкретных таблиц
pg_restore -U postgres -d mydb -t users -t orders mydb_backup.dump

# Параллельное восстановление (быстрее)
pg_restore -U postgres -d mydb -j 4 mydb_backup.dump

# Восстановление только схемы
pg_restore -U postgres -d mydb --schema-only mydb_backup.dump

# Восстановление только данных
pg_restore -U postgres -d mydb --data-only mydb_backup.dump

# Восстановление из plain SQL
psql -U postgres -d mydb -f backup.sql

# ФИЗИЧЕСКОЕ РЕЗЕРВНОЕ КОПИРОВАНИЕ (pg_basebackup)

# Базовый backup всего кластера
pg_basebackup -U postgres -D /backup/pgdata -F tar -z -P

# Флаги:
# -D: директория назначения
# -F tar: tar формат
# -z: сжатие gzip
# -P: показывать прогресс
# -X stream: включить WAL файлы

# Backup с streaming WAL
pg_basebackup -U replicator -h primary -D /backup/base \
    -F tar -z -P -X stream

# Backup в directory формате
pg_basebackup -U postgres -D /backup/pgdata -F plain -P

# POINT-IN-TIME RECOVERY (PITR)

# 1. Настройка continuous archiving в postgresql.conf
# wal_level = replica
# archive_mode = on
# archive_command = 'test ! -f /mnt/archive/%f && cp %p /mnt/archive/%f'
# archive_timeout = 60  # Форсировать архивирование каждые 60 сек

# 2. Создать директорию для архивов
sudo mkdir -p /mnt/archive
sudo chown postgres:postgres /mnt/archive

# 3. Сделать базовый backup
pg_basebackup -D /backup/base -F tar -z

# 4. В случае сбоя - восстановление
sudo systemctl stop postgresql
sudo rm -rf /var/lib/postgresql/16/main/*
sudo tar -xzf /backup/base/base.tar.gz -C /var/lib/postgresql/16/main/
sudo tar -xzf /backup/base/pg_wal.tar.gz -C /var/lib/postgresql/16/main/pg_wal/

# 5. Создать recovery.signal
sudo touch /var/lib/postgresql/16/main/recovery.signal

# 6. Настроить recovery в postgresql.conf или postgresql.auto.conf
# restore_command = 'cp /mnt/archive/%f %p'
# recovery_target_time = '2024-01-15 14:30:00'
# или
# recovery_target_name = 'before_disaster'
# или
# recovery_target_xid = '12345'

sudo nano /var/lib/postgresql/16/main/postgresql.auto.conf
# restore_command = 'cp /mnt/archive/%f %p'
# recovery_target_time = '2024-01-15 14:30:00'

# 7. Запустить PostgreSQL
sudo systemctl start postgresql

# 8. После успешного восстановления промотировать
SELECT pg_wal_replay_resume();

# BARMAN - профессиональный backup менеджер

# Установка Barman
sudo apt install barman barman-cli

# Настройка /etc/barman.conf
[barman]
barman_user = barman
configuration_files_directory = /etc/barman.d
barman_home = /var/lib/barman
log_file = /var/log/barman/barman.log
log_level = INFO
compression = gzip

# Настройка сервера /etc/barman.d/primary.conf
[primary]
description = "Primary PostgreSQL Server"
conninfo = host=primary port=5432 user=barman dbname=postgres
backup_method = postgres
streaming_conninfo = host=primary port=5432 user=streaming_barman
streaming_archiver = on
slot_name = barman
path_prefix = /usr/pgsql-16/bin
backup_directory = /var/lib/barman/primary
retention_policy = RECOVERY WINDOW OF 7 DAYS
wal_retention_policy = main
archiver = on

# На PostgreSQL сервере создать пользователей
CREATE USER barman SUPERUSER;
CREATE USER streaming_barman REPLICATION;

# Проверка конфигурации Barman
sudo -u barman barman check primary

# Первый backup
sudo -u barman barman backup primary

# Список backups
sudo -u barman barman list-backup primary

# Информация о backup
sudo -u barman barman show-backup primary latest

# Восстановление
sudo -u barman barman recover primary latest /var/lib/postgresql/16/main

# Восстановление на конкретное время
sudo -u barman barman recover primary latest /var/lib/postgresql/16/main \
    --target-time "2024-01-15 14:30:00"

# Удаление старых backups
sudo -u barman barman delete primary oldest

# WAL архивирование через Barman
sudo -u barman barman cron

# PGBACKREST - современная альтернатива

# Установка
sudo apt install pgbackrest

# Настройка /etc/pgbackrest/pgbackrest.conf
[global]
repo1-path=/var/lib/pgbackrest
repo1-retention-full=2
repo1-retention-diff=3

[main]
pg1-path=/var/lib/postgresql/16/main
pg1-port=5432

# На PostgreSQL настроить archive_command
# archive_command = 'pgbackrest --stanza=main archive-push %p'

# Создать stanza
sudo -u postgres pgbackrest --stanza=main stanza-create

# Полный backup
sudo -u postgres pgbackrest --stanza=main --type=full backup

# Инкрементальный backup
sudo -u postgres pgbackrest --stanza=main --type=incr backup

# Дифференциальный backup
sudo -u postgres pgbackrest --stanza=main --type=diff backup

# Список backups
sudo -u postgres pgbackrest --stanza=main info

# Восстановление
sudo systemctl stop postgresql
sudo -u postgres pgbackrest --stanza=main --delta restore
sudo systemctl start postgresql

# PITR с pgBackRest
sudo -u postgres pgbackrest --stanza=main \
    --type=time --target="2024-01-15 14:30:00" \
    --target-action=promote restore

# АВТОМАТИЗАЦИЯ BACKUPS

# Cron задание для ежедневных backups
sudo crontab -e -u postgres

# Добавить:
# Полный backup каждую неделю в воскресенье в 2:00
0 2 * * 0 /usr/bin/pgbackrest --stanza=main --type=full backup

# Инкрементальный backup каждый день в 2:00
0 2 * * 1-6 /usr/bin/pgbackrest --stanza=main --type=incr backup

# Или с pg_dump
0 2 * * * /usr/bin/pg_dump -U postgres mydb | gzip > /backup/mydb_$(date +\%Y\%m\%d).sql.gz

# Скрипт для ротации старых backups
cat > /usr/local/bin/rotate_backups.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/backup"
RETENTION_DAYS=7

find $BACKUP_DIR -name "*.dump" -mtime +$RETENTION_DAYS -delete
find $BACKUP_DIR -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

echo "Old backups deleted: $(date)" >> /var/log/backup_rotation.log
EOF

chmod +x /usr/local/bin/rotate_backups.sh

# Добавить в cron
0 3 * * * /usr/local/bin/rotate_backups.sh

# ОБЛАЧНЫЕ BACKUPS

# AWS S3 с pg_dump
pg_dump -U postgres mydb | gzip | aws s3 cp - s3://my-bucket/backups/mydb_$(date +\%Y\%m\%d).sql.gz

# Barman с S3
# В /etc/barman.d/primary.conf
# backup_options = concurrent_backup
# archiver = on
# archiver_batch_size = 50

# WAL-G для облачных backups
# Установка
wget https://github.com/wal-g/wal-g/releases/download/v2.0.1/wal-g-pg-ubuntu-20.04-amd64.tar.gz
tar -xzf wal-g-pg-ubuntu-20.04-amd64.tar.gz
sudo mv wal-g /usr/local/bin/

# Настройка переменных окружения
export WALG_S3_PREFIX=s3://my-bucket/postgres-backups
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
export AWS_REGION=us-east-1

# В postgresql.conf
# archive_mode = on
# archive_command = '/usr/local/bin/wal-g wal-push %p'
# restore_command = '/usr/local/bin/wal-g wal-fetch %f %p'

# Создать backup
wal-g backup-push /var/lib/postgresql/16/main

# Список backups
wal-g backup-list

# Восстановление
wal-g backup-fetch /var/lib/postgresql/16/main LATEST

# МОНИТОРИНГ BACKUPS

# Проверка последнего backup
SELECT 
    pg_backup_start_time() AS backup_start,
    pg_is_in_backup() AS in_backup;

# Размер backup
du -sh /backup/*

# Проверка WAL архивов
ls -lh /mnt/archive/ | tail -20

# Тест восстановления (важно!)
# Регулярно тестировать восстановление на тестовом сервере

cat > /usr/local/bin/test_restore.sh << 'EOF'
#!/bin/bash
TEST_DIR="/tmp/restore_test"
BACKUP_FILE="/backup/mydb_latest.dump"

# Создать тестовую БД
createdb test_restore

# Восстановить
pg_restore -d test_restore $BACKUP_FILE

# Проверить
psql -d test_restore -c "SELECT COUNT(*) FROM users;"

# Очистить
dropdb test_restore

if [ $? -eq 0 ]; then
    echo "✓ Restore test successful: $(date)" | tee -a /var/log/restore_tests.log
else
    echo "✗ Restore test FAILED: $(date)" | tee -a /var/log/restore_tests.log
    # Отправить алерт
    mail -s "Backup restore test failed" admin@example.com < /dev/null
fi
EOF

chmod +x /usr/local/bin/test_restore.sh

# Запускать тест раз в неделю
0 4 * * 0 /usr/local/bin/test_restore.sh

# СТРАТЕГИИ BACKUP

# 1. Простая (для маленьких БД < 100GB)
#    - Ежедневный pg_dump
#    - Хранить 7 дней
#    - Копировать на S3

# 2. Средняя (для БД 100GB - 1TB)
#    - Еженедельный полный pg_basebackup
#    - Ежедневный инкрементальный
#    - Continuous WAL archiving
#    - PITR capability
#    - Хранить 2 недели

# 3. Продакшн (для больших БД > 1TB)
#    - Barman или pgBackRest
#    - Непрерывная репликация
#    - Ежедневные автоматические backups
#    - PITR с 30-дневным окном
#    - Офсайт копии в облаке
#    - Регулярное тестирование восстановления

# DISASTER RECOVERY PLAN

# 1. RPO (Recovery Point Objective) - допустимая потеря данных
#    - 0 минут: Синхронная репликация
#    - < 5 минут: Streaming replication + WAL archiving
#    - < 1 час: Регулярные backups

# 2. RTO (Recovery Time Objective) - время восстановления
#    - < 5 минут: Автоматический failover (pg_auto_failover)
#    - < 30 минут: Ручной failover
#    - < 4 часа: Восстановление из backup

# Документация процедуры восстановления
cat > /root/disaster_recovery.md << 'EOF'
# Disaster Recovery Procedure

## Сценарий 1: Полный отказ primary сервера
1. Проверить состояние standby: `pg_isready -h standby`
2. Промотировать standby: `pg_ctl promote -D /var/lib/postgresql/16/main`
3. Обновить DNS или connection string на новый primary
4. Восстановить старый primary как новый standby

## Сценарий 2: Повреждение данных
1. Определить время повреждения
2. Восстановить из последнего backup перед повреждением
3. PITR до момента перед повреждением
4. Проверить целостность данных
5. Вернуть в продакшн

## Сценарий 3: Случайное удаление данных
1. Начать транзакцию и не коммитить
2. Если закоммичено - PITR до момента перед удалением
3. Восстановить данные в отдельную БД
4. Экспортировать удаленные данные
5. Импортировать обратно в продакшн БД
EOF

# МОНИТОРИНГ BACKUP STATUS

# Скрипт проверки свежести backups
cat > /usr/local/bin/check_backup_age.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/backup"
MAX_AGE_HOURS=25

LATEST_BACKUP=$(find $BACKUP_DIR -name "*.dump" -type f -printf '%T@ %p\n' | sort -n | tail -1 | cut -f2- -d" ")
BACKUP_AGE=$(( ($(date +%s) - $(stat -c %Y "$LATEST_BACKUP")) / 3600 ))

if [ $BACKUP_AGE -gt $MAX_AGE_HOURS ]; then
    echo "WARNING: Latest backup is $BACKUP_AGE hours old!"
    echo "Latest backup: $LATEST_BACKUP" | mail -s "Backup too old!" admin@example.com
    exit 1
else
    echo "OK: Latest backup is $BACKUP_AGE hours old"
    exit 0
fi
EOF

chmod +x /usr/local/bin/check_backup_age.sh

# Проверять каждый час
0 * * * * /usr/local/bin/check_backup_age.sh
```

**Упражнение:**

1. Создайте полный backup с pg_dump и pg_basebackup
2. Восстановите backup на тестовом сервере
3. Настройте continuous archiving для PITR
4. Установите и настройте Barman или pgBackRest
5. Создайте автоматизированные backup скрипты
6. Напишите и протестируйте disaster recovery план
7. Настройте мониторинг свежести backups

---

### Задача 17: Безопасность PostgreSQL

**Цель:** Обеспечить безопасность базы данных

**Практика:**

sql

```sql
-- УПРАВЛЕНИЕ ПОЛЬЗОВАТЕЛЯМИ И РОЛЯМИ

-- Создание ролей (групп)
CREATE ROLE readonly;
CREATE ROLE readwrite;
CREATE ROLE admin_role;

-- Создание пользователей
CREATE USER app_user WITH PASSWORD 'secure_password';
CREATE USER analyst WITH PASSWORD 'analyst_pass';
CREATE USER admin_user WITH PASSWORD 'admin_pass';

-- Назначение ролей пользователям
GRANT readonly TO analyst;
GRANT readwrite TO app_user;
GRANT admin_role TO admin_user;

-- Права для readonly роли
GRANT CONNECT ON DATABASE mydb TO readonly;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;

-- Автоматически давать SELECT на новые таблицы
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT ON TABLES TO readonly;

-- Права для readwrite роли
GRANT CONNECT ON DATABASE mydb TO readwrite;
GRANT USAGE ON SCHEMA public TO readwrite;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO readwrite;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO readwrite;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO readwrite;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT USAGE, SELECT ON SEQUENCES TO readwrite;

-- Суперпользователь (осторожно!)
CREATE USER superuser WITH SUPERUSER PASSWORD 'very_secure_password';

-- Пользователь с правом создания БД
CREATE USER db_creator WITH CREATEDB PASSWORD 'password';

-- Пользователь с правом создания ролей
CREATE USER role_manager WITH CREATEROLE PASSWORD 'password';

-- Ограничение подключений
ALTER USER app_user CONNECTION LIMIT 10;

-- Установка таймаута для пользователя
ALTER USER analyst SET statement_timeout = '30s';

-- Row Level Security (RLS)
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    title VARCHAR(200),
    content TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Включить RLS
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Политика: пользователи видят только свои документы
CREATE POLICY user_documents ON documents
    FOR ALL
    TO PUBLIC
    USING (user_id = current_setting('app.user_id')::INTEGER);

-- Политика для админов (видят все)
CREATE POLICY admin_all ON documents
    FOR ALL
    TO admin_role
    USING (true);

-- Использование RLS
SET app.user_id = '123';
SELECT * FROM documents;  -- Увидит только документы с user_id = 123

-- Более сложная политика
CREATE POLICY department_access ON employees
    FOR SELECT
    USING (
        department = current_setting('app.department')
        OR current_user IN (SELECT username FROM admins)
    );

-- Политики для разных операций
CREATE POLICY select_policy ON sensitive_data
    FOR SELECT
    USING (user_id = current_user::INTEGER OR is_public = true);

CREATE POLICY insert_policy ON sensitive_data
    FOR INSERT
    WITH CHECK (user_id = current_user::INTEGER);

CREATE POLICY update_policy ON sensitive_data
    FOR UPDATE
    USING (user_id = current_user::INTEGER)
    WITH CHECK (user_id = current_user::INTEGER);

-- ШИФРОВАНИЕ

-- Установка pgcrypto
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Хеширование паролей
CREATE TABLE users_secure (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100) UNIQUE,
    password_hash TEXT NOT NULL
);

-- Вставка с хешированием пароля
INSERT INTO users_secure (username, password_hash)
VALUES ('john', crypt('user_password', gen_salt('bf', 8)));

-- Проверка пароля
SELECT * FROM users_secure
WHERE username = 'john'
  AND password_hash = crypt('user_password', password_hash);

-- Шифрование данных
CREATE TABLE credit_cards (
    id SERIAL PRIMARY KEY,
    user_id INTEGER,
    card_number_encrypted BYTEA
);

-- Вставка зашифрованных данных
INSERT INTO credit_cards (user_id, card_number_encrypted)
VALUES (1, pgp_sym_encrypt('4111111111111111', 'encryption_key'));

-- Расшифровка
SELECT 
    id,
    user_id,
    pgp_sym_decrypt(card_number_encrypted, 'encryption_key') AS card_number
FROM credit_cards;

-- Асимметричное шифрование (публичный/приватный ключ)
-- Генерация ключей вне PostgreSQL
-- openssl genrsa -out private_key.pem 2048
-- openssl rsa -in private_key.pem -pubout -out public_key.pem

-- Чтение ключей в PostgreSQL
\set private_key `cat private_key.pem`
\set public_key `cat public_key.pem`

-- Шифрование с публичным ключом
INSERT INTO encrypted_data (data)
VALUES (pgp_pub_encrypt('sensitive data', :'public_key'));

-- Расшифровка с приватным ключом
SELECT pgp_pub_decrypt(data, :'private_key') FROM encrypted_data;

-- SSL/TLS ПОДКЛЮЧЕНИЯ

-- В postgresql.conf
# ssl = on
# ssl_cert_file = 'server.crt'
# ssl_key_file = 'server.key'
# ssl_ca_file = 'root.crt'

-- В pg_hba.conf требовать SSL
# hostssl    all    all    0.0.0.0/0    md5

-- Генерация самоподписанного сертификата
# openssl req -new -x509 -days 365 -nodes -text \
#   -out server.crt -keyout server.key \
#   -subj "/CN=postgres.example.com"
# chmod 600 server.key
# chown postgres:postgres server.key server.crt

-- Подключение с SSL
# psql "host=postgres.example.com dbname=mydb user=myuser sslmode=require"

-- АУДИТ И ЛОГИРОВАНИЕ

-- В postgresql.conf
# log_connections = on
# log_disconnections = on
# log_duration = on
# log_statement = 'all'  # или 'ddl' или 'mod'
# log_line_prefix = '%t [%p]: user=%u,db=%d,app=%a,client=%h '

-- Логирование медленных запросов
# log_min_duration_statement = 1000  # ms

-- Расширение для аудита
CREATE EXTENSION IF NOT EXISTS pgaudit;

-- Настройка pgaudit
ALTER SYSTEM SET pgaudit.log = 'write, ddl';
ALTER SYSTEM SET pgaudit.log_catalog = off;
SELECT pg_reload_conf();

-- Аудит конкретной роли
ALTER ROLE app_user SET pgaudit.log = 'read, write';

-- Просмотр логов
SELECT * FROM pg_read_file('postgresql-2024-01-15.log');

-- ЗАЩИТА ОТ SQL INJECTION

-- ПЛОХО (уязвимо к SQL injection)
-- query = "SELECT * FROM users WHERE username = '" + user_input + "';"

-- ХОРОШО (параметризованный запрос)
-- В приложении:
-- cursor.execute("SELECT * FROM users WHERE username = %s", (user_input,))

-- Подготовленные запросы в PostgreSQL
PREPARE get_user AS
SELECT * FROM users WHERE username = $1;

EXECUTE get_user('john');

DEALLOCATE get_user;

-- ОГРАНИЧЕНИЕ ДОСТУПА

-- Запретить DROP DATABASE
REVOKE CREATE ON DATABASE mydb FROM PUBLIC;

-- Запретить CREATE TABLE обычным пользователям
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
GRANT CREATE ON SCHEMA public TO admin_role;

-- Ограничить доступ к системным каталогам
REVOKE ALL ON pg_catalog.pg_authid FROM PUBLIC;

-- Отозвать права
REVOKE SELECT ON sensitive_table FROM analyst;

-- НАСТРОЙКА pg_hba.conf для безопасности

# Локальные подключения
local   all   postgres   peer
local   all   all        md5

# IPv4 локальные подключения
host    all   all   127.0.0.1/32   scram-sha-256

# Конкретные пользователи с конкретных IP
host    mydb   app_user   192.168.1.100/32   scram-sha-256

# Отклонить подключения извне
host    all   all   0.0.0.0/0   reject

# Разрешить только с доверенной сети
host    all   all   10.0.0.0/8   scram-sha-256

# SCRAM-SHA-256 аутентификация (безопаснее MD5)
# В postgresql.conf:
# password_encryption = scram-sha-256

-- Пересоздать пароли для существующих пользователей
ALTER USER app_user WITH PASSWORD 'new_password';

-- МОНИТОРИНГ БЕЗОПАСНОСТИ

-- Проверка прав пользователей
SELECT 
    grantee,
    table_schema,
    table_name,
    privilege_type
FROM information_schema.table_privileges
WHERE grantee = 'app_user';

-- Список всех ролей и их атрибутов
SELECT 
    rolname,
    rolsuper,
    rolinherit,
    rolcreaterole,
    rolcreatedb,
    rolcanlogin,
    rolconnlimit
FROM pg_roles
ORDER BY rolname;

-- Список членства в ролях
SELECT 
    r.rolname AS role,
    m.rolname AS member
FROM pg_auth_members am
JOIN pg_roles r ON r.oid = am.roleid
JOIN pg_roles m ON m.oid = am.member;

-- Активные подключения и их источники
SELECT 
    usename,
    application_name,
    client_addr,
    client_port,
    state,
    query
FROM pg_stat_activity
WHERE usename != 'postgres';

-- Неудачные попытки подключения (из логов)
-- grep "FATAL:  password authentication failed" /var/log/postgresql/postgresql-*.log

-- BEST PRACTICES

-- 1. Никогда не использовать postgres суперпользователя для приложений
-- 2. Использовать минимальные необходимые права (principle of least privilege)
-- 3. Регулярно ротировать пароли
-- 4. Использовать SCRAM-SHA-256 вместо MD5
-- 5. Включить SSL для всех подключений
-- 6. Логировать все DDL операции
-- 7. Регулярно проверять права доступа
-- 8. Использовать RLS для мультитенантных приложений
-- 9. Шифровать чувствительные данные
-- 10. Ограничивать подключения по IP в pg_hba.conf

-- CHECKLIST безопасности
CREATE TABLE security_checklist (
    item TEXT,
    status BOOLEAN DEFAULT FALSE
);

INSERT INTO security_checklist (item) VALUES
    ('Удален или защищен postgres суперпользователь'),
    ('Созданы роли с минимальными правами'),
    ('Включен SSL'),
    ('pg_hba.conf настроен корректно'),
    ('Используется SCRAM-SHA-256'),
    ('Включено логирование подключений'),
    ('Настроен аудит pgaudit'),
    ('Пароли захешированы'),
    ('Чувствительные данные зашифрованы'),
    ('RLS включен где необходимо'),
    ('Firewall настроен'),
    ('Регулярные security аудиты'),
    ('Мониторинг подозрительной активности'),
    ('Backups зашифрованы');

SELECT * FROM security_checklist WHERE NOT status;
```

**Упражнение:**

1. Создайте роли readonly, readwrite, admin с соответствующими правами
2. Настройте Row Level Security для мультитенантной таблицы
3. Реализуйте шифрование чувствительных данных
4. Настройте SSL подключения
5. Настройте pgaudit для аудита операций
6. Сконфигурируйте pg_hba.conf для безопасного доступа
7. Создайте скрипт для регулярной проверки безопасности

---

### Задача 18: Расширения PostgreSQL

**Цель:** Использовать мощные расширения для дополнительной функциональности

**Практика:**

sql

```sql
-- Просмотр доступных расширений
SELECT * FROM pg_available_extensions ORDER BY name;

-- Установленные расширения
SELECT * FROM pg_extension;

-- 1. PostGIS - геопространственные данные

CREATE EXTENSION postgis;

-- Создание таблицы с геоданными
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    coordinates GEOMETRY(POINT, 4326)  -- 4326 = WGS84
);

-- Вставка точек
INSERT INTO locations (name, coordinates) VALUES
    ('Office', ST_GeomFromText('POINT(-73.935242 40.730610)', 4326)),
    ('Home', ST_GeomFromText('POINT(-73.925242 40.735610)', 4326)),
    ('Store', ST_GeomFromText('POINT(-73.945242 40.725610)', 4326));

-- Расстояние между точками (в метрах)
SELECT 
    l1.name AS from_location,
    l2.name AS to_location,
    ST_Distance(
        ST_Transform(l1.coordinates, 3857),
        ST_Transform(l2.coordinates, 3857)
    ) AS distance_meters
FROM locations l1, locations l2
WHERE l1.id != l2.id;

-- Поиск точек в радиусе (1 км)
SELECT name, coordinates
FROM locations
WHERE ST_DWithin(
    ST_Transform(coordinates, 3857),
    ST_Transform(ST_GeomFromText('POINT(-73.935242 40.730610)', 4326), 3857),
    1000
);

-- Создание индекса для геоданных
CREATE INDEX idx_locations_coordinates 
ON locations USING GIST (coordinates);

-- 2. pg_trgm - fuzzy поиск текста

CREATE EXTENSION pg_trgm;

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    description TEXT
);

-- Индекс для fuzzy поиска
CREATE INDEX idx_products_name_trgm ON products USING GIN (name gin_trgm_ops);

-- Пои
```

Continue

16 нояб.

sql

```sql
-- Поиск по сходству
SELECT name, similarity(name, 'PostgreSQL') AS sim
FROM products
WHERE similarity(name, 'PostgreSQL') > 0.3
ORDER BY sim DESC;

-- Поиск с опечатками
SELECT * FROM products
WHERE name % 'PostgreSQL';  -- % оператор для similarity

-- Расстояние Левенштейна
SELECT name, levenshtein(name, 'PostgreSQL') AS distance
FROM products
ORDER BY distance
LIMIT 5;

-- 3. hstore - key-value хранилище

CREATE EXTENSION hstore;

CREATE TABLE user_settings (
    user_id INTEGER PRIMARY KEY,
    settings HSTORE
);

-- Вставка данных
INSERT INTO user_settings (user_id, settings) VALUES
    (1, 'theme => dark, language => en, notifications => true'),
    (2, 'theme => light, language => ru, notifications => false');

-- Доступ к значениям
SELECT user_id, settings->'theme' AS theme
FROM user_settings;

-- Обновление конкретного ключа
UPDATE user_settings
SET settings = settings || 'theme => blue'::hstore
WHERE user_id = 1;

-- Поиск по ключу
SELECT * FROM user_settings
WHERE settings ? 'notifications';

-- Поиск по значению
SELECT * FROM user_settings
WHERE settings @> 'theme => dark';

-- Индекс для hstore
CREATE INDEX idx_settings ON user_settings USING GIN (settings);

-- 4. uuid-ossp - генерация UUID

CREATE EXTENSION "uuid-ossp";

CREATE TABLE orders_uuid (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    customer_id INTEGER,
    total DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT NOW()
);

INSERT INTO orders_uuid (customer_id, total) VALUES (1, 99.99);

SELECT * FROM orders_uuid;

-- Разные версии UUID
SELECT uuid_generate_v1();  -- Основано на времени
SELECT uuid_generate_v4();  -- Случайный

-- 5. pg_stat_statements - анализ производительности

CREATE EXTENSION pg_stat_statements;

-- В postgresql.conf добавить:
-- shared_preload_libraries = 'pg_stat_statements'
-- pg_stat_statements.track = all

-- Перезапустить PostgreSQL

-- Топ медленных запросов
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time,
    stddev_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Топ по общему времени выполнения
SELECT 
    query,
    calls,
    total_exec_time,
    min_exec_time,
    max_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Сброс статистики
SELECT pg_stat_statements_reset();

-- 6. ltree - иерархические данные

CREATE EXTENSION ltree;

CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    path LTREE NOT NULL,
    name VARCHAR(100)
);

-- Вставка иерархии
INSERT INTO categories (path, name) VALUES
    ('electronics', 'Electronics'),
    ('electronics.computers', 'Computers'),
    ('electronics.computers.laptops', 'Laptops'),
    ('electronics.computers.desktops', 'Desktops'),
    ('electronics.phones', 'Phones'),
    ('electronics.phones.smartphones', 'Smartphones'),
    ('home', 'Home'),
    ('home.furniture', 'Furniture'),
    ('home.furniture.tables', 'Tables');

-- Найти все подкатегории
SELECT name, path FROM categories
WHERE path <@ 'electronics.computers';

-- Найти всех предков
SELECT name, path FROM categories
WHERE path @> 'electronics.computers.laptops';

-- Глубина в дереве
SELECT 
    name,
    path,
    nlevel(path) AS level
FROM categories
ORDER BY path;

-- Индекс для ltree
CREATE INDEX idx_categories_path ON categories USING GIST (path);

-- 7. pg_cron - планировщик задач

CREATE EXTENSION pg_cron;

-- Запускать VACUUM каждую ночь в 3:00
SELECT cron.schedule('nightly-vacuum', '0 3 * * *', 'VACUUM ANALYZE');

-- Очистка старых логов каждый день
SELECT cron.schedule(
    'cleanup-logs',
    '0 2 * * *',
    'DELETE FROM logs WHERE created_at < NOW() - INTERVAL ''30 days'''
);

-- Список запланированных задач
SELECT * FROM cron.job;

-- Удалить задачу
SELECT cron.unschedule('cleanup-logs');

-- История выполнения
SELECT * FROM cron.job_run_details
ORDER BY start_time DESC
LIMIT 10;

-- 8. tablefunc - pivot таблицы и crosstab

CREATE EXTENSION tablefunc;

-- Обычная таблица
CREATE TABLE sales_pivot (
    year INTEGER,
    quarter INTEGER,
    amount DECIMAL(10,2)
);

INSERT INTO sales_pivot VALUES
    (2023, 1, 10000),
    (2023, 2, 12000),
    (2023, 3, 15000),
    (2023, 4, 18000),
    (2024, 1, 11000),
    (2024, 2, 13000);

-- Pivot (crosstab)
SELECT * FROM crosstab(
    'SELECT year, quarter, amount FROM sales_pivot ORDER BY 1,2',
    'SELECT generate_series(1,4)'
) AS ct(year INTEGER, q1 DECIMAL, q2 DECIMAL, q3 DECIMAL, q4 DECIMAL);

-- 9. pg_trgm + unaccent - поиск с игнорированием диакритики

CREATE EXTENSION unaccent;

-- Поиск без учета акцентов
SELECT unaccent('Café') = unaccent('Cafe');  -- true

-- Создать индекс с unaccent
CREATE INDEX idx_products_name_unaccent 
ON products USING GIN (unaccent(name) gin_trgm_ops);

-- Поиск
SELECT * FROM products
WHERE unaccent(name) ILIKE unaccent('%cafe%');

-- 10. pgcrypto - криптография (уже рассматривали)

CREATE EXTENSION pgcrypto;

-- UUID v4
SELECT gen_random_uuid();

-- Хеширование
SELECT encode(digest('password', 'sha256'), 'hex');

-- 11. postgres_fdw - Foreign Data Wrapper (подключение к другим БД)

CREATE EXTENSION postgres_fdw;

-- Создать сервер
CREATE SERVER remote_db
    FOREIGN DATA WRAPPER postgres_fdw
    OPTIONS (host 'remote.example.com', port '5432', dbname 'remotedb');

-- Создать user mapping
CREATE USER MAPPING FOR CURRENT_USER
    SERVER remote_db
    OPTIONS (user 'remote_user', password 'remote_password');

-- Импортировать схему
IMPORT FOREIGN SCHEMA public
    FROM SERVER remote_db
    INTO public;

-- Или создать foreign таблицу вручную
CREATE FOREIGN TABLE remote_users (
    id INTEGER,
    username VARCHAR(100),
    email VARCHAR(100)
)
SERVER remote_db
OPTIONS (schema_name 'public', table_name 'users');

-- Теперь можно делать JOIN между локальными и удаленными таблицами
SELECT 
    l.order_id,
    r.username
FROM local_orders l
JOIN remote_users r ON l.user_id = r.id;

-- 12. pg_partman - автоматическое управление партициями

CREATE EXTENSION pg_partman;

-- Создать партиционированную таблицу
CREATE TABLE logs_managed (
    id BIGSERIAL,
    log_date TIMESTAMP NOT NULL,
    message TEXT
) PARTITION BY RANGE (log_date);

-- Настроить pg_partman
SELECT partman.create_parent(
    p_parent_table := 'public.logs_managed',
    p_control := 'log_date',
    p_type := 'native',
    p_interval := 'daily',
    p_premake := 7
);

-- pg_partman автоматически создаст партиции и будет управлять ими

-- Обновление партиций (запустить в cron)
SELECT partman.run_maintenance();

-- 13. TimescaleDB - для временных рядов

-- Установка TimescaleDB (требует отдельной установки)
CREATE EXTENSION timescaledb;

-- Создать обычную таблицу
CREATE TABLE sensor_data (
    time TIMESTAMPTZ NOT NULL,
    sensor_id INTEGER,
    temperature DOUBLE PRECISION,
    humidity DOUBLE PRECISION
);

-- Преобразовать в hypertable (основная фича TimescaleDB)
SELECT create_hypertable('sensor_data', 'time');

-- Вставка данных (как обычно)
INSERT INTO sensor_data VALUES
    (NOW(), 1, 22.5, 45.0),
    (NOW(), 2, 23.1, 46.5);

-- TimescaleDB автоматически партиционирует по времени

-- Агрегация по временным интервалам
SELECT 
    time_bucket('1 hour', time) AS hour,
    sensor_id,
    AVG(temperature) AS avg_temp
FROM sensor_data
WHERE time > NOW() - INTERVAL '24 hours'
GROUP BY hour, sensor_id
ORDER BY hour DESC;

-- Continuous aggregates (материализованные представления)
CREATE MATERIALIZED VIEW sensor_data_hourly
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 hour', time) AS hour,
    sensor_id,
    AVG(temperature) AS avg_temp,
    MAX(temperature) AS max_temp,
    MIN(temperature) AS min_temp
FROM sensor_data
GROUP BY hour, sensor_id;

-- Политика обновления
SELECT add_continuous_aggregate_policy('sensor_data_hourly',
    start_offset => INTERVAL '3 hours',
    end_offset => INTERVAL '1 hour',
    schedule_interval => INTERVAL '1 hour');

-- Retention policy (автоматическое удаление старых данных)
SELECT add_retention_policy('sensor_data', INTERVAL '30 days');

-- Compression (сжатие старых данных)
ALTER TABLE sensor_data SET (
    timescaledb.compress,
    timescaledb.compress_segmentby = 'sensor_id'
);

SELECT add_compression_policy('sensor_data', INTERVAL '7 days');

-- 14. pg_repack - онлайн дефрагментация

-- Установка pg_repack
-- apt install postgresql-16-repack

-- Использование (из командной строки)
-- pg_repack -d mydb -t users

-- 15. pg_hint_plan - принудительные hints для планировщика

CREATE EXTENSION pg_hint_plan;

-- Использование hints в комментариях
/*+ SeqScan(users) */
SELECT * FROM users WHERE id = 1;

/*+ IndexScan(users idx_users_email) */
SELECT * FROM users WHERE email = 'test@example.com';

/*+ HashJoin(users orders) */
SELECT * FROM users u JOIN orders o ON u.id = o.user_id;

-- СОЗДАНИЕ СВОЕГО РАСШИРЕНИЯ

-- Простое расширение для примера
-- Файл: my_extension--1.0.sql
CREATE FUNCTION hello(name TEXT) RETURNS TEXT AS $$
BEGIN
    RETURN 'Hello, ' || name || '!';
END;
$$ LANGUAGE plpgsql;

-- Файл: my_extension.control
-- comment = 'My custom extension'
-- default_version = '1.0'
-- relocatable = true

-- Установка
-- CREATE EXTENSION my_extension;

-- Использование
-- SELECT hello('World');

-- МОНИТОРИНГ РАСШИРЕНИЙ

-- Размер расширений
SELECT 
    e.extname,
    pg_size_pretty(sum(pg_total_relation_size(c.oid))) AS size
FROM pg_extension e
JOIN pg_depend d ON d.refobjid = e.oid
JOIN pg_class c ON c.oid = d.objid
WHERE d.deptype = 'e'
GROUP BY e.extname
ORDER BY sum(pg_total_relation_size(c.oid)) DESC;

-- Версии установленных расширений
SELECT 
    extname,
    extversion,
    extrelocatable
FROM pg_extension
ORDER BY extname;

-- Обновление расширения
ALTER EXTENSION postgis UPDATE TO '3.4.0';

-- Удаление расширения
DROP EXTENSION IF EXISTS pgcrypto CASCADE;

-- ПОПУЛЯРНЫЕ РАСШИРЕНИЯ ДЛЯ РАЗНЫХ ЗАДАЧ

-- Геоданные: postgis
-- Полнотекстовый поиск: pg_trgm, unaccent
-- Временные ряды: timescaledb
-- Аудит: pgaudit
-- Репликация: pglogical
-- Производительность: pg_stat_statements, pg_hint_plan
-- Криптография: pgcrypto
-- HTTP клиент: http
-- Очереди: pgq, pg_amqp
-- GraphQL: pg_graphql
```

**Упражнение:**

1. Установите PostGIS и создайте таблицу с геоданными
2. Настройте pg_trgm для fuzzy поиска
3. Используйте pg_stat_statements для анализа запросов
4. Реализуйте иерархию с ltree
5. Настройте pg_cron для автоматических задач
6. Попробуйте postgres_fdw для подключения к удаленной БД
7. Если работаете с временными рядами - установите TimescaleDB

---

## 📊 МОНИТОРИНГ И ДИАГНОСТИКА

### Задача 19: Комплексный мониторинг PostgreSQL

**Цель:** Настроить полный мониторинг БД

**Практика:**

sql

```sql
-- СИСТЕМНЫЕ ПРЕДСТАВЛЕНИЯ ДЛЯ МОНИТОРИНГА

-- 1. pg_stat_activity - активные подключения и запросы
SELECT 
    pid,
    usename,
    application_name,
    client_addr,
    state,
    query,
    query_start,
    state_change,
    NOW() - query_start AS duration
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY duration DESC;

-- Подключения по состоянию
SELECT 
    state,
    COUNT(*) as connections
FROM pg_stat_activity
GROUP BY state;

-- Подключения по пользователям
SELECT 
    usename,
    COUNT(*) as connections,
    MAX(NOW() - query_start) as max_duration
FROM pg_stat_activity
GROUP BY usename;

-- Убить долгие idle in transaction
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE state = 'idle in transaction'
  AND NOW() - state_change > INTERVAL '10 minutes';

-- 2. pg_stat_database - статистика БД
SELECT 
    datname,
    numbackends as connections,
    xact_commit,
    xact_rollback,
    blks_read,
    blks_hit,
    tup_returned,
    tup_fetched,
    tup_inserted,
    tup_updated,
    tup_deleted,
    conflicts,
    temp_files,
    temp_bytes,
    deadlocks
FROM pg_stat_database
WHERE datname = current_database();

-- Cache hit ratio
SELECT 
    datname,
    blks_hit::FLOAT / NULLIF((blks_read + blks_hit), 0) AS cache_hit_ratio
FROM pg_stat_database
WHERE datname = current_database();

-- 3. pg_stat_user_tables - статистика таблиц
SELECT 
    schemaname,
    relname,
    seq_scan,
    seq_tup_read,
    idx_scan,
    idx_tup_fetch,
    n_tup_ins,
    n_tup_upd,
    n_tup_del,
    n_live_tup,
    n_dead_tup,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
ORDER BY seq_scan DESC;

-- Таблицы с большим количеством мертвых строк
SELECT 
    schemaname,
    relname,
    n_live_tup,
    n_dead_tup,
    ROUND(n_dead_tup::FLOAT / NULLIF(n_live_tup, 0) * 100, 2) AS dead_ratio
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY dead_ratio DESC;

-- 4. pg_stat_user_indexes - использование индексов
SELECT 
    schemaname,
    relname,
    indexrelname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;

-- Неиспользуемые индексы
SELECT 
    schemaname,
    relname,
    indexrelname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelname NOT LIKE '%_pkey'
ORDER BY pg_relation_size(indexrelid) DESC;

-- 5. pg_statio_user_tables - I/O статистика
SELECT 
    relname,
    heap_blks_read,
    heap_blks_hit,
    ROUND(heap_blks_hit::NUMERIC / NULLIF((heap_blks_hit + heap_blks_read), 0) * 100, 2) AS cache_hit_ratio,
    idx_blks_read,
    idx_blks_hit
FROM pg_statio_user_tables
ORDER BY heap_blks_read DESC;

-- 6. pg_locks - блокировки
SELECT 
    pg_stat_activity.pid,
    pg_stat_activity.usename,
    pg_locks.mode,
    pg_locks.locktype,
    pg_locks.relation::regclass,
    pg_stat_activity.query,
    pg_stat_activity.state
FROM pg_locks
JOIN pg_stat_activity ON pg_locks.pid = pg_stat_activity.pid
WHERE NOT pg_locks.granted;

-- Ждущие блокировки с блокирующими запросами
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename  AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;

-- 7. pg_stat_replication - статистика репликации
SELECT 
    application_name,
    client_addr,
    state,
    sync_state,
    replay_lag,
    pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_bytes,
    pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn)) AS lag_size
FROM pg_stat_replication;

-- 8. Размеры объектов
-- Размер таблиц
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size,
    pg_size_pretty(pg_indexes_size(schemaname||'.'||tablename)) AS indexes_size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;

-- Размер базы данных
SELECT 
    datname,
    pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;

-- Рост базы данных (требует создания таблицы для хранения истории)
CREATE TABLE IF NOT EXISTS db_size_history (
    recorded_at TIMESTAMP DEFAULT NOW(),
    datname TEXT,
    size_bytes BIGINT
);

-- Записывать каждый день
INSERT INTO db_size_history (datname, size_bytes)
SELECT datname, pg_database_size(datname)
FROM pg_database;

-- Рост за последний месяц
WITH current_sizes AS (
    SELECT datname, size_bytes, recorded_at
    FROM db_size_history
    WHERE recorded_at = (SELECT MAX(recorded_at) FROM db_size_history)
),
old_sizes AS (
    SELECT datname, size_bytes
    FROM db_size_history
    WHERE recorded_at = (SELECT MAX(recorded_at) 
                        FROM db_size_history 
                        WHERE recorded_at < NOW() - INTERVAL '30 days')
)
SELECT 
    c.datname,
    pg_size_pretty(c.size_bytes) AS current_size,
    pg_size_pretty(c.size_bytes - COALESCE(o.size_bytes, 0)) AS growth,
    ROUND((c.size_bytes - COALESCE(o.size_bytes, 0))::NUMERIC / NULLIF(o.size_bytes, 0) * 100, 2) AS growth_percent
FROM current_sizes c
LEFT JOIN old_sizes o ON c.datname = o.datname;

-- 9. Bloat (раздувание таблиц и индексов)
-- Приблизительная оценка bloat
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size,
    ROUND(100 * pg_relation_size(schemaname||'.'||tablename) / 
          NULLIF(pg_total_relation_size(schemaname||'.'||tablename), 0), 2) AS table_pct,
    ROUND(100 * pg_indexes_size(schemaname||'.'||tablename) / 
          NULLIF(pg_total_relation_size(schemaname||'.'||tablename), 0), 2) AS indexes_pct
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;

-- PROMETHEUS + GRAFANA МОНИТОРИНГ

-- Установка postgres_exporter
-- wget https://github.com/prometheus-community/postgres_exporter/releases/download/v0.15.0/postgres_exporter-0.15.0.linux-amd64.tar.gz
-- tar xvf postgres_exporter-0.15.0.linux-amd64.tar.gz
-- cd postgres_exporter-0.15.0.linux-amd64

-- Создать пользователя для мониторинга
```

sql

```sql
CREATE USER postgres_exporter WITH PASSWORD 'exporter_password';
GRANT pg_monitor TO postgres_exporter;
```

bash

```bash
-- Настроить CONNECTION STRING
export DATA_SOURCE_NAME="postgresql://postgres_exporter:exporter_password@localhost:5432/postgres?sslmode=disable"

-- Запустить exporter
./postgres_exporter

-- Prometheus scrape config (prometheus.yml)
# scrape_configs:
#   - job_name: 'postgresql'
#     static_configs:
#       - targets: ['localhost:9187']

-- Grafana dashboards:
-- Dashboard ID: 9628 (PostgreSQL Database)
-- Dashboard ID: 12273 (PostgreSQL Overview)

-- СКРИПТ ДЛЯ ЕЖЕДНЕВНОГО HEALTH CHECK
```

bash

```bash
#!/bin/bash
# /usr/local/bin/pg_health_check.sh

PSQL="psql -U postgres -d postgres -t -A"
EMAIL="admin@example.com"
REPORT="/tmp/pg_health_report.txt"

echo "PostgreSQL Health Check - $(date)" > $REPORT
echo "======================================" >> $REPORT
echo "" >> $REPORT

# Проверка подключения
if ! $PSQL -c "SELECT 1" > /dev/null 2>&1; then
    echo "CRITICAL: Cannot connect to PostgreSQL!" >> $REPORT
    mail -s "PostgreSQL Health Check CRITICAL" $EMAIL < $REPORT
    exit 1
fi

# Проверка репликации
LAG=$($PSQL -c "SELECT EXTRACT(EPOCH FROM (NOW() - pg_last_xact_replay_timestamp()));" | head -1)
if [ -n "$LAG" ] && [ $(echo "$LAG > 60" | bc) -eq 1 ]; then
    echo "WARNING: Replication lag is $LAG seconds" >> $REPORT
fi

# Проверка bloat
BLOAT_TABLES=$($PSQL -c "
    SELECT COUNT(*)
    FROM pg_stat_user_tables
    WHERE n_dead_tup::FLOAT / NULLIF(n_live_tup, 0) > 0.2
    AND n_dead_tup > 10000;
")
if [ "$BLOAT_TABLES" -gt 0 ]; then
    echo "WARNING: $BLOAT_TABLES tables have high bloat (>20%)" >> $REPORT
fi

# Проверка connections
CONN_PERCENT=$($PSQL -c "
    SELECT ROUND(COUNT(*)::NUMERIC / 
                 (SELECT setting FROM pg_settings WHERE name = 'max_connections')::NUMERIC * 100)
    FROM pg_stat_activity;
")
if [ "$CONN_PERCENT" -gt 80 ]; then
    echo "WARNING: Using $CONN_PERCENT% of available connections" >> $REPORT
fi

# Проверка cache hit ratio
CACHE_HIT=$($PSQL -c "
    SELECT ROUND(SUM(blks_hit)::NUMERIC / NULLIF(SUM(blks_hit + blks_read), 0) * 100, 2)
    FROM pg_stat_database;
")
if [ $(echo "$CACHE_HIT < 95" | bc) -eq 1 ]; then
    echo "WARNING: Cache hit ratio is only $CACHE_HIT% (should be >95%)" >> $REPORT
fi

# Проверка disk space
DISK_USAGE=$(df -h /var/lib/postgresql | tail -1 | awk '{print $5}' | sed 's/%//')
if [ "$DISK_USAGE" -gt 80 ]; then
    echo "WARNING: Disk usage is $DISK_USAGE%" >> $REPORT
fi

# Проверка long running queries
LONG_QUERIES=$($PSQL -c "
    SELECT COUNT(*)
    FROM pg_stat_activity
    WHERE state != 'idle'
    AND NOW() - query_start > INTERVAL '5 minutes';
")
if [ "$LONG_QUERIES" -gt 0 ]; then
    echo "WARNING: $LONG_QUERIES queries running >5 minutes" >> $REPORT
fi

# Если есть проблемы - отправить email
if grep -q "WARNING\|CRITICAL" $REPORT; then
    mail -s "PostgreSQL Health Check - Issues Found" $EMAIL < $REPORT
else
    echo "OK: All checks passed" >> $REPORT
fi

# Сохранить в лог
cat $REPORT >> /var/log/postgresql/health_checks.log
```

bash

```bash
# Добавить в cron
crontab -e
# 0 9 * * * /usr/local/bin/pg_health_check.sh
```

**Упражнение:**

1. Настройте мониторинг активных подключений
2. Создайте dashboard для отслеживания производительности
3. Настройте алерты для критических метрик
4. Реализуйте скрипт health check
5. Настройте Prometheus + Grafana для визуализации
6. Создайте автоматические отчеты о состоянии БД

---

## 📚 ФИНАЛЬНЫЙ ПРОЕКТ

### Задача 20: Полнофункциональная база данных для e-commerce

**Цель:** Применить все изученные навыки

**Требования:**

1. Схема базы данных с нормализацией
2. Индексы для оптимизации
3. Партиционирование для больших таблиц
4. Репликация для отказоустойчивости
5. Backup стратегия
6. Мониторинг и алерты
7. Безопасность (роли, RLS, шифрование)
8. Full-text search для продуктов
9. JSON для гибких атрибутов
10. Stored procedures для бизнес-логики

**Схема:**

sql

```sql
-- 1. Создание схемы

CREATE SCHEMA ecommerce;
SET search_path TO ecommerce, public;

-- 2. Пользователи и аутентификация
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    last_login TIMESTAMPTZ,
    is_active BOOLEAN DEFAULT true,
    email_verified BOOLEAN DEFAULT false
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at DESC);

-- 3. Адреса доставки
CREATE TABLE addresses (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(
```

Continue

16 нояб.

sql

```sql
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,
    address_type VARCHAR(20) CHECK (address_type IN ('shipping', 'billing')),
    street VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country VARCHAR(100) NOT NULL,
    is_default BOOLEAN DEFAULT false,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_addresses_user_id ON addresses(user_id);

-- 4. Категории (иерархические)
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    parent_id INTEGER REFERENCES categories(id),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    path LTREE,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_categories_path ON categories USING GIST (path);
CREATE INDEX idx_categories_slug ON categories(slug);
CREATE INDEX idx_categories_parent ON categories(parent_id);

-- 5. Продукты
CREATE TABLE products (
    id BIGSERIAL PRIMARY KEY,
    category_id INTEGER REFERENCES categories(id),
    sku VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    description TEXT,
    long_description TEXT,
    price DECIMAL(10,2) NOT NULL CHECK (price >= 0),
    compare_price DECIMAL(10,2) CHECK (compare_price >= price),
    cost_price DECIMAL(10,2),
    stock INTEGER DEFAULT 0 CHECK (stock >= 0),
    weight DECIMAL(10,2),
    dimensions JSONB, -- {width, height, depth}
    attributes JSONB, -- Гибкие атрибуты
    images JSONB, -- Массив URL изображений
    search_vector TSVECTOR,
    is_active BOOLEAN DEFAULT true,
    is_featured BOOLEAN DEFAULT false,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Индексы для продуктов
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_sku ON products(sku);
CREATE INDEX idx_products_slug ON products(slug);
CREATE INDEX idx_products_price ON products(price);
CREATE INDEX idx_products_search ON products USING GIN (search_vector);
CREATE INDEX idx_products_attributes ON products USING GIN (attributes);
CREATE INDEX idx_products_active ON products(is_active) WHERE is_active = true;

-- Триггер для автообновления search_vector
CREATE OR REPLACE FUNCTION products_search_trigger() RETURNS trigger AS $$
BEGIN
    NEW.search_vector :=
        setweight(to_tsvector('english', coalesce(NEW.name, '')), 'A') ||
        setweight(to_tsvector('english', coalesce(NEW.description, '')), 'B') ||
        setweight(to_tsvector('english', coalesce(NEW.sku, '')), 'C');
    NEW.updated_at := NOW();
    RETURN NEW;
END
$$ LANGUAGE plpgsql;

CREATE TRIGGER tsvector_update BEFORE INSERT OR UPDATE
ON products FOR EACH ROW EXECUTE FUNCTION products_search_trigger();

-- 6. Инвентарь (партиционированный по дате)
CREATE TABLE inventory_log (
    id BIGSERIAL,
    product_id BIGINT REFERENCES products(id),
    change_type VARCHAR(20) CHECK (change_type IN ('restock', 'sale', 'return', 'adjustment')),
    quantity INTEGER NOT NULL,
    old_stock INTEGER,
    new_stock INTEGER,
    notes TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    created_by BIGINT REFERENCES users(id)
) PARTITION BY RANGE (created_at);

-- Создание партиций на 12 месяцев
DO $$
DECLARE
    start_date DATE;
    end_date DATE;
    partition_name TEXT;
BEGIN
    FOR i IN 0..11 LOOP
        start_date := DATE_TRUNC('month', CURRENT_DATE) + (i || ' months')::INTERVAL;
        end_date := start_date + INTERVAL '1 month';
        partition_name := 'inventory_log_' || TO_CHAR(start_date, 'YYYY_MM');
        
        EXECUTE FORMAT(
            'CREATE TABLE IF NOT EXISTS %I PARTITION OF inventory_log
             FOR VALUES FROM (%L) TO (%L)',
            partition_name, start_date, end_date
        );
    END LOOP;
END $$;

CREATE INDEX ON inventory_log(product_id, created_at DESC);

-- 7. Заказы (партиционированный)
CREATE TABLE orders (
    id BIGSERIAL,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    user_id BIGINT REFERENCES users(id),
    status VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled', 'refunded')),
    subtotal DECIMAL(10,2) NOT NULL,
    tax DECIMAL(10,2) DEFAULT 0,
    shipping_cost DECIMAL(10,2) DEFAULT 0,
    discount DECIMAL(10,2) DEFAULT 0,
    total DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    payment_method VARCHAR(50),
    payment_status VARCHAR(20) DEFAULT 'pending',
    shipping_address_id BIGINT REFERENCES addresses(id),
    billing_address_id BIGINT REFERENCES addresses(id),
    notes TEXT,
    metadata JSONB, -- Дополнительные данные
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    shipped_at TIMESTAMPTZ,
    delivered_at TIMESTAMPTZ
) PARTITION BY RANGE (created_at);

-- Партиции для заказов (по месяцам)
DO $$
DECLARE
    start_date DATE;
    end_date DATE;
    partition_name TEXT;
BEGIN
    FOR i IN 0..11 LOOP
        start_date := DATE_TRUNC('month', CURRENT_DATE) + (i || ' months')::INTERVAL;
        end_date := start_date + INTERVAL '1 month';
        partition_name := 'orders_' || TO_CHAR(start_date, 'YYYY_MM');
        
        EXECUTE FORMAT(
            'CREATE TABLE IF NOT EXISTS %I PARTITION OF orders
             FOR VALUES FROM (%L) TO (%L)',
            partition_name, start_date, end_date
        );
    END LOOP;
END $$;

CREATE INDEX ON orders(user_id, created_at DESC);
CREATE INDEX ON orders(order_number);
CREATE INDEX ON orders(status);

-- 8. Позиции заказа
CREATE TABLE order_items (
    id BIGSERIAL PRIMARY KEY,
    order_id BIGINT NOT NULL,
    product_id BIGINT REFERENCES products(id),
    sku VARCHAR(50),
    product_name VARCHAR(255) NOT NULL,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    price DECIMAL(10,2) NOT NULL,
    subtotal DECIMAL(10,2) NOT NULL,
    metadata JSONB -- Снимок атрибутов на момент заказа
);

CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_order_items_product ON order_items(product_id);

-- 9. Корзина покупок
CREATE TABLE cart (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,
    product_id BIGINT REFERENCES products(id) ON DELETE CASCADE,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    added_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(user_id, product_id)
);

CREATE INDEX idx_cart_user ON cart(user_id);

-- 10. Отзывы
CREATE TABLE reviews (
    id BIGSERIAL PRIMARY KEY,
    product_id BIGINT REFERENCES products(id) ON DELETE CASCADE,
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,
    order_id BIGINT, -- Только покупатели могут оставлять отзывы
    rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
    title VARCHAR(200),
    comment TEXT,
    is_verified BOOLEAN DEFAULT false, -- Проверенная покупка
    is_approved BOOLEAN DEFAULT false,
    helpful_count INTEGER DEFAULT 0,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_reviews_product ON reviews(product_id, is_approved);
CREATE INDEX idx_reviews_user ON reviews(user_id);
CREATE INDEX idx_reviews_rating ON reviews(rating);

-- 11. Избранное
CREATE TABLE wishlists (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,
    product_id BIGINT REFERENCES products(id) ON DELETE CASCADE,
    added_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(user_id, product_id)
);

CREATE INDEX idx_wishlists_user ON wishlists(user_id);

-- 12. Купоны и промокоды
CREATE TABLE coupons (
    id SERIAL PRIMARY KEY,
    code VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    discount_type VARCHAR(20) CHECK (discount_type IN ('percentage', 'fixed')),
    discount_value DECIMAL(10,2) NOT NULL,
    min_purchase DECIMAL(10,2),
    max_discount DECIMAL(10,2),
    usage_limit INTEGER,
    used_count INTEGER DEFAULT 0,
    valid_from TIMESTAMPTZ DEFAULT NOW(),
    valid_until TIMESTAMPTZ,
    is_active BOOLEAN DEFAULT true
);

CREATE INDEX idx_coupons_code ON coupons(code) WHERE is_active = true;

-- 13. История использования купонов
CREATE TABLE coupon_usage (
    id BIGSERIAL PRIMARY KEY,
    coupon_id INTEGER REFERENCES coupons(id),
    order_id BIGINT,
    user_id BIGINT REFERENCES users(id),
    discount_amount DECIMAL(10,2),
    used_at TIMESTAMPTZ DEFAULT NOW()
);

-- 14. Логи аудита (партиционированный)
CREATE TABLE audit_log (
    id BIGSERIAL,
    table_name VARCHAR(50),
    record_id BIGINT,
    action VARCHAR(20) CHECK (action IN ('INSERT', 'UPDATE', 'DELETE')),
    old_data JSONB,
    new_data JSONB,
    user_id BIGINT,
    ip_address INET,
    created_at TIMESTAMPTZ DEFAULT NOW()
) PARTITION BY RANGE (created_at);

-- Партиции для аудита (по месяцам)
DO $$
DECLARE
    start_date DATE;
    end_date DATE;
    partition_name TEXT;
BEGIN
    FOR i IN 0..11 LOOP
        start_date := DATE_TRUNC('month', CURRENT_DATE) + (i || ' months')::INTERVAL;
        end_date := start_date + INTERVAL '1 month';
        partition_name := 'audit_log_' || TO_CHAR(start_date, 'YYYY_MM');
        
        EXECUTE FORMAT(
            'CREATE TABLE IF NOT EXISTS %I PARTITION OF audit_log
             FOR VALUES FROM (%L) TO (%L)',
            partition_name, start_date, end_date
        );
    END LOOP;
END $$;

-- STORED PROCEDURES И ФУНКЦИИ

-- 1. Создание заказа (атомарная операция)
CREATE OR REPLACE FUNCTION create_order(
    p_user_id BIGINT,
    p_shipping_address_id BIGINT,
    p_billing_address_id BIGINT,
    p_payment_method VARCHAR,
    p_coupon_code VARCHAR DEFAULT NULL
) RETURNS BIGINT AS $$
DECLARE
    v_order_id BIGINT;
    v_order_number VARCHAR;
    v_subtotal DECIMAL(10,2);
    v_discount DECIMAL(10,2) := 0;
    v_total DECIMAL(10,2);
    v_coupon_id INTEGER;
    cart_item RECORD;
BEGIN
    -- Начать транзакцию
    -- Генерировать номер заказа
    v_order_number := 'ORD-' || TO_CHAR(NOW(), 'YYYYMMDD') || '-' || 
                      LPAD(NEXTVAL('orders_id_seq')::TEXT, 6, '0');
    
    -- Рассчитать сумму из корзины
    SELECT COALESCE(SUM(c.quantity * p.price), 0) INTO v_subtotal
    FROM cart c
    JOIN products p ON c.product_id = p.id
    WHERE c.user_id = p_user_id;
    
    IF v_subtotal = 0 THEN
        RAISE EXCEPTION 'Cart is empty';
    END IF;
    
    -- Применить купон если есть
    IF p_coupon_code IS NOT NULL THEN
        SELECT id, 
               CASE 
                   WHEN discount_type = 'percentage' THEN v_subtotal * discount_value / 100
                   ELSE discount_value
               END INTO v_coupon_id, v_discount
        FROM coupons
        WHERE code = p_coupon_code
          AND is_active = true
          AND (valid_until IS NULL OR valid_until > NOW())
          AND (usage_limit IS NULL OR used_count < usage_limit)
          AND (min_purchase IS NULL OR v_subtotal >= min_purchase);
        
        IF v_coupon_id IS NULL THEN
            RAISE EXCEPTION 'Invalid coupon code';
        END IF;
        
        -- Применить максимальную скидку если задана
        SELECT LEAST(v_discount, COALESCE(max_discount, v_discount)) INTO v_discount
        FROM coupons WHERE id = v_coupon_id;
    END IF;
    
    v_total := v_subtotal - v_discount;
    
    -- Создать заказ
    INSERT INTO orders (
        order_number, user_id, subtotal, discount, total,
        payment_method, shipping_address_id, billing_address_id
    ) VALUES (
        v_order_number, p_user_id, v_subtotal, v_discount, v_total,
        p_payment_method, p_shipping_address_id, p_billing_address_id
    ) RETURNING id INTO v_order_id;
    
    -- Перенести товары из корзины в заказ
    FOR cart_item IN 
        SELECT c.product_id, c.quantity, p.sku, p.name, p.price
        FROM cart c
        JOIN products p ON c.product_id = p.id
        WHERE c.user_id = p_user_id
    LOOP
        -- Проверить наличие на складе
        IF cart_item.quantity > (SELECT stock FROM products WHERE id = cart_item.product_id) THEN
            RAISE EXCEPTION 'Insufficient stock for product: %', cart_item.name;
        END IF;
        
        -- Добавить в order_items
        INSERT INTO order_items (
            order_id, product_id, sku, product_name, quantity, price, subtotal
        ) VALUES (
            v_order_id, cart_item.product_id, cart_item.sku, 
            cart_item.name, cart_item.quantity, cart_item.price,
            cart_item.quantity * cart_item.price
        );
        
        -- Уменьшить остатки
        UPDATE products 
        SET stock = stock - cart_item.quantity 
        WHERE id = cart_item.product_id;
        
        -- Записать в лог инвентаря
        INSERT INTO inventory_log (product_id, change_type, quantity, old_stock, new_stock)
        SELECT 
            cart_item.product_id, 'sale', -cart_item.quantity,
            stock + cart_item.quantity, stock
        FROM products WHERE id = cart_item.product_id;
    END LOOP;
    
    -- Очистить корзину
    DELETE FROM cart WHERE user_id = p_user_id;
    
    -- Записать использование купона
    IF v_coupon_id IS NOT NULL THEN
        INSERT INTO coupon_usage (coupon_id, order_id, user_id, discount_amount)
        VALUES (v_coupon_id, v_order_id, p_user_id, v_discount);
        
        UPDATE coupons SET used_count = used_count + 1 WHERE id = v_coupon_id;
    END IF;
    
    RETURN v_order_id;
END;
$$ LANGUAGE plpgsql;

-- 2. Получить популярные продукты
CREATE OR REPLACE FUNCTION get_popular_products(limit_count INTEGER DEFAULT 10)
RETURNS TABLE (
    product_id BIGINT,
    product_name VARCHAR,
    total_sold BIGINT,
    revenue DECIMAL
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        p.id,
        p.name,
        SUM(oi.quantity)::BIGINT AS total_sold,
        SUM(oi.subtotal) AS revenue
    FROM products p
    JOIN order_items oi ON p.id = oi.product_id
    JOIN orders o ON oi.order_id = o.id
    WHERE o.status NOT IN ('cancelled', 'refunded')
      AND o.created_at > NOW() - INTERVAL '30 days'
    GROUP BY p.id, p.name
    ORDER BY total_sold DESC
    LIMIT limit_count;
END;
$$ LANGUAGE plpgsql;

-- 3. Обновить средний рейтинг продукта
CREATE OR REPLACE FUNCTION update_product_rating()
RETURNS TRIGGER AS $$
BEGIN
    -- Обновить поле avg_rating в products (нужно добавить колонку)
    UPDATE products p
    SET 
        -- Добавим эти колонки позже
        -- avg_rating = (SELECT AVG(rating) FROM reviews WHERE product_id = NEW.product_id AND is_approved = true),
        -- review_count = (SELECT COUNT(*) FROM reviews WHERE product_id = NEW.product_id AND is_approved = true)
    WHERE id = NEW.product_id;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_rating_after_review
AFTER INSERT OR UPDATE ON reviews
FOR EACH ROW EXECUTE FUNCTION update_product_rating();

-- МАТЕРИАЛИЗОВАННЫЕ ПРЕДСТАВЛЕНИЯ

-- Статистика продаж по категориям
CREATE MATERIALIZED VIEW category_sales_stats AS
SELECT 
    c.id AS category_id,
    c.name AS category_name,
    c.path AS category_path,
    COUNT(DISTINCT p.id) AS product_count,
    COUNT(DISTINCT oi.order_id) AS order_count,
    SUM(oi.quantity) AS total_items_sold,
    SUM(oi.subtotal) AS total_revenue,
    AVG(oi.price) AS avg_price
FROM categories c
LEFT JOIN products p ON p.category_id = c.id
LEFT JOIN order_items oi ON oi.product_id = p.id
LEFT JOIN orders o ON o.id = oi.order_id
WHERE o.status NOT IN ('cancelled', 'refunded')
  OR o.id IS NULL
GROUP BY c.id, c.name, c.path;

CREATE UNIQUE INDEX ON category_sales_stats(category_id);

-- Обновлять каждый час
SELECT cron.schedule(
    'refresh-category-stats',
    '0 * * * *',
    'REFRESH MATERIALIZED VIEW CONCURRENTLY category_sales_stats'
);

-- ROW LEVEL SECURITY

-- Пользователи видят только свои заказы
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY user_orders_policy ON orders
    FOR SELECT
    USING (user_id = current_setting('app.user_id')::BIGINT);

-- Админы видят все
CREATE POLICY admin_orders_policy ON orders
    FOR ALL
    TO admin_role
    USING (true);

-- РОЛИ И ПРАВА

-- Создать роли
CREATE ROLE ecommerce_readonly;
CREATE ROLE ecommerce_app;
CREATE ROLE ecommerce_admin;

-- Права для readonly
GRANT CONNECT ON DATABASE ecommerce_db TO ecommerce_readonly;
GRANT USAGE ON SCHEMA ecommerce TO ecommerce_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA ecommerce TO ecommerce_readonly;

-- Права для приложения
GRANT CONNECT ON DATABASE ecommerce_db TO ecommerce_app;
GRANT USAGE ON SCHEMA ecommerce TO ecommerce_app;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA ecommerce TO ecommerce_app;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA ecommerce TO ecommerce_app;
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA ecommerce TO ecommerce_app;

-- Админ
GRANT ALL ON SCHEMA ecommerce TO ecommerce_admin;
GRANT ALL ON ALL TABLES IN SCHEMA ecommerce TO ecommerce_admin;
GRANT ALL ON ALL SEQUENCES IN SCHEMA ecommerce TO ecommerce_admin;

-- ИНДЕКСЫ ДЛЯ ПРОИЗВОДИТЕЛЬНОСТИ

-- Дополнительные индексы для частых запросов
CREATE INDEX idx_orders_user_status ON orders(user_id, status, created_at DESC);
CREATE INDEX idx_order_items_order_product ON order_items(order_id, product_id);
CREATE INDEX idx_products_category_active ON products(category_id, is_active) WHERE is_active = true;
CREATE INDEX idx_products_featured ON products(is_featured, created_at DESC) WHERE is_featured = true;

-- BACKUP СТРАТЕГИЯ

-- Настроить continuous archiving в postgresql.conf
-- wal_level = replica
-- archive_mode = on
-- archive_command = 'test ! -f /backup/wal/%f && cp %p /backup/wal/%f'

-- Скрипт для ежедневного backup
```

bash

```bash
#!/bin/bash
# /usr/local/bin/ecommerce_backup.sh

BACKUP_DIR="/backup/ecommerce"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="ecommerce_db"

# Создать директорию
mkdir -p $BACKUP_DIR/$DATE

# Backup
pg_dump -U postgres -d $DB_NAME -F d -j 4 -f $BACKUP_DIR/$DATE

# Сжать
tar -czf $BACKUP_DIR/ecommerce_$DATE.tar.gz -C $BACKUP_DIR $DATE
rm -rf $BACKUP_DIR/$DATE

# Удалить старые backups (> 7 дней)
find $BACKUP_DIR -name "ecommerce_*.tar.gz" -mtime +7 -delete

# Загрузить в S3
aws s3 cp $BACKUP_DIR/ecommerce_$DATE.tar.gz s3://my-backups/postgresql/

echo "Backup completed: $DATE"
```

**Упражнение:**

1. Реализуйте полную схему базы данных
2. Заполните тестовыми данными (используйте generate_series)
3. Настройте все индексы и партиции
4. Создайте stored procedures для бизнес-логики
5. Настройте Row Level Security
6. Реализуйте backup стратегию
7. Настройте мониторинг
8. Создайте документацию по API базы данных
9. Нагрузочное тестирование с pgbench
10. Оптимизируйте медленные запросы

---

## ✅ ЧЕКЛИСТ НАВЫКОВ

### Junior

- [ ]  Устанавливаю и подключаюсь к PostgreSQL
- [ ]  Создаю таблицы с разными типами данных
- [ ]  Выполняю CRUD операции
- [ ]  Использую базовые JOIN
- [ ]  Работаю с GROUP BY и агрегатными функциями
- [ ]  Понимаю и использую WHERE условия
- [ ]  Создаю простые индексы

### Middle

- [ ]  Работаю с транзакциями и уровнями изоляции
- [ ]  Использую подзапросы и CTE
- [ ]  Владею оконными функциями
- [ ]  Работаю с JSON/JSONB
- [ ]  Настраиваю полнотекстовый поиск
- [ ]  Партиционирую большие таблицы
- [ ]  Понимаю EXPLAIN и оптимизирую запросы

### Senior

- [ ]  Настраиваю репликацию
- [ ]  Оптимизирую производительность БД
- [ ]  Реализую backup и recovery стратегии
- [ ]  Настраиваю безопасность (роли, RLS, шифрование)
- [ ]  Использую расширения PostgreSQL
- [ ]  Настраиваю комплексный мониторинг
- [ ]  Проектирую масштабируемые схемы БД
- [ ]  Автоматизирую операции DevOps

---

## 📚 ДОПОЛНИТЕЛЬНЫЕ РЕСУРСЫ

### Документация

- Official PostgreSQL Documentation: [https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)
- PostgreSQL Wiki: [https://wiki.postgresql.org/](https://wiki.postgresql.org/)

### Книги

- "PostgreSQL: Up and Running" by Regina Obe & Leo Hsu
- "PostgreSQL 14 Administration Cookbook" by Simon Riggs
- "Mastering PostgreSQL 15" by Hans-Jürgen Schönig

### Инструменты

- pgAdmin - GUI для PostgreSQL
- DBeaver - Универсальный клиент БД
- pgcli - CLI с автодополнением
- PostgREST - RESTful API из PostgreSQL
- Hasura - GraphQL движок для PostgreSQL

### Онлайн ресурсы

- Postgres Weekly Newsletter
- Planet PostgreSQL Blog
- /r/PostgreSQL на Reddit
- PostgreSQL Slack Community

---

**Поздравляю! Вы прошли полный курс по PostgreSQL от новичка до эксперта. Продолжайте практиковаться и изучать новые возможности!** 🎉
