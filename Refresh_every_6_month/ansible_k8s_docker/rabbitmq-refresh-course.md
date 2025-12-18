# RabbitMQ Refresh: Ежегодный/Полугодовой курс для DevOps

**Цель:** Освежить в памяти ключевые концепции RabbitMQ за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
- **Краткой теории (Напоминалка):** Самое главное, что вы могли забыть
- **Практического задания:** Реальная задача, которую нужно решить
- **Бонусного задания (для роста):** Задача посложнее или с использованием новой фичи

## Предварительные требования:

- Docker установлен
- Python 3.8+ или другой язык программирования
- Базовые знания messaging систем
- Доступ к терминалу

---

## Модуль 1: Основы RabbitMQ и быстрый старт (20 минут)

### 🎯 Напоминалка

**Архитектура RabbitMQ:**

```
Producer → Exchange → Queue → Consumer
           ↓
       Bindings (routing rules)
```

**Основные компоненты:**

```
┌─────────────┐
│  Producer   │ - Отправляет сообщения
└──────┬──────┘
       │
       ↓
┌─────────────┐
│  Exchange   │ - Маршрутизирует сообщения по правилам
└──────┬──────┘
       │ (binding + routing key)
       ↓
┌─────────────┐
│    Queue    │ - Хранит сообщения
└──────┬──────┘
       │
       ↓
┌─────────────┐
│  Consumer   │ - Получает и обрабатывает сообщения
└─────────────┘
```

**Типы Exchange:**

