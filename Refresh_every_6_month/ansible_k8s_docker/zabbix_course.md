# Zabbix Refresh: Ежегодный/Полугодовой курс для DevOps

**Цель:** Освежить в памяти ключевые концепции Zabbix за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная задача, которую нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Предварительные требования:**
- Доступ к Zabbix серверу (6.0+) или возможность развернуть через Docker
- Базовые знания Linux
- SSH доступ к тестовым хостам

---

## Модуль 1: Архитектура и базовая настройка (20 минут)

### 🎯 Напоминалка

**Архитектура Zabbix:**
```
Zabbix Server
├── Database (MySQL/PostgreSQL)
├── Web Frontend (Apache/Nginx + PHP)
└── Zabbix Server Process
    ├── Pollers (сбор данных)
    ├── Trappers (прием данных)
    ├── Alerters (отправка уведомлений)
    ├── Timers (триггеры времени)
    └── Escalators (эскалация проблем)

Zabbix Agent (на мониторимых хостах)
├── Active checks (агент инициирует)
└── Passive checks (сервер опрашивает)

Zabbix Proxy (опционально)
└── Промежуточный сборщик для удаленных локаций
```

**Основные компоненты:**
- **Host** - устройство/сервер для мониторинга
- **Item** - метрика для сбора (CPU, memory, disk, etc.)
- **Trigger** - условие для генерации проблемы
- **Action** - что делать при срабатывании триггера
- **Template** - набор items/triggers для переиспользования
- **Host Group** - логическая группировка хостов

**Типы мониторинга:**
```
Zabbix Agent    # Активный/пассивный агент
SNMP            # Сетевое оборудование
IPMI            # Аппаратный мониторинг серверов
JMX             # Java приложения
HTTP Agent      # REST API мониторинг
Database        # Прямые запросы к БД
SSH/Telnet      # Выполнение команд
```

**Быстрая установка через Docker:**
```bash
# Docker Compose для Zabbix 6.4
cat > docker-compose.yml <<EOF
version: '3.8'
services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix_pwd
      POSTGRES_DB: zabbix
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - zabbix-net

  zabbix-server:
    image: zabbix/zabbix-server-pgsql:alpine-6.4-latest
    environment:
      DB_SERVER_HOST: postgres
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix_pwd
      POSTGRES_DB: zabbix
      ZBX_ENABLE_SNMP_TRAPS: "true"
    depends_on:
      - postgres
    ports:
      - "10051:10051"
    volumes:
      - zabbix-server-data:/var/lib/zabbix
    networks:
      - zabbix-net

  zabbix-web:
    image: zabbix/zabbix-web-nginx-pgsql:alpine-6.4-latest
    environment:
      DB_SERVER_HOST: postgres
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix_pwd
      POSTGRES_DB: zabbix
      ZBX_SERVER_HOST: zabbix-server
      PHP_TZ: Asia/Almaty
    depends_on:
      - postgres
      - zabbix-server
    ports:
      - "8080:8080"
    networks:
      - zabbix-net

  zabbix-agent:
    image: zabbix/zabbix-agent2:alpine-6.4-latest
    environment:
      ZBX_HOSTNAME: "Zabbix server"
      ZBX_SERVER_HOST: zabbix-server
    privileged: true
    pid: "host"
    networks:
      - zabbix-net

volumes:
  postgres-data:
  zabbix-server-data:

networks:
  zabbix-net:
    driver: bridge
EOF

# Запуск
docker-compose up -d

# Проверка
docker-compose ps
```

**Доступ к веб-интерфейсу:**
```
URL: http://localhost:8080
Login: Admin
Password: zabbix
```

**Установка Zabbix Agent на Linux:**
```bash
# Ubuntu/Debian
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
apt update
apt install zabbix-agent2

# CentOS/RHEL
rpm -Uvh https://repo.zabbix.com/zabbix/6.4/rhel/8/x86_64/zabbix-release-6.4-1.el8.noarch.rpm
dnf clean all
dnf install zabbix-agent2

# Конфигурация
vi /etc/zabbix/zabbix_agent2.conf
# Server=<IP_ZABBIX_SERVER>
# ServerActive=<IP_ZABBIX_SERVER>
# Hostname=<HOSTNAME>

# Запуск
systemctl enable zabbix-agent2
systemctl start zabbix-agent2
systemctl status zabbix-agent2
```

**Основные файлы конфигурации:**
```bash
# Сервер
/etc/zabbix/zabbix_server.conf

# Агент
/etc/zabbix/zabbix_agent2.conf
/etc/zabbix/zabbix_agent2.d/  # Дополнительные конфиги

# Веб
/etc/zabbix/web/zabbix.conf.php

# Логи
/var/log/zabbix/zabbix_server.log
/var/log/zabbix/zabbix_agent2.log
```

**Проверка связи:**
```bash
# С сервера проверить агента
zabbix_get -s <agent_host> -k agent.ping

# Проверить версию агента
zabbix_get -s <agent_host> -k agent.version

# Тест конфигурации
zabbix_agent2 -t agent.hostname

# Просмотр доступных метрик
zabbix_agent2 -p
```

### 💻 Задание

Разверни тестовое окружение:

1. **Запусти Zabbix через Docker Compose**
   ```bash
   docker-compose up -d
   docker-compose ps
   docker-compose logs -f zabbix-server
   ```

2. **Войди в веб-интерфейс**
   - Открой http://localhost:8080
   - Логин: Admin, Пароль: zabbix
   - Смени пароль на безопасный

3. **Добавь первый хост**
   - Configuration → Hosts → Create host
   - Host name: TestServer
   - Templates: Linux by Zabbix agent
   - Agent: DNS name или IP адрес агента
   - Port: 10050

4. **Проверь работу мониторинга**
   - Monitoring → Latest data
   - Выбери TestServer
   - Проверь поступление данных (CPU, Memory, Disk)

5. **Создай простой триггер**
   - Configuration → Hosts → TestServer → Triggers
   - Name: High CPU usage
   - Expression: `avg(/TestServer/system.cpu.util,5m)>80`
   - Severity: Warning

### 🚀 Бонус (новое)

**Настрой Zabbix Agent 2 с плагинами:**

Zabbix Agent 2 поддерживает плагины для расширенного мониторинга:

```bash
# /etc/zabbix/zabbix_agent2.conf
Plugins.SystemRun.LogRemoteCommands=1

# Docker monitoring
Plugins.Docker.Endpoint=unix:///var/run/docker.sock

# Redis monitoring
Plugins.Redis.Uri=tcp://localhost:6379

# PostgreSQL monitoring
Plugins.PostgreSQL.Uri=tcp://localhost:5432

# Перезапуск
systemctl restart zabbix-agent2

# Проверка плагинов
zabbix_agent2 -t docker.info
zabbix_agent2 -t redis.ping
```

**Используй Zabbix API для автоматизации:**

```bash
# Получить authentication token
curl -X POST http://localhost:8080/api_jsonrpc.php \
  -H "Content-Type: application/json-rpc" \
  -d '{
    "jsonrpc": "2.0",
    "method": "user.login",
    "params": {
      "username": "Admin",
      "password": "zabbix"
    },
    "id": 1
  }'

# Получить список хостов
curl -X POST http://localhost:8080/api_jsonrpc.php \
  -H "Content-Type: application/json-rpc" \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.get",
    "params": {
      "output": ["hostid", "host"]
    },
    "auth": "<YOUR_AUTH_TOKEN>",
    "id": 1
  }'
```

---

## Модуль 2: Items и сбор данных (25 минут)

### 🎯 Напоминалка

**Типы Items:**
```
Zabbix agent       # Сбор через агента
Simple check       # Ping, порты без агента
SNMP agent         # SNMP мониторинг
Zabbix internal    # Внутренние метрики Zabbix
Zabbix trapper     # Прием данных от внешних источников
External check     # Выполнение скриптов на сервере
Database monitor   # SQL запросы
HTTP agent         # REST API, веб-мониторинг
Calculated         # Вычисляемые на основе других items
Dependent          # Зависимые от других items
```

**Основные ключи агента:**
```bash
# Система
system.cpu.util[,idle]           # CPU утилизация
system.cpu.load[percpu,avg1]     # Load average
vm.memory.size[available]        # Доступная память
system.uptime                    # Время работы

# Диски
vfs.fs.size[/,free]              # Свободное место
vfs.fs.size[/,pfree]             # % свободного места
vfs.fs.inode[/,pfree]            # % свободных inode
vfs.file.exists[/path/file]      # Проверка существования

# Сеть
net.if.in[eth0]                  # Входящий трафик
net.if.out[eth0]                 # Исходящий трафик
net.tcp.listen[80]               # Проверка порта
net.tcp.service[http]            # Проверка сервиса

# Процессы
proc.num[nginx]                  # Количество процессов
proc.cpu.util[nginx]             # CPU процесса
proc.mem[nginx]                  # Память процесса

# Логи
log[/var/log/app.log,ERROR]      # Мониторинг логов
logrt["/var/log/app*.log",ERROR] # С regex pattern

# Пользовательские
system.run[command]              # Выполнение команды
```

**Update interval (интервал обновления):**
```
1s-23h59m59s     # Фиксированный интервал
1m;wd1-5,h9-18   # По расписанию (пн-пт, 9-18)
1h/1-7,00:00-24:00  # Гибкое расписание
```

**Value type (тип значения):**
```
Numeric (float)     # Числовое с плавающей точкой
Numeric (unsigned)  # Целое положительное
Character           # Строка (до 255 символов)
Log                 # Лог файлы с timestamp
Text                # Текст (до 64KB)
```

**Preprocessing (предобработка):**
```
Regular expression      # Извлечение через regex
JSONPath               # Парсинг JSON
XML XPath              # Парсинг XML
Custom multiplier      # Умножение на коэффициент
Change per second      # Delta в секунду
Discard unchanged      # Игнорировать дубли
Prometheus pattern     # Парсинг Prometheus метрик
JavaScript             # Кастомная обработка
```

**HTTP Agent item:**
```yaml
URL: https://api.example.com/metrics
Query fields:
  - name: token
    value: {$API_TOKEN}
Headers:
  - Authorization: Bearer {$API_TOKEN}
Request type: GET
Timeout: 3s
Follow redirects: Yes
Verify peer: Yes
Verify host: Yes

Preprocessing:
  - JSONPath: $.data.cpu_usage
  - Custom multiplier: 1
```

**Dependent items (зависимые):**
```
Master item: system.run[ps aux]
↓
Dependent item 1: Preprocessing → JSONPath → $.processes[?(@.name=="nginx")].count
Dependent item 2: Preprocessing → JSONPath → $.processes[?(@.name=="mysql")].cpu
Dependent item 3: Preprocessing → JSONPath → $.processes[?(@.name=="redis")].memory
```

**Calculated items:**
```
# Процент использованной памяти
100*(last(//vm.memory.size[total])-last(//vm.memory.size[available]))/last(//vm.memory.size[total])

# Суммарный трафик
last(//net.if.in[eth0])+last(//net.if.out[eth0])

# Средний CPU за 5 минут
avg(//system.cpu.util,5m)
```

### 💻 Задание

Настрой расширенный сбор метрик:

1. **Создай кастомный UserParameter**
   
   На хосте с агентом:
   ```bash
   # Создай скрипт
   cat > /usr/local/bin/check_service.sh <<'EOF'
   #!/bin/bash
   SERVICE=$1
   if systemctl is-active --quiet $SERVICE; then
     echo 1
   else
     echo 0
   fi
   EOF
   
   chmod +x /usr/local/bin/check_service.sh
   
   # Добавь UserParameter
   cat > /etc/zabbix/zabbix_agent2.d/custom.conf <<EOF
   UserParameter=custom.service.status[*],/usr/local/bin/check_service.sh $1
   UserParameter=custom.disk.usage,df -h / | awk 'NR==2 {print $5}' | sed 's/%//'
   EOF
   
   # Перезапусти агент
   systemctl restart zabbix-agent2
   
   # Тест
   zabbix_agent2 -t custom.service.status[nginx]
   ```

2. **Добавь item в Zabbix**
   - Configuration → Hosts → TestServer → Items → Create item
   - Name: Nginx service status
   - Type: Zabbix agent
   - Key: `custom.service.status[nginx]`
   - Type of information: Numeric (unsigned)
   - Update interval: 30s

3. **Настрой HTTP Agent для API мониторинга**
   
   Используем публичное API для теста:
   - Create item
   - Name: GitHub API Status
   - Type: HTTP agent
   - URL: `https://api.github.com/status`
   - Headers: `User-Agent: Zabbix`
   - Preprocessing:
     - JSONPath: `$.status`
     - Discard unchanged

4. **Создай Dependent items**
   
   Master item (один запрос к API):
   - Name: Docker Stats JSON
   - Type: Zabbix agent
   - Key: `system.run[docker stats --no-stream --format json]`
   
   Dependent items:
   - CPU Usage: Preprocessing → JSONPath → `$[0].CPUPerc`
   - Memory Usage: Preprocessing → JSONPath → `$[0].MemPerc`

5. **Настрой мониторинг логов**
   - Create item
   - Name: Application errors
   - Type: Zabbix agent (active)
   - Key: `log[/var/log/app/error.log,ERROR,,,skip]`
   - Type of information: Log
   - Update interval: 10s

### 🚀 Бонус (новое)

**Используй LLD (Low-Level Discovery) для автоматического обнаружения:**

1. **Создай Discovery Rule для дисков**
   ```bash
   # UserParameter для discovery
   cat > /etc/zabbix/zabbix_agent2.d/disk_discovery.conf <<'EOF'
   UserParameter=custom.disk.discovery,/usr/local/bin/disk_discovery.sh
   EOF
   
   # Скрипт discovery
   cat > /usr/local/bin/disk_discovery.sh <<'EOF'
   #!/bin/bash
   echo -n '{"data":['
   first=1
   df -h | awk 'NR>1 {print $1, $6}' | while read device mountpoint; do
     if [ $first -eq 1 ]; then
       first=0
     else
       echo -n ','
     fi
     echo -n '{"{#FSNAME}":"'$mountpoint'","{#FSDEVICE}":"'$device'"}'
   done
   echo ']}'
   EOF
   
   chmod +x /usr/local/bin/disk_discovery.sh
   systemctl restart zabbix-agent2
   ```

2. **В Zabbix создай Discovery Rule**
   - Configuration → Hosts → TestServer → Discovery rules
   - Name: Filesystem discovery
   - Type: Zabbix agent
   - Key: `custom.disk.discovery`
   - Update interval: 1h

3. **Создай Item prototype**
   - Name: `Free disk space on {#FSNAME}`
   - Key: `vfs.fs.size[{#FSNAME},free]`
   - Type: Numeric (unsigned)
   - Units: B

