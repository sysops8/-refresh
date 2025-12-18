# Kafka Refresh: Ежегодный/Полугодовой курс для DevOps

## Цель
Освежить в памяти ключевые концепции Apache Kafka за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

## Формат
Каждый раздел состоит из:
- **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
- **Практического задания**: Реальная задача, которую нужно решить
- **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

## Предварительные требования
- Docker и Docker Compose установлены
- Базовые знания Linux/CLI
- Python 3.8+ (для примеров кода)

---

## Модуль 1: Базовая архитектура и установка (20 минут)

### 🎯 Напоминалка

**Архитектура Kafka:**

```
Kafka Cluster
├── Broker 1, 2, 3         # Серверы Kafka
├── Topics                 # Логические каналы
│   └── Partitions        # Физические файлы
├── ZooKeeper/KRaft       # Координация
└── Clients               # Producers/Consumers
```

**Основные компоненты:**
- **Broker**: Сервер Kafka, хранит данные
- **Topic**: Категория сообщений
- **Partition**: Упорядоченный неизменяемый лог
- **Producer**: Отправляет сообщения
- **Consumer**: Читает сообщения
- **Consumer Group**: Группа с балансировкой
- **Offset**: Позиция в партиции

**Docker Compose:**

```yaml
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    ports: ["2181:2181"]
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    ports: ["9092:9092"]
    depends_on: [zookeeper]
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    ports: ["8080:8080"]
    depends_on: [kafka]
    environment:
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:29092
```

**Базовые команды:**

```bash
# Topics
kafka-topics --bootstrap-server localhost:9092 --list
kafka-topics --bootstrap-server localhost:9092 --create --topic test --partitions 3
kafka-topics --bootstrap-server localhost:9092 --describe --topic test

# Producer/Consumer
kafka-console-producer --bootstrap-server localhost:9092 --topic test
kafka-console-consumer --bootstrap-server localhost:9092 --topic test --from-beginning

# Consumer Groups
kafka-consumer-groups --bootstrap-server localhost:9092 --list
kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group my-group
```

### 💻 Задание

1. Запусти Kafka через Docker Compose
2. Создай топик с 3 партициями
3. Протестируй producer и consumer
4. Открой Kafka UI (http://localhost:8080)

### 🚀 Бонус

Настрой KRaft mode (Kafka без ZooKeeper) и протестируй kcat CLI tool.

---

## Модуль 2: Topics, Partitions и Replication (25 минут)

### 🎯 Напоминалка

**Topic Configuration:**

```bash
kafka-topics --bootstrap-server localhost:9092 \
  --create --topic orders \
  --partitions 6 --replication-factor 3 \
  --config retention.ms=604800000 \
  --config min.insync.replicas=2
```

**Ключевые параметры:**
- `retention.ms` - время хранения (7 дней по умолчанию)
- `segment.ms` - размер лог-сегмента
- `compression.type` - lz4, snappy, gzip, zstd
- `min.insync.replicas` - минимум синхронных реплик
- `cleanup.policy` - delete или compact

**Partitioning:**
- С ключом: `partition = hash(key) % partitions`
- Без ключа: sticky partitioning (Kafka 2.4+)
- Гарантия порядка только в партиции

**Replication:**
```
Partition 0: Leader (Broker 1), Followers (Broker 2, 3)
ISR (In-Sync Replicas) - синхронизированные
OSR (Out-of-Sync) - отстающие
```

### 💻 Задание

1. Создай multi-broker кластер (3 брокера)
2. Создай топик с RF=3, 6 партиций
3. Протестируй партиционирование с ключами
4. Останови broker и наблюдай failover
5. Измени retention через kafka-configs

### 🚀 Бонус

Настрой Log Compaction и Partition Reassignment.

---

## Модуль 3: Producers и производительность (30 минут)

### 🎯 Напоминалка

**Producer Configuration:**

```python
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    acks='all',                    # 0, 1, all
    retries=3,
    max_in_flight_requests_per_connection=5,
    enable_idempotence=True,       # exactly-once
    compression_type='lz4',
    batch_size=32768,              # 32KB
    linger_ms=10                   # задержка для batching
)
```

**Acks levels:**
- `acks=0`: Fire-and-forget (потери возможны)
- `acks=1`: Leader подтверждает (баланс)
- `acks=all`: Все ISR подтверждают (надёжно)

**Производительность:**
- Batching: `batch_size`, `linger_ms`
- Compression: lz4 (лучший баланс)
- Idempotence: гарантирует отсутствие дубликатов
- Transactions: exactly-once семантика

### 💻 Задание

1. Создай Python producer с разными конфигурациями
2. Бенчмарк: fast vs balanced vs reliable
3. Тест batching с разными размерами
4. Тест compression (none, lz4, gzip, zstd)
4. Измерь throughput и latency

### 🚀 Бонус

Настрой Schema Registry с Avro, Custom Partitioner, Transactional Producer.

---

## Модуль 4: Consumers и Consumer Groups (30 минут)

### 🎯 Напоминалка

**Consumer Configuration:**

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'my-topic',
    bootstrap_servers=['localhost:9092'],
    group_id='my-group',
    auto_offset_reset='earliest',     # earliest, latest
    enable_auto_commit=False,         # manual commit
    max_poll_records=500
)

