# PostgreSQL Refresh: Ежегодный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые концепции PostgreSQL за 3-4 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый модуль состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальные задачи, которые нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

---

## Модуль 1: Установка, подключение и базовые команды (20 минут)

### 🎯 Напоминалка

**Установка PostgreSQL:**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install postgresql postgresql-contrib

# RHEL/CentOS
sudo yum install postgresql-server postgresql-contrib
sudo postgresql-setup --initdb
sudo systemctl start postgresql

# Проверка статуса
sudo systemctl status postgresql
```

**Управление службой:**
```bash
# Запуск/остановка/перезапуск
sudo systemctl start postgresql
sudo systemctl stop postgresql
sudo systemctl restart postgresql
sudo systemctl reload postgresql    # Перечитать конфиг без остановки

# Автозапуск
sudo systemctl enable postgresql

# Проверка порта
sudo ss -tulpn | grep 5432
sudo netstat -plnt | grep 5432
```

**Подключение к PostgreSQL:**
```bash
# От пользователя postgres (peer аутентификация)
sudo -u postgres psql

# От другого пользователя
psql -U username -d database_name

# Удаленное подключение
psql -h hostname -p 5432 -U username -d database_name

# С указанием пароля (будет запрошен)
psql -U username -d database_name -W

# Строка подключения (connection string)
psql "postgresql://username:password@hostname:5432/database_name"

# Выполнение команды и выход
psql -U postgres -c "SELECT version();"

# Выполнение SQL файла
psql -U postgres -d mydb -f script.sql
```

**Основные psql команды:**
```sql
-- Мета-команды (начинаются с \)
\?                    -- Помощь по командам psql
\h                    -- Помощь по SQL командам
\h SELECT             -- Помощь по конкретной команде

-- Список объектов
\l                    -- Список баз данных
\l+                   -- С дополнительной информацией
\c database_name      -- Подключиться к другой БД
\dt                   -- Список таблиц
\dt+                  -- С размерами
\d table_name         -- Описание таблицы
\d+ table_name        -- Подробное описание
\di                   -- Список индексов
\dv                   -- Список view
\df                   -- Список функций
\dn                   -- Список схем
\du                   -- Список пользователей

-- Информация о БД
\conninfo             -- Текущее подключение
\timing               -- Включить время выполнения запросов
\x                    -- Расширенный вывод (построчно)
\q                    -- Выход

-- Вывод
\o filename           -- Вывод в файл
\o                    -- Отмена вывода в файл

-- История
\s                    -- История команд
\s filename           -- Сохранить историю в файл
```

**Базовые SQL команды:**
```sql
-- Создание БД
CREATE DATABASE mydb;
CREATE DATABASE mydb WITH ENCODING 'UTF8' LC_COLLATE='en_US.UTF-8' LC_CTYPE='en_US.UTF-8';

-- Удаление БД
DROP DATABASE mydb;
DROP DATABASE IF EXISTS mydb;

-- Создание пользователя
CREATE USER myuser WITH PASSWORD 'mypassword';
CREATE ROLE myuser WITH LOGIN PASSWORD 'mypassword';

-- Права доступа
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
GRANT SELECT, INSERT, UPDATE ON TABLE mytable TO myuser;
REVOKE INSERT ON TABLE mytable FROM myuser;

-- Смена пароля
ALTER USER myuser WITH PASSWORD 'newpassword';

-- Текущая БД и пользователь
SELECT current_database();
SELECT current_user;

-- Версия PostgreSQL
SELECT version();

-- Размер БД
SELECT pg_size_pretty(pg_database_size('mydb'));

-- Активные подключения
SELECT * FROM pg_stat_activity;
```

**Конфигурационные файлы:**
```bash
# Основной конфиг
/etc/postgresql/14/main/postgresql.conf     # Ubuntu/Debian
/var/lib/pgsql/14/data/postgresql.conf      # RHEL/CentOS

# Аутентификация
/etc/postgresql/14/main/pg_hba.conf         # Ubuntu/Debian
/var/lib/pgsql/14/data/pg_hba.conf          # RHEL/CentOS

# Поиск конфигов из psql
SHOW config_file;
SHOW hba_file;
SHOW data_directory;

# Перезагрузка конфигурации
SELECT pg_reload_conf();
-- или
sudo systemctl reload postgresql
```

### 💻 Задание

Выполни следующие задачи:

1. Проверь статус службы PostgreSQL
2. Подключись к PostgreSQL от пользователя postgres
3. Выведи версию PostgreSQL
4. Создай базу данных `test_db`
5. Подключись к базе `test_db`
6. Создай пользователя `testuser` с паролем
7. Выдай все права на базу `test_db` пользователю `testuser`
8. Выведи список всех баз данных с размерами
9. Проверь путь к конфигурационному файлу
10. Выведи количество активных подключений

### 🚀 Бонус (новое)

- Настрой удаленное подключение к PostgreSQL:
  - Измени `postgresql.conf`: `listen_addresses = '*'`
  - Добавь строку в `pg_hba.conf`: `host all all 0.0.0.0/0 md5`
  - Перезагрузи PostgreSQL
  - Проверь подключение с другого хоста

- Создай алиас для быстрого подключения в `.bashrc`:
  ```bash
  alias pgcon='psql -U postgres'
  alias pgtest='psql -U testuser -d test_db'
  ```

- Установи `pgcli` - улучшенный клиент с автодополнением:
  ```bash
  pip install pgcli
  pgcli -U postgres
  ```

---

## Модуль 2: Создание и управление таблицами (25 минут)

### 🎯 Напоминалка

**Создание таблиц:**
```sql
-- Базовая таблица
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Таблица с внешним ключом
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    published BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Таблица с ограничениями
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    total_amount NUMERIC(10, 2) CHECK (total_amount > 0),
    status VARCHAR(20) CHECK (status IN ('pending', 'completed', 'cancelled')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Временная таблица
CREATE TEMP TABLE temp_data (
    id INTEGER,
    value TEXT
);

-- Таблица как копия
CREATE TABLE users_backup AS SELECT * FROM users;
CREATE TABLE users_structure AS SELECT * FROM users WHERE 1=0;  -- Только структура
```

**Типы данных:**
```sql
-- Числовые
SMALLINT              -- 2 байта, -32768 до 32767
INTEGER, INT          -- 4 байта
BIGINT                -- 8 байт
NUMERIC(p, s)         -- Точные числа с фиксированной точкой
DECIMAL(p, s)         -- Синоним NUMERIC
REAL                  -- 4 байта, 6 знаков точности
DOUBLE PRECISION      -- 8 байт, 15 знаков точности
SERIAL                -- Автоинкремент INTEGER
BIGSERIAL             -- Автоинкремент BIGINT

-- Строковые
VARCHAR(n)            -- Переменная длина с лимитом
CHAR(n)               -- Фиксированная длина
TEXT                  -- Неограниченная длина

-- Дата и время
DATE                  -- Только дата
TIME                  -- Только время
TIMESTAMP             -- Дата и время
TIMESTAMPTZ           -- С часовым поясом
INTERVAL              -- Интервал времени

-- Булевы
BOOLEAN               -- true, false, null

-- Специальные
UUID                  -- Уникальный идентификатор
JSON, JSONB           -- JSON данные
ARRAY                 -- Массивы
HSTORE                -- Ключ-значение
```

**Изменение таблиц:**
```sql
-- Добавить колонку
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users ADD COLUMN age INTEGER DEFAULT 0;

-- Удалить колонку
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users DROP COLUMN IF EXISTS phone;

-- Переименовать колонку
ALTER TABLE users RENAME COLUMN username TO login;

-- Изменить тип колонки
ALTER TABLE users ALTER COLUMN age TYPE SMALLINT;
ALTER TABLE users ALTER COLUMN email TYPE VARCHAR(150);

-- Изменить значение по умолчанию
ALTER TABLE users ALTER COLUMN age SET DEFAULT 18;
ALTER TABLE users ALTER COLUMN age DROP DEFAULT;

-- Добавить/удалить NOT NULL
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
ALTER TABLE users ALTER COLUMN phone DROP NOT NULL;

-- Добавить ограничение
ALTER TABLE users ADD CONSTRAINT users_email_check CHECK (email LIKE '%@%');
ALTER TABLE users ADD CONSTRAINT users_age_check CHECK (age >= 0 AND age <= 150);

-- Удалить ограничение
ALTER TABLE users DROP CONSTRAINT users_age_check;

-- Переименовать таблицу
ALTER TABLE users RENAME TO app_users;
```

**Индексы:**
```sql
-- Создание индекса
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- Уникальный индекс
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Составной индекс
CREATE INDEX idx_posts_user_date ON posts(user_id, created_at);

-- Частичный индекс
CREATE INDEX idx_active_users ON users(username) WHERE active = true;

-- Индекс с выражением
CREATE INDEX idx_lower_email ON users(LOWER(email));

-- Различные типы индексов
CREATE INDEX idx_name_btree ON users USING btree(username);    -- По умолчанию
CREATE INDEX idx_location_gist ON locations USING gist(coords);
CREATE INDEX idx_tags_gin ON posts USING gin(tags);

-- Удаление индекса
DROP INDEX idx_users_email;
DROP INDEX IF EXISTS idx_users_email;

-- Список индексов
\di
SELECT * FROM pg_indexes WHERE tablename = 'users';
```

**Ограничения (Constraints):**
```sql
-- PRIMARY KEY
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    -- или
    id INTEGER,
    PRIMARY KEY (id)
);

-- UNIQUE
CREATE TABLE users (
    email VARCHAR(100) UNIQUE,
    -- или
    email VARCHAR(100),
    UNIQUE (email)
);

-- NOT NULL
CREATE TABLE orders (
    order_number VARCHAR(50) NOT NULL
);

-- CHECK
CREATE TABLE products (
    price NUMERIC(10, 2) CHECK (price > 0),
    quantity INTEGER CHECK (quantity >= 0)
);

-- FOREIGN KEY
CREATE TABLE orders (
    user_id INTEGER REFERENCES users(id),
    -- или с опциями
    user_id INTEGER,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE ON UPDATE CASCADE
);

-- DEFAULT
CREATE TABLE posts (
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'draft'
);
```

### 💻 Задание

Создай структуру базы данных для простого блога:

1. **Создай таблицу `authors`:**
   - `id` (primary key, автоинкремент)
   - `name` (varchar 100, not null)
   - `email` (varchar 100, unique, not null)
   - `bio` (text, nullable)
   - `created_at` (timestamp, default current_timestamp)

2. **Создай таблицу `categories`:**
   - `id` (primary key, автоинкремент)
   - `name` (varchar 50, unique, not null)
   - `slug` (varchar 50, unique, not null)

3. **Создай таблицу `articles`:**
   - `id` (primary key, автоинкремент)
   - `author_id` (foreign key к authors, on delete cascade)
   - `category_id` (foreign key к categories, on delete set null, nullable)
   - `title` (varchar 200, not null)
   - `slug` (varchar 200, unique, not null)
   - `content` (text, not null)
   - `published` (boolean, default false)
   - `views` (integer, default 0, check >= 0)
   - `created_at` (timestamp, default current_timestamp)
   - `updated_at` (timestamp, default current_timestamp)

4. **Создай индексы:**
   - На `authors.email`
   - На `articles.slug`
   - Составной индекс на `articles(author_id, published, created_at)`
   - Частичный индекс на опубликованные статьи: `articles(created_at) WHERE published = true`

5. **Выведи структуру всех созданных таблиц** (`\d+ table_name`)

6. **Выведи список всех индексов**

### 🚀 Бонус (новое)

- Добавь таблицу `article_tags` (many-to-many связь между articles и tags):
  ```sql
  CREATE TABLE tags (
      id SERIAL PRIMARY KEY,
      name VARCHAR(50) UNIQUE NOT NULL
  );
  
  CREATE TABLE article_tags (
      article_id INTEGER REFERENCES articles(id) ON DELETE CASCADE,
      tag_id INTEGER REFERENCES tags(id) ON DELETE CASCADE,
      PRIMARY KEY (article_id, tag_id)
  );
  ```

- Создай индекс для полнотекстового поиска:
  ```sql
  CREATE INDEX idx_articles_fulltext ON articles 
  USING gin(to_tsvector('english', title || ' ' || content));
  ```

- Добавь триггер для автоматического обновления `updated_at`:
  ```sql
  CREATE OR REPLACE FUNCTION update_modified_column()
  RETURNS TRIGGER AS $$
  BEGIN
      NEW.updated_at = CURRENT_TIMESTAMP;
      RETURN NEW;
  END;
  $$ LANGUAGE plpgsql;
  
  CREATE TRIGGER update_articles_modtime
  BEFORE UPDATE ON articles
  FOR EACH ROW
  EXECUTE FUNCTION update_modified_column();
  ```

---

## Модуль 3: Запросы и манипуляция данными (30 минут)

### 🎯 Напоминалка

**INSERT - вставка данных:**
```sql
-- Одна запись
INSERT INTO authors (name, email) VALUES ('John Doe', 'john@example.com');