**Настрой Web Scenarios для мониторинга веб-приложений:**

```
Configuration → Hosts → TestServer → Web → Create web scenario

Name: Website availability
Application: Web monitoring
Agent: Mozilla/5.0
Update interval: 1m

Steps:
  Step 1: Homepage
    URL: https://example.com
    Required status codes: 200
    Timeout: 15s
    
  Step 2: Login page
    URL: https://example.com/login
    Required string: "Username"
    
  Step 3: API endpoint
    URL: https://api.example.com/health
    JSONPath: $.status = "healthy"
```

---

## Модуль 3: Triggers и проблемы (30 минут)

### 🎯 Напоминалка

**Severity (серьезность проблем):**
```
Not classified   # Не классифицировано
Information      # Информация
Warning          # Предупреждение
Average          # Средняя
High             # Высокая
Disaster         # Критическая
```

**Основные функции триггеров:**
```
last()           # Последнее значение
avg()            # Среднее за период
min()/max()      # Минимум/максимум
sum()            # Сумма
count()          # Количество
change()         # Изменение значения
diff()           # Разница между значениями
abschange()      # Абсолютное изменение
nodata()         # Нет данных
date()/time()    # Дата и время
now()            # Текущее время (Unix timestamp)
```

**Примеры триггеров:**
```bash
# CPU выше 80% последние 5 минут
avg(/Host/system.cpu.util,5m)>80

# Свободно менее 10% дискового пространства
last(/Host/vfs.fs.size[/,pfree])<10

# Нет данных более 3 минут
nodata(/Host/agent.ping,3m)=1

# Процесс не запущен
last(/Host/proc.num[nginx])<1

# Изменился размер файла
change(/Host/vfs.file.size[/etc/passwd])<>0

# HTTP код не 200
last(/Host/web.page.regexp[,,,200])=0

# Load average выше количества CPU
last(/Host/system.cpu.load[percpu,avg1])>last(/Host/system.cpu.num)

# Используется более 90% памяти
last(/Host/vm.memory.size[available])<last(/Host/vm.memory.size[total])*0.1

# Комбинированное условие (CPU и Memory)
avg(/Host/system.cpu.util,5m)>80 and last(/Host/vm.memory.size[pavailable])<20
```

**Expression с hysteresis (гистерезис):**
```bash
# Проблема: CPU > 80%
avg(/Host/system.cpu.util,5m)>80

# Восстановление: CPU < 70% (предотвращение флаппинга)
avg(/Host/system.cpu.util,5m)<70
```

**Trigger dependencies (зависимости):**
```
Trigger A: Host unavailable
↓ depends on
Trigger B: High CPU
Trigger C: High Memory

# Если A сработал, B и C не будут показывать проблемы
```

**Problem suppression (подавление проблем):**
```
Можно настроить подавление:
- По времени (например, во время обслуживания)
- По другим триггерам
- Вручную через Maintenance

Configuration → Maintenance → Create maintenance period
Name: Monthly maintenance
Active since: 2025-01-01 02:00
Period type: One time only
Duration: 4 hours
Hosts/Groups: Production servers
```

**Event tags и tag-based permissions:**
```yaml
Trigger expression: avg(/Host/system.cpu.util,5m)>80
Tags:
  - Application: Database
  - Environment: Production
  - Team: DBA

# Используется для:
# - Фильтрации в Dashboard
# - Routing уведомлений
# - RBAC доступа
```

### 💻 Задание

Создай систему триггеров для комплексного мониторинга:

1. **Базовый триггер с гистерезисом**
   
   Configuration → Hosts → TestServer → Triggers → Create trigger
   
   ```
   Name: High CPU usage on {HOST.NAME}
   Severity: Warning
   
   Expression:
   Problem: avg(/TestServer/system.cpu.util,5m)>80
   Recovery: avg(/TestServer/system.cpu.util,5m)<70
   
   Recovery expression: Expression
   PROBLEM event generation mode: Multiple
   OK event closes: All problems
   
   Tags:
   - component: cpu
   - severity: warning
   ```

2. **Триггер с множественными условиями**
   ```
   Name: Server overload
   Severity: High
   
   Expression:
   avg(/TestServer/system.cpu.util,5m)>90 and 
   last(/TestServer/vm.memory.size[pavailable])<10 and
   avg(/TestServer/system.cpu.load[percpu,avg5],5m)>2
   
   Description:
   CPU: {ITEM.LASTVALUE1}
   Memory available: {ITEM.LASTVALUE2}%
   Load average: {ITEM.LASTVALUE3}
   ```

3. **Триггер для мониторинга процесса**
   ```
   Name: Nginx is not running
   Severity: High
   
   Expression:
   last(/TestServer/proc.num[nginx])<1
   
   Manual close: Allowed
   
   Tags:
   - service: nginx
   - application: webserver
   ```

4. **Триггер с nodata**
   ```
   Name: No data from agent
   Severity: Average
   
   Expression:
   nodata(/TestServer/agent.ping,5m)=1
   
   Description:
   Agent not responding for 5 minutes
   Last seen: {ITEM.LASTCLOCK}
   ```

5. **Настрой Trigger dependencies**
   ```
   Parent trigger: Host {HOST.NAME} is unavailable
   Expression: max(/TestServer/agent.ping,#3)=0
   
   Child triggers (зависят от parent):
   - High CPU usage
   - High memory usage
   - Service down
   
   Configuration → Triggers → Dependencies → Add
   ```

6. **Создай Maintenance период**
   ```
   Configuration → Maintenance → Create
   
   Name: Weekend maintenance
   Type: With data collection
   Active since: This Saturday 02:00
   Maintenance period: Every week, Saturday, 02:00-06:00
   
   Hosts: TestServer
   
   Description: Regular weekend maintenance window
   ```

### 🚀 Бонус (новое)

**Используй новые возможности Trigger expressions (Zabbix 6.0+):**

1. **Функция `forecast()` для предсказания**
   ```
   Name: Disk will be full in 24h
   Severity: Warning
   
   Expression:
   timeleft(/TestServer/vfs.fs.size[/,free],1h,"linear")<24h
   
   # Предсказывает заполнение диска на основе тренда
   ```

2. **Использование макросов в триггерах**
   ```
   # Создай User macro
   Administration → General → Macros
   Macro: {$CPU.UTIL.MAX}
   Value: 80
   
   # В триггере
   avg(/TestServer/system.cpu.util,5m)>{$CPU.UTIL.MAX}
   
   # Можно переопределить на уровне хоста
   Configuration → Hosts → TestServer → Macros
   {$CPU.UTIL.MAX} = 90
   ```

3. **Event correlation (корреляция событий)**
   ```
   Configuration → Event correlation → Create
   
   Name: Flapping service detection
   
   Conditions:
   - New event tag Application equals: nginx
   - Event tag value equals: service_down
   - Type of event: Problem
   
   Operations:
   - Close old events
   - Tag: flapping
   ```

4. **Service monitoring с SLA**
   ```
   Services → Create service
   
   Name: Web Application
   Parent: None
   
   Problem tags:
   - application: webapp
   
   SLA: 99.9%
   
   Child services:
   - Frontend
   - API
   - Database
   
   # Автоматический расчет SLA на основе триггеров
   ```

---

## Модуль 4: Actions и уведомления (30 минут)

### 🎯 Напоминалка

**Типы Actions:**
```
Trigger actions        # При срабатывании триггеров
Discovery actions      # При обнаружении хостов
Auto-registration      # При авторегистрации агентов
Internal actions       # Внутренние события Zabbix
```

**Media types (каналы уведомлений):**
```
Email                  # SMTP почта
SMS                    # SMS шлюзы
Jabber                 # XMPP
Ez Texting             # SMS сервис
Script                 # Кастомные скрипты
Webhook                # HTTP вебхуки
```

**Популярные webhooks:**
```
Slack
MS Teams
Telegram
PagerDuty
OpsGenie
Mattermost
Discord
Jira
ServiceNow
```

**Структура Action:**
```yaml
Conditions:          # Когда выполнять
  - Trigger severity >= Warning
  - Host group = Production
  - Trigger value = PROBLEM

Operations:          # Что делать
  - Send message to: Admin group
  - Media type: Slack
  - Send to: #alerts
  - Message: 
      Subject: Problem: {EVENT.NAME}
      Body: |
        Problem started: {EVENT.TIME} {EVENT.DATE}
        Host: {HOST.NAME}
        Severity: {EVENT.SEVERITY}
        {TRIGGER.URL}

Recovery operations: # При восстановлении
  - Send message: Problem resolved
  
Update operations:   # При обновлении проблемы
  - Send message: Problem updated
```

**Escalation (эскалация):**
```yaml
Step 1 (0-60s):
  - Send to: Operator group
  - Via: Slack

Step 2 (60-600s):
  - Send to: Team Lead
  - Via: Email + SMS

Step 3 (600-3600s):
  - Send to: Manager
  - Via: Phone call
  - Execute: escalation_script.sh
```

**Макросы в сообщениях:**
```
{HOST.NAME}           # Имя хоста
{HOST.IP}             # IP хоста
{EVENT.NAME}          # Название события
{EVENT.SEVERITY}      # Серьезность
{EVENT.DATE}          # Дата
{EVENT.TIME}          # Время
{TRIGGER.STATUS}      # Статус триггера
{TRIGGER.SEVERITY}    # Серьезность триггера
{ITEM.VALUE}          # Значение item
{ITEM.LASTVALUE}      # Последнее значение
{EVENT.AGE}           # Возраст проблемы
{EVENT.ACK.STATUS}    # Статус подтверждения
{TRIGGER.URL}         # URL на триггер
{EVENT.TAGS}          # Теги события
{EVENT.OPDATA}        # Operational data
```

**Шаблоны сообщений:**
```
Subject: {EVENT.SEVERITY}: {EVENT.NAME}

Body:
Problem: {EVENT.NAME}
Host: {HOST.NAME} ({HOST.IP})
Severity: {EVENT.SEVERITY}
Started: {EVENT.DATE} {EVENT.TIME}
Duration: {EVENT.AGE}

Current values:
{ITEM.VALUE1}

Link: {TRIGGER.URL}

---
Zabbix Monitoring System
```

### 💻 Задание

Настрой систему уведомлений:

1. **Настрой Email уведомления**
   
   Administration → Media types → Email → Configure
   ```
   SMTP server: smtp.gmail.com
   SMTP server port: 587
   SMTP helo: zabbix.local
   SMTP email: your-email@gmail.com
   Connection security: STARTTLS
   Authentication: Username and password
   Username: your-email@gmail.com
   Password: <app-password>
   ```

2. **Создай Media для пользователя**
   
   Administration → Users → Admin → Media → Add
   ```
   Type: Email
   Send to: your-email@example.com
   When active: 1-7,00:00-24:00
   Use if severity: All
   Status: Enabled
   ```

3. **Настрой Slack webhook**
   
   Administration → Media types → Create media type
   ```
   Name: Slack
   Type: Webhook
   
   Parameters:
   - alert_message: {ALERT.MESSAGE}
   - alert_subject: {ALERT.SUBJECT}
   - event_source: {EVENT.SOURCE}
   - event_value: {EVENT.VALUE}
   - trigger_id: {TRIGGER.ID}
   - zabbix_url: http://your-zabbix-server
   
   Script:
   try {
     var params = JSON.parse(value);
     var req = new HttpRequest();
     req.addHeader('Content-Type: application/json');
     var webhook_url = '<YOUR_SLACK_WEBHOOK>';
     var payload = {
       channel: '#alerts',
       username: 'Zabbix',
       text: params.alert_subject,
       attachments: [{
         color: params.event_value == '1' ? 'danger' : 'good',
         text: params.alert_message
       }]
     };
     req.post(webhook_url, JSON.stringify(payload));
     return 'OK';
   } catch (error) {
     throw 'Slack notification failed: ' + error;
   }
   ```

4. **Создай Action для проблем**
   
   Configuration → Actions → Trigger actions → Create action
   ```
   Name: Notify on problems
   
   Conditions:
   - Trigger severity >= Warning
   - Host group equals: Production
   
   Operations:
   Step 1-2 (0-60s):
     Send message to: Zabbix administrators
     Via: Slack
     Custom message: Yes
     Subject: 🔴 Problem: {EVENT.NAME}
     Message:
       Host: {HOST.NAME}
       Severity: {EVENT.SEVERITY}
       Time: {EVENT.DATE} {EVENT.TIME}
       Details: {ITEM.VALUE1}
       Link: {TRIGGER.URL}
   
   Step 3-4 (60-300s):
     Send message to: Team Lead
     Via: Email
   
   Recovery operations:
     Send message to: Zabbix administrators
     Via: Slack
     Subject: ✅ Resolved: {EVENT.NAME}
     Message:
       Host: {HOST.NAME}
       Duration: {EVENT.AGE}
       Resolved: {EVENT.RECOVERY.DATE} {EVENT.RECOVERY.TIME}
   
   Update operations:
     Send message to: Zabbix administrators
     Via: Slack
     Custom message: Yes
     Subject: 🔄 Updated: {EVENT.NAME}
   ```

5. **Тест уведомлений**
   ```bash
   # Создай проблему искусственно
   # На агенте
   stress --cpu 8 --timeout 300s
   
   # Или через триггер
   Configuration → Hosts → TestServer → Triggers
   # Временно снизь порог CPU до 10%
   
   # Проверь
   Monitoring → Problems
   # Должно прийти уведомление
   
   # Восстанови порог
   ```

### 🚀 Бонус (новое)

**Настрой Telegram уведомления через Webhook:**

1. **Создай Telegram бота**
   ```
   - Найди @BotFather в Telegram
   - /newbot
   - Получи токен: 123456789:ABCdefGHIjklMNOpqrsTUVwxyz
   - Найди свой Chat ID: @userinfobot
   ```

2. **Создай Media Type для Telegram**
   ```javascript
   // Administration → Media types → Create
   Name: Telegram
   Type: Webhook
   
   Parameters:
   - bot_token: <YOUR_BOT_TOKEN>
   - chat_id: {ALERT.SENDTO}
   - message: {ALERT.MESSAGE}
   - parse_mode: HTML
   
   Script:
   try {
     var params = JSON.parse(value);
     var req = new HttpRequest();
     var url = 'https://api.telegram.org/bot' + params.bot_token + '/sendMessage';
     
     var message = params.message;
     // Экранирование HTML
     message = message.replace(/&/g, '&amp;')
                     .replace(/</g, '&lt;')
                     .replace(/>/g, '&gt;');
     
     var payload = {
       chat_id: params.chat_id,
       text: message,
       parse_mode: params.parse_mode,
       disable_web_page_preview: true
     };
     
     req.addHeader('Content-Type: application/json');
     var response = req.post(url, JSON.stringify(payload));
     
     if (req.getStatus() != 200) {
       throw 'Response code: ' + req.getStatus();
     }
     
     return 'OK';
   } catch (error) {
     throw 'Telegram notification failed: ' + error;
   }
   ```

