# Zabbix Refresh: Ежегодный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые концепции Zabbix за 3-4 часа практики и узнать 2-3 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная задача, которую нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Предварительные требования:**
- Docker установлен и настроен
- Базовые знания Linux CLI
- Понимание сетевых протоколов (TCP/IP, HTTP)
- Опыт работы с системным мониторингом (желательно)

---

## Модуль 1: Базовая архитектура и установка (25 минут)

### 🎯 Напоминалка

**Zabbix - что это:**
```
Zabbix
├── Enterprise-class Monitoring Solution
├── Open Source (GPL v2)
├── Distributed Monitoring System
├── Агентный и безагентный мониторинг
└── Web-based интерфейс
```

**Основные компоненты:**
```bash
# Архитектура Zabbix
Zabbix Server
├── Ядро системы мониторинга
├── Сбор данных от агентов
├── Вычисление триггеров
├── Отправка уведомлений
└── Хранение конфигурации

Zabbix Agent (1 и 2)
├── Agent 1: C, классический
├── Agent 2: Go, современный
├── Активный режим (push)
├── Пассивный режим (pull)
└── Плагины и UserParameters

Zabbix Proxy
├── Сбор данных в удаленных локациях
├── Снижение нагрузки на Server
├── Работа при потере связи
└── Передача данных на Server

Web Interface (Frontend)
├── PHP-based интерфейс
├── Настройка мониторинга
├── Визуализация данных
├── Дашборды и графики
└── Управление пользователями

Database (Backend)
├── PostgreSQL (рекомендуется)
├── MySQL/MariaDB
├── Oracle (Enterprise)
└── TimescaleDB (для больших данных)
```

**Основные возможности:**
```bash
# Мониторинг
- Сервера и сетевое оборудование
- Приложения и сервисы
- Виртуальные машины и контейнеры
- Cloud ресурсы (AWS, Azure, GCP)
- SNMP устройства
- IPMI мониторинг
- JMX мониторинг

# Сбор данных
- Активная проверка (Server -> Agent)
- Пассивная проверка (Agent -> Server)
- SNMP traps
- Simple checks (ping, port check)
- Calculated items
- Dependent items

# Визуализация
- Графики (Graph)
- Дашборды (Dashboard)
- Карты сети (Maps)
- Экраны (Screens - legacy)
- SLA отчеты

# Оповещения
- Email, SMS, Jabber
- Webhook (Slack, Telegram, etc.)
- Scripts
- Escalations
- Maintenance periods
```

**Установка Docker-based:**
```bash
# Создай структуру директорий
mkdir -p ~/zabbix-docker/{mysql,zabbix}
cd ~/zabbix-docker

# docker-compose.yml для базовой установки
cat > docker-compose.yml <<'EOF'
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: zabbix-mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_pass
    volumes:
      - ./mysql:/var/lib/mysql
    command: 
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_bin
      - --default-authentication-plugin=mysql_native_password
    networks:
      - zabbix-net

  zabbix-server:
    image: zabbix/zabbix-server-mysql:alpine-6.4-latest
    container_name: zabbix-server
    environment:
      DB_SERVER_HOST: mysql
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_pass
      ZBX_CACHESIZE: 256M
      ZBX_HISTORYCACHESIZE: 128M
      ZBX_TRENDCACHESIZE: 128M
    ports:
      - "10051:10051"
    depends_on:
      - mysql
    volumes:
      - ./zabbix/alertscripts:/usr/lib/zabbix/alertscripts
      - ./zabbix/externalscripts:/usr/lib/zabbix/externalscripts
    networks:
      - zabbix-net

  zabbix-web:
    image: zabbix/zabbix-web-nginx-mysql:alpine-6.4-latest
    container_name: zabbix-web
    environment:
      DB_SERVER_HOST: mysql
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_pass
      ZBX_SERVER_HOST: zabbix-server
      PHP_TZ: Europe/Moscow
    ports:
      - "8080:8080"
    depends_on:
      - mysql
      - zabbix-server
    networks:
      - zabbix-net

  zabbix-agent:
    image: zabbix/zabbix-agent2:alpine-6.4-latest
    container_name: zabbix-agent
    environment:
      ZBX_HOSTNAME: "Zabbix server"
      ZBX_SERVER_HOST: zabbix-server
      ZBX_SERVER_PORT: 10051
      ZBX_PASSIVE_ALLOW: "true"
      ZBX_ACTIVE_ALLOW: "true"
    privileged: true
    networks:
      - zabbix-net

volumes:
  mysql_data:
  zabbix_data:

networks:
  zabbix-net:
    driver: bridge
EOF

# Запуск
docker-compose up -d

# Проверка статуса
docker-compose ps

# Логи
docker-compose logs -f zabbix-server

# Ожидай инициализации (2-3 минуты)
# Web интерфейс: http://localhost:8080
# Логин: Admin (с большой буквы!)
# Пароль: zabbix
```