-- Несколько записей
INSERT INTO authors (name, email) VALUES 
    ('Jane Smith', 'jane@example.com'),
    ('Bob Wilson', 'bob@example.com');

-- Все колонки (по порядку)
INSERT INTO categories VALUES (DEFAULT, 'Technology', 'technology');

-- С возвратом вставленных данных
INSERT INTO authors (name, email) 
VALUES ('Alice Brown', 'alice@example.com') 
RETURNING id, name;

-- Из другой таблицы
INSERT INTO users_backup SELECT * FROM users WHERE active = true;

-- При конфликте (UPSERT)
INSERT INTO users (username, email) 
VALUES ('john', 'john@example.com')
ON CONFLICT (username) 
DO UPDATE SET email = EXCLUDED.email;

-- Или игнорировать
INSERT INTO users (username, email) 
VALUES ('john', 'john@example.com')
ON CONFLICT (username) DO NOTHING;
```

**SELECT - выборка данных:**
```sql
-- Базовые запросы
SELECT * FROM authors;
SELECT name, email FROM authors;
SELECT DISTINCT category_id FROM articles;

-- WHERE - фильтрация
SELECT * FROM authors WHERE id = 1;
SELECT * FROM articles WHERE published = true;
SELECT * FROM articles WHERE views > 100;
SELECT * FROM articles WHERE title LIKE '%PostgreSQL%';
SELECT * FROM articles WHERE title ILIKE '%postgresql%';  -- Регистронезависимый
SELECT * FROM articles WHERE created_at > '2024-01-01';
SELECT * FROM articles WHERE category_id IS NULL;
SELECT * FROM articles WHERE category_id IN (1, 2, 3);
SELECT * FROM articles WHERE views BETWEEN 100 AND 1000;

-- Логические операторы
SELECT * FROM articles WHERE published = true AND views > 100;
SELECT * FROM articles WHERE category_id = 1 OR category_id = 2;
SELECT * FROM articles WHERE NOT published;

-- ORDER BY - сортировка
SELECT * FROM articles ORDER BY created_at DESC;
SELECT * FROM articles ORDER BY views DESC, created_at DESC;
SELECT * FROM articles ORDER BY title ASC NULLS LAST;

-- LIMIT и OFFSET - пагинация
SELECT * FROM articles LIMIT 10;
SELECT * FROM articles LIMIT 10 OFFSET 20;  -- Страница 3
SELECT * FROM articles ORDER BY id LIMIT 10 OFFSET 0;  -- Страница 1

-- Агрегатные функции
SELECT COUNT(*) FROM articles;
SELECT COUNT(*) FROM articles WHERE published = true;
SELECT AVG(views) FROM articles;
SELECT SUM(views) FROM articles;
SELECT MIN(created_at), MAX(created_at) FROM articles;

-- GROUP BY - группировка
SELECT author_id, COUNT(*) FROM articles GROUP BY author_id;
SELECT category_id, AVG(views) FROM articles GROUP BY category_id;
SELECT DATE(created_at), COUNT(*) FROM articles GROUP BY DATE(created_at);

-- HAVING - фильтрация после группировки
SELECT author_id, COUNT(*) as article_count 
FROM articles 
GROUP BY author_id 
HAVING COUNT(*) > 5;

SELECT category_id, AVG(views) as avg_views 
FROM articles 
GROUP BY category_id 
HAVING AVG(views) > 100;
```

**JOIN - объединение таблиц:**
```sql
-- INNER JOIN - только совпадающие записи
SELECT a.title, au.name 
FROM articles a
INNER JOIN authors au ON a.author_id = au.id;

-- LEFT JOIN - все из левой таблицы
SELECT a.title, c.name 
FROM articles a
LEFT JOIN categories c ON a.category_id = c.id;

-- RIGHT JOIN - все из правой таблицы
SELECT a.title, au.name 
FROM articles a
RIGHT JOIN authors au ON a.author_id = au.id;

-- FULL OUTER JOIN - все записи
SELECT a.title, au.name 
FROM articles a
FULL OUTER JOIN authors au ON a.author_id = au.id;

-- Множественные JOIN
SELECT 
    a.title,
    au.name as author_name,
    c.name as category_name
FROM articles a
INNER JOIN authors au ON a.author_id = au.id
LEFT JOIN categories c ON a.category_id = c.id;

-- Соединение с условием
SELECT a.title, au.name 
FROM articles a
INNER JOIN authors au ON a.author_id = au.id AND a.published = true;
```

**UPDATE - обновление данных:**
```sql
-- Обновить одну запись
UPDATE articles SET views = views + 1 WHERE id = 1;

-- Обновить несколько полей
UPDATE articles 
SET title = 'New Title', updated_at = CURRENT_TIMESTAMP 
WHERE id = 1;

-- Обновить с условием
UPDATE articles SET published = true WHERE views > 1000;

-- Обновить из другой таблицы
UPDATE articles a
SET category_id = c.id
FROM categories c
WHERE a.title LIKE '%' || c.name || '%';

-- С возвратом обновленных данных
UPDATE articles 
SET views = views + 1 
WHERE id = 1 
RETURNING id, title, views;
```

**DELETE - удаление данных:**
```sql
-- Удалить одну запись
DELETE FROM articles WHERE id = 1;

-- Удалить с условием
DELETE FROM articles WHERE published = false;

-- Удалить все (осторожно!)
DELETE FROM articles;

-- Более быстрое удаление всех (не логируется, сброс счетчиков)
TRUNCATE TABLE articles;
TRUNCATE TABLE articles RESTART IDENTITY;  -- Сбросить SERIAL
TRUNCATE TABLE articles CASCADE;  -- С зависимыми таблицами

-- С возвратом удаленных данных
DELETE FROM articles WHERE id = 1 RETURNING *;
```

**Подзапросы:**
```sql
-- В WHERE
SELECT * FROM articles 
WHERE author_id = (SELECT id FROM authors WHERE email = 'john@example.com');

-- С IN
SELECT * FROM articles 
WHERE author_id IN (SELECT id FROM authors WHERE name LIKE 'J%');

-- С EXISTS
SELECT * FROM authors a 
WHERE EXISTS (SELECT 1 FROM articles WHERE author_id = a.id);

-- В SELECT
SELECT 
    a.title,
    (SELECT name FROM authors WHERE id = a.author_id) as author_name
FROM articles a;

-- В FROM (derived table)
SELECT avg_views 
FROM (
    SELECT AVG(views) as avg_views 
    FROM articles 
    GROUP BY author_id
) as subquery;
```

**Common Table Expressions (CTE):**
```sql
-- Простой CTE
WITH published_articles AS (
    SELECT * FROM articles WHERE published = true
)
SELECT * FROM published_articles WHERE views > 100;

-- Множественные CTE
WITH 
active_authors AS (
    SELECT id, name FROM authors WHERE created_at > '2024-01-01'
),
popular_articles AS (
    SELECT * FROM articles WHERE views > 1000
)
SELECT a.title, au.name
FROM popular_articles a
JOIN active_authors au ON a.author_id = au.id;

-- Рекурсивный CTE (например, для категорий с родителями)
WITH RECURSIVE category_tree AS (
    SELECT id, name, parent_id, 0 as level
    FROM categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.id, c.name, c.parent_id, ct.level + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree;
```

### 💻 Задание

Используя созданные в предыдущем модуле таблицы:

1. **Вставь тестовые данные:**
   - 3-5 авторов
   - 3-4 категории
   - 10-15 статей (с разными авторами, категориями, статусами)

2. **Напиши запросы:**
   - Выведи все опубликованные статьи с именами авторов и названиями категорий
   - Найди топ-3 статьи по просмотрам
   - Подсчитай количество статей у каждого автора
   - Найди авторов, у которых больше 2 опубликованных статей
   - Выведи категории с средним количеством просмотров > 50
   - Найди статьи, опубликованные за последние 7 дней
   - Выведи авторов, у которых нет опубликованных статей

3. **Обновления:**
   - Увеличь просмотры на 10 для всех статей в категории "Technology"
   - Опубликуй все статьи с просмотрами > 100

4. **Удаление:**
   - Удали неопубликованные статьи старше 30 дней

### 🚀 Бонус (новое)

- Напиши запрос с использованием CTE для расчета рейтинга авторов:
  ```sql
  WITH author_stats AS (
      SELECT 
          author_id,
          COUNT(*) as total_articles,
          SUM(CASE WHEN published THEN 1 ELSE 0 END) as published_count,
          AVG(views) as avg_views
      FROM articles
      GROUP BY author_id
  )
  SELECT 
      au.name,
      ast.*,
      (published_count * 10 + avg_views) as rating
  FROM author_stats ast
  JOIN authors au ON ast.author_id = au.id
  ORDER BY rating DESC;
  ```

- Используй window functions для нумерации статей внутри каждой категории:
  ```sql
  SELECT 
      title,
      category_id,
      views,
      ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY views DESC) as rank
  FROM articles
  WHERE published = true;
  ```

- Создай материализованное представление (materialized view):
  ```sql
  CREATE MATERIALIZED VIEW article_stats AS
  SELECT 
      DATE(created_at) as date,
      COUNT(*) as total,
      SUM(CASE WHEN published THEN 1 ELSE 0 END) as published
  FROM articles
  GROUP BY DATE(created_at);
  
  -- Обновление
  REFRESH MATERIALIZED VIEW article_stats;
  ```

---

## Модуль 4: Производительность и индексы (30 минут)

### 🎯 Напоминалка

**EXPLAIN - анализ плана запроса:**
```sql
-- Просмотр плана выполнения
EXPLAIN SELECT * FROM articles WHERE published = true;

-- С фактической статистикой выполнения
EXPLAIN ANALYZE SELECT * FROM articles WHERE published = true;

-- С подробной информацией
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) 
SELECT * FROM articles WHERE published = true;

-- Форматы вывода
EXPLAIN (FORMAT JSON) SELECT * FROM articles;
EXPLAIN (FORMAT YAML) SELECT * FROM articles;

-- Понимание вывода EXPLAIN:
-- Seq Scan - последовательное сканирование (плохо для больших таблиц)
-- Index Scan - использование индекса (хорошо)
-- Index Only Scan - только по индексу (отлично)
-- Bitmap Index Scan - для больших выборок
-- cost=0.00..10.00 - оценка стоимости (начало..конец)
-- rows=100 - ожидаемое количество строк
-- width=50 - средняя ширина строки в байтах
```

**Типы индексов:**
```sql
-- B-tree (по умолчанию) - для большинства случаев
CREATE INDEX idx_articles_created ON articles(created_at);

-- Hash - только для равенства (=)
CREATE INDEX idx_articles_status ON articles USING hash(published);

-- GiST - для геометрических данных, полнотекстовый поиск
CREATE INDEX idx_articles_search ON articles USING gist(to_tsvector('english', content));

-- GIN - для массивов, JSONB, полнотекстовый поиск
CREATE INDEX idx_articles_tags ON articles USING gin(tags);

-- BRIN - для очень больших таблиц с естественной сортировкой
CREATE INDEX idx_logs_created ON logs USING brin(created_at);

-- Покрывающий индекс (включает дополнительные колонки)
CREATE INDEX idx_articles_cover ON articles(author_id) INCLUDE (title, created_at);

-- Уникальный индекс
CREATE UNIQUE INDEX idx_articles_slug ON articles(slug);

-- Частичный индекс (индексирует только часть данных)
CREATE INDEX idx_published_articles ON articles(created_at) WHERE published = true;

-- Индекс с выражением
CREATE INDEX idx_lower_email ON users(LOWER(email));

-- Многоколоночный индекс (порядок важен!)
CREATE INDEX idx_articles_compound ON articles(author_id, category_id, created_at);
```

**Статистика и оптимизация:**
```sql
-- Обновление статистики
ANALYZE;
ANALYZE articles;
ANALYZE VERBOSE;  -- С подробным выводом

