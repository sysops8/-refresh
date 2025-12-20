# Redis Refresh: Ежегодный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые концепции Redis за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная задача, которую нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Предварительные требования:**
- Docker установлен и настроен
- redis-cli установлен (или доступ через Docker)
- Базовые знания CLI
- Python или другой язык для клиентских примеров (опционально)

---

## Модуль 1: Базовая архитектура и установка (20 минут)

### 🎯 Напоминалка

**Redis - что это:**
```
Redis (Remote Dictionary Server)
├── In-Memory Data Structure Store
├── Key-Value база данных
├── Поддерживает различные типы данных
├── Однопоточная архитектура (single-threaded)
└── Persistence опции (RDB, AOF)
```

**Основные характеристики:**
```bash
# Скорость
- Операции: ~100,000 ops/sec на одном ядре
- Latency: sub-millisecond
- In-Memory: все данные в RAM

# Persistence
- RDB: Point-in-time snapshots
- AOF: Append-Only File (лог всех операций)
- Hybrid: RDB + AOF

# Репликация
- Master-Slave (теперь Primary-Replica)
- Async репликация
- Redis Sentinel для HA
- Redis Cluster для шардирования
```

**Установка и запуск:**
```bash
# Docker (рекомендуется для практики)
docker run -d --name redis-test \
  -p 6379:6379 \
  redis:7-alpine

# С persistent storage
docker run -d --name redis-persist \
  -p 6379:6379 \
  -v redis-data:/data \
  redis:7-alpine redis-server --appendonly yes

# Проверка
docker exec -it redis-test redis-cli ping
# Должен вернуть: PONG

# Остановка и удаление
docker stop redis-test
docker rm redis-test
docker volume rm redis-data
```

**redis-cli основы:**
```bash
# Подключение
redis-cli
redis-cli -h localhost -p 6379
redis-cli -a password  # С аутентификацией

# Базовые команды
PING                    # Проверка соединения
INFO                    # Информация о сервере
INFO stats              # Статистика
INFO memory             # Использование памяти
CONFIG GET *            # Вся конфигурация
CONFIG GET maxmemory    # Конкретный параметр
DBSIZE                  # Количество ключей
KEYS *                  # Все ключи (опасно в prod!)
SCAN 0 COUNT 10         # Итеративный поиск ключей
FLUSHDB                 # Очистить текущую БД
FLUSHALL                # Очистить все БД
SHUTDOWN                # Остановить сервер
```

**Базовые операции с ключами:**
```bash
# SET/GET
SET key "value"
GET key
SETNX key "value"       # SET if Not eXists
SETEX key 60 "value"    # SET с TTL в секундах
GETSET key "new"        # GET старое и SET новое
MSET k1 "v1" k2 "v2"   # Multiple SET
MGET k1 k2             # Multiple GET

# TTL (Time To Live)
EXPIRE key 60           # Установить TTL в секундах
EXPIREAT key 1735689600 # Expire в Unix timestamp
TTL key                 # Проверить оставшееся время
PERSIST key             # Убрать TTL
PTTL key               # TTL в миллисекундах

# Удаление
DEL key                 # Удалить ключ
DEL key1 key2 key3     # Удалить несколько
UNLINK key             # Async удаление (лучше для больших ключей)

# Проверка существования
EXISTS key              # 1 если есть, 0 если нет
EXISTS k1 k2 k3        # Количество существующих

# Переименование
RENAME old new          # Переименовать
RENAMENX old new       # Переименовать if new не существует

# Тип данных
TYPE key               # Возвращает тип значения
```

**Конфигурация:**
```bash
# redis.conf основные параметры
bind 0.0.0.0                    # На каких IP слушать
port 6379                       # Порт
daemonize yes                   # Запуск как демон
pidfile /var/run/redis.pid      # PID файл
loglevel notice                 # debug, verbose, notice, warning
logfile /var/log/redis.log      # Лог файл

# Memory
maxmemory 2gb                   # Максимум памяти
maxmemory-policy allkeys-lru    # Политика eviction

# Persistence
save 900 1                      # RDB: save после 900 сек если 1 изменение
save 300 10                     # RDB: save после 300 сек если 10 изменений
save 60 10000                   # RDB: save после 60 сек если 10000 изменений
appendonly yes                  # Включить AOF
appendfsync everysec            # AOF sync: always, everysec, no

# Security
requirepass yourpassword        # Установить пароль
rename-command CONFIG ""        # Переименовать/отключить команды
```

### 💻 Задание

Подготовь тестовое окружение:

1. Запусти Redis в Docker:
   ```bash
   docker run -d --name redis-practice \
     -p 6379:6379 \
     redis:7-alpine redis-server --loglevel verbose
   ```

2. Подключись к Redis:
   ```bash
   docker exec -it redis-practice redis-cli
   ```

3. Выполни базовые операции:
   ```bash
   # Проверь соединение
   PING
   
   # Установи несколько ключей
   SET user:1000:name "John Doe"
   SET user:1000:email "john@example.com"
   SET user:1000:age 30
   
   # Установи ключ с TTL
   SETEX session:abc123 3600 "session_data"
   
   # Проверь TTL
   TTL session:abc123
   
   # Получи значения
   GET user:1000:name
   MGET user:1000:name user:1000:email user:1000:age
   
   # Проверь информацию о сервере
   INFO server
   INFO stats
   INFO memory
   
   # Посмотри все ключи
   KEYS *
   
   # Узнай количество ключей
   DBSIZE
   ```

4. Проверь использование памяти:
   ```bash
   INFO memory
   # Обрати внимание на:
   # - used_memory_human
   # - used_memory_peak_human
   # - mem_fragmentation_ratio
   ```

5. Поэкспериментируй с TTL:
   ```bash
   SET temp "test"
   EXPIRE temp 10
   TTL temp
   # Подожди несколько секунд
   TTL temp
   GET temp
   # Подожди пока истечет
   GET temp  # Должен вернуть (nil)
   ```

### 🚀 Бонус (новое)

**1. Используй Redis Stack** (Redis с модулями):
```bash
# Останови старый контейнер
docker stop redis-practice
docker rm redis-practice

# Запусти Redis Stack (включает JSON, Search, TimeSeries и др.)
docker run -d --name redis-stack \
  -p 6379:6379 \
  -p 8001:8001 \
  redis/redis-stack:latest

# Подключись
docker exec -it redis-stack redis-cli

# Попробуй RedisJSON
JSON.SET user:2000 $ '{"name":"Jane","age":25,"city":"NYC"}'
JSON.GET user:2000
JSON.GET user:2000 $.name
```

**2. Настрой мониторинг с помощью redis-cli:**
```bash
# MONITOR - показывает все команды в реальном времени
redis-cli MONITOR

# В другом терминале выполни команды и увидишь их в MONITOR

# SLOWLOG - логи медленных запросов
CONFIG SET slowlog-log-slower-than 1000  # микросекунды
CONFIG SET slowlog-max-len 128
SLOWLOG GET 10  # Последние 10 медленных запросов
SLOWLOG LEN     # Количество в логе
SLOWLOG RESET   # Очистить лог
```

**3. Используй redis-cli в скриптах:**
```bash
# Массовая вставка из файла
cat <<EOF > data.txt
SET key1 "value1"
SET key2 "value2"
SET key3 "value3"
EOF

cat data.txt | redis-cli --pipe

# Экспорт всех ключей
redis-cli --scan --pattern 'user:*' | while read key; do
  echo "SET $key \"$(redis-cli GET $key)\""
done > backup.redis

# Benchmark
redis-cli --intrinsic-latency 100  # Latency базовой системы
redis-bench -t set,get -n 100000 -q  # Benchmark SET/GET
```

---

## Модуль 2: Типы данных и структуры (30 минут)

### 🎯 Напоминалка

**Основные типы данных Redis:**

**1. String (строки и числа):**
```bash
# Строки
SET name "John"
GET name
APPEND name " Doe"          # "John Doe"
STRLEN name                 # 8

# Числа
SET counter 0
INCR counter                # 1
INCRBY counter 5            # 6
DECR counter                # 5
DECRBY counter 2            # 3
INCRBYFLOAT price 0.5       # Для float

# Bits
SETBIT key 10 1             # Установить бит
GETBIT key 10               # Получить бит
BITCOUNT key                # Количество установленных битов
```

**2. List (списки):**
```bash
# Операции слева (head)
LPUSH tasks "task1"         # Добавить слева
LPUSH tasks "task2" "task3" # Добавить несколько
LPOP tasks                  # Извлечь слева

# Операции справа (tail)
RPUSH tasks "task4"         # Добавить справа
RPOP tasks                  # Извлечь справа

# Чтение
LRANGE tasks 0 -1           # Все элементы
LRANGE tasks 0 10           # Первые 10
LINDEX tasks 0              # Элемент по индексу
LLEN tasks                  # Длина списка

# Модификация
LSET tasks 0 "new_task"     # Установить значение по индексу
LINSERT tasks BEFORE "task1" "new"  # Вставить перед
LTRIM tasks 0 99            # Обрезать до 100 элементов

# Блокирующие операции (для очередей)
BLPOP tasks 5               # Блокирующий pop, timeout 5 сек
BRPOP tasks 5
BRPOPLPUSH source dest 5    # Atomic move между списками
```

**3. Set (множества):**
```bash
# Добавление/удаление
SADD tags "redis" "database" "nosql"
SREM tags "nosql"           # Удалить элемент
SPOP tags                   # Извлечь случайный
SPOP tags 2                 # Извлечь 2 случайных

# Проверка и чтение
SISMEMBER tags "redis"      # Проверить наличие
SMEMBERS tags               # Все элементы
SCARD tags                  # Количество элементов
SRANDMEMBER tags 3          # 3 случайных без удаления

# Операции над множествами
SADD set1 "a" "b" "c"
SADD set2 "b" "c" "d"
SUNION set1 set2            # Объединение: a,b,c,d
SINTER set1 set2            # Пересечение: b,c
SDIFF set1 set2             # Разность: a

# С сохранением результата
SUNIONSTORE result set1 set2
SINTERSTORE result set1 set2
SDIFFSTORE result set1 set2
```

**4. Hash (хеш-таблицы):**
```bash
# Операции
HSET user:1000 name "John"
HSET user:1000 email "john@example.com" age 30
HMSET user:1001 name "Jane" email "jane@example.com"  # Deprecated, используй HSET

# Чтение
HGET user:1000 name
HMGET user:1000 name email
HGETALL user:1000           # Все поля
HKEYS user:1000             # Все ключи
HVALS user:1000             # Все значения
HLEN user:1000              # Количество полей

# Проверка
HEXISTS user:1000 name      # Проверить наличие поля
HSETNX user:1000 name "New" # SET if Not eXists

# Инкремент
HINCRBY user:1000 age 1
HINCRBYFLOAT user:1000 balance 10.50

# Удаление
HDEL user:1000 age
```

**5. Sorted Set (сортированные множества):**
```bash
# Добавление (score member)
ZADD leaderboard 100 "player1"
ZADD leaderboard 200 "player2" 150 "player3"

# Чтение по рангу (позиции)
ZRANGE leaderboard 0 -1              # Все по возрастанию score
ZRANGE leaderboard 0 -1 WITHSCORES   # С очками
ZREVRANGE leaderboard 0 9 WITHSCORES # Top 10 по убыванию

# Чтение по score
ZRANGEBYSCORE leaderboard 100 200    # score от 100 до 200
ZRANGEBYSCORE leaderboard -inf +inf  # Все
ZREVRANGEBYSCORE leaderboard 200 100 # В обратном порядке

# Количество
ZCARD leaderboard               # Всего элементов
ZCOUNT leaderboard 100 200      # С score от 100 до 200

# Score операции
ZSCORE leaderboard "player1"    # Получить score
ZINCRBY leaderboard 50 "player1" # Увеличить score

# Ранг (позиция)
ZRANK leaderboard "player1"     # Позиция (с 0)
ZREVRANK leaderboard "player1"  # Позиция в обратном порядке

# Удаление
ZREM leaderboard "player1"      # По member
ZREMRANGEBYRANK leaderboard 0 2 # По рангу
ZREMRANGEBYSCORE leaderboard 0 100  # По score

# Операции над множествами
ZUNIONSTORE result 2 zset1 zset2 WEIGHTS 2 3  # Union с весами
ZINTERSTORE result 2 zset1 zset2               # Intersection
```

**Паттерны именования ключей:**
```bash
# Используй разделители для иерархии
user:1000:profile
user:1000:settings
user:1000:sessions:abc123

# Для коллекций
users:active        # Set активных пользователей
users:by_country:US # Set пользователей из US

# Для временных данных
session:abc123:data
cache:user:1000:feed

# Метаданные
user:1000:created_at
user:1000:last_login
```

### 💻 Задание

Реализуй простое приложение "Социальная сеть":

1. **Пользователи (Hash):**
```bash
# Создай несколько пользователей
HSET user:1000 username "john_doe" email "john@example.com" name "John Doe" followers 0 following 0
HSET user:1001 username "jane_smith" email "jane@example.com" name "Jane Smith" followers 0 following 0
HSET user:1002 username "bob_wilson" email "bob@example.com" name "Bob Wilson" followers 0 following 0

# Создай индекс username -> id
SET username:john_doe 1000
SET username:jane_smith 1001
SET username:bob_wilson 1002
```

2. **Подписки (Set):**
```bash
# john подписывается на jane и bob
SADD user:1000:following 1001 1002
SADD user:1001:followers 1000
SADD user:1002:followers 1000

# Обнови счетчики
HINCRBY user:1000 following 2
HINCRBY user:1001 followers 1
HINCRBY user:1002 followers 1

# Проверь взаимную подписку
SINTER user:1000:following user:1001:following

# Найди общих подписчиков
SADD user:1003:followers 1000 1001
SINTER user:1002:followers user:1003:followers
```

3. **Посты (List + Hash):**
```bash
# Создай пост
INCR post:id  # Генерируй ID
HSET post:1 user_id 1000 content "Hello Redis!" likes 0 created_at "2024-01-15T10:00:00Z"

# Добавь в ленту пользователя
LPUSH user:1000:posts 1

# Добавь в ленты подписчиков (fan-out on write)
LPUSH user:1001:feed 1
LPUSH user:1002:feed 1

# Получи последние 10 постов
LRANGE user:1001:feed 0 9
```

4. **Лайки (Set):**
```bash
# jane лайкает пост
SADD post:1:likes 1001
HINCRBY post:1 likes 1

# Проверь, лайкнул ли пользователь
SISMEMBER post:1:likes 1001

# Получи всех кто лайкнул
SMEMBERS post:1:likes
```

5. **Топ пользователей по подписчикам (Sorted Set):**
```bash
# Добавь в рейтинг (score = followers)
ZADD users:by_followers 1 1002
ZADD users:by_followers 1 1001
ZADD users:by_followers 2 1000

# Топ 10
ZREVRANGE users:by_followers 0 9 WITHSCORES

# Обнови когда followers меняется
ZINCRBY users:by_followers 1 1001
```

6. **Проверь результаты:**
```bash
# Профиль пользователя
HGETALL user:1000

# Подписчики
SMEMBERS user:1000:followers
SCARD user:1000:followers

# Лента
LRANGE user:1001:feed 0 -1

# Пост с деталями
HGETALL post:1

# Топ пользователей
ZREVRANGE users:by_followers 0 -1 WITHSCORES
```

### 🚀 Бонус (новое)

**1. Используй Streams для ленты активности (Redis 5.0+):**
```bash
# Создай stream событий
XADD user:1000:activity * action "posted" post_id 1 timestamp "2024-01-15T10:00:00Z"
XADD user:1000:activity * action "liked" post_id 2 timestamp "2024-01-15T10:05:00Z"

# Читай последние события
XRANGE user:1000:activity - + COUNT 10
XREVRANGE user:1000:activity + - COUNT 10

# Читай новые события (как consumer)
XREAD COUNT 10 STREAMS user:1000:activity 0

# Consumer groups для масштабирования
XGROUP CREATE user:1000:activity mygroup 0
XREADGROUP GROUP mygroup consumer1 COUNT 10 STREAMS user:1000:activity >

# Длина stream
XLEN user:1000:activity

# Удаление старых событий
XTRIM user:1000:activity MAXLEN ~ 1000
```