**Первоначальная настройка:**
```bash
# После входа в веб-интерфейс:
# 1. Смени пароль администратора
#    User settings -> Change password

# 2. Настрой локаль и часовой пояс
#    User settings -> Language/Time zone

# 3. Проверь работу сервера
#    Monitoring -> Dashboard

# 4. Проверь агента
#    Configuration -> Hosts -> Zabbix server
#    Availability: ZBX должен быть зеленым

# CLI проверки
docker exec -it zabbix-server zabbix_server -V
docker exec -it zabbix-agent zabbix_agent2 -V

# Проверка подключения к БД
docker exec -it zabbix-mysql mysql -uzabbix -pzabbix_pass -e "USE zabbix; SHOW TABLES;"
```

**Основные концепции:**
```bash
# Host - устройство для мониторинга
# Template - набор items, triggers, graphs
# Item - метрика для сбора
# Trigger - условие для алерта
# Action - действие при срабатывании триггера
# User - пользователь системы
# Host Group - логическая группа хостов
# Media Type - канал оповещений

# Иерархия наследования
Template
  ├── Items (метрики)
  ├── Triggers (условия)
  ├── Discovery Rules (автообнаружение)
  ├── Graphs (графики)
  └── Macros (переменные)
      ↓
Host (применяет Template)
  ├── Наследует все элементы
  ├── Может переопределить macros
  └── Может добавить свои items
```

### 💻 Задание

Подготовь тестовое окружение и базовый мониторинг:

1. **Запусти Zabbix stack:**
```bash
# Используй docker-compose.yml выше
docker-compose up -d

# Жди инициализации
sleep 60

# Проверь все контейнеры
docker-compose ps
# Все должны быть в статусе "Up"
```

2. **Первый вход и базовая настройка:**
```bash
# Открой http://localhost:8080
# Login: Admin
# Password: zabbix

# В интерфейсе:
# 1. User settings (иконка пользователя справа вверху)
# 2. Change password -> установи новый пароль
# 3. Language: English (US)
# 4. Time zone: выбери свой
# 5. Theme: можешь выбрать Dark
```

3. **Проверь работу встроенного мониторинга:**
```bash
# Monitoring -> Hosts
# Должен быть хост "Zabbix server"
# Availability - ZBX должен гореть зеленым

# Кликни на "Zabbix server"
# Latest data - посмотри собранные метрики
# Должны быть данные по CPU, memory, disk

# Monitoring -> Dashboard
# Должен быть дефолтный dashboard с виджетами
```

4. **Создай простой host для мониторинга:**
```bash
# Запусти тестовый веб-сервер
docker run -d --name test-nginx \
  --network zabbix-docker_zabbix-net \
  -p 8081:80 \
  nginx:alpine

# В Zabbix Web:
# Configuration -> Hosts -> Create host
# Host name: test-nginx
# Groups: Linux servers (create new if needed)
# Interfaces: Agent
#   - IP: test-nginx
#   - Port: 10050 (пока без агента)
# Templates: пока не добавляем
# Save

# Добавь Simple check
# Configuration -> Hosts -> test-nginx -> Items
# Create item:
#   - Name: HTTP service check
#   - Type: Simple check
#   - Key: net.tcp.service[http]
#   - Host interface: выбери тот что создал
#   - Type of information: Numeric (unsigned)
#   - Update interval: 30s
#   - Save
```

