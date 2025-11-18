# MySQL/Percona Refresh: Ежегодный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые концепции MySQL/Percona за 3-4 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый модуль состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальные задачи, которые нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

---

## Модуль 1: Подключение и базовые операции (20 минут)

### 🎯 Напоминалка

**Подключение к MySQL:**
```bash
# Локальное подключение
mysql -u username -p
mysql -u root -p database_name

# Удаленное подключение
mysql -h hostname -u username -p
mysql -h 192.168.1.100 -P 3306 -u username -p

# Подключение с выполнением команды
mysql -u root -p -e "SHOW DATABASES;"

# Подключение с файлом SQL
mysql -u root -p database_name < script.sql

# Экспорт результата в файл
mysql -u root -p -e "SELECT * FROM table" database_name > output.txt

# Неинтерактивный режим (скрипты)
mysql -u root -p"password" database_name -e "query"  # Не рекомендуется!
```

**Базовые команды в консоли MySQL:**
```sql
-- Информация о подключении
\s                              -- Status
SELECT USER();                  -- Текущий пользователь
SELECT DATABASE();              -- Текущая БД
SHOW PROCESSLIST;               -- Активные соединения

-- Работа с БД
SHOW DATABASES;
CREATE DATABASE dbname;
CREATE DATABASE IF NOT EXISTS dbname CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE dbname;
DROP DATABASE dbname;

-- Работа с таблицами
SHOW TABLES;
DESCRIBE table_name;
SHOW CREATE TABLE table_name;
SHOW TABLE STATUS;
SHOW FULL COLUMNS FROM table_name;

-- Выход
EXIT;
QUIT;
\q
```

**Основные типы данных:**
```sql
-- Числовые
TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT
DECIMAL(10,2)                   -- Точные числа
FLOAT, DOUBLE                   -- Приблизительные

-- Строковые
CHAR(10)                        -- Фиксированная длина
VARCHAR(255)                    -- Переменная длина
TEXT, MEDIUMTEXT, LONGTEXT      -- Большие тексты
ENUM('val1', 'val2')            -- Перечисление

-- Дата и время
DATE                            -- YYYY-MM-DD
TIME                            -- HH:MM:SS
DATETIME                        -- YYYY-MM-DD HH:MM:SS
TIMESTAMP                       -- Auto-updating timestamp
YEAR                            -- YYYY

-- Бинарные
BLOB, MEDIUMBLOB, LONGBLOB
BINARY, VARBINARY

-- JSON (MySQL 5.7+)
JSON                            -- Нативный JSON тип
```

**CRUD операции:**
```sql
-- CREATE (INSERT)
INSERT INTO users (name, email, created_at) 
VALUES ('John', 'john@example.com', NOW());

INSERT INTO users (name, email) 
VALUES 
  ('Alice', 'alice@example.com'),
  ('Bob', 'bob@example.com');

INSERT IGNORE INTO users ...;   -- Игнорировать дубликаты
INSERT INTO ... ON DUPLICATE KEY UPDATE ...;

-- READ (SELECT)
SELECT * FROM users;
SELECT name, email FROM users;
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE age > 18 AND status = 'active';
SELECT * FROM users ORDER BY created_at DESC;
SELECT * FROM users LIMIT 10;
SELECT * FROM users LIMIT 10 OFFSET 20;

-- UPDATE
UPDATE users SET email = 'new@example.com' WHERE id = 1;
UPDATE users SET status = 'inactive' WHERE last_login < DATE_SUB(NOW(), INTERVAL 1 YEAR);

-- DELETE
DELETE FROM users WHERE id = 1;
DELETE FROM users WHERE status = 'deleted';
TRUNCATE TABLE users;           -- Быстрая очистка таблицы

-- REPLACE (DELETE + INSERT)
REPLACE INTO users (id, name, email) VALUES (1, 'John', 'john@example.com');
```

**Конфигурационные файлы:**
```bash
# Основной конфиг
/etc/mysql/my.cnf                      # Debian/Ubuntu
/etc/my.cnf                            # RHEL/CentOS
/etc/percona-server.conf.d/            # Percona

# Пользовательский конфиг (для подключения без пароля)
~/.my.cnf
```

**Содержимое ~/.my.cnf:**
```ini
[client]
user=username
password=your_password
host=localhost
port=3306

[mysql]
database=default_db
```

### 💻 Задание

Выполни следующие задачи:

1. Подключись к MySQL и проверь версию:
   ```sql
   SELECT VERSION();
   ```

2. Создай базу данных `test_db` с кодировкой utf8mb4

3. Создай таблицу `employees`:
   ```sql
   CREATE TABLE employees (
     id INT AUTO_INCREMENT PRIMARY KEY,
     name VARCHAR(100) NOT NULL,
     email VARCHAR(100) UNIQUE,
     department VARCHAR(50),
     salary DECIMAL(10,2),
     hired_date DATE,
     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```

4. Вставь минимум 5 записей в таблицу

5. Выведи всех сотрудников отдела "Engineering" с зарплатой > 50000

6. Обнови email одного из сотрудников

7. Удали сотрудника с минимальной зарплатой

### 🚀 Бонус (новое)

- Используй `mycli` вместо обычного клиента (улучшенная CLI с автодополнением):
  ```bash
  pip install mycli
  mycli -u root
  ```
- Создай представление (VIEW) для активных сотрудников:
  ```sql
  CREATE VIEW active_employees AS 
  SELECT * FROM employees WHERE status = 'active';
  ```
- Используй транзакции для безопасной вставки данных:
  ```sql
  START TRANSACTION;
  INSERT INTO employees ...;
  -- Проверка
  ROLLBACK;  -- или COMMIT;
  ```
- Экспортируй структуру таблицы без данных:
  ```bash
  mysqldump -u root -p --no-data test_db employees > schema.sql
  ```

---

## Модуль 2: Индексы и оптимизация запросов (30 минут)

### 🎯 Напоминалка

**Типы индексов:**
```sql
-- PRIMARY KEY (уникальный, не NULL)
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  -- или
  PRIMARY KEY (id)
);

-- UNIQUE INDEX (уникальные значения)
CREATE UNIQUE INDEX idx_email ON users(email);
ALTER TABLE users ADD UNIQUE INDEX idx_email (email);

-- INDEX (обычный индекс)
CREATE INDEX idx_name ON users(name);
CREATE INDEX idx_name_email ON users(name, email);  -- Композитный

-- FULLTEXT INDEX (полнотекстовый поиск)
CREATE FULLTEXT INDEX idx_description ON products(description);

-- SPATIAL INDEX (геопространственные данные)
CREATE SPATIAL INDEX idx_location ON places(coordinates);

-- Просмотр индексов
SHOW INDEX FROM users;
SHOW KEYS FROM users;
```

**Управление индексами:**
```sql
-- Создание
CREATE INDEX idx_created_at ON users(created_at);
ALTER TABLE users ADD INDEX idx_status (status);

-- Удаление
DROP INDEX idx_name ON users;
ALTER TABLE users DROP INDEX idx_name;

-- Переименование (MySQL 5.7+)
ALTER TABLE users RENAME INDEX old_name TO new_name;

-- Проверка использования индекса
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
EXPLAIN FORMAT=JSON SELECT ...;

-- Форсировать использование индекса
SELECT * FROM users FORCE INDEX (idx_name) WHERE name = 'John';
SELECT * FROM users USE INDEX (idx_name) WHERE name = 'John';
SELECT * FROM users IGNORE INDEX (idx_name) WHERE name = 'John';
```

**EXPLAIN - анализ запросов:**
```sql
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Важные столбцы EXPLAIN:
-- type: ALL (плохо), index, range, ref, eq_ref, const (хорошо)
-- key: используемый индекс
-- rows: примерное количество строк для сканирования
-- Extra: дополнительная информация (Using filesort, Using temporary - плохо)

-- Расширенный анализ
EXPLAIN EXTENDED SELECT ...;
SHOW WARNINGS;  -- Показывает оптимизированный запрос

-- Анализ выполнения (MySQL 5.6+)
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```

**Оптимизация запросов:**
```sql
-- Используй LIMIT когда нужна часть данных
SELECT * FROM large_table LIMIT 100;

-- Выбирай только нужные столбцы
SELECT id, name FROM users;  -- Лучше чем SELECT *

-- Используй JOIN вместо подзапросов где возможно
-- Плохо:
SELECT * FROM orders WHERE user_id IN (SELECT id FROM users WHERE country = 'US');
-- Лучше:
SELECT o.* FROM orders o 
JOIN users u ON o.user_id = u.id 
WHERE u.country = 'US';

-- Используй EXISTS вместо IN для больших наборов
SELECT * FROM users u 
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- Избегай функций в WHERE (они блокируют индексы)
-- Плохо:
SELECT * FROM users WHERE YEAR(created_at) = 2024;
-- Лучше:
SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';

-- Используй UNION ALL вместо UNION если дубликаты не важны
SELECT name FROM users WHERE status = 'active'
UNION ALL
SELECT name FROM archived_users WHERE status = 'active';

-- Разбивай большие IN списки
-- Вместо IN (1,2,3,...,10000) используй временную таблицу или JOIN
```

**Статистика и анализ таблиц:**
```sql
-- Анализ таблицы (обновляет статистику)
ANALYZE TABLE users;

-- Оптимизация таблицы (дефрагментация)
OPTIMIZE TABLE users;

-- Проверка таблицы
CHECK TABLE users;

-- Ремонт таблицы (для MyISAM)
REPAIR TABLE users;

-- Размер таблиц
SELECT 
  table_name,
  ROUND(((data_length + index_length) / 1024 / 1024), 2) AS size_mb
FROM information_schema.TABLES
WHERE table_schema = 'database_name'
ORDER BY size_mb DESC;

-- Количество записей
SELECT COUNT(*) FROM users;
SELECT TABLE_ROWS FROM information_schema.TABLES 
WHERE TABLE_SCHEMA = 'database_name' AND TABLE_NAME = 'users';
```

**Профилирование запросов:**
```sql
-- Включить профилирование
SET profiling = 1;

-- Выполнить запросы
SELECT * FROM users WHERE email = 'test@example.com';

-- Посмотреть профили
SHOW PROFILES;

-- Детальная информация
SHOW PROFILE FOR QUERY 1;
SHOW PROFILE CPU, BLOCK IO FOR QUERY 1;

-- Выключить профилирование
SET profiling = 0;
```

### 💻 Задание

Используй таблицу `employees` из предыдущего модуля или создай новую с ~10,000 записей:

1. Создай индекс на столбце `department`

2. Проанализируй запрос с помощью EXPLAIN:
   ```sql
   EXPLAIN SELECT * FROM employees WHERE department = 'Engineering';
   ```

3. Найди все индексы таблицы `employees`

4. Создай композитный индекс на `(department, salary)`

5. Сравни производительность запроса до и после индекса:
   ```sql
   SELECT * FROM employees 
   WHERE department = 'Engineering' AND salary > 60000;
   ```

6. Найди 10 самых больших таблиц в базе данных

7. Оптимизируй следующий запрос (подсказка: функция в WHERE):
   ```sql
   SELECT * FROM employees 
   WHERE YEAR(hired_date) = 2023;
   ```

### 🚀 Бонус (новое)

- Используй `pt-query-digest` из Percona Toolkit для анализа slow query log:
  ```bash
  pt-query-digest /var/log/mysql/mysql-slow.log
  ```
- Создай покрывающий индекс (covering index):
  ```sql
  CREATE INDEX idx_covering ON employees(department, salary, name);
  -- Теперь запрос может получить все данные из индекса без обращения к таблице
  ```
- Используй `EXPLAIN FORMAT=JSON` для детального анализа
- Настрой slow query log для отслеживания медленных запросов:
  ```sql
  SET GLOBAL slow_query_log = 'ON';
  SET GLOBAL long_query_time = 2;
  SET GLOBAL slow_query_log_file = '/var/log/mysql/mysql-slow.log';
  ```

---

## Модуль 3: Пользователи, права и безопасность (25 минут)

### 🎯 Напоминалка

**Управление пользователями:**
```sql
-- Создание пользователя
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
CREATE USER 'username'@'%' IDENTIFIED BY 'password';  -- Любой хост
CREATE USER 'username'@'192.168.1.%' IDENTIFIED BY 'password';  -- Подсеть

-- Изменение пароля
ALTER USER 'username'@'localhost' IDENTIFIED BY 'new_password';
SET PASSWORD FOR 'username'@'localhost' = PASSWORD('new_password');

-- Для текущего пользователя
SET PASSWORD = PASSWORD('new_password');

-- Удаление пользователя
DROP USER 'username'@'localhost';

-- Список пользователей
SELECT user, host, authentication_string FROM mysql.user;
SELECT user, host FROM mysql.user;

-- Переименование
RENAME USER 'old_name'@'localhost' TO 'new_name'@'localhost';
```

**Управление правами:**
```sql
-- GRANT - выдача прав
-- Все права на все БД
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost';

-- Все права на конкретную БД
GRANT ALL PRIVILEGES ON database_name.* TO 'user'@'localhost';

-- Конкретные права на таблицу
GRANT SELECT, INSERT, UPDATE ON database_name.table_name TO 'user'@'localhost';

-- Только чтение
GRANT SELECT ON database_name.* TO 'readonly'@'localhost';

-- С правом передачи прав другим
GRANT SELECT ON database_name.* TO 'user'@'localhost' WITH GRANT OPTION;

-- REVOKE - отзыв прав
REVOKE INSERT, UPDATE ON database_name.* FROM 'user'@'localhost';
REVOKE ALL PRIVILEGES ON database_name.* FROM 'user'@'localhost';

-- Применить изменения
FLUSH PRIVILEGES;

-- Просмотр прав
SHOW GRANTS FOR 'username'@'localhost';
SHOW GRANTS FOR CURRENT_USER;
```

**Типы привилегий:**
```sql
-- На уровне сервера
ALL PRIVILEGES          -- Все права
CREATE USER             -- Создание пользователей
RELOAD                  -- FLUSH операции
SHUTDOWN                -- Остановка сервера
SUPER                   -- Административные операции
REPLICATION SLAVE       -- Репликация
REPLICATION CLIENT      -- Просмотр статуса репликации

-- На уровне БД/таблицы
SELECT, INSERT, UPDATE, DELETE
CREATE, DROP, ALTER
INDEX                   -- Создание/удаление индексов
REFERENCES              -- Внешние ключи
CREATE TEMPORARY TABLES
LOCK TABLES
EXECUTE                 -- Выполнение процедур

-- На уровне столбца
SELECT (column1, column2)
INSERT (column1, column2)
UPDATE (column1, column2)
```