-- Вакуумирование (очистка мертвых строк)
VACUUM;
VACUUM articles;
VACUUM FULL articles;  -- Полная перестройка (блокирует таблицу)
VACUUM ANALYZE articles;  -- Вакуум + обновление статистики

-- Автовакуум (проверка настроек)
SHOW autovacuum;
SELECT * FROM pg_stat_user_tables WHERE relname = 'articles';

-- Размер таблицы и индексов
SELECT 
    pg_size_pretty(pg_total_relation_size('articles')) as total_size,
    pg_size_pretty(pg_relation_size('articles')) as table_size,
    pg_size_pretty(pg_indexes_size('articles')) as indexes_size;

-- Неиспользуемые индексы
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0 AND indexrelname NOT LIKE 'pg_toast%'
ORDER BY pg_relation_size(indexrelid) DESC;

-- Дублирующиеся индексы
SELECT 
    a.indrelid::regclass as table,
    a.indexrelid::regclass as index1,
    b.indexrelid::regclass as index2,
    a.indkey as columns
FROM pg_index a
JOIN pg_index b ON a.indrelid = b.indrelid 
    AND a.indexrelid > b.indexrelid
    AND a.indkey = b.indkey;

-- Раздутые таблицы (bloat)
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

**Оптимизация запросов:**
```sql
-- Плохо: SELECT *
SELECT * FROM articles WHERE id = 1;

-- Хорошо: Только нужные колонки
SELECT id, title, content FROM articles WHERE id = 1;

-- Плохо: Без индекса
SELECT * FROM articles WHERE LOWER(title) = 'hello';

-- Хорошо: С индексом по выражению
CREATE INDEX idx_lower_title ON articles(LOWER(title));
SELECT * FROM articles WHERE LOWER(title) = 'hello';

-- Плохо: OR может не использовать индексы
SELECT * FROM articles WHERE author_id = 1 OR category_id = 2;

-- Хорошо: UNION или IN
SELECT * FROM articles WHERE author_id = 1
UNION
SELECT * FROM articles WHERE category_id = 2;

-- Плохо: Подзапрос в SELECT для каждой строки
SELECT 
    a.*,
    (SELECT name FROM authors WHERE id = a.author_id) as author
FROM articles a;

-- Хорошо: JOIN
SELECT a.*, au.name as author
FROM articles a
JOIN authors au ON a.author_id = au.id;

-- Используй LIMIT для пагинации
SELECT * FROM articles ORDER BY created_at DESC LIMIT 10 OFFSET 0;

-- Избегай OFFSET на больших значениях (медленно)
-- Вместо OFFSET используй WHERE с последним id
SELECT * FROM articles 
WHERE id > last_seen_id 
ORDER BY id 
LIMIT 10;
```

**Настройки производительности (postgresql.conf):**
```sql
-- Память
shared_buffers = 256MB            -- 25% от RAM (начальное значение)
effective_cache_size = 1GB        -- 50-75% от RAM
work_mem = 4MB                    -- Память на операцию сортировки/хеш
maintenance_work_mem = 64MB       -- Для VACUUM, CREATE INDEX

-- Контрольные точки
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100   -- Точность статистики (10-10000)

-- Параллелизм
max_worker_processes = 8
max_parallel_workers_per_gather = 2
max_parallel_workers = 8

-- Логирование медленных запросов
log_min_duration_statement = 1000  -- Логировать запросы > 1 сек
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
log_statement = 'none'             -- 'none', 'ddl', 'mod', 'all'

-- Применение изменений
SELECT pg_reload_conf();
-- или
sudo systemctl reload postgresql
```

### 💻 Задание

1. **Анализ производительности:**
   - Создай таблицу с 100,000+ записей:
     ```sql
     CREATE TABLE test_performance (
         id SERIAL PRIMARY KEY,
         user_id INTEGER,
         title VARCHAR(200),
         status VARCHAR(20),
         created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     );
     
     INSERT INTO test_performance (user_id, title, status)
     SELECT 
         (random() * 1000)::INTEGER,
         'Title ' || generate_series,
         CASE WHEN random() > 0.5 THEN 'active' ELSE 'inactive' END
     FROM generate_series(1, 100000);
     ```

2. **Проверь план запроса БЕЗ индекса:**
   ```sql
   EXPLAIN ANALYZE 
   SELECT * FROM test_performance WHERE user_id = 500;
   ```

3. **Создай индекс и повтори:**
   ```sql
   CREATE INDEX idx_test_user_id ON test_performance(user_id);
   EXPLAIN ANALYZE 
   SELECT * FROM test_performance WHERE user_id = 500;
   ```

4. **Сравни время выполнения**

5. **Найди неиспользуемые индексы** в твоей БД

6. **Проверь размеры таблиц и индексов**

7. **Выполни VACUUM ANALYZE** на большой таблице

### 🚀 Бонус (новое)

- Создай частичный индекс для активных записей:
  ```sql
  CREATE INDEX idx_active_records ON test_performance(user_id) 
  WHERE status = 'active';
  ```

- Используй `pg_stat_statements` для поиска медленных запросов:
  ```sql
  -- Включи расширение
  CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
  
  -- Топ-10 медленных запросов
  SELECT 
      query,
      calls,
      total_exec_time,
      mean_exec_time,
      max_exec_time
  FROM pg_stat_statements
  ORDER BY mean_exec_time DESC
  LIMIT 10;
  ```

- Создай покрывающий индекс:
  ```sql
  CREATE INDEX idx_cover ON test_performance(user_id) 
  INCLUDE (title, created_at);
  
  -- Теперь этот запрос будет использовать Index Only Scan
  EXPLAIN ANALYZE
  SELECT user_id, title, created_at 
  FROM test_performance 
  WHERE user_id = 500;
  ```

---

## Модуль 5: Транзакции и блокировки (25 минут)

### 🎯 Напоминалка

**Транзакции - ACID:**
```sql
-- Базовая транзакция
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Откат транзакции
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Что-то пошло не так
ROLLBACK;

-- Точки сохранения (savepoints)
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- Ошибка во втором UPDATE
ROLLBACK TO SAVEPOINT my_savepoint;
-- Первый UPDATE сохранен, второй отменен
COMMIT;

-- Автокоммит (по умолчанию включен)
-- Каждая команда - отдельная транзакция
UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Автоматически COMMIT

-- Отключение автокоммита в psql
\set AUTOCOMMIT off
```

**Уровни изоляции транзакций:**
```sql
-- Read Uncommitted (не поддерживается, работает как Read Committed)
BEGIN TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- Read Committed (по умолчанию)
-- Видит только зафиксированные изменения
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Repeatable Read
-- Видит снимок данных на начало транзакции
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Serializable
-- Самый строгий, предотвращает все аномалии
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Проверка текущего уровня
SHOW transaction_isolation;

-- Установка по умолчанию
SET default_transaction_isolation = 'repeatable read';
```

**Блокировки:**
```sql
-- Эксклюзивная блокировка строки (FOR UPDATE)
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- Другие транзакции будут ждать
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;

-- Разделяемая блокировка (FOR SHARE)
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR SHARE;
-- Другие могут читать, но не могут изменять
COMMIT;

-- Пропустить заблокированные строки
SELECT * FROM accounts WHERE id = 1 FOR UPDATE NOWAIT;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE SKIP LOCKED;

-- Блокировка таблицы
BEGIN;
LOCK TABLE accounts IN ACCESS EXCLUSIVE MODE;
-- Полная блокировка таблицы
COMMIT;

-- Различные режимы блокировки таблиц
LOCK TABLE accounts IN ROW SHARE MODE;
LOCK TABLE accounts IN ROW EXCLUSIVE MODE;
LOCK TABLE accounts IN SHARE MODE;
LOCK TABLE accounts IN SHARE ROW EXCLUSIVE MODE;
LOCK TABLE accounts IN EXCLUSIVE MODE;
LOCK TABLE accounts IN ACCESS EXCLUSIVE MODE;
```

**Мониторинг блокировок:**
```sql
-- Просмотр текущих блокировок
SELECT 
    pg_class.relname,
    pg_locks.locktype,
    pg_locks.mode,
    pg_locks.granted,
    pg_stat_activity.pid,
    pg_stat_activity.query
FROM pg_locks
JOIN pg_class ON pg_locks.relation = pg_class.oid
JOIN pg_stat_activity ON pg_locks.pid = pg_stat_activity.pid
WHERE pg_class.relname = 'accounts';

-- Блокирующие и заблокированные запросы
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
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

-- Убить блокирующий процесс
SELECT pg_terminate_backend(pid);
SELECT pg_cancel_backend(pid);  -- Мягкая отмена
```

**Deadlocks (взаимные блокировки):**
```sql
-- PostgreSQL автоматически обнаруживает deadlock и отменяет одну транзакцию

-- Пример deadlock:
-- Транзакция 1:
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- ждем...
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Транзакция 2 (одновременно):
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 2;
-- ждем...
UPDATE accounts SET balance = balance + 100 WHERE id = 1;  -- Deadlock!
COMMIT;

-- Проверка deadlock в логах
SELECT * FROM pg_stat_database WHERE datname = 'mydb';

-- Настройки
SHOW deadlock_timeout;  -- Время до обнаружения deadlock (по умолчанию 1s)
```

**Активные соединения и транзакции:**
```sql
-- Список активных соединений
SELECT 
    pid,
    usename,
    application_name,
    client_addr,
    state,
    query,
    query_start,
    state_change
FROM pg_stat_activity
WHERE state = 'active';

-- Длительные транзакции
SELECT 
    pid,
    usename,
    query,
    state,
    NOW() - xact_start AS duration
FROM pg_stat_activity
WHERE state != 'idle'
    AND xact_start IS NOT NULL
ORDER BY duration DESC;

-- Idle транзакции (могут держать блокировки)
SELECT 
    pid,
    usename,
    state,
    NOW() - state_change AS idle_duration,
    query
FROM pg_stat_activity
WHERE state = 'idle in transaction'
ORDER BY idle_duration DESC;

-- Количество соединений
SELECT 
    COUNT(*),
    state
FROM pg_stat_activity
GROUP BY state;

-- Лимиты соединений
SHOW max_connections;
SELECT count(*) FROM pg_stat_activity;
```

### 💻 Задание

1. **Работа с транзакциями:**
   ```sql
   -- Создай тестовую таблицу
   CREATE TABLE bank_accounts (
       id SERIAL PRIMARY KEY,
       name VARCHAR(100),
       balance NUMERIC(10, 2)
   );
   
   INSERT INTO bank_accounts (name, balance) VALUES
       ('Alice', 1000.00),
       ('Bob', 500.00),
       ('Charlie', 750.00);
   ```

2. **Выполни транзакцию перевода денег:**
   - Начни транзакцию
   - Сними 100 со счета Alice
   - Добавь 100 на счет Bob
   - Зафиксируй транзакцию
   - Проверь балансы

3. **Протестируй ROLLBACK:**
   - Начни транзакцию
   - Измени какие-то данные
   - Выполни ROLLBACK
   - Убедись, что изменения не сохранились

4. **Эксперимент с блокировками:**
   - Открой два терминала/подключения
   - В первом: начни транзакцию и сделай `SELECT ... FOR UPDATE`
   - Во втором: попробуй UPDATE той же строки
   - Наблюдай блокировку
   - Закоммить первую транзакцию и проверь вторую

5. **Мониторинг:**
   - Выведи все активные соединения
   - Найди свой PID и запрос
   - Посмотри текущие блокировки

### 🚀 Бонус (новое)