**Настрой условную эскалацию:**

```yaml
Action: Smart escalation

Conditions:
- Trigger severity >= Average
- Problem is not suppressed

Operations:
Step 1 (0-300s):
  IF {EVENT.SEVERITY} >= High:
    - Send to: On-call engineer
    - Via: Telegram + SMS
  ELSE:
    - Send to: Team channel
    - Via: Slack

Step 2 (300-900s):
  IF {EVENT.ACK.STATUS} = No:
    - Send to: Team Lead
    - Via: Email + Phone
  
Step 3 (900-3600s):
  IF {EVENT.STATUS} = PROBLEM:
    - Execute: escalation_script.sh
    - Create: JIRA ticket
    - Send to: Manager
```

**Используй Problem tags для роутинга:**

```yaml
Action: Tag-based routing

Conditions:
- Trigger severity >= Warning

Operations:
IF {EVENT.TAGS} contains "team=database":
  - Send to: DBA team
  - Via: PagerDuty

IF {EVENT.TAGS} contains "team=network":
  - Send to: Network team
  - Via: OpsGenie

IF {EVENT.TAGS} contains "application=critical":
  - Send to: All teams
  - Via: Multiple channels
```

---

## Модуль 5: Templates и массовое управление (25 минут)

### 🎯 Напоминалка

**Структура Template:**
```
Template
├── Items               # Метрики для сбора
├── Triggers            # Условия проблем
├── Graphs              # Графики
├── Discovery rules     # Правила обнаружения
├── Web scenarios       # Веб-мониторинг
├── Macros              # Переменные шаблона
└── Linked templates    # Связанные шаблоны
```

**Встроенные templates:**
```
Linux by Zabbix agent
Windows by Zabbix agent
MySQL by Zabbix agent
PostgreSQL by Zabbix agent
Apache by HTTP
Nginx by HTTP
Docker by Zabbix agent 2
Kubernetes cluster by HTTP
VMware by HTTP
```


**Template linking (наследование):**
```
Template App Generic
↓ linked to
Template App Database
↓ linked to
Template App MySQL
↓ applied to
Host: mysql-prod-01

# Хост наследует все items/triggers/graphs
```

**Template macros (переменные):**
```
{$CPU.UTIL.MAX}          # Максимальная загрузка CPU
{$MEMORY.UTIL.MAX}       # Максимальная загрузка памяти
{$DISK.PFREE.MIN}        # Минимум свободного места
{$SERVICE.NAME}          # Имя сервиса
{$API.TOKEN}             # API токен
{$SNMP.COMMUNITY}        # SNMP community

# Можно переопределить на уровне:
# - Template → Host group → Host
```

**Export/Import templates:**
```bash
# Export через веб-интерфейс
Configuration → Templates → Select → Export
Format: YAML (предпочтительно) или XML

# Import
Configuration → Templates → Import
Source: файл YAML/XML
Rules:
  - Create new
  - Update existing
  - Delete missing
```

**Template groups:**
```
Templates/Applications      # Приложения
Templates/Databases        # БД
Templates/Network devices  # Сетевые устройства
Templates/Operating systems # ОС
Templates/Modules          # Модули
Templates/Virtualization   # Виртуализация
```

**Value mapping:**
```yaml
# Преобразование числовых значений в текст
Configuration → General → Value mapping

Name: Service state
Mappings:
  0 → Down
  1 → Up
  2 → Unknown

# Использование в items
Item: Service status
Type of information: Numeric (unsigned)
Show value: Service state
```

**Cloning и mass update:**
```bash
# Clone host
Configuration → Hosts → Select host → Clone
# Копирует всю конфигурацию

# Mass update
Configuration → Hosts → Select multiple → Mass update
Update:
  - Templates
  - Macros
  - Inventory
  - Tags
  - Groups
```

### 💻 Задание

Создай production-ready template:

1. **Создай кастомный Template**
   
   Configuration → Templates → Create template
   ```
   Template name: Template App Custom Service
   Groups: Templates/Applications
   
   Description:
   Template for custom application monitoring
   - Service availability
   - Response time
   - Error rate
   - Custom metrics
   ```

2. **Добавь Macros**
   ```
   {$SERVICE.NAME} = myapp
   {$SERVICE.PORT} = 8080
   {$SERVICE.URL} = http://localhost:8080/health
   {$RESPONSE.TIME.MAX} = 1000
   {$ERROR.RATE.MAX} = 5
   ```

3. **Создай Items**
   
   Item 1: Service status
   ```
   Name: Service {$SERVICE.NAME} status
   Type: Simple check
   Key: net.tcp.service[tcp,localhost,{$SERVICE.PORT}]
   Type of information: Numeric (unsigned)
   Value mapping: Service state (0=Down, 1=Up)
   Update interval: 30s
   Applications: Service monitoring
   ```
   
   Item 2: Service response time
   ```
   Name: Service {$SERVICE.NAME} response time
   Type: HTTP agent
   Key: service.response.time
   URL: {$SERVICE.URL}
   Type of information: Numeric (float)
   Units: ms
   Update interval: 1m
   
   Preprocessing:
   - Regular expression: \n → (empty) # Remove newlines
   - JSONPath: $.response_time
   ```
   
   Item 3: Service error rate
   ```
   Name: Service {$SERVICE.NAME} error rate
   Type: Zabbix agent
   Key: system.run[grep -c ERROR /var/log/{$SERVICE.NAME}/error.log]
   Type of information: Numeric (unsigned)
   Update interval: 5m
   ```

4. **Создай Triggers**
   
   Trigger 1: Service down
   ```
   Name: Service {$SERVICE.NAME} is down
   Severity: High
   Expression: last(/Template App Custom Service/net.tcp.service[tcp,localhost,{$SERVICE.PORT}])=0
   Recovery: last(/Template App Custom Service/net.tcp.service[tcp,localhost,{$SERVICE.PORT}])=1
   Manual close: Allowed
   
   Tags:
   - service: {$SERVICE.NAME}
   - component: availability
   ```
   
   Trigger 2: High response time
   ```
   Name: Service {$SERVICE.NAME} high response time
   Severity: Warning
   Expression: avg(/Template App Custom Service/service.response.time,5m)>{$RESPONSE.TIME.MAX}
   
   Description:
   Current: {ITEM.LASTVALUE}
   Threshold: {$RESPONSE.TIME.MAX}
   ```
   
   Trigger 3: High error rate
   ```
   Name: Service {$SERVICE.NAME} high error rate
   Severity: Average
   Expression: last(/Template App Custom Service/system.run[grep -c ERROR /var/log/{$SERVICE.NAME}/error.log])>{$ERROR.RATE.MAX}
   ```

5. **Создай Graph**
   ```
   Configuration → Templates → Template App Custom Service → Graphs → Create graph
   
   Name: Service {$SERVICE.NAME} performance
   Width: 900
   Height: 200
   Show legend: Yes
   
   Items:
   - service.response.time (Line, Green)
   - Y axis: Response time (ms)
   ```

6. **Добавь Discovery Rule**
   ```
   Name: Service endpoints discovery
   Type: HTTP agent
   Key: service.endpoints.discovery
   URL: {$SERVICE.URL}/endpoints
   Update interval: 1h
   
   Preprocessing:
   - JSONPath: $.endpoints
   
   Item prototypes:
   Name: Endpoint {#ENDPOINT} response time
   Key: endpoint[{#ENDPOINT},response.time]
   ```

7. **Export template**
   ```
   Configuration → Templates → Template App Custom Service → Export
   Format: YAML
   Сохрани: template_app_custom_service.yaml
   ```

8. **Примени template к хосту**
   ```
   Configuration → Hosts → TestServer → Templates → Select
   Link new templates: Template App Custom Service
   
   Macros (override):
   {$SERVICE.NAME} = nginx
   {$SERVICE.PORT} = 80
   ```

### 🚀 Бонус (новое)

**Создай Template с LLD для Docker containers:**

```bash
# UserParameter для Docker discovery
cat > /etc/zabbix/zabbix_agent2.d/docker_discovery.conf <<'EOF'
UserParameter=docker.containers.discovery,docker ps --format '{"data":[{{range .}}{{if .}}{{printf "{" }}{{printf "\"{#CONTAINER.NAME}\":\"%s\"," .Names}}{{printf "\"{#CONTAINER.ID}\":\"%s\"," .ID}}{{printf "\"{#CONTAINER.IMAGE}\":\"%s\"" .Image}}{{printf "},"}}{{end}}{{end}}]}' | sed 's/,]/]/'
UserParameter=docker.container.stats[*],docker stats --no-stream --format "{{json .}}" $1 | jq -r '.$2'
EOF

systemctl restart zabbix-agent2
```

**Template: Docker Containers**
```yaml
Discovery rule:
  Name: Docker containers discovery
  Key: docker.containers.discovery
  Update interval: 5m

Item prototypes:
  1. Container CPU usage
     Key: docker.container.stats[{#CONTAINER.ID},CPUPerc]
     Preprocessing: 
       - Regular expression: %$ → (empty)
       - Custom multiplier: 1
  
  2. Container Memory usage
     Key: docker.container.stats[{#CONTAINER.ID},MemPerc]
  
  3. Container status
     Key: docker.container.stats[{#CONTAINER.ID},Status]

Trigger prototypes:
  1. Container {#CONTAINER.NAME} high CPU
     Expression: avg(//docker.container.stats[{#CONTAINER.ID},CPUPerc],5m)>80
  
  2. Container {#CONTAINER.NAME} is down
     Expression: find(//docker.container.stats[{#CONTAINER.ID},Status],,"like","Up")=0

Graph prototypes:
  Name: Container {#CONTAINER.NAME} resources
  Items:
    - CPU usage
    - Memory usage
```

**Массовое применение templates через API:**

```python
#!/usr/bin/env python3
import requests
import json

ZABBIX_URL = "http://localhost:8080/api_jsonrpc.php"
ZABBIX_USER = "Admin"
ZABBIX_PASSWORD = "zabbix"

def get_auth_token():
    payload = {
        "jsonrpc": "2.0",
        "method": "user.login",
        "params": {
            "username": ZABBIX_USER,
            "password": ZABBIX_PASSWORD
        },
        "id": 1
    }
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()['result']

def get_template_id(auth, template_name):
    payload = {
        "jsonrpc": "2.0",
        "method": "template.get",
        "params": {
            "output": ["templateid"],
            "filter": {"host": [template_name]}
        },
        "auth": auth,
        "id": 2
    }
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()['result'][0]['templateid']

def get_hosts_by_group(auth, group_name):
    payload = {
        "jsonrpc": "2.0",
        "method": "host.get",
        "params": {
            "output": ["hostid", "host"],
            "groupids": group_name,
            "selectGroups": ["name"]
        },
        "auth": auth,
        "id": 3
    }
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()['result']

def link_template_to_hosts(auth, template_id, host_ids):
    for host_id in host_ids:
        payload = {
            "jsonrpc": "2.0",
            "method": "host.update",
            "params": {
                "hostid": host_id,
                "templates": [{"templateid": template_id}]
            },
            "auth": auth,
            "id": 4
        }
        response = requests.post(ZABBIX_URL, json=payload)
        print(f"Linked template to host {host_id}: {response.json()}")

# Использование
auth = get_auth_token()
template_id = get_template_id(auth, "Template App Custom Service")
hosts = get_hosts_by_group(auth, "Production servers")
host_ids = [h['hostid'] for h in hosts]
link_template_to_hosts(auth, template_id, host_ids)
```

**Template inheritance с override:**

```yaml
Base Template: Template Module ICMP Ping
Items:
  - ICMP ping
  - ICMP loss
  - ICMP response time

↓ linked to

Template OS Linux
Macros override:
  {$ICMP.LOSS.MAX} = 20  # Override from base 50
  {$ICMP.TIME.MAX} = 150 # Override from base 300

↓ linked to

Host: linux-server-01
Macros override:
  {$ICMP.LOSS.MAX} = 10  # Override from template 20
```

---

## Модуль 6: Dashboards и визуализация (30 минут)

### 🎯 Напоминалка

**Типы виджетов:**
```
Graph (classic)        # Классический график
Graph prototype        # График из LLD
Simple graph           # Простой график
Plain text             # Текст
URL                    # Внешний URL в iframe
Action log             # Лог действий
Data overview          # Обзор данных
Clock                  # Часы
Map                    # Карта сети
Problems               # Список проблем
Problems by severity   # Проблемы по серьезности
Host availability      # Доступность хостов
System information     # Информация о системе
Item history           # История item
Top hosts              # Топ хостов
Trigger overview       # Обзор триггеров
Web monitoring         # Веб-мониторинг
Gauge                  # Шкала/спидометр
Pie chart              # Круговая диаграмма
Bar chart              # Столбчатая диаграмма
Geomap                 # Географическая карта
Honeycomb              # Сотовая диаграмма
Item value             # Значение item
SLA report             # SLA отчет
```

**Dashboard pages:**
```yaml
Dashboard: Infrastructure Overview
├── Page 1: Overview
│   ├── Widget: Problems by severity
│   ├── Widget: System status
│   └── Widget: Top CPU usage
├── Page 2: Servers
│   ├── Widget: Linux servers
│   └── Widget: Windows servers
├── Page 3: Network
│   ├── Widget: Network map
│   └── Widget: Bandwidth
└── Page 4: Applications
    ├── Widget: Databases
    └── Widget: Web servers
```

**Widget configuration:**
```yaml
Widget: Graph
Settings:
  Data set 1:
    Host: TestServer
    Item: CPU utilization
    Time shift: 0
    Aggregation function: avg
    Aggregate interval: 1m
  
  Displaying options:
    History data selection: Auto
    Show legend: Yes
    Working time: Yes
    Percentile line: 95
  
  Time period:
    From: now-1h
    To: now
  
  Axes:
    Left Y: Auto
    Right Y: Auto
  
  Problems:
    Show problems: Yes
    Problem hosts: TestServer
```

**Time selector:**
```
Last 1 hour        # now-1h/now
Last 24 hours      # now-1d/now
Last 7 days        # now-7d/now
Last 30 days       # now-30d/now
Last year          # now-1y/now
Custom period      # 2025-01-01 00:00 to 2025-01-31 23:59
```

**Sharing dashboards:**
```
Monitoring → Dashboards → Properties
Sharing options:
  - Private (default)
  - Public (anyone with link)
  - Public with authentication

Dashboard URL:
http://zabbix/zabbix.php?action=dashboard.view&dashboardid=X
```