5. **Проверь сбор данных:**
```bash
# Monitoring -> Hosts -> test-nginx -> Latest data
# Через 30-60 секунд должны появиться данные

# Проверь через CLI
docker exec -it zabbix-server zabbix_get -s test-nginx -k agent.ping
# Если агент не установлен, будет ошибка - это ок

# Simple check должен работать
# Monitoring -> Latest data -> filter by host: test-nginx
```

6. **Создай первый триггер:**
```bash
# Configuration -> Hosts -> test-nginx -> Triggers
# Create trigger:
#   - Name: HTTP service is down
#   - Severity: High
#   - Expression: 
#     {test-nginx:net.tcp.service[http].last()}=0
#   - Description: HTTP service is not responding
#   - Save

# Проверь триггер - останови nginx
docker stop test-nginx

# Подожди 30 секунд
# Monitoring -> Problems
# Должен появиться алерт "HTTP service is down"

# Запусти обратно
docker start test-nginx
# Проблема должна автоматически закрыться
```

### 🚀 Бонус (новое)

**1. Установи Zabbix Agent 2 на хост-машину:**

```bash
# Ubuntu/Debian
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update
sudo apt install zabbix-agent2

# CentOS/RHEL
sudo rpm -Uvh https://repo.zabbix.com/zabbix/6.4/rhel/8/x86_64/zabbix-release-6.4-1.el8.noarch.rpm
sudo yum install zabbix-agent2

# Настройка
sudo vim /etc/zabbix/zabbix_agent2.conf

# Измени:
Server=127.0.0.1          # IP Zabbix Server
ServerActive=127.0.0.1    # Для активных проверок
Hostname=MyLinuxHost      # Имя хоста

# Запуск
sudo systemctl enable zabbix-agent2
sudo systemctl start zabbix-agent2
sudo systemctl status zabbix-agent2

# Проверка
sudo tail -f /var/log/zabbix/zabbix_agent2.log
```

**2. Используй Zabbix API для автоматизации:**

```python
#!/usr/bin/env python3
import requests
import json

# Zabbix API credentials
ZABBIX_URL = "http://localhost:8080/api_jsonrpc.php"
ZABBIX_USER = "Admin"
ZABBIX_PASSWORD = "your_password"

class ZabbixAPI:
    def __init__(self, url, user, password):
        self.url = url
        self.auth_token = None
        self.authenticate(user, password)
    
    def call(self, method, params):
        headers = {'Content-Type': 'application/json'}
        payload = {
            "jsonrpc": "2.0",
            "method": method,
            "params": params,
            "id": 1
        }
        
        if self.auth_token:
            payload["auth"] = self.auth_token
        
        response = requests.post(
            self.url, 
            data=json.dumps(payload), 
            headers=headers
        )
        
        result = response.json()
        
        if 'error' in result:
            raise Exception(f"API Error: {result['error']}")
        
        return result.get('result')
    
    def authenticate(self, user, password):
        result = self.call('user.login', {
            'user': user,
            'password': password
        })
        self.auth_token = result
        print(f"Authenticated successfully")
    
    def get_hosts(self):
        return self.call('host.get', {
            'output': ['hostid', 'host', 'name'],
            'selectInterfaces': ['ip']
        })
    
    def create_host(self, hostname, ip, groupid, templateid):
        return self.call('host.create', {
            'host': hostname,
            'interfaces': [{
                'type': 1,
                'main': 1,
                'useip': 1,
                'ip': ip,
                'dns': '',
                'port': '10050'
            }],
            'groups': [{'groupid': groupid}],
            'templates': [{'templateid': templateid}]
        })

# Использование
zapi = ZabbixAPI(ZABBIX_URL, ZABBIX_USER, ZABBIX_PASSWORD)

# Получить все хосты
hosts = zapi.get_hosts()
print("\nCurrent hosts:")
for host in hosts:
    print(f"- {host['name']} ({host['host']}) - {host['interfaces'][0]['ip']}")

# Создать новый хост (раскомментируй если нужно)
# new_host = zapi.create_host(
#     hostname='new-server',
#     ip='192.168.1.100',
#     groupid='2',  # Linux servers
#     templateid='10001'  # Linux by Zabbix agent
# )
# print(f"\nCreated new host: {new_host}")
```

**3. Настрой Grafana интеграцию:**