**2. Используй HyperLogLog для подсчета уникальных просмотров:**
```bash
# Добавь просмотры поста
PFADD post:1:views user:1000 user:1001 user:1002
PFADD post:1:views user:1000  # Дубликат не учитывается

# Получи количество уникальных просмотров
PFCOUNT post:1:views

# Объедини просмотры нескольких постов
PFADD post:2:views user:1003 user:1004
PFMERGE total:views post:1:views post:2:views
PFCOUNT total:views
```

**3. Geo данные для локальных рекомендаций:**
```bash
# Добавь геолокации пользователей
GEOADD users:locations -122.27652 37.805186 user:1000  # San Francisco
GEOADD users:locations -118.24368 34.05223 user:1001   # Los Angeles
GEOADD users:locations -0.12574 51.50853 user:1002     # London

# Найди пользователей рядом (в радиусе 200 км)
GEORADIUS users:locations -122.27652 37.805186 200 km WITHDIST

# Расстояние между пользователями
GEODIST users:locations user:1000 user:1001 km
```

---
## Модуль 3: Persistence и Backup (продолжение)

bash

```bash
# Посмотри AOF файл (человекочитаемый)
docker exec -it redis-aof cat /data/appendonly.aof

# Сделай принудительный rewrite
BGREWRITEAOF

# Проверь размер до и после
docker exec -it redis-aof ls -lh /data/
```

6. **Протестируй восстановление из AOF:**

bash

```bash
# Останови контейнер
docker stop redis-aof

# Запусти снова
docker start redis-aof

# Подключись и проверь данные
docker exec -it redis-aof redis-cli
KEYS *
SMEMBERS tags
ZRANGE scores 0 -1 WITHSCORES
# Все данные должны быть на месте
```

7. **Создай backup скрипт:**

bash

```bash
cat > backup-redis.sh <<'EOF'
#!/bin/bash
BACKUP_DIR="/backup/redis"
DATE=$(date +%Y%m%d-%H%M%S)

mkdir -p $BACKUP_DIR

# RDB backup
docker exec redis-aof redis-cli BGSAVE
sleep 5
docker cp redis-aof:/data/dump.rdb $BACKUP_DIR/dump-$DATE.rdb

# AOF backup
docker exec redis-aof redis-cli BGREWRITEAOF
sleep 5
docker cp redis-aof:/data/appendonly.aof $BACKUP_DIR/appendonly-$DATE.aof

# Удали старые backup (старше 7 дней)
find $BACKUP_DIR -name "*.rdb" -mtime +7 -delete
find $BACKUP_DIR -name "*.aof" -mtime +7 -delete

echo "Backup completed: $DATE"
EOF

chmod +x backup-redis.sh
./backup-redis.sh
```

### 🚀 Бонус (новое)

**1. Используй Redis 7.0+ RDB с AOF timestamp:**

bash

```bash
# В redis.conf
aof-use-rdb-preamble yes

# Это создает гибридный файл:
# - Начало файла = RDB snapshot (быстрая загрузка)
# - Конец файла = AOF команды после snapshot
# Лучшее из обоих миров!
```

**2. Automatic failover backup с репликацией:**

bash

```bash
# Запусти master
docker run -d --name redis-master \
  -p 6379:6379 \
  redis:7-alpine

# Запусти replica для backup
docker run -d --name redis-backup \
  -p 6380:6379 \
  redis:7-alpine redis-server \
  --replicaof redis-master 6379 \
  --save 900 1 \
  --appendonly yes

# Backup делается на replica без нагрузки на master
docker exec redis-backup redis-cli BGSAVE
```

**3. Point-in-Time Recovery (PITR):**

bash

```bash
# Если включен AOF, можно восстановить на любую точку времени
# 1. Останови Redis
# 2. Найди нужную позицию в AOF по timestamp
# 3. Обрежь AOF файл до этой позиции
# 4. Запусти Redis

# Пример: восстановить на 10:00
docker exec redis-aof redis-cli SHUTDOWN
docker exec redis-aof sh -c "head -n 1000 /data/appendonly.aof > /data/appendonly-restored.aof"
# Замени файлы и запусти
```

---

## Модуль 4: Репликация и High Availability (30 минут)

### 🎯 Напоминалка

**Master-Replica репликация:**

bash

````bash
# На Replica
replicaof <master-ip> <master-port>
replicaof 192.168.1.100 6379

# Или через команду
REPLICAOF 192.168.1.100 6379
REPLICAOF NO ONE  # Стать master

# Настройки репликации
replica-read-only yes          # Replica только для чтения
replica-serve-stale-data yes   # Отдавать старые данные если связь потеряна
repl-diskless-sync no          # Sync через диск или напрямую
repl-backlog-size 1mb          # Размер буфера для partial resync
min-replicas-to-write 1        # Минимум реплик для записи
min-replicas-max-lag 10        # Макс lag в секундах
```

**Процесс репликации:**
```
1. Replica подключается к Master
2. Master делает BGSAVE (RDB snapshot)
3. Master отправляет RDB на Replica
4. Replica загружает RDB
5. Master отправляет буфер команд накопленных во время sync
6. Replica применяет команды
7. Continuous sync: Master отправляет все команды на Replica
````

**Redis Sentinel (HA):**

bash

```bash
# sentinel.conf
sentinel monitor mymaster 192.168.1.100 6379 2
sentinel auth-pass mymaster yourpassword
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 10000

# Запуск Sentinel
redis-sentinel /path/to/sentinel.conf

# Команды Sentinel
SENTINEL masters
SENTINEL master mymaster
SENTINEL replicas mymaster
SENTINEL sentinels mymaster
SENTINEL failover mymaster  # Ручной failover
SENTINEL reset mymaster     # Reset состояния
```

**Redis Cluster (шардирование):**

bash

```bash
# Минимум 3 master + 3 replica = 6 нод
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000

# Создание cluster
redis-cli --cluster create \
  127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 \
  127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
  --cluster-replicas 1

# Команды cluster
CLUSTER INFO
CLUSTER NODES
CLUSTER SLOTS
CLUSTER KEYSLOT key  # Определить слот для ключа

# Resharding
redis-cli --cluster reshard 127.0.0.1:7000
```

**Monitoring репликации:**

bash

```bash
# На Master
INFO replication

# Вывод:
# role:master
# connected_slaves:2
# slave0:ip=192.168.1.101,port=6379,state=online,offset=1234,lag=0
# slave1:ip=192.168.1.102,port=6379,state=online,offset=1234,lag=0

# На Replica
INFO replication
# role:slave
# master_host:192.168.1.100
# master_port:6379
# master_link_status:up
# master_last_io_seconds_ago:0
# master_sync_in_progress:0
```

### 💻 Задание

Настрой репликацию Master-Replica:

1. **Запусти Master:**

bash

```bash
docker network create redis-net

docker run -d --name redis-master \
  --network redis-net \
  -p 6379:6379 \
  redis:7-alpine redis-server \
  --appendonly yes \
  --requirepass masterpass
```

2. **Запусти 2 Replicas:**

bash

```bash
docker run -d --name redis-replica1 \
  --network redis-net \
  -p 6380:6379 \
  redis:7-alpine redis-server \
  --replicaof redis-master 6379 \
  --masterauth masterpass \
  --requirepass replicapass \
  --replica-read-only yes

docker run -d --name redis-replica2 \
  --network redis-net \
  -p 6381:6379 \
  redis:7-alpine redis-server \
  --replicaof redis-master 6379 \
  --masterauth masterpass \
  --requirepass replicapass \
  --replica-read-only yes
```

3. **Проверь репликацию:**

bash

```bash
# На Master
docker exec -it redis-master redis-cli -a masterpass
INFO replication
# Должен показать 2 connected slaves

# Добавь данные на Master
SET test:1 "value1"
SET test:2 "value2"
SADD myset "a" "b" "c"
```

4. **Проверь на Replicas:**

bash

```bash
# Replica 1
docker exec -it redis-replica1 redis-cli -a replicapass
GET test:1
SMEMBERS myset

# Replica 2
docker exec -it redis-replica2 redis-cli -a replicapass
GET test:2
SMEMBERS myset

# Попробуй записать на Replica (должно быть readonly)
SET test:3 "value3"
# Ошибка: READONLY You can't write against a read only replica
```

5. **Тестируй failover вручную:**

bash

```bash
# Останови Master
docker stop redis-master

# Повысь Replica1 до Master
docker exec -it redis-replica1 redis-cli -a replicapass
REPLICAOF NO ONE
INFO replication  # role должна быть master

# Переключи Replica2 на новый Master
docker exec -it redis-replica2 redis-cli -a replicapass
REPLICAOF redis-replica1 6379

# Теперь можешь писать в новый Master
docker exec -it redis-replica1 redis-cli -a replicapass
SET test:3 "value3"

# Проверь на Replica2
docker exec -it redis-replica2 redis-cli -a replicapass
GET test:3
```

6. **Мониторинг lag:**

bash

```bash
# Создай скрипт мониторинга
cat > monitor-replication.sh <<'EOF'
#!/bin/bash
while true; do
  echo "=== $(date) ==="
  docker exec redis-master redis-cli -a masterpass INFO replication | grep -E "role|connected_slaves|slave[0-9]"
  echo ""
  sleep 5
done
EOF

chmod +x monitor-replication.sh
./monitor-replication.sh
```

### 🚀 Бонус (новое)

**1. Настрой Redis Sentinel для автоматического failover:**

bash

```bash
# sentinel1.conf
cat > sentinel1.conf <<EOF
port 26379
sentinel monitor mymaster redis-master 6379 2
sentinel auth-pass mymaster masterpass
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 10000
EOF

# Запусти 3 Sentinel
docker run -d --name sentinel1 \
  --network redis-net \
  -p 26379:26379 \
  -v $(pwd)/sentinel1.conf:/etc/sentinel.conf \
  redis:7-alpine redis-sentinel /etc/sentinel.conf

docker run -d --name sentinel2 \
  --network redis-net \
  -p 26380:26379 \
  -v $(pwd)/sentinel1.conf:/etc/sentinel.conf \
  redis:7-alpine redis-sentinel /etc/sentinel.conf

docker run -d --name sentinel3 \
  --network redis-net \
  -p 26381:26379 \
  -v $(pwd)/sentinel1.conf:/etc/sentinel.conf \
  redis:7-alpine redis-sentinel /etc/sentinel.conf

# Проверь Sentinel
docker exec -it sentinel1 redis-cli -p 26379
SENTINEL masters
SENTINEL master mymaster
SENTINEL replicas mymaster

# Тестируй автоматический failover
docker stop redis-master
# Sentinel автоматически повысит одну из реплик
```

**2. Redis Cluster (3 master + 3 replica):**

bash

```bash
# Создай 6 нод
for port in 7000 7001 7002 7003 7004 7005; do
  docker run -d --name redis-$port \
    --network redis-net \
    -p $port:6379 \
    redis:7-alpine redis-server \
    --cluster-enabled yes \
    --cluster-config-file nodes.conf \
    --cluster-node-timeout 5000 \
    --appendonly yes \
    --port 6379
done

# Создай cluster
docker exec -it redis-7000 redis-cli --cluster create \
  redis-7000:6379 redis-7001:6379 redis-7002:6379 \
  redis-7003:6379 redis-7004:6379 redis-7005:6379 \
  --cluster-replicas 1

# Проверь cluster
docker exec -it redis-7000 redis-cli -c
CLUSTER INFO
CLUSTER NODES

# Тестируй sharding
SET user:1000 "data"  # Автоматически попадет на нужный shard
GET user:1000
```

---

## Модуль 5: Performance и Оптимизация (30 минут)

### 🎯 Напоминалка

**Memory Management:**

bash

```bash
# Политики eviction (когда memory full)
maxmemory-policy noeviction      # Ошибка при попытке записи
maxmemory-policy allkeys-lru     # Удалять любые ключи (LRU)
maxmemory-policy allkeys-lfu     # Удалять любые ключи (LFU - least frequently used)
maxmemory-policy allkeys-random  # Удалять случайные ключи
maxmemory-policy volatile-lru    # Удалять только с TTL (LRU)
maxmemory-policy volatile-lfu    # Удалять только с TTL (LFU)
maxmemory-policy volatile-random # Удалять случайные с TTL
maxmemory-policy volatile-ttl    # Удалять с наименьшим TTL

# Рекомендации:
# - allkeys-lru: для кеша
# - volatile-lru: для кеша + persistent данных
# - noeviction: для БД где потеря данных недопустима
```

**Memory анализ:**

bash

```bash
# Использование памяти
INFO memory
MEMORY STATS
MEMORY DOCTOR  # Рекомендации по оптимизации

# Память конкретного ключа
MEMORY USAGE key
MEMORY USAGE key SAMPLES 5  # С сэмплированием для коллекций

# Fragmentation
INFO memory | grep fragmentation
# mem_fragmentation_ratio > 1.5 = проблема
# Решение: рестарт Redis или MEMORY PURGE (Redis 4.0+)
```

**Slow Log:**

bash

```bash
# Настройка
CONFIG SET slowlog-log-slower-than 10000  # микросекунды (10ms)
CONFIG SET slowlog-max-len 128

# Просмотр
SLOWLOG GET 10
SLOWLOG LEN
SLOWLOG RESET

# Формат вывода:
# 1) id
# 2) timestamp
# 3) execution time (microseconds)
# 4) command + args
# 5) client IP:port
# 6) client name
```

**Latency Monitoring:**

bash

```bash
# Включить
CONFIG SET latency-monitor-threshold 100  # миллисекунды

# События латентности
LATENCY DOCTOR   # Анализ и рекомендации
LATENCY LATEST   # Последние события
LATENCY HISTORY command  # История для команды
LATENCY GRAPH command    # ASCII график
LATENCY RESET

# События:
# - command: медленные команды
# - fast-command: быстрые команды с всплесками
# - fork: fork для BGSAVE/BGREWRITEAOF
# - aof-write: запись в AOF
# - aof-fsync-always: fsync AOF
```

**Опасные команды (избегай в production):**

bash

```bash
KEYS *           # Блокирует сервер, используй SCAN
FLUSHALL/FLUSHDB # Удаляет всё
SAVE             # Блокирующий save, используй BGSAVE
SMEMBERS large_set  # Возвращает всё множество, используй SSCAN
HGETALL large_hash  # То же для hash
ZRANGE key 0 -1    # То же для sorted set
DEBUG commands   # Только для отладки
```

**Безопасные альтернативы:**

bash

```bash
# Вместо KEYS используй SCAN
SCAN 0 MATCH user:* COUNT 100
# Итеративный, не блокирует сервер

# Вместо SMEMBERS используй SSCAN
SSCAN myset 0 COUNT 100

# Вместо HGETALL используй HSCAN
HSCAN myhash 0 COUNT 100

# Вместо ZRANGE используй ZSCAN
ZSCAN myzset 0 COUNT 100
```

**Pipelining и массовые операции:**

bash

```bash
# Без pipelining: N round-trips
SET key1 val1
SET key2 val2
SET key3 val3

# С pipelining: 1 round-trip
# В redis-cli используй --pipe
echo -e "SET key1 val1\nSET key2 val2\nSET key3 val3" | redis-cli --pipe

# Массовые команды
MSET key1 val1 key2 val2 key3 val3
MGET key1 key2 key3
```

**Benchmarking:**

bash

```bash
# Базовый benchmark
redis-benchmark -t set,get -n 100000 -q

# С указанием размера данных
redis-benchmark -t set,get -n 100000 -d 100 -q

# Конкретные команды
redis-benchmark -t set -n 100000 -P 16 -q  # С pipelining

# Только latency
redis-benchmark -t ping -c 1 -n 100000

# Полный тест
redis-benchmark -h localhost -p 6379 -a password --csv
```

### 💻 Задание

Оптимизируй производительность Redis:

1. **Настрой memory management:**

bash

```bash
docker run -d --name redis-optimized \
  -p 6379:6379 \
  redis:7-alpine redis-server \
  --maxmemory 256mb \
  --maxmemory-policy allkeys-lru \
  --save "" \
  --appendonly no
```

2. **Заполни данными и анализируй memory:**

bash