**Dashboard slideshows:**
```
Monitoring → Dashboards → Create slideshow

Name: Infrastructure monitoring
Slides:
  1. Overview (30s)
  2. Server health (30s)
  3. Network status (30s)
  4. Application metrics (30s)

# Автоматическая ротация для мониторинга на больших экранах
```

**URL widget с кастомными дашбордами:**
```yaml
Widget: URL
URL: https://grafana.example.com/dashboard/xyz?kiosk
Refresh interval: 30s
Dynamic item: No

# Интеграция внешних систем:
# - Grafana
# - Kibana
# - Custom apps
```

### 💻 Задание

Создай production dashboard:

1. **Создай новый Dashboard**
   ```
   Monitoring → Dashboards → Create dashboard
   
   Name: Production Infrastructure
   Owner: Admin
   Default page display period: Last 1 hour
   Start slideshow: No
   ```

2. **Page 1: System Overview**
   
   Widget 1: Problems by severity
   ```
   Type: Problems by severity
   Name: Current problems
   Refresh interval: 1m
   
   Show:
     - Problems
     - Recent problems: 1h
   
   Host groups: Production servers
   Exclude host groups: (none)
   Hosts: (all)
   Problem: (all)
   Severity: Not classified to Disaster
   Tags: (none)
   Show tags: 3
   Show timeline: Yes
   ```
   
   Widget 2: Top hosts by CPU
   ```
   Type: Top hosts
   Name: Top 10 CPU usage
   Refresh interval: 1m
   
   Host groups: Production servers
   Hosts: (all)
   Items:
     - system.cpu.util
   
   Order: Top N
   Order column: Last value
   Count: 10
   ```
   
   Widget 3: System information
   ```
   Type: System information
   Name: Zabbix server info
   Refresh interval: 5m
   
   Show:
     - System information
     - High availability cluster
     - Server
   ```

3. **Page 2: Server Monitoring**
   
   Widget 1: Host availability
   ```
   Type: Host availability
   Name: Server availability
   
   Host groups: Production servers
   Interface type: Agent
   
   Layout:
     - Horizontal
     - Show hosts in maintenance
   ```
   
   Widget 2: CPU Usage Graph
   ```
   Type: Graph
   Name: CPU utilization
   Refresh interval: 1m
   
   Data sets:
     Set 1:
       Host: {HOST.HOST}
       Item: system.cpu.util
       Color: Green
       Width: 2
   
   Time period: now-1h to now
   Show legend: Yes
   Show problems: Yes
   ```
   
   Widget 3: Memory Usage Graph
   ```
   Type: Graph
   Name: Memory utilization
   
   Data sets:
     Set 1: Memory used
     Set 2: Memory available
     Set 3: Memory cached
   
   Overrides:
     - Stacked area
     - Different colors
   ```

4. **Page 3: Application Metrics**
   
   Widget 1: Item value (Gauge)
   ```
   Type: Gauge
   Name: Service response time
   Refresh interval: 30s
   
   Item: service.response.time
   
   Thresholds:
     - 0-500ms: Green
     - 500-1000ms: Yellow
     - 1000+ms: Red
   
   Units: ms
   Needle: Show
   Scale: Auto
   ```
   
   Widget 2: Pie chart
   ```
   Type: Pie chart
   Name: Problems by severity
   
   Data set:
     Host groups: Production
     Item: Problem count
     Aggregation: sum
   
   Legend: Right side
   Show percentages: Yes
   ```

5. **Page 4: Custom Maps**
   
   Widget: Map
   ```
   Type: Map
   Name: Infrastructure map
   
   Map: Create new map
   Elements:
     - Hosts (auto-positioned)
     - Triggers (show problems)
     - Links (network connections)
   
   Background image: Network topology
   Size: 1200x800
   ```

6. **Настрой Auto-refresh**
   ```
   Dashboard settings:
   Default page display period: Last 1 hour
   Auto-refresh: Yes
   ```

7. **Share Dashboard**
   ```
   Properties → Sharing
   Type: Public link
   Copy URL: http://localhost:8080/zabbix.php?action=dashboard.view&dashboardid=X
   
   # Можно открыть на TV/monitor без авторизации
   ```

### 🚀 Бонус (новое)

**Создай интерактивный Geomap:**

```yaml
Widget: Geomap
Name: Global infrastructure

Settings:
  Default view:
    Latitude: 48.8566 (Paris)
    Longitude: 2.3522
    Zoom: 5
  
  Host groups: Production servers
  
  Mappings:
    - Tag: location=europe → Icon: Server (Blue)
    - Tag: location=us → Icon: Server (Green)
    - Tag: location=asia → Icon: Server (Yellow)
  
  Show problems: Yes
  Minimum severity: Average

# На хостах добавь tags:
Host: paris-web-01
Tags:
  - location: europe
  - geo_lat: 48.8566
  - geo_lon: 2.3522
```

**Honeycomb widget для visualize многих метрик:**

```yaml
Widget: Honeycomb
Name: Service health matrix

Data:
  Host groups: Production servers
  Items:
    - system.cpu.util
    - vm.memory.size[pavailable]
    - vfs.fs.size[/,pfree]
  
  Color palette:
    - 0-70%: Green
    - 70-85%: Yellow
    - 85-100%: Red

# Показывает состояние всех серверов в виде сот
```

**JavaScript widget для кастомной визуализации:**

```javascript
// Monitoring → Dashboards → Add widget → Item value

// Custom JavaScript
<script>
// Get item value
var value = widget.getFieldValue('item');
var threshold = 80;

// Custom HTML
var html = '<div style="padding: 20px; text-align: center;">';
html += '<h2>CPU Usage</h2>';
html += '<div style="font-size: 48px; font-weight: bold; color: ' + 
        (value > threshold ? 'red' : 'green') + ';">';
html += value + '%';
html += '</div>';

if (value > threshold) {
  html += '<div style="color: red; margin-top: 10px;">⚠️ High CPU usage!</div>';
}

html += '</div>';

document.getElementById(widget.uniqueid).innerHTML = html;

// Auto-refresh every 30 seconds
setTimeout(function() {
  widget.refresh();
}, 30000);
</script>
```

**SLA Report widget:**

```yaml
Widget: SLA report
Name: Service availability SLA

Settings:
  Service: Web Application
  SLA: 99.9%
  Reporting period: Last 30 days
  
  Display:
    - SLA
    - Uptime
    - Downtime
    - Problem list
  
  Excluded downtimes:
    - Maintenance windows
    - Planned outages

# Автоматический расчет на основе триггеров
```

**Dashboard templates для разных ролей:**

```yaml
Dashboard: Executive View (CEO/CTO)
Widgets:
  - Overall SLA
  - Critical problems only
  - Service availability
  - Monthly trends

Dashboard: Operations View (DevOps)
Widgets:
  - All problems
  - Server metrics
  - Application performance
  - Recent changes
  - Alert history

Dashboard: Developer View
Widgets:
  - Application logs
  - API response times
  - Error rates
  - Database performance
  - Deployment status

Dashboard: NOC View (24/7 monitoring)
Widgets:
  - Network map
  - All active problems
  - Host availability
  - Auto-rotating slideshow
```

**Export/Import dashboards:**

```bash
# Export через API
curl -X POST http://localhost:8080/api_jsonrpc.php \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "dashboard.export",
    "params": {
      "dashboardids": ["1"]
    },
    "auth": "<TOKEN>",
    "id": 1
  }' | jq -r '.result' > dashboard.json

# Import
Monitoring → Dashboards → Import
Select file: dashboard.json
Rules: Create new / Update existing
```

---
## Модуль 7: Zabbix Proxy и распределенный мониторинг (25 минут)

### 🎯 Напоминалка

**Zabbix Proxy - зачем нужен:**
```
Сценарии использования:
- Мониторинг удаленных локаций
- Снижение нагрузки на Zabbix Server
- Мониторинг за NAT/firewall
- Географически распределенная инфраструктура
- Офлайн мониторинг (с последующей синхронизацией)
```

**Архитектура с Proxy:**
```
Zabbix Server (HQ)
├── Database
└── Web Interface

↓ (через интернет/VPN)

Zabbix Proxy (Remote Office 1)
├── Local Database (SQLite/MySQL/PostgreSQL)
└── Cache для офлайн работы
    ├── Zabbix Agent (Server 1)
    ├── Zabbix Agent (Server 2)
    └── SNMP devices

Zabbix Proxy (Remote Office 2)
└── (аналогично)
```

**Режимы работы Proxy:**
```yaml
Active mode:
  - Proxy инициирует соединение с Server
  - Proxy запрашивает конфигурацию
  - Proxy отправляет собранные данные
  - Подходит для Proxy за NAT
  - Порт 10051 на Server должен быть открыт

Passive mode:
  - Server инициирует соединение с Proxy
  - Server отправляет конфигурацию
  - Server запрашивает данные
  - Proxy должен быть доступен для Server
  - Порт 10051 на Proxy должен быть открыт
```

**Proxy configuration файл:**
```bash
# /etc/zabbix/zabbix_proxy.conf

# Основные параметры
ProxyMode=0              # 0=active, 1=passive
Server=zabbix-server-ip  # IP Zabbix Server
Hostname=proxy-office-1  # Уникальное имя Proxy

# Database
DBHost=localhost
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=password

# Network
ListenPort=10051
StartPollers=10          # Количество pollers
StartTrappers=5

# Cache (для офлайн работы)
ConfigFrequency=3600     # Частота получения конфиг (сек)
DataSenderFrequency=1    # Частота отправки данных (сек)
ProxyLocalBuffer=0       # Хранить данные локально (часы)
ProxyOfflineBuffer=24    # Буфер при недоступности Server (часы)

# Логи
LogFile=/var/log/zabbix/zabbix_proxy.log
LogFileSize=10
DebugLevel=3
```

**Установка Zabbix Proxy:**
```bash
# Ubuntu/Debian
apt install zabbix-proxy-sqlite3  # или mysql/postgresql

# Конфигурация
vi /etc/zabbix/zabbix_proxy.conf
# Настроить Server, Hostname, DBName

# Создать БД (SQLite)
# Автоматически при запуске

# Для MySQL
mysql -uroot -p
CREATE DATABASE zabbix_proxy CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON zabbix_proxy.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;

# Импорт схемы
zcat /usr/share/zabbix-sql-scripts/mysql/proxy.sql.gz | mysql -uzabbix -p zabbix_proxy

# Запуск
systemctl enable zabbix-proxy
systemctl start zabbix-proxy
systemctl status zabbix-proxy

# Проверка логов
tail -f /var/log/zabbix/zabbix_proxy.log
```

**Docker Compose для Proxy:**
```yaml
version: '3.8'
services:
  zabbix-proxy:
    image: zabbix/zabbix-proxy-sqlite3:alpine-6.4-latest
    container_name: zabbix-proxy-office1
    environment:
      ZBX_PROXYMODE: 0  # 0=active, 1=passive
      ZBX_SERVER_HOST: zabbix-server.example.com
      ZBX_HOSTNAME: proxy-office-1
      ZBX_CONFIGFREQUENCY: 3600
      ZBX_DATASENDERFREQUENCY: 1
      ZBX_PROXYOFFLINEBUFFER: 24
      ZBX_ENABLEREMOTECOMMANDS: 1
      ZBX_STARTPOLLERS: 10
    ports:
      - "10051:10051"
    volumes:
      - zabbix-proxy-data:/var/lib/zabbix
    restart: unless-stopped
    networks:
      - proxy-net

  zabbix-agent:
    image: zabbix/zabbix-agent2:alpine-6.4-latest
    environment:
      ZBX_HOSTNAME: "Server in Office 1"
      ZBX_SERVER_HOST: zabbix-proxy-office1
      ZBX_SERVER_PORT: 10051
    privileged: true
    pid: "host"
    networks:
      - proxy-net

volumes:
  zabbix-proxy-data:

networks:
  proxy-net:
    driver: bridge
```

**Регистрация Proxy на Server:**
```
Administration → Proxies → Create proxy

Name: proxy-office-1
Proxy mode: Active
Proxy address: (для passive mode)

Advanced configuration:
  Heartbeat frequency: 60s
  Configuration sync frequency: 3600s
```

**Назначение хостов на Proxy:**
```
Configuration → Hosts → Create host / Edit host

Host name: server-remote-1
Groups: Remote servers
Monitored by proxy: proxy-office-1
Interfaces: 10.10.10.10 (IP в удаленной сети)
```

**Мониторинг Proxy:**
```bash
# Встроенный мониторинг
Configuration → Hosts → Find: proxy-office-1
Template: Template App Zabbix Proxy

Items:
- zabbix[proxy,<name>,lastaccess]     # Последний контакт
- zabbix[proxy,<name>,delay]          # Задержка синхронизации
- zabbix[proxy,<name>,preprocessing_queue]  # Очередь обработки
- zabbix[proxy,<name>,history_queue]  # Очередь истории

# Проверка из CLI
zabbix_server -R config_cache_reload
tail -f /var/log/zabbix/zabbix_server.log | grep proxy
```

**High Availability для Proxy (Zabbix 7.0+):**
```yaml
# Несколько Proxy с одинаковым именем
Proxy 1: proxy-office-1 (active)
Proxy 2: proxy-office-1 (standby)

# Server автоматически переключается при недоступности
# Используется для критичных локаций
```

### 💻 Задание

Настрой распределенный мониторинг с Proxy:

1. **Разверни Zabbix Proxy через Docker**
```bash
   # Создай docker-compose.yml
   cat > docker-compose-proxy.yml <<'EOF'
   version: '3.8'
   services:
     zabbix-proxy:
       image: zabbix/zabbix-proxy-sqlite3:alpine-6.4-latest
       container_name: zabbix-proxy-test
       environment:
         ZBX_PROXYMODE: 0
         ZBX_SERVER_HOST: <YOUR_ZABBIX_SERVER_IP>
         ZBX_HOSTNAME: proxy-test
         ZBX_CONFIGFREQUENCY: 300
         ZBX_DATASENDERFREQUENCY: 1
       ports:
         - "10061:10051"
       volumes:
         - proxy-data:/var/lib/zabbix
       restart: unless-stopped

     test-agent:
       image: zabbix/zabbix-agent2:alpine-6.4-latest
       environment:
         ZBX_HOSTNAME: "Test server via proxy"
         ZBX_SERVER_HOST: zabbix-proxy-test
       privileged: true
       pid: "host"

   volumes:
     proxy-data:
   EOF

   # Запуск
   docker-compose -f docker-compose-proxy.yml up -d

   # Проверка
   docker-compose -f docker-compose-proxy.yml logs -f zabbix-proxy
```