1. **Direct** - точное совпадение routing key
2. **Fanout** - broadcast всем привязанным queue
3. **Topic** - pattern matching с wildcards (* и #)
4. **Headers** - маршрутизация по headers (редко используется)

**Запуск RabbitMQ:**

```bash
# Docker
docker run -d --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin \
  rabbitmq:3-management

# Проверка
docker logs rabbitmq
docker exec rabbitmq rabbitmqctl status

# Web UI
# http://localhost:15672
# Login: admin / admin
```

**Основные порты:**

- `5672` - AMQP protocol
- `15672` - Management UI
- `25672` - Erlang distribution (clustering)
- `4369` - EPMD (Erlang Port Mapper Daemon)

**CLI команды (rabbitmqctl):**

```bash
# Статус
docker exec rabbitmq rabbitmqctl status
docker exec rabbitmq rabbitmqctl cluster_status

# Пользователи
docker exec rabbitmq rabbitmqctl list_users
docker exec rabbitmq rabbitmqctl add_user myuser mypass
docker exec rabbitmq rabbitmqctl set_user_tags myuser administrator
docker exec rabbitmq rabbitmqctl set_permissions -p / myuser ".*" ".*" ".*"

# Virtual Hosts
docker exec rabbitmq rabbitmqctl list_vhosts
docker exec rabbitmq rabbitmqctl add_vhost /dev
docker exec rabbitmq rabbitmqctl delete_vhost /dev

# Queues
docker exec rabbitmq rabbitmqctl list_queues name messages consumers
docker exec rabbitmq rabbitmqctl list_queues name messages_ready messages_unacknowledged

# Exchanges
docker exec rabbitmq rabbitmqctl list_exchanges name type

# Bindings
docker exec rabbitmq rabbitmqctl list_bindings

# Connections
docker exec rabbitmq rabbitmqctl list_connections name peer_host peer_port state
docker exec rabbitmq rabbitmqctl close_connection "<connection_name>" "Reason"

# Channels
docker exec rabbitmq rabbitmqctl list_channels connection name number
```

**Python клиент (pika):**

```bash
pip install pika
```

```python
import pika

# Подключение
connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

# Создание queue
channel.queue_declare(queue='hello')

# Отправка
channel.basic_publish(
    exchange='',
    routing_key='hello',
    body='Hello World!'
)

# Получение
def callback(ch, method, properties, body):
    print(f"Received {body}")
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_consume(
    queue='hello',
    on_message_callback=callback
)

channel.start_consuming()
```

### 💻 Задание

**Подготовь тестовое окружение:**

1. **Запусти RabbitMQ:**
```bash
docker run -d --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin \
  rabbitmq:3-management

# Дождись запуска
docker logs -f rabbitmq
```

2. **Проверь Web UI:**
   - Открой http://localhost:15672
   - Залогинься: admin / admin
   - Изучи вкладки: Overview, Connections, Channels, Exchanges, Queues

3. **Создай простого Producer:**
```python
# producer.py
import pika
import sys

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.queue_declare(queue='test_queue', durable=True)

message = ' '.join(sys.argv[1:]) or "Hello World!"
channel.basic_publish(
    exchange='',
    routing_key='test_queue',
    body=message,
    properties=pika.BasicProperties(
        delivery_mode=2,  # make message persistent
    ))

print(f" [x] Sent '{message}'")
connection.close()
```

4. **Создай простого Consumer:**
```python
# consumer.py
import pika
import time

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.queue_declare(queue='test_queue', durable=True)

def callback(ch, method, properties, body):
    print(f" [x] Received {body.decode()}")
    time.sleep(body.count(b'.'))
    print(" [x] Done")
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue='test_queue', on_message_callback=callback)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

5. **Протестируй:**
```bash
# Терминал 1
python consumer.py

# Терминал 2
python producer.py "First message."
python producer.py "Second message.."
python producer.py "Third message..."

# Проверь в Web UI количество сообщений
```

### 🚀 Бонус (новое)

**1. Используй rabbitmqadmin для CLI управления:**

```bash
# Скачай rabbitmqadmin
docker exec rabbitmq cat /usr/local/bin/rabbitmqadmin > rabbitmqadmin
chmod +x rabbitmqadmin

# Использование
./rabbitmqadmin -u admin -p admin list queues
./rabbitmqadmin -u admin -p admin list exchanges
./rabbitmqadmin -u admin -p admin declare queue name=my_queue durable=true
./rabbitmqadmin -u admin -p admin publish routing_key=my_queue payload="test message"
```

**2. Настрой мониторинг с Prometheus:**

```bash
# Включи prometheus plugin
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_prometheus

# Проверь метрики
curl http://localhost:15692/metrics
```

**3. Используй Management API:**

```bash
# Получить информацию о queues
curl -u admin:admin http://localhost:15672/api/queues

# Получить сообщения из queue (non-destructive)
curl -u admin:admin http://localhost:15672/api/queues/%2F/test_queue/get \
  -d '{"count":5,"ackmode":"ack_requeue_true","encoding":"auto"}' \
  -H "Content-Type: application/json"

# Создать queue
curl -u admin:admin -X PUT http://localhost:15672/api/queues/%2F/my_new_queue \
  -d '{"durable":true,"auto_delete":false}' \
  -H "Content-Type: application/json"
```

---

## Модуль 2: Exchanges и Routing (25 минут)

### 🎯 Напоминалка

**Direct Exchange:**

```python
# Producer
channel.exchange_declare(exchange='direct_logs', exchange_type='direct')

channel.basic_publish(
    exchange='direct_logs',
    routing_key='error',  # exact match
    body=message
)

# Consumer
channel.queue_bind(
    exchange='direct_logs',
    queue=queue_name,
    routing_key='error'
)
```

**Fanout Exchange (Broadcast):**

```python
# Producer
channel.exchange_declare(exchange='logs', exchange_type='fanout')

channel.basic_publish(
    exchange='logs',
    routing_key='',  # ignored for fanout
    body=message
)

# Consumer
channel.queue_bind(exchange='logs', queue=queue_name)
```

**Topic Exchange (Pattern Matching):**

```python
# Producer
channel.exchange_declare(exchange='topic_logs', exchange_type='topic')

channel.basic_publish(
    exchange='topic_logs',
    routing_key='kern.critical',  # pattern: <word>.<word>
    body=message
)

# Consumer
channel.queue_bind(
    exchange='topic_logs',
    queue=queue_name,
    routing_key='kern.*'      # * = один word
)

channel.queue_bind(
    exchange='topic_logs',
    queue=queue_name,
    routing_key='*.critical'  # # = ноль или больше words
)
```

**Topic Wildcards:**

- `*` (star) - заменяет ровно одно слово
- `#` (hash) - заменяет ноль или больше слов

**Примеры routing keys:**

```
"stock.usd.nyse"
"nyse.vmw"
"quick.orange.rabbit"
"lazy.orange.elephant"

Bindings:
"*.orange.*"     → quick.orange.rabbit, lazy.orange.elephant
"*.*.rabbit"     → quick.orange.rabbit
"lazy.#"         → lazy.orange.elephant, lazy.pink.rabbit
"#"              → все сообщения
```

**Headers Exchange:**

```python
# Producer
channel.exchange_declare(exchange='headers_logs', exchange_type='headers')

channel.basic_publish(
    exchange='headers_logs',
    routing_key='',
    body=message,
    properties=pika.BasicProperties(
        headers={'format': 'pdf', 'type': 'report'}
    )
)

# Consumer
channel.queue_bind(
    exchange='headers_logs',
    queue=queue_name,
    arguments={
        'x-match': 'all',  # или 'any'
        'format': 'pdf',
        'type': 'report'
    }
)
```

**Альтернативный Exchange (fallback):**

```python
channel.exchange_declare(
    exchange='main_exchange',
    exchange_type='direct',
    arguments={
        'alternate-exchange': 'unrouted_exchange'
    }
)

# Сообщения без matching binding попадут в alternate exchange
```

### 💻 Задание

**Создай систему логирования с разными уровнями:**

**1. Создай producer для логов:**

```python
# log_producer.py
import pika
import sys

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.exchange_declare(exchange='logs_direct', exchange_type='direct')

severity = sys.argv[1] if len(sys.argv) > 1 else 'info'
message = ' '.join(sys.argv[2:]) or 'Hello World!'

channel.basic_publish(
    exchange='logs_direct',
    routing_key=severity,
    body=message
)

print(f" [x] Sent {severity}: {message}")
connection.close()
```

**2. Создай consumer для ошибок:**

```python
# log_consumer_errors.py
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.exchange_declare(exchange='logs_direct', exchange_type='direct')

result = channel.queue_declare(queue='', exclusive=True)
queue_name = result.method.queue

severities = ['error', 'critical']

for severity in severities:
    channel.queue_bind(
        exchange='logs_direct',
        queue=queue_name,
        routing_key=severity
    )

print(' [*] Waiting for error logs. To exit press CTRL+C')

def callback(ch, method, properties, body):
    print(f" [x] {method.routing_key}: {body.decode()}")

channel.basic_consume(
    queue=queue_name,
    on_message_callback=callback,
    auto_ack=True
)

channel.start_consuming()
```

**3. Создай Topic exchange для сложной маршрутизации:**

```python
# log_producer_topic.py
import pika
import sys

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.exchange_declare(exchange='topic_logs', exchange_type='topic')

routing_key = sys.argv[1] if len(sys.argv) > 2 else 'anonymous.info'
message = ' '.join(sys.argv[2:]) or 'Hello World!'

channel.basic_publish(
    exchange='topic_logs',
    routing_key=routing_key,
    body=message
)

print(f" [x] Sent {routing_key}: {message}")
connection.close()
```

**4. Создай consumer с pattern matching:**

```python
# log_consumer_topic.py
import pika
import sys

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.exchange_declare(exchange='topic_logs', exchange_type='topic')

result = channel.queue_declare(queue='', exclusive=True)
queue_name = result.method.queue

binding_keys = sys.argv[1:] if len(sys.argv) > 1 else ['#']

for binding_key in binding_keys:
    channel.queue_bind(
        exchange='topic_logs',
        queue=queue_name,
        routing_key=binding_key
    )

print(f' [*] Waiting for logs matching: {binding_keys}')

def callback(ch, method, properties, body):
    print(f" [x] {method.routing_key}: {body.decode()}")

channel.basic_consume(
    queue=queue_name,
    on_message_callback=callback,
    auto_ack=True
)

channel.start_consuming()
```

**5. Протестируй:**

```bash
# Терминал 1: слушаем все kern.* логи
python log_consumer_topic.py "kern.*"

# Терминал 2: слушаем критичные логи
python log_consumer_topic.py "*.critical"

# Терминал 3: отправляем
python log_producer_topic.py "kern.critical" "A critical kernel error"
python log_producer_topic.py "kern.info" "Kernel information"
python log_producer_topic.py "app.critical" "Application crashed"

# Проверь, кто что получил
```

### 🚀 Бонус (новое)

**1. Consistent Hash Exchange (для распределения нагрузки):**

```bash
# Включи plugin
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_consistent_hash_exchange
```

```python
channel.exchange_declare(
    exchange='hash_exchange',
    exchange_type='x-consistent-hash'
)

# Биндинг с весами
channel.queue_bind(
    exchange='hash_exchange',
    queue='queue1',
    routing_key='1'  # вес
)

channel.queue_bind(
    exchange='hash_exchange',
    queue='queue2',
    routing_key='2'  # двойной вес
)
```

**2. Exchange to Exchange Binding:**

```python
# Создаём иерархию exchanges
channel.exchange_declare(exchange='main', exchange_type='topic')
channel.exchange_declare(exchange='errors', exchange_type='fanout')

# Биндим exchange к exchange
channel.exchange_bind(
    destination='errors',
    source='main',
    routing_key='*.error'
)
```

**3. Dead Letter Exchange (DLX):**

```python
# Queue с DLX
channel.queue_declare(
    queue='main_queue',
    arguments={
        'x-dead-letter-exchange': 'dlx_exchange',
        'x-dead-letter-routing-key': 'dead_letters'
    }
)

channel.exchange_declare(exchange='dlx_exchange', exchange_type='direct')
channel.queue_declare(queue='dead_letters')
channel.queue_bind(
    exchange='dlx_exchange',
    queue='dead_letters',
    routing_key='dead_letters'
)
```

---

## Модуль 3: Надежность и Durability (30 минут)

### 🎯 Напоминалка

**Три уровня надежности:**

1. **Message Persistence** - сообщение сохраняется на диск
2. **Queue Durability** - queue переживает рестарт
3. **Publisher Confirms** - подтверждение доставки

**Durable Queue:**

```python
channel.queue_declare(
    queue='durable_queue',
    durable=True  # переживет рестарт RabbitMQ
)
```

**Persistent Messages:**

```python
channel.basic_publish(
    exchange='',
    routing_key='durable_queue',
    body=message,
    properties=pika.BasicProperties(
        delivery_mode=2,  # persistent
    )
)
```

**Manual Acknowledgments:**

```python
def callback(ch, method, properties, body):
    print(f"Processing {body}")
    # ... обработка ...
    
    # Success
    ch.basic_ack(delivery_tag=method.delivery_tag)
    
    # Reject and requeue
    # ch.basic_nack(delivery_tag=method.delivery_tag, requeue=True)
    
    # Reject and don't requeue
    # ch.basic_reject(delivery_tag=method.delivery_tag, requeue=False)

channel.basic_consume(
    queue='tasks',
    on_message_callback=callback,
    auto_ack=False  # manual ack
)
```

**Prefetch (QoS):**

```python
# Consumer получит максимум N unacked сообщений
channel.basic_qos(prefetch_count=1)

# Или глобально на channel
channel.basic_qos(prefetch_count=10, global_qos=True)
```

**Publisher Confirms:**

```python
# Включаем confirms
channel.confirm_delivery()

try:
    channel.basic_publish(
        exchange='',
        routing_key='queue',
        body=message,
        mandatory=True  # вернет ошибку если нет route
    )
    print("Message confirmed")
except pika.exceptions.UnroutableError:
    print("Message was returned")
except pika.exceptions.NackError:
    print("Message was nacked")
```

**Transactions (медленнее, чем confirms):**

```python
channel.tx_select()  # начало транзакции

try:
    channel.basic_publish(...)
    channel.basic_publish(...)
    channel.tx_commit()  # коммит
except Exception:
    channel.tx_rollback()  # откат
```

**Message TTL:**

```python
# Per-message TTL
channel.basic_publish(
    exchange='',
    routing_key='queue',
    body=message,
    properties=pika.BasicProperties(
        expiration='60000'  # 60 секунд в миллисекундах
    )
)

# Per-queue TTL
channel.queue_declare(
    queue='ttl_queue',
    arguments={
        'x-message-ttl': 60000  # все сообщения
    }
)
```

**Queue TTL:**

```python
# Queue удалится если не используется
channel.queue_declare(
    queue='temp_queue',
    arguments={
        'x-expires': 300000  # 5 минут
    }
)
```

**Queue Length Limit:**

```python
channel.queue_declare(
    queue='limited_queue',
    arguments={
        'x-max-length': 1000,  # максимум сообщений
        'x-overflow': 'reject-publish'  # или 'drop-head'
    }
)
```

**Priority Queue:**

```python
channel.queue_declare(
    queue='priority_queue',
    arguments={
        'x-max-priority': 10  # 0-10
    }
)

channel.basic_publish(
    exchange='',
    routing_key='priority_queue',
    body=message,
    properties=pika.BasicProperties(
        priority=5
    )
)
```

### 💻 Задание

**Создай надежную систему обработки задач:**

**1. Producer с подтверждениями:**

```python
# reliable_producer.py
import pika
import sys
import time

def main():
    connection = pika.BlockingConnection(
        pika.ConnectionParameters(
            'localhost',
            heartbeat=600,
            blocked_connection_timeout=300
        )
    )
    channel = connection.channel()
    
    # Durable queue
    channel.queue_declare(queue='task_queue', durable=True)
    
    # Enable confirms
    channel.confirm_delivery()
    
    for i in range(10):
        message = f'Task {i}'
        
        try:
            channel.basic_publish(
                exchange='',
                routing_key='task_queue',
                body=message,
                properties=pika.BasicProperties(
                    delivery_mode=2,  # persistent
                    priority=i % 10,  # priority 0-9
                ),
                mandatory=True
            )
            print(f" [✓] Sent and confirmed: {message}")
        except pika.exceptions.UnroutableError:
            print(f" [✗] Message was returned: {message}")
        except pika.exceptions.NackError:
            print(f" [✗] Message was nacked: {message}")
        
        time.sleep(0.5)
    
    connection.close()

if __name__ == '__main__':
    main()
```

**2. Consumer с manual ack и retry logic:**

```python
# reliable_consumer.py
import pika
import time
import random

MAX_RETRIES = 3

def callback(ch, method, properties, body):
    message = body.decode()
    print(f" [x] Received {message}")
    
    # Получаем количество попыток из headers
    headers = properties.headers or {}
    retry_count = headers.get('x-retry-count', 0)
    
    # Симулируем случайную ошибку
    if random.random() < 0.3:  # 30% chance of failure
        print(f" [!] Failed to process {message}")
        
        if retry_count < MAX_RETRIES:
            # Requeue with retry count
            ch.basic_reject(delivery_tag=method.delivery_tag, requeue=True)
            
            # Или отправляем в retry queue с задержкой
            ch.basic_publish(
                exchange='',
                routing_key='task_queue',
                body=body,
                properties=pika.BasicProperties(
                    headers={'x-retry-count': retry_count + 1},
                    delivery_mode=2
                )
            )
            ch.basic_ack(delivery_tag=method.delivery_tag)
            print(f" [↻] Requeued with retry count: {retry_count + 1}")
        else:
            # Max retries exceeded, send to DLQ
            print(f" [✗] Max retries exceeded, sending to DLQ")
            ch.basic_reject(delivery_tag=method.delivery_tag, requeue=False)
    else:
        # Обработка успешна
        time.sleep(message.count('.'))
        print(f" [✓] Done processing {message}")
        ch.basic_ack(delivery_tag=method.delivery_tag)

def main():
    connection = pika.BlockingConnection(
        pika.ConnectionParameters('localhost')
    )
    channel = connection.channel()
    
    channel.queue_declare(queue='task_queue', durable=True)
    
    # QoS: обрабатываем по одному сообщению
    channel.basic_qos(prefetch_count=1)
    
    channel.basic_consume(
        queue='task_queue',
        on_message_callback=callback,
        auto_ack=False
    )
    
    print(' [*] Waiting for messages. To exit press CTRL+C')
    channel.start_consuming()

if __name__ == '__main__':
    try:
        main()
    except KeyboardInterrupt:
        print('Interrupted')
```

**3. Настрой Dead Letter Queue:**

```python
# setup_dlq.py
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

# DLX Exchange
channel.exchange_declare(
    exchange='dlx',
    exchange_type='fanout',
    durable=True
)

# Dead Letter Queue
channel.queue_declare(
    queue='dead_letters',
    durable=True
)

channel.queue_bind(
    exchange='dlx',
    queue='dead_letters'
)

# Main queue с DLX
channel.queue_declare(
    queue='task_queue_with_dlx',
    durable=True,
    arguments={
        'x-dead-letter-exchange': 'dlx',
        'x-message-ttl': 60000,  # 60 секунд
        'x-max-length': 100,
    }
)

print("DLQ setup completed")
connection.close()
```

**4. Протестируй надежность:**

```bash
# Запусти consumer
python reliable_consumer.py

# В другом терминале отправь задачи
python reliable_producer.py

# Останови consumer (Ctrl+C) во время обработки
# Перезапусти - сообщения не потеряются

# Проверь в Web UI:
# - Количество unacked сообщений
# - Ready сообщения
# - Dead letters queue
```

### 🚀 Бонус (новое)

**1. Quorum Queues (для высокой доступности):**

```python
channel.queue_declare(
    queue='quorum_queue',
    durable=True,
    arguments={
        'x-queue-type': 'quorum',  # вместо classic
        'x-max-in-memory-length': 0,  # все на диск
    }
)
```

**2. Stream Queues (для больших объемов, RabbitMQ 3.9+):**

```python
channel.queue_declare(
    queue='stream_queue',
    durable=True,
    arguments={
        'x-queue-type': 'stream',
        'x-max-age': '7D',  # retention period
    }
)
```

**3. Delayed Message Plugin:**

```bash
# Включи plugin
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

```python
channel.exchange_declare(
    exchange='delayed',
    exchange_type='x-delayed-message',
    arguments={'x-delayed-type': 'direct'}
)

channel.basic_publish(
    exchange='delayed',
    routing_key='queue',
    body=message,
    properties=pika.BasicProperties(
        headers={'x-delay': 5000}  # 5 секунд задержки
    )
)
```

**4. Shovel для миграции сообщений:**

```bash
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_shovel
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_shovel_management
```

---

## Модуль 4: Clustering и High Availability (35 минут)

### 🎯 Напоминалка

**Типы кластеризации:**

1. **Classic Mirroring (deprecated)** - синхронные копии queue
2. **Quorum Queues** - Raft consensus, рекомендуется
3. **Federation** - асинхронная репликация между кластерами
4. **Shovel** - перемещение сообщений между брокерами

**Архитектура кластера:**

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  RabbitMQ 1  │◄────►│  RabbitMQ 2  │◄────►│  RabbitMQ 3  │
│   (node1)    │      │   (node2)    │      │   (node3)    │
└──────────────┘      └──────────────┘      └──────────────┘
       │                     │                     │
       └─────────────────────┴─────────────────────┘
                    Erlang Distribution
```

**Docker Compose для кластера:**

```yaml
# docker-compose-cluster.yml
version: '3.8'

services:
  rabbitmq1:
    image: rabbitmq:3-management
    hostname: rabbit1
    environment:
      - RABBITMQ_ERLANG_COOKIE=secret_cookie
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=admin
    ports:
      - "5672:5672"
      - "15672:15672"
    networks:
      - rabbitmq-cluster

  rabbitmq2:
    image: rabbitmq:3-management
    hostname: rabbit2
    environment:
      - RABBITMQ_ERLANG_COOKIE=secret_cookie
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=admin
    ports:
      - "5673:5672"
      - "15673:15672"
    networks:
      - rabbitmq-cluster
    depends_on:
      - rabbitmq1

  rabbitmq3:
    image: rabbitmq:3-management
    hostname: rabbit3
    environment:
      - RABBITMQ_ERLANG_COOKIE=secret_cookie
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=admin
    ports:
      - "5674:5672"
      - "15674:15672"
    networks:
      - rabbitmq-cluster
    depends_on:
      - rabbitmq1

networks:
  rabbitmq-cluster:
    driver: bridge
```

**Создание кластера:**

```bash
# Запусти ноды
docker-compose -f docker-compose-cluster.yml up -d

# Подожди пока все стартанут
sleep 10

# Присоедини rabbit2 к rabbit1
docker exec rabbitmq2 rabbitmqctl stop_app
docker exec rabbitmq2 rabbitmqctl reset
docker exec rabbitmq2 rabbitmqctl join_cluster rabbit@rabbit1
docker exec rabbitmq2 rabbitmqctl start_app

# Присоедини rabbit3 к rabbit1
docker exec rabbitmq3 rabbitmqctl stop_app
docker exec rabbitmq3 rabbitmqctl reset
docker exec rabbitmq3 rabbitmqctl join_cluster rabbit@rabbit1
docker exec rabbitmq3 rabbitmqctl start_app

# Проверь статус кластера
docker exec rabbitmq1 rabbitmqctl cluster_status
```

**Quorum Queue (рекомендуется для HA):**

```python
channel.queue_declare(
    queue='ha_queue',
    durable=True,
    arguments={
        'x-queue-type': 'quorum',
        'x-quorum-initial-group-size': 3,  # количество реплик
    }
)
```

**Classic Queue Mirroring (deprecated, но еще используется):**

```bash
# Policy для HA
docker exec rabbitmq1 rabbitmqctl set_policy ha-all \
  "^ha\." '{"ha-mode":"all"}' \
  --apply-to queues

# Или для конкретного количества реплик
docker exec rabbitmq1 rabbitmqctl set_policy ha-two \
  "^ha\." '{"ha-mode":"exactly","ha-params":2,"ha-sync-mode":"automatic"}' \
  --apply-to queues
```

**HAProxy для балансировки:**

```
# haproxy.cfg
global
    maxconn 4096

defaults
    mode tcp
    timeout connect 5s
    timeout client 30s
    timeout server 30s

listen rabbitmq
    bind *:5672
    mode tcp
    balance roundrobin
    server rabbit1 rabbit1:5672 check
    server rabbit2 rabbit2:5672 check
    server rabbit3 rabbit3:5672 check

listen rabbitmq_management
    bind *:15672
    mode http
    balance roundrobin
    server rabbit1 rabbit1:15672 check
    server rabbit2 rabbit2:15672 check
    server rabbit3 rabbit3:15672 check
```

**Federation для связи кластеров:**

```bash
# Включи plugin
docker exec rabbitmq1 rabbitmq-plugins enable rabbitmq_federation
docker exec rabbitmq1 rabbitmq-plugins enable rabbitmq_federation_management

# Настрой upstream
rabbitmqctl set_parameter federation-upstream my-upstream \
  '{"uri":"amqp://guest:guest@remote-rabbit:5672","trust-user-id":false}'

# Создай policy для federation
rabbitmqctl set_policy federate-me \
  "^federated\." '{"federation-upstream-set":"all"}' \
  --apply-to exchanges
```

**Network Partitions:**

```bash
# Проверка partition
docker exec rabbitmq1 rabbitmqctl cluster_status

# Стратегии восстановления (в rabbitmq.conf):
# cluster_partition_handling = pause_minority (default)
# cluster_partition_handling = autoheal
# cluster_partition_handling = ignore
```

### 💻 Задание

**Настрой HA кластер:**

**1. Создай кластер из 3 нод:**

```bash
# Создай файл docker-compose-cluster.yml (из примера выше)

# Запусти
docker-compose -f docker-compose-cluster.yml up -d

# Дождись запуска
sleep 15

# Создай кластер
docker exec rabbitmq2 rabbitmqctl stop_app
docker exec rabbitmq2 rabbitmqctl reset
docker exec rabbitmq2 rabbitmqctl join_cluster rabbit@rabbit1
docker exec rabbitmq2 rabbitmqctl start_app

docker exec rabbitmq3 rabbitmqctl stop_app
docker exec rabbitmq3 rabbitmqctl reset
docker exec rabbitmq3 rabbitmqctl join_cluster rabbit@rabbit1
docker exec rabbitmq3 rabbitmqctl start_app

# Проверь
docker exec rabbitmq1 rabbitmqctl cluster_status
```

**2. Создай Quorum Queue:**

```python
# cluster_producer.py
import pika
import sys

# Подключение к первой ноде
connection = pika.BlockingConnection(
    pika.ConnectionParameters(
        host='localhost',
        port=5672
    )
)
channel = connection.channel()

# Quorum queue для HA
channel.queue_declare(
    queue='cluster_queue',
    durable=True,
    arguments={
        'x-queue-type': 'quorum',
    }
)

for i in range(10):
    message = f'Message {i}'
    channel.basic_publish(
        exchange='',
        routing_key='cluster_queue',
        body=message,
        properties=pika.BasicProperties(
            delivery_mode=2,
        )
    )
    print(f" [x] Sent '{message}'")

connection.close()
```

**3. Создай consumer с подключением к разным нодам:**

```python
# cluster_consumer.py
import pika
import time
import random

NODES = [
    ('localhost', 5672),
    ('localhost', 5673),
    ('localhost', 5674),
]

def connect():
    """Подключение к случайной ноде"""
    host, port = random.choice(NODES)
    try:
        connection = pika.BlockingConnection(
            pika.ConnectionParameters(
                host=host,
                port=port,
                heartbeat=600,
                connection_attempts=3,
                retry_delay=2
            )
        )
        print(f" [✓] Connected to {host}:{port}")
        return connection
    except Exception as e:
        print(f" [✗] Failed to connect to {host}:{port}: {e}")
        return None

def callback(ch, method, properties, body):
    print(f" [x] Received {body.decode()}")
    time.sleep(1)
    ch.basic_ack(delivery_tag=method.delivery_tag)

def main():
    while True:
        connection = connect()
        if not connection:
            time.sleep(5)
            continue
        
        try:
            channel = connection.channel()
            channel.queue_declare(
                queue='cluster_queue',
                durable=True,
                arguments={'x-queue-type': 'quorum'}
            )
            channel.basic_qos(prefetch_count=1)
            channel.basic_consume(
                queue='cluster_queue',
                on_message_callback=callback,
                auto_ack=False
            )
            
            print(' [*] Waiting for messages')
            channel.start_consuming()
        except KeyboardInterrupt:
            break
        except Exception as e:
            print(f" [!] Error: {e}")
            time.sleep(5)
        finally:
            try:
                connection.close()
            except:
                pass

if __name__ == '__main__':
    main()
```

**4. Протестируй failover:**

```bash
# Терминал 1: запусти consumer
python cluster_consumer.py

# Терминал 2: отправь сообщения
python cluster_producer.py

# Терминал 3: останови одну ноду
docker stop rabbitmq2

# Проверь что consumer продолжает работать
# Отправь еще сообщений
python cluster_producer.py

# Верни ноду обратно
docker start rabbitmq2

# Останови другую ноду
docker stop rabbitmq1

# Consumer должен переподключиться автоматически
```

**5. Проверь состояние кластера:**

```bash
# Статус кластера
docker exec rabbitmq1 rabbitmqctl cluster_status

# Quorum queue статус
docker exec rabbitmq1 rabbitmqctl list_queues name type messages_ram messages_persistent state

# Проверь в Web UI:
# http://localhost:15672/#/queues
# Посмотри на "Node" и "Mirrors" колонки
```

### 🚀 Бонус (новое)

**1. Настрой автоматический sync политику:**

```bash
# Policy для quorum queues
docker exec rabbitmq1 rabbitmqctl set_policy quorum-policy \
  "^quorum\." \
  '{"queue-type":"quorum","delivery-limit":3}' \
  --apply-to queues
```

**2. Мониторинг кластера с Prometheus:**

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'rabbitmq'
    static_configs:
      - targets:
        - 'rabbit1:15692'
        - 'rabbit2:15692'
        - 'rabbit3:15692'
```

**3. Blue-Green deployment для кластера:**

```bash
# Drain одну ноду (перенести все queues)
docker exec rabbitmq1 rabbitmqctl suspend_listeners
docker exec rabbitmq1 rabbitmqctl await_startup

# Обнови ноду
docker stop rabbitmq1
docker rm rabbitmq1
# ... update and restart ...

# Верни в кластер
docker exec rabbitmq1 rabbitmqctl resume_listeners
```

---

## Модуль 5: Мониторинг и Observability (30 минут)

### 🎯 Напоминалка

**Management Plugin метрики:**

- Overview: Общая информация о брокере
- Connections: Активные подключения
- Channels: Каналы
- Exchanges: Exchange'ы и их трафик
- Queues: Queue'и, сообщения, consumers

**HTTP API endpoints:**

```bash
# Overview
curl -u admin:admin http://localhost:15672/api/overview

# Nodes
curl -u admin:admin http://localhost:15672/api/nodes

# Connections
curl -u admin:admin http://localhost:15672/api/connections

# Channels
curl -u admin:admin http://localhost:15672/api/channels

# Queues
curl -u admin:admin http://localhost:15672/api/queues

# Specific queue
curl -u admin:admin http://localhost:15672/api/queues/%2F/my_queue

# Consumers
curl -u admin:admin http://localhost:15672/api/consumers
```

**Prometheus метрики:**

```bash
# Включи prometheus plugin
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_prometheus

# Метрики доступны на
curl http://localhost:15692/metrics
```

**Ключевые метрики для мониторинга:**

```
# Queue metrics
rabbitmq_queue_messages_ready          # Готовые сообщения
rabbitmq_queue_messages_unacknowledged # Unacked сообщения
rabbitmq_queue_messages                # Всего сообщений
rabbitmq_queue_consumers               # Количество consumers

# Connection metrics
rabbitmq_connections                   # Активные соединения
rabbitmq_channels                      # Активные каналы

# Node metrics
rabbitmq_node_mem_used                 # Используемая память
rabbitmq_node_fd_used                  # File descriptors
rabbitmq_node_sockets_used             # Сокеты
rabbitmq_node_disk_free                # Свободное место

# Message rates
rabbitmq_queue_messages_published_total
rabbitmq_queue_messages_delivered_total
rabbitmq_queue_messages_redelivered_total
```

**Алерты (примеры):**

```yaml
# prometheus_alerts.yml
groups:
  - name: rabbitmq
    rules:
      # Слишком много сообщений в queue
      - alert: RabbitMQQueueMessagesHigh
        expr: rabbitmq_queue_messages > 1000
        for: 5m
        annotations:
          summary: "Queue {{ $labels.queue }} has too many messages"
      
      # Нет consumers
      - alert: RabbitMQNoConsumers
        expr: rabbitmq_queue_consumers == 0
        for: 5m
        annotations:
          summary: "Queue {{ $labels.queue }} has no consumers"
      
      # Высокое использование памяти
      - alert: RabbitMQMemoryHigh
        expr: rabbitmq_node_mem_used / rabbitmq_node_mem_limit > 0.9
        for: 5m
        annotations:
          summary: "Node {{ $labels.node }} memory usage > 90%"
      
      # Много unacked сообщений
      - alert: RabbitMQUnackedMessagesHigh
        expr: rabbitmq_queue_messages_unacknowledged > 100
        for: 10m
        annotations:
          summary: "Queue {{ $labels.queue }} has many unacked messages"
```

**Логирование:**

```bash
# Уровни логов
docker exec rabbitmq rabbitmqctl set_log_level debug
docker exec rabbitmq rabbitmqctl set_log_level info  # default
docker exec rabbitmq rabbitmqctl set_log_level warning
docker exec rabbitmq rabbitmqctl set_log_level error

# Логи
docker logs rabbitmq
docker logs -f rabbitmq --tail 100
```

**rabbitmq.conf для логирования:**

```
# rabbitmq.conf
log.file.level = info
log.console.level = info
log.file = /var/log/rabbitmq/rabbit.log
log.file.rotation.count = 5
log.file.rotation.size = 10485760
```

**Tracing (для отладки):**

```bash
# Включи tracing plugin
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_tracing

# Создай trace через Web UI или API
curl -u admin:admin -X PUT http://localhost:15672/api/traces/%2F/my-trace \
  -H "content-type:application/json" \
  -d '{"format":"text","pattern":"#"}'

# Выключи trace
curl -u admin:admin -X DELETE http://localhost:15672/api/traces/%2F/my-trace
```

### 💻 Задание

**Настрой полноценный мониторинг:**

**1. Настрой Prometheus + Grafana:**

```yaml
# docker-compose-monitoring.yml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
      - "15692:15692"
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=admin
    command: >
      bash -c "
        rabbitmq-plugins enable rabbitmq_prometheus &&
        rabbitmq-server
      "

  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alerts.yml:/etc/prometheus/alerts.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-storage:/var/lib/grafana

volumes:
  grafana-storage:
```

**prometheus.yml:**

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - 'alerts.yml'

scrape_configs:
  - job_name: 'rabbitmq'
    static_configs:
      - targets: ['rabbitmq:15692']
```

**alerts.yml:**

```yaml
groups:
  - name: rabbitmq_alerts
    rules:
      - alert: RabbitMQDown
        expr: up{job="rabbitmq"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "RabbitMQ is down"
      
      - alert: RabbitMQQueueMessagesHigh
        expr: rabbitmq_queue_messages > 1000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Queue {{ $labels.queue }} has {{ $value }} messages"
      
      - alert: RabbitMQNoConsumers
        expr: rabbitmq_queue_consumers == 0 and rabbitmq_queue_messages > 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Queue {{ $labels.queue }} has no consumers but has messages"
      
      - alert: RabbitMQMemoryHigh
        expr: (rabbitmq_node_mem_used / rabbitmq_node_mem_limit) > 0.9
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Memory usage > 90% on {{ $labels.node }}"
```

**2. Запусти мониторинг:**

```bash
# Запусти stack
docker-compose -f docker-compose-monitoring.yml up -d

# Проверь
docker-compose -f docker-compose-monitoring.yml ps

# Доступ:
# RabbitMQ Management: http://localhost:15672
# Prometheus: http://localhost:9090
# Grafana: http://localhost:3000 (admin/admin)
```

**3. Настрой Grafana:**

```bash
# Зайди в Grafana: http://localhost:3000
# Login: admin / admin

# Добавь Prometheus data source:
# Configuration → Data Sources → Add data source → Prometheus
# URL: http://prometheus:9090

# Импортируй RabbitMQ dashboard:
# Create → Import → Dashboard ID: 10991
# (RabbitMQ-Overview от rabbitmq-prometheus)
```

**4. Создай скрипт для генерации метрик:**

```python
# generate_metrics.py
import pika
import time
import random

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

# Создаём несколько queues
queues = ['metrics_queue_1', 'metrics_queue_2', 'metrics_queue_3']

for queue in queues:
    channel.queue_declare(queue=queue, durable=True)

print("Generating metrics...")

for i in range(100):
    queue = random.choice(queues)
    message = f'Message {i}'
    
    channel.basic_publish(
        exchange='',
        routing_key=queue,
        body=message,
        properties=pika.BasicProperties(delivery_mode=2)
    )
    
    if i % 10 == 0:
        print(f"Sent {i} messages")
    
    time.sleep(0.1)

connection.close()
print("Done!")
```

**5. Мониторь метрики:**

```bash
# Генерируй нагрузку
python generate_metrics.py

# Проверь метрики в Prometheus:
# http://localhost:9090/graph
# Запросы:
# - rabbitmq_queue_messages
# - rate(rabbitmq_queue_messages_published_total[1m])
# - rabbitmq_node_mem_used

# Смотри в Grafana dashboard
```

### 🚀 Бонус (новое)

**1. Custom метрики через StatsD:**

```bash
# Включи plugin
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_statsd
```

**2. ELK stack для логов:**

```yaml
# Добавь в docker-compose
  elasticsearch:
    image: elasticsearch:7.17.0
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"

  logstash:
    image: logstash:7.17.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

  kibana:
    image: kibana:7.17.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
```

**3. Health Check endpoints:**

```python
import requests

def check_rabbitmq_health():
    try:
        # Aliveness test
        response = requests.get(
            'http://localhost:15672/api/aliveness-test/%2F',
            auth=('admin', 'admin')
        )
        
        if response.status_code == 200:
            data = response.json()
            if data.get('status') == 'ok':
                print("✓ RabbitMQ is healthy")
                return True
        
        print("✗ RabbitMQ health check failed")
        return False
    except Exception as e:
        print(f"✗ Error: {e}")
        return False

if __name__ == '__main__':
    check_rabbitmq_health()
```

---

## Модуль 6: Security и Best Practices (25 минут)

### 🎯 Напоминалка

**Authentication:**

```bash
# Создание пользователя
docker exec rabbitmq rabbitmqctl add_user myuser mypassword

# Назначение тегов (roles)
docker exec rabbitmq rabbitmqctl set_user_tags myuser administrator
# Доступные теги: administrator, monitoring, policymaker, management

# Удаление пользователя
docker exec rabbitmq rabbitmqctl delete_user myuser

# Смена пароля
docker exec rabbitmq rabbitmqctl change_password myuser newpassword
```

**Permissions (vhost level):**

```bash
# Формат: configure, write, read permissions (regex)
docker exec rabbitmq rabbitmqctl set_permissions -p / myuser ".*" ".*" ".*"

# Только чтение
docker exec rabbitmq rabbitmqctl set_permissions -p / readonly_user "" "" ".*"

# Только запись
docker exec rabbitmq rabbitmqctl set_permissions -p / writeonly_user "" ".*" ""

# Specific queues
docker exec rabbitmq rabbitmqctl set_permissions -p / app_user "^app-.*" "^app-.*" "^app-.*"

# Просмотр permissions
docker exec rabbitmq rabbitmqctl list_permissions -p /
docker exec rabbitmq rabbitmqctl list_user_permissions myuser
```

**Virtual Hosts (изоляция):**

```bash
# Создание vhost
docker exec rabbitmq rabbitmqctl add_vhost /dev
docker exec rabbitmq rabbitmqctl add_vhost /prod

# Permissions для vhost
docker exec rabbitmq rabbitmqctl set_permissions -p /dev dev_user ".*" ".*" ".*"

# Удаление vhost
docker exec rabbitmq rabbitmqctl delete_vhost /dev
```

**TLS/SSL:**

**rabbitmq.conf:**

```
listeners.ssl.default = 5671

ssl_options.cacertfile = /path/to/ca_certificate.pem
ssl_options.certfile   = /path/to/server_certificate.pem
ssl_options.keyfile    = /path/to/server_key.pem
ssl_options.verify     = verify_peer
ssl_options.fail_if_no_peer_cert = true
```

**Python client с TLS:**

```python
import pika
import ssl

context = ssl.create_default_context(
    cafile="/path/to/ca_certificate.pem"
)
context.load_cert_chain(
    "/path/to/client_certificate.pem",
    "/path/to/client_key.pem"
)

ssl_options = pika.SSLOptions(context, "rabbitmq.example.com")

connection = pika.BlockingConnection(
    pika.ConnectionParameters(
        host='rabbitmq.example.com',
        port=5671,
        ssl_options=ssl_options,
        credentials=pika.PlainCredentials('user', 'pass')
    )
)
```

**Resource Limits:**

**rabbitmq.conf:**

```
# Memory
vm_memory_high_watermark.relative = 0.6
vm_memory_high_watermark_paging_ratio = 0.75

# Disk
disk_free_limit.absolute = 2GB
disk_free_limit.relative = 1.0

# Connection limits
connection_max = 1000

# Channel limits per connection
channel_max = 100
```

**Policy-based limits:**

```bash
# Max length policy
docker exec rabbitmq rabbitmqctl set_policy max-length-policy \
  "^limited-.*" \
  '{"max-length":1000,"overflow":"reject-publish"}' \
  --apply-to queues

# TTL policy
docker exec rabbitmq rabbitmqctl set_policy ttl-policy \
  "^temp-.*" \
  '{"message-ttl":60000,"expires":300000}' \
  --apply-to queues

# Max priority
docker exec rabbitmq rabbitmqctl set_policy priority-policy \
  "^priority-.*" \
  '{"max-priority":10}' \
  --apply-to queues
```

**Network Segmentation:**

```bash
# Bind to specific interface
listeners.tcp.default = 192.168.1.100:5672

# Management only on localhost
management.listener.port = 15672
management.listener.ip   = 127.0.0.1
```

**Audit Logging:**

```
# rabbitmq.conf
management.enable_queue_totals = true
management.enable_connection_lifecycle_events = true
```

### 💻 Задание

**Настрой secure RabbitMQ:**

**1. Создай пользователей с разными ролями:**

```bash
# Admin user
docker exec rabbitmq rabbitmqctl add_user admin_user admin_pass
docker exec rabbitmq rabbitmqctl set_user_tags admin_user administrator
docker exec rabbitmq rabbitmqctl set_permissions -p / admin_user ".*" ".*" ".*"

# Application user (producer)
docker exec rabbitmq rabbitmqctl add_user producer_user prod_pass
docker exec rabbitmq rabbitmqctl set_user_tags producer_user management
docker exec rabbitmq rabbitmqctl set_permissions -p / producer_user "" "^app\..*" ""

# Application user (consumer)
docker exec rabbitmq rabbitmqctl add_user consumer_user cons_pass
docker exec rabbitmq rabbitmqctl set_user_tags consumer_user management
docker exec rabbitmq rabbitmqctl set_permissions -p / consumer_user "" "" "^app\..*"

# Monitoring user
docker exec rabbitmq rabbitmqctl add_user monitor_user monitor_pass
docker exec rabbitmq rabbitmqctl set_user_tags monitor_user monitoring
docker exec rabbitmq rabbitmqctl set_permissions -p / monitor_user "" "" ".*"

# Проверь
docker exec rabbitmq rabbitmqctl list_users
docker exec rabbitmq rabbitmqctl list_permissions -p /
```

**2. Создай изолированные vhosts:**

```bash
# Development vhost
docker exec rabbitmq rabbitmqctl add_vhost /dev
docker exec rabbitmq rabbitmqctl add_user dev_user dev_pass
docker exec rabbitmq rabbitmqctl set_permissions -p /dev dev_user ".*" ".*" ".*"

# Production vhost
docker exec rabbitmq rabbitmqctl add_vhost /prod
docker exec rabbitmq rabbitmqctl add_user prod_user prod_pass
docker exec rabbitmq rabbitmqctl set_permissions -p /prod prod_user ".*" ".*" ".*"

# Проверь
docker exec rabbitmq rabbitmqctl list_vhosts
```

**3. Настрой producer с ограниченными правами:**

```python
# secure_producer.py
import pika

credentials = pika.PlainCredentials('producer_user', 'prod_pass')

connection = pika.BlockingConnection(
    pika.ConnectionParameters(
        host='localhost',
        credentials=credentials,
        virtual_host='/'
    )
)
channel = connection.channel()

# Может публиковать только в app.* queues
channel.queue_declare(queue='app.orders', durable=True)

try:
    channel.basic_publish(
        exchange='',
        routing_key='app.orders',
        body='Order #123'
    )
    print(" [✓] Message published successfully")
except Exception as e:
    print(f" [✗] Failed to publish: {e}")

# Попытка создать другую queue (должна fail)
try:
    channel.queue_declare(queue='other.queue')
    print(" [✗] Should not be able to create other.queue!")
except Exception as e:
    print(f" [✓] Correctly denied: {e}")

connection.close()
```

**4. Настрой consumer с ограниченными правами:**

```python
# secure_consumer.py
import pika

credentials = pika.PlainCredentials('consumer_user', 'cons_pass')

connection = pika.BlockingConnection(
    pika.ConnectionParameters(
        host='localhost',
        credentials=credentials,
        virtual_host='/'
    )
)
channel = connection.channel()

def callback(ch, method, properties, body):
    print(f" [x] Received {body.decode()}")
    ch.basic_ack(delivery_tag=method.delivery_tag)

try:
    channel.basic_consume(
        queue='app.orders',
        on_message_callback=callback,
        auto_ack=False
    )
    print(" [*] Waiting for messages")
    channel.start_consuming()
except KeyboardInterrupt:
    channel.stop_consuming()
except Exception as e:
    print(f" [✗] Error: {e}")

connection.close()
```

**5. Настрой resource limits:**

```bash
# Создай policy для limits
docker exec rabbitmq rabbitmqctl set_policy limits-policy \
  "^app\..*" \
  '{"max-length":10000,"max-length-bytes":104857600,"overflow":"reject-publish","message-ttl":3600000}' \
  --apply-to queues

# Проверь policies
docker exec rabbitmq rabbitmqctl list_policies
```

**6. Протестируй:**

```bash
# Запусти consumer
python secure_consumer.py

# В другом терминале запусти producer
python secure_producer.py

# Попробуй зайти в Web UI с разными пользователями
# admin_user - полный доступ
# producer_user/consumer_user - ограниченный доступ
# monitor_user - только просмотр
```

### 🚀 Бонус (новое)

**1. Настрой LDAP authentication:**

```
# rabbitmq.conf
auth_backends.1 = ldap
auth_backends.2 = internal

auth_ldap.servers.1  = ldap.example.com
auth_ldap.port       = 389
auth_ldap.user_dn_pattern = cn=${username},ou=users,dc=example,dc=com
```

**2. Включи audit logging:**

```bash
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_event_exchange
```

**3. Rate limiting для пользователей:**

```bash
# Установи max connections per user
docker exec rabbitmq rabbitmqctl set_user_limits producer_user '{"max-connections": 10}'
docker exec rabbitmq rabbitmqctl set_user_limits consumer_user '{"max-channels": 50}'
```

---

## Модуль 7: Production Deployment и Troubleshooting (35 минут)

### 🎯 Напоминалка

**Production Checklist:**

```
✅ Clustering (3+ nodes)
✅ Quorum queues для HA
✅ Monitoring (Prometheus + Grafana)
✅ Alerting настроен
✅ Backups (definitions + messages)
✅ Resource limits установлены
✅ RBAC настроен
✅ TLS включен
✅ Load balancer настроен
✅ Disaster recovery план
✅ Документация
```

**Kubernetes Deployment:**

```yaml
# rabbitmq-statefulset.yaml
apiVersion: v1
kind: Service
metadata:
  name: rabbitmq
  labels:
    app: rabbitmq
spec:
  clusterIP: None
  ports:
    - name: amqp
      port: 5672
      targetPort: 5672
    - name: management
      port: 15672
      targetPort: 15672
  selector:
    app: rabbitmq
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: rabbitmq
spec:
  serviceName: rabbitmq
  replicas: 3
  selector:
    matchLabels:
      app: rabbitmq
  template:
    metadata:
      labels:
        app: rabbitmq
    spec:
      containers:
      - name: rabbitmq
        image: rabbitmq:3-management
        env:
        - name: RABBITMQ_ERLANG_COOKIE
          valueFrom:
            secretKeyRef:
              name: rabbitmq-secret
              key: erlang-cookie
        - name: RABBITMQ_DEFAULT_USER
          valueFrom:
            secretKeyRef:
              name: rabbitmq-secret
              key: username
        - name: RABBITMQ_DEFAULT_PASS
          valueFrom:
            secretKeyRef:
              name: rabbitmq-secret
              key: password
        ports:
        - containerPort: 5672
          name: amqp
        - containerPort: 15672
          name: management
        volumeMounts:
        - name: rabbitmq-data
          mountPath: /var/lib/rabbitmq
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
          limits:
            cpu: 2000m
            memory: 2Gi
        livenessProbe:
          exec:
            command: ["rabbitmq-diagnostics", "status"]
          initialDelaySeconds: 60
          periodSeconds: 60
          timeoutSeconds: 15
        readinessProbe:
          exec:
            command: ["rabbitmq-diagnostics", "ping"]
          initialDelaySeconds: 20
          periodSeconds: 60
          timeoutSeconds: 10
  volumeClaimTemplates:
  - metadata:
      name: rabbitmq-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

**Backup и Restore:**

```bash
# Export definitions (exchanges, queues, bindings, users, etc)
docker exec rabbitmq rabbitmqctl export_definitions /tmp/definitions.json
docker cp rabbitmq:/tmp/definitions.json ./backup/definitions.json

# Import definitions
docker cp ./backup/definitions.json rabbitmq:/tmp/definitions.json
docker exec rabbitmq rabbitmqctl import_definitions /tmp/definitions.json

# Через HTTP API
curl -u admin:admin http://localhost:15672/api/definitions -o definitions.json
curl -u admin:admin -X POST -H "Content-Type: application/json" \
  -d @definitions.json http://localhost:15672/api/definitions

# Backup messages (требует shovel plugin)
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_shovel
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_shovel_management
```

**Common Issues и Solutions:**

**1. Memory Alarm:**

```bash
# Проверка
docker exec rabbitmq rabbitmqctl status | grep memory

# Решение
docker exec rabbitmq rabbitmqctl set_vm_memory_high_watermark 0.7

# Или в rabbitmq.conf
vm_memory_high_watermark.relative = 0.7
```

**2. Disk Space Alarm:**

```bash
# Проверка
docker exec rabbitmq rabbitmqctl status | grep disk

# Решение - освободи место или увеличь лимит
docker exec rabbitmq rabbitmqctl set_disk_free_limit 2GB

# Или в rabbitmq.conf
disk_free_limit.absolute = 2GB
```

**3. Connection Leak:**

```bash
# Найди зависшие connections
docker exec rabbitmq rabbitmqctl list_connections name state timeout | grep blocked

# Закрой connection
docker exec rabbitmq rabbitmqctl close_connection "<connection_name>" "Cleanup"

# Найди все connections от IP
docker exec rabbitmq rabbitmqctl list_connections peer_host | grep 192.168.1.100
```

**4. High Unacked Messages:**

```bash
# Проверка
docker exec rabbitmq rabbitmqctl list_queues name messages_unacknowledged

# Причины:
# - Consumer медленно обрабатывает
# - Consumer умер без ack
# - Prefetch слишком большой

# Решение - уменьши prefetch
channel.basic_qos(prefetch_count=1)
```

**5. Queue Blocked:**

```bash
# Проверка
docker exec rabbitmq rabbitmqctl list_queues name state

# Причины: memory alarm, disk alarm, flow control

# Проверь алармы
docker exec rabbitmq rabbitmqctl alarm_list
```

**Troubleshooting Commands:**

```bash
# Диагностика
docker exec rabbitmq rabbitmq-diagnostics status
docker exec rabbitmq rabbitmq-diagnostics check_running
docker exec rabbitmq rabbitmq-diagnostics check_local_alarms
docker exec rabbitmq rabbitmq-diagnostics check_port_connectivity
docker exec rabbitmq rabbitmq-diagnostics memory_breakdown

# Network
docker exec rabbitmq rabbitmq-diagnostics check_port_listener 5672
docker exec rabbitmq rabbitmq-diagnostics check_protocol_listener amqp

# Cluster
docker exec rabbitmq rabbitmq-diagnostics cluster_status
docker exec rabbitmq rabbitmq-diagnostics check_if_node_is_quorum_critical

# Consumers
docker exec rabbitmq rabbitmqctl list_consumers queue_name channel_pid

# Report
docker exec rabbitmq rabbitmq-diagnostics status > report.txt
docker exec rabbitmq rabbitmq-diagnostics environment >> report.txt
```

**Performance Tuning:**

```
# rabbitmq.conf

# Disk I/O
queue_index_embed_msgs_below = 4096

# Network
tcp_listen_options.backlog = 128
tcp_listen_options.nodelay = true
tcp_listen_options.sndbuf = 196608
tcp_listen_options.recbuf = 196608

# Channel
channel_max = 256

# Heartbeat
heartbeat = 60

# Collect statistics (disable in production for performance)
collect_statistics = coarse
collect_statistics_interval = 5000
```

### 💻 Задание

**Создай production-ready deployment:**

**1. Создай скрипт для backup:**

```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="./backups/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

echo "Starting RabbitMQ backup..."

# Backup definitions
docker exec rabbitmq rabbitmqctl export_definitions /tmp/definitions.json
docker cp rabbitmq:/tmp/definitions.json "$BACKUP_DIR/definitions.json"
echo "✓ Definitions backed up"

# Backup queue stats
docker exec rabbitmq rabbitmqctl list_queues name messages messages_ready messages_unacknowledged > "$BACKUP_DIR/queue_stats.txt"
echo "✓ Queue stats backed up"

# Backup connections
docker exec rabbitmq rabbitmqctl list_connections > "$BACKUP_DIR/connections.txt"
echo "✓ Connections backed up"

# Backup users
docker exec rabbitmq rabbitmqctl list_users > "$BACKUP_DIR/users.txt"
echo "✓ Users backed up"

# Compress
tar -czf "$BACKUP_DIR.tar.gz" -C ./backups "$(basename $BACKUP_DIR)"
rm -rf "$BACKUP_DIR"

echo "✓ Backup completed: $BACKUP_DIR.tar.gz"
```

**2. Создай скрипт для restore:**

```bash
#!/bin/bash
# restore.sh

if [ -z "$1" ]; then
    echo "Usage: $0 <backup_file.tar.gz>"
    exit 1
fi

BACKUP_FILE="$1"
TEMP_DIR="./temp_restore"

echo "Starting RabbitMQ restore from $BACKUP_FILE..."

# Extract
mkdir -p "$TEMP_DIR"
tar -xzf "$BACKUP_FILE" -C "$TEMP_DIR"

# Find definitions file
DEFINITIONS_FILE=$(find "$TEMP_DIR" -name "definitions.json")

if [ -z "$DEFINITIONS_FILE" ]; then
    echo "✗ definitions.json not found in backup"
    exit 1
fi

# Restore definitions
docker cp "$DEFINITIONS_FILE" rabbitmq:/tmp/definitions.json
docker exec rabbitmq rabbitmqctl import_definitions /tmp/definitions.json

echo "✓ Definitions restored"

# Cleanup
rm -rf "$TEMP_DIR"

echo "✓ Restore completed"
```

**3. Создай health check скрипт:**

```python
# health_check.py
import requests
import sys

def check_aliveness():
    try:
        response = requests.get(
            'http://localhost:15672/api/aliveness-test/%2F',
            auth=('admin', 'admin'),
            timeout=5
        )
        return response.status_code == 200 and response.json().get('status') == 'ok'
    except Exception as e:
        print(f"Aliveness check failed: {e}")
        return False

def check_memory():
    try:
        response = requests.get(
            'http://localhost:15672/api/nodes',
            auth=('admin', 'admin'),
            timeout=5
        )
        
        if response.status_code != 200:
            return False
        
        nodes = response.json()
        for node in nodes:
            mem_used = node.get('mem_used', 0)
            mem_limit = node.get('mem_limit', 1)
            mem_percent = (mem_used / mem_limit) * 100
            
            if mem_percent > 90:
                print(f"⚠ Memory usage high: {mem_percent:.1f}%")
                return False
        
        return True
    except Exception as e:
        print(f"Memory check failed: {e}")
        return False

def check_disk():
    try:
        response = requests.get(
            'http://localhost:15672/api/nodes',
            auth=('admin', 'admin'),
            timeout=5
        )
        
        if response.status_code != 200:
            return False
        
        nodes = response.json()
        for node in nodes:
            disk_free = node.get('disk_free', 0)
            disk_limit = node.get('disk_free_limit', 1)
            
            if disk_free < disk_limit:
                print(f"⚠ Disk space low: {disk_free} < {disk_limit}")
                return False
        
        return True
    except Exception as e:
        print(f"Disk check failed: {e}")
        return False

def check_queues():
    try:
        response = requests.get(
            'http://localhost:15672/api/queues',
            auth=('admin', 'admin'),
            timeout=5
        )
        
        if response.status_code != 200:
            return False
        
        queues = response.json()
        issues = []
        
        for queue in queues:
            name = queue['name']
            messages = queue.get('messages', 0)
            consumers = queue.get('consumers', 0)
            
            # Проверка: много сообщений, но нет consumers
            if messages > 100 and consumers == 0:
                issues.append(f"Queue '{name}' has {messages} messages but no consumers")
            
            # Проверка: много unacked сообщений
            unacked = queue.get('messages_unacknowledged', 0)
            if unacked > 1000:
                issues.append(f"Queue '{name}' has {unacked} unacked messages")
        
        if issues:
            for issue in issues:
                print(f"⚠ {issue}")
            return False
        
        return True
    except Exception as e:
        print(f"Queue check failed: {e}")
        return False

def main():
    checks = {
        'Aliveness': check_aliveness,
        'Memory': check_memory,
        'Disk': check_disk,
        'Queues': check_queues,
    }
    
    print("Running RabbitMQ health checks...")
    print("-" * 40)
    
    all_passed = True
    for name, check_func in checks.items():
        result = check_func()
        status = "✓ PASS" if result else "✗ FAIL"
        print(f"{name:20s} {status}")
        
        if not result:
            all_passed = False
    
    print("-" * 40)
    
    if all_passed:
        print("✓ All checks passed")
        sys.exit(0)
    else:
        print("✗ Some checks failed")
        sys.exit(1)

if __name__ == '__main__':
    main()
```

**4. Создай stress test:**

```python
# stress_test.py
import pika
import threading
import time
import argparse

class Producer(threading.Thread):
    def __init__(self, thread_id, num_messages):
        threading.Thread.__init__(self)
        self.thread_id = thread_id
        self.num_messages = num_messages
        self.sent = 0
    
    def run(self):
        connection = pika.BlockingConnection(
            pika.ConnectionParameters('localhost')
        )
        channel = connection.channel()
        channel.queue_declare(queue='stress_test', durable=True)
        
        for i in range(self.num_messages):
            message = f'Thread-{self.thread_id} Message-{i}'
            channel.basic_publish(
                exchange='',
                routing_key='stress_test',
                body=message,
                properties=pika.BasicProperties(delivery_mode=2)
            )
            self.sent += 1
        
        connection.close()
        print(f"Producer {self.thread_id} sent {self.sent} messages")

class Consumer(threading.Thread):
    def __init__(self, thread_id, duration):
        threading.Thread.__init__(self)
        self.thread_id = thread_id
        self.duration = duration
        self.received = 0
        self.running = True
    
    def callback(self, ch, method, properties, body):
        self.received += 1
        ch.basic_ack(delivery_tag=method.delivery_tag)
    
    def run(self):
        connection = pika.BlockingConnection(
            pika.ConnectionParameters('localhost')
        )
        channel = connection.channel()
        channel.queue_declare(queue='stress_test', durable=True)
        channel.basic_qos(prefetch_count=10)
        
        channel.basic_consume(
            queue='stress_test',
            on_message_callback=self.callback,
            auto_ack=False
        )
        
        start_time = time.time()
        while self.running and (time.time() - start_time) < self.duration:
            connection.process_data_events(time_limit=1)
        
        connection.close()
        print(f"Consumer {self.thread_id} received {self.received} messages")

def main():
    parser = argparse.ArgumentParser(description='RabbitMQ Stress Test')
    parser.add_argument('--producers', type=int, default=5, help='Number of producers')
    parser.add_argument('--consumers', type=int, default=5, help='Number of consumers')
    parser.add_argument('--messages', type=int, default=1000, help='Messages per producer')
    parser.add_argument('--duration', type=int, default=60, help='Consumer duration (seconds)')
    
    args = parser.parse_args()
    
    print(f"Starting stress test:")
    print(f"  Producers: {args.producers}")
    print(f"  Consumers: {args.consumers}")
    print(f"  Messages per producer: {args.messages}")
    print(f"  Consumer duration: {args.duration}s")
    print("-" * 40)
    
    # Start producers
    producers = []
    for i in range(args.producers):
        p = Producer(i, args.messages)
        p.start()
        producers.append(p)
    
    # Start consumers
    consumers = []
    for i in range(args.consumers):
        c = Consumer(i, args.duration)
        c.start()
        consumers.append(c)
    
    # Wait for completion
    for p in producers:
        p.join()
    
    print("-" * 40)
    print("All producers completed")
    
    time.sleep(args.duration)
    
    for c in consumers:
        c.running = False
    
    for c in consumers:
        c.join()
    
    print("-" * 40)
    total_sent = sum(p.sent for p in producers)
    total_received = sum(c.received for c in consumers)
    
    print(f"Total sent: {total_sent}")
    print(f"Total received: {total_received}")
    print(f"Throughput: {total_received / args.duration:.2f} msg/s")

if __name__ == '__main__':
    main()
```

**5. Запусти тесты:**

```bash
# Health check
python health_check.py

# Backup
chmod +x backup.sh
./backup.sh

# Stress test
python stress_test.py --producers 10 --consumers 10 --messages 5000

# Monitor во время stress test
watch -n 1 'docker exec rabbitmq rabbitmqctl list_queues name messages consumers'
```

### 🚀 Бонус (новое)

**1. Настрой автоматический failover с Consul:**

```yaml
# consul-template для динамического конфига
{{range service "rabbitmq"}}
server {{.Address}}:{{.Port}} check
{{end}}
```

**2. Blue-Green Deployment Strategy:**

```bash
# 1. Разверни новую версию (green)
docker-compose -f docker-compose-green.yml up -d

# 2. Drain старую версию (blue)
# Останови publishers на blue
# Дождись обработки всех сообщений

# 3. Переключи load balancer на green

# 4. Удали blue после подтверждения
```

**3. Chaos Engineering:**

```bash
# Установи pumba для chaos testing
docker run -d --name pumba \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gaiaadm/pumba \
  --random \
  --interval 1m \
  kill --signal SIGKILL \
  re2:^rabbitmq
```

---

## Модуль 8: Advanced Patterns и Use Cases ((30 минут))

### Event-Driven Architecture 

```python
# event_bus.py (продолжение)
        self.channel.basic_publish(
            exchange='events',
            routing_key=routing_key,
            body=json.dumps(event),
            properties=pika.BasicProperties(
                delivery_mode=2,
                content_type='application/json',
                message_id=event['id']
            )
        )
        print(f" [x] Published event: {event_type}")
        return event['id']
    
    def subscribe(self, event_pattern, callback):
        """Subscribe to events matching pattern"""
        result = self.channel.queue_declare(queue='', exclusive=True)
        queue_name = result.method.queue
        
        routing_key = event_pattern.replace('.', '_').lower()
        self.channel.queue_bind(
            exchange='events',
            queue=queue_name,
            routing_key=routing_key
        )
        
        def wrapper(ch, method, properties, body):
            event = json.loads(body)
            callback(event)
            ch.basic_ack(delivery_tag=method.delivery_tag)
        
        self.channel.basic_consume(
            queue=queue_name,
            on_message_callback=wrapper,
            auto_ack=False
        )
        print(f" [*] Subscribed to: {event_pattern}")
    
    def start(self):
        print(" [*] Starting event bus...")
        self.channel.start_consuming()
    
    def close(self):
        self.connection.close()


# order_service.py
import time
from event_bus import EventBus

def handle_payment_processed(event):
    print(f" [Order] Payment processed for order: {event['data']['order_id']}")
    # Ship order
    time.sleep(1)
    event_bus.publish('order.shipped', {
        'order_id': event['data']['order_id'],
        'tracking_number': 'TRACK123'
    })

def handle_payment_failed(event):
    print(f" [Order] Payment failed, cancelling order: {event['data']['order_id']}")
    # Cancel order

event_bus = EventBus()
event_bus.subscribe('payment.processed', handle_payment_processed)
event_bus.subscribe('payment.failed', handle_payment_failed)

# Create new order
order_data = {'order_id': 'ORD-001', 'amount': 99.99}
event_bus.publish('order.created', order_data)

event_bus.start()


# payment_service.py
import random
import time
from event_bus import EventBus

def handle_order_created(event):
    order_id = event['data']['order_id']
    amount = event['data']['amount']
    
    print(f" [Payment] Processing payment for order {order_id}: ${amount}")
    time.sleep(2)
    
    # Simulate payment processing
    if random.random() > 0.2:  # 80% success rate
        event_bus.publish('payment.processed', {
            'order_id': order_id,
            'amount': amount,
            'transaction_id': 'TXN-' + str(int(time.time()))
        })
    else:
        event_bus.publish('payment.failed', {
            'order_id': order_id,
            'reason': 'Insufficient funds'
        })

event_bus = EventBus()
event_bus.subscribe('order.created', handle_order_created)
event_bus.start()
```

### 5. Протестируй Event-Driven систему:

```bash
# Терминал 1: Order Service
python order_service.py

# Терминал 2: Payment Service
python payment_service.py

# Наблюдай за flow событий:
# order.created → payment.processed → order.shipped
```

---

## 🚀 Бонус (новое)

### 1. Message Compression:

```python
# compressed_producer.py
import pika
import zlib
import json

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.queue_declare(queue='compressed_queue', durable=True)

# Large message
large_data = {'users': [{'id': i, 'name': f'User{i}'} for i in range(1000)]}
message = json.dumps(large_data)

# Compress
compressed = zlib.compress(message.encode())

print(f"Original size: {len(message)} bytes")
print(f"Compressed size: {len(compressed)} bytes")
print(f"Compression ratio: {len(compressed)/len(message)*100:.1f}%")

channel.basic_publish(
    exchange='',
    routing_key='compressed_queue',
    body=compressed,
    properties=pika.BasicProperties(
        delivery_mode=2,
        content_encoding='gzip'
    )
)

connection.close()


# compressed_consumer.py
import pika
import zlib
import json

def callback(ch, method, properties, body):
    # Decompress
    if properties.content_encoding == 'gzip':
        decompressed = zlib.decompress(body).decode()
        data = json.loads(decompressed)
    else:
        data = json.loads(body.decode())
    
    print(f" [x] Received {len(data['users'])} users")
    ch.basic_ack(delivery_tag=method.delivery_tag)

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.queue_declare(queue='compressed_queue', durable=True)
channel.basic_consume(
    queue='compressed_queue',
    on_message_callback=callback,
    auto_ack=False
)

channel.start_consuming()
```

### 2. Circuit Breaker Pattern:

```python
# circuit_breaker.py
import pika
import time
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED
    
    def call(self, func, *args, **kwargs):
        if self.state == CircuitState.OPEN:
            if time.time() - self.last_failure_time > self.timeout:
                print(" [Circuit] Attempting recovery (HALF_OPEN)")
                self.state = CircuitState.HALF_OPEN
            else:
                raise Exception("Circuit breaker is OPEN")
        
        try:
            result = func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise e
    
    def on_success(self):
        self.failure_count = 0
        if self.state == CircuitState.HALF_OPEN:
            print(" [Circuit] Recovery successful (CLOSED)")
            self.state = CircuitState.CLOSED
    
    def on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        
        if self.failure_count >= self.failure_threshold:
            print(f" [Circuit] Too many failures (OPEN)")
            self.state = CircuitState.OPEN


# resilient_consumer.py
import pika
import time
import random

circuit_breaker = CircuitBreaker(failure_threshold=3, timeout=10)

def process_message(body):
    # Simulate occasional failures
    if random.random() < 0.3:
        raise Exception("Processing failed")
    print(f" [x] Processed: {body.decode()}")

def callback(ch, method, properties, body):
    try:
        circuit_breaker.call(process_message, body)
        ch.basic_ack(delivery_tag=method.delivery_tag)
    except Exception as e:
        print(f" [!] Error: {e}")
        # Requeue or send to DLQ
        ch.basic_nack(delivery_tag=method.delivery_tag, requeue=False)

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.queue_declare(queue='resilient_queue', durable=True)
channel.basic_qos(prefetch_count=1)
channel.basic_consume(
    queue='resilient_queue',
    on_message_callback=callback,
    auto_ack=False
)

print(" [*] Waiting for messages with circuit breaker")
channel.start_consuming()
```

### 3. Message Batching:

```python
# batch_producer.py
import pika
import json
import time

class BatchProducer:
    def __init__(self, batch_size=100, flush_interval=5):
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters('localhost')
        )
        self.channel = self.connection.channel()
        self.channel.queue_declare(queue='batch_queue', durable=True)
        
        self.batch_size = batch_size
        self.flush_interval = flush_interval
        self.batch = []
        self.last_flush = time.time()
    
    def add(self, message):
        self.batch.append(message)
        
        if len(self.batch) >= self.batch_size:
            self.flush()
        elif time.time() - self.last_flush > self.flush_interval:
            self.flush()
    
    def flush(self):
        if not self.batch:
            return
        
        batch_message = json.dumps({
            'count': len(self.batch),
            'messages': self.batch
        })
        
        self.channel.basic_publish(
            exchange='',
            routing_key='batch_queue',
            body=batch_message,
            properties=pika.BasicProperties(delivery_mode=2)
        )
        
        print(f" [x] Flushed batch of {len(self.batch)} messages")
        self.batch = []
        self.last_flush = time.time()
    
    def close(self):
        self.flush()
        self.connection.close()