```bash
docker exec -it redis-optimized redis-cli

# Создай разные типы данных
# Strings
for i in {1..1000}; do SET string:$i "value_$i"; done

# Hashes (более эффективны для объектов)
for i in {1..1000}; do HMSET user:$i name "user$i" email "user$i@example.com" age $((20 + RANDOM % 50)); done

# Lists
for i in {1..100}; do LPUSH list:$i item1 item2 item3 item4 item5; done

# Sets
for i in {1..100}; do SADD set:$i member1 member2 member3 member4 member5; done

# Sorted Sets
for i in {1..100}; do ZADD zset:$i 1 member1 2 member2 3 member3; done

# Проверь использование памяти
INFO memory
DBSIZE

# Память по типам
MEMORY USAGE string:1
MEMORY USAGE user:1
MEMORY USAGE list:1
MEMORY USAGE set:1
MEMORY USAGE zset:1
```

3. **Оптимизируй структуры:**

bash

```bash
# Hash более эффективен чем отдельные ключи
# Плохо: 1000 ключей
SET user:1:name "John"
SET user:1:email "john@example.com"
SET user:1:age 30

# Хорошо: 1 ключ
HMSET user:1 name "John" email "john@example.com" age 30

# Сравни память
MEMORY USAGE user:1:name
MEMORY USAGE user:1
```

4. **Настрой и проверь slow log:**

bash

```bash
# Настрой slow log
CONFIG SET slowlog-log-slower-than 1000  # 1ms
CONFIG SET slowlog-max-len 128

# Выполни медленные операции
DEBUG SLEEP 0.01
KEYS *  # Это будет медленно

# Проверь slow log
SLOWLOG GET 10

# Должен показать:
# 1) ID записи
# 2) Timestamp
# 3) Execution time в микросекундах
# 4) Команда и аргументы
```

5. **Benchmark производительности:**

bash

```bash
# Базовый benchmark
docker exec redis-optimized redis-benchmark -t set,get -n 100000 -q

# С разными размерами данных
docker exec redis-optimized redis-benchmark -t set,get -n 100000 -d 10 -q
docker exec redis-optimized redis-benchmark -t set,get -n 100000 -d 100 -q
docker exec redis-optimized redis-benchmark -t set,get -n 100000 -d 1000 -q

# С pipelining
docker exec redis-optimized redis-benchmark -t set -n 100000 -P 16 -q

# Latency тест
docker exec redis-optimized redis-benchmark -t ping -c 1 -n 10000

# Полный отчет в CSV
docker exec redis-optimized redis-benchmark --csv > benchmark-results.csv
```

6. **Используй SCAN вместо KEYS:**

bash

```bash
# Плохо (блокирует сервер)
KEYS user:*

# Хорошо (итеративный)
SCAN 0 MATCH user:* COUNT 100

# Скрипт для полного scan
cursor=0
while true; do
  result=$(redis-cli SCAN $cursor MATCH "user:*" COUNT 100)
  cursor=$(echo "$result" | head -1)
  echo "$result" | tail -n +2
  if [ "$cursor" = "0" ]; then
    break
  fi
done
```

### 🚀 Бонус (новое)

**1. Redis с аллокатором jemalloc (по умолчанию):**

bash

```bash
# Проверь текущий аллокатор
INFO memory | grep allocator

# jemalloc более эффективен для Redis
# Настройки в redis.conf:
# (обычно не требуется, уже оптимально)

# Мониторинг fragmentation
watch -n 1 'redis-cli INFO memory | grep fragmentation'
```

**2. Используй Redis модуль для анализа:**

bash

```bash
# RedisInsight - GUI для мониторинга
docker run -d --name redisinsight \
  -p 8001:8001 \
  redislabs/redisinsight:latest

# Открой http://localhost:8001
# Добавь свой Redis сервер
# Анализируй:
# - Memory usage по ключам
# - Slow commands
# - Key patterns
# - Recommendations
```

**3. Active defragmentation (Redis 4.0+):**

bash

```bash
# Автоматическая дефрагментация памяти
CONFIG SET activedefrag yes
CONFIG SET active-defrag-ignore-bytes 100mb
CONFIG SET active-defrag-threshold-lower 10
CONFIG SET active-defrag-threshold-upper 100
CONFIG SET active-defrag-cycle-min 5
CONFIG SET active-defrag-cycle-max 75

# Мониторинг
INFO memory | grep defrag
```

---

## Модуль 6: Security и Production Best Practices (25 минут)

### 🎯 Напоминалка

**Аутентификация:**

bash

```bash
# Простой пароль
requirepass your_strong_password

# Использование
redis-cli -a your_strong_password
AUTH your_strong_password

# ACL (Redis 6.0+) - более гибко
# Создание пользователей
ACL SETUSER alice on >password123 ~cached:* +get +set
ACL SETUSER bob on >secret456 ~* +@all -@dangerous

# Правила:
# on/off - включить/выключить пользователя
# >password - установить пароль
# ~pattern - разрешенные ключи (* = все)
# +command - разрешить команду
# -command - запретить команду
# +@category - разрешить категорию (@all, @read, @write, @dangerous)
# -@category - запретить категорию
```

**ACL команды:**

bash

```bash
# Просмотр
ACL LIST                  # Все пользователи и правила
ACL USERS                 # Список пользователей
ACL GETUSER alice         # Правила пользователя
ACL CAT                   # Категории команд
ACL CAT dangerous         # Команды в категории

# Управление
ACL SETUSER username ...  # Создать/изменить
ACL DELUSER username      # Удалить
ACL WHOAMI               # Текущий пользователь

# Сохранение
ACL SAVE                  # Сохранить в aclfile
CONFIG SET aclfile /path/to/users.acl
```

**Переименование опасных команд:**

bash

```bash
# В redis.conf
rename-command FLUSHDB ""              # Отключить
rename-command FLUSHALL ""
rename-command CONFIG "CONFIG_secret"   # Переименовать
rename-command KEYS ""

# Использование после переименования
CONFIG_secret GET maxmemory
```

**Network Security:**

bash

```bash
# Bind к конкретным интерфейсам
bind 127.0.0.1 ::1        # Только localhost
bind 0.0.0.0              # Все интерфейсы (опасно без firewall)
bind 10.0.1.100 10.0.1.101  # Конкретные IP

# Protected mode (по умолчанию в 3.2+)
protected-mode yes
# Если нет bind 127.0.0.1 и нет пароля - отклоняет подключения

# Отключить bind (только для dev)
bind 0.0.0.0
protected-mode no
# НЕ ДЕЛАЙ ТАК В PRODUCTION!
```

**TLS/SSL (Redis 6.0+):**

bash

```bash
# В redis.conf
port 0                    # Отключить обычный порт
tls-port 6379
tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt
tls-auth-clients yes      # Требовать клиентские сертификаты

# Подключение с TLS
redis-cli --tls \
  --cert /path/to/client.crt \
  --key /path/to/client.key \
  --cacert /path/to/ca.crt
```

**Firewall правила (iptables):**

bash

```bash
# Разрешить только с конкретных IP
iptables -A INPUT -p tcp -s 10.0.1.0/24 --dport 6379 -j ACCEPT
iptables -A INPUT -p tcp --dport 6379 -j DROP

# Или используй firewalld
firewall-cmd --permanent --zone=public --add-rich-rule='
  rule family="ipv4"
  source address="10.0.1.0/24"
  port protocol="tcp" port="6379" accept'
```

**Production конфигурация:**

bash

```bash
# redis.conf для production
# Memory
maxmemory 2gb
maxmemory-policy allkeys-lru

# Persistence (выбери одно или оба)
# RDB
save 900 1
save 300 10
save 60 10000
# AOF
appendonly yes
appendfsync everysec
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# Security
requirepass strong_password_here
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command CONFIG "CONFIG_a1b2c3"
bind 127.0.0.1
protected-mode yes

# Network
tcp-backlog 511
timeout 0
tcp-keepalive 300

# Limits
maxclients 10000

# Logging
loglevel notice
logfile /var/log/redis/redis-server.log

# Slow log
slowlog-log-slower-than 10000
slowlog-max-len 128

# Latency
latency-monitor-threshold 100

# Performance
# Отключи persistence для pure cache
# save ""
# appendonly no
```

**Мониторинг и алерты:**

bash

```bash
# Metrics для мониторинга
INFO stats | grep -E "total_commands_processed|instantaneous_ops_per_sec"
INFO memory | grep -E "used_memory_human|mem_fragmentation_ratio"
INFO replication | grep -E "role|connected_slaves|master_link_status"
INFO persistence | grep -E "rdb_last_save_time|aof_enabled"
INFO clients | grep -E "connected_clients|blocked_clients"

# Ключевые метрики:
# - used_memory / maxmemory
# - mem_fragmentation_ratio (должно быть < 1.5)
# - connected_clients / maxclients
# - evicted_keys (если > 0, нужно больше памяти)
# - rejected_connections
# - master_link_status (для replica)
# - rdb_last_save_time, aof_last_rewrite_time
```

### 💻 Задание

Настрой безопасный production Redis:

1. **Создай конфигурацию с security:**

bash

```bash
cat > redis-secure.conf <<EOF
# Network
bind 127.0.0.1
protected-mode yes
port 6379
tcp-backlog 511
timeout 300
tcp-keepalive 60

# Security
requirepass SecurePassword123!
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command CONFIG "CONFIG_xyz789"
rename-command DEBUG ""
rename-command SHUTDOWN "SHUTDOWN_xyz789"

# Memory
maxmem 512mb
maxmemory-policy allkeys-lru

# Persistence
appendonly yes appendfsync everysec save 900 1 save 300 10

# Logging
loglevel notice logfile ""

# Performance monitoring
slowlog-log-slower-than 10000 slowlog-max-len 128 latency-monitor-threshold 100

# Limits
maxclients 1000 EOF
```


2. **Запусти с этой конфигурацией:**
```bash
docker run -d --name redis-secure \
  -p 6379:6379 \
  -v $(pwd)/redis-secure.conf:/usr/local/etc/redis/redis.conf \
  redis:7-alpine redis-server /usr/local/etc/redis/redis.conf
```

3. **Настрой ACL (Redis 6.0+):**
```bash
# Подключись с паролем
docker exec -it redis-secure redis-cli -a SecurePassword123!

# Создай пользователей
ACL SETUSER admin on >AdminPass123 ~* +@all
ACL SETUSER readonly on >ReadPass123 ~* +@read -@write -@dangerous
ACL SETUSER app on >AppPass123 ~app:* +@all -@dangerous -@admin
ACL SETUSER cache on >CachePass123 ~cache:* +get +set +del +expire

# Проверь пользователей
ACL LIST
ACL USERS

# Сохрани ACL
ACL SAVE

# Тестируй readonly пользователя
AUTH readonly ReadPass123
GET key  # OK
SET key "value"  # Error: NOPERM

# Тестируй app пользователя
AUTH app AppPass123
SET app:user:1 "data"  # OK
SET other:key "data"  # Error: NOPERM
FLUSHDB  # Error: NOPERM
```

4. **Мониторинг security:**
```bash
# Создай скрипт мониторинга
cat > monitor-security.sh <<'EOF'
#!/bin/bash

REDIS_PASS="SecurePassword123!"

while true; do
  echo "=== Security Monitor $(date) ==="
  
  # Connected clients
  echo "Connected clients:"
  docker exec redis-secure redis-cli -a $REDIS_PASS CLIENT LIST | wc -l
  
  # Rejected connections
  echo "Rejected connections:"
  docker exec redis-secure redis-cli -a $REDIS_PASS INFO stats | grep rejected_connections
  
  # Auth failures (если есть модуль)
  # docker exec redis-secure redis-cli -a $REDIS_PASS ACL LOG 10
  
  # Slow commands
  echo "Slow commands:"
  docker exec redis-secure redis-cli -a $REDIS_PASS SLOWLOG LEN
  
  echo ""
  sleep 10
done
EOF

chmod +x monitor-security.sh
./monitor-security.sh
```

5. **Создай health check скрипт:**
```bash
cat > healthcheck.sh <<'EOF'
#!/bin/bash

REDIS_HOST="localhost"
REDIS_PORT="6379"
REDIS_PASS="SecurePassword123!"

# Ping test
if ! redis-cli -h $REDIS_HOST -p $REDIS_PORT -a $REDIS_PASS PING > /dev/null 2>&1; then
  echo "CRITICAL: Redis not responding"
  exit 2
fi

# Memory check
USED_MEMORY=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT -a $REDIS_PASS INFO memory | grep used_memory: | cut -d: -f2 | tr -d '\r')
MAX_MEMORY=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT -a $REDIS_PASS CONFIG GET maxmemory | tail -1)

if [ "$MAX_MEMORY" != "0" ]; then
  MEMORY_PCT=$((USED_MEMORY * 100 / MAX_MEMORY))
  if [ $MEMORY_PCT -gt 90 ]; then
    echo "WARNING: Memory usage ${MEMORY_PCT}%"
  fi
fi

# Replication check (if replica)
ROLE=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT -a $REDIS_PASS INFO replication | grep role: | cut -d: -f2 | tr -d '\r')
if [ "$ROLE" = "slave" ]; then
  MASTER_STATUS=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT -a $REDIS_PASS INFO replication | grep master_link_status: | cut -d: -f2 | tr -d '\r')
  if [ "$MASTER_STATUS" != "up" ]; then
    echo "CRITICAL: Master link down"
    exit 2
  fi
fi

echo "OK: Redis healthy"
exit 0
EOF

chmod +x healthcheck.sh
./healthcheck.sh
```

### 🚀 Бонус (новое)

**1. Настрой TLS для Redis:**
```bash
# Генерируй сертификаты
openssl genrsa -out ca-key.pem 4096
openssl req -new -x509 -days 3650 -key ca-key.pem -out ca-cert.pem \
  -subj "/CN=RedisCA"

openssl genrsa -out redis-key.pem 4096
openssl req -new -key redis-key.pem -out redis.csr \
  -subj "/CN=redis-server"
openssl x509 -req -days 3650 -in redis.csr -CA ca-cert.pem \
  -CAkey ca-key.pem -CAcreateserial -out redis-cert.pem

# redis-tls.conf
cat > redis-tls.conf <<EOF
port 0
tls-port 6379
tls-cert-file /tls/redis-cert.pem
tls-key-file /tls/redis-key.pem
tls-ca-cert-file /tls/ca-cert.pem
tls-auth-clients no
requirepass SecurePassword123!
EOF

# Запусти Redis с TLS
docker run -d --name redis-tls \
  -p 6379:6379 \
  -v $(pwd)/redis-tls.conf:/usr/local/etc/redis/redis.conf \
  -v $(pwd):/tls \
  redis:7-alpine redis-server /usr/local/etc/redis/redis.conf

# Подключись с TLS
redis-cli --tls --cert ./redis-cert.pem --key ./redis-key.pem \
  --cacert ./ca-cert.pem -a SecurePassword123!
```

**2. Redis с systemd hardening:**
```bash
# /etc/systemd/system/redis.service.d/hardening.conf
cat > /etc/systemd/system/redis.service.d/hardening.conf <<EOF
[Service]
# Security
NoNewPrivileges=true
PrivateTmp=true
PrivateDevices=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/redis /var/log/redis
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true
RestrictRealtime=true
RestrictNamespaces=true
LockPersonality=true
MemoryDenyWriteExecute=true
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX
SystemCallFilter=@system-service
SystemCallErrorNumber=EPERM

# Resource limits
LimitNOFILE=65535
LimitNPROC=65535
EOF

systemctl daemon-reload
systemctl restart redis
```

**3. Интеграция с Vault для secrets:**
```bash
# Храни пароли в Vault вместо конфига
# Используй init script для получения пароля

cat > redis-with-vault.sh <<'EOF'
#!/bin/bash
# Получи пароль из Vault
REDIS_PASS=$(vault kv get -field=password secret/redis/prod)

# Запусти Redis с паролем из Vault
redis-server --requirepass "$REDIS_PASS"
EOF
```

---

## Модуль 7: Patterns и Use Cases (30 минут)

### 🎯 Напоминалка

**1. Caching Pattern:**
```python
import redis
import json

r = redis.Redis(host='localhost', port=6379, decode_responses=True)

def get_user(user_id):
    # Cache-aside pattern
    cache_key = f"user:{user_id}"
    
    # Try cache first
    cached = r.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # Cache miss - get from DB
    user = get_from_database(user_id)
    
    # Store in cache with TTL
    r.setex(cache_key, 3600, json.dumps(user))
    
    return user

# Write-through pattern
def update_user(user_id, data):
    # Update DB
    update_database(user_id, data)
    
    # Update cache
    cache_key = f"user:{user_id}"
    r.setex(cache_key, 3600, json.dumps(data))
```