2. **Зарегистрируй Proxy на Server**
   ```
   Administration → Proxies → Create proxy
   
   Name: proxy-test
   Proxy mode: Active
   
   Encryption:
     Connections to proxy: No encryption
     Connections from proxy: No encryption
   
   Advanced configuration:
     Heartbeat frequency: 60
   
   # После создания подожди 1-2 минуты
   # Должен появиться зеленый индикатор
   ```

3. **Создай Host, мониторимый через Proxy**
   ```
   Configuration → Hosts → Create host
   
   Host name: TestServerViaProxy
   Groups: Linux servers
   Interfaces:
     Agent: DNS: test-agent / IP: 172.x.x.x
     Port: 10050
   
   Monitored by proxy: proxy-test
   
   Templates:
     - Linux by Zabbix agent
   ```

4. **Проверь работу Proxy**
```
   # В веб-интерфейсе
   Administration → Proxies
   # Проверь Last seen и Version
   
   # Проверь сбор данных
   Monitoring → Latest data
   Host: TestServerViaProxy
   # Должны поступать метрики
   
   # Логи proxy
   docker exec zabbix-proxy-test tail -f /var/log/zabbix/zabbix_proxy.log
```

5. **Настрой мониторинг самого Proxy**
   ```
   Configuration → Hosts → proxy-test
   Templates → Add
     - Template App Zabbix Proxy
   
   # Проверь items
   Monitoring → Latest data
   Host: proxy-test
   Items:
     - Proxy delay
     - Proxy queue
     - Last access time
   ```

6. **Симулируй отказ связи**
   ```bash
   # Останови proxy
   docker-compose -f docker-compose-proxy.yml stop zabbix-proxy
   
   # На Server проверь
   Administration → Proxies
   # Last seen должен устаревать
   
   # Triggers должны сработать
   Monitoring → Problems
   # Proxy proxy-test is unreachable
   
   # Восстанови
   docker-compose -f docker-compose-proxy.yml start zabbix-proxy
   
   # Проверь синхронизацию данных
   # Данные за время недоступности должны подгрузиться
   ```

### 🚀 Бонус (новое)

**Настрой Proxy с encryption:**

```bash
# На Proxy сгенерируй PSK
openssl rand -hex 32 > /etc/zabbix/proxy.psk

# Запиши PSK identity
echo "proxy-test-psk" > /etc/zabbix/proxy_psk_identity.txt

# Конфигурация Proxy
TLSConnect=psk
TLSAccept=psk
TLSPSKIdentity=proxy-test-psk
TLSPSKFile=/etc/zabbix/proxy.psk

# На Server
Administration → Proxies → proxy-test → Encryption

Connections to proxy:
  Encryption: PSK
  PSK identity: proxy-test-psk
  PSK: <содержимое proxy.psk>

Connections from proxy:
  Encryption: PSK
  PSK identity: proxy-test-psk
  PSK: <содержимое proxy.psk>
```

**Многоуровневая архитектура (Proxy → Proxy):**

```yaml
# НЕ ПОДДЕРЖИВАЕТСЯ напрямую в Zabbix
# Но можно реализовать через SSH tunnels

Zabbix Server (HQ)
↓
Zabbix Proxy (Regional DC)
↓ (SSH tunnel)
Zabbix Proxy (Branch Office)
↓
Agents

# На Regional DC:
ssh -L 10051:localhost:10051 branch-office-proxy -N -f

# Branch Office Proxy конфигурация:
Server=127.0.0.1  # Через SSH tunnel
```

**Автоматическая регистрация хостов через Proxy:**

```yaml
Configuration → Actions → Autoregistration actions → Create action

Name: Auto-register hosts via proxy-test

Conditions:
  - Proxy equals: proxy-test
  - Host metadata contains: Linux

Operations:
  - Add host
  - Add to host groups: Auto-registered Linux
  - Link to templates: Linux by Zabbix agent
  - Set monitored by proxy: proxy-test

# На агенте
HostMetadata=Linux auto-registered
ServerActive=proxy-test-ip:10051
```

**Load balancing между несколькими Proxy:**

```python
#!/usr/bin/env python3
# Скрипт для распределения хостов по Proxy

import requests
import json

ZABBIX_URL = "http://localhost:8080/api_jsonrpc.php"
AUTH_TOKEN = "<your_token>"

proxies = ["proxy-office-1", "proxy-office-2", "proxy-office-3"]

def get_proxy_load(proxy_name):
    """Получить нагрузку на proxy"""
    payload = {
        "jsonrpc": "2.0",
        "method": "proxy.get",
        "params": {
            "output": ["host", "lastaccess"],
            "selectHosts": "count",
            "filter": {"host": proxy_name}
        },
        "auth": AUTH_TOKEN,
        "id": 1
    }
    response = requests.post(ZABBIX_URL, json=payload)
    result = response.json()['result'][0]
    return int(result['hosts'])

def assign_host_to_proxy(host_id, proxy_id):
    """Назначить хост на proxy"""
    payload = {
        "jsonrpc": "2.0",
        "method": "host.update",
        "params": {
            "hostid": host_id,
            "proxy_hostid": proxy_id
        },
        "auth": AUTH_TOKEN,
        "id": 2
    }
    requests.post(ZABBIX_URL, json=payload)

# Получить proxy с минимальной нагрузкой
loads = {p: get_proxy_load(p) for p in proxies}
min_loaded_proxy = min(loads, key=loads.get)
print(f"Least loaded proxy: {min_loaded_proxy} ({loads[min_loaded_proxy]} hosts)")
```

---

## Модуль 8: Интеграции и расширения (30 минут)

### 🎯 Напоминалка

**Типы интеграций:**
```
Webhooks              # Исходящие HTTP запросы
External scripts      # Выполнение скриптов на Server
Global scripts        # Скрипты через веб-интерфейс
API                   # REST API для автоматизации
Database direct       # Прямой доступ к БД
Zabbix sender         # Отправка метрик извне
```

**Популярные интеграции:**
```
ITSM:
- Jira
- ServiceNow
- BMC Remedy

Chat/Collaboration:
- Slack
- Microsoft Teams
- Telegram
- Discord
- Mattermost

Incident Management:
- PagerDuty
- OpsGenie
- VictorOps

Monitoring/Observability:
- Grafana
- Prometheus
- ELK Stack
- Datadog

Automation:
- Ansible
- SaltStack
- Puppet
- Jenkins
```

**Zabbix API основы:**
```bash
# Endpoint
http://zabbix-server/api_jsonrpc.php

# Структура запроса
{
  "jsonrpc": "2.0",
  "method": "method.name",
  "params": {...},
  "auth": "token",
  "id": 1
}

# Авторизация
curl -X POST http://zabbix/api_jsonrpc.php \
  -H "Content-Type: application/json-rpc" \
  -d '{
    "jsonrpc": "2.0",
    "method": "user.login",
    "params": {
      "username": "Admin",
      "password": "zabbix"
    },
    "id": 1
  }'

# Response: {"result": "auth_token"}
```

**Основные API методы:**
```python
# Hosts
host.get              # Получить хосты
host.create           # Создать хост
host.update           # Обновить хост
host.delete           # Удалить хост

# Items
item.get              # Получить items
item.create           # Создать item
item.update           # Обновить item

# Triggers
trigger.get           # Получить триггеры
trigger.create        # Создать триггер

# Problems
problem.get           # Получить проблемы
event.acknowledge     # Подтвердить событие

# History
history.get           # Получить историю
trend.get             # Получить тренды

# Templates
template.get          # Получить templates
template.create       # Создать template
configuration.import  # Импорт конфигурации
configuration.export  # Экспорт конфигурации
```

**Global scripts:**
```bash
# Administration → Scripts → Create script

Name: Restart service
Type: Script
Execute on: Zabbix server
Commands:
  ssh -o StrictHostKeyChecking=no {HOST.CONN} "sudo systemctl restart nginx"

Host group: All
User group: Administrators
Enable confirmation: Yes
Confirmation text: Restart nginx on {HOST.NAME}?

# Использование
Monitoring → Hosts → Select host → Execute script
```

**External scripts:**
```bash
# Расположение: /usr/lib/zabbix/externalscripts/

# Пример: restart_service.sh
#!/bin/bash
HOST=$1
SERVICE=$2

ssh -o StrictHostKeyChecking=no zabbix@$HOST "sudo systemctl restart $SERVICE"
if [ $? -eq 0 ]; then
  echo "Service $SERVICE restarted successfully on $HOST"
  exit 0
else
  echo "Failed to restart $SERVICE on $HOST"
  exit 1
fi

# Сделать исполняемым
chmod +x /usr/lib/zabbix/externalscripts/restart_service.sh
chown zabbix:zabbix /usr/lib/zabbix/externalscripts/restart_service.sh

# В Actions
Operations → Run script
Script name: restart_service.sh {HOST.CONN} nginx
```

**Zabbix Sender:**
```bash
# Отправка метрик извне
zabbix_sender -z zabbix-server -s "hostname" -k "custom.metric" -o 42

# Массовая отправка из файла
cat metrics.txt
hostname custom.metric1 100
hostname custom.metric2 200

zabbix_sender -z zabbix-server -i metrics.txt

# В скрипте
#!/bin/bash
VALUE=$(command_to_get_value)
zabbix_sender -z zabbix-server -s "$(hostname)" -k "custom.metric" -o "$VALUE"

# Trapper item в Zabbix
Type: Zabbix trapper
Key: custom.metric
```

**Prometheus integration:**
```yaml
# Zabbix может собирать метрики из Prometheus

Item configuration:
  Type: HTTP agent
  URL: http://prometheus:9090/api/v1/query?query=up
  Request type: GET
  
  Preprocessing:
    - JSONPath: $.data.result[0].value[1]
    - JavaScript: 
        return value === "1" ? 1 : 0;

# Или использовать Prometheus remote write в Zabbix
```

### 💻 Задание

Настрой интеграции с внешними системами:

1. **Создай Jira интеграцию через Webhook**
   
   Administration → Media types → Create media type
   ```javascript
   Name: Jira
   Type: Webhook
   
   Parameters:
   - jira_url: https://your-domain.atlassian.net
   - username: your-email@example.com
   - api_token: <JIRA_API_TOKEN>
   - project_key: INCIDENT
   - issue_type: Bug
   - priority: {ALERT.SEVERITY}
   - summary: {ALERT.SUBJECT}
   - description: {ALERT.MESSAGE}
   
   Script:
   try {
     var params = JSON.parse(value);
     var req = new HttpRequest();
     
     var auth = btoa(params.username + ':' + params.api_token);
     req.addHeader('Content-Type: application/json');
     req.addHeader('Authorization: Basic ' + auth);
     
     var url = params.jira_url + '/rest/api/2/issue';
     
     var priority_map = {
       'Disaster': '1',  // Highest
       'High': '2',
       'Average': '3',
       'Warning': '4',
       'Information': '5' // Lowest
     };
     
     var payload = {
       fields: {
         project: { key: params.project_key },
         summary: params.summary,
         description: params.description,
         issuetype: { name: params.issue_type },
         priority: { id: priority_map[params.priority] || '3' }
       }
     };
     
     var response = req.post(url, JSON.stringify(payload));
     
     if (req.getStatus() != 201) {
       throw 'JIRA API error: ' + req.getStatus();
     }
     
     var result = JSON.parse(response);
     return 'Created issue: ' + result.key;
   } catch (error) {
     throw 'JIRA integration failed: ' + error;
   }
   ```

2. **Настрой Global Script для автоматизации**
```bash
   # На Zabbix Server создай скрипт
   cat > /usr/lib/zabbix/externalscripts/check_and_restart.sh <<'EOF'
   #!/bin/bash
   HOST=$1
   SERVICE=$2
   
   # Проверка статуса
   STATUS=$(ssh -o StrictHostKeyChecking=no zabbix@$HOST "systemctl is-active $SERVICE")
   
   if [ "$STATUS" != "active" ]; then
     # Рестарт
     ssh zabbix@$HOST "sudo systemctl restart $SERVICE"
     sleep 5
     
     # Проверка после рестарта
     NEW_STATUS=$(ssh zabbix@$HOST "systemctl is-active $SERVICE")
     
     if [ "$NEW_STATUS" = "active" ]; then
       echo "Service $SERVICE successfully restarted on $HOST"
       exit 0
     else
       echo "Failed to restart $SERVICE on $HOST"
       exit 1
     fi
   else
     echo "Service $SERVICE is already running on $HOST"
     exit 0
   fi
   EOF
```

```bash
   chmod +x /usr/lib/zabbix/externalscripts/check_and_restart.sh
   chown zabbix:zabbix /usr/lib/zabbix/externalscripts/check_and_restart.sh
```

   # В веб-интерфейсе
   Administration → Scripts → Create script
   
   Name: Check and restart service
   Type: Script
   Execute on: Zabbix server
   Commands: /usr/lib/zabbix/externalscripts/check_and_restart.sh {HOST.CONN} {$SERVICE.NAME}
   Description: Check service status and restart if needed
   
   Host group: Linux servers
   User group: Administrators
   Enable confirmation: Yes   

3. **Используй Zabbix API для автоматизации**
   ```python
   #!/usr/bin/env python3
   # auto_acknowledge.py - Автоматическое подтверждение проблем
   
   import requests
   import json
   from datetime import datetime
   
   ZABBIX_URL = "http://localhost:8080/api_jsonrpc.php"
   USERNAME = "Admin"
   PASSWORD = "zabbix"
   
   def api_request(method, params, auth=None):
       payload = {
           "jsonrpc": "2.0",
           "method": method,
           "params": params,
           "id": 1
       }
       if auth:
           payload["auth"] = auth
       
       response = requests.post(ZABBIX_URL, json=payload)
       result = response.json()
       
       if 'error' in result:
           raise Exception(result['error'])
       
       return result.get('result')
   
   # Логин
   auth = api_request("user.login", {
       "username": USERNAME,
       "password": PASSWORD
   })
   
   print(f"Authenticated: {auth[:10]}...")
   
   # Получить неподтвержденные проблемы Warning и ниже
   problems = api_request("problem.get", {
       "output": ["eventid", "name", "severity"],
       "severities": [1, 2, 3],  # Information, Warning, Average
       "acknowledged": 0,
       "recent": 1,
       "sortfield": ["eventid"],
       "sortorder": "DESC"
   }, auth)
   
   print(f"Found {len(problems)} unacknowledged problems")
   
   # Автоматическое подтверждение
   for problem in problems:
       eventid = problem['eventid']
       name = problem['name']
       
       api_request("event.acknowledge", {
           "eventids": eventid,
           "action": 6,  # 6 = Add message + Close problem
           "message": f"Auto-acknowledged by script at {datetime.now()}"
       }, auth)
       
       print(f"Acknowledged: {name}")
   
   print("Done!")
   ```