# Usage
producer = BatchProducer(batch_size=50, flush_interval=3)

for i in range(200):
    producer.add({'id': i, 'data': f'Message {i}'})
    time.sleep(0.01)

producer.close()


# batch_consumer.py
import pika
import json

def callback(ch, method, properties, body):
    batch = json.loads(body)
    print(f" [x] Received batch of {batch['count']} messages")
    
    # Process each message
    for message in batch['messages']:
        print(f"    Processing: {message['id']}")
    
    ch.basic_ack(delivery_tag=method.delivery_tag)

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.queue_declare(queue='batch_queue', durable=True)
channel.basic_consume(
    queue='batch_queue',
    on_message_callback=callback,
    auto_ack=False
)

print(" [*] Waiting for batches")
channel.start_consuming()
```

---

## Модуль 9: Финальный Проект (40 минут)

### 🎯 Цель
Создать полноценную microservices систему с RabbitMQ

### Архитектура системы

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   API        │────▶│  RabbitMQ    │────▶│   Workers    │
│  Gateway     │     │   Broker     │     │  (3 types)   │
└──────────────┘     └──────┬───────┘     └──────────────┘
                            │
                    ┌───────┴───────┐
                    ▼               ▼
              ┌──────────┐    ┌──────────┐
              │   Dead   │    │  Audit   │
              │  Letter  │    │   Log    │
              └──────────┘    └──────────┘
```