**2. Session Store:**
```python
def create_session(user_id):
    session_id = generate_session_id()
    session_key = f"session:{session_id}"
    
    session_data = {
        "user_id": user_id,
        "created_at": time.time(),
        "ip": request.remote_addr
    }
    
    # Session expires in 1 hour
    r.setex(session_key, 3600, json.dumps(session_data))
    return session_id

def get_session(session_id):
    session_key = f"session:{session_id}"
    data = r.get(session_key)
    if data:
        # Refresh TTL on access
        r.expire(session_key, 3600)
        return json.loads(data)
    return None
```

**3. Rate Limiting:**
```python
def rate_limit(user_id, max_requests=100, window=3600):
    key = f"rate_limit:{user_id}"
    
    # Sliding window counter
    current = r.incr(key)
    
    if current == 1:
        # First request - set expiry
        r.expire(key, window)
    
    return current <= max_requests

# Token bucket algorithm
def token_bucket_limit(user_id, tokens=10, refill_rate=1):
    key = f"tokens:{user_id}"
    
    current_tokens = r.get(key)
    if current_tokens is None:
        r.set(key, tokens - 1)
        r.expire(key, 60)
        return True
    
    current_tokens = int(current_tokens)
    if current_tokens > 0:
        r.decr(key)
        return True
    
    return False
```

**4. Distributed Lock:**
```python
import uuid
import time

def acquire_lock(lock_name, timeout=10):
    lock_key = f"lock:{lock_name}"
    identifier = str(uuid.uuid4())
    
    # SET NX (только если не существует) + EX (TTL)
    acquired = r.set(lock_key, identifier, nx=True, ex=timeout)
    
    return identifier if acquired else None

def release_lock(lock_name, identifier):
    lock_key = f"lock:{lock_name}"
    
    # Используй Lua script для atomic операции
    lua_script = """
    if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("del", KEYS[1])
    else
        return 0
    end
    """
    
    r.eval(lua_script, 1, lock_key, identifier)

# Использование
lock_id = acquire_lock("critical_section")
if lock_id:
    try:
        # Critical section
        perform_critical_operation()
    finally:
        release_lock("critical_section", lock_id)
```

**5. Pub/Sub Messaging:**
```python
# Publisher
def publish_message(channel, message):
    r.publish(channel, json.dumps(message))

# Subscriber
def subscribe_to_channel(channel):
    pubsub = r.pubsub()
    pubsub.subscribe(channel)
    
    for message in pubsub.listen():
        if message['type'] == 'message':
            data = json.loads(message['data'])
            handle_message(data)
```

**6. Leaderboard:**
```python
def add_score(user_id, score):
    r.zadd("leaderboard", {user_id: score})

def get_rank(user_id):
    # Rank (0-based, ascending)
    rank = r.zrank("leaderboard", user_id)
    return rank + 1 if rank is not None else None

def get_top_players(count=10):
    # Top players (descending)
    return r.zrevrange("leaderboard", 0, count-1, withscores=True)

def get_around_user(user_id, range=2):
    rank = r.zrank("leaderboard", user_id)
    if rank is None:
        return []
    
    start = max(0, rank - range)
    end = rank + range
    return r.zrange("leaderboard", start, end, withscores=True)
```

**7. Time Series Data:**
```python
def record_metric(metric_name, value, timestamp=None):
    if timestamp is None:
        timestamp = int(time.time())
    
    key = f"metrics:{metric_name}"
    r.zadd(key, {f"{timestamp}:{value}": timestamp})
    
    # Удали старые данные (> 24 часа)
    r.zremrangebyscore(key, 0, timestamp - 86400)

def get_metrics(metric_name, start_time, end_time):
    key = f"metrics:{metric_name}"
    data = r.zrangebyscore(key, start_time, end_time)
    
    return [(int(item.split(':')[0]), float(item.split(':')[1])) 
            for item in data]
```

**8. Job Queue:**
```python
def enqueue_job(queue_name, job_data):
    job_id = str(uuid.uuid4())
    job_key = f"job:{job_id}"
    
    # Store job data
    r.hset(job_key, mapping={
        "id": job_id,
        "data": json.dumps(job_data),
        "status": "pending",
        "created_at": time.time()
    })
    
    # Add to queue
    r.lpush(f"queue:{queue_name}", job_id)
    return job_id

def process_job(queue_name, timeout=30):
    # Blocking pop from queue
    result = r.brpop(f"queue:{queue_name}", timeout)
    
    if result:
        queue, job_id = result
        job_key = f"job:{job_id}"
        
        # Get job data
        job_data = r.hgetall(job_key)
        
        # Update status
        r.hset(job_key, "status", "processing")
        
        return job_id, json.loads(job_data["data"])
    
    return None, None
```

### 💻 Задание

Реализуй несколько практических use cases:

1. **Cache-aside pattern для API:**
```python
import redis
import json
import time

r = redis.Redis(host='localhost', port=6379, decode_responses=True)

# Симуляция медленной БД
def get_from_db(key):
    time.sleep(1)  # Симулируем задержку
    return {"data": f"value_for_{key}", "timestamp": time.time()}

def cached_get(key, ttl=60):
    cache_key = f"cache:{key}"
    
    # Try cache
    cached = r.get(cache_key)
    if cached:
        print(f"Cache HIT for {key}")
        return json.loads(cached)
    
    # Cache miss
    print(f"Cache MISS for {key}")
    data = get_from_db(key)
    
    # Store in cache
    r.setex(cache_key, ttl, json.dumps(data))
    return data

# Тест
start = time.time()
data1 = cached_get("user:1000")  # Miss, ~1 sec
print(f"First call: {time.time() - start:.2f}s")

start = time.time()
data2 = cached_get("user:1000")  # Hit, ~0.001 sec
print(f"Second call: {time.time() - start:.2f}s")
```

2. **Rate Limiter с разными алгоритмами:**
```python
# Fixed Window
def fixed_window_limit(user_id, max_requests=10, window=60):
    key = f"rate:{user_id}:{int(time.time() // window)}"
    current = r.incr(key)
    
    if current == 1:
        r.expire(key, window)
    
    return current <= max_requests

# Sliding Window Log
def sliding_window_limit(user_id, max_requests=10, window=60):
    key = f"rate:sliding:{user_id}"
    now = time.time()
    
    # Удали старые записи
    r.zremrangebyscore(key, 0, now - window)
    
    # Проверь текущее количество
    current_count = r.zcard(key)
    
    if current_count < max_requests:
        # Добавь новую запись
        r.zadd(key, {str(uuid.uuid4()): now})
        r.expire(key, window)
        return True
    
    return False

# Token Bucket (более точный)
def token_bucket(user_id, capacity=10, refill_rate=1):
    key = f"rate:bucket:{user_id}"
    last_key = f"rate:bucket:{user_id}:last"
    
    now = time.time()
    last_refill = float(r.get(last_key) or now)
    
    # Рассчитай текущие токены
    current_tokens = int(r.get(key) or capacity)
    time_passed = now - last_refill
    tokens_to_add = int(time_passed * refill_rate)
    current_tokens = min(capacity, current_tokens + tokens_to_add)
    
    if current_tokens > 0:
        # Используй токен
        r.set(key, current_tokens - 1)
        r.set(last_key, now)
        r.expire(key, 3600)
        r.expire(last_key, 3600)
        return True
    
    return False

# Тест
for i in range(15):
    allowed = fixed_window_limit("user:1", max_requests=10, window=60)
    print(f"Request {i+1}: {'ALLOWED' if allowed else 'BLOCKED'}")
    time.sleep(0.1)
```

3. **Distributed Lock для критических секций:**
```python
import uuid
import time

class RedisLock:
    def __init__(self, redis_client, lock_name, timeout=10):
        self.redis = redis_client
        self.lock_name = f"lock:{lock_name}"
        self.timeout = timeout
        self.identifier = None
    
    def __enter__(self):
        self.identifier = str(uuid.uuid4())
        
        # Retry logic
        for _ in range(10):
            acquired = self.redis.set(
                self.lock_name,
                self.identifier,
                nx=True,
                ex=self.timeout
            )
            if acquired:
                return self
            time.sleep(0.1)
        
        raise Exception(f"Could not acquire lock: {self.lock_name}")
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        # Atomic release using Lua
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        self.redis.eval(lua_script, 1, self.lock_name, self.identifier)

# Использование
def update_counter():
    with RedisLock(r, "counter_lock"):
        # Critical section
        current = int(r.get("counter") or 0)
        time.sleep(0.1)  # Симуляция работы
        r.set("counter", current + 1)

# Тест с многопоточностью
import threading

r.set("counter", 0)

threads = []
for i in range(10):
    t = threading.Thread(target=update_counter)
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print(f"Final counter: {r.get('counter')}")  # Должно быть 10
```

4. **Leaderboard с временными окнами:**
```python
def add_player_score(player_id, score, game_mode="ranked"):
    # Глобальный leaderboard
    r.zadd(f"leaderboard:{game_mode}", {player_id: score})
    
    # Недельный leaderboard
    week = int(time.time() // (7 * 86400))
    r.zadd(f"leaderboard:{game_mode}:week:{week}", {player_id: score})
    
    # Дневной leaderboard
    day = int(time.time() // 86400)
    r.zadd(f"leaderboard:{game_mode}:day:{day}", {player_id: score})

def get_leaderboard(game_mode="ranked", period="all", count=10):
    if period == "week":
        week = int(time.time() // (7 * 86400))
        key = f"leaderboard:{game_mode}:week:{week}"
    elif period == "day":
        day = int(time.time() // 86400)
        key = f"leaderboard:{game_mode}:day:{day}"
    else:
        key = f"leaderboard:{game_mode}"
    
    return r.zrevrange(key, 0, count-1, withscores=True)

def get_player_stats(player_id, game_mode="ranked"):
    # Глобальный ранг
    global_rank = r.zrevrank(f"leaderboard:{game_mode}", player_id)
    global_score = r.zscore(f"leaderboard:{game_mode}", player_id)
    
    # Недельный ранг
    week = int(time.time() // (7 * 86400))
    week_rank = r.zrevrank(f"leaderboard:{game_mode}:week:{week}", player_id)
    
    return {
        "global_rank": global_rank + 1 if global_rank is not None else None,
        "global_score": global_score,
        "week_rank": week_rank + 1 if week_rank is not None else None
    }

# Тест
for i in range(100):
    add_player_score(f"player:{i}", 1000 + i * 10)

print("Top 10:")
for rank, (player, score) in enumerate(get_leaderboard(count=10), 1):
    print(f"{rank}. {player}: {score}")

print("\nPlayer stats:")
print(get_player_stats("player:50"))
```

5. **Pub/Sub для real-time notifications:**
```python
import threading

# Publisher
def publish_notification(user_id, message):
    channel = f"notifications:{user_id}"
    r.publish(channel, json.dumps({
        "message": message,
        "timestamp": time.time()
    }))

# Subscriber
def subscribe_notifications(user_id):
    pubsub = r.pubsub()
    channel = f"notifications:{user_id}"
    pubsub.subscribe(channel)
    
    print(f"Subscribed to {channel}")
    
    for message in pubsub.listen():
        if message['type'] == 'message':
            data = json.loads(message['data'])
            print(f"[{user_id}] Received: {data['message']}")

# Тест
# В отдельном потоке запусти subscriber
subscriber_thread = threading.Thread(
    target=subscribe_notifications,
    args=("user:1000",)
)
subscriber_thread.daemon = True
subscriber_thread.start()

time.sleep(1)  # Дай подписаться

# Отправь notifications
for i in range(5):
    publish_notification("user:1000", f"Notification {i+1}")
    time.sleep(1)
```

### 🚀 Бонус (новое)

**1. Bloom Filter для membership testing:**
```python
# Используй RedisBloom модуль или реализуй свой
# pip install redis-py-cluster

from redisbloom.client import Client as RedisBloom

rb = RedisBloom(host='localhost', port=6379)

# Создай Bloom filter
rb.bfCreate("users:exists", 0.01, 1000000)

# Добавь элементы
for i in range(10000):
    rb.bfAdd("users:exists", f"user:{i}")

# Проверь существование (очень быстро, O(1))
print(rb.bfExists("users:exists", "user:5000"))  # True
print(rb.bfExists("users:exists", "user:99999"))  # False (может быть false positive)
```

**2. RedisJSON для complex структур:**
```python
# pip install rejson

from rejson import Client, Path

rj = Client(host='localhost', port=6379, decode_responses=True)

# Сохрани JSON
user = {
    "name": "John Doe",
    "email": "john@example.com",
    "address": {
        "city": "New York",
        "country": "USA"
    },
    "tags": ["redis", "python", "devops"]
}

rj.jsonset("user:1000", Path.rootPath(), user)

# Получи весь JSON
print(rj.jsonget("user:1000"))

# Получи часть
print(rj.jsonget("user:1000", Path('.address.city')))

# Обнови часть
rj.jsonset("user:1000", Path('.address.city'), "San Francisco")

# Добавь в массив
rj.jsonarrappend("user:1000", Path('.tags'), "docker")
```

**3. RedisTimeSeries для метрик:**
```python
# pip install redistimeseries

from redistimeseries.client import Client

rts = Client(host='localhost', port=6379)

# Создай time series
rts.create("metrics:cpu", retention_msecs=86400000)  # 24 hours

# Добавь данные
import time
for i in range(100):
    rts.add("metrics:cpu", int(time.time() * 1000), 50 + i % 50)
    time.sleep(0.1)

# Запроси данные
data = rts.range("metrics:cpu", 0, int
````

ВАЖНО: Продолжи выдачу в режиме UPDATE

18 дек.

Хорошо, продолжу именно в режиме UPDATE. Вот продолжение модуля 7:

python

```python
# Запроси данные
data = rts.range("metrics:cpu", 0, int(time.time() * 1000))
print(f"Data points: {len(data)}")

# Агрегация
avg = rts.range("metrics:cpu", 0, int(time.time() * 1000), 
                aggregation_type='avg', bucket_size_msec=10000)
print(f"Average CPU: {avg}")
```
## Модуль 8: Monitoring и Troubleshooting (25 минут)

### 🎯 Напоминалка

**Основные метрики для мониторинга:**

bash

```bash
# Memory
used_memory              # Используемая память
used_memory_rss          # RSS память (resident set size)
used_memory_peak         # Пиковое использование
mem_fragmentation_ratio  # Фрагментация (должно быть < 1.5)
maxmemory               # Лимит памяти
evicted_keys            # Вытесненные ключи

# Performance
instantaneous_ops_per_sec    # Операций в секунду
total_commands_processed     # Всего команд
keyspace_hits / keyspace_misses  # Hit rate кеша
expired_keys                 # Истекшие ключи

# Clients
connected_clients        # Подключенных клиентов
blocked_clients         # Заблокированных клиентов
rejected_connections    # Отклоненные подключения

# Persistence
rdb_last_save_time      # Последний RDB save
aof_last_rewrite_time   # Последний AOF rewrite
rdb_changes_since_last_save  # Изменений с последнего save

# Replication
connected_slaves        # Количество реплик
master_link_status      # Статус связи с master (up/down)
master_last_io_seconds_ago  # Последнее IO с master
```

**INFO команды:**

bash

```bash
INFO                    # Вся информация
INFO server            # Версия, uptime, OS
INFO clients           # Клиенты
INFO memory            # Память
INFO persistence       # RDB/AOF
INFO stats             # Статистика
INFO replication       # Репликация
INFO cpu               # CPU usage
INFO commandstats      # Статистика по командам
INFO keyspace          # Информация о ключах
INFO cluster           # Cluster info
```

**Prometheus exporter:**

bash

```bash
# Redis exporter для Prometheus
docker run -d --name redis-exporter \
  -p 9121:9121 \
  oliver006/redis_exporter:latest \
  --redis.addr=redis://redis:6379 \
  --redis.password=yourpass

# Метрики доступны на http://localhost:9121/metrics
```

**Типичные проблемы:**

**1. High Memory Usage:**

bash

```bash
# Проверь используемую память
INFO memory

# Найди большие ключи
redis-cli --bigkeys

# Или более детально
redis-cli --memkeys