for message in consumer:
    process(message)
    consumer.commit()  # manual commit
```

**Consumer Groups:**
- Партиции распределяются между consumers
- Rebalance при изменении группы
- Offset хранится в `__consumer_offsets`

**Offset Management:**
- Auto-commit (простота, риск дубликатов)
- Manual commitSync (блокирующий)
- Manual commitAsync (производительность)
- Commit specific offsets (контроль)

**Rebalance Strategies:**
- Range (по умолчанию до 3.0)
- RoundRobin (равномерно)
- Sticky (минимум перемещений)
- Cooperative Sticky (incremental, default 3.0+)

### 💻 Задание

1. Создай базовый consumer
2. Протестируй consumer group (3 инстанса)
3. Manual offset management
4. Rebalance listener
5. Проверь consumer lag через kafka-consumer-groups

### 🚀 Бонус

Exactly-Once Semantics, Parallel processing с pause/resume, Consumer performance monitoring.

---

## Модуль 5: Kafka Connect (25 минут)

### 🎯 Напоминалка

**Kafka Connect:**
- Source Connectors: БД, файлы, API → Kafka
- Sink Connectors: Kafka → БД, S3, Elasticsearch
- Distributed mode для production
- REST API для управления

**Docker Compose:**

```yaml
kafka-connect:
  image: confluentinc/cp-kafka-connect:7.5.0
  ports: ["8083:8083"]
  environment:
    CONNECT_BOOTSTRAP_SERVERS: 'kafka:29092'
    CONNECT_GROUP_ID: connect-cluster
    CONNECT_CONFIG_STORAGE_TOPIC: connect-configs
    CONNECT_OFFSET_STORAGE_TOPIC: connect-offsets
    CONNECT_STATUS_STORAGE_TOPIC: connect-status
```

**REST API:**

```bash
# Список connectors
curl http://localhost:8083/connectors

# Создание
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d @connector.json

# Статус
curl http://localhost:8083/connectors/my-connector/status
```

### 💻 Задание

1. Запусти Kafka Connect
2. Настрой PostgreSQL в Docker
3. JDBC Source Connector для синхронизации таблицы
4. Проверь данные в Kafka топике
5. Добавь/измени данные в БД и наблюдай CDC

### 🚀 Бонус

Debezium CDC, S3 Sink Connector, Custom SMT (Single Message Transform).

---

## Модуль 6: Kafka Streams (30 минут)

### 🎯 Напоминалка

**Kafka Streams:**
- Библиотека для stream processing (не отдельный кластер)
- KStream (поток событий) vs KTable (changelog)
- Stateless: map, filter, flatMap
- Stateful: groupBy, count, aggregate, join
- Windowing: tumbling, hopping, session

**Python с Faust:**

```python
import faust

app = faust.App('myapp', broker='kafka://localhost:9092')

class Order(faust.Record):
    user_id: str
    amount: float

orders = app.topic('orders', value_type=Order)
totals = app.Table('user_totals', default=float)