### 💻 Финальное Задание

#### 1. API Gateway:

```python
# api_gateway.py
from flask import Flask, request, jsonify
import pika
import json
import uuid

app = Flask(__name__)

def get_channel():
    connection = pika.BlockingConnection(
        pika.ConnectionParameters('localhost')
    )
    channel = connection.channel()
    
    # Declare exchanges
    channel.exchange_declare(
        exchange='tasks',
        exchange_type='topic',
        durable=True
    )
    
    return connection, channel

@app.route('/api/task', methods=['POST'])
def create_task():
    data = request.json
    task_type = data.get('type', 'general')
    
    connection, channel = get_channel()
    
    task_id = str(uuid.uuid4())
    message = {
        'task_id': task_id,
        'type': task_type,
        'data': data.get('data', {}),
        'priority': data.get('priority', 5)
    }
    
    routing_key = f'task.{task_type}'
    
    channel.basic_publish(
        exchange='tasks',
        routing_key=routing_key,
        body=json.dumps(message),
        properties=pika.BasicProperties(
            delivery_mode=2,
            priority=message['priority'],
            message_id=task_id,
            content_type='application/json'
        )
    )
    
    connection.close()
    
    return jsonify({
        'status': 'success',
        'task_id': task_id,
        'message': f'Task {task_type} queued'
    }), 202

@app.route('/health', methods=['GET'])
def health():
    try:
        connection, channel = get_channel()
        # Test connection
        channel.queue_declare(queue='health_check', passive=True)
        connection.close()
        return jsonify({'status': 'healthy'}), 200
    except Exception as e:
        return jsonify({'status': 'unhealthy', 'error': str(e)}), 503

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

#### 2. Worker Base Class:

```python
# worker_base.py
import pika
import json
import time
import logging
from abc import ABC, abstractmethod

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