```bash
# Запусти Grafana
docker run -d \
  --name=grafana \
  --network zabbix-docker_zabbix-net \
  -p 3000:3000 \
  grafana/grafana

# Открой http://localhost:3000
# Login: admin
# Password: admin

# Установи Zabbix plugin
docker exec -it grafana grafana-cli plugins install alexanderzobnin-zabbix-app
docker restart grafana

# В Grafana:
# Configuration -> Data sources -> Add data source
# Выбери Zabbix
# URL: http://zabbix-web:8080/api_jsonrpc.php
# Username: Admin
# Password: твой пароль
# Save & Test

# Создай dashboard с Zabbix метриками
# Create -> Dashboard -> Add new panel
# Data source: Zabbix
# Выбери метрики для визуализации
```

---

## Модуль 2: Items и Data Collection (35 минут)

### 🎯 Напоминалка

**Типы Items:**

```bash
# Zabbix agent - данные от агента
key: system.cpu.load[percpu,avg1]
key: vm.memory.size[available]
key: vfs.fs.size[/,used]

# Simple checks - без агента
key: net.tcp.service[http]
key: net.tcp.service[ssh]
key: icmpping[,3,,,500]

# SNMP - для сетевого оборудования
OID: .1.3.6.1.2.1.1.3.0  # System uptime
OID: .1.3.6.1.2.1.2.2.1.10.1  # Interface incoming traffic

# IPMI - датчики сервера
sensor: Temperature
sensor: Fan Speed
sensor: Voltage

# JMX - Java приложения
jmx: java.lang:type=Memory
jmx: jmx["java.lang:type=Threading","ThreadCount"]

# Database monitor
sql: SELECT COUNT(*) FROM users
sql: SELECT pg_database_size('mydb')

# HTTP agent - REST API
url: https://api.example.com/status
headers: Authorization: Bearer {TOKEN}

# Script - кастомные скрипты
script: /usr/local/bin/check_custom.sh

# Calculated - вычисляемые
formula: last("item1") + last("item2")
formula: avg("item1",5m)

# Dependent items - зависимые
master_item: получает JSON
dependent: парсит JSON поля
```

**Item Value Types:**

```bash
# Numeric (float) - 64-bit число с плавающей точкой
type: 0
examples: CPU load, temperature, speed

# Character - строка до 255 символов
type: 1
examples: version, status, short text

# Log - лог файлы
type: 2
examples: /var/log/syslog, application logs

# Numeric (unsigned) - 64-bit беззнаковое
type: 3
examples: counters, rates, packet count

# Text - текст любой длины
type: 4
examples: configuration dumps, long outputs

# Update intervals
30s - стандартный интервал
1m  - минута
1h  - час
1d  - день
{$MACRO} - через макрос
```

**User Parameters (Agent 1):**

```bash
# /etc/zabbix/zabbix_agent2.conf

# Простой параметр
UserParameter=custom.check,echo "OK"

# С параметром
UserParameter=custom.count[*],wc -l $1

# Сложный скрипт
UserParameter=mysql.ping,mysqladmin -uroot ping | grep -c alive

# С несколькими параметрами
UserParameter=custom.port[*],netstat -an | grep -c "$1.*$2"

# Использование:
# Item key: custom.check
# Item key: custom.count[/var/log/syslog]
# Item key: custom.port[LISTEN,80]
```

**Agent 2 Plugins:**

```bash
# /etc/zabbix/zabbix_agent2.conf

# Встроенные плагины
Plugins.SystemRun.LogRemoteCommands=1

# Кастомный плагин
# Plugins.Docker.Endpoint=unix:///var/run/docker.sock

# Проверить доступные плагины
zabbix_agent2 -p

# Примеры ключей встроенных плагинов:
# docker.containers[state,running]
# docker.container_info[container_id,Memory]
# postgres.ping[]
# redis.ping[]
```

**Preprocessing (предобработка данных):**

```bash
# Regular expression
regex: \bError\b
output: \1

# JSONPath
$.data.temperature
$.items[0].value

# XML XPath
/root/item[@id='1']/value

# Arithmetic
multiply: 1024  # bytes to KB
custom multiplier: 0.001  # ms to seconds

# Change per second
Δ per second

# Discard unchanged
с heartbeat

# Prometheus pattern
metric_name{label="value"}

# JavaScript
return JSON.parse(value).result;

# Trim spaces, Convert to decimal, etc.
```