# Analyze конкретный ключ
MEMORY USAGE keyname SAMPLES 5

# Решения:
# - Увеличь maxmemory
# - Настрой eviction policy
# - Оптимизируй структуры данных
# - Добавь TTL на ключи
```

**2. High Latency:**

bash

```bash
# Включи latency monitoring
CONFIG SET latency-monitor-threshold 100

# Проверь latency события
LATENCY LATEST
LATENCY DOCTOR
LATENCY GRAPH event-name

# Проверь slow log
SLOWLOG GET 10

# Причины:
# - Медленные команды (KEYS, SMEMBERS на больших наборах)
# - Swapping (проверь used_memory_rss vs used_memory)
# - Persistence (BGSAVE/AOF rewrite)
# - Сетевые проблемы
```

**3. Connection Issues:**

bash

```bash
# Проверь количество подключений
INFO clients

# Текущие клиенты
CLIENT LIST

# Убей проблемного клиента
CLIENT KILL ip:port

# Причины:
# - Достигнут maxclients
# - Сетевые проблемы
# - timeout слишком короткий
# - Клиент не закрывает соединения
```

**4. Replication Lag:**

bash

```bash
# На replica проверь
INFO replication

# master_last_io_seconds_ago - должно быть < 1
# master_link_status - должно быть "up"

# Причины:
# - Сетевая задержка между master и replica
# - Replica перегружена
# - Медленный диск на replica
# - repl_backlog_size слишком маленький
```

**Debugging команды:**

bash

```bash
# Real-time мониторинг команд
MONITOR

# Статистика по командам
INFO commandstats

# Память конкретного ключа
MEMORY USAGE key

# Debug команды (только для dev!)
DEBUG OBJECT key
DEBUG SEGFAULT  # Креш сервера!
```

### 💻 Задание

Настрой мониторинг и troubleshooting:

1. **Создай мониторинг скрипт:**

bash

```bash
cat > redis-monitor.sh <<'EOF'
#!/bin/bash

REDIS_HOST="localhost"
REDIS_PORT="6379"
REDIS_PASS=""

# Цвета для вывода
RED='\033[0;31m'
YELLOW='\033[1;33m'
GREEN='\033[0;32m'
NC='\033[0m'

while true; do
  clear
  echo "=== Redis Monitoring Dashboard ==="
  echo "Time: $(date)"
  echo ""
  
  # Memory
  echo "=== MEMORY ==="
  USED_MEM=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT INFO memory | grep "used_memory_human:" | cut -d: -f2)
  MAX_MEM=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT CONFIG GET maxmemory | tail -1)
  FRAG=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT INFO memory | grep "mem_fragmentation_ratio:" | cut -d: -f2)
  
  echo "Used Memory: $USED_MEM"
  echo "Max Memory: $MAX_MEM"
  echo "Fragmentation Ratio: $FRAG"
  
  # Performance
  echo ""
  echo "=== PERFORMANCE ==="
  OPS=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT INFO stats | grep "instantaneous_ops_per_sec:" | cut -d: -f2)
  HITS=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT INFO stats | grep "keyspace_hits:" | cut -d: -f2)
  MISSES=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT INFO stats | grep "keyspace_misses:" | cut -d: -f2)
  
  if [ "$MISSES" != "0" ]; then
    HIT_RATE=$(echo "scale=2; $HITS * 100 / ($HITS + $MISSES)" | bc)
  else
    HIT_RATE="100.00"
  fi
  
  echo "Ops/sec: $OPS"
  echo "Hit Rate: ${HIT_RATE}%"
  
  # Clients
  echo ""
  echo "=== CLIENTS ==="
  CLIENTS=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT INFO clients | grep "connected_clients:" | cut -d: -f2)
  BLOCKED=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT INFO clients | grep "blocked_clients:" | cut -d: -f2)
  
  echo "Connected: $CLIENTS"
  echo "Blocked: $BLOCKED"
  
  # Replication (if replica)
  ROLE=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT INFO replication | grep "role:" | cut -d: -f2 | tr -d '\r')
  if [ "$ROLE" = "slave" ]; then
    echo ""
    echo "=== REPLICATION ==="
    MASTER_STATUS=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT INFO replication | grep "master_link_status:" | cut -d: -f2)
    LAG=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT INFO replication | grep "master_last_io_seconds_ago:" | cut -d: -f2)
    echo "Master Link: $MASTER_STATUS"
    echo "Lag: ${LAG}s"
  fi
  
  # Alerts
  echo ""
  echo "=== ALERTS ==="
  
  # Check fragmentation
  FRAG_NUM=$(echo $FRAG | tr -d '\r')
  if (( $(echo "$FRAG_NUM > 1.5" | bc -l) )); then
    echo -e "${RED}HIGH FRAGMENTATION: $FRAG_NUM${NC}"
  fi
  
  # Check hit rate
  if (( $(echo "$HIT_RATE < 80" | bc -l) )); then
    echo -e "${YELLOW}LOW HIT RATE: ${HIT_RATE}%${NC}"
  fi
  
  sleep 5
done
EOF

chmod +x redis-monitor.sh
./redis-monitor.sh
```

2. **Настрой Prometheus exporter:**

bash

```bash
# Запусти Redis (если еще не запущен)
docker run -d --name redis-mon \
  --network redis-net \
  -p 6379:6379 \
  redis:7-alpine

# Запусти Redis Exporter
docker run -d --name redis-exporter \
  --network redis-net \
  -p 9121:9121 \
  oliver006/redis_exporter:latest \
  --redis.addr=redis://redis-mon:6379

# Проверь метрики
curl http://localhost:9121/metrics | grep redis_

# Запусти Prometheus (опционально)
cat > prometheus.yml <<EOF
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'redis'
    static_configs:
      - targets: ['localhost:9121']
EOF

docker run -d --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus

# Открой http://localhost:9090
# Query примеры:
# redis_memory_used_bytes
# rate(redis_commands_processed_total[1m])
# redis_connected_clients
```

3. **Troubleshooting сценарии:**

bash

```bash
# Сценарий 1: Найди большие ключи
redis-cli --bigkeys

# Сценарий 2: Analyze memory по pattern
redis-cli --memkeys --memkeys-samples 10000

# Сценарий 3: Найди медленные команды
redis-cli SLOWLOG GET 20

# Сценарий 4: Real-time мониторинг
redis-cli MONITOR | head -100

# Сценарий 5: Проверь latency
redis-cli --latency
redis-cli --latency-history
redis-cli --latency-dist

# Сценарий 6: Intrinsic latency (базовая система)
redis-cli --intrinsic-latency 30
```

4. **Создай alerting скрипт:**

bash

```bash
cat > redis-alerts.sh <<'EOF'
#!/bin/bash

REDIS_HOST="localhost"
REDIS_PORT="6379"
WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

check_memory() {
  USED=$(redis-cli -h $REDIS_HOST INFO memory | grep "used_memory:" | cut -d: -f2 | tr -d '\r')
  MAX=$(redis-cli -h $REDIS_HOST CONFIG GET maxmemory | tail -1)
  
  if [ "$MAX" != "0" ]; then
    PCT=$((USED * 100 / MAX))
    if [ $PCT -gt 90 ]; then
      send_alert "CRITICAL" "Memory usage at ${PCT}%"
    elif [ $PCT -gt 80 ]; then
      send_alert "WARNING" "Memory usage at ${PCT}%"
    fi
  fi
}

check_replication() {
  ROLE=$(redis-cli -h $REDIS_HOST INFO replication | grep "role:" | cut -d: -f2 | tr -d '\r')
  
  if [ "$ROLE" = "slave" ]; then
    STATUS=$(redis-cli -h $REDIS_HOST INFO replication | grep "master_link_status:" | cut -d: -f2 | tr -d '\r')
    if [ "$STATUS" != "up" ]; then
      send_alert "CRITICAL" "Master link is DOWN"
    fi
    
    LAG=$(redis-cli -h $REDIS_HOST INFO replication | grep "master_last_io_seconds_ago:" | cut -d: -f2 | tr -d '\r')
    if [ "$LAG" -gt 10 ]; then
      send_alert "WARNING" "Replication lag: ${LAG}s"
    fi
  fi
}

check_clients() {
  CLIENTS=$(redis-cli -h $REDIS_HOST INFO clients | grep "connected_clients:" | cut -d: -f2 | tr -d '\r')
  MAX=$(redis-cli -h $REDIS_HOST CONFIG GET maxclients | tail -1)
  
  PCT=$((CLIENTS * 100 / MAX))
  if [ $PCT -gt 90 ]; then
    send_alert "WARNING" "Client connections at ${PCT}% (${CLIENTS}/${MAX})"
  fi
}

send_alert() {
  LEVEL=$1
  MESSAGE=$2
  TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
  
  echo "[$TIMESTAMP] $LEVEL: $MESSAGE"
  
  # Отправь в Slack (раскомментируй если нужно)
  # curl -X POST -H 'Content-type: application/json' \
  #   --data "{\"text\":\"Redis Alert [$LEVEL]: $MESSAGE\"}" \
  #   $WEBHOOK_URL
}

# Main loop
while true; do
  check_memory
  check_replication
  check_clients
  sleep 60
done
EOF

chmod +x redis-alerts.sh
./redis-alerts.sh
```

### 🚀 Бонус (новое)

**1. Grafana Dashboard для Redis:**

bash

```bash
# Запусти Grafana
docker run -d --name grafana \
  -p 3000:3000 \
  grafana/grafana:latest

# Открой http://localhost:3000 (admin/admin)
# Добавь Prometheus data source
# Импортируй dashboard ID: 11835 (Redis Dashboard)

# Или создай свой dashboard с метриками:
# - redis_memory_used_bytes
# - rate(redis_commands_total[1m])
# - redis_connected_clients
# - redis_keyspace_hits_total / redis_keyspace_misses_total
```

**2. RedisInsight для GUI мониторинга:**

bash

```bash
docker run -d --name redisinsight \
  -p 8001:8001 \
  redis/redisinsight:latest

# Открой http://localhost:8001
# Добавь свой Redis
# Features:
# - Browser (просмотр ключей)
# - Workbench (CLI + command helper)
# - Analysis (memory analysis)
# - Profiler (real-time commands)
# - Slowlog viewer
```

**3. Custom healthcheck endpoint:**

python

````python
from flask import Flask, jsonify
import redis

app = Flask(__name__)
r = redis.Redis(host='localhost', port=6379)

@app.route('/health')
def health():
    try:
        # Ping test
        r.ping()
        
        # Memory check
        info = r.info('memory')
        memory_used = info['used_memory']
        memory_max = info.get('maxmemory', 0)
        
        # Replication check
        repl_info = r.info('replication')
        role = repl_info['role']
        
        status = {
            "status": "healthy",
            "memory_used_mb": memory_used / 1024 / 1024,
            "role": role
        }
        
        if role == 'slave':
            status['master_link'] = repl_info['master_link_status']
        
        return jsonify(status), 200
        
    except Exception as e:
        return jsonify({"status": "unhealthy", "error": str(e)}), 503

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

## Финальный проект (60 минут)

### Задача: Развернуть Production-Ready Redis Infrastructure

Создай полноценную Redis инфраструктуру с высокой доступностью, мониторингом и безопасностью.

**Архитектура:**
- 1 Master Redis
- 2 Replica Redis
- 3 Redis Sentinel (для автоматического failover)
- Redis Exporter (Prometheus metrics)
- Grafana (visualization)
- Backup система
- Security (пароли, ACL, TLS опционально)

**Структура проекта:**
```
redis-infrastructure/
├── docker-compose.yml
├── redis/
│   ├── master.conf
│   ├── replica.conf
│   └── sentinel.conf
├── scripts/
│   ├── backup.sh
│   ├── monitor.sh
│   └── healthcheck.sh
├── monitoring/
│   ├── prometheus.yml
│   └── grafana-dashboard.json
└── README.md
````

**1. docker-compose.yml:**

yaml

```yaml
version: '3.8'

services:
  redis-master:
    image: redis:7-alpine
    container_name: redis-master
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./redis/master.conf:/usr/local/etc/redis/redis.conf
      - redis-master-data:/data
    ports:
      - "6379:6379"
    networks:
      - redis-net

  redis-replica1:
    image: redis:7-alpine
    container_name: redis-replica1
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./redis/replica.conf:/usr/local/etc/redis/redis.conf
      - redis-replica1-data:/data
    ports:
      - "6380:6379"
    depends_on:
      - redis-master
    networks:
      - redis-net

  redis-replica2:
    image: redis:7-alpine
    container_name: redis-replica2
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./redis/replica.conf:/usr/local/etc/redis/redis.conf
      - redis-replica2-data:/data
    ports:
      - "6381:6379"
    depends_on:
      - redis-master
    networks:
      - redis-net

  sentinel1:
    image: redis:7-alpine
    container_name: sentinel1
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    volumes:
      - ./redis/sentinel.conf:/usr/local/etc/redis/sentinel.conf
    ports:
      - "26379:26379"
    depends_on:
      - redis-master
    networks:
      - redis-net

  sentinel2:
    image: redis:7-alpine
    container_name: sentinel2
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    volumes:
      - ./redis/sentinel.conf:/usr/local/etc/redis/sentinel.conf
    ports:
      - "26380:26379"
    depends_on:
      - redis-master
    networks:
      - redis-net

  sentinel3:
    image: redis:7-alpine
    container_name: sentinel3
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    volumes:
      - ./redis/sentinel.conf:/usr/local/etc/redis/sentinel.conf
    ports:
      - "26381:26379"
    depends_on:
      - redis-master
    networks:
      - redis-net

  redis-exporter:
    image: oliver006/redis_exporter:latest
    container_name: redis-exporter
    environment:
      - REDIS_ADDR=redis://redis-master:6379
      - REDIS_PASSWORD=YourStrongPassword123
    ports:
      - "9121:9121"
    depends_on:
      - redis-master
    networks:
      - redis-net

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"
    networks:
      - redis-net

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
    networks:
      - redis-net

volumes:
  redis-master-data:
  redis-replica1-data:
  redis-replica2-data:
  prometheus-data:
  grafana-data:

networks:
  redis-net:
    driver: bridge
```

**2. redis/master.conf:**

conf

```conf
bind 0.0.0.0
port 6379
protected-mode yes

# Security
requirepass YourStrongPassword123
masterauth YourStrongPassword123

# Memory
maxmemory 512mb
maxmemory-policy allkeys-lru

# Persistence
appendonly yes
appendfsync everysec
save 900 1
save 300 10
save 60 10000

# Replication
min-replicas-to-write 1
min-replicas-max-lag 10

# Logging
loglevel notice
slowlog-log-slower-than 10000
slowlog-max-len 128

# Performance
latency-monitor-threshold 100
```

**3. redis/replica.conf:**

conf

```conf
bind 0.0.0.0
port 6379
protected-mode yes

# Security
requirepass YourStrongPassword123
masterauth YourStrongPassword123

# Replication
replicaof redis-master 6379
replica-read-only yes

# Persistence
appendonly yes
appendfsync everysec

# Logging
loglevel notice
```

**4. redis/sentinel.conf:**

conf

```conf
port 26379
sentinel monitor mymaster redis-master 6379 2
sentinel auth-pass mymaster YourStrongPassword123
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 10000
```

**5. monitoring/prometheus.yml:**

yaml

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']
```

**6. scripts/backup.sh:**

bash

```bash
#!/bin/bash

BACKUP_DIR="/backup/redis"
DATE=$(date +%Y%m%d-%H%M%S)
REDIS_PASS="YourStrongPassword123"

mkdir -p $BACKUP_DIR

# Backup from replica (не нагружаем master)
docker exec redis-replica1 redis-cli -a $REDIS_PASS BGSAVE
sleep 5

docker cp redis-replica1:/data/dump.rdb $BACKUP_DIR/dump-$DATE.rdb
docker cp redis-replica1:/data/appendonly.aof $BACKUP_DIR/appendonly-$DATE.aof

# Compress
gzip $BACKUP_DIR/dump-$DATE.rdb
gzip $BACKUP_DIR/appendonly-$DATE.aof

# Удали старые (> 7 дней)
find $BACKUP_DIR -name "*.gz" -mtime +7 -delete

echo "Backup completed: $DATE"
```

**7. scripts/healthcheck.sh:**

bash

```bash
#!/bin/bash

REDIS_PASS="YourStrongPassword123"