class WorkerBase(ABC):
    def __init__(self, worker_type, queue_name, routing_keys):
        self.worker_type = worker_type
        self.queue_name = queue_name
        self.routing_keys = routing_keys
        self.logger = logging.getLogger(worker_type)
        
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters(
                host='localhost',
                heartbeat=600,
                blocked_connection_timeout=300
            )
        )
        self.channel = self.connection.channel()
        
        self.setup_queues()
    
    def setup_queues(self):
        # Main exchange
        self.channel.exchange_declare(
            exchange='tasks',
            exchange_type='topic',
            durable=True
        )
        
        # DLX
        self.channel.exchange_declare(
            exchange='dlx',
            exchange_type='fanout',
            durable=True
        )
        
        # Dead letter queue
        self.channel.queue_declare(
            queue='dead_letters',
            durable=True
        )
        self.channel.queue_bind(
            exchange='dlx',
            queue='dead_letters'
        )
        
        # Worker queue
        self.channel.queue_declare(
            queue=self.queue_name,
            durable=True,
            arguments={
                'x-dead-letter-exchange': 'dlx',
                'x-max-priority': 10,
                'x-message-ttl': 3600000  # 1 hour
            }
        )
        
        # Bind routing keys
        for routing_key in self.routing_keys:
            self.channel.queue_bind(
                exchange='tasks',
                queue=self.queue_name,
                routing_key=routing_key
            )
        
        self.channel.basic_qos(prefetch_count=1)
    
    @abstractmethod
    def process_task(self, task):
        """Override this method to implement task processing"""
        pass
    
    def callback(self, ch, method, properties, body):
        try:
            task = json.loads(body)
            self.logger.info(f"Processing task: {task['task_id']}")
            
            start_time = time.time()
            result = self.process_task(task)
            duration = time.time() - start_time
            
            self.logger.info(
                f"Task {task['task_id']} completed in {duration:.2f}s"
            )
            
            # Acknowledge
            ch.basic_ack(delivery_tag=method.delivery_tag)
            
            # Publish result to audit log
            self.publish_audit_log(task, result, duration)
            
        except Exception as e:
            self.logger.error(f"Error processing task: {e}")
            
            # Reject and don't requeue (goes to DLX)
            ch.basic_nack(
                delivery_tag=method.delivery_tag,
                requeue=False
            )
    
    def publish_audit_log(self, task, result, duration):
        self.channel.exchange_declare(
            exchange='audit',
            exchange_type='fanout',
            durable=True
        )
        
        audit_message = {
            'task_id': task['task_id'],
            'worker_type': self.worker_type,
            'duration': duration,
            'result': result,
            'timestamp': time.time()
        }
        
        self.channel.basic_publish(
            exchange='audit',
            routing_key='',
            body=json.dumps(audit_message),
            properties=pika.BasicProperties(delivery_mode=2)
        )
    
    def start(self):
        self.logger.info(f"Starting {self.worker_type} worker...")
        self.channel.basic_consume(
            queue=self.queue_name,
            on_message_callback=self.callback,
            auto_ack=False
        )
        
        try:
            self.channel.start_consuming()
        except KeyboardInterrupt:
            self.logger.info("Stopping worker...")
            self.channel.stop_consuming()
            self.connection.close()