- Создай функцию для безопасного перевода денег:
```sql
  CREATE OR REPLACE FUNCTION transfer_money(
      from_account INTEGER,
      to_account INTEGER,
      amount NUMERIC
  ) RETURNS BOOLEAN AS $
  BEGIN
      -- Проверка достаточности средств
      IF (SELECT balance FROM bank_accounts WHERE id = from_account) < amount THEN
          RAISE EXCEPTION 'Insufficient funds';
      END IF;
      
      -- Перевод
      UPDATE bank_accounts SET balance = balance - amount WHERE id = from_account;
      UPDATE bank_accounts SET balance = balance + amount WHERE id = to_account;
      
      RETURN TRUE;
  EXCEPTION
      WHEN OTHERS THEN
          RAISE NOTICE 'Transfer failed: %', SQLERRM;
          RETURN FALSE;
  END;
  $ LANGUAGE plpgsql;
  
  -- Использование
SELECT * FROM get_active_users();

-- Функция с OUT параметрами
CREATE OR REPLACE FUNCTION get_user_stats(user_id INTEGER, 
    OUT total_posts INTEGER, 
    OUT total_views INTEGER)
AS $
BEGIN
    SELECT COUNT(*), COALESCE(SUM(views), 0) 
    INTO total_posts, total_views
    FROM articles 
    WHERE author_id = user_id;
END;
$ LANGUAGE plpgsql;

-- Использование
SELECT * FROM get_user_stats(1);

-- Функция с условиями
CREATE OR REPLACE FUNCTION get_user_level(total_points INTEGER)
RETURNS TEXT AS $
BEGIN
    IF total_points >= 1000 THEN
        RETURN 'Expert';
    ELSIF total_points >= 500 THEN
        RETURN 'Advanced';
    ELSIF total_points >= 100 THEN
        RETURN 'Intermediate';
    ELSE
        RETURN 'Beginner';
    END IF;
END;
$ LANGUAGE plpgsql;

-- Функция с циклом
CREATE OR REPLACE FUNCTION generate_series_text(start_num INTEGER, end_num INTEGER)
RETURNS TEXT AS $
DECLARE
    result TEXT := '';
    i INTEGER;
BEGIN
    FOR i IN start_num..end_num LOOP
        result := result || i || ',';
    END LOOP;
    RETURN TRIM(TRAILING ',' FROM result);
END;
$ LANGUAGE plpgsql;

-- Функция с обработкой ошибок
CREATE OR REPLACE FUNCTION safe_divide(numerator NUMERIC, denominator NUMERIC)
RETURNS NUMERIC AS $
BEGIN
    IF denominator = 0 THEN
        RAISE EXCEPTION 'Division by zero';
    END IF;
    RETURN numerator / denominator;
EXCEPTION
    WHEN OTHERS THEN
        RAISE NOTICE 'Error: %', SQLERRM;
        RETURN NULL;
END;
$ LANGUAGE plpgsql;

-- Удаление функции
DROP FUNCTION IF EXISTS function_name(param_types);
```

**Процедуры (PostgreSQL 11+):**
```sql
-- Процедура (не возвращает значение, может использовать COMMIT)
CREATE OR REPLACE PROCEDURE update_user_stats()
LANGUAGE plpgsql AS $
BEGIN
    UPDATE users SET last_login = CURRENT_TIMESTAMP;
    COMMIT;
END;
$;

-- Вызов процедуры
CALL update_user_stats();

-- Процедура с параметрами
CREATE OR REPLACE PROCEDURE archive_old_data(days_old INTEGER)
LANGUAGE plpgsql AS $
BEGIN
    DELETE FROM logs WHERE created_at < CURRENT_DATE - days_old;
    COMMIT;
END;
$;

-- Вызов
CALL archive_old_data(30);

-- Процедура с транзакциями
CREATE OR REPLACE PROCEDURE batch_insert(batch_size INTEGER)
LANGUAGE plpgsql AS $
DECLARE
    i INTEGER;
BEGIN
    FOR i IN 1..batch_size LOOP
        INSERT INTO test_table (data) VALUES ('Data ' || i);
        IF i % 100 = 0 THEN
            COMMIT;
        END IF;
    END LOOP;
    COMMIT;
END;
$;
```

**Триггеры:**
```sql
-- Функция для триггера
CREATE OR REPLACE FUNCTION update_modified_timestamp()
RETURNS TRIGGER AS $
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$ LANGUAGE plpgsql;

-- Создание триггера
CREATE TRIGGER update_articles_timestamp
BEFORE UPDATE ON articles
FOR EACH ROW
EXECUTE FUNCTION update_modified_timestamp();

-- Триггер AFTER INSERT
CREATE OR REPLACE FUNCTION log_new_user()
RETURNS TRIGGER AS $
BEGIN
    INSERT INTO user_logs (user_id, action, created_at)
    VALUES (NEW.id, 'created', CURRENT_TIMESTAMP);
    RETURN NEW;
END;
$ LANGUAGE plpgsql;

CREATE TRIGGER log_user_creation
AFTER INSERT ON users
FOR EACH ROW
EXECUTE FUNCTION log_new_user();

-- Триггер BEFORE DELETE
CREATE OR REPLACE FUNCTION prevent_admin_delete()
RETURNS TRIGGER AS $
BEGIN
    IF OLD.role = 'admin' THEN
        RAISE EXCEPTION 'Cannot delete admin users';
    END IF;
    RETURN OLD;
END;
$ LANGUAGE plpgsql;

CREATE TRIGGER check_admin_delete
BEFORE DELETE ON users
FOR EACH ROW
EXECUTE FUNCTION prevent_admin_delete();

-- Триггер на уровне выражения (statement level)
CREATE OR REPLACE FUNCTION audit_table_changes()
RETURNS TRIGGER AS $
BEGIN
    INSERT INTO audit_log (table_name, operation, timestamp)
    VALUES (TG_TABLE_NAME, TG_OP, CURRENT_TIMESTAMP);
    RETURN NULL;
END;
$ LANGUAGE plpgsql;

CREATE TRIGGER audit_articles
AFTER INSERT OR UPDATE OR DELETE ON articles
FOR EACH STATEMENT
EXECUTE FUNCTION audit_table_changes();

-- Условный триггер (с WHEN)
CREATE TRIGGER update_timestamp_when_published
BEFORE UPDATE ON articles
FOR EACH ROW
WHEN (NEW.published = true AND OLD.published = false)
EXECUTE FUNCTION update_modified_timestamp();

-- Отключить/включить триггер
ALTER TABLE articles DISABLE TRIGGER update_articles_timestamp;
ALTER TABLE articles ENABLE TRIGGER update_articles_timestamp;

-- Удалить триггер
DROP TRIGGER IF EXISTS trigger_name ON table_name;

-- Список триггеров
\dft
SELECT * FROM information_schema.triggers WHERE event_object_table = 'articles';
```

**Агрегатные функции:**
```sql
-- Встроенные агрегатные функции
SELECT 
    COUNT(*) as total,
    AVG(views) as avg_views,
    SUM(views) as total_views,
    MIN(views) as min_views,
    MAX(views) as max_views,
    STDDEV(views) as stddev_views
FROM articles;

-- STRING_AGG (конкатенация строк)
SELECT 
    author_id,
    STRING_AGG(title, ', ' ORDER BY created_at DESC) as articles
FROM articles
GROUP BY author_id;

-- ARRAY_AGG (массив значений)
SELECT 
    author_id,
    ARRAY_AGG(id ORDER BY created_at DESC) as article_ids
FROM articles
GROUP BY author_id;

-- JSON_AGG (JSON массив)
SELECT 
    author_id,
    JSON_AGG(JSON_BUILD_OBJECT('id', id, 'title', title)) as articles
FROM articles
GROUP BY author_id;

-- Создание своей агрегатной функции
CREATE OR REPLACE FUNCTION median_transfn(state INTEGER[], val INTEGER)
RETURNS INTEGER[] AS $
BEGIN
    RETURN array_append(state, val);
END;
$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION median_finalfn(state INTEGER[])
RETURNS NUMERIC AS $
DECLARE
    sorted INTEGER[];
    len INTEGER;
BEGIN
    sorted := ARRAY(SELECT unnest(state) ORDER BY 1);
    len := array_length(sorted, 1);
    IF len % 2 = 0 THEN
        RETURN (sorted[len/2] + sorted[len/2 + 1]) / 2.0;
    ELSE
        RETURN sorted[len/2 + 1];
    END IF;
END;
$ LANGUAGE plpgsql;

CREATE AGGREGATE median(INTEGER) (
    SFUNC = median_transfn,
    STYPE = INTEGER[],
    FINALFUNC = median_finalfn
);

-- Использование
SELECT median(views) FROM articles;
```

**Window Functions:**
```sql
-- ROW_NUMBER
SELECT 
    title,
    author_id,
    views,
    ROW_NUMBER() OVER (PARTITION BY author_id ORDER BY views DESC) as rank
FROM articles;

-- RANK и DENSE_RANK
SELECT 
    title,
    views,
    RANK() OVER (ORDER BY views DESC) as rank,
    DENSE_RANK() OVER (ORDER BY views DESC) as dense_rank
FROM articles;

-- LAG и LEAD (предыдущее/следующее значение)
SELECT 
    title,
    created_at,
    views,
    LAG(views) OVER (ORDER BY created_at) as previous_views,
    LEAD(views) OVER (ORDER BY created_at) as next_views
FROM articles;

-- FIRST_VALUE и LAST_VALUE
SELECT 
    title,
    author_id,
    views,
    FIRST_VALUE(title) OVER (PARTITION BY author_id ORDER BY views DESC) as best_article
FROM articles;

-- Кумулятивная сумма
SELECT 
    DATE(created_at) as date,
    COUNT(*) as daily_count,
    SUM(COUNT(*)) OVER (ORDER BY DATE(created_at)) as cumulative_count
FROM articles
GROUP BY DATE(created_at);

-- Скользящее среднее
SELECT 
    DATE(created_at) as date,
    AVG(views) as avg_views,
    AVG(AVG(views)) OVER (ORDER BY DATE(created_at) ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as moving_avg_7days
FROM articles
GROUP BY DATE(created_at);
```

**Хранимые процедуры для batch операций:**
```sql
-- Batch update
CREATE OR REPLACE PROCEDURE batch_update_status(
    batch_size INTEGER DEFAULT 1000
)
LANGUAGE plpgsql AS $
DECLARE
    updated_count INTEGER := 0;
    total_updated INTEGER := 0;
BEGIN
    LOOP
        UPDATE articles
        SET status = 'archived'
        WHERE id IN (
            SELECT id 
            FROM articles 
            WHERE status = 'draft' 
            AND created_at < CURRENT_DATE - INTERVAL '1 year'
            LIMIT batch_size
        );
        
        GET DIAGNOSTICS updated_count = ROW_COUNT;
        total_updated := total_updated + updated_count;
        
        EXIT WHEN updated_count = 0;
        
        COMMIT;
        RAISE NOTICE 'Updated % rows (total: %)', updated_count, total_updated;
    END LOOP;
    
    RAISE NOTICE 'Batch update completed. Total rows: %', total_updated;
END;
$;

-- Вызов
CALL batch_update_status(500);
```

### 💻 Задание

1. **Создай функции:**
   ```sql
   -- Функция для подсчета статистики автора
   CREATE OR REPLACE FUNCTION get_author_stats(author_id INTEGER)
   RETURNS TABLE(
       total_articles INTEGER,
       published_articles INTEGER,
       total_views INTEGER,
       avg_views NUMERIC
   ) AS $
   BEGIN
       RETURN QUERY
       SELECT 
           COUNT(*)::INTEGER,
           COUNT(*) FILTER (WHERE published = true)::INTEGER,
           COALESCE(SUM(views), 0)::INTEGER,
           COALESCE(AVG(views), 0)::NUMERIC(10,2)
       FROM articles
       WHERE articles.author_id = get_author_stats.author_id;
   END;
   $ LANGUAGE plpgsql;
   
   -- Функция для очистки старых данных
   CREATE OR REPLACE FUNCTION cleanup_old_logs(days_old INTEGER)
   RETURNS INTEGER AS $
   DECLARE
       deleted_count INTEGER;
   BEGIN
       DELETE FROM logs 
       WHERE created_at < CURRENT_DATE - days_old * INTERVAL '1 day';
       
       GET DIAGNOSTICS deleted_count = ROW_COUNT;
       RETURN deleted_count;
   END;
   $ LANGUAGE plpgsql;
   ```

2. **Создай триггер:**
   ```sql
   -- Таблица для логирования
   CREATE TABLE article_changes (
       id SERIAL PRIMARY KEY,
       article_id INTEGER,
       old_title TEXT,
       new_title TEXT,
       changed_by TEXT DEFAULT current_user,
       changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   
   -- Функция триггера
   CREATE OR REPLACE FUNCTION log_article_title_change()
   RETURNS TRIGGER AS $
   BEGIN
       IF OLD.title IS DISTINCT FROM NEW.title THEN
           INSERT INTO article_changes (article_id, old_title, new_title)
           VALUES (NEW.id, OLD.title, NEW.title);
       END IF;
       RETURN NEW;
   END;
   $ LANGUAGE plpgsql;
   
   -- Создать триггер
   CREATE TRIGGER track_article_title_changes
   AFTER UPDATE ON articles
   FOR EACH ROW
   EXECUTE FUNCTION log_article_title_change();
   ```