**Роли (MySQL 8.0+):**
```sql
-- Создание роли
CREATE ROLE 'app_read', 'app_write';

-- Выдача прав роли
GRANT SELECT ON app_db.* TO 'app_read';
GRANT INSERT, UPDATE, DELETE ON app_db.* TO 'app_write';

-- Назначение ролей пользователям
GRANT 'app_read' TO 'user1'@'localhost';
GRANT 'app_read', 'app_write' TO 'user2'@'localhost';

-- Активация ролей
SET DEFAULT ROLE ALL TO 'user1'@'localhost';

-- Просмотр ролей
SHOW GRANTS FOR 'user1'@'localhost' USING 'app_read';
```

**Безопасность - лучшие практики:**
```sql
-- 1. Удалить анонимных пользователей
DELETE FROM mysql.user WHERE User='';

-- 2. Запретить удаленный root login
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1');

-- 3. Удалить тестовую БД
DROP DATABASE IF EXISTS test;

-- 4. Использовать сильные пароли
-- Проверка политики паролей (MySQL 5.7+)
SHOW VARIABLES LIKE 'validate_password%';

-- 5. Включить SSL для подключений
-- В my.cnf:
[mysqld]
require_secure_transport=ON

-- Для конкретного пользователя
ALTER USER 'username'@'%' REQUIRE SSL;

-- 6. Оптимизировать таблицу
OPTIMIZE TABLE users;

-- 7. Проверить блокировки
SHOW OPEN TABLES WHERE In_use > 0;
SELECT * FROM information_schema.INNODB_TRX;
SELECT * FROM information_schema.INNODB_LOCKS;
SELECT * FROM information_schema.INNODB_LOCK_WAITS;

-- 8. Убить блокирующий процесс
SELECT r.trx_id waiting_trx_id, r.trx_mysql_thread_id waiting_thread,
       b.trx_id blocking_trx_id, b.trx_mysql_thread_id blocking_thread
FROM information_schema.INNODB_LOCK_WAITS w
INNER JOIN information_schema.INNODB_TRX b ON b.trx_id = w.blocking_trx_id
INNER JOIN information_schema.INNODB_TRX r ON r.trx_id = w.requesting_trx_id;

KILL blocking_thread;
```

**Проблема: Высокое использование CPU**
```sql
-- 1. Найти активные запросы
SHOW FULL PROCESSLIST;

-- 2. Найти долгие запросы
SELECT * FROM information_schema.PROCESSLIST 
WHERE TIME > 5 AND COMMAND != 'Sleep'
ORDER BY TIME DESC;

-- 3. Проверить Performance Schema
SELECT DIGEST_TEXT, COUNT_STAR, AVG_TIMER_WAIT/1000000000000 AS avg_sec
FROM performance_schema.events_statements_summary_by_digest
ORDER BY AVG_TIMER_WAIT DESC LIMIT 10;

-- 4. Проверить отсутствие индексов
SELECT * FROM sys.statements_with_full_table_scans;

-- 5. Убить проблемный запрос
KILL QUERY process_id;
KILL process_id;
```

**Проблема: Высокое использование памяти**
```sql
-- 1. Проверить InnoDB Buffer Pool
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SHOW STATUS LIKE 'Innodb_buffer_pool_pages%';

-- 2. Проверить временные таблицы
SHOW STATUS LIKE 'Created_tmp_disk_tables';
SHOW STATUS LIKE 'Created_tmp_tables';

-- 3. Проверить количество соединений
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';

-- 4. Проверить использование памяти по пользователям
SELECT * FROM sys.memory_by_user_by_current_bytes;

-- 5. Проверить глобальное использование
SELECT * FROM sys.memory_global_total;

-- 6. Найти большие временные таблицы в процессах
SELECT * FROM information_schema.PROCESSLIST 
WHERE STATE LIKE '%tmp%' OR STATE LIKE '%Sorting%';
```

**Проблема: Высокое использование диска**
```bash
# 1. Проверить размер баз данных
mysql -e "SELECT table_schema AS 'Database', 
ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.TABLES
GROUP BY table_schema
ORDER BY SUM(data_length + index_length) DESC;"

# 2. Проверить размер таблиц
mysql -e "SELECT table_name AS 'Table',
ROUND(((data_length + index_length) / 1024 / 1024), 2) AS 'Size (MB)'
FROM information_schema.TABLES
WHERE table_schema = 'database_name'
ORDER BY (data_length + index_length) DESC
LIMIT 20;"

# 3. Проверить binary logs
ls -lh /var/log/mysql/mysql-bin.*
du -sh /var/log/mysql/

# 4. Очистить старые binary logs
mysql -e "PURGE BINARY LOGS BEFORE DATE_SUB(NOW(), INTERVAL 3 DAY);"

# 5. Проверить relay logs (на слейвах)
ls -lh /var/log/mysql/relay-bin.*

# 6. Проверить ibdata файл
ls -lh /var/lib/mysql/ibdata1

# 7. Очистить общий лог (если включен)
> /var/log/mysql/general.log

# 8. Удалить старые slow query logs
find /var/log/mysql/ -name "mysql-slow.log.*" -mtime +7 -delete
```

**Проблема: "Too many connections"**
```sql
-- 1. Проверить текущие соединения
SHOW STATUS LIKE 'Threads_connected';
SHOW VARIABLES LIKE 'max_connections';

-- 2. Посмотреть откуда соединения
SELECT user, host, COUNT(*) as connections
FROM information_schema.PROCESSLIST
GROUP BY user, host
ORDER BY connections DESC;

-- 3. Увеличить лимит (временно)
SET GLOBAL max_connections = 500;

-- 4. Найти "спящие" соединения
SELECT * FROM information_schema.PROCESSLIST 
WHERE COMMAND = 'Sleep' 
ORDER BY TIME DESC;

-- 5. Убить старые соединения
-- Осторожно! Проверь перед выполнением
SELECT CONCAT('KILL ', id, ';') 
FROM information_schema.PROCESSLIST 
WHERE COMMAND = 'Sleep' AND TIME > 300;

-- 6. Настроить wait_timeout (my.cnf)
[mysqld]
wait_timeout = 300
interactive_timeout = 300

-- 7. Использовать connection pooling на уровне приложения
```

**Проблема: Deadlock**
```sql
-- 1. Посмотреть последний deadlock
SHOW ENGINE INNODB STATUS\G
-- Найти секцию "LATEST DETECTED DEADLOCK"

-- 2. Включить логирование всех deadlocks (Percona Server)
SET GLOBAL innodb_print_all_deadlocks = ON;

-- 3. Проверить текущие блокировки
SELECT * FROM information_schema.INNODB_LOCKS;
SELECT * FROM information_schema.INNODB_LOCK_WAITS;

-- 4. Проверить транзакции
SELECT * FROM information_schema.INNODB_TRX;

-- Предотвращение deadlocks:
-- - Всегда обращаться к таблицам в одном порядке
-- - Использовать короткие транзакции
-- - Использовать правильные индексы
-- - Использовать меньший уровень изоляции если возможно
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

**Проблема: Репликация отстает**
```sql
-- 1. Проверить задержку
SHOW SLAVE STATUS\G
-- Смотреть Seconds_Behind_Master

-- 2. Проверить ошибки
-- Last_IO_Error, Last_SQL_Error

-- 3. Проверить нагрузку на слейв
SHOW PROCESSLIST;

-- 4. Проверить parallel replication (MySQL 5.7+)
SHOW VARIABLES LIKE 'slave_parallel_workers';
SET GLOBAL slave_parallel_workers = 4;

-- 5. Пропустить ошибку (осторожно!)
STOP SLAVE SQL_THREAD;
SET GLOBAL sql_slave_skip_counter = 1;
START SLAVE SQL_THREAD;

-- 6. Использовать pt-slave-restart (Percona Toolkit)
pt-slave-restart --error-numbers=1062,1032

-- 7. Перестроить слейв если сильно отстал
# На мастере:
mysqldump --master-data=2 --single-transaction --all-databases > dump.sql
# На слейве:
mysql < dump.sql
CHANGE MASTER TO ... # из dump.sql
START SLAVE;
```

**Проблема: Поврежденная таблица**
```sql
-- 1. Проверить таблицу
CHECK TABLE table_name;

-- 2. Для InnoDB - проверить в логе
-- Посмотреть /var/log/mysql/error.log

-- 3. Попробовать восстановить
REPAIR TABLE table_name;  -- Только для MyISAM

-- 4. Для InnoDB - dump и restore
mysqldump database_name table_name > table_backup.sql
DROP TABLE table_name;
mysql database_name < table_backup.sql

-- 5. Или ALTER TABLE (пересоздает таблицу)
ALTER TABLE table_name ENGINE=InnoDB;

-- 6. Для MyISAM можно использовать myisamchk
# Остановить MySQL
sudo systemctl stop mysql
# Восстановить
sudo myisamchk --recover /var/lib/mysql/database/table.MYI
sudo systemctl start mysql
```

**Проблема: InnoDB не запускается**
```bash
# 1. Проверить error log
sudo tail -100 /var/log/mysql/error.log

# 2. Попробовать восстановление (my.cnf)
[mysqld]
innodb_force_recovery = 1  # Начать с 1, максимум 6

# Перезапустить MySQL
sudo systemctl restart mysql

# 3. Если получилось - сделать дамп
mysqldump --all-databases > full_backup.sql

# 4. Удалить и пересоздать InnoDB
sudo systemctl stop mysql
sudo rm /var/lib/mysql/ibdata1
sudo rm /var/lib/mysql/ib_logfile*
sudo systemctl start mysql

# 5. Восстановить данные
mysql < full_backup.sql

# 6. Убрать innodb_force_recovery из my.cnf
# 7. Перезапустить
sudo systemctl restart mysql
```

**Полезные команды для диагностики:**
```bash
# Мониторинг в реальном времени
watch -n 1 'mysql -e "SHOW PROCESSLIST"'
watch -n 1 'mysql -e "SHOW STATUS LIKE \"Threads%\""'

# Проверка системных ресурсов
top -u mysql
htop -u mysql
iotop -o  # Смотреть I/O

# Сетевые соединения
netstat -an | grep 3306 | wc -l
ss -tan | grep 3306 | wc -l

# Лог файлы
tail -f /var/log/mysql/error.log
tail -f /var/log/mysql/mysql-slow.log

# Percona Toolkit для диагностики
pt-stalk --run-time=30 --sleep=30 --threshold=100  # Собирает диагностику
pt-summary  # Системная информация
pt-mysql-summary  # MySQL информация
```

### 💻 Задание

Симулируй и реши проблемы:

1. Создай "медленный" запрос (без индекса) и найди его:
   ```sql
   -- Создай таблицу без индексов
   CREATE TABLE slow_table (id INT, data VARCHAR(100));
   -- Вставь много записей
   INSERT INTO slow_table SELECT seq, CONCAT('data', seq) 
   FROM seq_1_to_10000;
   -- Выполни медленный запрос
   SELECT * FROM slow_table WHERE data = 'data5000';
   -- Найди его в PROCESSLIST или Performance Schema
   ```

2. Проверь текущие блокировки:
   ```sql
   SELECT * FROM information_schema.INNODB_TRX;
   ```

3. Найди все таблицы без первичного ключа:
   ```sql
   SELECT tables.table_schema, tables.table_name
   FROM information_schema.tables
   LEFT JOIN information_schema.table_constraints constraints
     ON tables.table_schema = constraints.table_schema
     AND tables.table_name = constraints.table_name
     AND constraints.constraint_type = 'PRIMARY KEY'
   WHERE tables.table_schema NOT IN ('information_schema', 'mysql', 'performance_schema', 'sys')
     AND constraints.constraint_name IS NULL;
   ```

4. Проверь размер binary logs и рассчитай сколько места они занимают

5. Найди все временные таблицы на диске за последний час:
   ```sql
   SHOW STATUS LIKE 'Created_tmp_disk_tables';
   ```

6. Посмотри последние ошибки в error log:
   ```bash
   sudo tail -50 /var/log/mysql/error.log | grep -i error
   ```

### 🚀 Бонус (новое)

- Используй `pt-stalk` для автоматического сбора диагностики при высокой нагрузке:
  ```bash
  pt-stalk --function=status --variable=Threads_running --threshold=25 --cycles=5
  ```
- Создай скрипт мониторинга который проверяет:
  - Connections > 80%
  - Replication lag > 60 seconds
  - Slow queries > 100/hour
  - Disk usage > 85%
  И отправляет алерты
- Настрой автоматическую ротацию slow query log:
  ```bash
  # /etc/logrotate.d/mysql-slow
  /var/log/mysql/mysql-slow.log {
    daily
    rotate 7
    missingok
    compress
    delaycompress
    postrotate
      mysql -e "FLUSH SLOW LOGS"
    endscript
  }
  ```

---

## Модуль 8: Продвинутые возможности (30 минут)

### 🎯 Напоминалка

**Хранимые процедуры:**
```sql
-- Создание процедуры
DELIMITER $
CREATE PROCEDURE GetUserById(IN user_id INT)
BEGIN
  SELECT * FROM users WHERE id = user_id;
END$
DELIMITER ;

-- Вызов
CALL GetUserById(1);

-- С OUT параметром
DELIMITER $
CREATE PROCEDURE GetUserCount(OUT total INT)
BEGIN
  SELECT COUNT(*) INTO total FROM users;
END$
DELIMITER ;

-- Вызов
CALL GetUserCount(@count);
SELECT @count;

-- С условиями и циклами
DELIMITER $
CREATE PROCEDURE ProcessUsers()
BEGIN
  DECLARE done INT DEFAULT FALSE;
  DECLARE user_id INT;
  DECLARE cur CURSOR FOR SELECT id FROM users;
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
  
  OPEN cur;
  
  read_loop: LOOP
    FETCH cur INTO user_id;
    IF done THEN
      LEAVE read_loop;
    END IF;
    
    -- Обработка
    UPDATE users SET processed = 1 WHERE id = user_id;
  END LOOP;
  
  CLOSE cur;
END$
DELIMITER ;

-- Просмотр процедур
SHOW PROCEDURE STATUS WHERE Db = 'database_name';
SHOW CREATE PROCEDURE GetUserById;

-- Удаление
DROP PROCEDURE IF EXISTS GetUserById;
```

**Функции:**
```sql
-- Создание функции
DELIMITER $
CREATE FUNCTION GetUserEmail(user_id INT) RETURNS VARCHAR(100)
DETERMINISTIC
BEGIN
  DECLARE email VARCHAR(100);
  SELECT users.email INTO email FROM users WHERE id = user_id;
  RETURN email;