```

#### 3. Specific Workers:

```python
# email_worker.py
from worker_base import WorkerBase
import time

class EmailWorker(WorkerBase):
    def __init__(self):
        super().__init__(
            worker_type='email',
            queue_name='email_queue',
            routing_keys=['task.email', 'task.email.*']
        )
    
    def process_task(self, task):
        # Simulate email sending
        recipient = task['data'].get('recipient')
        subject = task['data'].get('subject')
        
        self.logger.info(f"Sending email to {recipient}: {subject}")
        time.sleep(2)  # Simulate delay
        
        return {
            'status': 'sent',
            'recipient': recipient
        }

if __name__ == '__main__':
    worker = EmailWorker()
    worker.start()


# image_worker.py
from worker_base import WorkerBase
import time
import random

class ImageWorker(WorkerBase):
    def __init__(self):
        super().__init__(
            worker_type='image',
            queue_name='image_queue',
            routing_keys=['task.image', 'task.image.*']
        )
    
    def process_task(self, task):
        # Simulate image processing
        image_url = task['data'].get('url')
        operation = task['data'].get('operation', 'resize')
        
        self.logger.info(f"Processing image {image_url}: {operation}")
        time.sleep(random.randint(3, 7))  # Simulate variable processing time
        
        return {
            'status': 'processed',
            'url': image_url,
            'operation': operation
        }