# Check master
if ! docker exec redis-master redis-cli -a $REDIS_PASS PING > /dev/null 2>&1; then
  echo "CRITICAL: Master is DOWN"
  exit 2
fi

# Check replicas
for replica in redis-replica1 redis-replica2; do
  if ! docker exec $replica redis-cli -a $REDIS_PASS PING > /dev/null 2>&1; then
    echo "WARNING: $replica is DOWN"
  fi
done

# Check sentinel
SENTINELS=$(docker exec sentinel1 redis-cli -p 26379 SENTINEL sentinels mymaster | grep -c "name")
if [ "$SENTINELS" -lt 2 ]; then
  echo "WARNING: Less than 3 sentinels running"
fi

echo "OK: All services healthy"
exit 0
```

**Deployment и тестирование:**

bash

```bash
# 1. Создай структуру
mkdir -p redis-infrastructure/{redis,scripts,monitoring}
cd redis-infrastructure

# 2. Создай все конфиг файлы (см. выше)

# 3. Запусти инфраструктуру
docker-compose up -d

# 4. Проверь статус
docker-compose ps

# 5. Проверь репликацию
docker exec redis-master redis-cli -a YourStrongPassword123 INFO replication

# 6. Проверь Sentinel
docker exec sentinel1 redis-cli -p 26379 SENTINEL masters

# 7. Тестируй failover
docker stop redis-master
# Sentinel автоматически повысит реплику
docker exec sentinel1 redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster

# 8. Открой мониторинг
# Prometheus: http://localhost:9090
# Grafana: http://localhost:3000 (admin/admin)
#   - Добавь Prometheus data source (http://prometheus:9090)
#   - Импортируй dashboard ID: 11835

# 9. Запусти backup
chmod +x scripts/backup.sh
./scripts/backup.sh

# 10. Healthcheck
chmod +x scripts/healthcheck.sh
./scripts/healthcheck.sh
```

**Расширения (опционально):**

1. **Добавь HAProxy для load balancing:**

yaml

```yaml
haproxy:
  image: haproxy:latest
  volumes:
    - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
  ports:
    - "6380:6379"  # Read балансировка по репликам
```

2. **Добавь TLS encryption**
3. **Интегрируй с Vault для secrets**
4. **Настрой alerting в Grafana**
5. **Добавь CI/CD для автоматического deployment**

---

## Чек-лист навыков

После прохождения курса ты должен уметь:

### Базовые навыки:

- ✅ Устанавливать и настраивать Redis
- ✅ Работать с основными типами данных
- ✅ Использовать redis-cli уверенно
- ✅ Настраивать persistence (RDB/AOF)
- ✅ Создавать backup и восстанавливать данные
- ✅ Работать с TTL и expiration

### Продвинутые навыки:

- ✅ Настраивать Master-Replica репликацию
- ✅ Использовать Redis Sentinel для HA
- ✅ Настраивать Redis Cluster
- ✅ Оптимизировать производительность
- ✅ Применять security best practices
- ✅ Работать с ACL (Redis 6.0+)

### Expert навыки:

- ✅ Настраивать полный мониторинг (Prometheus/Grafana)
- ✅ Troubleshooting сложных проблем
- ✅ Реализовывать различные patterns (cache, rate limiting, locks)
- ✅ Использовать Redis modules (JSON, TimeSeries, Bloom)
- ✅ Проектировать HA архитектуры
- ✅ Автоматизировать backup и recovery

---

## Справочная секция: Быстрые шпаргалки

### Топ-20 команд Redis

bash

```bash
# Strings
SET key value
GET key
DEL key
EXISTS key
EXPIRE key seconds

# Lists
LPUSH key value
RPUSH key value
LPOP key
RPOP key
LRANGE key 0 -1

# Sets
SADD key member
SMEMBERS key
SISMEMBER key member

# Hashes
HSET key field value
HGET key field
HGETALL key

# Sorted Sets
ZADD key score member
ZRANGE key 0 -1 WITHSCORES
ZRANK key member

# Generic
KEYS pattern  # Опасно в prod!
SCAN cursor MATCH pattern COUNT count  # Используй это
TTL key
INFO
```

### Конфигурация для разных use cases

**Pure Cache:**

conf

```conf
maxmemory 2gb
maxmemory-policy allkeys-lru
save ""
appendonly no
```

**Persistent Database:**

conf

```conf
maxmemory 0
save 900 1
save 300 10
appendonly yes
appendfsync everysec
```

**Session Store:**

conf

```conf
maxmemory 1gb
maxmemory-policy volatile-lru
appendonly yes
```

**Queue:**

conf

```conf
maxmemory 512mb
maxmemory-policy noeviction
appendonly yes
```

### Performance Checklist

**Перед production:**

- ✅ Установлен maxmemory
- ✅ Выбрана eviction policy
- ✅ Настроен persistence (если нужен)
- ✅ Включен requirepass
- ✅ Настроен bind address
- ✅ Переименованы опасные команды
- ✅ Настроен slowlog
- ✅ Настроен latency monitoring
- ✅ Replica для HA (если критично)
- ✅ Sentinel для автоматического failover
- ✅ Мониторинг и alerting
- ✅ Backup стратегия
- ✅ Documented recovery procedure

### Troubleshooting Quick Guide

**High Memory:**

bash

```bash
INFO memory
MEMORY STATS
redis-cli --bigkeys
# Решение: eviction policy, TTL, больше памяти
```

**High Latency:**

bash

```bash
LATENCY DOCTOR
SLOWLOG GET 10
redis-cli --latency
```
#### Решение: избегай KEYS, используй SCAN, проверь swapping

**Connection Issues:**
```bash
INFO clients
CLIENT LIST
CONFIG GET maxclients
# Решение: увеличь maxclients, проверь firewall
```

**Replication Lag:**
```bash
INFO replication
# Смотри: master_last_io_seconds_ago
# Решение: сеть, replica resources, repl_backlog_size
```
## Модуль 9: Redis в облаках и управляемые сервисы (25 минут)

### 🎯 Напоминалка

**AWS ElastiCache for Redis:**

bash

```bash
# Особенности
- Полностью управляемый сервис
- Автоматические backups
- Multi-AZ для HA
- Автоматический failover
- Поддержка cluster mode
- Версии: Redis 6.x, 7.x
- Шифрование at-rest и in-transit

# Архитектуры
1. Single Node (dev/test)
2. Primary + Replica (HA)
3. Cluster Mode (шардирование + HA)

# Endpoint типы
- Primary Endpoint: для записи
- Reader Endpoint: для чтения (балансировка по репликам)
- Configuration Endpoint: для cluster mode

# Backup
- Автоматические snapshots (RDB)
- Manual snapshots
- Восстановление в новый cluster
```

**Azure Cache for Redis:**

bash

```bash
# Тиры
- Basic: Single node, без SLA
- Standard: Primary + Replica, 99.9% SLA
- Premium: Cluster, persistence, VNet
- Enterprise: Redis Enterprise, 99.999% SLA

# Особенности
- Geo-replication (Premium+)
- Data persistence: RDB и AOF
- VNet integration
- Private Link support
- Zone redundancy
- Active geo-replication (Enterprise)

# Scaling
- Vertical: изменение тира
- Horizontal: cluster sharding (Premium+)
```

**GCP Memorystore for Redis:**

bash

````bash
# Тиры
- Basic: Single node, 99.9% SLA
- Standard: HA с репликой, 99.9% SLA

# Особенности
- Автоматическое failover
- Automated backups
- Point-in-time recovery
- VPC peering
- Private IP
- Maintenance windows
- Import/Export

# Версии
- Redis 5.0, 6.x, 7.0
- Managed updates
```

**Сравнение облачных провайдеров:**
```
Feature              | AWS ElastiCache | Azure Cache | GCP Memorystore
---------------------|----------------|-------------|----------------
Max Memory           | 6.38 TB        | 1.2 TB      | 300 GB
Cluster Mode         | ✅              | ✅ Premium   | ❌
Geo-Replication      | ❌              | ✅ Premium   | ❌
Multi-AZ             | ✅              | ✅           | ✅
Encryption at-rest   | ✅              | ✅ Premium   | ✅
Encryption in-transit| ✅              | ✅           | ✅
VPC/VNet             | ✅              | ✅ Premium   | ✅
Backup/Restore       | ✅              | ✅           | ✅
Monitoring           | CloudWatch     | Azure Mon.  | Cloud Monitoring
Cost                 | $$$            | $$          | $$
````

**Terraform для AWS ElastiCache:**

hcl

```hcl
resource "aws_elasticache_replication_group" "redis" {
  replication_group_id       = "my-redis-cluster"
  replication_group_description = "Redis cluster for production"
  engine                     = "redis"
  engine_version            = "7.0"
  node_type                 = "cache.r6g.large"
  num_cache_clusters        = 3
  parameter_group_name      = "default.redis7"
  port                      = 6379
  
  # HA
  automatic_failover_enabled = true
  multi_az_enabled          = true
  
  # Security
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  auth_token                = var.redis_auth_token
  
  # Backup
  snapshot_retention_limit  = 7
  snapshot_window          = "03:00-05:00"
  
  # Maintenance
  maintenance_window       = "sun:05:00-sun:07:00"
  
  # Network
  subnet_group_name        = aws_elasticache_subnet_group.redis.name
  security_group_ids       = [aws_security_group.redis.id]
  
  tags = {
    Environment = "production"
    Project     = "myapp"
  }
}

resource "aws_elasticache_subnet_group" "redis" {
  name       = "redis-subnet-group"
  subnet_ids = var.private_subnet_ids
}