@app.agent(orders)
async def process_orders(stream):
    async for order in stream:
        totals[order.user_id] += order.amount
```

### 💻 Задание

1. Установи faust-streaming
2. Создай word count приложение
3. Windowed aggregation (tumbling 30s)
4. Stream joining (orders + users)
5. Запусти несколько workers

### 🚀 Бонус

Interactive Queries REST API, Complex Event Processing (fraud detection).

---

## Модуль 7: Мониторинг (25 минут)

### 🎯 Напоминалка

**Ключевые метрики:**

Broker:
- UnderReplicatedPartitions (должно быть 0)
- OfflinePartitionsCount (должно быть 0)
- BytesInPerSec / BytesOutPerSec
- RequestLatency

Producer:
- record-send-rate
- record-error-rate
- batch-size-avg

Consumer:
- records-lag-max (критично!)
- fetch-latency-avg
- commit-latency-avg

**Мониторинг:**

```yaml
prometheus:
  image: prom/prometheus:latest
  ports: ["9090:9090"]

grafana:
  image: grafana/grafana:latest
  ports: ["3000:3000"]

jmx-exporter:
  image: bitnami/jmx-exporter:latest
  ports: ["5556:5556"]
```

### 💻 Задание

1. Настрой JMX экспорт
2. Скрипт для мониторинга consumer lag
3. Alert при high lag
4. Dashboard с метриками (Flask API)

### 🚀 Бонус

Cruise Control, Custom metrics с StatsD, Kafka Manager (CMAK).

---

## Модуль 8: Безопасность (25 минут)

### 🎯 Напоминалка

**Уровни безопасности:**
1. Encryption (SSL/TLS)
2. Authentication (SASL, SSL)
3. Authorization (ACLs)
4. Quotas

**SSL Configuration:**

```properties
# Broker
listeners=SSL://0.0.0.0:9093
ssl.keystore.location=/ssl/kafka.keystore.jks
ssl.truststore.location=/ssl/kafka.truststore.jks
```

**ACLs:**

```bash
kafka-acls --bootstrap-server localhost:9092 \
  --add --allow-principal User:alice \
  --operation Read --operation Write \
  --topic orders

kafka-acls --bootstrap-server localhost:9092 \
  --add --allow-principal User:alice \
  --operation Read --group my-group
```

**Quotas:**

```bash
kafka-configs --bootstrap-server localhost:9092 \
  --alter --add-config 'producer_byte_rate=1048576' \
  --entity-type users --entity-name alice
```

### 💻 Задание

1. Создай SSL сертификаты
2. Настрой Kafka с SSL
3. Создай ACLs для разных пользователей
4. Настрой quotas
5. Протестируй SSL подключение

### 🚀 Бонус

OAuth 2.0, Encryption at rest, Audit logging.

---

## Модуль 9: Production Best Practices (30 минут)

### 🎯 Напоминалка

**Cluster Sizing:**

```
Партиций per broker: < 4000
Размер партиции: < 25GB
Replication factor: 3
min.insync.replicas: 2
```

**Broker Configuration:**

```properties
num.network.threads=8
num.io.threads=16
log.segment.bytes=1073741824
log.retention.hours=168
compression.type=producer
min.insync.replicas=2
```

**Topic Best Practices:**
- Начни с 3-6 партиций
- RF=3 для production
- min.insync.replicas=2
- compression.type=lz4
- Мониторь consumer lag

**Disaster Recovery:**
- MirrorMaker 2 для репликации
- Backup метаданных
- Runbook для failover
- Regular testing

### 💻 Задание

Финальный проект: Production-ready Kafka для e-commerce

**Требования:**
- 3 brokers с SSL
- Topics: orders, inventory, payments
- Producers с idempotence
- Consumer groups
- Kafka Connect (JDBC, S3)
- Monitoring (Prometheus + Grafana)
- ACLs и quotas
- Documentation

### 🚀 Бонус

Multi-datacenter репликация, Cruise Control auto-balancing, Capacity planning.

---

## Модуль 10: Troubleshooting (20 минут)

### 🎯 Напоминалка

**Частые проблемы:**

**High Consumer Lag:**
```bash
kafka-consumer-groups --bootstrap-server localhost:9092 \
  --describe --group my-group