if __name__ == '__main__':
    worker = ImageWorker()
    worker.start()


# analytics_worker.py
from worker_base import WorkerBase
import time

class AnalyticsWorker(WorkerBase):
    def __init__(self):
        super().__init__(
            worker_type='analytics',
            queue_name='analytics_queue',
            routing_keys=['task.analytics', 'task.analytics.*']
        )
    
    def process_task(self, task):
        # Simulate analytics computation
        metric_type = task['data'].get('metric')
        
        self.logger.info(f"Computing analytics: {metric_type}")
        time.sleep(4)
        
        return {
            'status': 'computed',
            'metric': metric_type,
            'value': 42.5  # Mock result
        }

if __name__ == '__main__':
    worker = AnalyticsWorker()
    worker.start()
```

#### 4. Audit Logger:

```python
# audit_logger.py
import pika
import json
import logging
from datetime import datetime

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger('audit')

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.exchange_declare(
    exchange='audit',
    exchange_type='fanout',
    durable=True
)

result = channel.queue_declare(queue='audit_log', durable=True)
channel.queue_bind(exchange='audit', queue='audit_log')

def callback(ch, method, properties, body):
    audit = json.loads(body)
    
    timestamp = datetime.fromtimestamp(audit['timestamp']).isoformat()
    
    logger.info(
        f"[{timestamp}] Task {audit['task_id']} "
        f"processed by {audit['worker_type']} "
        f"in {audit['duration']:.2f}s"
    )
    
    # Could write to database or file here
    
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_consume(
    queue='audit_log',
    on_message_callback=callback,
    auto_ack=False
)