END$
DELIMITER ;

-- Использование
SELECT GetUserEmail(1);
SELECT id, name, GetUserEmail(id) as email FROM orders;

-- Просмотр функций
SHOW FUNCTION STATUS WHERE Db = 'database_name';
SHOW CREATE FUNCTION GetUserEmail;

-- Удаление
DROP FUNCTION IF EXISTS GetUserEmail;
```

**Триггеры:**
```sql
-- BEFORE INSERT триггер
DELIMITER $
CREATE TRIGGER users_before_insert
BEFORE INSERT ON users
FOR EACH ROW
BEGIN
  SET NEW.created_at = NOW();
  SET NEW.updated_at = NOW();
END$
DELIMITER ;

-- AFTER INSERT триггер (аудит)
DELIMITER $
CREATE TRIGGER users_after_insert
AFTER INSERT ON users
FOR EACH ROW
BEGIN
  INSERT INTO audit_log (table_name, action, record_id, timestamp)
  VALUES ('users', 'INSERT', NEW.id, NOW());
END$
DELIMITER ;

-- BEFORE UPDATE триггер
DELIMITER $
CREATE TRIGGER users_before_update
BEFORE UPDATE ON users
FOR EACH ROW
BEGIN
  SET NEW.updated_at = NOW();
END$
DELIMITER ;

-- AFTER DELETE триггер
DELIMITER $
CREATE TRIGGER users_after_delete
AFTER DELETE ON users
FOR EACH ROW
BEGIN
  INSERT INTO deleted_users SELECT OLD.*, NOW();
END$
DELIMITER ;

-- Просмотр триггеров
SHOW TRIGGERS FROM database_name;
SHOW CREATE TRIGGER users_before_insert;

-- Удаление
DROP TRIGGER IF EXISTS users_before_insert;
```

**События (Event Scheduler):**
```sql
-- Включить Event Scheduler
SET GLOBAL event_scheduler = ON;

-- Проверить статус
SHOW VARIABLES LIKE 'event_scheduler';
SHOW PROCESSLIST;  -- Должен быть процесс "Event Scheduler"

-- Создать одноразовое событие
CREATE EVENT cleanup_old_logs
ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 HOUR
DO
  DELETE FROM logs WHERE created_at < DATE_SUB(NOW(), INTERVAL 30 DAY);

-- Создать повторяющееся событие
CREATE EVENT daily_cleanup
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_TIMESTAMP
DO
  DELETE FROM logs WHERE created_at < DATE_SUB(NOW(), INTERVAL 30 DAY);

-- С временем начала и окончания
CREATE EVENT weekly_report
ON SCHEDULE EVERY 1 WEEK
STARTS '2024-01-01 00:00:00'
ENDS '2024-12-31 23:59:59'
DO
  CALL generate_weekly_report();

-- Просмотр событий
SHOW EVENTS FROM database_name;
SHOW CREATE EVENT daily_cleanup;

-- Изменение
ALTER EVENT daily_cleanup DISABLE;
ALTER EVENT daily_cleanup ENABLE;

-- Удаление
DROP EVENT IF EXISTS daily_cleanup;
```

**Партиционирование:**
```sql
-- RANGE партиционирование (по дате)
CREATE TABLE orders (
  id INT NOT NULL,
  order_date DATE NOT NULL,
  amount DECIMAL(10,2),
  PRIMARY KEY (id, order_date)
)
PARTITION BY RANGE (YEAR(order_date)) (
  PARTITION p2022 VALUES LESS THAN (2023),
  PARTITION p2023 VALUES LESS THAN (2024),
  PARTITION p2024 VALUES LESS THAN (2025),
  PARTITION pfuture VALUES LESS THAN MAXVALUE
);

-- LIST партиционирование
CREATE TABLE customers (
  id INT NOT NULL,
  country VARCHAR(50),
  name VARCHAR(100),
  PRIMARY KEY (id, country)
)
PARTITION BY LIST COLUMNS(country) (
  PARTITION p_us VALUES IN ('USA', 'Canada', 'Mexico'),
  PARTITION p_eu VALUES IN ('Germany', 'France', 'UK'),
  PARTITION p_asia VALUES IN ('China', 'Japan', 'India'),
  PARTITION p_other VALUES IN (DEFAULT)
);

-- HASH партиционирование
CREATE TABLE logs (
  id INT NOT NULL,
  message TEXT,
  PRIMARY KEY (id)
)
PARTITION BY HASH(id)
PARTITIONS 4;

-- Просмотр партиций
SELECT 
  TABLE_NAME,
  PARTITION_NAME,
  PARTITION_METHOD,
  TABLE_ROWS
FROM information_schema.PARTITIONS
WHERE TABLE_SCHEMA = 'database_name'
  AND TABLE_NAME = 'orders';

-- Добавление партиции
ALTER TABLE orders ADD PARTITION (
  PARTITION p2025 VALUES LESS THAN (2026)
);

-- Удаление партиции (и данных!)
ALTER TABLE orders DROP PARTITION p2022;

-- Усечение партиции (очистка данных)
ALTER TABLE orders TRUNCATE PARTITION p2023;

-- Обмен партиции с таблицей
CREATE TABLE orders_archive LIKE orders;
ALTER TABLE orders EXCHANGE PARTITION p2022 WITH TABLE orders_archive;
```

**JSON тип данных (MySQL 5.7+):**
```sql
-- Создание таблицы с JSON
CREATE TABLE products (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  attributes JSON
);

-- Вставка JSON
INSERT INTO products (id, name, attributes) VALUES
(1, 'Laptop', '{"brand": "Dell", "ram": "16GB", "cpu": "i7"}'),
(2, 'Phone', '{"brand": "Apple", "storage": "128GB", "color": "Black"}');

-- Извлечение значений
SELECT id, name, 
  attributes->'$.brand' as brand,
  attributes->>'$.brand' as brand_text,  -- Без кавычек
  JSON_EXTRACT(attributes, '$.ram') as ram
FROM products;

-- Поиск по JSON
SELECT * FROM products 
WHERE attributes->'$.brand' = '"Dell"';

SELECT * FROM products 
WHERE JSON_EXTRACT(attributes, '$.ram') = '"16GB"';

-- Обновление JSON
UPDATE products 
SET attributes = JSON_SET(attributes, '$.warranty', '2 years')
WHERE id = 1;

UPDATE products 
SET attributes = JSON_INSERT(attributes, '$.new_field', 'value')
WHERE id = 1;

UPDATE products 
SET attributes = JSON_REMOVE(attributes, '$.color')
WHERE id = 2;

-- JSON функции
SELECT JSON_KEYS(attributes) FROM products;
SELECT JSON_LENGTH(attributes) FROM products;
SELECT JSON_TYPE(attributes->'$.brand') FROM products;
SELECT JSON_VALID('{"key": "value"}');  -- 1 if valid

-- JSON массивы
CREATE TABLE users (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  tags JSON
);

INSERT INTO users VALUES 
(1, 'John', '["admin", "developer", "manager"]');

-- Поиск в массиве
SELECT * FROM users 
WHERE JSON_CONTAINS(tags, '"admin"');

-- Индексирование JSON (виртуальный столбец)
ALTER TABLE products 
ADD COLUMN brand VARCHAR(50) AS (attributes->>'$.brand') STORED;

CREATE INDEX idx_brand ON products(brand);
```

**Full-Text Search:**
```sql
-- Создание FULLTEXT индекса
CREATE TABLE articles (
  id INT PRIMARY KEY,
  title VARCHAR(200),
  content TEXT,
  FULLTEXT (title, content)
);

-- Или добавить к существующей таблице
ALTER TABLE articles ADD FULLTEXT(title, content);

-- Поиск в натуральном режиме
SELECT * FROM articles 
WHERE MATCH(title, content) AGAINST('mysql database');

-- Поиск в булевом режиме
SELECT * FROM articles 
WHERE MATCH(title, content) AGAINST('+mysql -oracle' IN BOOLEAN MODE);

-- Операторы Boolean mode:
-- + обязательное слово
-- - исключить слово
-- > увеличить релевантность
-- < уменьшить релевантность
-- () группировка
-- * wildcard
-- "" точная фраза

-- С релевантностью
SELECT id, title, 
  MATCH(title, content) AGAINST('mysql database') AS relevance
FROM articles
WHERE MATCH(title, content) AGAINST('mysql database')
ORDER BY relevance DESC;

-- Query Expansion (находит связанные термины)
SELECT * FROM articles 
WHERE MATCH(title, content) 
AGAINST('database' WITH QUERY EXPANSION);

-- Настройка (my.cnf)
[mysqld]
ft_min_word_len = 3
ft_max_word_len = 84
innodb_ft_min_token_size = 3
innodb_ft_max_token_size = 84
```

**Общие табличные выражения - CTE (MySQL 8.0+):**
```sql
-- Простой CTE
WITH user_orders AS (
  SELECT user_id, COUNT(*) as order_count
  FROM orders
  GROUP BY user_id
)
SELECT u.name, uo.order_count
FROM users u
JOIN user_orders uo ON u.id = uo.user_id
WHERE uo.order_count > 10;