4. **Настрой интеграцию с Grafana**
   ```bash
   # В Grafana установи Zabbix datasource
   # Plugins → Add plugin → Zabbix
   
   # Configuration → Data sources → Add data source → Zabbix
   
   URL: http://zabbix-server/api_jsonrpc.php
   Username: Admin
   Password: zabbix
   
   # Создай dashboard в Grafana
   # Add panel → Select Zabbix datasource
   # Query: 
   #   Group: Linux servers
   #   Host: *
   #   Application: CPU
   #   Item: system.cpu.util
   ```

5. **Создай custom trapper для внешних метрик**

   ##### В Zabbix создай item
   Configuration → Hosts → TestServer → Items → Create item
   
   Name: Custom application metric
   Type: Zabbix trapper
   Key: custom.app.metric
   Type of information: Numeric (float)
   
```bash
   # Скрипт для отправки метрик
   cat > /usr/local/bin/send_metrics.sh <<'EOF'
   #!/bin/bash
   
   # Получить метрику из приложения
   METRIC_VALUE=$(curl -s http://localhost:8080/metrics | jq '.response_time')
   
   # Отправить в Zabbix
   zabbix_sender \
     -z zabbix-server \
     -s "TestServer" \
     -k "custom.app.metric" \
     -o "$METRIC_VALUE"
   EOF
```
 ```bash
   chmod +x /usr/local/bin/send_metrics.sh
```
```
   # Cron для регулярной отправки
   */5 * * * * /usr/local/bin/send_metrics.sh
```

6. **Тест webhook в Slack для нестандартного форматирования**
   ```javascript
   // Administration → Media types → Create media type
   
   Name: Slack Enhanced
   Type: Webhook
   
   Script:
   try {
     var params = JSON.parse(value);
     var req = new HttpRequest();
     req.addHeader('Content-Type: application/json');
     
     var webhook_url = '<YOUR_SLACK_WEBHOOK>';
     
     // Иконки по severity
     var severity_icons = {
       'Disaster': ':fire:',
       'High': ':red_circle:',
       'Average': ':large_orange_diamond:',
       'Warning': ':warning:',
       'Information': ':information_source:'
     };
     
     // Цвета
     var severity_colors = {
       'Disaster': 'danger',
       'High': 'danger',
       'Average': 'warning',
       'Warning': 'warning',
       'Information': 'good'
     };
     
     var severity = params.event_severity || 'Information';
     var icon = severity_icons[severity] || ':question:';
     var color = severity_colors[severity] || '#808080';
     
     var payload = {
       channel: '#alerts',
       username: 'Zabbix Monitoring',
       icon_emoji: ':chart_with_upwards_trend:',
       attachments: [{
         color: color,
         title: icon + ' ' + params.alert_subject,
         text: params.alert_message,
         fields: [
           {
             title: 'Host',
             value: params.host_name,
             short: true
           },
           {
             title: 'Severity',
             value: severity,
             short: true
           },
           {
             title: 'Time',
             value: params.event_date + ' ' + params.event_time,
             short: true
           },
           {
             title: 'Event ID',
             value: params.event_id,
             short: true
           }
         ],
         footer: 'Zabbix Monitoring System',
         footer_icon: 'https://www.zabbix.com/favicon.ico',
         ts: Math.floor(Date.now() / 1000)
       }]
     };
     
     if (params.trigger_url) {
       payload.attachments[0].actions = [{
         type: 'button',
         text: 'View in Zabbix',
         url: params.trigger_url
       }];
     }
     
     var response = req.post(webhook_url, JSON.stringify(payload));
     
     if (req.getStatus() != 200) {
       throw 'Slack webhook failed: ' + req.getStatus();
     }
     
     return 'OK';
```

---

## Модуль 9: Автоматизация и API (30 минут)

### 🎯 Напоминалка

**Zabbix API** - RESTful JSON-RPC 2.0 API для автоматизации:

**Базовая структура запроса:**
```json
{
  "jsonrpc": "2.0",
  "method": "method.name",
  "params": {
    "param1": "value1"
  },
  "auth": "authentication_token",
  "id": 1
}
```

**Authentication:**
```bash
# Получение токена
curl -X POST http://zabbix/api_jsonrpc.php \
  -H "Content-Type: application/json-rpc" \
  -d '{
    "jsonrpc": "2.0",
    "method": "user.login",
    "params": {
      "username": "Admin",
      "password": "zabbix"
    },
    "id": 1
  }'

# Ответ:
# {"jsonrpc":"2.0","result":"auth_token_here","id":1}
```

**Основные методы API:**
```yaml
Hosts:
  - host.get: Получить хосты
  - host.create: Создать хост
  - host.update: Обновить хост
  - host.delete: Удалить хост

Items:
  - item.get: Получить items
  - item.create: Создать item
  - history.get: Получить историю

Triggers:
  - trigger.get: Получить триггеры
  - problem.get: Получить проблемы

Templates:
  - template.get: Получить шаблоны
  - configuration.import: Импорт конфигурации
  - configuration.export: Экспорт конфигурации

Users & Groups:
  - user.get: Получить пользователей
  - usergroup.get: Получить группы

Maintenance:
  - maintenance.create: Создать maintenance
  - maintenance.delete: Удалить maintenance
```

**Примеры запросов:**

**1. Получить все хосты:**
```bash
curl -X POST http://zabbix/api_jsonrpc.php \
  -H "Content-Type: application/json-rpc" \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.get",
    "params": {
      "output": ["hostid", "host", "status"],
      "selectInterfaces": ["ip"],
      "selectGroups": ["name"]
    },
    "auth": "AUTH_TOKEN",
    "id": 1
  }'
```

**2. Создать хост:**
```bash
curl -X POST http://zabbix/api_jsonrpc.php \
  -H "Content-Type: application/json-rpc" \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.create",
    "params": {
      "host": "NewServer",
      "interfaces": [{
        "type": 1,
        "main": 1,
        "useip": 1,
        "ip": "192.168.1.100",
        "dns": "",
        "port": "10050"
      }],
      "groups": [{"groupid": "2"}],
      "templates": [{"templateid": "10001"}]
    },
    "auth": "AUTH_TOKEN",
    "id": 1
  }'
```

**3. Получить проблемы:**
```bash
curl -X POST http://zabbix/api_jsonrpc.php \
  -H "Content-Type: application/json-rpc" \
  -d '{
    "jsonrpc": "2.0",
    "method": "problem.get",
    "params": {
      "output": "extend",
      "selectAcknowledges": "extend",
      "recent": true,
      "sortfield": ["eventid"],
      "sortorder": "DESC"
    },
    "auth": "AUTH_TOKEN",
    "id": 1
  }'
```

**4. Создать maintenance:**
```bash
curl -X POST http://zabbix/api_jsonrpc.php \
  -H "Content-Type: application/json-rpc" \
  -d '{
    "jsonrpc": "2.0",
    "method": "maintenance.create",
    "params": {
      "name": "Weekend maintenance",
      "active_since": 1704067200,
      "active_till": 1704153600,
      "hostids": ["10084"],
      "timeperiods": [{
        "timeperiod_type": 0,
        "start_date": 1704067200,
        "period": 86400
      }]
    },
    "auth": "AUTH_TOKEN",
    "id": 1
  }'
```

**Python wrapper - pyzabbix:**
```python
from pyzabbix import ZabbixAPI

# Подключение
zapi = ZabbixAPI('http://zabbix')
zapi.login('Admin', 'zabbix')

# Получить все хосты
hosts = zapi.host.get(output=['hostid', 'host'])
for host in hosts:
    print(f"{host['hostid']}: {host['host']}")

# Создать хост
host = zapi.host.create(
    host='NewServer',
    interfaces=[{
        'type': 1,
        'main': 1,
        'useip': 1,
        'ip': '192.168.1.100',
        'dns': '',
        'port': '10050'
    }],
    groups=[{'groupid': '2'}],
    templates=[{'templateid': '10001'}]
)

# Получить проблемы
problems = zapi.problem.get(
    output='extend',
    recent=True,
    sortfield='eventid',
    sortorder='DESC'
)

# Создать maintenance
maintenance = zapi.maintenance.create(
    name='Server maintenance',
    active_since=1704067200,
    active_till=1704153600,
    hostids=['10084'],
    timeperiods=[{
        'timeperiod_type': 0,
        'start_date': 1704067200,
        'period': 3600
    }]
)

# Logout
zapi.user.logout()
```

**Ansible Zabbix модули:**
```yaml
---
- name: Manage Zabbix
  hosts: localhost
  tasks:
    - name: Create host
      community.zabbix.zabbix_host:
        server_url: http://zabbix
        login_user: Admin
        login_password: zabbix
        host_name: AnsibleHost
        host_groups:
          - Linux servers
        link_templates:
          - Template OS Linux
        interfaces:
          - type: agent
            main: 1
            useip: 1
            ip: 192.168.1.100
            port: 10050
        status: enabled
        state: present
    
    - name: Create maintenance
      community.zabbix.zabbix_maintenance:
        server_url: http://zabbix
        login_user: Admin
        login_password: zabbix
        name: Maintenance window
        host_names:
          - AnsibleHost
        state: present
        minutes: 60
    
    - name: Get host info
      community.zabbix.zabbix_host_info:
        server_url: http://zabbix
        login_user: Admin
        login_password: zabbix
        host_name: AnsibleHost
      register: host_info
    
    - name: Display info
      debug:
        var: host_info
```

**Terraform Zabbix Provider:**
```hcl
terraform {
  required_providers {
    zabbix = {
      source = "tpretz/zabbix"
      version = "0.16.0"
    }
  }
}

provider "zabbix" {
  username = "Admin"
  password = "zabbix"
  url      = "http://zabbix/api_jsonrpc.php"
}

# Создать host group
resource "zabbix_host_group" "app_servers" {
  name = "Application Servers"
}

# Создать host
resource "zabbix_host" "app01" {
  host   = "app01.example.com"
  groups = [zabbix_host_group.app_servers.id]
  
  interface {
    type = "agent"
    main = true
    ip   = "192.168.1.50"
    port = 10050
  }
  
  template_ids = [
    "10001"  # Template OS Linux
  ]
}

# Создать item
resource "zabbix_item" "app01_custom" {
  hostid      = zabbix_host.app01.id
  key         = "custom.metric"
  name        = "Custom Metric"
  type        = "zabbix_agent"
  value_type  = "unsigned"
  delay       = "60s"
}
```

### 💻 Задание

Автоматизируй управление Zabbix через API:

1. **Создай Python скрипт для массового добавления хостов**
   
   ```python
   #!/usr/bin/env python3
   from pyzabbix import ZabbixAPI
   import csv
   
   # Подключение
   zapi = ZabbixAPI('http://localhost:8080')
   zapi.login('Admin', 'zabbix')
   
   # Читаем CSV со списком хостов
   # Format: hostname,ip,group_name,template_name
   with open('hosts.csv', 'r') as f:
       reader = csv.DictReader(f)
       for row in reader:
           # Получаем group ID
           groups = zapi.hostgroup.get(
               filter={'name': row['group_name']}
           )
           if not groups:
               # Создаем группу если не существует
               group = zapi.hostgroup.create(name=row['group_name'])
               group_id = group['groupids'][0]
           else:
               group_id = groups[0]['groupid']
           
           # Получаем template ID
           templates = zapi.template.get(
               filter={'host': row['template_name']}
           )
           template_id = templates[0]['templateid'] if templates else None
           
           # Создаем хост
           try:
               host = zapi.host.create(
                   host=row['hostname'],
                   interfaces=[{
                       'type': 1,  # Agent
                       'main': 1,
                       'useip': 1,
                       'ip': row['ip'],
                       'dns': '',
                       'port': '10050'
                   }],
                   groups=[{'groupid': group_id}],
                   templates=[{'templateid': template_id}] if template_id else []
               )
               print(f"✓ Created host: {row['hostname']}")
           except Exception as e:
               print(f"✗ Failed to create {row['hostname']}: {e}")
   
   zapi.user.logout()
   print("\nDone!")
   ```
   
   Создай CSV файл `hosts.csv`:
   ```csv
   hostname,ip,group_name,template_name
   web01,192.168.1.101,Web Servers,Template OS Linux
   web02,192.168.1.102,Web Servers,Template OS Linux
   db01,192.168.1.201,Database Servers,Template OS Linux
   ```
   
   Установи pyzabbix и запусти:
   ```bash
   pip3 install pyzabbix
   python3 bulk_add_hosts.py
   ```

2. **Создай скрипт для получения отчета о проблемах**
   
   ```python
   #!/usr/bin/env python3
   from pyzabbix import ZabbixAPI
   from datetime import datetime
   import json
   
   zapi = ZabbixAPI('http://localhost:8080')
   zapi.login('Admin', 'zabbix')
   
   # Получаем активные проблемы
   problems = zapi.problem.get(
       output='extend',
       selectAcknowledges='extend',
       selectTags='extend',
       selectSuppressionData='extend',
       recent=True,
       sortfield='eventid',
       sortorder='DESC',
       limit=100
   )
   
   print("=" * 80)
   print(f"ZABBIX PROBLEMS REPORT - {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
   print("=" * 80)
   print(f"\nTotal active problems: {len(problems)}\n")
   
   # Группируем по severity
   by_severity = {}
   for problem in problems:
       severity = problem['severity']
       if severity not in by_severity:
           by_severity[severity] = []
       by_severity[severity].append(problem)
   
   severity_names = {
       '0': 'Not classified',
       '1': 'Information',
       '2': 'Warning',
       '3': 'Average',
       '4': 'High',
       '5': 'Disaster'
   }
   
   for severity in sorted(by_severity.keys(), reverse=True):
       print(f"\n{severity_names[severity]} ({len(by_severity[severity])} problems):")
       print("-" * 80)
       for problem in by_severity[severity][:5]:  # Показываем топ-5
           print(f"  • {problem['name']}")
           print(f"    Time: {datetime.fromtimestamp(int(problem['clock']))}")
           print(f"    Acknowledged: {'Yes' if problem['acknowledged'] == '1' else 'No'}")
   
   zapi.user.logout()
   ```