logger.info("Audit logger started...")
channel.start_consuming()
```

#### 5. Dead Letter Monitor:

```python
# dlq_monitor.py
import pika
import json
import logging

logging.basicConfig(
    level=logging.WARNING,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger('dlq')

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.queue_declare(queue='dead_letters', durable=True)

def callback(ch, method, properties, body):
    try:
        task = json.loads(body)
        logger.warning(
            f"Dead letter detected: Task {task['task_id']} "
            f"of type {task['type']}"
        )
        
        # Could implement retry logic or alerting here
        
    except Exception as e:
        logger.error(f"Error processing dead letter: {e}")
    
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_consume(
    queue='dead_letters',
    on_message_callback=callback,
    auto_ack=False
)

logger.info("Dead letter monitor started...")
channel.start_consuming()
```

#### 6. Docker Compose для всей системы:

```yaml
# docker-compose-final.yml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3-management
    hostname: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
      - "15692:15692"
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=admin
    command: >
      bash -c "
        rabbitmq-plugins enable rabbitmq_prometheus &&
        rabbitmq-server
      "
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 30s
      timeout: 10s
      retries: 5

  api:
    build: .
    command: python api_gateway.py
    ports:
      - "5000:5000"
    depends_on:
      - rabbitmq
    environment:
      - RABBITMQ_HOST=rabbitmq

  email_worker:
    build: .
    command: python email_worker.py
    depends_on:
      - rabbitmq
    deploy:
      replicas: 2

  image_worker:
    build: .
    command: python image_worker.py
    depends_on:
      - rabbitmq
    deploy:
      replicas: 3

  analytics_worker:
    build: .
    command: python analytics_worker.py
    depends_on:
      - rabbitmq

  audit_logger:
    build: .
    command: python audit_logger.py
    depends_on:
      - rabbitmq

  dlq_monitor:
    build: .
    command: python dlq_monitor.py
    depends_on:
      - rabbitmq

  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
```

#### 7. Load Test:

```python
# load_test.py
import requests
import time
import random
from concurrent.futures import ThreadPoolExecutor

API_URL = "http://localhost:5000/api/task"

task_types = ['email', 'image', 'analytics']

def send_task():
    task_type = random.choice(task_types)
    
    payload = {
        'type': task_type,
        'data': {
            'recipient': 'user@example.com' if task_type == 'email' else None,
            'url': 'http://example.com/image.jpg' if task_type == 'image' else None,
            'metric': 'conversion_rate' if task_type == 'analytics' else None
        },
        'priority': random.randint(1, 10)
    }
    
    try:
        response = requests.post(API_URL, json=payload, timeout=5)
        if response.status_code == 202:
            print(f"✓ Task {task_type} queued: {response.json()['task_id']}")
        else:
            print(f"✗ Failed: {response.status_code}")
    except Exception as e:
        print(f"✗ Error: {e}")

def run_load_test(num_requests=100, num_threads=10):
    print(f"Starting load test: {num_requests} requests, {num_threads} threads")
    print("-" * 60)
    
    start_time = time.time()
    
    with ThreadPoolExecutor(max_workers=num_threads) as executor:
        futures = [executor.submit(send_task) for _ in range(num_requests)]
        
        for future in futures:
            future.result()
    
    duration = time.time() - start_time
    
    print("-" * 60)
    print(f"Completed in {duration:.2f}s")
    print(f"Throughput: {num_requests/duration:.2f} requests/sec")

if __name__ == '__main__':
    run_load_test(num_requests=200, num_threads=20)
```

#### 8. Запуск финального проекта:

```bash
# 1. Запусти все сервисы
docker-compose -f docker-compose-final.yml up -d

# 2. Проверь здоровье системы
curl http://localhost:5000/health

# 3. В разных терминалах запусти workers
python email_worker.py &
python image_worker.py &
python analytics_worker.py &
python audit_logger.py &
python dlq_monitor.py &

# 4. Отправь тестовые задачи
curl -X POST http://localhost:5000/api/task \
  -H "Content-Type: application/json" \
  -d '{"type":"email","data":{"recipient":"test@example.com","subject":"Hello"}}'

# 5. Запусти нагрузочный тест
python load_test.py

# 6. Мониторинг в Web UI
# RabbitMQ: http://localhost:15672
# Prometheus: http://localhost:9090
# Grafana: http://localhost:3000

# 7. Проверь метрики
watch -n 1 'docker exec rabbitmq rabbitmqctl list_queues name messages consumers'
```

---

## 🎓 Заключение

### Что ты изучил:

✅ **Основы RabbitMQ**: Exchanges, Queues, Bindings, Routing  
✅ **Надежность**: Durability, Acknowledgments, Publisher Confirms  
✅ **High Availability**: Clustering, Quorum Queues  
✅ **Мониторинг**: Prometheus, Grafana, Health Checks  
✅ **Security**: RBAC, Virtual Hosts, Permissions  
✅ **Advanced Patterns**: RPC, Event-Driven, SAGA, Circuit Breaker  
✅ **Production**: Deployment, Troubleshooting, Performance Tuning  

### Следующие шаги:

1. **Практика**: Реализуй реальный проект с RabbitMQ
2. **Изучи**: Kafka для stream processing
3. **Освой**: Service Mesh (Istio, Linkerd)
4. **Внедри**: В production с полным мониторингом