-- Рекурсивный CTE (например, дерево категорий)
WITH RECURSIVE category_tree AS (
  -- Anchor member: корневые категории
  SELECT id, name, parent_id, 1 as level
  FROM categories
  WHERE parent_id IS NULL
  
  UNION ALL
  
  -- Recursive member
  SELECT c.id, c.name, c.parent_id, ct.level + 1
  FROM categories c
  JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree
ORDER BY level, name;

-- Числовая последовательность (генерация данных)
WITH RECURSIVE numbers AS (
  SELECT 1 as n
  UNION ALL
  SELECT n + 1 FROM numbers WHERE n < 100
)
SELECT * FROM numbers;
```

**Window Functions (MySQL 8.0+):**
```sql
-- ROW_NUMBER - нумерация строк
SELECT 
  name,
  department,
  salary,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as rank
FROM employees;

-- RANK и DENSE_RANK
SELECT 
  name,
  salary,
  RANK() OVER (ORDER BY salary DESC) as rank,
  DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank
FROM employees;

-- NTILE - разбить на N групп
SELECT 
  name,
  salary,
  NTILE(4) OVER (ORDER BY salary) as quartile
FROM employees;

-- LAG и LEAD - предыдущее/следующее значение
SELECT 
  order_date,
  amount,
  LAG(amount, 1) OVER (ORDER BY order_date) as prev_amount,
  LEAD(amount, 1) OVER (ORDER BY order_date) as next_amount
FROM orders;

-- FIRST_VALUE и LAST_VALUE
SELECT 
  name,
  department,
  salary,
  FIRST_VALUE(salary) OVER (PARTITION BY department ORDER BY salary DESC) as highest_salary,
  LAST_VALUE(salary) OVER (PARTITION BY department ORDER BY salary DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) as lowest_salary
FROM employees;

-- Агрегатные функции как оконные
SELECT 
  order_date,
  amount,
  SUM(amount) OVER (ORDER BY order_date) as running_total,
  AVG(amount) OVER (ORDER BY order_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as moving_avg_7days
FROM orders;
```

### 💻 Задание

Практика продвинутых возможностей:

1. Создай хранимую процедуру, которая принимает department и возвращает количество сотрудников:
   ```sql
   DELIMITER $
   CREATE PROCEDURE GetEmployeeCountByDept(IN dept VARCHAR(50), OUT emp_count INT)
   BEGIN
     SELECT COUNT(*) INTO emp_count FROM employees WHERE department = dept;
   END$
   DELIMITER ;
   ```

2. Создай триггер, который автоматически обновляет поле `updated_at` при изменении записи

3. Создай событие, которое каждый день в полночь удаляет записи старше 30 дней из таблицы логов

4. Добавь JSON поле в таблицу и вставь несколько записей с JSON данными

5. Создай FULLTEXT индекс на текстовом поле и выполни полнотекстовый поиск

6. Если MySQL 8.0+: Создай CTE запрос, который находит топ-5 сотрудников по зарплате в каждом департаменте

### 🚀 Бонус (новое)

- Создай рекурсивный CTE для построения дерева (например, структура компании или категорий)
- Используй Window Functions для расчета running total (накопительная сумма продаж)
- Создай партиционированную таблицу для логов по месяцам
- Напиши процедуру, которая делает архивацию старых данных (перенос в архивную таблицу)
- Создай JSON-based конфигурационную систему с валидацией

---

## Финальный проект (60 минут)

### Задача: Система мониторинга и управления MySQL

Создай комплексное решение для управления MySQL сервером, которое включает:

### Требования:

**1. Структура базы данных:**
```sql
CREATE DATABASE mysql_monitoring;
USE mysql_monitoring;

-- Таблица для хранения метрик
CREATE TABLE metrics_history (
  id INT AUTO_INCREMENT PRIMARY KEY,
  metric_name VARCHAR(100),
  metric_value DECIMAL(15,2),
  collected_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_status_time (status, started_at)
);
```

**2. Хранимые процедуры для мониторинга:**
```sql
-- Процедура сбора метрик
DELIMITER $
CREATE PROCEDURE CollectMetrics()
BEGIN
  -- CPU-подобные метрики
  INSERT INTO metrics_history (metric_name, metric_value)
  SELECT 'threads_connected', VARIABLE_VALUE 
  FROM performance_schema.global_status 
  WHERE VARIABLE_NAME = 'Threads_connected';
  
  INSERT INTO metrics_history (metric_name, metric_value)
  SELECT 'slow_queries', VARIABLE_VALUE 
  FROM performance_schema.global_status 
  WHERE VARIABLE_NAME = 'Slow_queries';
  
  -- Buffer pool hit rate
  INSERT INTO metrics_history (metric_name, metric_value)
  SELECT 'buffer_pool_hit_rate',
    ROUND(100 * (1 - read_requests / (read_requests + reads)), 2)
  FROM (
    SELECT 
      SUM(IF(VARIABLE_NAME = 'Innodb_buffer_pool_read_requests', VARIABLE_VALUE, 0)) as read_requests,
      SUM(IF(VARIABLE_NAME = 'Innodb_buffer_pool_reads', VARIABLE_VALUE, 0)) as reads
    FROM performance_schema.global_status
    WHERE VARIABLE_NAME IN ('Innodb_buffer_pool_read_requests', 'Innodb_buffer_pool_reads')
  ) stats;
END$

-- Процедура проверки здоровья системы
CREATE PROCEDURE CheckSystemHealth()
BEGIN
  DECLARE conn_usage DECIMAL(5,2);
  DECLARE slow_query_count INT;
  DECLARE repl_lag INT;
  
  -- Проверка использования соединений
  SELECT 
    ROUND(100 * threads_connected / max_conn, 2),
    threads_connected
  INTO conn_usage, @threads_connected
  FROM (
    SELECT VARIABLE_VALUE as threads_connected 
    FROM performance_schema.global_status 
    WHERE VARIABLE_NAME = 'Threads_connected'
  ) t,
  (
    SELECT VARIABLE_VALUE as max_conn 
    FROM performance_schema.global_variables 
    WHERE VARIABLE_NAME = 'max_connections'
  ) m;
  
  IF conn_usage > 80 THEN
    INSERT INTO alerts (alert_type, severity, message, details)
    VALUES (
      'high_connections',
      'warning',
      'Connection usage is high',
      JSON_OBJECT('usage_percent', conn_usage, 'current', @threads_connected)
    );
  END IF;
  
  -- Проверка медленных запросов за последний час
  SELECT CAST(VARIABLE_VALUE AS UNSIGNED) INTO slow_query_count
  FROM performance_schema.global_status
  WHERE VARIABLE_NAME = 'Slow_queries';
  
  IF slow_query_count > 100 THEN
    INSERT INTO alerts (alert_type, severity, message, details)
    VALUES (
      'high_slow_queries',
      'warning',
      'High number of slow queries',
      JSON_OBJECT('count', slow_query_count)
    );
  END IF;
  
  -- Проверка репликации (если настроена)
  SELECT COUNT(*) INTO @has_slave
  FROM performance_schema.replication_connection_status;
  
  IF @has_slave > 0 THEN
    -- Проверить задержку репликации
    -- Здесь можно добавить проверку Seconds_Behind_Master
    NULL;
  END IF;
END$

-- Процедура генерации отчета
CREATE PROCEDURE GenerateHealthReport()
BEGIN
  -- Общая информация
  SELECT 
    'System Info' as section,
    JSON_OBJECT(
      'version', @@version,
      'uptime_hours', ROUND(VARIABLE_VALUE / 3600, 2),
      'max_connections', (SELECT VARIABLE_VALUE FROM performance_schema.global_variables WHERE VARIABLE_NAME = 'max_connections')
    ) as data
  FROM performance_schema.global_status
  WHERE VARIABLE_NAME = 'Uptime'
  
  UNION ALL
  
  -- Использование соединений
  SELECT 
    'Connections' as section,
    JSON_OBJECT(
      'current', threads.VARIABLE_VALUE,
      'max', max_conn.VARIABLE_VALUE,
      'usage_percent', ROUND(100 * threads.VARIABLE_VALUE / max_conn.VARIABLE_VALUE, 2)
    )
  FROM 
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME = 'Threads_connected') threads,
    (SELECT VARIABLE_VALUE FROM performance_schema.global_variables WHERE VARIABLE_NAME = 'max_connections') max_conn
  
  UNION ALL
  
  -- Топ-5 самых медленных запросов
  SELECT 
    'Slow Queries' as section,
    JSON_ARRAYAGG(
      JSON_OBJECT(
        'query', SUBSTRING(DIGEST_TEXT, 1, 100),
        'avg_time_sec', ROUND(AVG_TIMER_WAIT/1000000000000, 2),
        'count', COUNT_STAR
      )
    )
  FROM (
    SELECT DIGEST_TEXT, AVG_TIMER_WAIT, COUNT_STAR
    FROM performance_schema.events_statements_summary_by_digest
    ORDER BY AVG_TIMER_WAIT DESC
    LIMIT 5
  ) t;
END$

DELIMITER ;
```

**3. Триггеры для аудита:**
```sql
-- Аудит изменений схемы (триггер на системные таблицы не работает, используй Event)
DELIMITER $
CREATE TRIGGER backup_history_after_insert
AFTER INSERT ON backup_history
FOR EACH ROW
BEGIN
  IF NEW.status = 'failed' THEN
    INSERT INTO alerts (alert_type, severity, message, details)
    VALUES (
      'backup_failed',
      'critical',
      CONCAT('Backup failed for ', NEW.database_name),
      JSON_OBJECT(
        'backup_type', NEW.backup_type,
        'database', NEW.database_name,
        'file_path', NEW.file_path
      )
    );
  END IF;
END$
DELIMITER ;
```

**4. События для автоматического мониторинга:**
```sql
DELIMITER $
-- Событие для сбора метрик каждые 5 минут
CREATE EVENT collect_metrics_event
ON SCHEDULE EVERY 5 MINUTE
DO
  CALL CollectMetrics();

-- Событие для проверки здоровья каждые 10 минут
CREATE EVENT health_check_event
ON SCHEDULE EVERY 10 MINUTE
DO
  CALL CheckSystemHealth();

-- Событие для очистки старых метрик (раз в день)
CREATE EVENT cleanup_old_metrics
ON SCHEDULE EVERY 1 DAY
DO
  DELETE FROM metrics_history 
  WHERE collected_at < DATE_SUB(NOW(), INTERVAL 30 DAY);

-- Событие для очистки разрешенных алертов
CREATE EVENT cleanup_old_alerts
ON SCHEDULE EVERY 1 DAY
DO
  DELETE FROM alerts 
  WHERE resolved_at IS NOT NULL 
    AND resolved_at < DATE_SUB(NOW(), INTERVAL 90 DAY);

DELIMITER ;
```

**5. Bash скрипты для автоматизации:**

**mysql_backup.sh:**
```bash
#!/bin/bash
set -euo pipefail

# Конфигурация
DB_USER="backup_user"
DB_PASS="backup_password"
BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# Функция логирования
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"
}

# Функция записи в БД
log_to_db() {
    local backup_type=$1
    local db_name=$2
    local file_path=$3
    local file_size=$4
    local checksum=$5
    local status=$6
    local started=$7
    local completed=$8
    
    mysql -u "$DB_USER" -p"$DB_PASS" mysql_monitoring <<EOF
INSERT INTO backup_history 
  (backup_type, database_name, file_path, file_size, checksum, status, started_at, completed_at)
VALUES 
  ('$backup_type', '$db_name', '$file_path', $file_size, '$checksum', '$status', '$started', '$completed');
EOF
}

# Создать директорию
mkdir -p "$BACKUP_DIR"

# Получить список баз данных
databases=$(mysql -u "$DB_USER" -p"$DB_PASS" -e "SHOW DATABASES;" | grep -Ev "Database|information_schema|performance_schema|mysql|sys")

for db in $databases; do
    log "Starting backup of $db"
    
    BACKUP_FILE="$BACKUP_DIR/${db}_${DATE}.sql.gz"
    START_TIME=$(date +'%Y-%m-%d %H:%M:%S')
    
    # Выполнить бэкап
    if mysqldump -u "$DB_USER" -p"$DB_PASS" \
        --single-transaction \
        --routines \
        --triggers \
        --events \
        --quick \
        "$db" | gzip > "$BACKUP_FILE"; then
        
        END_TIME=$(date +'%Y-%m-%d %H:%M:%S')
        FILE_SIZE=$(stat -f%z "$BACKUP_FILE" 2>/dev/null || stat -c%s "$BACKUP_FILE")
        CHECKSUM=$(md5sum "$BACKUP_FILE" | awk '{print $1}')
        
        log "Backup successful: $BACKUP_FILE ($(du -h "$BACKUP_FILE" | cut -f1))"
        
        # Записать в БД
        log_to_db "full" "$db" "$BACKUP_FILE" "$FILE_SIZE" "$CHECKSUM" "success" "$START_TIME" "$END_TIME"
        
        # Проверить целостность
        gunzip -t "$BACKUP_FILE"
        if [ $? -eq 0 ]; then
            log "Backup integrity check passed"
        else
            log "WARNING: Backup integrity check failed!"
        fi
    else
        END_TIME=$(date +'%Y-%m-%d %H:%M:%S')
        log "ERROR: Backup failed for $db"
        log_to_db "full" "$db" "$BACKUP_FILE" 0 "" "failed" "$START_TIME" "$END_TIME"
    fi
done

# Удалить старые бэкапы
log "Cleaning up backups older than $RETENTION_DAYS days"
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

log "Backup process completed"
```

**mysql_monitor.sh:**
```bash
#!/bin/bash
set -euo pipefail

DB_USER="monitor_user"
DB_PASS="monitor_password"
ALERT_EMAIL="admin@example.com"

# Проверка соединений
check_connections() {
    local result=$(mysql -u "$DB_USER" -p"$DB_PASS" -sN <<EOF
SELECT 
  ROUND(100 * current_conn / max_conn, 2) as usage,
  current_conn,
  max_conn
FROM (
  SELECT VARIABLE_VALUE as current_conn 
  FROM performance_schema.global_status 
  WHERE VARIABLE_NAME = 'Threads_connected'
) t,
(
  SELECT VARIABLE_VALUE as max_conn 
  FROM performance_schema.global_variables 
  WHERE VARIABLE_NAME = 'max_connections'
) m;
EOF
    )
    
    local usage=$(echo "$result" | awk '{print $1}')
    local current=$(echo "$result" | awk '{print $2}')
    local max=$(echo "$result" | awk '{print $3}')
    
    if (( $(echo "$usage > 80" | bc -l) )); then
        echo "ALERT: Connection usage is $usage% ($current/$max)"
        # Отправить email или уведомление
    fi
}

# Проверка репликации
check_replication() {
    local lag=$(mysql -u "$DB_USER" -p"$DB_PASS" -e "SHOW SLAVE STATUS\G" 2>/dev/null | \
                grep "Seconds_Behind_Master" | awk '{print $2}')
    
    if [ ! -z "$lag" ] && [ "$lag" != "NULL" ]; then
        if [ "$lag" -gt 60 ]; then
            echo "ALERT: Replication lag is $lag seconds"
        fi
    fi
}

# Проверка места на диске
check_disk_space() {
    local usage=$(df -h /var/lib/mysql | awk 'NR==2 {print $5}' | sed 's/%//')
    
    if [ "$usage" -gt 85 ]; then
        echo "ALERT: Disk usage is $usage%"
    fi
}

# Проверка медленных запросов
check_slow_queries() {
    local count=$(mysql -u "$DB_USER" -p"$DB_PASS" -sN <<EOF
SELECT COUNT(*) 
FROM performance_schema.events_statements_summary_by_digest 
WHERE AVG_TIMER_WAIT > 2000000000000;
EOF
    )
    
    if [ "$count" -gt 10 ]; then
        echo "ALERT: $count slow queries detected"
    fi
}

# Запуск всех проверок
echo "=== MySQL Health Check $(date) ==="
check_connections
check_replication
check_disk_space
check_slow_queries
echo "=== Check completed ==="
```

**6. Веб-дашборд (простой HTML + JS):**

**dashboard.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>MySQL Monitoring Dashboard</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; background: #f5f5f5; }
        .container { max-width: 1200px; margin: 0 auto; }
        .metric-card { 
            background: white; 
            padding: 20px; 
            margin: 10px 0; 
            border-radius: 5px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        .metric-value { font-size: 36px; font-weight: bold; color: #333; }
        .metric-label { color: #666; margin-top: 5px; }
        .alert { 
            padding: 15px; 
            margin: 10px 0; 
            border-radius: 5px;
            border-left: 4px solid;
        }
        .alert.critical { border-color: #dc3545; background: #f8d7da; }
        .alert.warning { border-color: #ffc107; background: #fff3cd; }
        .alert.info { border-color: #17a2b8; background: #d1ecf1; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { padding: 10px; text-align: left; border-bottom: 1px solid #ddd; }
        th { background: #007bff; color: white; }
    </style>
</head>
<body>
    <div class="container">
        <h1>MySQL Monitoring Dashboard</h1>
        
        <div id="metrics">
            <h2>Key Metrics</h2>
            <!-- Метрики будут добавлены через JavaScript -->
        </div>
        
        <div id="alerts">
            <h2>Active Alerts</h2>
            <!-- Алерты будут добавлены через JavaScript -->
        </div>
        
        <div id="slow-queries">
            <h2>Slow Queries</h2>
            <!-- Медленные запросы -->
        </div>
    </div>
    
    <script>
        // В реальном проекте здесь был бы AJAX запрос к API
        // который выполняет SQL запросы и возвращает JSON
        
        async function fetchMetrics() {
            // Пример данных (в реальности - из API)
            const metrics = {
                connections: { current: 45, max: 150, percent: 30 },
                slow_queries: 23,
                buffer_pool_hit_rate: 99.2,
                uptime_hours: 720
            };
            
            displayMetrics(metrics);
        }
        
        function displayMetrics(metrics) {
            const metricsDiv = document.getElementById('metrics');
            metricsDiv.innerHTML = `
                <div class="metric-card">
                    <div class="metric-value">${metrics.connections.percent}%</div>
                    <div class="metric-label">Connection Usage (${metrics.connections.current}/${metrics.connections.max})</div>
                </div>
                <div class="metric-card">
                    <div class="metric-value">${metrics.slow_queries}</div>
                    <div class="metric-label">Slow Queries (last hour)</div>
                </div>
                <div class="metric-card">
                    <div class="metric-value">${metrics.buffer_pool_hit_rate}%</div>
                    <div class="metric-label">Buffer Pool Hit Rate</div>
                </div>
                <div class="metric-card">
                    <div class="metric-value">${metrics.uptime_hours}h</div>
                    <div class="metric-label">Uptime</div>
                </div>
            `;
        }
        
        // Загрузить данные при открытии страницы
        fetchMetrics();
        
        // Обновлять каждые 30 секунд
        setInterval(fetchMetrics, 30000);
    </script>
</body>
</html>
```

### Задачи для реализации:

1. **Настрой базу данных:**
   - Создай все таблицы из схемы
   - Создай все процедуры и функции
   - Настрой события

2. **Реализуй скрипты:**
   - Создай и протестируй скрипт бэкапа
   - Создай скрипт мониторинга
   - Настрой cron задачи для автоматического выполнения

3. **Тестирование:**
   - Запусти `CollectMetrics()` и проверь таблицу metrics_history
   - Запусти `CheckSystemHealth()` и создай искусственную нагрузку
   - Проверь что события выполняются корректно
   - Сделай бэкап и проверь запись в backup_history

4. **Мониторинг:**
   - Создай представления для удобного просмотра метрик
   - Настрой алерты для критических ситуаций
   - Создай отчеты за день/неделю/месяц

5. **Оптимизация:**
   - Добавь индексы где необходимо
   - Настрой партиционирование для больших таблиц
   - Оптимизируй запросы в процедурах