3. **Создай скрипт для создания maintenance window**
   
   ```python
   #!/usr/bin/env python3
   from pyzabbix import ZabbixAPI
   from datetime import datetime, timedelta
   import argparse
   
   parser = argparse.ArgumentParser(description='Create maintenance window')
   parser.add_argument('--hosts', nargs='+', required=True, help='Host names')
   parser.add_argument('--hours', type=int, default=2, help='Duration in hours')
   parser.add_argument('--description', default='Maintenance', help='Description')
   args = parser.parse_args()
   
   zapi = ZabbixAPI('http://localhost:8080')
   zapi.login('Admin', 'zabbix')
   
   # Получаем host IDs
   hosts = zapi.host.get(
       filter={'host': args.hosts},
       output=['hostid']
   )
   host_ids = [h['hostid'] for h in hosts]
   
   if not host_ids:
       print("No hosts found!")
       exit(1)
   
   # Создаем maintenance
   now = int(datetime.now().timestamp())
   duration = args.hours * 3600
   
   maintenance = zapi.maintenance.create(
       name=f"{args.description} - {datetime.now().strftime('%Y-%m-%d %H:%M')}",
       active_since=now,
       active_till=now + duration,
       hostids=host_ids,
       timeperiods=[{
           'timeperiod_type': 0,  # One time
           'start_date': now,
           'period': duration
       }]
   )
   
   print(f"✓ Maintenance created for {len(host_ids)} host(s)")
   print(f"  Duration: {args.hours} hour(s)")
   print(f"  Ends: {datetime.fromtimestamp(now + duration)}")
   
   zapi.user.logout()
   ```
   
   Использование:
   ```bash
   python3 create_maintenance.py --hosts web01 web02 --hours 4 --description "Patching"
   ```

4. **Проверь работу скриптов**
   ```bash
   # Запусти все скрипты
   python3 bulk_add_hosts.py
   python3 problems_report.py
   python3 create_maintenance.py --hosts web01 --hours 1
   ```

### 🚀 Бонус (новое)

**Создай Ansible playbook для полной автоматизации:**

```yaml
---
- name: Zabbix Complete Setup
  hosts: localhost
  vars:
    zabbix_url: http://localhost:8080
    zabbix_user: Admin
    zabbix_password: zabbix
  
  tasks:
    - name: Install pyzabbix
      pip:
        name: pyzabbix
        state: present
    
    - name: Create host groups
      community.zabbix.zabbix_hostgroup:
        server_url: "{{ zabbix_url }}"
        login_user: "{{ zabbix_user }}"
        login_password: "{{ zabbix_password }}"
        name: "{{ item }}"
        state: present
      loop:
        - Web Servers
        - Database Servers
        - Application Servers
    
    - name: Create hosts from inventory
      community.zabbix.zabbix_host:
        server_url: "{{ zabbix_url }}"
        login_user: "{{ zabbix_user }}"
        login_password: "{{ zabbix_password }}"
        host_name: "{{ item.name }}"
        host_groups:
          - "{{ item.group }}"
        link_templates:
          - Template OS Linux
        interfaces:
          - type: agent
            main: 1
            useip: 1
            ip: "{{ item.ip }}"
            port: 10050
        status: enabled
        state: present
      loop:
        - { name: 'web01', ip: '192.168.1.101', group: 'Web Servers' }
        - { name: 'web02', ip: '192.168.1.102', group: 'Web Servers' }
        - { name: 'db01', ip: '192.168.1.201', group: 'Database Servers' }
    
    - name: Create custom user macro
      community.zabbix.zabbix_usermacro:
        server_url: "{{ zabbix_url }}"
        login_user: "{{ zabbix_user }}"
        login_password: "{{ zabbix_password }}"
        host_name: web01
        macro_name: CUSTOM_THRESHOLD
        macro_value: "80"
    
    - name: Create action for notifications
      community.zabbix.zabbix_action:
        server_url: "{{ zabbix_url }}"
        login_user: "{{ zabbix_user }}"
        login_password: "{{ zabbix_password }}"
        name: "Notify on High severity"
        event_source: trigger
        status: enabled
        conditions:
          - type: trigger_severity
            operator: ">="
            value: High
        operations:
          - type: send_message
            send_to_users:
              - Admin
            message:
              subject: "Problem: {EVENT.NAME}"
              body: |
                Problem started: {EVENT.TIME} {EVENT.DATE}
                Host: {HOST.NAME}
                Severity: {EVENT.SEVERITY}
```

**CI/CD интеграция для экспорта/импорта конфигурации:**

```bash
#!/bin/bash
# export_zabbix_config.sh

API_URL="http://localhost:8080/api_jsonrpc.php"
AUTH_TOKEN=$(curl -s -X POST $API_URL \
  -H "Content-Type: application/json-rpc" \
  -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"Admin","password":"zabbix"},"id":1}' \
  | jq -r '.result')

# Export templates
curl -s -X POST $API_URL \
  -H "Content-Type: application/json-rpc" \
  -d "{
    \"jsonrpc\": \"2.0\",
    \"method\": \"configuration.export\",
    \"params\": {
      \"options\": {
        \"templates\": [\"10001\", \"10047\"]
      },
      \"format\": \"yaml\"
    },
    \"auth\": \"$AUTH_TOKEN\",
    \"id\": 1
  }" | jq -r '.result' > templates_backup.yaml

# Export hosts
curl -s -X POST $API_URL \
  -H "Content-Type: application/json-rpc" \
  -d "{
    \"jsonrpc\": \"2.0\",
    \"method\": \"configuration.export\",
    \"params\": {
      \"options\": {
        \"hosts\": [\"10084\"]
      },
      \"format\": \"yaml\"
    },
    \"auth\": \"$AUTH_TOKEN\",
    \"id\": 1
  }" | jq -r '.result' > hosts_backup.yaml

echo "✓ Configuration exported"

# Commit to Git
git add templates_backup.yaml hosts_backup.yaml
git commit -m "Zabbix config backup $(date +%Y-%m-%d)"
git push
```

---

## Модуль 10: Интеграции и расширенные возможности (30 минут)

### 🎯 Напоминалка

**Интеграция с внешними системами:**

**1. Grafana интеграция:**
```yaml
Способы интеграции:
  - Zabbix plugin для Grafana
  - Прямое подключение к Zabbix DB
  - API запросы из Grafana
  
Преимущества:
  - Красивые dashboard'ы
  - Объединение данных из разных источников
  - Удобная визуализация
```

**Установка Zabbix plugin для Grafana:**
```bash
# Установка плагина
grafana-cli plugins install alexanderzobnin-zabbix-app

# Или через Docker
docker run -d \
  -p 3000:3000 \
  -e "GF_INSTALL_PLUGINS=alexanderzobnin-zabbix-app" \
  --name=grafana \
  grafana/grafana

# Конфигурация в Grafana UI:
# Configuration → Data Sources → Add data source → Zabbix
# URL: http://zabbix/api_jsonrpc.php
# Username: Admin
# Password: zabbix
```

**2. Prometheus экспорт метрик:**
```bash
# Zabbix может экспортировать метрики в Prometheus формате
# Через отдельный экспортер

# Docker
docker run -d \
  --name zabbix-exporter \
  -p 9091:9091 \
  -e ZBX_SERVER=zabbix-server \
  -e ZBX_USERNAME=Admin \
  -e ZBX_PASSWORD=zabbix \
  zabbix/zabbix-web-service:alpine-6.4-latest

# Prometheus config
scrape_configs:
  - job_name: 'zabbix'
    static_configs:
      - targets: ['localhost:9091']
```

**3. Slack/MS Teams/Telegram интеграция:**

**Slack webhook:**
```javascript
// Media type: Webhook
// Name: Slack

// Script:
try {
    var params = JSON.parse(value);
    var req = new HttpRequest();
    req.addHeader('Content-Type: application/json');
    
    var webhook_url = '{ALERT.SENDTO}';
    
    var severity_colors = {
        'Disaster': '#FF0000',
        'High': '#FF8C00',
        'Average': '#FFA500',
        'Warning': '#FFD700',
        'Information': '#00BFFF',
        'Not classified': '#CCCCCC'
    };
    
    var color = severity_colors[params.severity] || '#808080';
    
    var payload = {
        attachments: [{
            color: color,
            title: params.subject,
            text: params.message,
            footer: 'Zabbix Monitoring',
            ts: Math.floor(Date.now() / 1000)
        }]
    };
    
    req.post(webhook_url, JSON.stringify(payload));
    return 'OK';
} catch (error) {
    throw 'Slack notification failed: ' + error;
}

// Parameters:
// - subject: {ALERT.SUBJECT}
// - message: {ALERT.MESSAGE}
// - severity: {EVENT.SEVERITY}
```

**Telegram bot:**
```javascript
// Media type: Webhook
// Name: Telegram

try {
    var params = JSON.parse(value);
    var req = new HttpRequest();
    
    var bot_token = '<YOUR_BOT_TOKEN>';
    var chat_id = '{ALERT.SENDTO}';
    var url = 'https://api.telegram.org/bot' + bot_token + '/sendMessage';
    
    var severity_emoji = {
        'Disaster': '🔴',
        'High': '🟠',
        'Average': '🟡',
        'Warning': '🟢',
        'Information': '🔵'
    };
    
    var emoji = severity_emoji[params.severity] || '⚪';
    
    var message = emoji + ' *' + params.severity + '*\n\n' +
                  '*Problem:* ' + params.subject + '\n' +
                  params.message;
    
    var payload = {
        chat_id: chat_id,
        text: message,
        parse_mode: 'Markdown',
        disable_web_page_preview: true
    };
    
    req.addHeader('Content-Type: application/json');
    req.post(url, JSON.stringify(payload));
    
    return 'OK';
} catch (error) {
    throw 'Telegram notification failed: ' + error;
}
```

**4. ITSM интеграция (JIRA, ServiceNow):**

**JIRA webhook:**
```javascript
try {
    var params = JSON.parse(value);
    var req = new HttpRequest();
    req.addHeader('Content-Type: application/json');
    req.addHeader('Authorization: Basic ' + btoa('username:api_token'));
    
    var jira_url = 'https://your-domain.atlassian.net/rest/api/2/issue';
    
    var issue = {
        fields: {
            project: { key: 'OPS' },
            summary: params.subject,
            description: params.message,
            issuetype: { name: 'Bug' },
            priority: { name: params.priority },
            labels: ['zabbix', 'monitoring']
        }
    };
    
    var response = req.post(jira_url, JSON.stringify(issue));
    var result = JSON.parse(response);
    
    return 'Created: ' + result.key;
} catch (error) {
    throw 'JIRA integration failed: ' + error;
}
```

**5. PagerDuty интеграция:**
```javascript
try {
    var params = JSON.parse(value);
    var req = new HttpRequest();
    req.addHeader('Content-Type: application/json');
    
    var routing_key = '<YOUR_INTEGRATION_KEY>';
    var url = 'https://events.pagerduty.com/v2/enqueue';
    
    var event_action = params.event_value === '1' ? 'trigger' : 'resolve';
    
    var payload = {
        routing_key: routing_key,
        event_action: event_action,
        dedup_key: params.trigger_id,
        payload: {
            summary: params.subject,
            severity: params.severity.toLowerCase(),
            source: params.host,
            custom_details: {
                message: params.message,
                event_id: params.event_id
            }
        }
    };
    
    req.post(url, JSON.stringify(payload));
    return 'OK';
} catch (error) {
    throw 'PagerDuty notification failed: ' + error;
}
```

**External Scripts:**

Zabbix может вызывать внешние скрипты для уведомлений:

```bash
# /usr/lib/zabbix/alertscripts/custom_alert.sh
#!/bin/bash

TO="$1"
SUBJECT="$2"
MESSAGE="$3"

# Отправка в Slack
curl -X POST \
  -H 'Content-type: application/json' \
  --data "{\"text\":\"$SUBJECT\n$MESSAGE\"}" \
  "$TO"

# Или отправка email через mailx
echo "$MESSAGE" | mailx -s "$SUBJECT" "$TO"

# Или запись в syslog
logger -t zabbix-alert "$SUBJECT: $MESSAGE"

# Или вызов другого API
curl -X POST https://api.example.com/alert \
  -H "Content-Type: application/json" \
  -d "{\"subject\":\"$SUBJECT\",\"message\":\"$MESSAGE\"}"
```

Настройка в Zabbix:
```
Administration → Media types → Create media type
Type: Script
Script name: custom_alert.sh

Parameters:
  - {ALERT.SENDTO}
  - {ALERT.SUBJECT}
  - {ALERT.MESSAGE}
```

**Zabbix Sender для отправки данных:**

```bash
# Установка
apt install zabbix-sender

# Отправка одного значения
zabbix_sender -z zabbix-server -s "ServerName" -k custom.key -o 100

# Отправка нескольких значений из файла
cat > data.txt <<EOF
ServerName custom.metric1 10
ServerName custom.metric2 20
ServerName custom.metric3 30
EOF

zabbix_sender -z zabbix-server -i data.txt

# В скрипте мониторинга
#!/bin/bash
VALUE=$(get_custom_metric)
zabbix_sender -z zabbix-server -s "$(hostname)" -k custom.app.metric -o "$VALUE"
```

**Zabbix Get для тестирования:**

```bash
# Установка
apt install zabbix-get

# Проверка значения item
zabbix_get -s 192.168.1.100 -k system.cpu.load[percpu,avg1]

# Проверка UserParameter
zabbix_get -s 192.168.1.100 -k custom.metric[param1,param2]

# С таймаутом
zabbix_get -s 192.168.1.100 -k web.page.get[localhost] -t 10
```

**SNMP Trap интеграция:**

```bash
# Конфигурация для приема SNMP traps
# /etc/snmp/snmptrapd.conf
authCommunity log,execute,net public
perl do "/usr/share/zabbix/snmptraps/zabbix_trap_receiver.pl";

# Запуск snmptrapd
systemctl enable snmptrapd
systemctl start snmptrapd

# В Zabbix создать item
Type: SNMP trap
Key: snmptrap.fallback
Type of information: Log
```

**Database как источник данных:**

```yaml
# ODBC monitoring
Configuration → Hosts → Items → Create item

Type: Database monitor
Key: db.odbc.select[description,dsn]
Database monitoring: <SQL query>
User: <username>
Password: <password>

# Пример SQL
SELECT COUNT(*) FROM users WHERE last_login > NOW() - INTERVAL 1 HOUR
```

### 💻 Задание

Настрой интеграции с внешними системами:

1. **Создай Telegram бота для уведомлений**
   
   ```bash
   # 1. Создай бота через @BotFather в Telegram
   # Получи token
   
   # 2. Получи chat_id
   # Отправь сообщение своему боту, затем:
   curl https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates | jq
   # Найди chat.id
   
   # 3. В Zabbix Web UI
   # Administration → Media types → Create media type
   # Name: Telegram
   # Type: Webhook
   # Добавь параметры:
   # - bot_token: <YOUR_TOKEN>
   # - chat_id: {ALERT.SENDTO}
   # - message: {ALERT.MESSAGE}
   # - severity: {EVENT.SEVERITY}
   
   # Script (JavaScript):
   ```
   
   ```javascript
   try {
       var params = JSON.parse(value);
       var req = new HttpRequest();
       
       var url = 'https://api.telegram.org/bot' + params.bot_token + '/sendMessage';
       
       var severity_emoji = {
           'Disaster': '🔴',
           'High': '🟠',
           'Average': '🟡',
           'Warning': '🟢',
           'Information': '🔵',
           'Not classified': '⚪'
       };
       
       var emoji = severity_emoji[params.severity] || '⚪';
       var message = emoji + ' *' + params.severity + '*\n\n' + params.message;
       
       var payload = {
           chat_id: params.chat_id,
           text: message,
           parse_mode: 'Markdown'
       };
       
       req.addHeader('Content-Type: application/json');
       var response = req.post(url, JSON.stringify(payload));
       
       if (req.getStatus() != 200) {
           throw 'Response code: ' + req.getStatus();
       }
       
       return 'OK';
   } catch (error) {
       throw 'Telegram notification failed: ' + error;
   }
   ```
   
   ```bash
   # 4. Добавь media пользователю
   # Administration → Users → Admin → Media → Add
   # Type: Telegram
   # Send to: <YOUR_CHAT_ID>
   ```

2. **Создай custom external script**
   
```bash
   # Создай скрипт для логирования в syslog
   cat > /usr/lib/zabbix/alertscripts/syslog_alert.sh <<'EOF'
   #!/bin/bash
   
   TO="$1"
   SUBJECT="$2"
   MESSAGE="$3"
   
   # Определяем priority по subject
   if echo "$SUBJECT" | grep -q "Disaster\|High"; then
       PRIORITY="alert"
   elif echo "$SUBJECT" | grep -q "Average\|Warning"; then
       PRIORITY="warning"
   else
       PRIORITY="info"
   fi
   
   # Отправляем в syslog
   logger -t zabbix-alert -p user.$PRIORITY "$SUBJECT: $MESSAGE"
   
   # Также пишем в файл для истории
   echo "[$(date)] [$PRIORITY] $SUBJECT" >> /var/log/zabbix_alerts.log
   echo "$MESSAGE" >> /var/log/zabbix_alerts.log
   echo "---" >> /var/log/zabbix_alerts.log
   
   exit 0
   EOF
   
   chmod +x /usr/lib/zabbix/alertscripts/syslog_alert.sh
   
   # В Zabbix Web UI
   # Administration → Media types → Create media type
   # Type: Script
   # Script name: syslog_alert.sh
   # Script parameters:
   #   {ALERT.SENDTO}
   #   {ALERT.SUBJECT}
   #   {ALERT.MESSAGE}
```

3. **Настрой Zabbix Sender для отправки custom метрик**
   
```bash
   # Установи zabbix-sender
   apt install zabbix-sender
   
   # Создай Trapper item в Zabbix
   # Configuration → Hosts → TestServer → Items → Create item
   # Name: Custom Application Metric
   # Type: Zabbix trapper
   # Key: custom.app.value
   # Type of information: Numeric (float)
   
   # Создай скрипт для отправки метрик
   cat > /usr/local/bin/send_custom_metric.sh <<'EOF'
   #!/bin/bash
   
   ZABBIX_SERVER="localhost"
   HOSTNAME="TestServer"
   
   # Получаем значение метрики (например, из приложения)
   METRIC_VALUE=$(echo "scale=2; $RANDOM / 100" | bc)
   
   # Отправляем в Zabbix
   zabbix_sender -z "$ZABBIX_SERVER" \
                 -s "$HOSTNAME" \
                 -k "custom.app.value" \
                 -o "$METRIC_VALUE"
   EOF
   
   chmod +x /usr/local/bin/send_custom_metric.sh
   
   # Добавь в cron для периодической отправки
   echo "*/5 * * * * /usr/local/bin/send_custom_metric.sh" | crontab -
   
   # Проверь
   /usr/local/bin/send_custom_metric.sh
```

4. **Создай Dashboard в Grafana с данными из Zabbix**
   
   ```bash
   # Если Grafana не установлена
   docker run -d \
     -p 3000:3000 \
     -e "GF_INSTALL_PLUGINS=alexanderzobnin-zabbix-app" \
     --name=grafana \
     grafana/grafana
   
   # Логин: admin / admin
   
   # В Grafana UI:
   # 1. Configuration → Plugins → Zabbix → Enable
   # 2. Configuration → Data Sources → Add data source → Zabbix
   #    URL: http://host.docker.internal:8080/api_jsonrpc.php
   #    Username: Admin
   #    Password: zabbix
   # 3. Create → Dashboard → Add new panel
   #    Data source: Zabbix
   #    Group: Linux servers
   #    Host: TestServer
   #    Application: CPU
   #    Item: CPU utilization
   ```

5. **Проверь все интеграции**
   ```bash
   # Создай тестовую проблему
   # В Zabbix создай trigger с низким порогом
   
   # Проверь получение уведомлений:
   # - В Telegram
   # - В syslog: tail -f /var/log/syslog | grep zabbix
   # - В файле: tail -f /var/log/zabbix_alerts.log
   
   # Проверь метрики в Grafana
   ```

### 🚀 Бонус (новое)

**Интеграция с Kubernetes:**

```yaml
# Мониторинг Kubernetes через Zabbix
# Используя Zabbix Kubernetes monitoring

apiVersion: v1
kind: ConfigMap
metadata:
  name: zabbix-agent-config
  namespace: monitoring
data:
  zabbix_agentd.conf: |
    Server=zabbix-server
    ServerActive=zabbix-server
    Hostname=k8s-cluster
    
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: zabbix-agent
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: zabbix-agent
  template:
    metadata:
      labels:
        app: zabbix-agent
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: zabbix-agent
        image: zabbix/zabbix-agent2:alpine-6.4-latest
        env:
        - name: ZBX_SERVER_HOST
          value: "zabbix-server"
        - name: ZBX_HOSTNAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
```

**Webhook для автоматического создания maintenance:**

```python
#!/usr/bin/env python3
# Flask webhook для автоматизации maintenance

from flask import Flask, request
from pyzabbix import ZabbixAPI
from datetime import datetime, timedelta

app = Flask(__name__)

@app.route('/maintenance/create', methods=['POST'])
def create_maintenance():
    data = request.json
    
    zapi = ZabbixAPI('http://localhost:8080')
    zapi.login('Admin', 'zabbix')
    
    # Получаем hosts
    hosts = zapi.host.get(
        filter={'host': data['hosts']},
        output=['hostid']
    )
    host_ids = [h['hostid'] for h in hosts]
    
    # Создаем maintenance
    now = int(datetime.now().timestamp())
    duration = data.get('duration_hours', 2) * 3600
    
    maintenance = zapi.maintenance.create(
        name=f"Auto Maintenance - {data.get('reason', 'Deployment')}",
        active_since=now,
        active_till=now + duration,
        hostids=host_ids,
        timeperiods=[{
            'timeperiod_type': 0,
            'start_date': now,
            'period': duration
        }]
    )
    
    zapi.user.logout()
    
    return {
        'status': 'success',
        'maintenance_id': maintenance['maintenanceids'][0],
        'hosts_count': len(host_ids),
        'duration_hours': data.get('duration_hours', 2)
    }

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Использование из CI/CD:**
```bash
# В GitLab CI / Jenkins
curl -X POST http://zabbix-webhook:5000/maintenance/create \
  -H "Content-Type: application/json" \
  -d '{
    "hosts": ["web01", "web02"],
    "duration_hours": 1,
    "reason": "Deployment v2.5.0"
  }'
```

---

## Модуль 10: Оптимизация и производительность (30 минут)

### 🎯 Напоминалка

**Оптимизация производительности Zabbix:**

**1. Database оптимизация:**

```sql
-- Проверка размера таблиц
SELECT 
  table_name,
  ROUND(((data_length + index_length) / 1024 / 1024), 2) AS "Size (MB)"
FROM information_schema.TABLES
WHERE table_schema = "zabbix"
ORDER BY (data_length + index_length) DESC;

-- Самые большие таблицы:
-- history* - численные данные
-- trends* - агрегированные данные
-- events - события

-- Партиционирование больших таблиц (MySQL)
ALTER TABLE history_uint PARTITION BY RANGE (clock) (
  PARTITION p2024_12 VALUES LESS THAN (UNIX_TIMESTAMP('2025-01-01')),
  PARTITION p2025_01 VALUES LESS THAN (UNIX_TIMESTAMP('2025-02-01')),
  PARTITION p2025_02 VALUES LESS THAN (UNIX_TIMESTAMP('2025-03-01')),
  PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- Удаление старых партиций
ALTER TABLE history_uint DROP PARTITION p2024_11;

-- Индексы для ускорения
CREATE INDEX events_clock ON events(clock);
CREATE INDEX history_clock ON history(clock);

-- Анализ медленных запросов
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;
```

**2. Housekeeping оптимизация:**

```yaml
Administration → General → Housekeeping

Settings:
  Enable internal housekeeping: Yes
  
Data storage period:
  Events and alerts: 90 days
  Services: 90 days
  Audit: 90 days
  User sessions: 90 days
  History: 90 days      # Можно снизить до 30-60
  Trends: 365 days      # Или больше для долгосрочного анализа
  
Override item history/trend period:
  Enable: Yes
  # Для конкретных items можно установить меньший период
```

**Партиционирование через скрипт:**
```bash
# Автоматическое партиционирование
# https://github.com/cavaliercoder/zabbix-partition-manager

git clone https://github.com/cavaliercoder/zabbix-partition-manager
cd zabbix-partition-manager

# Конфигурация
cat > config.yml <<EOF
database:
  host: localhost
  port: 3306
  name: zabbix
  user: zabbix
  password: password

tables:
  - name: history
    partition_period: day
    retention_period: 90
  - name: history_uint
    partition_period: day
    retention_period: 90
  - name: trends
    partition_period: month
    retention_period: 365
EOF

# Добавь в cron
echo "0 2 * * * /path/to/partition-manager.py -c config.yml" | crontab -
```

**3. Server конфигурация:**

```bash
# /etc/zabbix/zabbix_server.conf

### Cache оптимизация ###
CacheSize=256M              # Кэш конфигурации (больше для >1000 хостов)
HistoryCacheSize=128M       # Кэш для истории данных
HistoryIndexCacheSize=64M   # Индекс кэша истории
TrendCacheSize=32M          # Кэш для трендов
ValueCacheSize=256M         # Кэш значений items

### Process оптимизация ###
StartPollers=20             # Поллеры для пассивных проверок
StartPollersUnreachable=5   # Для недоступных хостов
StartTrappers=10            # Для active checks и trappers
StartPingers=5              # ICMP проверки
StartDiscoverers=3          # Discovery процессы
StartHTTPPollers=5          # HTTP agent checks
StartTimers=2               # Таймеры
StartEscalators=3           # Эскалация

### Database ###
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=password

# Connection pool
StartDBSyncers=8            # Синхронизаторы БД (больше при высокой нагрузке)

### Timeouts ###
Timeout=30                  # Таймаут для проверок
TrapperTimeout=300          # Таймаут trappers

### Logging ###
LogSlowQueries=3000         # Логировать медленные запросы (мс)
```

**Рекомендации по ресурсам:**

```yaml
Малое окружение (до 100 хостов):
  CPU: 2-4 cores
  RAM: 4-8 GB
  Disk: SSD 50 GB
  Database: MySQL/PostgreSQL на том же сервере

Среднее окружение (100-1000 хостов):
  CPU: 4-8 cores
  RAM: 16-32 GB
  Disk: SSD 200+ GB
  Database: Отдельный сервер с SSD

Большое окружение (1000+ хостов):
  CPU: 16+ cores
  RAM: 64+ GB
  Disk: NVMe SSD 500+ GB
  Database: Отдельный кластер
  Proxy: Множественные для распределения нагрузки
```

**4. Frontend оптимизация:**

```bash
# PHP настройки (/etc/php/*/fpm/pool.d/www.conf или php.ini)
pm.max_children = 50
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
pm.max_requests = 500

# PHP memory
memory_limit = 256M
max_execution_time = 300
max_input_time = 300
post_max_size = 32M

# Nginx cache
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=zabbix_cache:10m max_size=1g inactive=60m;

server {
    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    location / {
        proxy_cache zabbix_cache;
        proxy_cache_valid 200 5m;
    }
}
```

**5. Мониторинг производительности Zabbix:**

```yaml
Template: Template App Zabbix Server

Key Items для мониторинга:
  # Queue
  - zabbix[queue]                    # Items в очереди
  - zabbix[queue,10m]                # Items задержанные >10мин
  
  # Процессы
  - zabbix[process,poller,avg,busy]  # Загрузка поллеров
  - zabbix[process,trapper,avg,busy] # Загрузка trappers
  - zabbix[process,http poller,avg,busy]
  
  # Cache
  - zabbix[vcache,buffer,pfree]      # Свободный кэш %
  - zabbix[rcache,buffer,pfree]      # Кэш конфигурации
  
  # Database
  - zabbix[requiredperformance]      # Требуемая производительность
  - zabbix[wcache,values]            # Записей в кэше для БД
  
  # Items
  - zabbix[items]                    # Общее количество items
  - zabbix[items_unsupported]        # Неподдерживаемые items

Triggers:
  - Queue is growing
  - High poller utilization
  - Cache utilization is high
  - Required performance is too high
```

**6. Item оптимизация:**

```yaml
Лучшие практики:
  # Увеличь интервалы для некритичных метрик
  Critical items: 30s-1m
  Standard items: 5m
  Non-critical: 10m-1h
  
  # Используй calculated items вместо множественных
  Вместо:
    - item1: CPU user
    - item2: CPU system
    - item3: Total CPU (user+system)
  Используй:
    - item1: CPU stats (JSON)
    - calculated1: JSONPath для user
    - calculated2: JSONPath для system
  
  # Отключи ненужные items в templates
  Disable items которые не используются
  
  # Используй trends вместо history для графиков
  History: 30 дней
  Trends: 1-2 года
  
  # Для высокочастотных метрик:
  Preprocessing → Custom multiplier → 0.001
  Храни в меньшей точности если возможно
```

**7. Proxy эффективное использование:**

```yaml
Когда использовать Proxy:
  - Удаленные локации (>100ms latency)
  - >500 хостов в одной локации
  - Требования безопасности (DMZ)
  - Offline мониторинг
  
Распределение нагрузки:
  Location 1: Proxy1 (500 hosts)
  Location 2: Proxy2 (500 hosts)
  Location 3: Proxy3 (500 hosts)
  Central: Server (агрегация)
  
Результат:
  - Сниженная нагрузка на Server
  - Быстрее response time
  - Более надежный мониторинг
```

**8.


---