3. **Протестируй функции:**
   - Вызови `get_author_stats()` для нескольких авторов
   - Вызови `cleanup_old_logs(30)`

4. **Протестируй триггер:**
   - Обнови title статьи
   - Проверь таблицу `article_changes`

5. **Window Functions:**
   - Выведи ранкинг статей по просмотрам внутри каждой категории
   - Найди для каждой статьи предыдущую и следующую по дате
   - Посчитай кумулятивное количество статей по датам

### 🚀 Бонус (новое)

- Создай процедуру для архивации старых данных с партицированием:
  ```sql
  CREATE OR REPLACE PROCEDURE archive_partition(
      source_table TEXT,
      archive_table TEXT,
      cutoff_date DATE
  )
  LANGUAGE plpgsql AS $
  DECLARE
      moved_count INTEGER;
  BEGIN
      -- Создать архивную таблицу если не существует
      EXECUTE format(
          'CREATE TABLE IF NOT EXISTS %I (LIKE %I INCLUDING ALL)',
          archive_table, source_table
      );
      
      -- Переместить данные
      EXECUTE format(
          'WITH moved AS (
              DELETE FROM %I 
              WHERE created_at < $1 
              RETURNING *
          )
          INSERT INTO %I SELECT * FROM moved',
          source_table, archive_table
      ) USING cutoff_date;
      
      GET DIAGNOSTICS moved_count = ROW_COUNT;
      RAISE NOTICE 'Archived % rows from % to %', 
          moved_count, source_table, archive_table;
      
      COMMIT;
  END;
  $;
  ```

- Создай функцию для динамического поиска:
  ```sql
  CREATE OR REPLACE FUNCTION search_articles(
      search_term TEXT DEFAULT NULL,
      author_filter INTEGER DEFAULT NULL,
      published_only BOOLEAN DEFAULT true
  )
  RETURNS TABLE(id INTEGER, title TEXT, author_name TEXT, views INTEGER) AS $
  BEGIN
      RETURN QUERY EXECUTE
          format('
              SELECT a.id, a.title, au.name, a.views
              FROM articles a
              JOIN authors au ON a.author_id = au.id
              WHERE ($1 IS NULL OR a.title ILIKE ''%%'' || $1 || ''%%'')
              AND ($2 IS NULL OR a.author_id = $2)
              AND ($3 = false OR a.published = true)
              ORDER BY a.views DESC
          ')
          USING search_term, author_filter, published_only;
  END;
  $ LANGUAGE plpgsql;
  
  -- Использование
  SELECT * FROM search_articles('PostgreSQL', NULL, true);
  ```

- Создай материализованное представление с автообновлением:
  ```sql
  CREATE MATERIALIZED VIEW daily_stats AS
  SELECT 
      DATE(created_at) as date,
      COUNT(*) as total_articles,
      SUM(CASE WHEN published THEN 1 ELSE 0 END) as published_count,
      AVG(views) as avg_views
  FROM articles
  GROUP BY DATE(created_at);
  
  CREATE UNIQUE INDEX ON daily_stats(date);
  
  -- Функция для автоматического обновления
  CREATE OR REPLACE FUNCTION refresh_daily_stats()
  RETURNS void AS $
  BEGIN
      REFRESH MATERIALIZED VIEW CONCURRENTLY daily_stats;
  END;
  $ LANGUAGE plpgsql;
  
  -- Настроить автоматическое обновление через cron или pg_cron
  ```

---

## Финальный проект (60 минут)

### Задача: Создание системы мониторинга и управления базой данных

Создай комплексное решение для мониторинга, обслуживания и оптимизации PostgreSQL.

### Требования:

**1. Схема мониторинга:**
```sql
-- База данных для мониторинга
CREATE DATABASE monitoring_db;

-- Таблицы для сбора метрик
CREATE TABLE db_metrics (
    id SERIAL PRIMARY KEY,
    metric_name VARCHAR(100),
    metric_value NUMERIC,
    database_name VARCHAR(100),
    collected_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE slow_queries (
    id SERIAL PRIMARY KEY,
    query_text TEXT,
    execution_time NUMERIC,
    calls INTEGER,
    database_name VARCHAR(100),
    detected_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE connection_logs (
    id SERIAL PRIMARY KEY,
    database_name VARCHAR(100),
    username VARCHAR(100),
    connection_count INTEGER,
    max_connections INTEGER,
    logged_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE table_bloat (
    id SERIAL PRIMARY KEY,
    table_name VARCHAR(200),
    total_size BIGINT,
    dead_tuples BIGINT,
    bloat_ratio NUMERIC,
    checked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**2. Функции сбора метрик:**
```sql
-- Функция для сбора основных метрик БД
CREATE OR REPLACE FUNCTION collect_db_metrics()
RETURNS void AS $
BEGIN
    -- Размер БД
    INSERT INTO db_metrics (metric_name, metric_value, database_name)
    SELECT 'database_size', pg_database_size(datname), datname
    FROM pg_database
    WHERE datname NOT IN ('template0', 'template1');
    
    -- Cache hit ratio
    INSERT INTO db_metrics (metric_name, metric_value, database_name)
    SELECT 
        'cache_hit_ratio',
        CASE 
            WHEN sum(heap_blks_hit) + sum(heap_blks_read) = 0 THEN 100
            ELSE (sum(heap_blks_hit)::float / (sum(heap_blks_hit) + sum(heap_blks_read))) * 100
        END,
        current_database()
    FROM pg_statio_user_tables;
    
    -- Количество подключений
    INSERT INTO connection_logs (database_name, username, connection_count, max_connections)
    SELECT 
        datname,
        usename,
        count(*),
        (SELECT setting::int FROM pg_settings WHERE name = 'max_connections')
    FROM pg_stat_activity
    WHERE datname IS NOT NULL
    GROUP BY datname, usename;
END;
$ LANGUAGE plpgsql;

-- Функция для поиска медленных запросов
CREATE OR REPLACE FUNCTION detect_slow_queries(threshold_ms NUMERIC DEFAULT 1000)
RETURNS void AS $
BEGIN
    INSERT INTO slow_queries (query_text, execution_time, calls, database_name)
    SELECT 
        query,
        mean_exec_time,
        calls,
        current_database()
    FROM pg_stat_statements
    WHERE mean_exec_time > threshold_ms
    AND query NOT LIKE '%pg_stat_statements%'
    ORDER BY mean_exec_time DESC
    LIMIT 10;
END;
$ LANGUAGE plpgsql;

-- Функция для определения bloat
CREATE OR REPLACE FUNCTION check_table_bloat()
RETURNS void AS $
BEGIN
    INSERT INTO table_bloat (table_name, total_size, dead_tuples, bloat_ratio)
    SELECT 
        schemaname || '.' || tablename,
        pg_total_relation_size(schemaname||'.'||tablename),
        n_dead_tup,
        CASE 
            WHEN n_live_tup > 0 THEN (n_dead_tup::float / n_live_tup) * 100
            ELSE 0
        END
    FROM pg_stat_user_tables
    WHERE n_dead_tup > 1000
    ORDER BY n_dead_tup DESC;
END;
$ LANGUAGE plpgsql;
```

**3. Процедуры обслуживания:**
```sql
-- Автоматический VACUUM для раздутых таблиц
CREATE OR REPLACE PROCEDURE auto_vacuum_bloated_tables(bloat_threshold NUMERIC DEFAULT 20)
LANGUAGE plpgsql AS $
DECLARE
    table_record RECORD;
    vacuum_count INTEGER := 0;
BEGIN
    FOR table_record IN 
        SELECT 
            schemaname,
            tablename,
            n_dead_tup,
            n_live_tup
        FROM pg_stat_user_tables
        WHERE n_live_tup > 0 
        AND (n_dead_tup::float / n_live_tup) * 100 > bloat_threshold
    LOOP
        RAISE NOTICE 'Vacuuming %.%', table_record.schemaname, table_record.tablename;
        EXECUTE format('VACUUM ANALYZE %I.%I', table_record.schemaname, table_record.tablename);
        vacuum_count := vacuum_count + 1;
    END LOOP;
    
    RAISE NOTICE 'Vacuumed % tables', vacuum_count;
END;
$;

-- Анализ и пересоздание индексов
CREATE OR REPLACE PROCEDURE reindex_fragmented_indexes(min_size_mb INTEGER DEFAULT 100)
LANGUAGE plpgsql AS $
DECLARE
    index_record RECORD;
BEGIN
    FOR index_record IN
        SELECT 
            schemaname,
            indexrelname,
            pg_relation_size(indexrelid) / 1024 / 1024 as size_mb
        FROM pg_stat_user_indexes
        WHERE idx_scan = 0
        AND pg_relation_size(indexrelid) / 1024 / 1024 > min_size_mb
    LOOP
        RAISE NOTICE 'Reindexing %.%', index_record.schemaname, index_record.indexrelname;
        EXECUTE format('REINDEX INDEX CONCURRENTLY %I.%I', 
            index_record.schemaname, index_record.indexrelname);
    END LOOP;
END;
$;

-- Очистка старых метрик
CREATE OR REPLACE PROCEDURE cleanup_old_metrics(retention_days INTEGER DEFAULT 30)
LANGUAGE plpgsql AS $
DECLARE
    deleted_count INTEGER;
BEGIN
    DELETE FROM db_metrics WHERE collected_at < CURRENT_DATE - retention_days;
    GET DIAGNOSTICS deleted_count = ROW_COUNT;
    RAISE NOTICE 'Deleted % old metric records', deleted_count;
    
    DELETE FROM slow_queries WHERE detected_at < CURRENT_DATE - retention_days;
    GET DIAGNOSTICS deleted_count = ROW_COUNT;
    RAISE NOTICE 'Deleted % old slow query records', deleted_count;
    
    COMMIT;
END;
$;
```

**4. Представления для отчетов:**
```sql
-- Представление текущего здоровья БД
CREATE OR REPLACE VIEW db_health_dashboard AS
SELECT 
    'Database Size' as metric,
    pg_size_pretty(pg_database_size(current_database())) as value,
    NULL as status
UNION ALL
SELECT 
    'Total Connections',
    count(*)::TEXT,
    CASE 
        WHEN count(*) > (SELECT setting::int * 0.8 FROM pg_settings WHERE name = 'max_connections') 
        THEN 'WARNING'
        ELSE 'OK'
    END
FROM pg_stat_activity
UNION ALL
SELECT 
    'Active Queries',
    count(*)::TEXT,
    CASE WHEN count(*) > 50 THEN 'WARNING' ELSE 'OK' END
FROM pg_stat_activity
WHERE state = 'active'
UNION ALL
SELECT 
    'Cache Hit Ratio',
    round((sum(heap_blks_hit)::float / NULLIF(sum(heap_blks_hit) + sum(heap_blks_read), 0)) * 100, 2)::TEXT || '%',
    CASE 
        WHEN (sum(heap_blks_hit)::float / NULLIF(sum(heap_blks_hit) + sum(heap_blks_read), 0)) * 100 < 90 
        THEN 'WARNING'
        ELSE 'OK'
    END
FROM pg_statio_user_tables
UNION ALL
SELECT 
    'Long Running Queries (>5min)',
    count(*)::TEXT,
    CASE WHEN count(*) > 0 THEN 'WARNING' ELSE 'OK' END
FROM pg_stat_activity
WHERE state != 'idle' AND query_start < now() - interval '5 minutes';

-- Представление топ медленных запросов
CREATE OR REPLACE VIEW top_slow_queries AS
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time,
    rows
FROM pg_stat_statements
WHERE query NOT LIKE '%pg_stat%'
ORDER BY mean_exec_time DESC
LIMIT 20;

-- Представление проблемных таблиц
CREATE OR REPLACE VIEW problematic_tables AS
SELECT 
    schemaname || '.' || tablename as table_name,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
    n_dead_tup,
    n_live_tup,
    CASE 
        WHEN n_live_tup > 0 THEN round((n_dead_tup::float / n_live_tup) * 100, 2)
        ELSE 0
    END as bloat_ratio,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;
```

**5. Bash скрипт для автоматизации:**
```bash
#!/bin/bash
# postgres_maintenance.sh