### Критерии оценки:

- ✅ Все таблицы созданы и работают
- ✅ Процедуры выполняются без ошибок
- ✅ События запускаются по расписанию
- ✅ Скрипты корректно работают и логируют результаты
- ✅ Система мониторинга обнаруживает проблемы
- ✅ Бэкапы создаются автоматически и проверяются
- ✅ Код документирован и понятен

---

## Справочная секция: Быстрые шпаргалки

### Полезные запросы для ежедневной работы

**Размеры баз и таблиц:**
```sql
-- Размер всех баз данных
SELECT 
  table_schema AS 'Database',
  ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.TABLES
GROUP BY table_schema
ORDER BY SUM(data_length + index_length) DESC;

-- Размер таблиц в конкретной БД
SELECT 
  table_name AS 'Table',
  ROUND(((data_length + index_length) / 1024 / 1024), 2) AS 'Size (MB)',
  ROUND((data_length / 1024 / 1024), 2) AS 'Data (MB)',
  ROUND((index_length / 1024 / 1024), 2) AS 'Index (MB)',
  table_rows AS 'Rows'
FROM information_schema.TABLES
WHERE table_schema = 'database_name'
ORDER BY (data_length + index_length) DESC;

-- Фрагментация таблиц
SELECT 
  table_schema,
  table_name,
  ROUND(data_length / 1024 / 1024, 2) AS data_mb,
  ROUND(data_free / 1024 / 1024, 2) AS free_mb,
  ROUND(data_free / data_length * 100, 2) AS fragmentation_percent
FROM information_schema.TABLES
WHERE table_schema NOT IN ('information_schema', 'mysql', 'performance_schema', 'sys')
  AND data_free > 0
ORDER BY fragmentation_percent DESC;
```

**Активность и производительность:**
```sql
-- Топ запросов по времени выполнения
SELECT 
  SUBSTRING(DIGEST_TEXT, 1, 100) AS query,
  COUNT_STAR AS exec_count,
  ROUND(AVG_TIMER_WAIT / 1000000000000, 2) AS avg_time_sec,
  ROUND(SUM_TIMER_WAIT / 1000000000000, 2) AS total_time_sec
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;

-- Топ таблиц по I/O
SELECT 
  OBJECT_SCHEMA AS db,
  OBJECT_NAME AS table_name,
  COUNT_READ,
  COUNT_WRITE,
  ROUND(SUM_TIMER_WAIT / 1000000000000, 2) AS total_time_sec
FROM performance_schema.table_io_waits_summary_by_table
WHERE OBJECT_SCHEMA NOT IN ('mysql', 'performance_schema', 'sys')
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;

-- Неиспользуемые индексы
SELECT 
  OBJECT_SCHEMA AS db,
  OBJECT_NAME AS table_name,
  INDEX_NAME
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE INDEX_NAME IS NOT NULL
  AND COUNT_STAR = 0
  AND OBJECT_SCHEMA NOT IN ('mysql', 'performance_schema', 'sys')
ORDER BY OBJECT_SCHEMA, OBJECT_NAME;

-- Дублирующиеся индексы
SELECT 
  a.table_schema,
  a.table_name,
  a.index_name AS index1,
  b.index_name AS index2,
  a.column_name
FROM information_schema.statistics a
JOIN information_schema.statistics b 
  ON a.table_schema = b.table_schema
  AND a.table_name = b.table_name
  AND a.seq_in_index = b.seq_in_index
  AND a.column_name = b.column_name
WHERE a.index_name != b.index_name
  AND a.index_name < b.index_name
ORDER BY a.table_schema, a.table_name;
```

**Блокировки и транзакции:**
```sql
-- Текущие блокировки
SELECT 
  r.trx_id waiting_trx,
  r.trx_mysql_thread_id waiting_thread,
  r.trx_query waiting_query,
  b.trx_id blocking_trx,
  b.trx_mysql_thread_id blocking_thread,
  b.trx_query blocking_query
FROM information_schema.INNODB_LOCK_WAITS w
JOIN information_schema.INNODB_TRX b ON b.trx_id = w.blocking_trx_id
JOIN information_schema.INNODB_TRX r ON r.trx_id = w.requesting_trx_id;

-- Долгие транзакции
SELECT 
  trx_id,
  trx_state,
  trx_started,
  TIMESTAMPDIFF(SECOND, trx_started, NOW()) AS duration_sec,
  trx_query,
  trx_rows_locked,
  trx_rows_modified
FROM information_schema.INNODB_TRX
WHERE TIMESTAMPDIFF(SECOND, trx_started, NOW()) > 60
ORDER BY trx_started;
```

### Команды для быстрой диагностики

**One-liners для консоли:**
```bash
# Проверка статуса MySQL
systemctl status mysql | head -5

# Текущие соединения
mysql -e "SHOW STATUS LIKE 'Threads_connected';"

# Использование buffer pool
mysql -e "SHOW STATUS LIKE 'Innodb_buffer_pool%';" | grep -E "read_requests|reads"

# Медленные запросы
mysql -e "SHOW STATUS LIKE 'Slow_queries';"

# Размер binary logs
du -sh /var/log/mysql/mysql-bin.*

# Топ процессов по времени
mysql -e "SELECT ID, USER, HOST, DB, TIME, STATE, LEFT(INFO, 50) FROM information_schema.PROCESSLIST ORDER BY TIME DESC LIMIT 10;"

# Проверка репликации
mysql -e "SHOW SLAVE STATUS\G" | grep -E "Slave_IO_Running|Slave_SQL_Running|Seconds_Behind_Master"

# Kill всех процессов пользователя (осторожно!)
mysql -e "SELECT CONCAT('KILL ', id, ';') FROM information_schema.PROCESSLIST WHERE user='username';" | mysql

# Проверка фрагментации
mysql -e "SELECT table_schema, table_name, data_free FROM information_schema.TABLES WHERE table_schema='db_name' AND data_free > 0;"
```

### Алиасы для .bashrc

```bash
# MySQL алиасы
alias mysql-status='mysql -e "SHOW STATUS"'
alias mysql-vars='mysql -e "SHOW VARIABLES"'
alias mysql-proc='mysql -e "SHOW FULL PROCESSLIST"'
alias mysql-slow='mysql -e "SHOW STATUS LIKE \"Slow_queries\""'
alias mysql-conn='mysql -e "SHOW STATUS LIKE \"Threads_connected\"; SHOW VARIABLES LIKE \"max_connections\""'
alias mysql-size='mysql -e "SELECT table_schema AS DB, ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS Size_MB FROM information_schema.TABLES GROUP BY table_schema ORDER BY Size_MB DESC;"'

# Percona Toolkit
alias pt-summary='pt-mysql-summary -- --user=root'
alias pt-slow='pt-query-digest /var/log/mysql/mysql-slow.log | head -50'

# Логи
alias mysql-error='tail -50 /var/log/mysql/error.log'
alias mysql-slow-log='tail -50 /var/log/mysql/mysql-slow.log'

# Бэкап
alias mysql-backup-all='mysqldump --all-databases --single-transaction --routines --triggers --events | gzip > /backup/mysql/all_$(date +\%Y\%m\%d).sql.gz'
```

### Настройка my.cnf для разных сценариев

**Для веб-приложения (OLTP):**
```ini
[mysqld]
# Основное
max_connections = 500
max_connect_errors = 100
wait_timeout = 600
interactive_timeout = 600

# InnoDB
innodb_buffer_pool_size = 8G        # 70-80% RAM
innodb_log_file_size = 1G
innodb_flush_log_at_trx_commit = 2  # Для производительности
innodb_flush_method = O_DIRECT
innodb_file_per_table = 1

# Query Cache (отключить в MySQL 8)
query_cache_type = 0
query_cache_size = 0

# Временные таблицы
tmp_table_size = 128M
max_heap_table_size = 128M

# Threads
thread_cache_size = 50

# Tables
table_open_cache = 4000
table_definition_cache = 2000

# Безопасность
skip-name-resolve = 1
local_infile = 0
```

**Для аналитики (OLAP):**
```ini
[mysqld]
# Больше памяти для сортировок и join
sort_buffer_size = 4M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
join_buffer_size = 8M

# Большие временные таблицы
tmp_table_size = 512M
max_heap_table_size = 512M

# InnoDB
innodb_buffer_pool_size = 12G
innodb_log_file_size = 2G
innodb_read_io_threads = 8
innodb_write_io_threads = 8

# Параллельные запросы (MySQL 8)
innodb_parallel_read_threads = 4
```

**Для репликации (слейв):**
```ini
[mysqld]
server-id = 2
read_only = 1
relay_log = /var/log/mysql/relay-bin
relay_log_recovery = 1
slave_parallel_workers = 4
slave_parallel_type = LOGICAL_CLOCK
```

---

## План повторения

### При первом прохождении (3-4 часа):
- Модули 1-4 обязательно (основы)
- Модули 5-6 базовые задания
- Модуль 7 прочитать и попрактиковать
- Начать финальный проект (минимум БД + процедуры)

### При повторном прохождении (через 6 месяцев):
- Фокус на бонусные задания
- Модули 5-6 полностью (репликация и мониторинг)
- Модуль 8 - продвинутые возможности
- Доделать финальный проект полностью
- Добавить свои кейсы из реальной работы

### Для закрепления:
- Автоматизируй бэкапы для production БД
- Настрой мониторинг с алертами
- Оптимизируй медленные запросы на реальных проектах
- Практикуй troubleshooting на реальных проблемах
- Изучи execution plans сложных запросов

### Что отслеживать при повторных прохождениях:
- ✅ Помню ли основные SQL конструкции без подглядывания?
- ✅ Могу ли быстро найти и оптимизировать медленные запросы?
- ✅ Уверенно ли работаю с бэкапами и восстановлением?
- ✅ Знаю ли как диагностировать проблемы производительности?
- ✅ Могу ли настроить и поддерживать репликацию?
- ✅ Понимаю ли метрики мониторинга и их значения?

---

## Чеклист навыков MySQL DevOps/SysAdmin

После прохождения курса ты должен уметь:

### Базовые навыки:
- [ ] Подключаться к MySQL разными способами
- [ ] Выполнять CRUD операции уверенно
- [ ] Создавать и изменять таблицы
- [ ] Понимать типы данных и их использование
- [ ] Работать с пользователями и правами

### Оптимизация и индексы:
- [ ] Создавать правильные индексы
- [ ] Читать и понимать EXPLAIN
- [ ] Оптимизировать медленные запросы
- [ ] Работать с Performance Schema
- [ ] Анализировать slow query log

### Backup и Recovery:
- [ ] Делать полные и инкрементальные бэкапы
- [ ] Восстанавливать данные из бэкапов
- [ ] Использовать mysqldump и XtraBackup
- [ ] Настраивать Point-in-Time Recovery
- [ ] Автоматизировать процесс бэкапов

### Репликация:
- [ ] Настраивать Master-Slave репликацию
- [ ] Работать с GTID репликацией
- [ ] Мониторить задержку репликации
- [ ] Решать проблемы с репликацией
- [ ] Выполнять failover

### Мониторинг:
- [ ] Собирать и анализировать метрики
- [ ] Использовать Performance Schema
- [ ] Настраивать slow query log
- [ ] Работать с Percona Toolkit
- [ ] Настраивать системы мониторинга (PMM, Grafana)

### Безопасность:
- [ ] Настраивать права пользователей правильно
- [ ] Работать с ролями (MySQL 8.0+)
- [ ] Настраивать SSL соединения
- [ ] Проводить аудит безопасности
- [ ] Защищать от SQL injection

### Troubleshooting:
- [ ] Диагностировать проблемы с производительностью
- [ ] Решать проблемы с блокировками
- [ ] Восстанавливать поврежденные таблицы
- [ ] Анализировать высокое использование ресурсов
- [ ] Работать с deadlocks

### Продвинутые возможности:
- [ ] Писать хранимые процедуры и функции
- [ ] Создавать триггеры
- [ ] Использовать события (Event Scheduler)
- [ ] Работать с JSON данными
- [ ] Использовать Window Functions (MySQL 8.0+)
- [ ] Настраивать партиционирование

---

## Дополнительные ресурсы

**Официальная документация:**
- MySQL Documentation: https://dev.mysql.com/doc/
- Percona Server Documentation: https://docs.percona.com/
- MariaDB Documentation: https://mariadb.com/kb/

**Книги:**
- "High Performance MySQL" by Baron Schwartz
- "MySQL Troubleshooting" by Sveta Smirnova
- "Effective MySQL" series
- "MySQL Administrator's Bible"

**Онлайн ресурсы:**
- MySQL Planet: https://planet.mysql.com/
- Percona Blog: https://www.percona.com/blog/
- MySQL Performance Blog
- Use The Index, Luke: https://use-the-index-luke.com/

**Инструменты:**
- Percona Toolkit: https://www.percona.com/software/database-tools/percona-toolkit
- PMM (Percona Monitoring and Management)
- pt-query-digest, pt-online-schema-change
- mysqltuner: https://github.com/major/MySQLTuner-perl
- mycli: https://www.mycli.net/

**Практика:**
- MySQL Sandbox для локального тестирования
- Docker контейнеры для экспериментов
- Участие в MySQL User Groups
- Изучение open-source проектов использующих MySQL

**Сертификации:**
- MySQL Database Administrator (OCA)
- MySQL Database Administrator (OCP)
- Percona MySQL Certification

---

## Полезные скрипты

### Скрипт автоматической оптимизации

**mysql_optimize.sh:**
```bash
#!/bin/bash
# Оптимизация таблиц с высокой фрагментацией

DB_USER="admin"
DB_PASS="password"
FRAGMENTATION_THRESHOLD=20

mysql -u "$DB_USER" -p"$DB_PASS" -Ns <<'EOF' | while read db table frag; do
SELECT 
  table_schema,
  table_name,
  ROUND(data_free / data_length * 100, 2) AS fragmentation
FROM information_schema.TABLES
WHERE table_schema NOT IN ('information_schema', 'mysql', 'performance_schema', 'sys')
  AND data_length > 0
  AND data_free > 0
  AND (data_free / data_length * 100) > 20
ORDER BY fragmentation DESC;
EOF
  if (( $(echo "$frag > $FRAGMENTATION_THRESHOLD" | bc -l) )); then
    echo "Optimizing $db.$table (fragmentation: $frag%)"
    mysql -u "$DB_USER" -p"$DB_PASS" -e "OPTIMIZE TABLE $db.$table;"
  fi
done
```

### Скрипт проверки репликации