resource "aws_security_group" "redis" {
  name        = "redis-sg"
  description = "Security group for Redis"
  vpc_id      = var.vpc_id
  
  ingress {
    from_port   = 6379
    to_port     = 6379
    protocol    = "tcp"
    cidr_blocks = [var.vpc_cidr]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

**CloudFormation для AWS:**

yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  RedisReplicationGroup:
    Type: AWS::ElastiCache::ReplicationGroup
    Properties:
      ReplicationGroupId: my-redis
      ReplicationGroupDescription: Production Redis
      Engine: redis
      EngineVersion: '7.0'
      CacheNodeType: cache.r6g.large
      NumCacheClusters: 3
      AutomaticFailoverEnabled: true
      MultiAZEnabled: true
      AtRestEncryptionEnabled: true
      TransitEncryptionEnabled: true
      AuthToken: !Ref RedisAuthToken
      SnapshotRetentionLimit: 7
      SnapshotWindow: '03:00-05:00'
      PreferredMaintenanceWindow: 'sun:05:00-sun:07:00'
      CacheSubnetGroupName: !Ref RedisSubnetGroup
      SecurityGroupIds:
        - !Ref RedisSecurityGroup
```

**Подключение к облачному Redis:**

python

````python
import redis
import ssl

# AWS ElastiCache (with TLS)
r = redis.Redis(
    host='my-redis.abc123.ng.0001.use1.cache.amazonaws.com',
    port=6379,
    password='your-auth-token',
    ssl=True,
    ssl_cert_reqs=ssl.CERT_REQUIRED,
    decode_responses=True
)

# Azure Cache (with TLS)
r = redis.Redis(
    host='myredis.redis.cache.windows.net',
    port=6380,
    password='your-access-key',
    ssl=True,
    decode_responses=True
)

# GCP Memorystore (no TLS by default)
r = redis.Redis(
    host='10.0.0.3',  # Private IP
    port=6379,
    decode_responses=True
)
```

### 💻 Задание

Спроектируй облачную архитектуру (на бумаге или в draw.io):

1. **AWS Multi-AZ setup:**
```
Region: us-east-1
├── VPC: 10.0.0.0/16
│   ├── Private Subnet AZ-A: 10.0.1.0/24
│   │   └── ElastiCache Node 1 (Primary)
│   ├── Private Subnet AZ-B: 10.0.2.0/24
│   │   └── ElastiCache Node 2 (Replica)
│   └── Private Subnet AZ-C: 10.0.3.0/24
│       └── ElastiCache Node 3 (Replica)
├── Application Servers (EC2/ECS)
│   └── Connect to Primary Endpoint
└── CloudWatch Alarms
    ├── CPUUtilization > 75%
    ├── DatabaseMemoryUsagePercentage > 80%
    └── ReplicationLag > 30s
````

2. **Создай Cost Estimation:**

bash

```bash
# AWS Pricing Calculator
# Example: cache.r6g.large (3 nodes)
# - Instance: $0.226/hour * 3 = $0.678/hour
# - Data transfer: ~$0.09/GB
# - Backups: $0.085/GB-month
# Monthly: ~$500 + data transfer + backups

# Azure Cache Premium P1 (6GB)
# - Instance: ~$0.356/hour = ~$260/month
# - Geo-replication: additional cost

# GCP Memorystore Standard M1 (5GB)
# - Instance: ~$0.054/hour = ~$40/month
```

3. **Напиши миграцию с self-hosted на AWS:**

bash

```bash
#!/bin/bash
# migrate-to-aws.sh

SOURCE_HOST="self-hosted-redis.example.com"
SOURCE_PORT="6379"
SOURCE_PASS="source-password"

TARGET_HOST="my-redis.abc123.ng.0001.use1.cache.amazonaws.com"
TARGET_PORT="6379"
TARGET_PASS="target-auth-token"

# 1. Создай backup источника
redis-cli -h $SOURCE_HOST -p $SOURCE_PORT -a $SOURCE_PASS BGSAVE

# 2. Дождись завершения
while [ $(redis-cli -h $SOURCE_HOST -a $SOURCE_PASS INFO persistence | grep rdb_bgsave_in_progress | cut -d: -f2 | tr -d '\r') -eq 1 ]; do
  echo "Waiting for BGSAVE to complete..."
  sleep 5
done

# 3. Скопируй RDB файл
scp user@source-server:/var/lib/redis/dump.rdb ./dump.rdb

# 4. Для ElastiCache используй redis-cli с --pipe для массовой загрузки
# Или используй AWS DMS (Database Migration Service)

# 5. Верификация - сравни количество ключей
SOURCE_KEYS=$(redis-cli -h $SOURCE_HOST -p $SOURCE_PORT -a $SOURCE_PASS DBSIZE)
TARGET_KEYS=$(redis-cli -h $TARGET_HOST -p $TARGET_PORT -a $TARGET_PASS --tls DBSIZE)

echo "Source keys: $SOURCE_KEYS"
echo "Target keys: $TARGET_KEYS"

if [ "$SOURCE_KEYS" -eq "$TARGET_KEYS" ]; then
  echo "Migration successful!"
else
  echo "Migration incomplete. Keys mismatch."
  exit 1
fi
```

### 🚀 Бонус (новое)

**1. Multi-Region Disaster Recovery (AWS):**

bash

```bash
# Region 1 (Primary): us-east-1
# Region 2 (DR): us-west-2

# Setup:
# 1. ElastiCache в обоих регионах
# 2. Global Datastore (Redis 5.0.6+, cluster mode)
# 3. Автоматическая репликация между регионами
# 4. Failover < 1 минута

# Terraform
resource "aws_elasticache_global_replication_group" "example" {
  global_replication_group_id_suffix = "my-redis"
  primary_replication_group_id      = aws_elasticache_replication_group.primary.id
}

resource "aws_elasticache_replication_group" "secondary" {
  replication_group_id        = "my-redis-secondary"
  global_replication_group_id = aws_elasticache_global_replication_group.example.id
  # ... остальные настройки
}
```

**2. Azure Active Geo-Replication:**

bash

```bash
# Premium tier feature
# Два независимых кластера
# Асинхронная репликация
# Можно читать и писать в оба
# Conflict resolution автоматический

az redis create \
  --name myredis-primary \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Premium \
  --vm-size P1

az redis create \
  --name myredis-secondary \
  --resource-group myResourceGroup \
  --location westus \
  --sku Premium \
  --vm-size P1

# Настрой geo-replication
az redis link create \
  --name myredis-primary \
  --resource-group myResourceGroup \
  --linked-server myredis-secondary \
  --linked-server-location westus \
  --linked-server-role Secondary
```

**3. Hybrid Cloud setup:**

bash

```bash
# Self-hosted Redis + Cloud Redis
# Use case: gradual migration, burst capacity

# Setup VPN/Direct Connect между on-prem и cloud
# Настрой репликацию: on-prem master -> cloud replica
# Переключение: cloud становится master

# AWS Site-to-Site VPN
# Azure ExpressRoute
# GCP Cloud Interconnect
```

---

## Модуль 10: Advanced темы и новые фичи (30 минут)

### 🎯 Напоминалка

**Lua Scripting:**

lua

```lua
-- Атомарная операция: get и set
local current = redis.call('GET', KEYS[1])
if current then
  redis.call('SET', KEYS[1], ARGV[1])
  return current
else
  return nil
end
```

bash

```bash
# Использование
redis-cli EVAL "return redis.call('SET', KEYS[1], ARGV[1])" 1 mykey myvalue

# Загрузка скрипта
SCRIPT LOAD "script content"
# Возвращает SHA1 hash

# Выполнение по hash
EVALSHA sha1 numkeys key [key ...] arg [arg ...]

# Управление скриптами
SCRIPT EXISTS sha1 [sha1 ...]
SCRIPT FLUSH
SCRIPT KILL  # Убить running скрипт
```

**Типичные Lua patterns:**

lua

```lua
-- Rate limiting (token bucket)
local key = KEYS[1]
local capacity = tonumber(ARGV[1])
local timestamp = tonumber(ARGV[2])
local requested = tonumber(ARGV[3])

local current = redis.call('HMGET', key, 'tokens', 'timestamp')
local tokens = tonumber(current[1]) or capacity
local last_time = tonumber(current[2]) or timestamp

local delta = timestamp - last_time
local refill = delta * (capacity / 60)  -- refill rate per second
tokens = math.min(capacity, tokens + refill)

if tokens >= requested then
  tokens = tokens - requested
  redis.call('HMSET', key, 'tokens', tokens, 'timestamp', timestamp)
  return 1
else
  return 0
end

-- Distributed lock с TTL
local key = KEYS[1]
local id = ARGV[1]
local ttl = ARGV[2]

if redis.call('SET', key, id, 'NX', 'EX', ttl) then
  return 1
else
  return 0
end
```

**Transactions (MULTI/EXEC):**

bash

```bash
# Optimistic locking с WATCH
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC

# Если mykey изменился между WATCH и EXEC - transaction отменяется
```

python

```python
# Python пример
def increment_with_retry(r, key, max_retries=10):
    for i in range(max_retries):
        try:
            with r.pipeline() as pipe:
                pipe.watch(key)
                current = int(pipe.get(key) or 0)
                pipe.multi()
                pipe.set(key, current + 1)
                pipe.execute()
                return current + 1
        except redis.WatchError:
            continue
    raise Exception("Too many retries")
```

**Redis Streams (детально):**

bash

```bash
# Добавление в stream
XADD mystream * field1 value1 field2 value2
XADD mystream MAXLEN ~ 1000 * field value  # С ограничением длины

# Чтение
XRANGE mystream - +           # Все сообщения
XRANGE mystream 1609459200000 +  # С определенного timestamp
XREAD COUNT 10 STREAMS mystream 0  # Последние 10

# Consumer Groups
XGROUP CREATE mystream mygroup 0
XREADGROUP GROUP mygroup consumer1 COUNT 10 STREAMS mystream >

# Acknowledge
XACK mystream mygroup message-id

# Pending messages
XPENDING mystream mygroup
XPENDING mystream mygroup - + 10 consumer1

# Claim pending messages (если consumer умер)
XCLAIM mystream mygroup consumer2 3600000 message-id

# Auto-claim (Redis 6.2+)
XAUTOCLAIM mystream mygroup consumer2 3600000 0
```

**Redis Modules:**

**1. RedisJSON:**

bash

```bash
# Установка (если self-hosted)
redis-server --loadmodule /path/to/rejson.so

# Команды
JSON.SET user:1 $ '{"name":"John","age":30,"address":{"city":"NYC"}}'
JSON.GET user:1
JSON.GET user:1 $.name
JSON.GET user:1 $.address.city

# Модификация
JSON.SET user:1 $.age 31
JSON.NUMINCRBY user:1 $.age 1
JSON.ARRAPPEND user:1 $.tags '"redis"'

# Удаление
JSON.DEL user:1 $.address
```

**2. RedisSearch:**

bash

```bash
# Создание индекса
FT.CREATE idx:users ON HASH PREFIX 1 user: SCHEMA name TEXT age NUMERIC city TAG

# Добавление данных (обычный HSET)
HSET user:1 name "John Doe" age 30 city "NYC"

# Поиск
FT.SEARCH idx:users "John"
FT.SEARCH idx:users "@city:{NYC}"
FT.SEARCH idx:users "@age:[25 35]"
FT.SEARCH idx:users "John @city:{NYC}"

# Агрегация
FT.AGGREGATE idx:users "*" GROUPBY 1 @city REDUCE COUNT 0 AS count
```

**3. RedisTimeSeries:**

bash

```bash
# Создание time series
TS.CREATE temperature:sensor1 RETENTION 86400000 LABELS sensor_id 1 location room1

# Добавление данных
TS.ADD temperature:sensor1 * 23.5
TS.MADD temperature:sensor1 1609459200000 22.1 temperature:sensor2 1609459200000 24.3

# Запросы
TS.RANGE temperature:sensor1 1609459200000 1609545600000
TS.RANGE temperature:sensor1 - + AGGREGATION avg 3600000  # Hourly average

# Multi-series запросы
TS.MRANGE 1609459200000 + FILTER location=room1

# Правила агрегации
TS.CREATERULE temperature:sensor1 temperature:sensor1:avg AGGREGATION avg 60000
```

**4. RedisBloom (Probabilistic Data Structures):**

bash

```bash
# Bloom Filter
BF.RESERVE users:exists 0.01 1000000
BF.ADD users:exists user:1
BF.EXISTS users:exists user:1
BF.MADD users:exists user:2 user:3 user:4

# Cuckoo Filter (может удалять)
CF.RESERVE items:exists 1000000
CF.ADD items:exists item:1
CF.DEL items:exists item:1

# Count-Min Sketch (подсчет частоты)
CMS.INITBYDIM page:views 1000 10
CMS.INCRBY page:views /home 1 /about 1
CMS.QUERY page:views /home

# Top-K (топ K элементов)
TOPK.RESERVE popular:items 10 1000 10 0.9
TOPK.ADD popular:items item1 item2 item3
TOPK.LIST popular:items
```

**Redis 7.0 новые фичи:**

bash

```bash
# Redis Functions (замена Lua scripting)
FUNCTION LOAD "#!lua name=mylib
redis.register_function('myfunc', function(keys, args)
  return redis.call('GET', keys[1])
end)"

FCALL myfunc 1 mykey

# Sharded Pub/Sub
SSUBSCRIBE channel  # Subscribe на конкретный shard
SPUBLISH channel message

# ACL v2
ACL SETUSER alice +@all -@dangerous

# Command introspection
COMMAND DOCS SET
COMMAND GETKEYS SET key value

# EXPIRETIME / PEXPIRETIME
SET key value EX 100
EXPIRETIME key  # Возвращает Unix timestamp истечения
```

**Pipeline и массовые операции:**

python

```python
import redis

r = redis.Redis()

# Без pipeline: N round-trips
for i in range(10000):
    r.set(f"key:{i}", f"value:{i}")

# С pipeline: 1 round-trip (или несколько по 1000)
pipe = r.pipeline()
for i in range(10000):
    pipe.set(f"key:{i}", f"value:{i}")
pipe.execute()

# Транзакции в pipeline
pipe = r.pipeline(transaction=True)
pipe.multi()
pipe.incr("counter")
pipe.expire("counter", 60)
pipe.execute()
```

### 💻 Задание

Реализуй продвинутые patterns:

**1. Rate Limiter на Lua:**

lua

```lua
-- rate_limiter.lua
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local current_time = tonumber(ARGV[3])

local window_start = current_time - window
redis.call('ZREMRANGEBYSCORE', key, '-inf', window_start)

local current_count = redis.call('ZCARD', key)

if current_count < limit then
  redis.call('ZADD', key, current_time, current_time)
  redis.call('EXPIRE', key, window)
  return 1
else
  return 0
end
```

python

```python
# Использование
import time
import redis

r = redis.Redis()

with open('rate_limiter.lua', 'r') as f:
    rate_limiter_script = f.read()

rate_limiter = r.register_script(rate_limiter_script)

def check_rate_limit(user_id, limit=10, window=60):
    key = f"rate_limit:{user_id}"
    current_time = int(time.time())
    return rate_limiter(keys=[key], args=[limit, window, current_time])

# Тест
for i in range(15):
    allowed = check_rate_limit("user:123", limit=10, window=60)
    print(f"Request {i+1}: {'ALLOWED' if allowed else 'BLOCKED'}")
```

**2. Distributed Queue с приоритетами:**

python

```python
import redis
import json
import time

class PriorityQueue:
    def __init__(self, redis_client, queue_name):
        self.r = redis_client
        self.queue = f"queue:{queue_name}"
        self.processing = f"processing:{queue_name}"
    
    def enqueue(self, task, priority=0):
        """Priority: lower = higher priority"""
        task_data = json.dumps(task)
        self.r.zadd(self.queue, {task_data: priority})
    
    def dequeue(self, timeout=10):
        """Dequeue с переносом в processing set"""
        # Atomic: get task with lowest score
        pipeline = self.r.pipeline()
        pipeline.watch(self.queue)
        
        tasks = self.r.zrange(self.queue, 0, 0)
        if not tasks:
            return None
        
        task_data = tasks[0]
        
        pipeline.multi()
        pipeline.zrem(self.queue, task_data)
        pipeline.sadd(self.processing, task_data)
        pipeline.execute()
        
        return json.loads(task_data)
    
    def complete(self, task):
        """Mark task as completed"""
        task_data = json.dumps(task)
        self.r.srem(self.processing, task_data)
    
    def requeue_processing(self):
        """Requeue stuck tasks from processing set"""
        processing_tasks = self.r.smembers(self.processing)
        for task_data in processing_tasks:
            self.r.srem(self.processing, task_data)
            self.r.zadd(self.queue, {task_data: 0})

# Использование
r = redis.Redis()
q = PriorityQueue(r, "tasks")

# Producer
q.enqueue({"type": "email", "to": "user@example.com"}, priority=5)
q.enqueue({"type": "sms", "to": "+1234567890"}, priority=1)  # Higher priority
q.enqueue({"type": "notification", "user_id": 123}, priority=10)

# Consumer
while True:
    task = q.dequeue()
    if task:
        print(f"Processing: {task}")
        # Process task
        time.sleep(1)
        q.complete(task)
    else:
        time.sleep(0.1)
```

**3. Session Store с автоматическим обновлением TTL:**

python

```python
import redis
import json
import uuid
from datetime import datetime, timedelta

class SessionStore:
    def __init__(self, redis_client, ttl=3600):
        self.r = redis_client
        self.ttl = ttl
    
    def create(self, user_id, data=None):
        """Create new session"""
        session_id = str(uuid.uuid4())
        session_key = f"session:{session_id}"
        
        session_data = {
            "session_id": session_id,
            "user_id": user_id,
            "created_at": datetime.now().isoformat(),
            "last_access": datetime.now().isoformat(),
            "data": data or {}
        }
        
        self.r.setex(session_key, self.ttl, json.dumps(session_data))
        return session_id
    
    def get(self, session_id, refresh=True):
        """Get session and optionally refresh TTL"""
        session_key = f"session:{session_id}"
        data = self.r.get(session_key)
        
        if not data:
            return None
        
        session = json.loads(data)
        
        if refresh:
            session["last_access"] = datetime.now().isoformat()
            self.r.setex(session_key, self.ttl, json.dumps(session))
        
        return session
    
    def update(self, session_id, data):
        """Update session data"""
        session = self.get(session_id, refresh=False)
        if not session:
            return False
        
        session["data"].update(data)
        session["last_access"] = datetime.now().isoformat()
        
        session_key = f"session:{session_id}"
        self.r.setex(session_key, self.ttl, json.dumps(session))
        return True
    
    def destroy(self, session_id):
        """Destroy session"""
        session_key = f"session:{session_id}"
        self.r.delete(session_key)
    
    def get_user_sessions(self, user_id):
        """Get all sessions for user (requires secondary index)"""
        # Используй SCAN для поиска всех session:* ключей
        sessions = []
        for key in self.r.scan_iter(match="session:*"):
            data = self.r.get(key)
            if data:
                session = json.loads(data)
                if session["user_id"] == user_id:
                    sessions.append(session)
        return sessions

# Использование
r = redis.Redis()
store = SessionStore(r, ttl=1800)  # 30 минут

# Create session
sid = store.create("user:123", {"role": "admin"})
print(f"Session created: {sid}")

# Get session (auto-refreshes TTL)
session = store.get(sid)
print(f"Session data: {session}")

# Update
store.update(sid, {"last_page": "/dashboard"})

# Destroy
store.destroy(sid)
```

**4. Real-time Leaderboard с временными окнами:**

python

```python
import redis
import time
from datetime import datetime, timedelta

class Leaderboard:
    def __init__(self, redis_client, name):
        self.r = redis_client
        self.name = name
    
    def add_score(self, player_id, score, timestamp=None):
        """Add score to all leaderboards"""
        if timestamp is None:
            timestamp = time.time()
        
        # All-time leaderboard
        self.r.zadd(f"lb:{self.name}:all", {player_id: score})
        
        # Daily leaderboard
        day = datetime.fromtimestamp(timestamp).strftime("%Y-%m-%d")
        self.r.zadd(f"lb:{self.name}:daily:{day}", {player_id: score})
        self.r.expire(f"lb:{self.name}:daily:{day}", 86400 * 7)  # Keep 7 days
        
        # Weekly leaderboard
        week = datetime.fromtimestamp(timestamp).strftime("%Y-W%U")
        self.r.zadd(f"lb:{self.name}:weekly:{week}", {player_id: score})
        self.r.expire(f"lb:{self.name}:weekly:{week}", 86400 * 30)  # Keep 30 days
    
    def get_top(self, period="all", count=10, date=None):
        """Get top players"""
        if period == "all":
            key = f"lb:{self.name}:all"
        elif period == "daily":
            day = date or datetime.now().strftime("%Y-%m-%d")
            key = f"lb:{self.name}:daily:{day}"
        elif period == "weekly":
            week = date or datetime.now().strftime("%Y-W%U")
            key = f"lb:{self.name}:weekly:{week}"
        else:
            raise ValueError("Invalid period")
        
        return self.r.zrevrange(key, 0, count-1, withscores=True)
    
    def get_rank(self, player_id, period="all", date=None):
        """Get player rank"""
        if period == "all":
            key = f"lb:{self.name}:all"
        elif period == "daily":
            day = date or datetime.now().strftime("%Y-%m-%d")
            key = f"lb:{self.name}:daily:{day}"
        elif period == "weekly":
            week = date or datetime.now().strftime("%Y-W%U")
            key = f"lb:{self.name}:weekly:{week}"
        
        rank = self.r.zrevrank(key, player_id)
        score = self.r.zscore(key, player_id)
        return {
        "rank": rank + 1 if rank is not None else None,
        "score": score
    }

	def get_around(self, player_id, period="all", range_size=5):
	    """Get players around specific player"""
	    if period == "all":
	        key = f"lb:{self.name}:all"
	    elif period == "daily":
	        day = datetime.now().strftime("%Y-%m-%d")
	        key = f"lb:{self.name}:daily:{day}"
	    elif period == "weekly":
	        week = datetime.now().strftime("%Y-W%U")
	        key = f"lb:{self.name}:weekly:{week}"
	    
	    rank = self.r.zrevrank(key, player_id)
	    if rank is None:
	        return []
	    
	    start = max(0, rank - range_size)
	    end = rank + range_size
	    
	    return self.r.zrevrange(key, start, end, withscores=True)
	    
	    # Использование
		r = redis.Redis() lb = Leaderboard(r, "global")
		
		# Add scores		
		lb.add_score("player:1", 1000) lb.add_score("player:2", 1500)
		lb.add_score("player:3", 800) lb.add_score("player:4", 2000)
		lb.add_score("player:5", 1200)
		
		# Get top 3		
		print("Top 3 all-time:") for rank, (player, score) in enumerate(lb.get_top("all", 3), 1): print(f"{rank}. {player}: {score}")
		
		# Get player rank		
		print("\nPlayer 2 stats:") print(lb.get_rank("player:2", "all"))
		
		# Get players around player 3		
		print("\nAround player 3:") print(lb.get_around("player:3", "all", range_size=2))
	}

````

### 🚀 Бонус (новое)

**1. Redis как Message Broker (альтернатива RabbitMQ/Kafka для простых случаев):**
```python
import redis
import json
import threading
import time

class RedisMessageBroker:
    def __init__(self, redis_client):
        self.r = redis_client
        self.pubsub = self.r.pubsub()
        self.running = False
    
    def publish(self, topic, message):
        """Publish message to topic"""
        self.r.publish(topic, json.dumps(message))
    
    def subscribe(self, topics, callback):
        """Subscribe to topics with callback"""
        self.pubsub.subscribe(**{topic: callback for topic in topics})
        self.running = True
        
        thread = threading.Thread(target=self._listen)
        thread.daemon = True
        thread.start()
    
    def _listen(self):
        """Listen for messages"""
        while self.running:
            message = self.pubsub.get_message()
            if message and message['type'] == 'message':
                # Callback вызывается автоматически через subscribe
                pass
            time.sleep(0.001)
    
    def stop(self):
        """Stop listening"""
        self.running = False
        self.pubsub.close()

# Использование
r = redis.Redis()
broker = RedisMessageBroker(r)

# Subscriber
def handle_order(message):
    data = json.loads(message['data'])
    print(f"New order: {data}")

def handle_notification(message):
    data = json.loads(message['data'])
    print(f"Notification: {data}")

broker.subscribe(['orders', 'notifications'], lambda msg: handle_order(msg) if msg['channel'] == b'orders' else handle_notification(msg))

# Publisher
broker.publish('orders', {'order_id': 123, 'total': 99.99})
broker.publish('notifications', {'user_id': 456, 'message': 'Order shipped'})

time.sleep(1)
broker.stop()
```

**2. Redis Streams для Event Sourcing:**
```python
import redis
import json
import uuid

class EventStore:
    def __init__(self, redis_client):
        self.r = redis_client
    
    def append_event(self, aggregate_id, event_type, data):
        """Append event to aggregate stream"""
        stream_key = f"events:{aggregate_id}"
        
        event = {
            'event_id': str(uuid.uuid4()),
            'event_type': event_type,
            'timestamp': int(time.time() * 1000),
            'data': json.dumps(data)
        }
        
        event_id = self.r.xadd(stream_key, event)
        return event_id
    
    def get_events(self, aggregate_id, start='-', end='+'):
        """Get all events for aggregate"""
        stream_key = f"events:{aggregate_id}"
        events = self.r.xrange(stream_key, start, end)
        
        return [
            {
                'event_id': event_id.decode(),
                'event_type': data[b'event_type'].decode(),
                'timestamp': int(data[b'timestamp']),
                'data': json.loads(data[b'data'])
            }
            for event_id, data in events
        ]
    
    def rebuild_aggregate(self, aggregate_id, aggregate_class):
        """Rebuild aggregate from events"""
        events = self.get_events(aggregate_id)
        aggregate = aggregate_class(aggregate_id)
        
        for event in events:
            aggregate.apply_event(event)
        
        return aggregate

# Example: Order aggregate
class Order:
    def __init__(self, order_id):
        self.order_id = order_id
        self.items = []
        self.status = 'draft'
        self.total = 0
    
    def apply_event(self, event):
        if event['event_type'] == 'OrderCreated':
            self.status = 'created'
        elif event['event_type'] == 'ItemAdded':
            self.items.append(event['data'])
            self.total += event['data']['price']
        elif event['event_type'] == 'OrderPlaced':
            self.status = 'placed'
        elif event['event_type'] == 'OrderShipped':
            self.status = 'shipped'

# Использование
r = redis.Redis()
store = EventStore(r)

order_id = "order:123"

# Append events
store.append_event(order_id, 'OrderCreated', {})
store.append_event(order_id, 'ItemAdded', {'sku': 'ABC', 'price': 29.99})
store.append_event(order_id, 'ItemAdded', {'sku': 'DEF', 'price': 49.99})
store.append_event(order_id, 'OrderPlaced', {})

# Rebuild aggregate
order = store.rebuild_aggregate(order_id, Order)
print(f"Order {order.order_id}: {order.status}, Total: ${order.total}")
```

**3. Geo-Distributed Caching с CRDTs:**
```python
# Conflict-free Replicated Data Types для multi-datacenter
# Используй Redis + последние timestamps для разрешения конфликтов

class LWWRegister:  # Last-Write-Wins Register
    def __init__(self, redis_client, key):
        self.r = redis_client
        self.key = key
    
    def set(self, value, timestamp=None):
        if timestamp is None:
            timestamp = time.time()
        
        # Store value with timestamp
        self.r.hset(self.key, mapping={
            'value': value,
            'timestamp': timestamp
        })
    
    def get(self):
        data = self.r.hgetall(self.key)
        if not data:
            return None
        return data[b'value'].decode()
    
    def merge(self, other_value, other_timestamp):
        """Merge with value from another datacenter"""
        current = self.r.hgetall(self.key)
        
        if not current:
            self.set(other_value, other_timestamp)
            return
        
        current_timestamp = float(current[b'timestamp'])
        
        # Last-write-wins: keep newer value
        if other_timestamp > current_timestamp:
            self.set(other_value, other_timestamp)
```

---

## Дополнительные материалы и ресурсы

### Книги:
1. **"Redis in Action"** - Josiah Carlson (классика)
2. **"Redis Essentials"** - Maxwell Dayvson Da Silva
3. **"Mastering Redis"** - Jeremy Nelson
4. **"Redis 4.x Cookbook"** - Pengcheng Huang

### Online курсы:
1. **Redis University** (бесплатно) - university.redis.com
   - RU101: Introduction to Redis Data Structures
   - RU102: Redis Streams
   - RU202: Redis Streams (Advanced)
   - RU301: Running Redis at Scale

2. **Pluralsight** - Redis courses

3. **YouTube каналы:**
   - Redis Official Channel
   - TechWorld with Nana (Redis tutorial)

### Официальная документация:
- **redis.io/documentation** - главная документация
- **redis.io/commands** - reference по всем командам
- **redis.io/topics** - углубленные темы
- **github.com/redis/redis** - исходный код

### Инструменты:
1. **RedisInsight** - GUI клиент от Redis Labs
2. **redis-cli** - стандартный CLI
3. **redis-benchmark** - benchmarking tool
4. **redis-check-aof / redis-check-rdb** - проверка файлов
5. **Another Redis Desktop Manager** - open-source GUI
6. **Medis** - macOS GUI client

### Community:
- **Redis Discord** - discord.gg/redis
- **Reddit** - r/redis
- **Stack Overflow** - тэг [redis]
- **Redis Labs Blog** - redis.com/blog

### Мониторинг инструменты:
1. **Prometheus + Grafana**
2. **Datadog Redis integration**
3. **New Relic Redis monitoring**
4. **Redis Enterprise (платный)**
5. **RedisInsight** (бесплатный)

### Best practices чеклисты:

**Development:**
- ✅ Используй connection pooling
- ✅ Set reasonable timeouts
- ✅ Handle connection errors gracefully
- ✅ Use pipelines for batch operations
- ✅ Avoid KEYS in production (use SCAN)
- ✅ Set TTL on cache keys
- ✅ Use appropriate data structures

**Operations:**
- ✅ Enable persistence (если нужен)
- ✅ Configure maxmemory и eviction policy
- ✅ Set up replication для HA
- ✅ Use Sentinel или Cluster для автоматического failover
- ✅ Regular backups
- ✅ Monitor key metrics
- ✅ Set up alerts
- ✅ Document recovery procedures
- ✅ Regular security audits
- ✅ Keep Redis updated

**Security:**
- ✅ Always use password (requirepass)
- ✅ Use ACL для fine-grained permissions
- ✅ Bind to specific interfaces
- ✅ Enable protected mode
- ✅ Use TLS для encryption in-transit
- ✅ Rename dangerous commands
- ✅ Firewall rules
- ✅ Regular security updates
- ✅ Audit logs monitoring
- ✅ Least privilege principle

---

## Финальный чек-лист знаний

После полного прохождения курса ты должен уметь:

### Beginner (Базовый уровень):
- ✅ Понимать что такое Redis и когда его использовать
- ✅ Устанавливать Redis (Docker, package manager)
- ✅ Работать с redis-cli уверенно
- ✅ Использовать все базовые типы данных (String, List, Set, Hash, Sorted Set)
- ✅ Работать с TTL и expiration
- ✅ Понимать разницу между RDB и AOF
- ✅ Делать backup и restore

### Intermediate (Средний уровень):
- ✅ Настраивать Master-Replica репликацию
- ✅ Понимать и использовать разные eviction policies
- ✅ Работать с transactions (MULTI/EXEC)
- ✅ Использовать Pub/Sub для messaging
- ✅ Писать простые Lua скрипты
- ✅ Настраивать security (passwords, ACL)
- ✅ Использовать pipelining для оптимизации
- ✅ Мониторить базовые метрики

### Advanced (Продвинутый уровень):
- ✅ Настраивать Redis Sentinel для HA
- ✅ Работать с Redis Cluster
- ✅ Использовать Redis Streams для event-driven архитектур
- ✅ Писать сложные Lua скрипты
- ✅ Реализовывать различные patterns (rate limiting, distributed locks, leaderboards)
- ✅ Оптимизировать memory usage
- ✅ Troubleshooting production issues
- ✅ Настраивать полный monitoring stack (Prometheus/Grafana)

### Expert (Экспертный уровень):
- ✅ Проектировать multi-region архитектуры
- ✅ Работать с Redis модулями (JSON, Search, TimeSeries, Bloom)
- ✅ Использовать Redis в Kubernetes
- ✅ Настраивать Redis в облаках (AWS, Azure, GCP)
- ✅ Понимать внутреннее устройство Redis
- ✅ Оптимизировать для specific workloads
- ✅ Capacity planning
- ✅ Disaster recovery planning и execution
- ✅ Вносить вклад в Redis community

---

## Практические сценарии (Real-world use cases)

### 1. E-commerce Platform

Use cases:

- Session storage (1M+ concurrent users)
- Shopping cart (temporary data, TTL)
- Product catalog cache
- Real-time inventory
- Flash sale countdown (atomic decrements)
- User wish lists (Sets)
- Recently viewed products (Lists with LTRIM)
- Recommendation scores (Sorted Sets)
- Rate limiting API calls
- Distributed locks для order processing

Architecture:

- Redis Cluster (6 nodes: 3 masters + 3 replicas)
- Sentinel для HA
- Read replicas для read-heavy operations
- Separate Redis для sessions vs cache

```

### 2. Social Media App
```

Use cases:

- News feed (Lists + Sorted Sets)
- Followers/Following (Sets с SINTER для mutual friends)
- Trending topics (Sorted Sets с time-decay scoring)
- Real-time notifications (Pub/Sub)
- User activity streams (Redis Streams)
- Like counters (INCR)
- Comments cache (Hashes)
- Online users presence (Sets с TTL)
- Direct messages queue (Lists с BRPOP)

Architecture:

- Multi-region setup для global users
- Redis Streams для event sourcing
- Separate Redis instances по feature
- Heavy use of Lua scripts для atomic operations

```

### 3. IoT Platform
```

Use cases:

- Device metrics (TimeSeries)
- Device state (Hashes)
- Alerts queue (Lists)
- Anomaly detection (Bloom Filters)
- Time-series aggregations
- Device groups (Sets)
- Command queue для devices (Lists)
- Real-time dashboards (Pub/Sub)

Architecture:

- RedisTimeSeries module
- High write throughput configuration
- Retention policies для old data
- Aggregation rules

```

### 4. Gaming Platform
```

Use cases:

- Leaderboards (Sorted Sets)
- Player sessions
- Real-time multiplayer state
- Matchmaking queue
- Player inventory (Hashes)
- Achievement tracking
- Chat rooms (Pub/Sub)
- Game events (Streams)
- Anti-cheat rate limiting

Architecture:

- Low latency setup
- Geo-distributed для global players
- Heavy use of Sorted Sets
- Lua scripts для atomic game logic


---

## Заключение

Поздравляю с завершением полного курса Redis!

**Теперь ты знаешь:**
- ✅ Все основные концепции Redis от А до Я
- ✅ Production-ready конфигурации
- ✅ High availability архитектуры
- ✅ Performance оптимизация
- ✅ Security best practices
- ✅ Monitoring и troubleshooting
- ✅ Облачные решения
- ✅ Продвинутые техники и patterns
- ✅ Real-world use cases

**Рекомендуемый план дальнейшего развития:**

**Первые 3 месяца:**
1. Используй Redis в своих проектах
2. Поэкспериментируй с разными data structures
3. Реализуй 2-3 patterns из курса
4. Настрой базовый мониторинг

**3-6 месяцев:**
1. Изучи один Redis module детально (RedisJSON или RedisSearch)
2. Настрой production HA setup
3. Проведи performance тестирование
4. Напиши несколько Lua скриптов

**6-12 месяцев:**
1. Пройди Redis University курсы
2. Настрой multi-region setup
3. Вноси вклад в open-source проекты
4. Поделись знаниями через blog или meetup
5. Рассмотри сертификацию (если есть от Redis)

**Повторяй этот курс каждые 6-12 месяцев:**
- Redis активно развивается
- Новые версии добавляют features
- Твой опыт растет - будешь замечать новые детали
- Best practices эволюционируют

**Полезные привычки:**
- 📖 Читай Redis release notes при выходе новых версий
- 👥 Участвуй в Redis community (Discord, Reddit)
- 🔧 Регулярно мониторь свои production instances
- 📝 Документируй свои кейсы и решения
- 🎓 Делись знаниями с командой

**Помни главное:**
- Redis - это инструмент, а не решение всех проблем
- Правильное использование важнее скорости
- Мониторинг и наблюдаемость критичны
- Security должна быть на первом месте
- Документация - твой лучший друг

**Дополнительные challenge'ы для практики:**
1. Настрой Redis с нуля в production
2. Проведи миграцию данных между Redis кластерами
3. Оптимизируй memory usage на 50%
4. Реализуй custom Redis module
5. Напиши библиотеку-обертку для своего языка
6. Создай Redis operator для Kubernetes
7. Сделай fork Redis и добавь свою фичу
8. Напиши performance benchmarking suite

**Куда идти дальше:**
- **Для backend разработчиков:** Интеграция Redis с фреймворками, ORM patterns
- **Для DevOps/SRE:** Kubernetes operators, Infrastructure as Code, автоматизация
- **Для архитекторов:** Multi-region patterns, capacity planning, cost optimization
- **Для Data Engineers:** Redis Streams, time-series, analytics workloads

**Последний совет:**
Не пытайся запомнить всё - используй этот курс как справочник. Важно понимать концепции и знать где искать детали.

---

**Happy Redis-ing! 🚀⚡️**

Спасибо за прохождение курса! Если есть вопросы или нужна помощь - Redis community всегда готово помочь.

_Курс обновлён: Декабрь 2024_
_Версия Redis: 7.2_
_Поддерживаемые модули: RedisJSON, RedisSearch, RedisTimeSeries, RedisBloom_

---

## Appendix: Быстрые команды для ежедневной работы
```bash
# Health check one-liner
redis-cli PING && redis-cli INFO server | grep uptime

# Quick memory check
redis-cli INFO memory | grep -E "used_memory_human|mem_fragmentation_ratio"

# Find big keys
redis-cli --bigkeys --i 0.01

# Monitor real-time
redis-cli MONITOR | head -100

# Check replication
redis-cli INFO replication | grep -E "role|master_link_status|connected_slaves"

# Slow queries
redis-cli SLOWLOG GET 10

# Connected clients
redis-cli CLIENT LIST | wc -l

# Keys count
redis-cli DBSIZE

# Latency check
redis-cli --latency

# Export keys by pattern
redis-cli --scan --pattern 'user:*' | xargs -L 1 redis-cli DUMP

# Atomic increment with limit check (Lua)
redis-cli EVAL "local v = redis.call('GET',KEYS[1]) if not v or tonumber(v) < tonumber(ARGV[1]) then return redis.call('INCR',KEYS[1]) else return 0 end" 1 counter 100
```

**Всё! Курс завершён на 100%. Удачи в изучении и использовании Redis! **