DB_NAME="mydb"
DB_USER="postgres"
LOG_FILE="/var/log/postgres_maintenance.log"
EMAIL="admin@example.com"

log_message() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Сбор метрик
collect_metrics() {
    log_message "Collecting metrics..."
    psql -U "$DB_USER" -d "$DB_NAME" -c "SELECT collect_db_metrics();" >> "$LOG_FILE" 2>&1
    psql -U "$DB_USER" -d "$DB_NAME" -c "SELECT detect_slow_queries();" >> "$LOG_FILE" 2>&1
    psql -U "$DB_USER" -d "$DB_NAME" -c "SELECT check_table_bloat();" >> "$LOG_FILE" 2>&1
}

# Проверка здоровья
check_health() {
    log_message "Checking database health..."
    psql -U "$DB_USER" -d "$DB_NAME" -c "SELECT * FROM db_health_dashboard;" | tee -a "$LOG_FILE"
    
    # Проверить на WARNING
    WARNINGS=$(psql -U "$DB_USER" -d "$DB_NAME" -t -c "SELECT count(*) FROM db_health_dashboard WHERE status = 'WARNING';")
    
    if [ "$WARNINGS" -gt 0 ]; then
        log_message "WARNING: Found $WARNINGS issues"
        # Отправить email (требует настройки mail)
        # psql -U "$DB_USER" -d "$DB_NAME" -c "SELECT * FROM db_health_dashboard WHERE status = 'WARNING';" | mail -s "PostgreSQL Health Warning" "$EMAIL"
    fi
}

# Обслуживание
maintenance() {
    log_message "Running maintenance tasks..."
    
    # Auto vacuum
    psql -U "$DB_USER" -d "$DB_NAME" -c "CALL auto_vacuum_bloated_tables();" >> "$LOG_FILE" 2>&1
    
    # Очистка старых метрик
    psql -U "$DB_USER" -d "$DB_NAME" -c "CALL cleanup_old_metrics(30);" >> "$LOG_FILE" 2>&1
    
    # ANALYZE
    log_message "Running ANALYZE..."
    psql -U "$DB_USER" -d "$DB_NAME" -c "ANALYZE;" >> "$LOG_FILE" 2>&1
}

# Создание отчета
generate_report() {
    REPORT_FILE="/tmp/postgres_report_$(date +%Y%m%d).html"
    
    cat > "$REPORT_FILE" << EOF
<html>
<head><title>PostgreSQL Health Report - $(date +%Y-%m-%d)</title></head>
<body>
<h1>PostgreSQL Health Report</h1>
<h2>Generated: $(date)</h2>

<h3>Health Dashboard</h3>
<pre>
$(psql -U "$DB_USER" -d "$DB_NAME" -H -c "SELECT * FROM db_health_dashboard;")
</pre>

<h3>Top 10 Slow Queries</h3>
<pre>
$(psql -U "$DB_USER" -d "$DB_NAME" -H -c "SELECT * FROM top_slow_queries LIMIT 10;")
</pre>

<h3>Problematic Tables</h3>
<pre>
$(psql -U "$DB_USER" -d "$DB_NAME" -H -c "SELECT * FROM problematic_tables LIMIT 10;")
</pre>

</body>
</html>
EOF
    
    log_message "Report generated: $REPORT_FILE"
}

# Main
case "$1" in
    collect)
        collect_metrics
        ;;
    check)
        check_health
        ;;
    maintain)
        maintenance
        ;;
    report)
        generate_report
        ;;
    full)
        collect_metrics
        check_health
        maintenance
        generate_report
        ;;
    *)
        echo "Usage: $0 {collect|check|maintain|report|full}"
        exit 1
esac

log_message "Script completed"
```

**6. Настройка cron:**
```bash
# Добавить в crontab
# crontab -e

# Сбор метрик каждые 15 минут
*/15 * * * * /path/to/postgres_maintenance.sh collect

# Про
  SELECT transfer_money(1, 2, 100.00);
  ```

- Настрой логирование всех транзакций длительностью > 100ms:
  ```sql
  ALTER DATABASE mydb SET log_min_duration_statement = 100;
  ```

- Создай скрипт для мониторинга длительных транзакций:
  ```sql
  SELECT 
      pid,
      usename,
      application_name,
      client_addr,
      NOW() - xact_start as transaction_duration,
      NOW() - query_start as query_duration,
      state,
      query
  FROM pg_stat_activity
  WHERE xact_start IS NOT NULL
      AND NOW() - xact_start > INTERVAL '1 minute'
  ORDER BY xact_start;
  ```

---

## Модуль 6: Бэкап и восстановление (25 минут)

### 🎯 Напоминалка

**pg_dump - логический бэкап:**
```bash
# Бэкап одной базы данных
pg_dump -U postgres mydb > mydb_backup.sql
pg_dump -U postgres -d mydb -f mydb_backup.sql

# С компрессией
pg_dump -U postgres mydb | gzip > mydb_backup.sql.gz

# Кастомный формат (рекомендуется, поддерживает параллельное восстановление)
pg_dump -U postgres -d mydb -F c -f mydb_backup.dump

# Directory format (параллельный дамп)
pg_dump -U postgres -d mydb -F d -j 4 -f mydb_backup_dir

# Только схема (без данных)
pg_dump -U postgres -d mydb --schema-only -f schema.sql

# Только данные (без схемы)
pg_dump -U postgres -d mydb --data-only -f data.sql

# Конкретная таблица
pg_dump -U postgres -d mydb -t users -f users_backup.sql

# Несколько таблиц
pg_dump -U postgres -d mydb -t users -t orders -f backup.sql

# Исключить таблицы
pg_dump -U postgres -d mydb -T logs -T temp_data -f backup.sql

# С verbose и сжатием
pg_dump -U postgres -d mydb -F c -v -Z 9 -f mydb_backup.dump

# Удаленный бэкап
pg_dump -h remote_host -U postgres -d mydb -f backup.sql
```

**pg_dumpall - бэкап всех баз:**
```bash
# Все базы данных
pg_dumpall -U postgres > all_databases.sql
pg_dumpall -U postgres -f all_databases.sql

# Только роли и табличные пространства
pg_dumpall -U postgres --roles-only -f roles.sql
pg_dumpall -U postgres --tablespaces-only -f tablespaces.sql

# Глобальные объекты (роли, табличные пространства)
pg_dumpall -U postgres --globals-only -f globals.sql
```

**Восстановление из дампа:**
```bash
# Из SQL файла
psql -U postgres -d mydb < mydb_backup.sql
psql -U postgres -d mydb -f mydb_backup.sql

# Из сжатого файла
gunzip < mydb_backup.sql.gz | psql -U postgres -d mydb
zcat mydb_backup.sql.gz | psql -U postgres -d mydb

# Из кастомного формата
pg_restore -U postgres -d mydb -v mydb_backup.dump

# Параллельное восстановление
pg_restore -U postgres -d mydb -j 4 -v mydb_backup.dump

# Восстановление из directory format
pg_restore -U postgres -d mydb -F d -j 4 mydb_backup_dir

# Создать БД и восстановить
pg_restore -U postgres -C -d postgres mydb_backup.dump

# Восстановить только конкретную таблицу
pg_restore -U postgres -d mydb -t users mydb_backup.dump

# Восстановить только схему
pg_restore -U postgres -d mydb --schema-only mydb_backup.dump

# Восстановить только данные
pg_restore -U postgres -d mydb --data-only mydb_backup.dump

# С очисткой существующих объектов
pg_restore -U postgres -d mydb --clean mydb_backup.dump

# Не останавливаться на ошибках
pg_restore -U postgres -d mydb --no-owner --no-privileges -e mydb_backup.dump
```

**Физический бэкап (базовый):**
```bash
# Остановить PostgreSQL
sudo systemctl stop postgresql

# Скопировать data directory
sudo tar -czf postgres_physical_backup.tar.gz /var/lib/postgresql/14/main/

# Запустить PostgreSQL
sudo systemctl start postgresql

# Восстановление:
# 1. Остановить PostgreSQL
# 2. Удалить/переместить текущий data directory
# 3. Распаковать бэкап
# 4. Исправить права
# 5. Запустить PostgreSQL
```

**pg_basebackup - онлайн физический бэкап:**
```bash
# Базовый бэкап
pg_basebackup -U postgres -D /backup/base -F tar -z -P

# Параметры:
# -D - директория для бэкапа
# -F - формат (plain, tar)
# -z - компрессия
# -P - показывать прогресс
# -X - включить WAL файлы (stream, fetch)

# С WAL архивом
pg_basebackup -U postgres -D /backup/base -F tar -z -P -X stream

# Проверочный бэкап
pg_basebackup -U postgres -D /backup/base -F tar -z -P --checkpoint=fast
```

**Continuous Archiving (WAL archiving):**
```sql
-- Настройка в postgresql.conf
wal_level = replica                -- Уровень логирования WAL
archive_mode = on                  -- Включить архивирование
archive_command = 'cp %p /archive/%f'  -- Команда архивирования

-- Или с компрессией
archive_command = 'gzip < %p > /archive/%f.gz'

-- Проверка архивирования
SELECT * FROM pg_stat_archiver;

-- Ручное переключение WAL
SELECT pg_switch_wal();
```

**Point-in-Time Recovery (PITR):**
```bash
# 1. Восстановить базовый бэкап
tar -xzf base.tar.gz -C /var/lib/postgresql/14/main/

# 2. Создать recovery.signal (PostgreSQL 12+)
touch /var/lib/postgresql/14/main/recovery.signal

# 3. Настроить postgresql.conf или создать postgresql.auto.conf
cat > /var/lib/postgresql/14/main/postgresql.auto.conf << EOF
restore_command = 'gunzip < /archive/%f.gz > %p'
recovery_target_time = '2024-11-15 14:30:00'
recovery_target_action = 'promote'
EOF

# 4. Запустить PostgreSQL
sudo systemctl start postgresql

# PostgreSQL восстановит данные до указанного времени
```

**Репликация (Streaming Replication):**
```sql
-- На мастере (postgresql.conf):
wal_level = replica
max_wal_senders = 3
wal_keep_size = 64

-- Создать пользователя для репликации
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'password';

-- В pg_hba.conf на мастере:
host replication replicator replica_ip/32 md5

-- На реплике:
# 1. Создать базовый бэкап с мастера
pg_basebackup -h master_host -D /var/lib/postgresql/14/main -U replicator -P -X stream

# 2. Создать standby.signal
touch /var/lib/postgresql/14/main/standby.signal

# 3. Настроить postgresql.auto.conf
cat > /var/lib/postgresql/14/main/postgresql.auto.conf << EOF
primary_conninfo = 'host=master_host port=5432 user=replicator password=password'
EOF

# 4. Запустить реплику
sudo systemctl start postgresql

-- Проверка репликации на мастере:
SELECT * FROM pg_stat_replication;

-- На реплике:
SELECT pg_is_in_recovery();
```

**Автоматизация бэкапов:**
```bash
#!/bin/bash
# backup_postgres.sh

BACKUP_DIR="/backup/postgres"
DATE=$(date +%Y%m%d_%H%M%S)
DATABASE="mydb"
RETENTION_DAYS=7

# Создать директорию
mkdir -p $BACKUP_DIR

# Бэкап
pg_dump -U postgres -d $DATABASE -F c -f $BACKUP_DIR/${DATABASE}_${DATE}.dump

# Компрессия
gzip $BACKUP_DIR/${DATABASE}_${DATE}.dump

# Удалить старые бэкапы
find $BACKUP_DIR -name "${DATABASE}_*.dump.gz" -mtime +$RETENTION_DAYS -delete