**check_replication.sh:**
```bash
#!/bin/bash
# Проверка состояния репликации

DB_USER="monitor"
DB_PASS="password"

# Получить статус репликации
STATUS=$(mysql -u "$DB_USER" -p"$DB_PASS" -e "SHOW SLAVE STATUS\G")

if [ -z "$STATUS" ]; then
    echo "ERROR: Not a slave server or replication not configured"
    exit 1
fi

# Проверить IO и SQL потоки
IO_RUNNING=$(echo "$STATUS" | grep "Slave_IO_Running:" | awk '{print $2}')
SQL_RUNNING=$(echo "$STATUS" | grep "Slave_SQL_Running:" | awk '{print $2}')
SECONDS_BEHIND=$(echo "$STATUS" | grep "Seconds_Behind_Master:" | awk '{print $2}')

echo "=== Replication Status ==="
echo "IO Thread: $IO_RUNNING"
echo "SQL Thread: $SQL_RUNNING"
echo "Seconds Behind Master: $SECONDS_BEHIND"

if [ "$IO_RUNNING" != "Yes" ] || [ "$SQL_RUNNING" != "Yes" ]; then
    echo "ERROR: Replication is not running properly!"
    echo "$STATUS" | grep "Last_Error"
    exit 1
fi

if [ "$SECONDS_BEHIND" != "NULL" ] && [ "$SECONDS_BEHIND" -gt 60 ]; then
    echo "WARNING: Replication lag is $SECONDS_BEHIND seconds"
    exit 1
fi

echo "Replication is healthy"
exit 0
```

### Скрипт анализа производительности

**mysql_performance_report.sh:**
```bash
#!/bin/bash
# Генерация отчета о производительности

DB_USER="monitor"
DB_PASS="password"
OUTPUT_FILE="mysql_performance_$(date +%Y%m%d_%H%M%S).txt"

{
    echo "=== MySQL Performance Report ==="
    echo "Generated: $(date)"
    echo ""
    
    echo "=== Server Info ==="
    mysql -u "$DB_USER" -p"$DB_PASS" -e "
        SELECT 
            @@version AS version,
            @@hostname AS hostname,
            @@port AS port,
            @@datadir AS datadir;
    "
    
    echo ""
    echo "=== Uptime and Load ==="
    mysql -u "$DB_USER" -p"$DB_PASS" -e "
        SELECT 
            ROUND(VARIABLE_VALUE / 3600, 2) AS uptime_hours,
            (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME='Threads_connected') AS connections,
            (SELECT VARIABLE_VALUE FROM performance_schema.global_variables WHERE VARIABLE_NAME='max_connections') AS max_connections,
            (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME='Slow_queries') AS slow_queries
        FROM performance_schema.global_status
        WHERE VARIABLE_NAME = 'Uptime';
    "
    
    echo ""
    echo "=== InnoDB Buffer Pool ==="
    mysql -u "$DB_USER" -p"$DB_PASS" -e "
        SELECT 
            VARIABLE_NAME,
            VARIABLE_VALUE
        FROM performance_schema.global_status
        WHERE VARIABLE_NAME LIKE 'Innodb_buffer_pool%'
        ORDER BY VARIABLE_NAME;
    "
    
    echo ""
    echo "=== Top 10 Slow Queries ==="
    mysql -u "$DB_USER" -p"$DB_PASS" -e "
        SELECT 
            SUBSTRING(DIGEST_TEXT, 1, 80) AS query,
            COUNT_STAR AS count,
            ROUND(AVG_TIMER_WAIT / 1000000000000, 2) AS avg_sec,
            ROUND(SUM_TIMER_WAIT / 1000000000000, 2) AS total_sec
        FROM performance_schema.events_statements_summary_by_digest
        ORDER BY SUM_TIMER_WAIT DESC
        LIMIT 10;
    "
    
    echo ""
    echo "=== Database Sizes ==="
    mysql -u "$DB_USER" -p"$DB_PASS" -e "
        SELECT 
            table_schema AS 'Database',
            ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
        FROM information_schema.TABLES
        GROUP BY table_schema
        ORDER BY SUM(data_length + index_length) DESC;
    "
    
    echo ""
    echo "=== Table without Primary Keys ==="
    mysql -u "$DB_USER" -p"$DB_PASS" -e "
        SELECT 
            t.table_schema,
            t.table_name
        FROM information_schema.tables t
        LEFT JOIN information_schema.table_constraints tc
            ON t.table_schema = tc.table_schema
            AND t.table_name = tc.table_name
            AND tc.constraint_type = 'PRIMARY KEY'
        WHERE t.table_schema NOT IN ('information_schema', 'mysql', 'performance_schema', 'sys')
            AND tc.constraint_name IS NULL;
    "
    
} > "$OUTPUT_FILE"

echo "Report saved to: $OUTPUT_FILE"
cat "$OUTPUT_FILE"
```

---

## Советы по прохождению курса

1. **Используй тестовую среду.** Не экспериментируй на production! Используй Docker, VM или выделенный сервер для тестирования.

2. **Читай EXPLAIN.** Каждый запрос, который пишешь, анализируй с помощью EXPLAIN. Это войдёт в привычку.

3. **Веди заметки.** Создай свою базу знаний с часто используемыми запросами и решениями проблем.

4. **Практикуй на реальных данных.** Скачай открытые датасеты и практикуйся на них.

5. **Изучай логи.** Error log и slow query log - твои лучшие друзья при troubleshooting.

6. **Автоматизируй.** Любую задачу, которую делаешь дважды, стоит автоматизировать скриптом.

7. **Мониторь постоянно.** Настрой систему мониторинга и регулярно проверяй метрики.

8. **Делай бэкапы.** И проверяй восстановление! Бэкап без теста восстановления - не бэкап.

9. **Тестируй перед production.** Любые изменения конфигурации или схемы сначала тестируй.

10. **Учись у других.** Читай блоги, смотри конференции, участвуй в community.

---

## Типичные ошибки и как их избежать

### ❌ Ошибка 1: SELECT * в production
```sql
-- Плохо
SELECT * FROM large_table WHERE id = 1;

-- Хорошо
SELECT id, name, email FROM large_table WHERE id = 1;
```

### ❌ Ошибка 2: Отсутствие индексов на JOIN колонках
```sql
-- Создавай индексы на FK
ALTER TABLE orders ADD INDEX idx_user_id (user_id);
```

### ❌ Ошибка 3: Использование функций в WHERE
```sql
-- Плохо (индекс не используется)
SELECT * FROM users WHERE DATE(created_at) = '2024-01-01';

-- Хорошо (индекс работает)
SELECT * FROM users 
WHERE created_at >= '2024-01-01' 
  AND created_at < '2024-01-02';
```

### ❌ Ошибка 4: N+1 запросы
```sql
-- Плохо (в цикле)
FOR each order DO
  SELECT * FROM users WHERE id = order.user_id;
END FOR;

-- Хорошо (один JOIN)
SELECT o.*, u.* 
FROM orders o 
JOIN users u ON o.user_id = u.id;
```

### ❌ Ошибка 5: Забыть про транзакции
```sql
-- Плохо
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Хорошо
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

### ❌ Ошибка 6: Не ограничивать UPDATE/DELETE
```sql
-- Опасно!
DELETE FROM logs WHERE created_at < '2024-01-01';

-- Безопаснее
DELETE FROM logs 
WHERE created_at < '2024-01-01' 
LIMIT 10000;
-- Запустить несколько раз
```

### ❌ Ошибка 7: Игнорировать предупреждения
```sql
-- После каждого запроса проверяй
SHOW WARNINGS;
```

### ❌ Ошибка 8: Хранить пароли в открытом виде
```bash
# Плохо
mysql -u root -pMyPassword

# Хорошо
mysql_config_editor set --login-path=local --user=root --password
mysql --login-path=local
```

### ❌ Ошибка 9: Не делать бэкапы перед ALTER TABLE
```bash
# Всегда делай бэкап!
mysqldump database_name table_name > backup.sql
# Теперь можно ALTER
```

### ❌ Ошибка 10: Не проверять результат восстановления
```bash
# После восстановления бэкапа
# Проверь количество записей
mysql -e "SELECT COUNT(*) FROM table_name;"
# Проверь несколько случайных записей
```

---

## Заключение

**Этот курс - не одноразовое действие, а регулярная практика!**

🎯 **Проходи курс каждые 6-12 месяцев:**
- Освежишь забытые команды
- Узнаешь новые техники
- Улучшишь понимание MySQL
- Повысишь скорость работы

📊 **Метрики успеха:**
- Можешь настроить репликацию за 30 минут
- Находишь медленные запросы за 5 минут
- Восстанавливаешь БД из бэкапа без паники
- Понимаешь 90% метрик мониторинга
- Решаешь типичные проблемы без гугления

🚀 **Следующие шаги:**
1. Пройди весь курс хотя бы один раз
2. Примени знания на своих проектах
3. Автоматизируй рутинные задачи
4. Настрой мониторинг для своих серверов
5. Вернись к курсу через 6 месяцев

💪 **Remember:**
> "In MySQL we trust, but always backup!" 

**Удачи в освоении MySQL! Пусть твои запросы будут быстрыми, а бэкапы - надежными!** 🎉🐬metric_time (metric_name, collected_at)
) PARTITION BY RANGE (YEAR(collected_at)) (
  PARTITION p2024 VALUES LESS THAN (2025),
  PARTITION p2025 VALUES LESS THAN (2026),
  PARTITION pfuture VALUES LESS THAN MAXVALUE
);

-- Таблица для алертов
CREATE TABLE alerts (
  id INT AUTO_INCREMENT PRIMARY KEY,
  alert_type VARCHAR(50),
  severity ENUM('info', 'warning', 'critical'),
  message TEXT,
  details JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  resolved_at TIMESTAMP NULL,
  INDEX idx_severity_time (severity, created_at)
);

-- Таблица для аудита изменений схемы
CREATE TABLE schema_changes (
  id INT AUTO_INCREMENT PRIMARY KEY,
  database_name VARCHAR(64),
  table_name VARCHAR(64),
  change_type VARCHAR(50),
  ddl_statement TEXT,
  executed_by VARCHAR(100),
  executed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Таблица для истории бэкапов
CREATE TABLE backup_history (
  id INT AUTO_INCREMENT PRIMARY KEY,
  backup_type ENUM('full', 'incremental', 'differential'),
  database_name VARCHAR(64),
  file_path VARCHAR(500),
  file_size BIGINT,
  checksum VARCHAR(64),
  status ENUM('success', 'failed', 'in_progress'),
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  INDEX idx_граничить количество подключений
CREATE USER 'username'@'localhost' 
  IDENTIFIED BY 'password'
  WITH MAX_QUERIES_PER_HOUR 1000
       MAX_UPDATES_PER_HOUR 500
       MAX_CONNECTIONS_PER_HOUR 100
       MAX_USER_CONNECTIONS 10;

-- 7. Включить аудит (Percona Server)
INSTALL PLUGIN audit_log SONAME 'audit_log.so';
SET GLOBAL audit_log_policy = 'ALL';
```

**Конфигурация в my.cnf:**
```ini
[mysqld]
# Безопасность
skip-name-resolve                  # Не резолвить имена (быстрее)
local-infile=0                     # Отключить LOAD DATA LOCAL INFILE
secure-file-priv=/var/lib/mysql-files/  # Ограничить загрузку файлов

# SSL
ssl-ca=/path/to/ca.pem
ssl-cert=/path/to/server-cert.pem
ssl-key=/path/to/server-key.pem

# Логирование
general_log=1                      # Общий лог (осторожно, много места)
general_log_file=/var/log/mysql/general.log
slow_query_log=1                   # Медленные запросы
slow_query_log_file=/var/log/mysql/slow.log
long_query_time=2                  # Секунды
log_queries_not_using_indexes=1    # Логировать запросы без индексов
```

### 💻 Задание

Выполни задачи по безопасности:

1. Создай нового пользователя `app_user` с паролем, который может подключаться только с localhost

2. Выдай этому пользователю права SELECT, INSERT, UPDATE на базу `test_db`

3. Создай пользователя `readonly_user` только с правами SELECT

4. Проверь права созданных пользователей с помощью `SHOW GRANTS`

5. Посмотри список всех пользователей в системе

6. Попробуй подключиться под новым пользователем и выполнить разрешенные и запрещенные операции

7. Найди в конфигурации MySQL настройки безопасности:
   ```bash
   grep -E "bind-address|skip-networking|port" /etc/mysql/my.cnf
   ```

### 🚀 Бонус (новое)

- Создай роль для API приложения (MySQL 8.0+):
  ```sql
  CREATE ROLE 'api_role';
  GRANT SELECT, INSERT, UPDATE ON app_db.* TO 'api_role';
  CREATE USER 'api_user'@'%' IDENTIFIED BY 'password';
  GRANT 'api_role' TO 'api_user'@'%';
  SET DEFAULT ROLE 'api_role' TO 'api_user'@'%';
  ```
- Настрой SSL соединения для MySQL
- Используй `mysql_config_editor` для безопасного хранения паролей:
  ```bash
  mysql_config_editor set --login-path=local --host=localhost --user=root --password
  mysql --login-path=local
  ```
- Включи аудит логирование (если Percona Server):
  ```sql
  INSTALL PLUGIN audit_log SONAME 'audit_log.so';
  SET GLOBAL audit_log_file = '/var/log/mysql/audit.log';
  ```

---

## Модуль 4: Backup и Recovery (30 минут)

### 🎯 Напоминалка

**Типы бэкапов:**

1. **Logical backup** - SQL дамп (медленнее, универсальнее)
2. **Physical backup** - копия файлов (быстрее, требует остановки или специальных инструментов)
3. **Full backup** - полный бэкап
4. **Incremental backup** - только изменения

**mysqldump - логический бэкап:**
```bash
# Одна база данных
mysqldump -u root -p database_name > backup.sql
mysqldump -u root -p database_name table_name > table_backup.sql

# Несколько баз
mysqldump -u root -p --databases db1 db2 db3 > backup.sql

# Все базы данных
mysqldump -u root -p --all-databases > all_backup.sql

# Только структура (без данных)
mysqldump -u root -p --no-data database_name > schema.sql

# Только данные (без структуры)
mysqldump -u root -p --no-create-info database_name > data.sql

# С сжатием
mysqldump -u root -p database_name | gzip > backup.sql.gz

# С добавлением DROP TABLE
mysqldump -u root -p --add-drop-table database_name > backup.sql

# Для репликации (с координатами binlog)
mysqldump -u root -p --master-data=2 --single-transaction database_name > backup.sql

# С триггерами, процедурами, событиями
mysqldump -u root -p --routines --triggers --events database_name > backup.sql

# Быстрый дамп больших таблиц
mysqldump -u root -p --quick --single-transaction database_name > backup.sql
```