**Discovery Rules (Low-Level Discovery - LLD):**

```bash
# Автоматическое обнаружение:
# - Файловых систем
# - Сетевых интерфейсов
# - SNMP OID
# - Кастомных объектов

# Пример LLD правила
Key: vfs.fs.discovery
Returns: JSON с массивом {#FSNAME}, {#FSTYPE}

# Item prototype:
Key: vfs.fs.size[{#FSNAME},used]
Key: vfs.fs.size[{#FSNAME},pfree]

# Trigger prototype:
Expression: {HOST:vfs.fs.size[{#FSNAME},pfree].last()}<10

# Кастомный LLD
#!/bin/bash
echo '{'
echo '  "data":['
first=1
for db in $(mysql -e "SHOW DATABASES;" -s --skip-column-names); do
  [ $first -eq 0 ] && echo ','
  echo -n "    {\"{#DBNAME}\":\"$db\"}"
  first=0
done
echo '  ]'
echo '}'
```

### 💻 Задание

Настрой комплексный мониторинг с различными типами items:

1. **Установи Zabbix Agent 2 в Docker контейнер:**

```bash
# Создай Dockerfile для тестового хоста с агентом
cat > Dockerfile.monitored <<'EOF'
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    wget \
    nginx \
    mysql-client \
    && rm -rf /var/lib/apt/lists/*

# Установка Zabbix Agent 2
RUN wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb && \
    dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb && \
    apt-get update && \
    apt-get install -y zabbix-agent2 && \
    rm -rf /var/lib/apt/lists/*

# Конфигурация агента
RUN echo "Server=zabbix-server" > /etc/zabbix/zabbix_agent2.conf && \
    echo "ServerActive=zabbix-server" >> /etc/zabbix/zabbix_agent2.conf && \
    echo "Hostname=monitored-host" >> /etc/zabbix/zabbix_agent2.conf && \
    echo "LogFile=/var/log/zabbix/zabbix_agent2.log" >> /etc/zabbix/zabbix_agent2.conf

# Запуск скрипт
COPY start.sh /start.sh
RUN chmod +x /start.sh

CMD ["/start.sh"]
EOF

# Создай start script
cat > start.sh <<'EOF'
#!/bin/bash
service zabbix-agent2 start
service nginx start
tail -f /var/log/zabbix/zabbix_agent2.log
EOF

# Собери образ
docker build -f Dockerfile.monitored -t monitored-host .

# Запусти контейнер
docker run -d --name monitored-host \
  --network zabbix-docker_zabbix-net \
  monitored-host

# Проверь агент
docker exec monitored-host zabbix_agent2 -t agent.ping
```

2. **Создай Host с Agent мониторингом:**

```bash
# В Zabbix Web:
# Configuration -> Hosts -> Create host
# 
# Host name: monitored-host
# Groups: Linux servers
# Interfaces: 
#   Agent:
#     IP: monitored-host
#     Port: 10050
# Templates: Linux by Zabbix agent active
# Save

# Жди 1-2 минуты
# Monitoring -> Hosts
# Availability ZBX должен стать зеленым

# Проверь Latest data
# Monitoring -> Latest data
# Host: monitored-host
# Должны появиться метрики CPU, Memory, Disk
```

3. **Создай кастомные UserParameters:**

```bash
# Заходим в контейнер
docker exec -it monitored-host bash

# Создаем кастомный скрипт
cat > /usr/local/bin/check_nginx_connections.sh <<'SCRIPT'
#!/bin/bash
# Проверка количества активных соединений nginx
ps aux | grep nginx | grep -v grep | wc -l
SCRIPT

chmod +x /usr/local/bin/check_nginx_connections.sh

# Добавляем UserParameter
echo "UserParameter=nginx.connections,/usr/local/bin/check_nginx_connections.sh" >> /etc/zabbix/zabbix_agent2.conf

# Перезапускаем агент
service zabbix-agent2 restart

# Тестируем
zabbix_agent2 -t nginx.connections

# Выходим из контейнера
exit
```

Продолжаю создание курса...

<function_calls>
<invoke name="artifacts">
<parameter name="command">update