# Логирование
echo "$(date): Backup completed - ${DATABASE}_${DATE}.dump.gz" >> $BACKUP_DIR/backup.log
```

### 💻 Задание

1. **Создай тестовую базу данных с данными:**
   ```sql
   CREATE DATABASE backup_test;
   \c backup_test
   
   CREATE TABLE test_data (
       id SERIAL PRIMARY KEY,
       data TEXT,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   
   INSERT INTO test_data (data)
   SELECT 'Test data ' || generate_series
   FROM generate_series(1, 1000);
   ```

2. **Сделай бэкап разными способами:**
   - Plain SQL формат
   - Custom формат
   - Только схема
   - Только одна таблица

3. **Восстанови данные:**
   - Создай новую БД `backup_test_restore`
   - Восстанови данные из бэкапа
   - Проверь, что все данные на месте

4. **Бэкап всех баз:**
   - Используй `pg_dumpall`
   - Сохрани только глобальные объекты (роли)

5. **Автоматизация:**
   - Создай скрипт бэкапа
   - Добавь в cron для ежедневного выполнения
   - Настрой ротацию старых бэкапов

### 🚀 Бонус (новое)

- Настрой WAL архивирование:
  ```sql
  -- В postgresql.conf
  wal_level = replica
  archive_mode = on
  archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'
  ```

- Создай скрипт для проверки бэкапов:
  ```bash
  #!/bin/bash
  # verify_backup.sh
  
  BACKUP_FILE=$1
  TEST_DB="backup_verify_test"
  
  # Удалить тестовую БД если существует
  psql -U postgres -c "DROP DATABASE IF EXISTS $TEST_DB"
  
  # Создать тестовую БД
  psql -U postgres -c "CREATE DATABASE $TEST_DB"
  
  # Восстановить бэкап
  if pg_restore -U postgres -d $TEST_DB $BACKUP_FILE; then
      echo "Backup verification successful"
      # Проверить количество таблиц
      TABLE_COUNT=$(psql -U postgres -d $TEST_DB -t -c "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema='public'")
      echo "Tables restored: $TABLE_COUNT"
  else
      echo "Backup verification failed"
      exit 1
  fi
  
  # Удалить тестовую БД
  psql -U postgres -c "DROP DATABASE $TEST_DB"
  ```

- Настрой простую репликацию (если есть вторая VM/контейнер):
  - Создай пользователя для репликации
  - Настрой мастер и реплику
  - Проверь репликацию

---

## Модуль 7: Пользователи, роли и безопасность (25 минут)

### 🎯 Напоминалка

**Создание ролей и пользователей:**
```sql
-- Роль = Пользователь в PostgreSQL (USER это алиас для ROLE с LOGIN)

-- Создание роли без входа
CREATE ROLE readonly;
CREATE ROLE admin;

-- Создание пользователя (роль с правом входа)
CREATE USER myuser WITH PASSWORD 'mypassword';
-- Или
CREATE ROLE myuser WITH LOGIN PASSWORD 'mypassword';

-- С дополнительными опциями
CREATE USER developer WITH 
    PASSWORD 'password'
    VALID UNTIL '2025-12-31'
    CONNECTION LIMIT 10;

-- Суперпользователь
CREATE USER admin WITH PASSWORD 'password' SUPERUSER;

-- С правами создания БД и ролей
CREATE USER dbadmin WITH PASSWORD 'password' CREATEDB CREATEROLE;

-- Изменение пароля
ALTER USER myuser WITH PASSWORD 'newpassword';

-- Изменение атрибутов
ALTER ROLE myuser WITH SUPERUSER;
ALTER ROLE myuser WITH NOSUPERUSER;
ALTER ROLE myuser WITH CREATEDB;
ALTER ROLE myuser WITH VALID UNTIL '2025-12-31';
ALTER ROLE myuser WITH CONNECTION LIMIT 5;

-- Переименование
ALTER ROLE myuser RENAME TO newusername;

-- Удаление роли
DROP ROLE myuser;
DROP ROLE IF EXISTS myuser;

-- Список ролей
\du
SELECT * FROM pg_roles;
SELECT * FROM pg_user;
```

**Права доступа (GRANT/REVOKE):**
```sql
-- Права на базу данных
GRANT CONNECT ON DATABASE mydb TO myuser;
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
REVOKE CONNECT ON DATABASE mydb FROM myuser;

-- Права на схему
GRANT USAGE ON SCHEMA public TO myuser;
GRANT CREATE ON SCHEMA public TO myuser;
GRANT ALL ON SCHEMA public TO myuser;

-- Права на таблицу
GRANT SELECT ON TABLE users TO myuser;
GRANT INSERT ON TABLE users TO myuser;
GRANT UPDATE ON TABLE users TO myuser;
GRANT DELETE ON TABLE users TO myuser;
GRANT ALL PRIVILEGES ON TABLE users TO myuser;

-- Права на все таблицы в схеме
GRANT SELECT ON ALL TABLES IN SCHEMA public TO myuser;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO myuser;

-- Права на последовательности (sequences)
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO myuser;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA public TO myuser;

-- Права на конкретные колонки
GRANT SELECT (id, username) ON TABLE users TO myuser;
GRANT UPDATE (email) ON TABLE users TO myuser;

-- Права на функции
GRANT EXECUTE ON FUNCTION my_function() TO myuser;

-- Права по умолчанию для новых объектов
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT ON TABLES TO myuser;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT USAGE ON SEQUENCES TO myuser;

-- Отзыв прав
REVOKE SELECT ON TABLE users FROM myuser;
REVOKE ALL PRIVILEGES ON TABLE users FROM myuser;
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM myuser;

-- Просмотр прав
\dp users
\z users
SELECT * FROM information_schema.table_privileges WHERE grantee = 'myuser';
```

**Группы ролей:**
```sql
-- Создать группу
CREATE ROLE readonly_group;
CREATE ROLE readwrite_group;
CREATE ROLE admin_group;

-- Назначить права группе
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_group;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO readwrite_group;

-- Добавить пользователя в группу
GRANT readonly_group TO myuser;
GRANT readwrite_group TO developer;

-- Пользователь может иметь несколько ролей
GRANT readonly_group TO myuser;
GRANT readwrite_group TO myuser;

-- Удалить пользователя из группы
REVOKE readonly_group FROM myuser;

-- Установить роль по умолчанию
ALTER ROLE myuser SET ROLE readonly_group;

-- Переключение роли в сессии
SET ROLE readonly_group;
SET ROLE developer;
RESET ROLE;  -- Вернуться к оригинальной роли

-- Просмотр членства в ролях
\du
SELECT * FROM pg_roles WHERE rolname = 'myuser';
```

**Row Level Security (RLS):**
```sql
-- Включить RLS для таблицы
ALTER TABLE articles ENABLE ROW LEVEL SECURITY;

-- Создать политику
CREATE POLICY user_articles ON articles
    FOR SELECT
    USING (author_id = current_user_id());

-- Политика для INSERT
CREATE POLICY user_insert ON articles
    FOR INSERT
    WITH CHECK (author_id = current_user_id());

-- Политика для UPDATE
CREATE POLICY user_update ON articles
    FOR UPDATE
    USING (author_id = current_user_id())
    WITH CHECK (author_id = current_user_id());

-- Политика для DELETE
CREATE POLICY user_delete ON articles
    FOR DELETE
    USING (author_id = current_user_id());

-- Политика для всех операций
CREATE POLICY user_all ON articles
    FOR ALL
    USING (author_id = current_user_id())
    WITH CHECK (author_id = current_user_id());

-- Политика для суперпользователей (видят всё)
CREATE POLICY admin_all ON articles
    FOR ALL
    TO admin_role
    USING (true);

-- Удалить политику
DROP POLICY user_articles ON articles;

-- Отключить RLS
ALTER TABLE articles DISABLE ROW LEVEL SECURITY;

-- Просмотр политик
\d+ articles
SELECT * FROM pg_policies WHERE tablename = 'articles';
```

**Аутентификация (pg_hba.conf):**
```bash
# Формат: TYPE  DATABASE  USER  ADDRESS  METHOD

# Локальные подключения (Unix socket)
local   all       all                   peer      # Доверие системной аутентификации
local   all       all                   md5       # Пароль
local   all       all                   scram-sha-256  # Современное шифрование

# IPv4 локальные
host    all       all    127.0.0.1/32   md5
host    all       all    127.0.0.1/32   scram-sha-256

# IPv6 локальные
host    all       all    ::1/128        md5

# Удаленные подключения
host    all       all    0.0.0.0/0      md5       # Все IP (небезопасно!)
host    all       all    192.168.1.0/24 md5       # Локальная сеть
host    mydb      myuser 192.168.1.100/32 md5     # Конкретный IP

# SSL подключения
hostssl all       all    0.0.0.0/0      md5

# Репликация
host    replication  replicator  192.168.1.0/24  md5

# После изменений
SELECT pg_reload_conf();
-- или
sudo systemctl reload postgresql
```

**SSL/TLS шифрование:**
```bash
# В postgresql.conf
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_ca_file = 'root.crt'

# Генерация самоподписанного сертификата
openssl req -new -x509 -days 365 -nodes -text \
    -out server.crt -keyout server.key -subj "/CN=postgres.example.com"

chmod 600 server.key
chown postgres:postgres server.key server.crt

# Подключение с SSL
psql "postgresql://user@host/db?sslmode=require"
psql "sslmode=require host=hostname dbname=mydb user=myuser"

# Режимы SSL:
# disable - без SSL
# allow - попытка SSL, fallback на незашифрованное
# prefer - попытка SSL, fallback на незашифрованное (по умолчанию)
# require - требовать SSL
# verify-ca - требовать SSL и проверять CA
# verify-full - требовать SSL и проверять hostname
```

**Аудит и логирование:**
```sql
-- В postgresql.conf

-- Логирование подключений
log_connections = on
log_disconnections = on

-- Логирование всех запросов
log_statement = 'all'  # none, ddl, mod, all

-- Логирование медленных запросов
log_min_duration_statement = 1000  # миллисекунды

-- Детальный формат логов
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '

-- Логирование DDL
log_statement = 'ddl'

-- Логирование изменений данных
log_statement = 'mod'  # INSERT, UPDATE, DELETE

-- Применить
SELECT pg_reload_conf();
```

**pgAudit расширение:**
```sql
-- Установка
CREATE EXTENSION pgaudit;

-- Настройка в postgresql.conf
shared_preload_libraries = 'pgaudit'
pgaudit.log = 'write, ddl'  # read, write, function, role, ddl, misc

-- Аудит для конкретной роли
ALTER ROLE myuser SET pgaudit.log = 'read, write';

-- Просмотр логов
SELECT * FROM pg_stat_statements;
```

### 💻 Задание

1. **Создай структуру ролей:**
   ```sql
   -- Группы ролей
   CREATE ROLE app_readonly;
   CREATE ROLE app_readwrite;
   CREATE ROLE app_admin;
   ```

2. **Настрой права для групп:**
   - `app_readonly` - только SELECT на все таблицы
   - `app_readwrite` - SELECT, INSERT, UPDATE на все таблицы
   - `app_admin` - все права

3. **Создай пользователей:**
   - `analyst` (член группы app_readonly)
   - `developer` (член группы app_readwrite)
   - `admin_user` (член группы app_admin)

4. **Протестируй права:**
   - Подключись как `analyst` и попробуй SELECT (должно работать)
   - Попробуй INSERT (должно быть запрещено)
   - Подключись как `developer` и попробуй INSERT (должно работать)

5. **Настрой Row Level Security:**
   ```sql
   -- Создай таблицу документов с владельцем
   CREATE TABLE documents (
       id SERIAL PRIMARY KEY,
       title VARCHAR(200),
       content TEXT,
       owner VARCHAR(50) DEFAULT current_user,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   
   -- Включи RLS
   ALTER TABLE documents ENABLE ROW LEVEL SECURITY;
   
   -- Создай политику (пользователи видят только свои документы)
   CREATE POLICY user_documents ON documents
       FOR ALL
       USING (owner = current_user);
   ```

6. **Проверь настройки безопасности:**
   - Просмотри `pg_hba.conf`
   - Проверь, включен ли SSL
   - Посмотри логи подключений

### 🚀 Бонус (новое)

- Создай функцию для безопасной смены пароля с проверкой сложности:
  ```sql
  CREATE OR REPLACE FUNCTION change_password(old_pass TEXT, new_pass TEXT)
  RETURNS BOOLEAN AS $
  BEGIN
      -- Проверка длины
      IF LENGTH(new_pass) < 8 THEN
          RAISE EXCEPTION 'Password must be at least 8 characters';
      END IF;
      
      -- Проверка сложности (есть цифра и буква)
      IF new_pass !~ '[0-9]' OR new_pass !~ '[a-zA-Z]' THEN
          RAISE EXCEPTION 'Password must contain letters and numbers';
      END IF;
      
      -- Здесь должна быть проверка старого пароля
      -- В реальности это делается через системные функции
      
      EXECUTE format('ALTER USER %I WITH PASSWORD %L', current_user, new_pass);
      RETURN TRUE;
  END;
  $ LANGUAGE plpgsql SECURITY DEFINER;
  ```

- Настрой автоматическую ротацию паролей:
  ```sql
  CREATE TABLE password_history (
      username VARCHAR(50),
      password_hash TEXT,
      changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );
  
  CREATE OR REPLACE FUNCTION log_password_change()
  RETURNS EVENT_TRIGGER AS $
  BEGIN
      -- Логика логирования изменения пароля
  END;
  $ LANGUAGE plpgsql;
  ```

- Создай скрипт для аудита прав доступа:
  ```sql
  -- Скрипт для вывода всех пользователей и их прав
  SELECT 
      r.rolname as username,
      ARRAY_AGG(DISTINCT b.rolname) as member_of,
      r.rolsuper as is_superuser,
      r.rolcreatedb as can_create_db,
      r.rolcreaterole as can_create_role
  FROM pg_roles r
  LEFT JOIN pg_auth_members m ON r.oid = m.member
  LEFT JOIN pg_roles b ON m.roleid = b.oid
  WHERE r.rolcanlogin = true
  GROUP BY r.rolname, r.rolsuper, r.rolcreatedb, r.rolcreaterole
  ORDER BY r.rolname;
  ```

---

## Модуль 8: Мониторинг и troubleshooting (30 минут)

### 🎯 Напоминалка

**Системные представления (System Views):**
```sql
-- Активность базы данных
SELECT * FROM pg_stat_database;
SELECT * FROM pg_stat_database WHERE datname = 'mydb';

-- Статистика таблиц
SELECT * FROM pg_stat_user_tables;
SELECT * FROM pg_stat_user_tables WHERE relname = 'users';

-- Статистика индексов
SELECT * FROM pg_stat_user_indexes;
SELECT * FROM pg_statio_user_indexes;

-- Текущие подключения и запросы
SELECT * FROM pg_stat_activity;
SELECT * FROM pg_stat_activity WHERE state = 'active';

-- Блокировки
SELECT * FROM pg_locks;

-- Репликация
SELECT * FROM pg_stat_replication;

-- Статистика выполнения запросов (требует pg_stat_statements)
SELECT * FROM pg_stat_statements;

-- WAL статистика
SELECT * FROM pg_stat_wal;

-- Прогресс VACUUM
SELECT * FROM pg_stat_progress_vacuum;

-- Прогресс создания индекса
SELECT * FROM pg_stat_progress_create_index;
```

**Размеры объектов:**
```sql
-- Размер базы данных
SELECT 
    datname,
    pg_size_pretty(pg_database_size(datname)) as size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;

-- Размеры таблиц
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as table_size,
    pg_size_pretty(pg_indexes_size(schemaname||'.'||tablename)) as indexes_size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;

-- Самые большие таблицы в БД
SELECT 
    relname as table_name,
    pg_size_pretty(pg_total_relation_size(relid)) as total_size,
    pg_size_pretty(pg_relation_size(relid)) as table_size,
    pg_size_pretty(pg_total_relation_size(relid) - pg_relation_size(relid)) as external_size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC;

-- Размеры индексов
SELECT 
    indexrelname as index_name,
    relname as table_name,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC;
```

**Производительность запросов:**
```sql
-- Включение pg_stat_statements
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- В postgresql.conf:
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.track = all

-- Самые медленные запросы
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

-- Самые частые запросы
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 10;

-- Запросы с наибольшим временем выполнения
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    (total_exec_time / sum(total_exec_time) OVER ()) * 100 as percentage
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Сброс статистики
SELECT pg_stat_statements_reset();
```

**Cache hit ratio:**
```sql
-- Эффективность кэша (должно быть > 95%)
SELECT 
    sum(heap_blks_read) as heap_read,
    sum(heap_blks_hit) as heap_hit,
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) * 100 as cache_hit_ratio
FROM pg_statio_user_tables;

-- По таблицам
SELECT 
    relname,
    heap_blks_read,
    heap_blks_hit,
    CASE 
        WHEN heap_blks_hit + heap_blks_read = 0 THEN 0
        ELSE (heap_blks_hit::float / (heap_blks_hit + heap_blks_read)) * 100
    END as cache_hit_ratio
FROM pg_statio_user_tables
WHERE heap_blks_read > 0
ORDER BY heap_blks_read DESC;

-- Кэш для индексов
SELECT 
    indexrelname,
    idx_blks_read,
    idx_blks_hit,
    CASE 
        WHEN idx_blks_hit + idx_blks_read = 0 THEN 0
        ELSE (idx_blks_hit::float / (idx_blks_hit + idx_blks_read)) * 100
    END as cache_hit_ratio
FROM pg_statio_user_indexes
WHERE idx_blks_read > 0;
```

**Мониторинг соединений:**
```sql
-- Текущие соединения по базам данных
SELECT 
    datname,
    count(*) as connections,
    max(EXTRACT(EPOCH FROM (now() - query_start))) as longest_query_seconds
FROM pg_stat_activity
GROUP BY datname
ORDER BY connections DESC;

-- Долгие запросы (> 5 минут)
SELECT 
    pid,
    now() - query_start as duration,
    usename,
    datname,
    state,
    query
FROM pg_stat_activity
WHERE state != 'idle'
    AND query_start < now() - interval '5 minutes'
ORDER BY duration DESC;

-- Idle транзакции
SELECT 
    pid,
    now() - state_change as idle_duration,
    usename,
    datname,
    query
FROM pg_stat_activity
WHERE state = 'idle in transaction'
ORDER BY idle_duration DESC;

-- Заблокированные запросы
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_query,
    blocking_activity.query AS blocking_query,
    blocked_activity.application_name AS blocked_application
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
```

**Анализ bloat (раздувание):**
```sql
-- Раздувание таблиц (приблизительно)
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
    n_dead_tup,
    n_live_tup,
    CASE 
        WHEN n_live_tup > 0 THEN round((n_dead_tup::float / n_live_tup) * 100, 2)
        ELSE 0
    END as dead_tup_ratio
FROM pg_stat_user_tables
WHERE n_dead_tup > 0
ORDER BY n_dead_tup DESC;

-- Когда последний раз был VACUUM
SELECT 
    schemaname,
    relname,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze,
    n_dead_tup
FROM pg_stat_user_tables
ORDER BY last_autovacuum NULLS FIRST;
```

**Логи и ошибки:**
```bash
# Просмотр логов PostgreSQL
sudo tail -f /var/log/postgresql/postgresql-14-main.log

# Или через journalctl
sudo journalctl -u postgresql -f

# Поиск ошибок
sudo grep ERROR /var/log/postgresql/postgresql-14-main.log

# Поиск медленных запросов
sudo grep "duration:" /var/log/postgresql/postgresql-14-main.log | grep -v "duration: 0"

# Deadlocks
sudo grep "deadlock" /var/log/postgresql/postgresql-14-main.log
```

**Системные метрики:**
```bash
# CPU и память PostgreSQL процессов
ps aux | grep postgres

# Подробная информация
top -p $(pgrep -d',' postgres)

# Открытые файлы
lsof -p $(pgrep -d',' postgres) | wc -l

# Сетевые соединения
netstat -an | grep 5432 | wc -l

# Диск I/O
iostat -x 1
```

**Готовые скрипты мониторинга:**
```sql
-- Полная статистика базы данных
CREATE OR REPLACE VIEW db_health_check AS
SELECT 
    'Database Size' as metric,
    pg_size_pretty(pg_database_size(current_database())) as value
UNION ALL
SELECT 
    'Active Connections',
    count(*)::text
FROM pg_stat_activity
WHERE state = 'active'
UNION ALL
SELECT 
    'Idle Connections',
    count(*)::text
FROM pg_stat_activity
WHERE state = 'idle'
UNION ALL
SELECT 
    'Long Running Queries (>5min)',
    count(*)::text
FROM pg_stat_activity
WHERE state != 'idle'
    AND query_start < now() - interval '5 minutes'
UNION ALL
SELECT 
    'Blocked Queries',
    count(DISTINCT blocked_locks.pid)::text
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
WHERE NOT blocked_locks.granted;

-- Использование
SELECT * FROM db_health_check;
```

### 💻 Задание

1. **Установи pg_stat_statements:**
   ```sql
   CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
   ```

2. **Создай нагрузку на БД:**
   ```sql
   -- Выполни несколько разных запросов
   SELECT COUNT(*) FROM articles;
   SELECT * FROM articles WHERE published = true LIMIT 10;
   SELECT a.*, au.name FROM articles a JOIN authors au ON a.author_id = au.id;
   ```

3. **Анализ производительности:**
   - Найди топ-5 самых медленных запросов
   - Найди топ-5 самых частых запросов
   - Проверь cache hit ratio (должен быть > 90%)

4. **Мониторинг соединений:**
   - Выведи количество активных соединений по базам данных
   - Найди все idle транзакции
   - Проверь, нет ли долгих запросов

5. **Анализ размеров:**
   - Найди топ-10 самых больших таблиц
   - Найди топ-10 самых больших индексов
   - Проверь bloat (мертвые строки) в таблицах

6. **Проверь настройки:**
   ```sql
   SHOW shared_buffers;
   SHOW work_mem;
   SHOW effective_cache_size;
   SHOW max_connections;
   ```

### 🚀 Бонус (новое)

- Создай комплексный скрипт мониторинга:
  ```bash
  #!/bin/bash
  # postgres_monitor.sh
  
  DB_NAME="mydb"
  THRESHOLD_CONNECTIONS=80
  THRESHOLD_CACHE_HIT=90
  THRESHOLD_BLOAT=20
  
  echo "=== PostgreSQL Health Check ==="
  echo "Date: $(date)"
  echo ""
  
  # Connections
  CONNECTIONS=$(psql -U postgres -d $DB_NAME -t -c "SELECT count(*) FROM pg_stat_activity WHERE datname='$DB_NAME'")
  echo "Connections: $CONNECTIONS"
  
  # Cache hit ratio
  CACHE_HIT=$(psql -U postgres -d $DB_NAME -t -c "SELECT round((sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read))) * 100, 2) FROM pg_statio_user_tables")
  echo "Cache Hit Ratio: ${CACHE_HIT}%"
  
  # Long running queries
  LONG_QUERIES=$(psql -U postgres -d $DB_NAME -t -c "SELECT count(*) FROM pg_stat_activity WHERE state != 'idle' AND query_start < now() - interval '5 minutes'")
  echo "Long Running Queries (>5min): $LONG_QUERIES"
  
  # Database size
  DB_SIZE=$(psql -U postgres -d $DB_NAME -t -c "SELECT pg_size_pretty(pg_database_size('$DB_NAME'))")
  echo "Database Size: $DB_SIZE"
  
  # Alerts
  if [ "$CONNECTIONS" -gt "$THRESHOLD_CONNECTIONS" ]; then
      echo "ALERT: Too many connections!"
  fi
  
  if [ "${CACHE_HIT%.*}" -lt "$THRESHOLD_CACHE_HIT" ]; then
      echo "ALERT: Low cache hit ratio!"
  fi
  ```

- Настрой экспорт метрик для Prometheus:
  ```bash
  # Установи postgres_exporter
  sudo apt install prometheus-postgres-exporter
  
  # Или используй готовые запросы для мониторинга
  ```

- Создай дашборд для мониторинга (если есть Grafana/Prometheus)

---

## Модуль 9: Расширенные функции и процедуры (25 минут)

### 🎯 Напоминалка

**Создание функций:**
```sql
-- Простая функция
CREATE OR REPLACE FUNCTION get_full_name(first_name TEXT, last_name TEXT)
RETURNS TEXT AS $
BEGIN
    RETURN first_name || ' ' || last_name;
END;
$ LANGUAGE plpgsql;

-- Использование
SELECT get_full_name('John', 'Doe');

-- Функция с параметрами по умолчанию
CREATE OR REPLACE FUNCTION calculate_discount(price NUMERIC, discount NUMERIC DEFAULT 0)
RETURNS NUMERIC AS $
BEGIN
    RETURN price * (1 - discount / 100);
END;
$ LANGUAGE plpgsql;

-- Функция возвращающая таблицу
CREATE OR REPLACE FUNCTION get_active_users()
RETURNS TABLE (id INTEGER, username TEXT, email TEXT) AS $
BEGIN
    RETURN QUERY 
    SELECT u.id, u.username, u.email 
    FROM users u 
    WHERE u.active = true;
END;
$ LANGUAGE plpgsql;

-- Использование