**Восстановление из mysqldump:**
```bash
# Восстановление базы
mysql -u root -p database_name < backup.sql

# Создать БД и восстановить
mysql -u root -p -e "CREATE DATABASE database_name"
mysql -u root -p database_name < backup.sql

# Из сжатого бэкапа
gunzip < backup.sql.gz | mysql -u root -p database_name
zcat backup.sql.gz | mysql -u root -p database_name

# С прогресс-баром
pv backup.sql | mysql -u root -p database_name

# Восстановить конкретную таблицу
sed -n '/CREATE TABLE `table_name`/,/UNLOCK TABLES/p' backup.sql | mysql -u root -p database_name
```

**Percona XtraBackup - физический бэкап:**
```bash
# Установка (Debian/Ubuntu)
wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
sudo dpkg -i percona-release_latest.generic_all.deb
sudo apt update
sudo apt install percona-xtrabackup-80

# Полный бэкап
xtrabackup --backup --target-dir=/backup/full --user=root --password=pass

# Подготовка бэкапа (важно!)
xtrabackup --prepare --target-dir=/backup/full

# Восстановление (сервер должен быть остановлен!)
sudo systemctl stop mysql
xtrabackup --copy-back --target-dir=/backup/full --datadir=/var/lib/mysql
sudo chown -R mysql:mysql /var/lib/mysql
sudo systemctl start mysql

# Инкрементальный бэкап
xtrabackup --backup --target-dir=/backup/inc1 --incremental-basedir=/backup/full --user=root --password=pass

# Подготовка инкрементального бэкапа
xtrabackup --prepare --apply-log-only --target-dir=/backup/full
xtrabackup --prepare --apply-log-only --target-dir=/backup/full --incremental-dir=/backup/inc1
xtrabackup --prepare --target-dir=/backup/full

# Streaming бэкап (на удаленный сервер)
xtrabackup --backup --stream=xbstream --target-dir=./ | ssh user@remote "cat - > backup.xbstream"
```

**mydumper/myloader - многопоточный дамп:**
```bash
# Установка
sudo apt install mydumper

# Бэкап (4 потока)
mydumper -u root -p password -o /backup/mydumper -t 4

# Бэкап конкретной БД
mydumper -u root -p password -B database_name -o /backup/db_backup -t 4

# Восстановление
myloader -u root -p password -d /backup/mydumper -t 4

# Только конкретные таблицы
mydumper -u root -p password -B database_name -T table1,table2 -o /backup
```

**Binary Log - для Point-in-Time Recovery:**
```sql
-- Включение binary log (в my.cnf)
[mysqld]
server-id=1
log-bin=/var/log/mysql/mysql-bin
binlog_format=ROW
expire_logs_days=7

-- Просмотр binary logs
SHOW BINARY LOGS;
SHOW MASTER STATUS;

-- Просмотр содержимого
SHOW BINLOG EVENTS IN 'mysql-bin.000001';

-- С помощью утилиты
mysqlbinlog /var/log/mysql/mysql-bin.000001

-- Point-in-Time восстановление
# 1. Восстановить из полного бэкапа
mysql -u root -p < full_backup.sql

# 2. Применить binlog до определенной позиции
mysqlbinlog --stop-position=12345 mysql-bin.000001 | mysql -u root -p

# 3. Или по времени
mysqlbinlog --stop-datetime="2024-01-15 10:30:00" mysql-bin.000001 | mysql -u root -p

-- Очистка старых binary logs
PURGE BINARY LOGS BEFORE '2024-01-01 00:00:00';
PURGE BINARY LOGS TO 'mysql-bin.000010';
```

**Автоматизация бэкапов:**
```bash
#!/bin/bash
# backup_mysql.sh

# Конфигурация
BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7
MYSQL_USER="backup_user"
MYSQL_PASS="backup_password"

# Создать директорию
mkdir -p "$BACKUP_DIR"

# Бэкап всех баз
mysqldump -u "$MYSQL_USER" -p"$MYSQL_PASS" \
  --all-databases \
  --single-transaction \
  --routines \
  --triggers \
  --events \
  --master-data=2 \
  | gzip > "$BACKUP_DIR/all_databases_$DATE.sql.gz"

# Проверка успешности
if [ $? -eq 0 ]; then
    echo "Backup successful: all_databases_$DATE.sql.gz"
    
    # Удалить старые бэкапы
    find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete
else
    echo "Backup failed!"
    exit 1
fi

# Бэкап на удаленный сервер (опционально)
# rsync -avz "$BACKUP_DIR/" user@remote:/backup/mysql/
```

**Cron задача для автоматического бэкапа:**
```bash
# Добавить в crontab
crontab -e

# Ежедневный бэкап в 2:00 AM
0 2 * * * /path/to/backup_mysql.sh >> /var/log/mysql_backup.log 2>&1

# Еженедельный полный бэкап в воскресенье
0 3 * * 0 /path/to/full_backup.sh >> /var/log/mysql_backup.log 2>&1
```

### 💻 Задание

Выполни задачи по бэкапу и восстановлению:

1. Создай бэкап базы `test_db` с помощью mysqldump

2. Создай новую базу `test_db_restore` и восстанови в неё данные из бэкапа

3. Сделай бэкап только структуры (без данных) базы `test_db`

4. Экспортируй только одну таблицу `employees`

5. Проверь, включен ли binary log:
   ```sql
   SHOW VARIABLES LIKE 'log_bin';
   SHOW MASTER STATUS;
   ```

6. Посмотри размер всех бэкапов:
   ```bash
   du -sh /path/to/backups/*
   ```

7. Создай скрипт автоматического бэкапа (используй шаблон выше)

### 🚀 Бонус (новое)

- Установи и используй Percona XtraBackup для физического бэкапа
- Создай инкрементальный бэкап с XtraBackup
- Настрой Point-in-Time Recovery: сделай полный бэкап, внеси изменения, затем восстанови до определенного момента
- Используй `mydumper` для многопоточного бэкапа большой БД
- Настрой автоматическую загрузку бэкапов в S3 или другое облачное хранилище:
  ```bash
  # Пример с AWS S3
  aws s3 cp backup.sql.gz s3://mybucket/mysql-backups/
  ```
- Проверь целостность бэкапа:
  ```bash
  # Для gzip
  gunzip -t backup.sql.gz
  
  # MD5 чексумма
  md5sum backup.sql.gz > backup.sql.gz.md5
  md5sum -c backup.sql.gz.md5
  ```

---

## Модуль 5: Репликация и High Availability (35 минут)

### 🎯 Напоминалка

**Типы репликации MySQL:**

1. **Asynchronous Replication** - стандартная репликация (мастер не ждет слейвов)
2. **Semi-Synchronous Replication** - мастер ждет подтверждения хотя бы от одного слейва
3. **Group Replication** - синхронная multi-master репликация (MySQL 8.0+)

**Базовая Master-Slave репликация:**

**На мастере:**
```sql
-- 1. Включить binary log (в my.cnf)
[mysqld]
server-id=1
log-bin=/var/log/mysql/mysql-bin
binlog_format=ROW

-- 2. Перезапустить MySQL
sudo systemctl restart mysql

-- 3. Создать пользователя для репликации
CREATE USER 'repl'@'%' IDENTIFIED BY 'replication_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;

-- 4. Получить позицию в binlog
SHOW MASTER STATUS;
-- Запомнить File и Position!
```

**На слейве:**
```sql
-- 1. Конфигурация (в my.cnf)
[mysqld]
server-id=2
log-bin=/var/log/mysql/mysql-bin
relay-log=/var/log/mysql/relay-bin
read_only=1

-- 2. Перезапустить MySQL
sudo systemctl restart mysql

-- 3. Настроить репликацию
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='replication_password',
  MASTER_LOG_FILE='mysql-bin.000001',  -- Из SHOW MASTER STATUS
  MASTER_LOG_POS=12345;                 -- Из SHOW MASTER STATUS

-- 4. Запустить репликацию
START SLAVE;

-- 5. Проверить статус
SHOW SLAVE STATUS\G

-- Важные поля:
-- Slave_IO_Running: Yes
-- Slave_SQL_Running: Yes
-- Seconds_Behind_Master: 0 (желательно)
-- Last_Error: (должно быть пусто)
```

**Управление репликацией на слейве:**
```sql
-- Статус репликации
SHOW SLAVE STATUS\G

-- Остановка/запуск
STOP SLAVE;
START SLAVE;

-- Остановить только IO или SQL поток
STOP SLAVE IO_THREAD;
STOP SLAVE SQL_THREAD;
START SLAVE IO_THREAD;
START SLAVE SQL_THREAD;

-- Пропустить ошибку репликации
STOP SLAVE;
SET GLOBAL sql_slave_skip_counter = 1;
START SLAVE;

-- Изменить позицию репликации
STOP SLAVE;
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=4567;
START SLAVE;

-- Сброс репликации (удалить relay logs)
RESET SLAVE;
RESET SLAVE ALL;  -- Полностью удалить конфигурацию
```

**GTID репликация (MySQL 5.6+):**
```sql
-- На мастере (my.cnf)
[mysqld]
server-id=1
log-bin=mysql-bin
gtid_mode=ON
enforce_gtid_consistency=ON
binlog_format=ROW

-- На слейве (my.cnf)
[mysqld]
server-id=2
relay-log=relay-bin
gtid_mode=ON
enforce_gtid_consistency=ON
log_slave_updates=ON
read_only=ON

-- Настройка репликации (проще чем с позициями!)
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='password',
  MASTER_AUTO_POSITION=1;  -- Автоматическая позиция!

START SLAVE;

-- Просмотр GTID
SHOW MASTER STATUS;  -- На мастере
SHOW SLAVE STATUS\G  -- Executed_Gtid_Set на слейве
```

**Semi-Synchronous репликация:**
```sql
-- На мастере
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
SET GLOBAL rpl_semi_sync_master_enabled = 1;
SET GLOBAL rpl_semi_sync_master_timeout = 10000;  -- 10 секунд

-- На слейве
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
SET GLOBAL rpl_semi_sync_slave_enabled = 1;

-- Перезапустить IO поток на слейве
STOP SLAVE IO_THREAD;
START SLAVE IO_THREAD;

-- Проверка
SHOW STATUS LIKE 'Rpl_semi_sync%';
```

**Multi-Source репликация (MySQL 5.7+):**
```sql
-- Слейв может реплицировать с нескольких мастеров
CHANGE MASTER TO
  MASTER_HOST='master1_ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=12345
  FOR CHANNEL 'master1';

CHANGE MASTER TO
  MASTER_HOST='master2_ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=12345
  FOR CHANNEL 'master2';

START SLAVE FOR CHANNEL 'master1';
START SLAVE FOR CHANNEL 'master2';

-- Проверка
SHOW SLAVE STATUS FOR CHANNEL 'master1'\G
```

**Мониторинг репликации:**
```sql
-- Задержка репликации
SHOW SLAVE STATUS\G
-- Смотреть Seconds_Behind_Master

-- Скрипт мониторинга
SELECT 
  VARIABLE_VALUE as Seconds_Behind_Master 
FROM performance_schema.global_status 
WHERE VARIABLE_NAME='Seconds_Behind_Master';

-- Мониторинг GTID
SELECT @@global.gtid_executed;
SELECT @@global.gtid_purged;

-- Проверка репликации (выполнить на мастере и слейве)
-- Мастер:
CREATE TABLE repl_test (id INT, ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP);
INSERT INTO repl_test (id) VALUES (1);

-- Слейв (через несколько секунд):
SELECT * FROM repl_test;
```

**Переключение мастера (failover):**
```bash
# 1. Убедиться что слейв догнал мастер
mysql -e "SHOW SLAVE STATUS\G" | grep Seconds_Behind_Master

# 2. Остановить репликацию
mysql -e "STOP SLAVE;"

# 3. Сделать слейв мастером
mysql -e "RESET MASTER;"
mysql -e "SET GLOBAL read_only = 0;"

# 4. На других слейвах переключить мастера
mysql -e "STOP SLAVE;"
mysql -e "CHANGE MASTER TO MASTER_HOST='new_master_ip', ..."
mysql -e "START SLAVE;"
```

**ProxySQL для балансировки:**
```sql
-- Установка ProxySQL
wget https://github.com/sysown/proxysql/releases/download/v2.5.0/proxysql_2.5.0-ubuntu22_amd64.deb
sudo dpkg -i proxysql_2.5.0-ubuntu22_amd64.deb
sudo systemctl start proxysql

-- Подключение к админке ProxySQL
mysql -u admin -padmin -h 127.0.0.1 -P6032

-- Добавление серверов
INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (1, 'master_ip', 3306);
INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (2, 'slave1_ip', 3306);
INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (2, 'slave2_ip', 3306);

-- Добавление пользователя
INSERT INTO mysql_users(username, password, default_hostgroup) VALUES ('app_user', 'password', 1);

-- Правила маршрутизации (SELECT на слейвы)
INSERT INTO mysql_query_rules(rule_id, active, match_pattern, destination_hostgroup, apply) 
VALUES (1, 1, '^SELECT.*FOR UPDATE, 1, 1);

INSERT INTO mysql_query_rules(rule_id, active, match_pattern, destination_hostgroup, apply) 
VALUES (2, 1, '^SELECT', 2, 1);

-- Применить конфигурацию
LOAD MYSQL SERVERS TO RUNTIME;
LOAD MYSQL USERS TO RUNTIME;
LOAD MYSQL QUERY RULES TO RUNTIME;
SAVE MYSQL SERVERS TO DISK;
SAVE MYSQL USERS TO DISK;
SAVE MYSQL QUERY RULES TO DISK;
```

### 💻 Задание

**Внимание:** Для полноценной практики нужно минимум 2 сервера/VM. Можно использовать Docker.

1. Проверь, настроена ли репликация в текущей системе:
   ```sql
   SHOW MASTER STATUS;
   SHOW SLAVE STATUS\G
   ```

2. Проверь server-id текущего сервера:
   ```sql
   SELECT @@server_id;
   ```

3. Проверь, включен ли binary log:
   ```sql
   SHOW VARIABLES LIKE 'log_bin';
   SHOW BINARY LOGS;
   ```

4. Если есть слейвы, проверь их задержку:
   ```sql
   SHOW SLAVE STATUS\G
   -- Найти Seconds_Behind_Master
   ```