Решения:
- Больше consumers
- Увеличить max.poll.records
- Оптимизировать processing
- Добавить партиций
```

**Under-replicated partitions:**
```bash
kafka-topics --bootstrap-server localhost:9092 \
  --describe --under-replicated-partitions

Причины:
- Broker перегружен
- Сетевые проблемы
- Disk I/O issues
```

**High Latency:**
```
Метрики: request-latency-avg, fetch-latency-avg

Решения:
- Увеличить batch.size
- Настроить linger.ms
- Проверить compression
- Hardware upgrade
```

**Out of Memory:**
```bash
export KAFKA_HEAP_OPTS="-Xmx4G -Xms4G"
export KAFKA_JVM_PERFORMANCE_OPTS="-XX:+UseG1GC"
```

### 💻 Задание

1. Симулируй high lag и исправь
2. Тест failover при падении broker
3. Оптимизация медленного consumer
4. Log analysis для диагностики
5. Создай runbook

### 🚀 Бонус

Performance tuning, Capacity planning calculator, Automated recovery.

---

## Справочная секция

### Быстрые команды

```bash
# Алиасы
alias kt='kafka-topics --bootstrap-server localhost:9092'
alias kp='kafka-console-producer --bootstrap-server localhost:9092'
alias kc='kafka-console-consumer --bootstrap-server localhost:9092'
alias kg='kafka-consumer-groups --bootstrap-server localhost:9092'

# Типовые задачи
kt --list
kt --describe --topic my-topic
kg --describe --group my-group
kc --topic my-topic --from-beginning --max-messages 10
```

### Python Templates

**Producer:**
```python
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    acks='all',
    retries=3,
    enable_idempotence=True
)

producer.send('topic', b'message')
producer.flush()
```

**Consumer:**
```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'topic',
    bootstrap_servers=['localhost:9092'],
    group_id='group',
    enable_auto_commit=False
)

for msg in consumer:
    process(msg)
    consumer.commit()
```

### Полезные инструменты

- **kcat** - CLI для Kafka
- **Kafka UI** - Web интерфейс
- **CMAK** - Cluster management
- **Cruise Control** - Auto-balancing
- **Burrow** - Lag monitoring

### Production Checklist

✅ RF >= 3
✅ min.insync.replicas >= 2
✅ SSL/TLS настроен
✅ ACLs созданы
✅ Monitoring работает
✅ Alerting настроен
✅ Backup процедуры
✅ Runbook готов
✅ Capacity plan

### Метрики Мониторинга

**Critical:**
- UnderReplicatedPartitions = 0
- OfflinePartitionsCount = 0
- ActiveControllerCount = 1

**Performance:**
- BytesInPerSec / BytesOutPerSec
- RequestLatency < 100ms
- ConsumerLag < 1000

---

## План повторения

**Первое прохождение (2-3 часа):**
- Модули 1-4 обязательно
- Модули 5-6 рекомендуется
- Простой финальный проект

**Повтор через 6-12 месяцев:**
- Бегло просмотри теорию
- Бонусные задания
- Модули 7-10 обязательно
- Полный финальный проект

**Для закрепления:**
- Используй Kafka в проектах
- Настрой production кластер
- Сертификация Confluent
- Kafka Internals

---

## Дополнительные ресурсы

**Документация:**
- https://kafka.apache.org/documentation/
- https://docs.confluent.io/

**Книги:**
- "Kafka: The Definitive Guide"
- "Kafka Streams in Action"
- "Designing Event-Driven Systems"

**Курсы:**
- Confluent Developer Training
- Udemy Kafka courses

**Сообщество:**
- Kafka Users Slack
- Stack Overflow
- Confluent Forum

---

## Заключение

Поздравляю! Ты освежил знания по Apache Kafka.

**Следующие шаги:**
1. Применяй в production
2. Изучай advanced (Tiered Storage, KRaft)
3. Получи сертификацию
4. Делись знаниями

Повторяй курс каждые 6-12 месяцев!

Happy Kafka streaming! 🚀