5. Создай пользователя для репликации (даже если нет слейвов - для практики)

6. Изучи конфигурацию репликации в my.cnf:
   ```bash
   grep -E "server-id|log-bin|gtid" /etc/mysql/my.cnf
   ```

### 🚀 Бонус (новое)

- Настрой базовую Master-Slave репликацию между двумя Docker контейнерами:
  ```bash
  # docker-compose.yml для тестирования репликации
  version: '3'
  services:
    master:
      image: percona:8.0
      environment:
        MYSQL_ROOT_PASSWORD: root
      volumes:
        - ./master.cnf:/etc/my.cnf.d/master.cnf
    slave:
      image: percona:8.0
      environment:
        MYSQL_ROOT_PASSWORD: root
      volumes:
        - ./slave.cnf:/etc/my.cnf.d/slave.cnf
  ```
- Настрой GTID репликацию вместо классической
- Протестируй failover: остановить мастер и переключить слейв на мастер
- Установи и настрой Orchestrator для автоматического failover:
  ```bash
  wget https://github.com/openark/orchestrator/releases/download/v3.2.6/orchestrator_3.2.6_amd64.deb
  sudo dpkg -i orchestrator_3.2.6_amd64.deb
  ```
- Настрой ProxySQL для балансировки нагрузки между мастером и слейвами

---

## Модуль 6: Мониторинг и производительность (30 минут)

### 🎯 Напоминалка

**Системные переменные:**
```sql
-- Просмотр переменных
SHOW VARIABLES;
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES LIKE '%cache%';

-- Изменение (на время сессии)
SET max_connections = 500;

-- Изменение (глобально, до перезапуска)
SET GLOBAL max_connections = 500;

-- Постоянное изменение - в my.cnf:
[mysqld]
max_connections = 500

-- Динамические vs статические переменные
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';  -- Статическая (требует рестарт)
SHOW VARIABLES LIKE 'max_connections';          -- Динамическая
```

**Статус сервера:**
```sql
-- Общий статус
SHOW STATUS;
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Questions';
SHOW STATUS LIKE 'Uptime';

-- Важные метрики
SHOW STATUS LIKE 'Threads%';
SHOW STATUS LIKE 'Connections';
SHOW STATUS LIKE 'Slow_queries';
SHOW STATUS LIKE 'Aborted%';
SHOW STATUS LIKE 'Table_locks%';

-- InnoDB статистика
SHOW ENGINE INNODB STATUS\G

-- Открытые таблицы
SHOW OPEN TABLES;
SHOW STATUS LIKE 'Open_tables';
SHOW STATUS LIKE 'Opened_tables';
```

**Активные процессы:**
```sql
-- Список процессов
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;

-- Из information_schema
SELECT * FROM information_schema.PROCESSLIST;

-- Найти долгие запросы
SELECT * FROM information_schema.PROCESSLIST 
WHERE TIME > 10 
ORDER BY TIME DESC;

-- Убить процесс
KILL 12345;
KILL QUERY 12345;  -- Только запрос, не соединение
```

**Performance Schema (MySQL 5.6+):**
```sql
-- Включение (my.cnf)
[mysqld]
performance_schema = ON

-- Проверка
SHOW VARIABLES LIKE 'performance_schema';

-- Топ запросов по времени выполнения
SELECT 
  DIGEST_TEXT,
  COUNT_STAR,
  AVG_TIMER_WAIT/1000000000000 AS avg_time_sec,
  SUM_TIMER_WAIT/1000000000000 AS total_time_sec
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;

-- Топ запросов по количеству
SELECT 
  DIGEST_TEXT,
  COUNT_STAR,
  AVG_TIMER_WAIT/1000000000000 AS avg_time_sec
FROM performance_schema.events_statements_summary_by_digest
ORDER BY COUNT_STAR DESC
LIMIT 10;

-- I/O статистика по таблицам
SELECT 
  OBJECT_SCHEMA,
  OBJECT_NAME,
  COUNT_READ,
  COUNT_WRITE,
  SUM_TIMER_WAIT/1000000000000 AS total_time_sec
FROM performance_schema.table_io_waits_summary_by_table
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;

-- Использование индексов
SELECT 
  OBJECT_SCHEMA,
  OBJECT_NAME,
  INDEX_NAME,
  COUNT_STAR,
  SUM_TIMER_WAIT/1000000000000 AS total_time_sec
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE INDEX_NAME IS NOT NULL
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;

-- Неиспользуемые индексы
SELECT 
  OBJECT_SCHEMA,
  OBJECT_NAME,
  INDEX_NAME
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE INDEX_NAME IS NOT NULL
  AND COUNT_STAR = 0
  AND OBJECT_SCHEMA != 'mysql'
ORDER BY OBJECT_SCHEMA, OBJECT_NAME;

-- Сброс статистики
TRUNCATE TABLE performance_schema.events_statements_summary_by_digest;
```

**sys Schema (MySQL 5.7+):**
```sql
-- Топ запросов
SELECT * FROM sys.statement_analysis LIMIT 10;
SELECT * FROM sys.statements_with_runtimes_in_95th_percentile LIMIT 10;

-- Неиспользуемые индексы
SELECT * FROM sys.schema_unused_indexes;

-- Таблицы без первичного ключа
SELECT * FROM sys.schema_tables_with_full_table_scans;

-- I/O активность
SELECT * FROM sys.io_global_by_file_by_bytes LIMIT 10;

-- Ожидания
SELECT * FROM sys.waits_global_by_latency LIMIT 10;

-- Пользовательская статистика
SELECT * FROM sys.user_summary;
SELECT * FROM sys.user_summary_by_statement_type;

-- Память
SELECT * FROM sys.memory_global_total;
SELECT * FROM sys.memory_by_user_by_current_bytes;
```

**Slow Query Log:**
```sql
-- Включение (my.cnf)
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 2
log_queries_not_using_indexes = 1

-- Или динамически
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;
SET GLOBAL log_queries_not_using_indexes = 'ON';

-- Проверка
SHOW VARIABLES LIKE 'slow_query_log%';
SHOW VARIABLES LIKE 'long_query_time';
```

**Анализ slow query log:**
```bash
# mysqldumpslow (встроенная утилита)
mysqldumpslow -s t -t 10 /var/log/mysql/mysql-slow.log  # Топ по времени
mysqldumpslow -s c -t 10 /var/log/mysql/mysql-slow.log  # Топ по количеству
mysqldumpslow -s l -t 10 /var/log/mysql/mysql-slow.log  # Топ по lock time

# pt-query-digest (Percona Toolkit)
pt-query-digest /var/log/mysql/mysql-slow.log

# С фильтрацией
pt-query-digest --filter '$event->{user} eq "app_user"' /var/log/mysql/mysql-slow.log

# Вывод в таблицу
pt-query-digest --review h=localhost,D=test,t=query_review /var/log/mysql/mysql-slow.log
```

**Ключевые метрики для мониторинга:**
```sql
-- Connections
SHOW STATUS LIKE 'Max_used_connections';
SHOW VARIABLES LIKE 'max_connections';
-- Использование: Max_used_connections / max_connections (должно быть < 80%)

-- Threads
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Threads_running';

-- Buffer Pool (InnoDB)
SHOW STATUS LIKE 'Innodb_buffer_pool_%';
-- Важные:
-- Innodb_buffer_pool_read_requests  (чтения из кэша)
-- Innodb_buffer_pool_reads           (чтения с диска)
-- Hit rate = read_requests / (read_requests + reads) * 100% (должен быть > 99%)

-- Временные таблицы
SHOW STATUS LIKE 'Created_tmp%';
-- Created_tmp_disk_tables - плохо (на диске)
-- Created_tmp_tables - всего
-- Ratio = tmp_disk_tables / tmp_tables (должно быть < 10%)

-- Таблицы
SHOW STATUS LIKE 'Opened_tables';
SHOW VARIABLES LIKE 'table_open_cache';
-- Если Opened_tables растет быстро, увеличить table_open_cache

-- Блокировки
SHOW STATUS LIKE 'Table_locks_waited';
SHOW STATUS LIKE 'Table_locks_immediate';

-- Медленные запросы
SHOW STATUS LIKE 'Slow_queries';

-- Aborted connections
SHOW STATUS LIKE 'Aborted_connects';
SHOW STATUS LIKE 'Aborted_clients';
```

**Оптимизация конфигурации (my.cnf):**
```ini
[mysqld]
# === Основное ===
max_connections = 500                    # Макс. соединений
max_connect_errors = 100                 # После этого хост блокируется

# === InnoDB ===
innodb_buffer_pool_size = 8G            # 70-80% RAM для dedicated сервера
innodb_log_file_size = 512M             # Размер лога (чем больше, тем меньше I/O)
innodb_flush_log_at_trx_commit = 2      # 1=safe, 2=faster (риск потери 1 сек)
innodb_flush_method = O_DIRECT          # Избегать двойного кэширования

# === Query Cache (deprecated в MySQL 8.0) ===
query_cache_type = 0                    # Выключить в 8.0
query_cache_size = 0

# === Temporary Tables ===
tmp_table_size = 64M                    # Макс. размер временной таблицы в памяти
max_heap_table_size = 64M               # Должен быть = tmp_table_size

# === Thread Cache ===
thread_cache_size = 50                  # Кэш потоков

# === Table Cache ===
table_open_cache = 4000                 # Кэш открытых таблиц

# === Slow Query Log ===
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 2
log_queries_not_using_indexes = 1

# === Binary Log ===
log_bin = /var/log/mysql/mysql-bin
binlog_format = ROW
expire_logs_days = 7
max_binlog_size = 100M

# === Replication ===
server-id = 1
read_only = 0                           # 1 для слейвов

# === Performance Schema ===
performance_schema = ON

# === Безопасность ===
skip-name-resolve = 1
local_infile = 0
```

**Инструменты мониторинга:**
```bash
# mytop - top для MySQL
mytop -u root -p -d database_name

# innotop - мониторинг InnoDB
innotop -u root -p

# mysqladmin - встроенная утилита
mysqladmin -u root -p status
mysqladmin -u root -p extended-status
mysqladmin -u root -p processlist
mysqladmin -u root -p variables

# Percona Toolkit
pt-query-digest slow.log
pt-mysql-summary
pt-stalk                                # Собирает диагностику при проблемах
pt-diskstats
pt-summary

# Monitoring запросов в реальном времени
watch -n 1 'mysql -u root -p -e "SHOW PROCESSLIST"'
```

### 💻 Задание

Выполни задачи мониторинга:

1. Проверь текущее количество подключений:
   ```sql
   SHOW STATUS LIKE 'Threads_connected';
   SHOW VARIABLES LIKE 'max_connections';
   ```

2. Найди топ-5 самых медленных запросов (если включен Performance Schema):
   ```sql
   SELECT DIGEST_TEXT, COUNT_STAR, AVG_TIMER_WAIT/1000000000000 AS avg_time_sec
   FROM performance_schema.events_statements_summary_by_digest
   ORDER BY AVG_TIMER_WAIT DESC LIMIT 5;
   ```

3. Проверь hit rate InnoDB buffer pool:
   ```sql
   SHOW STATUS LIKE 'Innodb_buffer_pool_read%';
   ```

4. Найди все процессы, выполняющиеся дольше 5 секунд:
   ```sql
   SELECT * FROM information_schema.PROCESSLIST WHERE TIME > 5;
   ```

5. Проверь размер всех баз данных:
   ```sql
   SELECT 
     table_schema AS 'Database',
     ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
   FROM information_schema.TABLES
   GROUP BY table_schema
   ORDER BY SUM(data_length + index_length) DESC;
   ```

6. Проверь, включен ли slow query log

7. Посмотри текущую конфигурацию InnoDB:
   ```sql
   SHOW VARIABLES LIKE 'innodb%';
   ```

### 🚀 Бонус (новое)

- Установи и используй PMM (Percona Monitoring and Management):
  ```bash
  docker run -d -p 443:443 --name pmm-server percona/pmm-server:2
  # Добавить клиент
  pmm-admin config --server-insecure-tls --server-url=https://admin:admin@localhost:443
  pmm-admin add mysql --query-source=perfschema
  ```
- Настрой Prometheus + Grafana для мониторинга MySQL
- Создай скрипт, который проверяет основные метрики и алертит при проблемах:
  ```bash
  #!/bin/bash
  # Проверка connections
  CURRENT=$(mysql -e "SHOW STATUS LIKE 'Threads_connected'" | awk 'NR==2 {print $2}')
  MAX=$(mysql -e "SHOW VARIABLES LIKE 'max_connections'" | awk 'NR==2 {print $2}')
  USAGE=$((CURRENT * 100 / MAX))
  if [ $USAGE -gt 80 ]; then
    echo "ALERT: Connection usage is $USAGE%"
  fi
  ```
- Используй `pt-mysql-summary` для полного анализа сервера:
  ```bash
  pt-mysql-summary -- --user=root --password=pass
  ```

---

## Модуль 7: Troubleshooting типичных проблем (25 минут)

### 🎯 Напоминалка

**Проблема: MySQL не запускается**
```bash
# 1. Проверить статус
sudo systemctl status mysql

# 2. Посмотреть логи
sudo journalctl -u mysql -n 100
sudo tail -100 /var/log/mysql/error.log

# 3. Проверить синтаксис конфигурации
mysqld --help --verbose | head -n 20

# 4. Проверить права на файлы
ls -la /var/lib/mysql/
sudo chown -R mysql:mysql /var/lib/mysql/

# 5. Проверить место на диске
df -h
du -sh /var/lib/mysql/

# 6. Попробовать запустить в безопасном режиме
sudo mysqld_safe --skip-grant-tables --skip-networking &

# 7. Проверить порт
sudo netstat -tulpn | grep 3306
sudo lsof -i :3306

# Типичные ошибки:
# - Нет места на диске
# - Неправильные права на файлы
# - Ошибка в my.cnf
# - Порт уже занят
# - Поврежденные системные таблицы
```

**Проблема: Медленные запросы**
```sql
-- 1. Проверить EXPLAIN
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- 2. Проверить индексы
SHOW INDEX FROM users;

-- 3. Проверить статистику таблицы
SHOW TABLE STATUS LIKE 'users'\G

-- 4. Обновить статистику
ANALYZE TABLE users;

-- 5. Проверить фрагментацию
SELECT 
  TABLE_NAME,
  ROUND(DATA_LENGTH/1024/1024, 2) AS data_mb,
  ROUND(DATA_FREE/1024/1024, 2) AS data_free_mb
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'database_name'
  AND DATA_FREE > 0;

-- 6. О