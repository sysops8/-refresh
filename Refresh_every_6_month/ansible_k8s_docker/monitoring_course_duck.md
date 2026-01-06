# Мониторинг для DevOps: Ежегодный/Полугодовой курс-освежитель

**Цель:** Освежить в памяти ключевые концепции мониторинга за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная задача, которую нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Предварительные требования:**
- Базовое понимание Linux/Unix
- Доступ к серверу или виртуальной машине
- Docker установлен (для большинства заданий)
- Базовые знания командной строки

---

## Модуль 1: Основы мониторинга и метрики (20 минут)

### 🎯 Напоминалка

**Четыре золотых сигнала (Four Golden Signals):**
```
1. Latency (Задержка)      - Время ответа на запросы
2. Traffic (Трафик)        - Количество запросов
3. Errors (Ошибки)         - Процент неудачных запросов
4. Saturation (Насыщение)  - Загрузка ресурсов (CPU, память, диск)
```

**Типы метрик:**
```
Counter   - Монотонно возрастающее значение (запросы, ошибки)
Gauge     - Текущее значение (CPU, память, температура)
Histogram - Распределение значений (latency buckets)
Summary   - Статистика за период (percentiles)
```

**USE Method (для ресурсов):**
```
Utilization - Среднее время занятости ресурса
Saturation  - Степень перегрузки
Errors      - Количество ошибок
```

**RED Method (для сервисов):**
```
Rate     - Запросов в секунду
Errors   - Количество ошибок
Duration - Время ответа
```

**Уровни мониторинга:**
```
┌─────────────────────────────────┐
│   Application (APM)             │  - Код, транзакции
├─────────────────────────────────┤
│   Service/Container             │  - Docker, K8s
├─────────────────────────────────┤
│   Operating System              │  - CPU, RAM, Disk
├─────────────────────────────────┤
│   Infrastructure                │  - Network, Hardware
└─────────────────────────────────┘
```

**Ключевые метрики Linux:**
```bash
# CPU
top, htop
mpstat -P ALL 1

# Memory
free -m
vmstat 1

# Disk I/O
iostat -x 1
iotop

# Network
iftop
nethogs
ss -s

# Process
ps aux --sort=-%mem | head
ps aux --sort=-%cpu | head
```

**Метрики приложений:**
```
- Request rate (req/s)
- Error rate (%)
- Response time (ms) - p50, p95, p99
- Active connections
- Queue depth
- Database query time
- Cache hit ratio
```

### 💻 Задание

Настрой базовый мониторинг системы:

1. **Установи и запусти Node Exporter** (для сбора метрик хоста):
```bash
# Через Docker
docker run -d \
  --name node-exporter \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  prom/node-exporter:latest \
  --path.rootfs=/host

# Проверка
curl http://localhost:9100/metrics | head -20
```

2. **Изучи основные метрики**:
```bash
# CPU
curl -s http://localhost:9100/metrics | grep node_cpu_seconds_total

# Memory
curl -s http://localhost:9100/metrics | grep node_memory

# Disk
curl -s http://localhost:9100/metrics | grep node_disk

# Network
curl -s http://localhost:9100/metrics | grep node_network
```

3. **Создай простой bash скрипт** для мониторинга (`monitor.sh`):
```bash
#!/bin/bash

echo "=== System Monitoring Report ==="
echo "Date: $(date)"
echo ""

# CPU Usage
echo "CPU Usage:"
top -bn1 | grep "Cpu(s)" | awk '{print "  User: " $2 ", System: " $4 ", Idle: " $8}'

# Memory Usage
echo ""
echo "Memory Usage:"
free -h | awk 'NR==2{printf "  Total: %s, Used: %s (%.2f%%)\n", $2, $3, $3*100/$2}'

# Disk Usage
echo ""
echo "Disk Usage:"
df -h / | awk 'NR==2{printf "  Total: %s, Used: %s (%s)\n", $2, $3, $5}'

# Load Average
echo ""
echo "Load Average:"
uptime | awk -F'load average:' '{print "  " $2}'

# Top 5 processes by CPU
echo ""
echo "Top 5 processes by CPU:"
ps aux --sort=-%cpu | head -6 | tail -5 | awk '{printf "  %s: %.1f%%\n", $11, $3}'

# Top 5 processes by Memory
echo ""
echo "Top 5 processes by Memory:"
ps aux --sort=-%mem | head -6 | tail -5 | awk '{printf "  %s: %.1f%%\n", $11, $4}'
```

4. Запусти скрипт:
```bash
chmod +x monitor.sh
./monitor.sh
```

### 🚀 Бонус (новое)

**Настрой cAdvisor** для мониторинга Docker контейнеров:
```bash
docker run -d \
  --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  gcr.io/cadvisor/cadvisor:latest

# Открой в браузере
http://localhost:8080
```

**Создай свой custom exporter** на Python:
```python
# custom_exporter.py
from prometheus_client import start_http_server, Gauge, Counter
import time
import random

# Создаем метрики
request_gauge = Gauge('app_requests_in_progress', 'Number of requests in progress')
request_counter = Counter('app_requests_total', 'Total number of requests')
error_counter = Counter('app_errors_total', 'Total number of errors')

def process_request():
    """Симулируем обработку запроса"""
    request_gauge.inc()
    request_counter.inc()
    
    # Случайная ошибка
    if random.random() < 0.1:
        error_counter.inc()
    
    time.sleep(random.uniform(0.1, 0.5))
    request_gauge.dec()

if __name__ == '__main__':
    start_http_server(8000)
    print("Exporter started on port 8000")
    
    while True:
        process_request()
        time.sleep(random.uniform(0.5, 2))
```

---

## Модуль 2: Prometheus - сбор и хранение метрик (30 минут)

### 🎯 Напоминалка

**Архитектура Prometheus:**
```
┌─────────────┐
│   Targets   │ ← HTTP Pull (scrape)
│  (Metrics)  │
└──────┬──────┘
       │
   ┌───▼────┐
   │ Prom-  │
   │ etheus │ ← Time Series DB (TSDB)
   │ Server │
   └───┬────┘
       │
   ┌───▼────┐
   │ Alert- │
   │ manager│
   └────────┘
```

**Prometheus config structure:**
```yaml
global:
  scrape_interval: 15s      # Как часто собирать метрики
  evaluation_interval: 15s  # Как часто проверять правила

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

**PromQL основы:**
```promql
# Instant vector - текущее значение
node_cpu_seconds_total

# Range vector - значения за период
node_cpu_seconds_total[5m]

# Фильтры
node_cpu_seconds_total{mode="idle"}
node_cpu_seconds_total{mode!="idle"}
node_cpu_seconds_total{mode=~"user|system"}

# Агрегация
sum(node_cpu_seconds_total)
avg(node_cpu_seconds_total)
max(node_cpu_seconds_total)
min(node_cpu_seconds_total)
count(node_cpu_seconds_total)

# По label
sum(node_cpu_seconds_total) by (mode)
sum(node_cpu_seconds_total) by (cpu)

# Функции
rate(node_cpu_seconds_total[5m])           # Скорость изменения
irate(node_cpu_seconds_total[5m])          # Мгновенная скорость
increase(node_cpu_seconds_total[5m])       # Увеличение за период
delta(node_cpu_seconds_total[5m])          # Изменение
```

**Распространенные запросы:**
```promql
# CPU utilization
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage %
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage %
100 - ((node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100)

# Network traffic
rate(node_network_receive_bytes_total[5m])
rate(node_network_transmit_bytes_total[5m])

# HTTP request rate
rate(http_requests_total[5m])

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])

# Latency percentiles (для histogram)
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))
```

**Metric types в деталях:**
```promql
# Counter - только растет
http_requests_total
# Используй rate() или increase()
rate(http_requests_total[5m])

# Gauge - может расти и падать
node_memory_MemAvailable_bytes
# Используй напрямую или с функциями агрегации
avg(node_memory_MemAvailable_bytes)

# Histogram - распределение значений
http_request_duration_seconds_bucket
http_request_duration_seconds_sum
http_request_duration_seconds_count
# Используй histogram_quantile()
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Summary - предрасчитанные квантили
http_request_duration_seconds{quantile="0.95"}
```

**Recording rules** (для оптимизации):
```yaml
groups:
  - name: example
    interval: 30s
    rules:
    - record: job:node_cpu_utilization:avg
      expr: 100 - (avg by (job) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**Alerting rules:**
```yaml
groups:
  - name: alerts
    rules:
    - alert: HighCPUUsage
      expr: job:node_cpu_utilization:avg > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High CPU usage on {{ $labels.instance }}"
        description: "CPU usage is {{ $value }}%"
```

### 💻 Задание

Настрой полноценный Prometheus:

1. **Создай docker-compose.yml**:
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alerts.yml:/etc/prometheus/alerts.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
      - '--web.enable-lifecycle'
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    command:
      - '--path.rootfs=/host'
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    privileged: true
    restart: unless-stopped

volumes:
  prometheus-data:
```

2. **Создай prometheus.yml**:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Загрузка правил алертов
rule_files:
  - "alerts.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
```

3. **Создай alerts.yml**:
```yaml
groups:
  - name: system_alerts
    rules:
    - alert: HighCPUUsage
      expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High CPU usage detected"
        description: "CPU usage is above 80% (current value: {{ $value }}%)"

    - alert: HighMemoryUsage
      expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High memory usage detected"
        description: "Memory usage is above 90% (current value: {{ $value }}%)"

    - alert: DiskSpaceLow
      expr: (1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100 > 85
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Low disk space"
        description: "Disk usage is above 85% (current value: {{ $value }}%)"

    - alert: InstanceDown
      expr: up == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Instance {{ $labels.instance }} down"
        description: "{{ $labels.instance }} has been down for more than 1 minute"
```

4. **Запусти stack**:
```bash
docker-compose up -d

# Проверка
docker-compose ps
curl http://localhost:9090/api/v1/targets
```

5. **Открой Prometheus UI** и попробуй запросы:
```
Перейди: http://localhost:9090

Попробуй запросы:
- node_cpu_seconds_total
- rate(node_cpu_seconds_total[5m])
- 100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
- node_memory_MemAvailable_bytes / 1024 / 1024 / 1024
```

### 🚀 Бонус (новое)

**Настрой Service Discovery** для автоматического обнаружения целей:

**File-based SD** (`file_sd.json`):
```json
[
  {
    "targets": ["node-exporter:9100"],
    "labels": {
      "job": "node",
      "env": "production"
    }
  },
  {
    "targets": ["cadvisor:8080"],
    "labels": {
      "job": "containers",
      "env": "production"
    }
  }
]
```

Добавь в `prometheus.yml`:
```yaml
scrape_configs:
  - job_name: 'dynamic-targets'
    file_sd_configs:
      - files:
        - '/etc/prometheus/file_sd.json'
        refresh_interval: 30s
```

**Настрой Pushgateway** для метрик batch jobs:
```bash
docker run -d \
  --name pushgateway \
  -p 9091:9091 \
  prom/pushgateway

# Push метрику
echo "backup_duration_seconds 125.5" | curl --data-binary @- http://localhost:9091/metrics/job/backup/instance/db1

# Добавь в prometheus.yml
scrape_configs:
  - job_name: 'pushgateway'
    static_configs:
      - targets: ['pushgateway:9091']
    honor_labels: true
```

**Recording rules для производительности**:
```yaml
# recording_rules.yml
groups:
  - name: performance_rules
    interval: 30s
    rules:
    # CPU utilization per instance
    - record: instance:node_cpu_utilization:rate5m
      expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
    
    # Memory utilization per instance
    - record: instance:node_memory_utilization:ratio
      expr: 1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)
    
    # Request rate per job
    - record: job:http_requests:rate5m
      expr: sum(rate(http_requests_total[5m])) by (job)
```

---

## Модуль 3: Grafana - визуализация данных (30 минут)

### 🎯 Напоминалка

**Архитектура Grafana:**
```
┌──────────┐      ┌──────────┐      ┌──────────┐
│ Data     │◄─────│ Grafana  │◄─────│ Users    │
│ Sources  │      │ Server   │      │          │
└──────────┘      └──────────┘      └──────────┘
   │                    │
   │                    │
   ▼                    ▼
Prometheus       Dashboards
InfluxDB         Alerts
Elasticsearch    Users
Loki             Teams
```

**Типы панелей:**
```
Graph        - Временные ряды
Stat         - Одно значение
Gauge        - Шкала
Bar Gauge    - Горизонтальные полоски
Table        - Таблица
Heatmap      - Тепловая карта
Logs         - Логи
```

**Переменные дашборда:**
```
Query      - Из данных (label_values(metric, label))
Custom     - Список значений
Constant   - Константа
Interval   - Временной интервал
Data source - Выбор источника данных
```

**Полезные функции Grafana:**
```
$__interval        - Динамический интервал
$__rate_interval   - Рекомендуемый интервал для rate()
$timeFilter        - Временной фильтр
$__from / $__to    - Начало/конец периода

# Пример с переменной
rate(http_requests_total{job="$job"}[$__rate_interval])
```

**Templating examples:**
```promql
# Переменная instance
label_values(node_cpu_seconds_total, instance)

# Переменная job
label_values(up, job)

# Переменная mountpoint
label_values(node_filesystem_size_bytes, mountpoint)

# Использование в запросе
node_filesystem_avail_bytes{instance="$instance", mountpoint="$mountpoint"}
```

**Alert channels:**
```
Email
Slack
PagerDuty
Webhook
Telegram
Discord
Teams
OpsGenie
```

**Dashboard best practices:**
```
1. Используй Row для группировки панелей
2. Добавляй описания к панелям
3. Используй переменные для гибкости
4. Указывай единицы измерения
5. Используй цветовые пороги
6. Добавляй ссылки на runbook'и
7. Группируй связанные метрики
8. Используй consistent naming
```

### 💻 Задание

Настрой Grafana и создай dashboard:

1. **Добавь Grafana в docker-compose.yml**:
```yaml
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    restart: unless-stopped
    depends_on:
      - prometheus

volumes:
  grafana-data:
```

2. **Создай provisioning для автоматической настройки** (`grafana/provisioning/datasources/prometheus.yml`):
```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
```

3. **Создай provisioning для dashboard** (`grafana/provisioning/dashboards/dashboard.yml`):
```yaml
apiVersion: 1

providers:
  - name: 'Default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /etc/grafana/provisioning/dashboards
```

4. **Запусти Grafana**:
```bash
# Создай директории
mkdir -p grafana/provisioning/datasources
mkdir -p grafana/provisioning/dashboards

docker-compose up -d grafana

# Открой в браузере
http://localhost:3000
# Login: admin
# Password: admin
```

5. **Создай System Monitoring Dashboard** вручную:

**Panel 1: CPU Usage**
- Visualization: Time series
- Query: `100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)`
- Legend: CPU Usage %
- Unit: Percent (0-100)
- Threshold: Yellow at 70, Red at 90

**Panel 2: Memory Usage**
- Visualization: Time series
- Query: `(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100`
- Legend: Memory Usage %
- Unit: Percent (0-100)

**Panel 3: Disk Usage**
- Visualization: Gauge
- Query: `100 - ((node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100)`
- Unit: Percent (0-100)
- Threshold: Green 0-70, Yellow 70-85, Red 85-100

**Panel 4: Network Traffic**
- Visualization: Time series
- Query A: `rate(node_network_receive_bytes_total[5m]) / 1024 / 1024`
- Query B: `rate(node_network_transmit_bytes_total[5m]) / 1024 / 1024`
- Unit: MB/s

**Panel 5: Top Processes by CPU**
- Visualization: Table
- Query: `topk(5, irate(process_cpu_seconds_total[5m]))`

6. **Создай переменные для dashboard**:
- Variable: instance
  - Type: Query
  - Query: `label_values(node_cpu_seconds_total, instance)`
  
Измени запросы на использование переменной:
```promql
100 - (avg(irate(node_cpu_seconds_total{instance="$instance", mode="idle"}[5m])) * 100)
```

### 🚀 Бонус (новое)

**Создай JSON dashboard через provisioning** (`grafana/provisioning/dashboards/system-overview.json`):
```json
{
  "dashboard": {
    "title": "System Overview",
    "tags": ["system", "monitoring"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 0},
        "type": "timeseries",
        "title": "CPU Usage",
        "targets": [
          {
            "expr": "100 - (avg(irate(node_cpu_seconds_total{instance=\"$instance\",mode=\"idle\"}[5m])) * 100)",
            "legendFormat": "CPU Usage %"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "thresholds": {
              "steps": [
                {"value": 0, "color": "green"},
                {"value": 70, "color": "yellow"},
                {"value": 90, "color": "red"}
              ]
            }
          }
        }
      }
    ],
    "templating": {
      "list": [
        {
          "name": "instance",
          "type": "query",
          "datasource": "Prometheus",
          "query": "label_values(node_cpu_seconds_total, instance)",
          "refresh": 1
        }
      ]
    }
  }
}
```

**Настрой Alerting в Grafana**:
1. Configuration → Alerting → Contact points
2. Создай Email contact point
3. Создай Alert rule:
   - Name: High CPU Alert
   - Query: `avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100 < 20`
   - Condition: WHEN last() OF query(A) IS BELOW 20
   - For: 5m

**Установи Grafana plugins**:
```bash
# Установка через UI
Configuration → Plugins → Search

# Полезные плагины:
- Pie Chart
- Worldmap Panel
- Clock Panel
- Status Panel

# Через CLI (в контейнере)
docker exec grafana grafana-cli plugins install grafana-piechart-panel
docker restart grafana
```

---

## Модуль 4: Логирование и централизация логов (30 минут)

### 🎯 Напоминалка

**Уровни логирования:**

```
TRACE   - Детальная информация для отладки
DEBUG   - Отладочная информация
INFO    - Информационные сообщения
WARN    - Предупреждения
ERROR   - Ошибки, не критичные для работы
FATAL   - Критические ошибки, приложение падает
```

**Structured logging (JSON):**

json

````json
{
  "timestamp": "2025-01-15T10:30:00Z",
  "level": "ERROR",
  "service": "api",
  "message": "Database connection failed",
  "error": "connection timeout",
  "user_id": "12345",
  "request_id": "abc-123",
  "duration_ms": 5000
}
````

**ELK Stack:**
```
Elasticsearch  - Хранение и поиск
Logstash       - Обработка и парсинг
Kibana         - Визуализация
```

**Alternative: Loki Stack:**
```
Loki           - Хранение логов (как Prometheus для логов)
Promtail       - Агент сбора (как node-exporter)
Grafana        - Визуализация
```

**Log aggregation patterns:**
````
┌──────────┐
│   App    │────┐
└──────────┘    │
                │    ┌─────────┐    ┌──────────────┐
┌──────────┐    ├───►│ Log     │───►│ Centralized  │
│   App    │────┤    │ Shipper │    │ Log Storage  │
└──────────┘    │    └─────────┘    └──────────────┘
                │
┌──────────┐    │
│   App    │────┘
└──────────┘
````

**Полезные команды для логов:**

bash

```bash
# journalctl (systemd)
journalctl -u nginx                  # Логи сервиса
journalctl -f                        # Follow логи
journalctl --since "1 hour ago"
journalctl -p err                    # Только ошибки
journalctl --disk-usage              # Размер логов

# Docker logs
docker logs <container>
docker logs -f <container>
docker logs --tail 100 <container>
docker logs --since 1h <container>

# Традиционные логи Linux
tail -f /var/log/syslog
tail -f /var/log/nginx/access.log
grep "ERROR" /var/log/application.log
zgrep "pattern" /var/log/old.log.gz  # Поиск в сжатых логах

# Логи с временными метками
tail -f /var/log/app.log | ts '%Y-%m-%d %H:%M:%S'

# Многофайловый tail
multitail /var/log/nginx/access.log /var/log/nginx/error.log

# Анализ логов
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10  # Top 10 IP
grep "500" access.log | wc -l  # Количество 500 ошибок
```

**Log rotation:**

bash

```bash
# logrotate конфигурация (/etc/logrotate.d/app)
/var/log/app/*.log {
    daily                # Ротация каждый день
    rotate 7             # Хранить 7 архивов
    compress             # Сжимать старые
    delaycompress        # Не сжимать последний
    missingok            # Не ошибаться если файла нет
    notifempty           # Не ротировать пустые
    create 0640 app app  # Создать с правами
    sharedscripts
    postrotate
        systemctl reload app > /dev/null
    endscript
}

# Тестирование
logrotate -d /etc/logrotate.d/app    # Dry run
logrotate -f /etc/logrotate.d/app    # Принудительная ротация
```

**Логирование в приложениях:**

**Python (structured logging):**

python

```python
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            "timestamp": datetime.utcnow().isoformat(),
            "level": record.levelname,
            "service": "my-api",
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno
        }
        if record.exc_info:
            log_data["exception"] = self.formatException(record.exc_info)
        return json.dumps(log_data)

logging.basicConfig(level=logging.INFO)
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger = logging.getLogger()
logger.handlers = [handler]

# Использование
logger.info("User logged in", extra={"user_id": "123", "ip": "192.168.1.1"})
logger.error("Database error", extra={"query": "SELECT *", "duration_ms": 5000})
```

**Node.js (Winston):**

javascript

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  defaultMeta: { service: 'api-service' },
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
    new winston.transports.Console({
      format: winston.format.simple()
    })
  ]
});

// Использование
logger.info('User action', { user_id: '123', action: 'login' });
logger.error('Database error', { error: err.message, query: sql });
```

**Loki query patterns (LogQL):**

logql

```logql
# Базовый поиск
{job="varlogs"}

# Фильтры
{job="varlogs"} |= "error"                    # Содержит "error"
{job="varlogs"} != "debug"                    # Не содержит "debug"
{job="varlogs"} |~ "error|ERROR"              # Regex
{job="varlogs"} !~ "info|INFO"                # Негативный regex

# JSON parsing
{job="varlogs"} | json | level="error"
{job="varlogs"} | json | response_time > 1000

# Агрегация
rate({job="varlogs"}[5m])                     # Лог-записей в секунду
sum(rate({job="varlogs"}[5m])) by (level)     # По уровню
count_over_time({job="varlogs"}[1h])          # Количество за час

# Pattern extraction
{job="varlogs"} | pattern `<_> level=<level> <_>`
{job="varlogs"} | regexp `status=(?P<status>\d+)`

# Метрики из логов
sum(rate({job="api"} | json | status="500" [5m]))
```

**Elasticsearch query patterns:**

json

```json
// Базовый поиск
GET /logs-*/_search
{
  "query": {
    "match": {
      "message": "error"
    }
  }
}

// Временной диапазон
GET /logs-*/_search
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-1h",
        "lte": "now"
      }
    }
  }
}

// Комбинированный запрос
GET /logs-*/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "level": "ERROR" }},
        { "match": { "service": "api" }}
      ],
      "filter": [
        { "range": { "@timestamp": { "gte": "now-1h" }}}
      ]
    }
  },
  "aggs": {
    "errors_by_service": {
      "terms": { "field": "service.keyword" }
    }
  }
}
```

**Fluentd/Fluent Bit basics:**

conf

````conf
# Fluentd конфигурация (fluent.conf)
<source>
  @type tail
  path /var/log/nginx/access.log
  pos_file /var/log/td-agent/nginx-access.log.pos
  tag nginx.access
  <parse>
    @type nginx
  </parse>
</source>

<filter nginx.access>
  @type record_transformer
  <record>
    hostname "#{Socket.gethostname}"
    service "nginx"
  </record>
</filter>

<match nginx.access>
  @type elasticsearch
  host elasticsearch
  port 9200
  index_name nginx-access
  type_name _doc
</match>

# Fluent Bit конфигурация (более легковесная альтернатива)
[INPUT]
    Name              tail
    Path              /var/log/containers/*.log
    Parser            docker
    Tag               kube.*

[FILTER]
    Name                kubernetes
    Match               kube.*
    Kube_URL            https://kubernetes.default.svc:443
    Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token

[OUTPUT]
    Name              loki
    Match             *
    Host              loki
    Port              3100
```

**Log best practices:**
```
1. Всегда используй structured logging (JSON)
2. Включай контекст: request_id, user_id, trace_id
3. Логируй на правильном уровне:
   - DEBUG: детали для разработки
   - INFO: нормальные операции
   - WARN: потенциальные проблемы
   - ERROR: ошибки требующие внимания
4. Не логируй sensitive data (пароли, токены, PII)
5. Используй correlation IDs для трейсинга
6. Ротируй логи автоматически
7. Централизуй логи со всех систем
8. Настрой алерты на критичные паттерны
````

### 💻 Задание

Настрой централизованное логирование с Loki:

1. **Создай docker-compose.yml для Loki stack**:

yaml

```yaml
version: '3.8'

services:
  loki:
    image: grafana/loki:2.9.3
    container_name: loki
    ports:
      - "3100:3100"
    volumes:
      - ./loki-config.yml:/etc/loki/local-config.yaml
      - loki-data:/loki
    command: -config.file=/etc/loki/local-config.yaml
    restart: unless-stopped

  promtail:
    image: grafana/promtail:2.9.3
    container_name: promtail
    volumes:
      - ./promtail-config.yml:/etc/promtail/config.yml
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    command: -config.file=/etc/promtail/config.yml
    restart: unless-stopped
    depends_on:
      - loki

  grafana:
    image: grafana/grafana:10.2.3
    container_name: grafana-logs
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-logs-data:/var/lib/grafana
      - ./grafana-datasources.yml:/etc/grafana/provisioning/datasources/datasources.yml
    restart: unless-stopped
    depends_on:
      - loki

  # Тестовое приложение генерирующее логи
  log-generator:
    image: mingrammer/flog
    container_name: log-generator
    command: -f json -l -d 1 -s 1
    restart: unless-stopped

volumes:
  loki-data:
  grafana-logs-data:
```

2. **Создай loki-config.yml**:

yaml

```yaml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

common:
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory

query_range:
  results_cache:
    cache:
      embedded_cache:
        enabled: true
        max_size_mb: 100

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093

# Retention (удаление старых логов)
limits_config:
  retention_period: 168h  # 7 дней
```

3. **Создай promtail-config.yml**:

yaml

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  # Docker контейнеры
  - job_name: docker
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s
    relabel_configs:
      - source_labels: ['__meta_docker_container_name']
        regex: '/(.*)'
        target_label: 'container'
      - source_labels: ['__meta_docker_container_log_stream']
        target_label: 'stream'
    pipeline_stages:
      - json:
          expressions:
            level: level
            message: message
            timestamp: timestamp
      - labels:
          level:
          stream:

  # Системные логи
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log

  # Application logs (с парсингом JSON)
  - job_name: app-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: app
          __path__: /var/log/app/*.log
    pipeline_stages:
      - json:
          expressions:
            timestamp: timestamp
            level: level
            service: service
            message: message
            user_id: user_id
      - timestamp:
          source: timestamp
          format: RFC3339
      - labels:
          level:
          service:
```

4. **Создай grafana-datasources.yml**:

yaml

```yaml
apiVersion: 1

datasources:
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    isDefault: true
    editable: true
    jsonData:
      maxLines: 1000
```

5. **Запусти stack**:

bash

```bash
# Создай директории
mkdir -p logs/app

# Запусти
docker-compose up -d

# Проверь статус
docker-compose ps
curl http://localhost:3100/ready

# Проверь логи
curl http://localhost:3100/loki/api/v1/label
```

6. **Создай Python скрипт для генерации тестовых логов** (`generate_logs.py`):

python

````python
#!/usr/bin/env python3
import json
import random
import time
from datetime import datetime

levels = ['DEBUG', 'INFO', 'WARN', 'ERROR']
services = ['api', 'frontend', 'database', 'cache']
messages = {
    'DEBUG': ['Query executed', 'Cache hit', 'Function called'],
    'INFO': ['User logged in', 'Request processed', 'Task completed'],
    'WARN': ['Slow query detected', 'High memory usage', 'Rate limit approaching'],
    'ERROR': ['Database connection failed', 'Timeout occurred', '500 Internal Server Error']
}

def generate_log():
    level = random.choices(levels, weights=[10, 60, 20, 10])[0]
    service = random.choice(services)
    message = random.choice(messages[level])
    
    log_entry = {
        "timestamp": datetime.utcnow().isoformat() + "Z",
        "level": level,
        "service": service,
        "message": message,
        "request_id": f"req-{random.randint(1000, 9999)}",
        "user_id": f"user-{random.randint(1, 100)}",
        "duration_ms": random.randint(10, 5000) if level in ['WARN', 'ERROR'] else random.randint(10, 500)
    }
    
    return json.dumps(log_entry)

if __name__ == "__main__":
    print("Starting log generation...")
    while True:
        log = generate_log()
        print(log)
        # Сохранение в файл
        with open('/var/log/app/application.log', 'a') as f:
            f.write(log + '\n')
        time.sleep(random.uniform(0.1, 2))
````

7. **Открой Grafana и создай dashboard**:
```
URL: http://localhost:3001
Login: admin
Password: admin

Примеры запросов для панелей:

# Общее количество логов по уровню
sum(rate({job="docker"}[1m])) by (level)

# Логи с ошибками
{job="docker"} |= "ERROR"

# Top services по количеству логов
topk(5, sum(rate({job="docker"}[5m])) by (container))

# Логи конкретного сервиса
{job="docker", container="log-generator"}

# Медленные запросы (если duration > 1000ms)
{job="docker"} | json | duration_ms > 1000
````

8. **Проверь работу**:

bash

```bash
# Логи в Loki
curl -G -s "http://localhost:3100/loki/api/v1/query" \
  --data-urlencode 'query={job="docker"}' | jq

# Количество логов
curl -G -s "http://localhost:3100/loki/api/v1/query" \
  --data-urlencode 'query=count_over_time({job="docker"}[1h])' | jq

# Метрики Promtail
curl http://localhost:9080/metrics
```

### 🚀 Бонус (новое)

**1. Настрой ELK Stack для сравнения**:

`docker-compose-elk.yml`:

yaml

```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.3
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    restart: unless-stopped

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.3
    container_name: logstash
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5000:5000"
      - "9600:9600"
    environment:
      - "LS_JAVA_OPTS=-Xmx256m -Xms256m"
    depends_on:
      - elasticsearch
    restart: unless-stopped

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.3
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch
    restart: unless-stopped

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.11.3
    container_name: filebeat
    user: root
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    command: filebeat -e -strict.perms=false
    depends_on:
      - elasticsearch
    restart: unless-stopped

volumes:
  elasticsearch-data:
```

`logstash.conf`:

conf

```conf
input {
  beats {
    port => 5000
  }
}

filter {
  if [message] =~ /^\{.*\}$/ {
    json {
      source => "message"
    }
  }
  
  date {
    match => ["timestamp", "ISO8601"]
    target => "@timestamp"
  }
  
  mutate {
    remove_field => ["message"]
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  
  stdout {
    codec => rubydebug
  }
}
```

`filebeat.yml`:

yaml

```yaml
filebeat.inputs:
  - type: container
    paths:
      - '/var/lib/docker/containers/*/*.log'
    processors:
      - add_docker_metadata:
          host: "unix:///var/run/docker.sock"

output.logstash:
  hosts: ["logstash:5000"]

logging.level: info
```

**2. Создай log alerting rules**:

Для Loki (через Grafana Alerting):

yaml

```yaml
# Alert: High Error Rate
groups:
  - name: log_alerts
    interval: 1m
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate({job="docker"} |= "ERROR" [5m])) > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} errors/sec"

      - alert: ServiceDown
        expr: |
          absent(rate({job="docker", container="api"}[5m]))
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.container }} is down"
```

**3. Настрой log parsing для сложных форматов**:

Nginx access log parsing в Promtail:

yaml

````yaml
- job_name: nginx
  static_configs:
    - targets:
        - localhost
      labels:
        job: nginx
        __path__: /var/log/nginx/access.log
  pipeline_stages:
    - regex:
        expression: '^(?P<remote_addr>[\w\.]+) - (?P<remote_user>[^ ]*) \[(?P<time_local>.*)\] "(?P<method>[^ ]*) (?P<request>[^ ]*) (?P<protocol>[^ ]*)" (?P<status>[\d]+) (?P<body_bytes_sent>[\d]+) "(?P<http_referer>[^"]*)" "(?P<http_user_agent>[^"]*)"'
    - labels:
        method:
        status:
    - timestamp:
        source: time_local
        format: 02/Jan/2006:15:04:05 -0700
```

**4. Создай log analysis dashboard**:

Grafana panels для анализа логов:
```
Panel 1: Log volume over time
Query: sum(rate({job="docker"}[1m])) by (level)
Visualization: Time series

Panel 2: Top error messages
Query: topk(10, sum(rate({job="docker"} |= "ERROR" [5m])) by (message))
Visualization: Bar chart

Panel 3: Logs table
Query: {job="docker"}
Visualization: Logs

Panel 4: Response time distribution
Query: quantile_over_time(0.95, {job="docker"} | json | unwrap duration_ms [5m])
Visualization: Gauge

Panel 5: Service health
Query: count(rate({job="docker"}[1m])) by (container)
Visualization: Stat
````

**5. Настрой log sampling для высоконагруженных систем**:

yaml

```yaml
# Promtail sampling configuration
scrape_configs:
  - job_name: high-volume-app
    static_configs:
      - targets:
          - localhost
        labels:
          job: app
          __path__: /var/log/app/*.log
    pipeline_stages:
      # Сохраняй только ERROR и WARN + sample INFO/DEBUG
      - match:
          selector: '{job="app"}'
          stages:
            - json:
                expressions:
                  level: level
            - drop:
                expression: "level == 'DEBUG' and __sample__ > 0.1"  # 10% DEBUG
            - drop:
                expression: "level == 'INFO' and __sample__ > 0.5"   # 50% INFO
```

**6. Log retention и archiving**:

yaml

```yaml
# Loki retention config
limits_config:
  retention_period: 168h  # 7 дней

# Compactor для очистки старых логов
compactor:
  working_directory: /loki/compactor
  shared_store: filesystem
  compaction_interval: 10m
  retention_enabled: true
  retention_delete_delay: 2h
  retention_delete_worker_count: 150
```

**7. Интеграция с Alertmanager**:

yaml

```yaml
# Loki ruler config для отправки алертов
ruler:
  storage:
    type: local
    local:
      directory: /loki/rules
  rule_path: /tmp/rules
  alertmanager_url: http://alertmanager:9093
  ring:
    kvstore:
      store: inmemory
  enable_api: true
```

Rules file (`/loki/rules/alerts.yml`):

yaml

````yaml
groups:
  - name: logs
    interval: 1m
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate({job="docker"} |= "ERROR" [5m])) > 1
        for: 5m
        labels:
          severity: warning
          team: backend
        annotations:
          summary: "High error rate in {{ $labels.container }}"
          description: "Error rate: {{ $value }} errors/sec"
          dashboard: "http://grafana:3000/d/logs"
```

**8. Сравнение Loki vs ELK**:
```
Loki преимущества:
✅ Легковесный (меньше ресурсов)
✅ Интеграция с Prometheus/Grafana
✅ Простая конфигурация
✅ Хорошо для Kubernetes
✅ Дешевле в эксплуатации

ELK преимущества:
✅ Мощный полнотекстовый поиск
✅ Богатые возможности индексации
✅ Advanced analytics
✅ Больше плагинов и интеграций
✅ Mature ecosystem

Выбор:
- Loki: для метрик-ориентированного подхода, K8s
- ELK: для сложного анализа логов, compliance
````

---

## Итоги модуля 4

После прохождения этого модуля ты должен уметь:

✅ Понимать различные подходы к логированию 
✅ Настраивать Loki + Promtail + Grafana
✅ Писать LogQL запросы 
✅ Парсить различные форматы логов 
✅ Создавать дашборды для анализа логов 
✅ Настраивать алерты на основе логов 
✅ Управлять retention и rotation 
✅ Сравнивать Loki и ELK стеки

**Следующий модуль:** Alerting и Notification (как настроить умные алерты и избежать alert fatigue)


## Модуль 5: Alerting и Notification - умные алерты без alert fatigue (35 минут)

### 🎯 Напоминалка

**Философия алертинга:**

```
Хороший алерт = Actionable + Urgent + Real Problem

❌ Плохой алерт: "CPU usage > 80%"
✅ Хороший алерт: "API response time > 1s for 5min, affecting users"

Правило: Если алерт не требует действия прямо сейчас - это не алерт, это метрика
```

**Уровни severity:**

```
CRITICAL (P1)  - Полный outage, требует немедленных действий
                 Пример: сервис недоступен, потеря данных

WARNING (P2)   - Деградация сервиса, требует действий в ближайшее время
                 Пример: высокая latency, скоро закончится место

INFO (P3)      - Информационное уведомление, не требует срочных действий
                 Пример: deployment завершен, плановое обслуживание
```

**Alertmanager архитектура:**

```
┌─────────────┐
│ Prometheus  │─┐
└─────────────┘ │
                ├──► ┌──────────────┐     ┌─────────────┐
┌─────────────┐ │    │ Alertmanager │────►│ Receivers   │
│    Loki     │─┤    │              │     │ (Slack/etc) │
└─────────────┘ │    │ - Grouping   │     └─────────────┘
                │    │ - Inhibition │
┌─────────────┐ │    │ - Silencing  │
│   Custom    │─┘    │ - Routing    │
└─────────────┘      └──────────────┘
```

**Alert states:**

```
Inactive  ──► Pending  ──► Firing  ──► Resolved
               (for)         │
                            ↓
                         Silenced
```

**Ключевые концепции:**

**1. Grouping** - объединение похожих алертов:

yaml

```yaml
# Вместо 100 алертов о down нодах
# Один grouped алерт: "50 nodes are down in cluster-prod"
route:
  group_by: ['alertname', 'cluster']
  group_wait: 30s
  group_interval: 5m
```

**2. Inhibition** - подавление зависимых алертов:

yaml

```yaml
# Если кластер down, не слать алерты о каждом сервисе в нем
inhibit_rules:
  - source_match:
      alertname: ClusterDown
    target_match:
      cluster: production
    equal: ['cluster']
```

**3. Silencing** - временное отключение алертов:

bash

```bash
# Во время maintenance window
amtool silence add alertname=HighCPU --duration=2h --comment="Planned maintenance"
```

**4. Routing** - маршрутизация по командам/каналам:

yaml

```yaml
route:
  routes:
    - match:
        team: backend
      receiver: backend-team
    - match:
        severity: critical
      receiver: pagerduty
```

**Prometheus alerting rules структура:**

yaml

```yaml
groups:
  - name: example
    interval: 30s
    rules:
    - alert: HighErrorRate
      expr: |
        rate(http_requests_total{status=~"5.."}[5m]) 
        / 
        rate(http_requests_total[5m]) 
        > 0.05
      for: 5m
      labels:
        severity: warning
        team: backend
        service: api
      annotations:
        summary: "High error rate on {{ $labels.instance }}"
        description: "Error rate is {{ $value | humanizePercentage }}"
        dashboard: "https://grafana.com/d/api-dashboard"
        runbook: "https://wiki.com/runbooks/high-error-rate"
```

**Alert best practices:**

**1. Название алерта (говорящее):**

yaml

```yaml
❌ alert: HighCPU
✅ alert: InstanceHighCPUUsage

❌ alert: Error
✅ alert: APIHighErrorRate5xx
```

**2. For clause (избегаем flapping):**

yaml

```yaml
# Не алертить на кратковременные спайки
for: 5m  # Алерт только если условие true 5 минут подряд
```

**3. Аннотации (полезный контекст):**

yaml

```yaml
annotations:
  summary: "Краткое описание проблемы"
  description: "{{ $labels.instance }} has {{ $value }}% CPU usage"
  dashboard: "Ссылка на dashboard"
  runbook: "Ссылка на runbook с решением"
  impact: "Users experiencing slow response times"
```

**4. Labels для routing:**

yaml

```yaml
labels:
  severity: critical|warning|info
  team: backend|frontend|data
  service: api|web|worker
  environment: prod|staging|dev
```

**Типичные алерты инфраструктуры:**

yaml

```yaml
# Instance down
- alert: InstanceDown
  expr: up == 0
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "Instance {{ $labels.instance }} down"

# High CPU
- alert: HighCPUUsage
  expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
  for: 10m
  labels:
    severity: warning

# High Memory
- alert: HighMemoryUsage
  expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
  for: 5m
  labels:
    severity: warning

# Disk space low
- alert: DiskSpaceLow
  expr: (1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100 > 85
  for: 5m
  labels:
    severity: warning

# High disk I/O
- alert: HighDiskIO
  expr: rate(node_disk_io_time_seconds_total[5m]) > 0.9
  for: 10m
  labels:
    severity: warning
```

**Типичные алерты приложений:**

yaml

````yaml
# High error rate
- alert: HighErrorRate
  expr: |
    sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
    /
    sum(rate(http_requests_total[5m])) by (service)
    > 0.05
  for: 5m
  labels:
    severity: critical

# Slow response time
- alert: SlowResponseTime
  expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
  for: 10m
  labels:
    severity: warning

# High request rate (DDoS?)
- alert: UnusuallyHighTraffic
  expr: sum(rate(http_requests_total[5m])) > 1000
  for: 5m
  labels:
    severity: warning

# Database connection pool exhausted
- alert: DatabaseConnectionPoolNearLimit
  expr: database_connections_active / database_connections_max > 0.9
  for: 5m
  labels:
    severity: warning

# Queue backed up
- alert: QueueBacklog
  expr: queue_depth > 1000
  for: 10m
  labels:
    severity: warning

# Certificate expiring soon
- alert: CertificateExpiringSoon
  expr: (ssl_certificate_expiry_timestamp - time()) / 86400 < 30
  for: 1h
  labels:
    severity: warning
```

**Alert fatigue - как избежать:**
```
Проблема: Слишком много алертов → игнорируются → пропущены реальные проблемы

Решения:
1. ✅ Алертить только на симптомы, а не причины
   ❌ CPU high, Memory high, Disk full (причины)
   ✅ Users can't login, API is slow (симптомы)

2. ✅ Используй правильные threshold
   ❌ CPU > 50% (слишком чувствительно)
   ✅ CPU > 80% for 10 minutes (разумно)

3. ✅ Группируй похожие алерты
   ❌ 50 алертов "pod X down"
   ✅ 1 алерт "50 pods down in namespace Y"

4. ✅ Inhibition rules для зависимых алертов
   Если кластер down → не слать алерты о сервисах

5. ✅ Правильное время суток
   Non-critical алерты только в рабочее время

6. ✅ SLO-based alerting
   Алертить когда error budget исчерпывается

7. ✅ Регулярный review и cleanup
   Удаляй неактуальные алерты
```

**Notification channels:**
```
Критичность    Канал           Когда использовать
═══════════════════════════════════════════════════════════════
Critical       PagerDuty       Production outage, требует немедленного действия
               OpsGenie        
               
Warning        Slack           Требует внимания, но не срочно
               Teams           
               
Info           Email           FYI, статистика, отчеты
               Webhook         Интеграция с другими системами
               
Все уровни     Grafana         Для визуализации и анализа
````

**Alertmanager команды:**

bash

```bash
# Статус
amtool config show
amtool config routes
amtool alert query

# Silences
amtool silence add alertname=HighCPU --duration=2h --comment="Maintenance"
amtool silence query
amtool silence expire <silence-id>

# Проверка конфига
amtool check-config alertmanager.yml

# Отправка тестового алерта
amtool alert add alertname=Test severity=warning

# API запросы
curl -X GET http://localhost:9093/api/v2/alerts
curl -X GET http://localhost:9093/api/v2/silences
curl -X GET http://localhost:9093/api/v2/status
```

### 💻 Задание

Настрой полноценную систему алертинга:

1. **Добавь Alertmanager в docker-compose.yml**:

yaml

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alerts.yml:/etc/prometheus/alerts.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
      - alertmanager-data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    command:
      - '--path.rootfs=/host'
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'

  # Webhook receiver для тестирования
  webhook-receiver:
    image: ghcr.io/tarampampam/webhook-tester:latest
    container_name: webhook-receiver
    ports:
      - "8080:8080"
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_UNIFIED_ALERTING_ENABLED=true
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana-datasources.yml:/etc/grafana/provisioning/datasources/datasources.yml
    restart: unless-stopped
    depends_on:
      - prometheus

volumes:
  prometheus-data:
  alertmanager-data:
  grafana-data:
```

2. **Создай prometheus.yml с алертингом**:

yaml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'local'
    environment: 'dev'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

# Load rules
rule_files:
  - "alerts.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'alertmanager'
    static_configs:
      - targets: ['alertmanager:9093']
```

3. **Создай alerts.yml с правилами**:

yaml

```yaml
groups:
  - name: infrastructure
    interval: 30s
    rules:
    # Instance down
    - alert: InstanceDown
      expr: up == 0
      for: 2m
      labels:
        severity: critical
        team: infrastructure
      annotations:
        summary: "Instance {{ $labels.instance }} is down"
        description: "{{ $labels.job }} on {{ $labels.instance }} has been down for more than 2 minutes."
        dashboard: "http://localhost:3000/d/node-exporter"
        runbook: "https://runbooks.example.com/InstanceDown"

    # High CPU
    - alert: HighCPUUsage
      expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
      for: 5m
      labels:
        severity: warning
        team: infrastructure
      annotations:
        summary: "High CPU usage on {{ $labels.instance }}"
        description: "CPU usage is {{ $value | humanize }}% on {{ $labels.instance }}"
        dashboard: "http://localhost:3000/d/node-exporter"

    # High Memory
    - alert: HighMemoryUsage
      expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
      for: 5m
      labels:
        severity: warning
        team: infrastructure
      annotations:
        summary: "High memory usage on {{ $labels.instance }}"
        description: "Memory usage is {{ $value | humanize }}% on {{ $labels.instance }}"

    # Disk space critical
    - alert: DiskSpaceCritical
      expr: (1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100 > 90
      for: 5m
      labels:
        severity: critical
        team: infrastructure
      annotations:
        summary: "Critical disk space on {{ $labels.instance }}"
        description: "Disk usage is {{ $value | humanize }}% on {{ $labels.instance }}"
        impact: "System may become unresponsive if disk fills up"

    # Disk space warning
    - alert: DiskSpaceWarning
      expr: (1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100 > 80
      for: 10m
      labels:
        severity: warning
        team: infrastructure
      annotations:
        summary: "Low disk space on {{ $labels.instance }}"
        description: "Disk usage is {{ $value | humanize }}% on {{ $labels.instance }}"

  - name: alertmanager
    interval: 30s
    rules:
    # Alertmanager down
    - alert: AlertmanagerDown
      expr: up{job="alertmanager"} == 0
      for: 2m
      labels:
        severity: critical
        team: monitoring
      annotations:
        summary: "Alertmanager is down"
        description: "Alertmanager has been down for more than 2 minutes. Alerts may not be delivered!"

    # Too many alerts firing
    - alert: TooManyAlerts
      expr: count(ALERTS{alertstate="firing"}) > 10
      for: 5m
      labels:
        severity: warning
        team: monitoring
      annotations:
        summary: "Too many alerts firing"
        description: "There are {{ $value }} alerts currently firing. This may indicate a systemic issue."

  - name: prometheus
    interval: 30s
    rules:
    # Prometheus target missing
    - alert: PrometheusTargetMissing
      expr: up == 0
      for: 2m
      labels:
        severity: critical
        team: monitoring
      annotations:
        summary: "Prometheus target missing"
        description: "A Prometheus target has disappeared. Instance: {{ $labels.instance }}"

    # Prometheus config reload failed
    - alert: PrometheusConfigReloadFailed
      expr: prometheus_config_last_reload_successful == 0
      for: 5m
      labels:
        severity: critical
        team: monitoring
      annotations:
        summary: "Prometheus config reload failed"
        description: "Prometheus config reload has failed on {{ $labels.instance }}"

  - name: deadman
    interval: 30s
    rules:
    # Deadman switch - алерт который всегда должен firing
    - alert: DeadMansSwitch
      expr: vector(1)
      labels:
        severity: info
        team: monitoring
      annotations:
        summary: "Monitoring system is alive"
        description: "This is a deadman switch. It should always be firing. If you don't receive this, monitoring is broken."
```

4. **Создай alertmanager.yml с routing и receivers**:

yaml

```yaml
global:
  resolve_timeout: 5m
  # Slack (раскомментируй и настрой при необходимости)
  # slack_api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'

# Templates для красивых сообщений
templates:
  - '/etc/alertmanager/templates/*.tmpl'

# Route tree
route:
  receiver: 'default'
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 5m
  repeat_interval: 4h
  
  routes:
    # Critical алерты → webhook + log
    - match:
        severity: critical
      receiver: critical-alerts
      group_wait: 10s
      repeat_interval: 1h
      continue: true

    # Infrastructure team
    - match:
        team: infrastructure
      receiver: infrastructure-team
      group_wait: 30s
      repeat_interval: 4h

    # Monitoring team
    - match:
        team: monitoring
      receiver: monitoring-team

    # Deadman switch (для проверки что alerting работает)
    - match:
        alertname: DeadMansSwitch
      receiver: deadman
      repeat_interval: 5m

# Inhibition rules (подавление зависимых алертов)
inhibit_rules:
  # Если instance down, не слать другие алерты с того же instance
  - source_match:
      severity: critical
      alertname: InstanceDown
    target_match:
      severity: warning
    equal: ['instance']

  # Если диск критичен, не слать warning о диске
  - source_match:
      alertname: DiskSpaceCritical
    target_match:
      alertname: DiskSpaceWarning
    equal: ['instance', 'mountpoint']

# Receivers (каналы уведомлений)
receivers:
  - name: 'default'
    webhook_configs:
      - url: 'http://webhook-receiver:8080/webhook/default'
        send_resolved: true

  - name: 'critical-alerts'
    webhook_configs:
      - url: 'http://webhook-receiver:8080/webhook/critical'
        send_resolved: true
    # Uncomment for Slack
    # slack_configs:
    #   - channel: '#alerts-critical'
    #     title: '🚨 CRITICAL ALERT'
    #     text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
    #     send_resolved: true

  - name: 'infrastructure-team'
    webhook_configs:
      - url: 'http://webhook-receiver:8080/webhook/infrastructure'
        send_resolved: true
    # Uncomment for Slack
    # slack_configs:
    #   - channel: '#team-infrastructure'
    #     title: '⚠️ Infrastructure Alert'
    #     text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'

  - name: 'monitoring-team'
    webhook_configs:
      - url: 'http://webhook-receiver:8080/webhook/monitoring'
        send_resolved: true

  - name: 'deadman'
    webhook_configs:
      - url: 'http://webhook-receiver:8080/webhook/deadman'
        send_resolved: false
```

5. **Создай grafana-datasources.yml**:

yaml

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
    jsonData:
      httpMethod: POST
      
  - name: Alertmanager
    type: alertmanager
    access: proxy
    url: http://alertmanager:9093
    editable: true
    jsonData:
      implementation: prometheus
```

6. **Запусти и протестируй**:

bash

```bash
# Запуск
docker-compose up -d

# Проверка Prometheus
curl http://localhost:9090/api/v1/rules

# Проверка Alertmanager
curl http://localhost:9093/api/v2/status

# Проверка алертов в Prometheus
curl http://localhost:9090/api/v1/alerts | jq

# Список firing алертов
curl http://localhost:9093/api/v2/alerts | jq '.[] | select(.status.state == "active")'
```

7. **Создай скрипт для генерации тестовой нагрузки** (`stress_test.sh`):

bash

```bash
#!/bin/bash

echo "Starting stress test to trigger alerts..."

# CPU stress (триггернет HighCPUUsage)
echo "Generating CPU load..."
docker run --rm --name cpu-stress \
  polinux/stress \
  stress --cpu 4 --timeout 300s &

# Заполнение диска (для DiskSpaceWarning)
# ВНИМАНИЕ: Будь осторожен с этим на проде!
# echo "Filling disk space..."
# dd if=/dev/zero of=/tmp/largefile bs=1M count=10000

echo "Stress test running. Check alerts in:"
echo "- Prometheus: http://localhost:9090/alerts"
echo "- Alertmanager: http://localhost:9093"
echo "- Webhook receiver: http://localhost:8080"
echo ""
echo "Wait 5-10 minutes for alerts to fire..."
```

8. **Проверь UI и алерты**:

bash

```bash
# Prometheus Alerts UI
open http://localhost:9090/alerts

# Alertmanager UI
open http://localhost:9093

# Grafana Alerting
open http://localhost:3000/alerting/list

# Webhook receiver (проверь полученные алерты)
open http://localhost:8080
```

9. **Протестируй silencing**:

bash

```bash
# Установи amtool
go install github.com/prometheus/alertmanager/cmd/amtool@latest
# или
brew install amtool

# Настрой amtool
cat > ~/.config/amtool/config.yml <<EOF
alertmanager.url: http://localhost:9093
EOF

# Создай silence на время теста
amtool silence add \
  alertname=HighCPUUsage \
  --duration=1h \
  --comment="Testing alert system" \
  --author="devops@example.com"

# Проверь silences
amtool silence query

# Удали silence
amtool silence expire <silence-id>
```

### 🚀 Бонус (новое)

**1. Интеграция со Slack**:

Обнови `alertmanager.yml`:

yaml

```yaml
global:
  slack_api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'

receivers:
  - name: 'slack-critical'
    slack_configs:
      - channel: '#alerts-critical'
        username: 'Alertmanager'
        icon_emoji: ':fire:'
        title: '🚨 {{ .GroupLabels.alertname }}'
        text: |
          {{ range .Alerts }}
          *Alert:* {{ .Labels.alertname }}
          *Severity:* {{ .Labels.severity }}
          *Instance:* {{ .Labels.instance }}
          *Description:* {{ .Annotations.description }}
          *Dashboard:* {{ .Annotations.dashboard }}
          {{ end }}
        send_resolved: true
        color: '{{ if eq .Status "firing" }}danger{{ else }}good{{ end }}'
```

**2. Custom notification template**:

Создай `templates/slack.tmpl`:

gotmpl

```gotmpl
{{ define "slack.title" }}
[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .GroupLabels.alertname }}
{{ end }}

{{ define "slack.text" }}
{{ range .Alerts }}
*Alert:* {{ .Labels.alertname }} - `{{ .Labels.severity }}`
*Instance:* {{ .Labels.instance }}
*Summary:* {{ .Annotations.summary }}
*Description:* {{ .Annotations.description }}
{{ if .Annotations.runbook }}*Runbook:* {{ .Annotations.runbook }}{{ end }}
{{ if .Annotations.dashboard }}*Dashboard:* {{ .Annotations.dashboard }}{{ end }}
*Started:* {{ .StartsAt.Format "2006-01-02 15:04:05 MST" }}
{{ if .EndsAt }}*Ended:* {{ .EndsAt.Format "2006-01-02 15:04:05 MST" }}{{ end }}
{{ end }}
{{ end }}

{{ define "slack.color" }}
{{ if eq .Status "firing" }}
  {{ if eq .CommonLabels.severity "critical" }}danger{{ else }}warning{{ end }}
{{ else }}
good
{{ end }}
{{ end }}
```

**3. PagerDuty интеграция** (для critical alerts):

yaml

```yaml
receivers:
  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_SERVICE_KEY'
        description: '{{ .GroupLabels.alertname }}: {{ .Annotations.summary }}'
        severity: '{{ .CommonLabels.severity }}'
        details:
          firing: '{{ .Alerts.Firing | len }}'
          resolved: '{{ .Alerts.Resolved | len }}'
          instance: '{{ .CommonLabels.instance }}'
        client: 'Alertmanager'
        client_url: 'http://alertmanager:9093'
        send_resolved: true
```

**4. Email notifications с HTML template**:

yaml

```yaml
receivers:
  - name: 'email-team'
    email_configs:
      - to: 'team@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'alertmanager@example.com'
        auth_password: 'your-app-password'
        headers:
          Subject: '{{ if eq .Status "firing" }}🚨{{ else }}✅{{ end }} [{{ .Status | toUpper }}] {{ .GroupLabels.alertname }}'
        html: |
          <!DOCTYPE html>
          <html>
          <body>
            <h2 style="color: {{ if eq .Status "firing" }}#d9534f{{ else }}#5cb85c{{ end }}">
              {{ if eq .Status "firing" }}🚨 Firing Alerts{{ else }}✅ Resolved{{ end }}
            </h2>
            {{ range .Alerts }}
            <div style="border-left: 4px solid {{ if eq .Status "firing" }}#d9534f{{ else }}#5cb85c{{ end }}; padding: 10px; margin: 10px 0;">
              <h3>{{ .Labels.alertname }}</h3>
              <p><strong>Severity:</strong> {{ .Labels.severity }}</p>
              <p><strong>Instance:</strong> {{ .Labels.instance }}</p>
              <p><strong>Description:</strong> {{ .Annotations.description }}</p>
              {{ if .Annotations.runbook }}
              <p><a href="{{ .Annotations.runbook }}">📖 Runbook</a></p>
              {{ end }}
              {{ if .Annotations.dashboard }}
              <p><a href="{{ .Annotations.dashboard }}">📊 Dashboard</a></p>
              {{ end }}
            </div>
            {{ end }}
          </body>
          </html>
        send_resolved: true
```

**5. Webhook для интеграции с Jira/ServiceNow**:

Создай `webhook_handler.py`:

python

```python
#!/usr/bin/env python3
from flask import Flask, request, jsonify
import json
import requests

app = Flask(__name__)

@app.route('/webhook/jira', methods=['POST'])
def jira_webhook():
    """Создает Jira ticket для критичных алертов"""
    data = request.json
    
    # Фильтруем только firing и critical
    if data['status'] == 'firing':
        for alert in data['alerts']:
            if alert['labels'].get('severity') == 'critical':
                create_jira_ticket(alert)
    
    return jsonify({'status': 'ok'}), 200

def create_jira_ticket(alert):
    """Создает Jira ticket через API"""
    jira_url = "https://your-jira.atlassian.net/rest/api/2/issue"
    
    ticket = {
        "fields": {
            "project": {"key": "OPS"},
            "summary": f"[ALERT] {alert['labels']['alertname']}",
            "description": alert['annotations']['description'],
            "issuetype": {"name": "Incident"}, 
            "priority": {"name": "Critical"}, 
            "labels": ["alert", "monitoring"] } }
            # Отправка в Jira
	response = requests.post(
		jira_url,
		json=ticket,
		auth=('user@example.com', 'jira-api-token'),
		headers={'Content-Type': 'application/json'}
	)
	print(f"Jira ticket created: {response.json().get('key')}")	
	if name == 'main': app.run(host='0.0.0.0', port=5000)
```
**6. SLO-based alerting** (продвинутый подход):
```yaml
groups:
  - name: slo_alerts
    interval: 30s
    rules:
    # Error budget burn rate
    - alert: ErrorBudgetBurnRateTooHigh
      expr: |
        (
          sum(rate(http_requests_total{status=~"5.."}[1h]))
          /
          sum(rate(http_requests_total[1h]))
        ) > (1 - 0.999) * 10  # 10x SLO burn rate
      for: 5m
      labels:
        severity: critical
        team: sre
      annotations:
        summary: "Error budget burning too fast"
        description: "Current error rate is {{ $value | humanizePercentage }}. At this rate, monthly error budget will be exhausted in {{ with printf \"(1-0.999)*730/%f\" $value }}{{ . }}{{ end }} hours."
        dashboard: "http://localhost:3000/d/slo-dashboard"

    # SLO violation
    - alert: SLOViolation
      expr: |
        (
          1 - (
            sum(rate(http_requests_total{status!~"5.."}[30d]))
            /
            sum(rate(http_requests_total[30d]))
          )
        ) > 0.001  # Нарушение 99.9% SLO
      for: 1h
      labels:
        severity: warning
        team: sre
      annotations:
        summary: "SLO violation detected"
        description: "30-day error rate is {{ $value | humanizePercentage }}, violating 99.9% SLO"
```

**7. Multi-window multi-burn-rate alerts** (Google SRE подход):
```yaml
groups:
  - name: multiwindow_multiburn_alerts
    interval: 30s
    rules:
    # Fast burn (нужно действовать немедленно)
    - alert: ErrorBudgetFastBurn
      expr: |
        (
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m]))
        ) > 14.4 * (1 - 0.999)  # 14.4x burn rate
        and
        (
          sum(rate(http_requests_total{status=~"5.."}[1h]))
          /
          sum(rate(http_requests_total[1h]))
        ) > 14.4 * (1 - 0.999)
      for: 2m
      labels:
        severity: critical
        burn_rate: fast
      annotations:
        summary: "Fast error budget burn"
        description: "Error budget will be exhausted in 2 hours at current rate"

    # Slow burn (требует внимания в ближайшее время)
    - alert: ErrorBudgetSlowBurn
      expr: |
        (
          sum(rate(http_requests_total{status=~"5.."}[30m]))
          /
          sum(rate(http_requests_total[30m]))
        ) > 6 * (1 - 0.999)  # 6x burn rate
        and
        (
          sum(rate(http_requests_total{status=~"5.."}[6h]))
          /
          sum(rate(http_requests_total[6h]))
        ) > 6 * (1 - 0.999)
      for: 15m
      labels:
        severity: warning
        burn_rate: slow
      annotations:
        summary: "Slow error budget burn"
        description: "Error budget will be exhausted in 5 days at current rate"
```

**8. Alert aggregation dashboard**:

Создай Python скрипт для анализа алертов (`alert_analysis.py`):
```python
#!/usr/bin/env python3
import requests
from collections import Counter
from datetime import datetime, timedelta

ALERTMANAGER_URL = "http://localhost:9093"

def get_alerts():
    """Получить все алерты из Alertmanager"""
    response = requests.get(f"{ALERTMANAGER_URL}/api/v2/alerts")
    return response.json()

def analyze_alerts():
    """Анализ паттернов алертов"""
    alerts = get_alerts()
    
    # Статистика
    total_alerts = len(alerts)
    firing_alerts = [a for a in alerts if a['status']['state'] == 'active']
    
    # По severity
    severity_counter = Counter(
        alert['labels'].get('severity', 'unknown') 
        for alert in firing_alerts
    )
    
    # По team
    team_counter = Counter(
        alert['labels'].get('team', 'unknown') 
        for alert in firing_alerts
    )
    
    # Самые частые алерты
    alert_counter = Counter(
        alert['labels']['alertname'] 
        for alert in firing_alerts
    )
    
    # Вывод отчета
    print("=" * 60)
    print("ALERT ANALYSIS REPORT")
    print("=" * 60)
    print(f"Total alerts: {total_alerts}")
    print(f"Firing alerts: {len(firing_alerts)}")
    print()
    
    print("By Severity:")
    for severity, count in severity_counter.most_common():
        print(f"  {severity}: {count}")
    print()
    
    print("By Team:")
    for team, count in team_counter.most_common():
        print(f"  {team}: {count}")
    print()
    
    print("Top 5 Most Frequent Alerts:")
    for alertname, count in alert_counter.most_common(5):
        print(f"  {alertname}: {count}")
    print("=" * 60)

if __name__ == "__main__":
    analyze_alerts()
```

**9. Alert testing framework**:

Создай `alert_test.py`:
```python
#!/usr/bin/env python3
"""
Тестирование алертов - отправляем тестовые метрики и проверяем
что алерты срабатывают
"""
import requests
import time
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway

def test_high_cpu_alert():
    """Тест алерта HighCPUUsage"""
    print("Testing HighCPUUsage alert...")
    
    registry = CollectorRegistry()
    cpu_gauge = Gauge('node_cpu_seconds_total', 
                      'CPU time', 
                      ['mode', 'instance'], 
                      registry=registry)
    
    # Симулируем высокую CPU нагрузку
    cpu_gauge.labels(mode='idle', instance='test-instance').set(0.1)
    cpu_gauge.labels(mode='user', instance='test-instance').set(0.8)
    
    # Push в Pushgateway
    push_to_gateway('localhost:9091', job='test', registry=registry)
    
    print("Metrics pushed. Wait 5 minutes and check alerts...")
    print("http://localhost:9090/alerts")

def test_disk_space_alert():
    """Тест алерта DiskSpaceCritical"""
    print("Testing DiskSpaceCritical alert...")
    
    registry = CollectorRegistry()
    disk_total = Gauge('node_filesystem_size_bytes',
                       'Filesystem size',
                       ['mountpoint', 'instance'],
                       registry=registry)
    disk_avail = Gauge('node_filesystem_avail_bytes',
                       'Available space',
                       ['mountpoint', 'instance'],
                       registry=registry)
    
    # Симулируем 95% использования диска
    disk_total.labels(mountpoint='/', instance='test-instance').set(100e9)  # 100GB
    disk_avail.labels(mountpoint='/', instance='test-instance').set(5e9)    # 5GB
    
    push_to_gateway('localhost:9091', job='test', registry=registry)
    
    print("Metrics pushed. Check alerts...")

if __name__ == "__main__":
    print("Starting alert tests...")
    test_high_cpu_alert()
    time.sleep(2)
    test_disk_space_alert()
    print("\nTests completed. Monitor alerts for next 10 minutes.")
```

**10. Alert maintenance calendar integration**:
```python
#!/usr/bin/env python3
"""
Автоматическое создание silences во время maintenance windows
"""
import requests
from datetime import datetime, timedelta

ALERTMANAGER_URL = "http://localhost:9093"

def create_maintenance_silence(service, duration_hours, comment):
    """Создать silence на время maintenance"""
    
    now = datetime.utcnow()
    starts_at = now.isoformat() + "Z"
    ends_at = (now + timedelta(hours=duration_hours)).isoformat() + "Z"
    
    silence = {
        "matchers": [
            {
                "name": "service",
                "value": service,
                "isRegex": False
            }
        ],
        "startsAt": starts_at,
        "endsAt": ends_at,
        "createdBy": "maintenance-script",
        "comment": comment
    }
    
    response = requests.post(
        f"{ALERTMANAGER_URL}/api/v2/silences",
        json=silence
    )
    
    if response.status_code == 200:
        silence_id = response.json()['silenceID']
        print(f"✅ Silence created: {silence_id}")
        print(f"   Service: {service}")
        print(f"   Duration: {duration_hours} hours")
        print(f"   Ends at: {ends_at}")
        return silence_id
    else:
        print(f"❌ Failed to create silence: {response.text}")
        return None

if __name__ == "__main__":
    # Пример: Maintenance на API сервисе на 2 часа
    create_maintenance_silence(
        service="api",
        duration_hours=2,
        comment="Planned database migration"
    )
```

---

## Итоги модуля 5

После прохождения этого модуля ты должен уметь:

✅ Настраивать Alertmanager с routing и inhibition
✅ Писать качественные alert rules в Prometheus
✅ Интегрировать с различными каналами уведомлений (Slack, PagerDuty, Email)
✅ Использовать grouping, inhibition и silencing
✅ Создавать SLO-based alerts
✅ Избегать alert fatigue через правильную настройку
✅ Тестировать и отлаживать alerts
✅ Создавать custom notification templates
✅ Автоматизировать maintenance windows

**Ключевые принципы алертинга:**
1. Alert на симптомы, а не на причины
2. Каждый алерт должен требовать действия
3. Используй правильные severity уровни
4. Группируй и подавляй зависимые алерты
5. Регулярно review и cleanup алертов
6. Документируй runbooks для каждого алерта
7. Тестируй алерты регулярно

**Следующий модуль:** Distributed Tracing и Application Performance Monitoring (APM)
## Модуль 6: Distributed Tracing и Application Performance Monitoring (40 минут)

### 🎯 Напоминалка

**Три столпа Observability:**

```
┌─────────────┐
│   METRICS   │  - Что происходит? (CPU, memory, requests/sec)
└─────────────┘
       │
┌─────────────┐
│    LOGS     │  - Что произошло? (события, ошибки)
└─────────────┘
       │
┌─────────────┐
│   TRACES    │  - Почему это произошло? (путь запроса через систему)
└─────────────┘
```

**Distributed Tracing - зачем нужен:**

```
Проблема в микросервисах:
User Request → API Gateway → Auth Service → Order Service → Payment Service → Database
                                                                   ↓
                                              ❌ SLOW RESPONSE (5 seconds)

Вопрос: Где bottleneck?
- API Gateway: 50ms
- Auth Service: 100ms
- Order Service: 200ms
- Payment Service: 4500ms ← НАЙДЕНО!
- Database: 150ms
```

**Основные концепции:**

**Trace** - полный путь одного запроса через систему:

```
Trace ID: abc123
├─ Span 1: API Gateway (50ms)
├─ Span 2: Auth Service (100ms)
├─ Span 3: Order Service (200ms)
│  ├─ Span 4: DB Query (50ms)
│  └─ Span 5: Cache Check (10ms)
└─ Span 6: Payment Service (4500ms)
   └─ Span 7: External API Call (4400ms) ← Проблема!
```

**Span** - единица работы в системе:

yaml

````yaml
Span:
  trace_id: "abc123"
  span_id: "span456"
  parent_span_id: "span789"
  operation_name: "POST /api/orders"
  start_time: "2025-01-15T10:00:00Z"
  duration: 200ms
  tags:
    http.method: "POST"
    http.status_code: 200
    service.name: "order-service"
    db.statement: "SELECT * FROM orders"
  logs:
    - timestamp: "2025-01-15T10:00:00.050Z"
      message: "Order validated"
```

**Популярные системы трейсинга:**
```
Jaeger       - CNCF проект, от Uber, Go
Zipkin       - От Twitter, Java
Tempo        - От Grafana Labs, интеграция с Loki
OpenTelemetry - Стандарт (объединение OpenTracing + OpenCensus)
AWS X-Ray    - Managed сервис от AWS
Datadog APM  - Commercial
New Relic    - Commercial
```

**OpenTelemetry (OTel) - современный стандарт:**
```
┌──────────────────────────────────┐
│     Your Application             │
│  ┌────────────────────────────┐  │
│  │  OpenTelemetry SDK         │  │
│  │  - Auto-instrumentation    │  │
│  │  - Manual instrumentation  │  │
│  └────────────┬───────────────┘  │
└───────────────┼──────────────────┘
                │
        ┌───────▼────────┐
        │ OTel Collector │ - Обработка, фильтрация
        └───────┬────────┘
                │
    ┌───────────┴───────────┐
    │                       │
┌───▼────┐            ┌─────▼──┐
│ Jaeger │            │ Tempo  │
└────────┘            └────────┘
```

**Sampling (выборка трейсов):**
```
Проблема: Нельзя хранить 100% трейсов (слишком дорого)

Виды sampling:
1. Head sampling (решение в начале)
   - Probabilistic: 10% всех трейсов
   - Rate limiting: 100 трейсов/сек
   
2. Tail sampling (решение в конце)
   - Все медленные запросы (> 1s)
   - Все запросы с ошибками
   - 1% нормальных запросов

Рекомендация: Tail sampling + всегда сохранять ошибки
```

**APM (Application Performance Monitoring) - что включает:**
```
1. Трейсинг (Distributed Tracing)
2. Профилирование (CPU, Memory profiling)
3. Error tracking
4. Real User Monitoring (RUM)
5. Database query analysis
6. External services monitoring
```

**Ключевые метрики APM:**

**RED метрики (для сервисов):**
```
Rate     - Requests per second
Error    - Error rate (%)
Duration - Request latency (p50, p95, p99)
```

**USE метрики (для ресурсов):**
```
Utilization - % времени занятости
Saturation  - Длина очереди
Errors      - Количество ошибок
```

**Service metrics:**
```
Apdex Score = (Satisfied + Tolerating/2) / Total Requests
- Satisfied: < 1s
- Tolerating: 1-4s
- Frustrated: > 4s

Throughput = Requests per second
Error Rate = Errors / Total Requests
Availability = Uptime / Total Time
````

**Context Propagation (как передается trace_id):**

**HTTP Headers:**

http

````http
# W3C Trace Context (стандарт)
traceparent: 00-abc123def456-span789-01
tracestate: vendor1=value1,vendor2=value2

# Jaeger
uber-trace-id: abc123:span456:0:1

# Zipkin
X-B3-TraceId: abc123
X-B3-SpanId: span456
X-B3-ParentSpanId: parent789
X-B3-Sampled: 1
```

**gRPC Metadata:**
```
grpc-trace-bin: <binary trace context>
````

**Instrumentation подходы:**

**Auto-instrumentation** (автоматический):

python

```python
# Python с OpenTelemetry
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor

FlaskInstrumentor().instrument()      # Автоматически Flask
RequestsInstrumentor().instrument()   # Автоматически requests
```

**Manual instrumentation** (ручной):

python

```python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

@app.route('/api/order')
def create_order():
    with tracer.start_as_current_span("create_order") as span:
        span.set_attribute("order.id", order_id)
        span.set_attribute("user.id", user_id)
        
        # Ваш код
        result = process_order(order_id)
        
        span.add_event("Order processed")
        return result
```

**Язык-специфичные библиотеки:**

**Python:**

python

```python
# OpenTelemetry
opentelemetry-api
opentelemetry-sdk
opentelemetry-instrumentation-flask
opentelemetry-instrumentation-django
opentelemetry-instrumentation-sqlalchemy
opentelemetry-exporter-jaeger
```

**Node.js:**

javascript

```javascript
// OpenTelemetry
@opentelemetry/api
@opentelemetry/sdk-node
@opentelemetry/auto-instrumentations-node
@opentelemetry/exporter-jaeger
```

**Go:**

go

```go
// OpenTelemetry
go.opentelemetry.io/otel
go.opentelemetry.io/otel/trace
go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp
```

**Java:**

java

````java
// OpenTelemetry Java Agent (auto-instrumentation)
java -javaagent:opentelemetry-javaagent.jar \
     -Dotel.service.name=my-service \
     -jar myapp.jar
```

**Jaeger UI - основные возможности:**
```
1. Search traces:
   - По service name
   - По operation name
   - По tags
   - По duration
   - По времени

2. Trace timeline:
   - Визуализация spans
   - Waterfall view
   - Gantt chart

3. Dependencies graph:
   - Карта зависимостей сервисов
   - Направление вызовов

4. Comparison:
   - Сравнение трейсов
   - A/B testing результаты
```

**Service Map (карта сервисов):**
```
┌────────────┐
│   User     │
└─────┬──────┘
      │ HTTP
┌─────▼──────┐
│ API Gateway│
└─────┬──────┘
      │
   ┌──┴───┬────────┐
   │      │        │
┌──▼──┐ ┌─▼───┐ ┌─▼────┐
│Auth │ │Order│ │User  │
│Svc  │ │Svc  │ │Svc   │
└──┬──┘ └──┬──┘ └──────┘
   │       │
   │   ┌───▼─────┐
   │   │Payment  │
   │   │Svc      │
   │   └───┬─────┘
   │       │
   └───┬───┴─────┐
       │         │
   ┌───▼──┐  ┌───▼──┐
   │ DB   │  │Cache │
   └──────┘  └──────┘
```

**Error tracking интеграция:**
```
Связь трейсов с ошибками:

Exception в коде → Trace ID → Полный путь запроса
                               + stack trace
                               + request params
                               + user context
```

**Database query analysis:**
```
Частые проблемы:
1. N+1 queries
   - 1 запрос списка + N запросов деталей
   
2. Missing indexes
   - Full table scan
   
3. Slow queries
   - Сложные JOIN
   - Большие SELECT *
   
4. Connection pool exhaustion
   - Не закрытые соединения
```

**Профилирование (CPU/Memory):**
```
Continuous Profiling:
- Flamegraph визуализация
- Какие функции занимают больше времени
- Memory allocations
- Goroutines/Threads

Инструменты:
- pprof (Go)
- py-spy (Python)
- async-profiler (Java)
- Pyroscope (unified)
````

**Real User Monitoring (RUM):**

javascript

````javascript
// Frontend трейсинг
import { WebTracerProvider } from '@opentelemetry/sdk-trace-web';

const provider = new WebTracerProvider();
const tracer = provider.getTracer('frontend-app');

// Track page load
const span = tracer.startSpan('page_load');
span.setAttribute('page.url', window.location.href);

window.addEventListener('load', () => {
  span.end();
});

// Track user interactions
button.addEventListener('click', () => {
  const span = tracer.startSpan('button_click');
  span.setAttribute('button.id', button.id);
  // ... действие
  span.end();
});
```

**Best practices:**
```
1. ✅ Всегда передавай trace context между сервисами
2. ✅ Добавляй полезные attributes (user_id, order_id, etc)
3. ✅ Логируй trace_id во всех логах
4. ✅ Используй semantic conventions (стандартные имена)
5. ✅ Настрой правильный sampling
6. ✅ Не логируй sensitive данные в spans
7. ✅ Используй tail sampling для ошибок
8. ✅ Храни трейсы минимум 7 дней
9. ✅ Интегрируй с алертингом
10. ✅ Создай runbook для распространенных паттернов
````

**Semantic Conventions (стандартные имена):**

yaml

```yaml
# HTTP
span.name: "GET /api/users"
http.method: "GET"
http.url: "https://api.example.com/users"
http.status_code: 200
http.route: "/api/users"

# Database
span.name: "SELECT users"
db.system: "postgresql"
db.operation: "SELECT"
db.statement: "SELECT * FROM users WHERE id = ?"
db.name: "production"

# RPC
span.name: "UserService.GetUser"
rpc.system: "grpc"
rpc.service: "UserService"
rpc.method: "GetUser"

# Messaging
span.name: "process_order"
messaging.system: "kafka"
messaging.destination: "orders"
messaging.operation: "process"
```

### 💻 Задание

Настрой полноценный distributed tracing с Jaeger:

1. **Создай docker-compose.yml для Jaeger stack**:

yaml

```yaml
version: '3.8'

services:
  # Jaeger all-in-one (для development)
  jaeger:
    image: jaegertracing/all-in-one:1.52
    container_name: jaeger
    environment:
      - COLLECTOR_ZIPKIN_HOST_PORT=:9411
      - COLLECTOR_OTLP_ENABLED=true
    ports:
      - "5775:5775/udp"   # accept zipkin.thrift (deprecated)
      - "6831:6831/udp"   # accept jaeger.thrift compact
      - "6832:6832/udp"   # accept jaeger.thrift binary
      - "5778:5778"       # serve configs
      - "16686:16686"     # Jaeger UI
      - "14250:14250"     # model.proto
      - "14268:14268"     # jaeger.thrift
      - "14269:14269"     # Admin port: health, metrics
      - "4317:4317"       # OTLP gRPC
      - "4318:4318"       # OTLP HTTP
      - "9411:9411"       # Zipkin compatible
    restart: unless-stopped

  # OpenTelemetry Collector (опционально, для обработки)
  otel-collector:
    image: otel/opentelemetry-collector-contrib:0.91.0
    container_name: otel-collector
    command: ["--config=/etc/otel-collector-config.yml"]
    volumes:
      - ./otel-collector-config.yml:/etc/otel-collector-config.yml
    ports:
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
      - "8888:8888"   # Prometheus metrics
      - "8889:8889"   # Prometheus exporter metrics
    restart: unless-stopped
    depends_on:
      - jaeger

  # Demo приложение - Frontend
  frontend:
    build: ./demo-app/frontend
    container_name: frontend
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
      - OTEL_SERVICE_NAME=frontend
      - BACKEND_URL=http://backend:5000
    ports:
      - "8080:8080"
    depends_on:
      - otel-collector
      - backend
    restart: unless-stopped

  # Demo приложение - Backend
  backend:
    build: ./demo-app/backend
    container_name: backend
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
      - OTEL_SERVICE_NAME=backend
      - DATABASE_URL=postgresql://user:password@postgres:5432/demo
      - REDIS_URL=redis://redis:6379
    ports:
      - "5000:5000"
    depends_on:
      - postgres
      - redis
      - otel-collector
    restart: unless-stopped

  # PostgreSQL database
  postgres:
    image: postgres:16-alpine
    container_name: postgres
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=demo
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    restart: unless-stopped

  # Redis cache
  redis:
    image: redis:7-alpine
    container_name: redis
    ports:
      - "6379:6379"
    restart: unless-stopped

  # Grafana для визуализации
  grafana:
    image: grafana/grafana:10.2.3
    container_name: grafana-tracing
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_FEATURE_TOGGLES_ENABLE=traceqlEditor
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana-datasources.yml:/etc/grafana/provisioning/datasources/datasources.yml
    restart: unless-stopped
    depends_on:
      - jaeger

volumes:
  postgres-data:
  grafana-data:
```

2. **Создай otel-collector-config.yml**:

yaml

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

  # Prometheus metrics receiver
  prometheus:
    config:
      scrape_configs:
        - job_name: 'otel-collector'
          scrape_interval: 10s
          static_configs:
            - targets: ['0.0.0.0:8888']

processors:
  # Batch processor для эффективности
  batch:
    timeout: 10s
    send_batch_size: 1024

  # Memory limiter
  memory_limiter:
    check_interval: 1s
    limit_mib: 512

  # Tail sampling - сохраняем все ошибки и медленные запросы
  tail_sampling:
    decision_wait: 10s
    num_traces: 100
    expected_new_traces_per_sec: 10
    policies:
      # Всегда сохраняем ошибки
      - name: error-traces
        type: status_code
        status_code:
          status_codes: [ERROR]
      
      # Медленные запросы (> 1s)
      - name: slow-traces
        type: latency
        latency:
          threshold_ms: 1000
      
      # 10% остальных
      - name: probabilistic-policy
        type: probabilistic
        probabilistic:
          sampling_percentage: 10

  # Добавление ресурсных атрибутов
  resource:
    attributes:
      - key: environment
        value: development
        action: insert

  # Attributes processor
  attributes:
    actions:
      - key: db.statement
        action: delete  # Удаляем SQL для безопасности (опционально)

exporters:
  # Jaeger exporter
  jaeger:
    endpoint: jaeger:14250
    tls:
      insecure: true

  # Logging exporter (для отладки)
  logging:
    loglevel: info

  # Prometheus exporter для метрик
  prometheus:
    endpoint: "0.0.0.0:8889"

service:
  pipelines:
    # Traces pipeline
    traces:
      receivers: [otlp]
      processors: [memory_limiter, tail_sampling, batch, resource, attributes]
      exporters: [jaeger, logging]
    
    # Metrics pipeline
    metrics:
      receivers: [otlp, prometheus]
      processors: [memory_limiter, batch]
      exporters: [prometheus, logging]
```

3. **Создай demo-app/backend (Python Flask)**:

`demo-app/backend/Dockerfile`:

dockerfile

````dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

`demo-app/backend/requirements.txt`:
```
flask==3.0.0
psycopg2-binary==2.9.9
redis==5.0.1
requests==2.31.0
opentelemetry-api==1.21.0
opentelemetry-sdk==1.21.0
opentelemetry-instrumentation-flask==0.42b0
opentelemetry-instrumentation-requests==0.42b0
opentelemetry-instrumentation-psycopg2==0.42b0
opentelemetry-instrumentation-redis==0.42b0
opentelemetry-exporter-otlp==1.21.0
````

`demo-app/backend/app.py`:

python

```python
from flask import Flask, jsonify, request
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.instrumentation.psycopg2 import Psycopg2Instrumentor
from opentelemetry.instrumentation.redis import RedisInstrumentor
from opentelemetry.sdk.resources import Resource
import psycopg2
import redis
import time
import random
import requests
import os

# Настройка OpenTelemetry
resource = Resource.create({
    "service.name": os.getenv("OTEL_SERVICE_NAME", "backend"),
    "service.version": "1.0.0",
    "deployment.environment": "development"
})

provider = TracerProvider(resource=resource)
processor = BatchSpanProcessor(
    OTLPSpanExporter(
        endpoint=os.getenv("OTEL_EXPORTER_OTLP_ENDPOINT", "http://localhost:4317"),
        insecure=True
    )
)
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)

# Создаем tracer
tracer = trace.get_tracer(__name__)

# Создаем Flask app
app = Flask(__name__)

# Auto-instrumentation
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
Psycopg2Instrumentor().instrument()
RedisInstrumentor().instrument()

# Database connection
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://user:password@localhost:5432/demo")
REDIS_URL = os.getenv("REDIS_URL", "redis://localhost:6379")

def get_db_connection():
    """Get database connection"""
    return psycopg2.connect(DATABASE_URL)

def get_redis_connection():
    """Get Redis connection"""
    return redis.from_url(REDIS_URL)

# Инициализация БД
def init_db():
    """Initialize database"""
    with get_db_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("""
                CREATE TABLE IF NOT EXISTS users (
                    id SERIAL PRIMARY KEY,
                    name VARCHAR(100),
                    email VARCHAR(100),
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            """)
            cur.execute("""
                CREATE TABLE IF NOT EXISTS orders (
                    id SERIAL PRIMARY KEY,
                    user_id INTEGER REFERENCES users(id),
                    product VARCHAR(100),
                    amount DECIMAL(10, 2),
                    status VARCHAR(20),
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            """)
            conn.commit()

# Routes
@app.route('/health')
def health():
    """Health check endpoint"""
    return jsonify({"status": "healthy"}), 200

@app.route('/api/users', methods=['GET'])
def get_users():
    """Get all users"""
    with tracer.start_as_current_span("get_users") as span:
        span.set_attribute("db.operation", "SELECT")
        
        # Симулируем случайную задержку
        time.sleep(random.uniform(0.01, 0.1))
        
        # Проверяем cache
        r = get_redis_connection()
        cached = r.get("users:all")
        
        if cached:
            span.add_event("Cache hit")
            span.set_attribute("cache.hit", True)
            import json
            return jsonify(json.loads(cached)), 200
        
        span.add_event("Cache miss")
        span.set_attribute("cache.hit", False)
        
        # Query database
        with get_db_connection() as conn:
            with conn.cursor() as cur:
                cur.execute("SELECT id, name, email FROM users")
                users = [
                    {"id": row[0], "name": row[1], "email": row[2]}
                    for row in cur.fetchall()
                ]
        
        # Cache result
        import json
        r.setex("users:all", 60, json.dumps(users))
        
        return jsonify(users), 200

@app.route('/api/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    """Get user by ID"""
    with tracer.start_as_current_span("get_user_by_id") as span:
        span.set_attribute("user.id", user_id)
        
        # Симулируем задержку
        time.sleep(random.uniform(0.01, 0.05))
        
        with get_db_connection() as conn:
            with conn.cursor() as cur:
                cur.execute(
                    "SELECT id, name, email FROM users WHERE id = %s",
                    (user_id,)
                )
                row = cur.fetchone()
                
                if not row:
                    span.set_attribute("http.status_code", 404)
                    return jsonify({"error": "User not found"}), 404
                
                user = {"id": row[0], "name": row[1], "email": row[2]}
        
        return jsonify(user), 200

@app.route('/api/users', methods=['POST'])
def create_user():
    """Create new user"""
    with tracer.start_as_current_span("create_user") as span:
        data = request.json
        
        span.set_attribute("user.name", data.get("name"))
        span.set_attribute("user.email", data.get("email"))
        
        # Валидация
        if not data.get("name") or not data.get("email"):
            span.set_attribute("error", True)
            span.add_event("Validation failed")
            return jsonify({"error": "Name and email required"}), 400
        
        # Симулируем задержку
        time.sleep(random.uniform(0.05, 0.15))
        
        with get_db_connection() as conn:
            with conn.cursor() as cur:
                cur.execute(
                    "INSERT INTO users (name, email) VALUES (%s, %s) RETURNING id",
                    (data["name"], data["email"])
                )
                user_id = cur.fetchone()[0]
                conn.commit()
        
        # Invalidate cache
        r = get_redis_connection()
        r.delete("users:all")
        
        span.add_event("User created", {"user.id": user_id})
        
        return jsonify({"id": user_id, "name": data["name"], "email": data["email"]}), 201

@app.route('/api/orders', methods=['POST'])
def create_order():
    """Create new order - демонстрирует complex trace"""
    with tracer.start_as_current_span("create_order") as span:
        data = request.json
        
        user_id = data.get("user_id")
        product = data.get("product")
        amount = data.get("amount")
        
        span.set_attribute("order.user_id", user_id)
        span.set_attribute("order.product", product)
        span.set_attribute("order.amount", amount)
        
        # Валидация
        if not all([user_id, product, amount]):
            span.set_attribute("error", True)
            return jsonify({"error": "Missing required fields"}), 400
        
        # Step 1: Check user exists
        with tracer.start_as_current_span("check_user") as user_span:
            user_span.set_attribute("user.id", user_id)
            time.sleep(random.uniform(0.01, 0.05))
            
            with get_db_connection() as conn:
                with conn.cursor() as cur:
                    cur.execute("SELECT id FROM users WHERE id = %s", (user_id,))
                    if not cur.fetchone():
                        span.set_attribute("error", True)
                        return jsonify({"error": "User not found"}), 404
        
        # Step 2: Process payment (симулируем external API)
        with tracer.start_as_current_span("process_payment") as payment_span:
            payment_span.set_attribute("payment.amount", amount)
            
            # Симулируем случайную задержку payment gateway
            delay = random.uniform(0.1, 0.5)
            
            # 10% шанс медленного payment
            if random.random() < 0.1:
                delay = random.uniform(2, 5)
                payment_span.set_attribute("payment.slow", True)
            
            # 5% шанс ошибки payment
            if random.random() < 0.05:
                time.sleep(delay)
                payment_span.set_attribute("error", True)
                payment_span.add_event("Payment failed")
                span.set_attribute("error", True)
                return jsonify({"error": "Payment processing failed"}), 500
            
            time.sleep(delay)
            payment_span.add_event("Payment successful")
        
        # Step 3: Create order
        with tracer.start_as_current_span("save_order") as save_span:
            time.sleep(random.uniform(0.02, 0.08))
            
            with get_db_connection() as conn:
                with conn.cursor() as cur:
                    cur.execute(
                        """
                        INSERT INTO orders (user_id, product, amount, status)
                        VALUES (%s, %s, %s, %s)
                        RETURNING id
                        """,
                        (user_id, product, amount, "completed")
                    )
                    order_id = cur.fetchone()[0]
                conn.commit()
        
        save_span.set_attribute("order.id", order_id)
    
    # Step 4: Send notification (async, в реальности)
    with tracer.start_as_current_span("send_notification") as notif_span:
        notif_span.set_attribute("notification.type", "email")
        time.sleep(random.uniform(0.05, 0.15))
        notif_span.add_event("Notification sent")
    
    span.add_event("Order created successfully", {"order.id": order_id})
    
    return jsonify({
        "id": order_id,
        "user_id": user_id,
        "product": product,
        "amount": amount,
        "status": "completed"
    }), 201


@app.route('/api/slow') def slow_endpoint(): """Intentionally slow endpoint для демонстрации""" with tracer.start_as_current_span("slow_operation") as span: span.add_event("Starting slow operation")


    # Симулируем N+1 query проблему
    with get_db_connection() as conn:
        with conn.cursor() as cur:
            # Получаем всех пользователей
            cur.execute("SELECT id FROM users LIMIT 10")
            user_ids = [row[0] for row in cur.fetchall()]
            
            # N+1: для каждого пользователя отдельный запрос заказов
            for user_id in user_ids:
                with tracer.start_as_current_span(f"get_orders_for_user_{user_id}"):
                    cur.execute(
                        "SELECT * FROM orders WHERE user_id = %s",
                        (user_id,)
                    )
                    cur.fetchall()
                    time.sleep(0.1)  # Симулируем задержку
    
    span.add_event("Slow operation completed")
    return jsonify({"message": "Completed slow operation"}), 200


if **name** == '**main**': init_db() app.run(host='0.0.0.0', port=5000, debug=True)

````

4. **Создай demo-app/frontend (Node.js Express)**:

`demo-app/frontend/Dockerfile`:
```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

CMD ["node", "app.js"]
```

`demo-app/frontend/package.json`:
```json
{
  "name": "frontend-service",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.2",
    "axios": "^1.6.2",
    "@opentelemetry/api": "^1.7.0",
    "@opentelemetry/sdk-node": "^0.45.1",
    "@opentelemetry/auto-instrumentations-node": "^0.40.1",
    "@opentelemetry/exporter-trace-otlp-grpc": "^0.45.1",
    "@opentelemetry/resources": "^1.19.0",
    "@opentelemetry/semantic-conventions": "^1.19.0"
  }
}
```

`demo-app/frontend/app.js`:
```javascript
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-grpc');
const { Resource } = require('@opentelemetry/resources');
const { SemanticResourceAttributes } = require('@opentelemetry/semantic-conventions');

// Initialize OpenTelemetry SDK
const sdk = new NodeSDK({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: process.env.OTEL_SERVICE_NAME || 'frontend',
    [SemanticResourceAttributes.SERVICE_VERSION]: '1.0.0',
    [SemanticResourceAttributes.DEPLOYMENT_ENVIRONMENT]: 'development',
  }),
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT || 'http://localhost:4317',
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();

// Graceful shutdown
process.on('SIGTERM', () => {
  sdk.shutdown()
    .then(() => console.log('Tracing terminated'))
    .catch((error) => console.log('Error terminating tracing', error))
    .finally(() => process.exit(0));
});

// Express app
const express = require('express');
const axios = require('axios');
const { trace, context } = require('@opentelemetry/api');

const app = express();
const PORT = 8080;
const BACKEND_URL = process.env.BACKEND_URL || 'http://localhost:5000';

app.use(express.json());

const tracer = trace.getTracer('frontend-service');

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.get('/', (req, res) => {
  res.send(`
    <html>
      <head><title>Demo Tracing App</title></head>
      <body>
        <h1>Distributed Tracing Demo</h1>
        <h2>Try these endpoints:</h2>
        <ul>
          <li><a href="/users">GET /users</a> - List users</li>
          <li><a href="/user/1">GET /user/:id</a> - Get user by ID</li>
          <li>POST /user - Create user</li>
          <li>POST /order - Create order (complex trace)</li>
          <li><a href="/slow">GET /slow</a> - Slow endpoint (N+1 problem)</li>
        </ul>
        <p>View traces in <a href="http://localhost:16686" target="_blank">Jaeger UI</a></p>
      </body>
    </html>
  `);
});

app.get('/users', async (req, res) => {
  const span = tracer.startSpan('frontend.get_users');
  
  try {
    span.addEvent('Fetching users from backend');
    const response = await axios.get(`${BACKEND_URL}/api/users`);
    span.addEvent('Users fetched successfully');
    res.json(response.data);
  } catch (error) {
    span.recordException(error);
    span.setStatus({ code: 2, message: error.message });
    res.status(500).json({ error: error.message });
  } finally {
    span.end();
  }
});

app.get('/user/:id', async (req, res) => {
  const span = tracer.startSpan('frontend.get_user');
  span.setAttribute('user.id', req.params.id);
  
  try {
    const response = await axios.get(`${BACKEND_URL}/api/users/${req.params.id}`);
    res.json(response.data);
  } catch (error) {
    span.recordException(error);
    if (error.response && error.response.status === 404) {
      res.status(404).json({ error: 'User not found' });
    } else {
      res.status(500).json({ error: error.message });
    }
  } finally {
    span.end();
  }
});

app.post('/user', async (req, res) => {
  const span = tracer.startSpan('frontend.create_user');
  span.setAttribute('user.name', req.body.name);
  
  try {
    const response = await axios.post(`${BACKEND_URL}/api/users`, req.body);
    res.status(201).json(response.data);
  } catch (error) {
    span.recordException(error);
    res.status(error.response?.status || 500).json({ 
      error: error.response?.data?.error || error.message 
    });
  } finally {
    span.end();
  }
});

app.post('/order', async (req, res) => {
  const span = tracer.startSpan('frontend.create_order');
  span.setAttribute('order.user_id', req.body.user_id);
  span.setAttribute('order.product', req.body.product);
  
  try {
    span.addEvent('Creating order');
    const response = await axios.post(`${BACKEND_URL}/api/orders`, req.body);
    span.addEvent('Order created successfully');
    res.status(201).json(response.data);
  } catch (error) {
    span.recordException(error);
    span.setStatus({ code: 2, message: error.message });
    res.status(error.response?.status || 500).json({ 
      error: error.response?.data?.error || error.message 
    });
  } finally {
    span.end();
  }
});

app.get('/slow', async (req, res) => {
  const span = tracer.startSpan('frontend.slow_endpoint');
  
  try {
    const response = await axios.get(`${BACKEND_URL}/api/slow`);
    res.json(response.data);
  } catch (error) {
    span.recordException(error);
    res.status(500).json({ error: error.message });
  } finally {
    span.end();
  }
});

app.listen(PORT, () => {
  console.log(`Frontend service listening on port ${PORT}`);
  console.log(`Jaeger UI: http://localhost:16686`);
});
```

5. **Создай grafana-datasources.yml**:
```yaml
apiVersion: 1

datasources:
  - name: Jaeger
    type: jaeger
    access: proxy
    url: http://jaeger:16686
    isDefault: true
    editable: true
    jsonData:
      tracesToLogs:
        datasourceUid: 'loki'
        tags: ['trace_id']
        
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    editable: true
    jsonData:
      httpMethod: POST
      exemplarTraceIdDestinations:
        - name: trace_id
          datasourceUid: 'jaeger'
```

6. **Запусти и протестируй**:
```bash
# Собери и запусти
docker-compose up --build -d

# Проверь статус
docker-compose ps

# Инициализируй тестовые данные
curl -X POST http://localhost:8080/user \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe", "email": "john@example.com"}'

curl -X POST http://localhost:8080/user \
  -H "Content-Type: application/json" \
  -d '{"name": "Jane Smith", "email": "jane@example.com"}'

# Создай несколько заказов для генерации трейсов
for i in {1..10}; do
  curl -X POST http://localhost:8080/order \
    -H "Content-Type: application/json" \
    -d "{\"user_id\": 1, \"product\": \"Product $i\", \"amount\": $((RANDOM % 100 + 10))}"
  sleep 1
done

# Тест slow endpoint
curl http://localhost:8080/slow
```

7. **Открой Jaeger UI и исследуй трейсы**:
```bash
# Jaeger UI
open http://localhost:16686

# Grafana
open http://localhost:3000

# Frontend app
open http://localhost:8080
```

8. **Создай load generator для реалистичных трейсов** (`load_test.sh`):
```bash
#!/bin/bash

echo "Starting load test..."

# Функция для случайных запросов
generate_load() {
  local endpoint=$1
  local method=$2
  local data=$3
  
  if [ "$method" == "GET" ]; then
    curl -s "$endpoint" > /dev/null
  else
    curl -s -X POST "$endpoint" \
      -H "Content-Type: application/json" \
      -d "$data" > /dev/null
  fi
}

# Бесконечный цикл генерации трафика
while true; do
  # 60% GET users
  if [ $((RANDOM % 10)) -lt 6 ]; then
    generate_load "http://localhost:8080/users" "GET"
  fi
  
  # 20% GET specific user
  if [ $((RANDOM % 10)) -lt 2 ]; then
    user_id=$((RANDOM % 5 + 1))
    generate_load "http://localhost:8080/user/$user_id" "GET"
  fi
  
  # 15% Create order
  if [ $((RANDOM % 20)) -lt 3 ]; then
    user_id=$((RANDOM % 5 + 1))
    product="Product-$RANDOM"
    amount=$((RANDOM % 100 + 10))
    generate_load "http://localhost:8080/order" "POST" \
      "{\"user_id\": $user_id, \"product\": \"$product\", \"amount\": $amount}"
  fi
  
  # 5% Slow endpoint
  if [ $((RANDOM % 20)) -lt 1 ]; then
    generate_load "http://localhost:8080/slow" "GET"
  fi
  
  # Случайная задержка между запросами
  sleep $(awk -v min=0.1 -v max=2 'BEGIN{srand(); print min+rand()*(max-min)}')
done
```

Запуск load test:
```bash
chmod +x load_test.sh
./load_test.sh &
```

### 🚀 Бонус (новое)

**1. Интеграция трейсов с логами**:

Обнови `demo-app/backend/app.py` для логирования с trace_id:
```python
import logging
from opentelemetry import trace

# Настройка логирования с trace context
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - [trace_id=%(otelTraceID)s span_id=%(otelSpanID)s] - %(message)s'
)

class TraceContextFilter(logging.Filter):
    """Add trace context to log records"""
    def filter(self, record):
        span = trace.get_current_span()
        if span:
            ctx = span.get_span_context()
            record.otelTraceID = format(ctx.trace_id, '032x') if ctx.trace_id else ''
            record.otelSpanID = format(ctx.span_id, '016x') if ctx.span_id else ''
        else:
            record.otelTraceID = ''
            record.otelSpanID = ''
        return True

logger = logging.getLogger(__name__)
logger.addFilter(TraceContextFilter())

# Используй в коде
@app.route('/api/users', methods=['GET'])
def get_users():
    logger.info("Fetching all users")
    # ... код
```

**2. Custom span attributes и events**:
```python
@app.route('/api/complex-operation', methods=['POST'])
def complex_operation():
    with tracer.start_as_current_span("complex_operation") as span:
        # Business metrics
        span.set_attribute("business.operation_type", "purchase")
        span.set_attribute("business.customer_tier", "premium")
        span.set_attribute("business.discount_applied", True)
        
        # Performance metrics
        span.set_attribute("cache.enabled", True)
        span.set_attribute("db.pool_size", 10)
        
        # События с метаданными
        span.add_event("validation_started", {
            "validator.version": "1.0",
            "rules.count": 5
        })
        
        # Симулируем работу
        time.sleep(0.1)
        
        span.add_event("validation_completed", {
            "validation.passed": True,
            "rules.failed": 0
        })
        
        # Exception handling
        try:
            risky_operation()
        except Exception as e:
            span.record_exception(e)
            span.set_status(trace.Status(trace.StatusCode.ERROR, str(e)))
            raise
        
        return jsonify({"status": "success"})
```

**3. Grafana Tempo (альтернатива Jaeger)**:

Добавь в `docker-compose.yml`:
```yaml
  tempo:
    image: grafana/tempo:latest
    container_name: tempo
    command: ["-config.file=/etc/tempo.yml"]
    volumes:
      - ./tempo.yml:/etc/tempo.yml
      - tempo-data:/tmp/tempo
    ports:
      - "3200:3200"   # Tempo HTTP
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
    restart: unless-stopped

volumes:
  tempo-data:
```

`tempo.yml`:
```yaml
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: 0.0.0.0:4317
        http:
          endpoint: 0.0.0.0:4318

ingester:
  max_block_duration: 5m

compactor:
  compaction:
    block_retention: 168h  # 7 days

storage:
  trace:
    backend: local
    local:
      path: /tmp/tempo/blocks
    wal:
      path: /tmp/tempo/wal

query_frontend:
  search:
    enabled: true
```

**4. Exemplars - связь метрик и трейсов**:

В Prometheus metrics добавь exemplars:
```python
from prometheus_client import Counter, Histogram
from opentelemetry import trace

request_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

@app.route('/api/endpoint')
def endpoint():
    span = trace.get_current_span()
    ctx = span.get_span_context()
    trace_id = format(ctx.trace_id, '032x')
    
    with request_duration.labels(method='GET', endpoint='/api/endpoint').time():
        # Add exemplar
        request_duration.labels(method='GET', endpoint='/api/endpoint').observe(
            0.5,
            exemplar={'trace_id': trace_id}
        )
        
        return jsonify({"result": "ok"})
```

**5. Service dependency graph automation**:

Создай `dependency_analyzer.py`:
```python
#!/usr/bin/env python3
"""
Анализ зависимостей сервисов из Jaeger
"""
import requests
from collections import defaultdict
import json

JAEGER_URL = "http://localhost:16686"

def get_services():
    """Получить список сервисов"""
    response = requests.get(f"{JAEGER_URL}/api/services")
    return response.json()['data']

def get_dependencies():
    """Получить зависимости сервисов"""
    # lookback in microseconds (24 hours)
    lookback = 24 * 60 * 60 * 1000000
    
    response = requests.get(
        f"{JAEGER_URL}/api/dependencies",
        params={'endTs': None, 'lookback': lookback}
    )
    return response.json()['data']

def build_graph():
    """Построить граф зависимостей"""
    dependencies = get_dependencies()
    
    graph = defaultdict(list)
    for dep in dependencies:
        parent = dep['parent']
        child = dep['child']
        call_count = dep['callCount']
        
        graph[parent].append({
            'service': child,
            'calls': call_count
        })
    
    return graph

def print_graph(graph):
    """Вывести граф"""
    print("\n=== Service Dependency Graph ===\n")
    
    for service, dependencies in sorted(graph.items()):
        print(f"{service}")
        for dep in dependencies:
            print(f"  └─> {dep['service']} ({dep['calls']} calls)")
        print()

def export_mermaid(graph):
    """Экспорт в Mermaid диаграмму"""
    print("\n=== Mermaid Diagram ===\n")
    print("```mermaid")
    print("graph LR")
    
    for service, dependencies in graph.items():
        for dep in dependencies:
            # Безопасные имена для Mermaid
            safe_parent = service.replace('-', '_')
            safe_child = dep['service'].replace('-', '_')
            print(f"    {safe_parent}[{service}] --> {safe_child}[{dep['service']}]")
    
    print("```\n")

if __name__ == "__main__":
    print("Analyzing service dependencies from Jaeger...")
    
    services = get_services()
    print(f"\nFound {len(services)} services:")
    for svc in services:
        print(f"  - {svc}")
    
    graph = build_graph()
    print_graph(graph)
    export_mermaid(graph)
```

**6. Continuous Profiling интеграция**:

Добавь Pyroscope для continuous profiling:
```yaml
  pyroscope:
    image: grafana/pyroscope:latest
    container_name: pyroscope
    ports:
      - "4040:4040"
    volumes:
      - pyroscope-data:/var/lib/pyroscope
    restart: unless-stopped

volumes:
  pyroscope-data:
```

В Python app:
```python
import pyroscope

pyroscope.configure(
    application_name="backend-service",
    server_address="http://pyroscope:4040",
    tags={
        "environment": "development",
        "service": "backend"
    }
)
```

**7. Error tracking с Sentry интеграция**:
```python
import sentry_sdk
from sentry_sdk.integrations.flask import FlaskIntegration
from opentelemetry import trace

sentry_sdk.init(
    dsn="your-sentry-dsn",
    integrations=[FlaskIntegration()],
    traces_sample_rate=1.0,
    before_send=lambda event, hint: attach_trace_context(event)
)

def attach_trace_context(event):
    """Добавить trace context в Sentry events"""
    span = trace.get_current_span()
    if span:
        ctx = span.get_span_context()
        event['contexts']['trace'] = {
            'trace_id': format(ctx.trace_id, '032x'),
            'span_id': format(ctx.span_id, '016x')
        }
    return event
```

**8. Custom dashboard в Grafana**:

JSON dashboard (`grafana-tracing-dashboard.json`):
```json
{
  "dashboard": {
    "title": "Distributed Tracing Overview",
    "panels": [
      {
        "id": 1,
        "title": "Request Rate by Service",
        "type": "timeseries",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (service)"
          }
        ]
      },
      {
        "id": 2,
        "title": "P95 Latency by Service",
        "type": "timeseries",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
          }
        ]
      },
      {
        "id": 3,
        "title": "Error Rate",
        "type": "stat",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m]))"
          }
        ]
      },
      {
        "id": 4,
        "title": "Trace Search",
        "type": "jaeger",
        "datasource": "Jaeger"
      }
    ]
  }
}
```

---

## Итоги модуля 6

После прохождения этого модуля ты должен уметь:

✅ Понимать концепции distributed tracing (trace, span, context)
✅ Настраивать Jaeger/Tempo для сбора трейсов
✅ Инструментировать приложения с OpenTelemetry
✅ Использовать auto и manual instrumentation
✅ Настраивать sampling стратегии
✅ Анализировать трейсы для поиска bottleneck'ов
✅ Связывать трейсы с логами и метриками
✅ Строить service dependency graphs
✅ Интегрировать с error tracking
✅ Использовать continuous profiling

**Ключевые выводы:**
1. Tracing показывает КАК работает система (путь запроса)
2. Используй tail sampling для сохранения важных трейсов
3. Всегда передавай trace context между сервисами
4. Добавляй полезные attributes и events
5. Интегрируй tracing с metrics и logs
6. Используй semantic conventions
7. Регулярно анализируй service dependencies

**Следующий модуль:** Infrastructure as Code для мониторинга (автоматизация deployment monitoring stack)


---
## Модуль 7: Infrastructure as Code для мониторинга (35 минут)

### 🎯 Напоминалка

**IaC для мониторинга - зачем:**

```
Проблема: Ручная настройка мониторинга
❌ Долго (часы на setup)
❌ Ошибки (человеческий фактор)
❌ Не воспроизводимо
❌ Сложно масштабировать
❌ Нет version control

Решение: Infrastructure as Code
✅ Быстро (минуты на deploy)
✅ Надежно (автоматизация)
✅ Воспроизводимо (идентичные окружения)
✅ Масштабируемо (легко добавить новые сервисы)
✅ Version control (Git history)
```

**Основные подходы:**

```
1. Configuration Management:
   - Ansible
   - Chef
   - Puppet
   - SaltStack

2. Container Orchestration:
   - Docker Compose
   - Kubernetes (Helm)
   - Docker Swarm

3. Infrastructure Provisioning:
   - Terraform
   - Pulumi
   - CloudFormation (AWS)

4. GitOps:
   - ArgoCD
   - Flux
   - Jenkins X
```

**Terraform для мониторинга:**

```
Provider Support:
- Prometheus (rules, alertmanager config)
- Grafana (dashboards, data sources, folders)
- PagerDuty (services, escalation policies)
- Datadog (monitors, dashboards)
- AWS CloudWatch (alarms, dashboards)

Преимущества:
- Декларативный синтаксис
- State management
- Plan/Apply workflow
- Module reusability
```

**Ansible для мониторинга:**

```
Использование:
- Установка monitoring agents
- Конфигурация exporters
- Deployment monitoring stack
- Управление dashboards

Преимущества:
- Agentless (SSH)
- Простой YAML синтаксис
- Большая библиотека modules
- Idempotent operations
```

**Helm для Kubernetes:**

```
Официальные charts:
- prometheus-community/kube-prometheus-stack
- grafana/grafana
- grafana/loki-stack
- jaegertracing/jaeger
- elastic/elasticsearch

Преимущества:
- Templating
- Values override
- Release management
- Dependency management
```

**GitOps workflow:**

```
┌──────────┐
│   Git    │ (Source of Truth)
└────┬─────┘
     │
     │ Push
     ▼
┌──────────┐
│  CI/CD   │ (Validation, Testing)
└────┬─────┘
     │
     │ Deploy
     ▼
┌──────────┐
│ Cluster  │ (Auto-sync)
└──────────┘

Principles:
1. Declarative configuration
2. Version controlled
3. Automated deployment
4. Self-healing
```

**Prometheus Configuration Management:**

yaml

```yaml
# prometheus.yml как код
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: {{ cluster_name }}
    environment: {{ environment }}

# Template переменные
alerting:
  alertmanagers:
  - static_configs:
    - targets: 
      {{ range .AlertmanagerTargets }}
      - {{ . }}
      {{ end }}

scrape_configs:
  {{ range .Jobs }}
  - job_name: {{ .Name }}
    static_configs:
      - targets: {{ .Targets }}
        labels: {{ .Labels }}
  {{ end }}
```

**Grafana Dashboard as Code:**

json

```json
{
  "dashboard": {
    "title": "{{ .Title }}",
    "tags": {{ .Tags | toJson }},
    "timezone": "browser",
    "panels": [
      {{ range .Panels }}
      {
        "id": {{ .ID }},
        "title": "{{ .Title }}",
        "type": "{{ .Type }}",
        "targets": [
          {
            "expr": "{{ .Query }}",
            "legendFormat": "{{ .Legend }}"
          }
        ]
      }{{ if not (last $.Panels .) }},{{ end }}
      {{ end }}
    ]
  }
}
```

**Alert Rules as Code:**

yaml

````yaml
# alerts.yml template
groups:
{{ range .AlertGroups }}
  - name: {{ .Name }}
    interval: {{ .Interval }}
    rules:
    {{ range .Rules }}
    - alert: {{ .Name }}
      expr: |
        {{ .Expression }}
      for: {{ .For }}
      labels:
        severity: {{ .Severity }}
        team: {{ .Team }}
      annotations:
        summary: {{ .Summary }}
        description: {{ .Description }}
        runbook: {{ .Runbook }}
    {{ end }}
{{ end }}
````

**Monitoring Stack Components:**
```
Full Stack:
├── Metrics Collection
│   ├── Prometheus
│   ├── Node Exporter
│   ├── Blackbox Exporter
│   └── Custom Exporters
├── Logs Collection
│   ├── Loki
│   ├── Promtail
│   └── Fluentd/Fluent Bit
├── Tracing
│   ├── Jaeger/Tempo
│   └── OpenTelemetry Collector
├── Visualization
│   └── Grafana
├── Alerting
│   └── Alertmanager
└── Notification
    ├── Slack
    ├── PagerDuty
    └── Email
```

**Directory Structure (best practices):**
````
monitoring-infrastructure/
├── terraform/
│   ├── modules/
│   │   ├── prometheus/
│   │   ├── grafana/
│   │   ├── alertmanager/
│   │   └── loki/
│   ├── environments/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── prod/
│   └── main.tf
├── ansible/
│   ├── playbooks/
│   │   ├── install-prometheus.yml
│   │   ├── configure-exporters.yml
│   │   └── deploy-dashboards.yml
│   ├── roles/
│   │   ├── prometheus/
│   │   ├── grafana/
│   │   └── node-exporter/
│   └── inventory/
├── kubernetes/
│   ├── helm/
│   │   ├── values-dev.yaml
│   │   ├── values-staging.yaml
│   │   └── values-prod.yaml
│   └── manifests/
├── docker/
│   ├── docker-compose.yml
│   └── docker-compose.override.yml
├── config/
│   ├── prometheus/
│   │   ├── prometheus.yml
│   │   └── alerts/
│   ├── grafana/
│   │   └── dashboards/
│   └── alertmanager/
│       └── alertmanager.yml
└── scripts/
    ├── deploy.sh
    ├── backup.sh
    └── validate.sh
````

**Validation & Testing:**

bash

````bash
# Prometheus config validation
promtool check config prometheus.yml
promtool check rules alerts.yml

# Alert testing
promtool test rules test-alerts.yml

# Grafana dashboard validation
grafana-cli admin validate-dashboard dashboard.json

# Terraform validation
terraform validate
terraform plan

# Ansible syntax check
ansible-playbook --syntax-check playbook.yml
ansible-lint playbook.yml
````

**Backup & Disaster Recovery:**
````
Что бэкапить:
1. ✅ Prometheus TSDB (опционально, данные ephemeral)
2. ✅ Grafana database (dashboards, users, settings)
3. ✅ Alertmanager data (silences, notification log)
4. ✅ Configuration files (prometheus.yml, alerts, etc)
5. ✅ Custom exporters config

Инструменты:
- Prometheus: snapshots API
- Grafana: grafana-backup tool, API export
- Velero: Kubernetes backup
- Restic: filesystem backup
````

**Environment Management:**

yaml

````yaml
# Разные конфигурации для окружений
Dev:
  retention: 7d
  replicas: 1
  resources: small
  scrape_interval: 30s

Staging:
  retention: 14d
  replicas: 2
  resources: medium
  scrape_interval: 15s

Production:
  retention: 30d
  replicas: 3
  resources: large
  scrape_interval: 15s
  high_availability: true
````

**Secrets Management:**
````
Options:
1. HashiCorp Vault
   - Centralized secrets
   - Dynamic credentials
   - Audit logging

2. Kubernetes Secrets
   - Native K8s
   - External Secrets Operator

3. AWS Secrets Manager
   - Managed service
   - Rotation support

4. Sealed Secrets
   - GitOps friendly
   - Encrypted in Git

Best Practice: Never commit secrets to Git!
````

**CI/CD Pipeline для мониторинга:**

yaml

```yaml
# .github/workflows/monitoring.yml
name: Deploy Monitoring Stack

on:
  push:
    branches: [main]
    paths:
      - 'monitoring/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Validate Prometheus Config
        run: |
          docker run --rm -v $PWD:/prometheus prom/prometheus:latest \
            promtool check config /prometheus/config/prometheus.yml
      
      - name: Test Alert Rules
        run: |
          docker run --rm -v $PWD:/prometheus prom/prometheus:latest \
            promtool test rules /prometheus/tests/alerts-test.yml
      
      - name: Validate Grafana Dashboards
        run: |
          for file in grafana/dashboards/*.json; do
            jq empty "$file" || exit 1
          done

  deploy-dev:
    needs: validate
    runs-on: ubuntu-latest
    environment: dev
    steps:
      - name: Deploy to Dev
        run: |
          helm upgrade --install monitoring ./helm \
            --values values-dev.yaml \
            --namespace monitoring \
            --create-namespace

  deploy-prod:
    needs: deploy-dev
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to Production
        run: |
          helm upgrade --install monitoring ./helm \
            --values values-prod.yaml \
            --namespace monitoring
```

### 💻 Задание

Создай полноценную IaC инфраструктуру для мониторинга:

1. **Создай Terraform проект для Grafana**:

`terraform/main.tf`:

hcl

```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    grafana = {
      source  = "grafana/grafana"
      version = "~> 2.9.0"
    }
  }
}

provider "grafana" {
  url  = var.grafana_url
  auth = var.grafana_auth
}

# Data source для Prometheus
resource "grafana_data_source" "prometheus" {
  type = "prometheus"
  name = "Prometheus"
  url  = var.prometheus_url
  
  is_default = true
  
  json_data_encoded = jsonencode({
    httpMethod    = "POST"
    timeInterval  = "30s"
  })
}

# Data source для Loki
resource "grafana_data_source" "loki" {
  type = "loki"
  name = "Loki"
  url  = var.loki_url
  
  json_data_encoded = jsonencode({
    maxLines = 1000
  })
}

# Folder для dashboards
resource "grafana_folder" "monitoring" {
  title = "Monitoring"
}

resource "grafana_folder" "applications" {
  title = "Applications"
}

# Dashboard - Node Exporter
resource "grafana_dashboard" "node_exporter" {
  folder      = grafana_folder.monitoring.id
  config_json = file("${path.module}/dashboards/node-exporter.json")
}

# Dashboard - Application Metrics
resource "grafana_dashboard" "application" {
  folder      = grafana_folder.applications.id
  config_json = templatefile("${path.module}/dashboards/application.json.tpl", {
    datasource = grafana_data_source.prometheus.name
    environment = var.environment
  })
}

# Alert notification channel - Slack
resource "grafana_contact_point" "slack" {
  name = "Slack Alerts"
  
  slack {
    url  = var.slack_webhook_url
    text = templatefile("${path.module}/templates/slack-message.tpl", {})
  }
}

# Alert notification channel - PagerDuty
resource "grafana_contact_point" "pagerduty" {
  name = "PagerDuty"
  
  pagerduty {
    integration_key = var.pagerduty_key
    severity        = "critical"
  }
}

# Notification policy
resource "grafana_notification_policy" "main" {
  group_by      = ["alertname", "grafana_folder"]
  group_wait    = "10s"
  group_interval = "5m"
  repeat_interval = "4h"
  
  policy {
    matcher {
      label = "severity"
      match = "="
      value = "critical"
    }
    contact_point = grafana_contact_point.pagerduty.name
    continue      = true
  }
  
  policy {
    matcher {
      label = "severity"
      match = "="
      value = "warning"
    }
    contact_point = grafana_contact_point.slack.name
  }
}

# Alert rule - High CPU
resource "grafana_rule_group" "infrastructure" {
  name             = "Infrastructure Alerts"
  folder_uid       = grafana_folder.monitoring.uid
  interval_seconds = 60
  
  rule {
    name      = "HighCPUUsage"
    condition = "C"
    
    data {
      ref_id = "A"
      
      relative_time_range {
        from = 600
        to   = 0
      }
      
      datasource_uid = grafana_data_source.prometheus.uid
      model = jsonencode({
        expr         = "100 - (avg(irate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)"
        refId        = "A"
        intervalMs   = 1000
        maxDataPoints = 43200
      })
    }
    
    data {
      ref_id = "B"
      
      relative_time_range {
        from = 0
        to   = 0
      }
      
      datasource_uid = "__expr__"
      model = jsonencode({
        expression = "A"
        reducer    = "last"
        refId      = "B"
        type       = "reduce"
      })
    }
    
    data {
      ref_id = "C"
      
      relative_time_range {
        from = 0
        to   = 0
      }
      
      datasource_uid = "__expr__"
      model = jsonencode({
        expression = "B > 80"
        refId      = "C"
        type       = "threshold"
      })
    }
    
    no_data_state  = "NoData"
    exec_err_state = "Error"
    for            = "5m"
    
    annotations = {
      summary     = "High CPU usage detected"
      description = "CPU usage is above 80%"
      runbook_url = "https://runbooks.example.com/high-cpu"
    }
    
    labels = {
      severity = "warning"
      team     = "infrastructure"
    }
  }
}

# Organization settings
resource "grafana_organization_preferences" "main" {
  theme            = "dark"
  home_dashboard_uid = grafana_dashboard.node_exporter.uid
  timezone         = "UTC"
}

# Team
resource "grafana_team" "infrastructure" {
  name  = "Infrastructure Team"
  email = "infra@example.com"
}

# Service Account для API access
resource "grafana_service_account" "automation" {
  name = "automation"
  role = "Admin"
}

resource "grafana_service_account_token" "automation" {
  name               = "automation-token"
  service_account_id = grafana_service_account.automation.id
}
```

`terraform/variables.tf`:

hcl

```hcl
variable "grafana_url" {
  description = "Grafana URL"
  type        = string
  default     = "http://localhost:3000"
}

variable "grafana_auth" {
  description = "Grafana auth (admin:password)"
  type        = string
  sensitive   = true
}

variable "prometheus_url" {
  description = "Prometheus URL"
  type        = string
  default     = "http://prometheus:9090"
}

variable "loki_url" {
  description = "Loki URL"
  type        = string
  default     = "http://loki:3100"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "development"
}

variable "slack_webhook_url" {
  description = "Slack webhook URL"
  type        = string
  sensitive   = true
}

variable "pagerduty_key" {
  description = "PagerDuty integration key"
  type        = string
  sensitive   = true
}
```

`terraform/outputs.tf`:

hcl

```hcl
output "prometheus_datasource_uid" {
  description = "Prometheus data source UID"
  value       = grafana_data_source.prometheus.uid
}

output "loki_datasource_uid" {
  description = "Loki data source UID"
  value       = grafana_data_source.loki.uid
}

output "automation_token" {
  description = "Automation service account token"
  value       = grafana_service_account_token.automation.key
  sensitive   = true
}

output "dashboard_urls" {
  description = "URLs of created dashboards"
  value = {
    node_exporter = "${var.grafana_url}/d/${grafana_dashboard.node_exporter.uid}"
    application   = "${var.grafana_url}/d/${grafana_dashboard.application.uid}"
  }
}
```

`terraform/dashboards/application.json.tpl`:

json

```json
{
  "dashboard": {
    "title": "Application Metrics - ${environment}",
    "tags": ["application", "${environment}"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Request Rate",
        "type": "timeseries",
        "gridPos": {
          "h": 8,
          "w": 12,
          "x": 0,
          "y": 0
        },
        "targets": [
          {
            "datasource": "${datasource}",
            "expr": "sum(rate(http_requests_total{environment=\"${environment}\"}[5m])) by (service)",
            "legendFormat": "{{service}}"
          }
        ]
      },
      {
        "id": 2,
        "title": "Error Rate",
        "type": "stat",
        "gridPos": {
          "h": 8,
          "w": 12,
          "x": 12,
          "y": 0
        },
        "targets": [
          {
            "datasource": "${datasource}",
            "expr": "sum(rate(http_requests_total{environment=\"${environment}\",status=~\"5..\"}[5m])) / sum(rate(http_requests_total{environment=\"${environment}\"}[5m]))",
            "legendFormat": "Error Rate"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percentunit",
            "thresholds": {
              "steps": [
                {"value": 0, "color": "green"},
                {"value": 0.01, "color": "yellow"},
                {"value": 0.05, "color": "red"}
              ]
            }
          }
        }
      }
    ]
  }
}
```

2. **Создай Ansible playbook для deployment**:

`ansible/inventory/hosts.ini`:

ini

```ini
[monitoring_servers]
monitoring-01 ansible_host=192.168.1.10 ansible_user=ubuntu
monitoring-02 ansible_host=192.168.1.11 ansible_user=ubuntu

[app_servers]
app-01 ansible_host=192.168.1.20 ansible_user=ubuntu
app-02 ansible_host=192.168.1.21 ansible_user=ubuntu
app-03 ansible_host=192.168.1.22 ansible_user=ubuntu

[all:vars]
ansible_python_interpreter=/usr/bin/python3
environment=production
```

`ansible/playbooks/deploy-monitoring-stack.yml`:

yaml

````yaml
---
- name: Deploy Monitoring Stack
  hosts: monitoring_servers
  become: yes
  vars:
    prometheus_version: "2.48.0"
    grafana_version: "10.2.3"
    alertmanager_version: "0.26.0"
    node_exporter_version: "1.7.0"
    
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
    
    - name: Install prerequisites
      apt:
        name:
          - apt-transport-https
          - software-properties-common
          - wget
          - curl
          - tar
        state: present
    
    - name: Create monitoring user
      user:
        name: monitoring
        system: yes
        shell: /bin/false
        create_home: no
    
    - name: Deploy Prometheus
      include_role:
        name: prometheus
      vars:
        prometheus_config_template: "{{ playbook_dir }}/../config/prometheus/prometheus.yml.j2"
        prometheus_alerts_dir: "{{ playbook_dir }}/../config/prometheus/alerts"
    
    - name: Deploy Alertmanager
      include_role:
        name: alertmanager
      vars:
        alertmanager_config_template: "{{ playbook_dir }}/../config/alertmanager/alertmanager.yml.j2"
    
    - name: Deploy Grafana
      include_role:
        name: grafana
      vars:
        grafana_provisioning_dir: "{{ playbook_dir }}/../config/grafana/provisioning"

- name: Install Node Exporter on all servers
  hosts: all
  become: yes
  tasks:
    - name: Deploy Node Exporter
      include_role:
        name: node_exporter

- name: Configure Application Monitoring
  hosts: app_servers
  become: yes
  tasks:
    - name: Install application exporters
      include_role:
        name: app_exporter
````

`ansible/roles/prometheus/tasks/main.yml`:

yaml

````yaml
---
- name: Create Prometheus directories
  file:
    path: "{{ item }}"
    state: directory
    owner: monitoring
    group: monitoring
    mode: '0755'
  loop:
    - /etc/prometheus
    - /etc/prometheus/rules
    - /var/lib/prometheus

- name: Download Prometheus
  get_url:
    url: "https://github.com/prometheus/prometheus/releases/download/v{{ prometheus_version }}/prometheus-{{ prometheus_version }}.linux-amd64.tar.gz"
    dest: "/tmp/prometheus-{{ prometheus_version }}.tar.gz"

- name: Extract Prometheus
  unarchive:
    src: "/tmp/prometheus-{{ prometheus_version }}.tar.gz"
    dest: /tmp
    remote_src: yes

- name: Install Prometheus binaries
  copy:
    src: "/tmp/prometheus-{{ prometheus_version }}.linux-amd64/{{ item }}"
    dest: "/usr/local/bin/{{ item }}"
    owner: monitoring
    group: monitoring
    mode: '0755'
    remote_src: yes
  loop:
    - prometheus
    - promtool

- name: Copy Prometheus configuration
  template:
    src: "{{ prometheus_config_template }}"
    dest: /etc/prometheus/prometheus.yml
    owner: monitoring
    group: monitoring
    mode: '0644'
  notify: reload prometheus

- name: Copy alert rules
  copy:
    src: "{{ prometheus_alerts_dir }}/"
    dest: /etc/prometheus/rules/
    owner: monitoring
    group: monitoring
    mode: '0644'
  notify: reload prometheus

- name: Create Prometheus systemd service
  template:
    src: prometheus.service.j2
    dest: /etc/systemd/system/prometheus.service
    owner: root
    group: root
    mode: '0644'
  notify: restart prometheus

- name: Enable and start Prometheus
  systemd:
    name: prometheus
    enabled: yes
    state: started
    daemon_reload: yes
````

`ansible/roles/prometheus/templates/prometheus.service.j2`:

ini

```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=monitoring
Group=monitoring
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --storage.tsdb.retention.time={{ prometheus_retention | default('30d') }} \
  --web.enable-lifecycle \
  --web.enable-admin-api

Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

`ansible/roles/prometheus/handlers/main.yml`:

yaml

````yaml
---
- name: reload prometheus
  uri:
    url: "http://localhost:9090/-/reload"
    method: POST
  listen: reload prometheus

- name: restart prometheus
  systemd:
    name: prometheus
    state: restarted
  listen: restart prometheus
````

3. **Создай Helm chart для Kubernetes**:

`helm/monitoring/Chart.yaml`:

yaml

```yaml
apiVersion: v2
name: monitoring-stack
description: Complete monitoring stack for Kubernetes
type: application
version: 1.0.0
appVersion: "1.0"

dependencies:
  - name: kube-prometheus-stack
    version: "55.0.0"
    repository: "https://prometheus-community.github.io/helm-charts"
    condition: prometheus.enabled
    
  - name: loki-stack
    version: "2.9.11"
    repository: "https://grafana.github.io/helm-charts"
    condition: loki.enabled
    
  - name: jaeger
    version: "0.71.11"
    repository: "https://jaegertracing.github.io/helm-charts"
    condition: jaeger.enabled
```

`helm/monitoring/values.yaml`:

yaml

```yaml
# Global settings
global:
  environment: production
  clusterName: main-cluster

# Prometheus configuration
prometheus:
  enabled: true
  
kube-prometheus-stack:
  prometheus:
    prometheusSpec:
      retention: 30d
      retentionSize: "50GB"
      replicas: 2
      storageSpec:
        volumeClaimTemplate:
          spec:
            accessModes: ["ReadWriteOnce"]
            resources:
              requests:
                storage: 100Gi
      
      # Additional scrape configs
      additionalScrapeConfigs:
        - job_name: 'custom-app'
          kubernetes_sd_configs:
            - role: pod
          relabel_configs:
            - source_labels: [__meta_kubernetes_pod_label_app]
              regex: my-app
              action: keep
      
      # Remote write (для long-term storage)
      remoteWrite:
        - url: http://thanos-receiver:19291/api/v1/receive
          queueConfig:
            capacity: 10000
            maxShards: 50
  
  # Alert rules
  additionalPrometheusRulesMap:
    custom-alerts:
      groups:
        - name: custom_application_alerts
          interval: 30s
          rules:
            - alert: ApplicationDown
              expr: up{job="my-app"} == 0
              for: 2m
              labels:
                severity: critical
                team: backend
              annotations:
                summary: "Application {{ $labels.instance }} is down"
                description: "Application has been down for more than 2 minutes"
  
  # Alertmanager
  alertmanager:
    config:
      global:
        resolve_timeout: 5m
        slack_api_url: {{ .Values.slack.webhookUrl }}
      
      route:
        group_by: ['alertname', 'cluster', 'service']
        group_wait: 10s
        group_interval: 5m
        repeat_interval: 4h
        receiver: 'default'
        
        routes:
          - match:
              severity: critical
            receiver: pagerduty
            continue: true
          
          - match:
              severity: warning
            receiver: slack
      
      receivers:
        - name: 'default'
          webhook_configs:
            - url: 'http://webhook-receiver:8080/webhook'
        
        - name: 'slack'
          slack_configs:
            - channel: '#alerts'
              title: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
              text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
        
        - name: 'pagerduty'
          pagerduty_configs:
            - service_key: {{ .Values.pagerduty.serviceKey }}
  
  # Grafana
  grafana:
    enabled: true
    adminPassword: {{ .Values.grafana.adminPassword }}
    
    persistence:
      enabled: true
      size: 10Gi
    
    dashboardProviders:
      dashboardproviders.yaml:
        apiVersion: 1
        providers:
          - name: 'default'
            orgId: 1
            folder: ''
            type: file
            disableDeletion: false
            editable: true
            options:
              path: /var/lib/grafana/dashboards/default
    
    dashboards:
      default:
        node-exporter:
          gnetId: 1860
          revision: 31
          datasource: Prometheus
        
        kubernetes-cluster:
          gnetId: 7249
          revision: 1
          datasource: Prometheus

# Loki configuration
loki:
  enabled: true

loki-stack:
  loki:
    persistence:
      enabled: true
      size: 50Gi
    
    config:
      limits_config:
        retention_period: 168h  # 7 days
      
      compactor:
        retention_enabled: true
  
  promtail:
    config:
      clients:
        - url: http://loki:3100/loki/api/v1/push

# Jaeger configuration
jaeger:
  enabled: true

jaeger:
  storage:
    type: elasticsearch
  
  elasticsearch:
    replicas: 3
    minimumMasterNodes: 2
```

`helm/monitoring/values-dev.yaml`:
```yaml
global:
  environment: development
  clusterName: dev-cluster

kube-prometheus-stack:
  prometheus:
    prometheusSpec:
      retention: 7d
      replicas: 1
      storageSpec:
        volumeClaimTemplate:
          spec:
            resources:
              requests:
                storage: 20Gi

loki-stack:
  loki:
    config:
      limits_config:
        retention_period: 48h

jaeger:
  enabled: false  # Disable in dev
```

4. **Создай GitOps configuration для ArgoCD**:

`argocd/monitoring-app.yaml`:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: monitoring-stack
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/your-org/monitoring-infrastructure.git
    targetRevision: main
    path: helm/monitoring
    helm:
      valueFiles:
        - values.yaml
        - values-{{ .Values.environment }}.yaml
  
  destination:
    server: https://kubernetes.default.svc
    namespace: monitoring
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
    
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas  # Ignore HPA changes
```

5. **Создай CI/CD pipeline**:

`.github/workflows/deploy-monitoring.yml`:
```yaml
name: Deploy Monitoring Stack

on:
  push:
    branches: [main, develop]
    paths:
      - 'helm/**'
      - 'config/**'
      - 'terraform/**'
  pull_request:
    branches: [main]

env:
  HELM_VERSION: v3.13.0
  TERRAFORM_VERSION: 1.6.0

jobs:
  validate:
    name: Validate Configurations
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Helm
        uses: azure/setup-helm@v3
        with:
          version: ${{ env.HELM_VERSION }}
      
      - name: Lint Helm Charts
        run: |
          helm lint helm/monitoring
          helm lint helm/monitoring --values helm/monitoring/values-dev.yaml
          helm lint helm/monitoring --values helm/monitoring/values-prod.yaml
      
      - name: Validate Prometheus Config
        run: |
          docker run --rm -v $PWD/config/prometheus:/prometheus \
            prom/prometheus:latest \
            promtool check config /prometheus/prometheus.yml
      
      - name: Test Alert Rules
        run: |
          docker run --rm -v $PWD:/workspace \
            prom/prometheus:latest \
            promtool test rules /workspace/tests/alerts-test.yml
      
      - name: Validate Grafana Dashboards
        run: |
          for file in config/grafana/dashboards/*.json; do
            echo "Validating $file"
            jq empty "$file" || exit 1
          done
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: ${{ env.TERRAFORM_VERSION }}
      
      - name: Terraform Format Check
        run: terraform fmt -check -recursive terraform/
      
      - name: Terraform Validate
        run: |
          cd terraform
          terraform init -backend=false
          terraform validate

  deploy-dev:
    name: Deploy to Development
    needs: validate
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: development
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Setup Helm
        uses: azure/setup-helm@v3
        with:
          version: ${{ env.HELM_VERSION }}
      
      - name: Configure kubeconfig
        run: |
          echo "${{ secrets.KUBECONFIG_DEV }}" | base64 -d > ~/.kube/config
      
      - name: Deploy Helm Chart
        run: |
          helm upgrade --install monitoring ./helm/monitoring \
            --namespace monitoring \
            --create-namespace \
            --values helm/monitoring/values-dev.yaml \
            --wait \
            --timeout 10m
      
      - name: Run Smoke Tests
        run: |
          kubectl wait --for=condition=ready pod \
            -l app.kubernetes.io/name=prometheus \
            -n monitoring \
            --timeout=300s
          
          kubectl wait --for=condition=ready pod \
            -l app.kubernetes.io/name=grafana \
            -n monitoring \
            --timeout=300s

  deploy-prod:
    name: Deploy to Production
    needs: validate
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Helm
        uses: azure/setup-helm@v3
        with:
          version: ${{ env.HELM_VERSION }}
      
      - name: Configure kubeconfig
        run: |
          echo "${{ secrets.KUBECONFIG_PROD }}" | base64 -d > ~/.kube/config
      
      - name: Backup current state
        run: |
          helm get values monitoring -n monitoring > backup-values.yaml
          kubectl get configmap -n monitoring -o yaml > backup-configmaps.yaml
      
      - name: Deploy Helm Chart
        run: |
          helm upgrade --install monitoring ./helm/monitoring \
            --namespace monitoring \
            --create-namespace \
            --values helm/monitoring/values-prod.yaml \
            --wait \
            --timeout 15m
      
      - name: Verify Deployment
        run: |
          kubectl rollout status deployment/monitoring-grafana -n monitoring
          kubectl rollout status statefulset/prometheus-monitoring-kube-prometheus-prometheus -n monitoring
      
      - name: Notify Slack
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Monitoring stack deployment to production: ${{ job.status }}'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

6. **Создай backup script**:

`scripts/backup-monitoring.sh`:
```bash
#!/bin/bash

set -e

# Configuration
BACKUP_DIR="/backup/monitoring"
RETENTION_DAYS=30
DATE=$(date +%Y%m%d-%H%M%S)
NAMESPACE="monitoring"

echo "Starting monitoring backup at $(date)"

# Create backup directory
mkdir -p "$BACKUP_DIR/$DATE"

# Backup Grafana
echo "Backing up Grafana..."
kubectl exec -n $NAMESPACE deployment/grafana -- \
  grafana-cli admin export-dashboard > "$BACKUP_DIR/$DATE/grafana-dashboards.json"

# Backup Grafana database
kubectl exec -n $NAMESPACE deployment/grafana -- \
  sqlite3 /var/lib/grafana/grafana.db .dump > "$BACKUP_DIR/$DATE/grafana-db.sql"

# Backup Prometheus config
echo "Backing up Prometheus configuration..."
kubectl get configmap -n $NAMESPACE prometheus-config -o yaml > \
  "$BACKUP_DIR/$DATE/prometheus-config.yaml"

# Backup Alert rules
kubectl get prometheusrule -n $NAMESPACE -o yaml > \
  "$BACKUP_DIR/$DATE/prometheus-rules.yaml"

# Backup Alertmanager config
kubectl get secret -n $NAMESPACE alertmanager-config -o yaml > \
  "$BACKUP_DIR/$DATE/alertmanager-config.yaml"

# Backup PVCs
echo "Backing up PVCs..."
kubectl get pvc -n $NAMESPACE -o yaml > "$BACKUP_DIR/$DATE/pvcs.yaml"

# Create tar archive
echo "Creating archive..."
tar -czf "$BACKUP_DIR/monitoring-backup-$DATE.tar.gz" -C "$BACKUP_DIR" "$DATE"

# Remove temporary directory
rm -rf "$BACKUP_DIR/$DATE"

# Cleanup old backups
echo "Cleaning up old backups..."
find "$BACKUP_DIR" -name "monitoring-backup-*.tar.gz" -mtime +$RETENTION_DAYS -delete

# Upload to S3 (optional)
if [ -n "$AWS_S3_BUCKET" ]; then
  echo "Uploading to S3..."
  aws s3 cp "$BACKUP_DIR/monitoring-backup-$DATE.tar.gz" \
    "s3://$AWS_S3_BUCKET/monitoring-backups/"
fi

echo "Backup completed successfully at $(date)"
echo "Backup location: $BACKUP_DIR/monitoring-backup-$DATE.tar.gz"
```

7. **Создай validation tests**:

`tests/alerts-test.yml`:
```yaml
# Unit tests для alert rules
rule_files:
  - ../config/prometheus/alerts/*.yml

evaluation_interval: 1m

tests:
  # Test HighCPUUsage alert
  - interval: 1m
    input_series:
      - series: 'node_cpu_seconds_total{mode="idle",instance="localhost:9100"}'
        values: '100+0x10'  # Idle CPU = 100 (постоянно)
      
      - series: 'node_cpu_seconds_total{mode="system",instance="localhost:9100"}'
        values: '0+10x10'   # System CPU растёт
    
    alert_rule_test:
      - eval_time: 5m
        alertname: HighCPUUsage
        exp_alerts:
          - exp_labels:
              severity: warning
              instance: localhost:9100
            exp_annotations:
              summary: "High CPU usage on localhost:9100"
  
  # Test DiskSpaceCritical alert
  - interval: 1m
    input_series:
      - series: 'node_filesystem_size_bytes{mountpoint="/",instance="localhost:9100"}'
        values: '100000000000+0x10'  # 100GB total
      
      - series: 'node_filesystem_avail_bytes{mountpoint="/",instance="localhost:9100"}'
        values: '5000000000+0x10'    # 5GB available (95% used)
    
    alert_rule_test:
      - eval_time: 5m
        alertname: DiskSpaceCritical
        exp_alerts:
          - exp_labels:
              severity: critical
              instance: localhost:9100
              mountpoint: "/"
            exp_annotations:
              summary: "Critical disk space on localhost:9100"
  
  # Test no alert when metrics are normal
  - interval: 1m
    input_series:
      - series: 'node_cpu_seconds_total{mode="idle",instance="localhost:9100"}'
        values: '100+10x10'  # Normal idle CPU
    
    alert_rule_test:
      - eval_time: 10m
        alertname: HighCPUUsage
        exp_alerts: []  # No alerts expected
```

8. **Запуск и тестирование**:
```bash
# Terraform
cd terraform
terraform init
terraform plan -var="grafana_auth=admin:admin"
terraform apply -var="grafana_auth=admin:admin"

# Ansible
cd ansible
ansible-playbook -i inventory/hosts.ini playbooks/deploy-monitoring-stack.yml

# Helm (local test)
helm install monitoring ./helm/monitoring \
  --namespace monitoring \
  --create-namespace \
  --values helm/monitoring/values-dev.yaml \
  --dry-run --debug

# Real deployment
helm install monitoring ./helm/monitoring \
  --namespace monitoring \
  --create-namespace \
  --values helm/monitoring/values-prod.yaml

# Verify
kubectl get pods -n monitoring
helm list -n monitoring

# Run tests
promtool test rules tests/alerts-test.yml

# Backup
./scripts/backup-monitoring.sh
```

### 🚀 Бонус (новое)

**1. Monitoring as Code with Jsonnet**:

`jsonnet/dashboards/application.jsonnet`:
```jsonnet
local grafana = import 'grafonnet/grafana.libsonnet';
local dashboard = grafana.dashboard;
local row = grafana.row;
local prometheus = grafana.prometheus;
local graphPanel = grafana.graphPanel;
local statPanel = grafana.statPanel;

dashboard.new(
  'Application Metrics',
  tags=['application', 'monitoring'],
  editable=true,
)
.addRow(
  row.new(title='Request Metrics')
  .addPanel(
    graphPanel.new(
      'Request Rate',
      datasource='Prometheus',
      format='reqps',
    )
    .addTarget(
      prometheus.target(
        'sum(rate(http_requests_total[5m])) by (service)',
        legendFormat='{{service}}',
      )
    )
  )
  .addPanel(
    statPanel.new(
      'Error Rate',
      datasource='Prometheus',
      unit='percentunit',
    )
    .addTarget(
      prometheus.target(
        'sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))',
      )
    )
    .addThresholds([
      { value: 0, color: 'green' },
      { value: 0.01, color: 'yellow' },
      { value: 0.05, color: 'red' },
    ])
  )
)
```

Компиляция:
```bash
jsonnet -J vendor dashboards/application.jsonnet > dashboards/application.json
```

**2. Monitoring Configuration Testing Framework**:

`tests/integration_test.py`:
```python
#!/usr/bin/env python3
"""
Integration tests для monitoring stack
"""
import requests
import time
import pytest

PROMETHEUS_URL = "http://localhost:9090"
GRAFANA_URL = "http://localhost:3000"
ALERTMANAGER_URL = "http://localhost:9093"

class TestPrometheus:
    def test_prometheus_healthy(self):
        """Test Prometheus health"""
        response = requests.get(f"{PROMETHEUS_URL}/-/healthy")
        assert response.status_code == 200
    
    def test_prometheus_targets(self):
        """Test all targets are up"""
        response = requests.get(f"{PROMETHEUS_URL}/api/v1/targets")
        data = response.json()
        
        active_targets = data['data']['activeTargets']
        down_targets = [t for t in active_targets if t['health'] != 'up']
        
        assert len(down_targets) == 0, f"Down targets: {down_targets}"
    
    def test_prometheus_rules_loaded(self):
        """Test alert rules are loaded"""
        response = requests.get(f"{PROMETHEUS_URL}/api/v1/rules")
        data = response.json()
        
        groups = data['data']['groups']
        assert len(groups) > 0, "No alert rule groups found"
    
    def test_query_works(self):
        """Test PromQL queries work"""
        query = "up"
        response = requests.get(
            f"{PROMETHEUS_URL}/api/v1/query",
            params={'query': query}
        )
        data = response.json()
        
        assert data['status'] == 'success'
        assert len(data['data']['result']) > 0

class TestGrafana:
    def test_grafana_healthy(self):
        """Test Grafana health"""
        response = requests.get(f"{GRAFANA_URL}/api/health")
        assert response.status_code == 200
    
    def test_datasources_configured(self):
        """Test datasources are configured"""
        response = requests.get(
            f"{GRAFANA_URL}/api/datasources",
            auth=('admin', 'admin')
        )
        datasources = response.json()
        
        assert len(datasources) > 0, "No datasources configured"
        
        # Check Prometheus datasource
        prometheus_ds = [ds for ds in datasources if ds['type'] == 'prometheus']
        assert len(prometheus_ds) > 0, "Prometheus datasource not found"
    
    def test_dashboards_exist(self):
        """Test dashboards are provisioned"""
        response = requests.get(
            f"{GRAFANA_URL}/api/search",
            auth=('admin', 'admin')
        )
        dashboards = response.json()
        
        assert len(dashboards) > 0, "No dashboards found"

class TestAlertmanager:
    def test_alertmanager_healthy(self):
        """Test Alertmanager health"""
        response = requests.get(f"{ALERTMANAGER_URL}/-/healthy")
        assert response.status_code == 200
    
    def test_alertmanager_config(self):
        """Test Alertmanager configuration is valid"""
        response = requests.get(f"{ALERTMANAGER_URL}/api/v2/status")
        data = response.json()
        
        assert 'config' in data
        assert 'original' in data['config']

class TestEndToEnd:
    def test_alert_flow(self):
        """Test complete alert flow"""
        # 1. Trigger alert by sending bad metrics
        # 2. Wait for alert to fire
        # 3. Check alert in Alertmanager
        # 4. Verify notification sent
        
        # Wait for alert to evaluate
        time.sleep(60)
        
        # Check for firing alerts
        response = requests.get(f"{ALERTMANAGER_URL}/api/v2/alerts")
        alerts = response.json()
        
        # Should have some alerts
        assert isinstance(alerts, list)

if __name__ == "__main__":
    pytest.main([__file__, "-v"])
```

**3. Automated Dashboard Sync**:

`scripts/sync-dashboards.py`:
```python
#!/usr/bin/env python3
"""
Sync Grafana dashboards between environments
"""
import requests
import json
import os
from pathlib import Path

class GrafanaDashboardSync:
    def __init__(self, source_url, target_url, api_key):
        self.source_url = source_url
        self.target_url = target_url
        self.headers = {'Authorization': f'Bearer {api_key}'}
    
    def export_dashboards(self, output_dir):
        """Export all dashboards from source"""
        # Search all dashboards
        response = requests.get(
            f"{self.source_url}/api/search?type=dash-db",
            headers=self.headers
        )
        dashboards = response.json()
        
        Path(output_dir).mkdir(parents=True, exist_ok=True)
        
        for dash in dashboards:
            uid = dash['uid']
            title = dash['title']
            
            # Get dashboard JSON
            response = requests.get(
                f"{self.source_url}/api/dashboards/uid/{uid}",
                headers=self.headers
            )
            dashboard_json = response.json()
            
            # Save to file
            filename = f"{output_dir}/{uid}_{title.replace(' ', '_')}.json"
            with open(filename, 'w') as f:
                json.dump(dashboard_json, f, indent=2)
            
            print(f"Exported: {title}")
    
    def import_dashboards(self, input_dir):
        """Import dashboards to target"""
        for file_path in Path(input_dir).glob('*.json'):
            with open(file_path, 'r') as f:
                dashboard_data = json.load(f)
            
            # Prepare import payload
            payload = {
                'dashboard': dashboard_data['dashboard'],
                'folderId': 0,
                'overwrite': True
            }
            
            # Import dashboard
            response = requests.post(
                f"{self.target_url}/api/dashboards/db",
                headers=self.headers,
                json=payload
            )
            
            if response.status_code == 200:
                print(f"Imported: {file_path.name}")
            else:
                print(f"Failed to import {file_path.name}: {response.text}")

if __name__ == "__main__":
    import argparse
    
    parser = argparse.ArgumentParser()
    parser.add_argument('--source', required=True)
    parser.add_argument('--target', required=True)
    parser.add_argument('--api-key', required=True)
    parser.add_argument('--export-dir', default='./dashboards')
    parser.add_argument('--action', choices=['export', 'import', 'sync'], required=True)
    
    args = parser.parse_args()
    
    sync = GrafanaDashboardSync(args.source, args.target, args.api_key)
    
    if args.action in ['export', 'sync']:
        sync.export_dashboards(args.export_dir)
    
    if args.action in ['import', 'sync']:
        sync.import_dashboards(args.export_dir)
```

**4. Cost Optimization для Cloud Monitoring**:

`terraform/modules/cost-optimization/main.tf`:
```hcl
# Intelligent tiering для Prometheus storage
resource "aws_s3_bucket" "prometheus_long_term" {
  bucket = "prometheus-long-term-storage"
  
  lifecycle_rule {
    enabled = true
    
    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }
    
    transition {
      days          = 90
      storage_class = "GLACIER"
    }
    
    expiration {
      days = 365
    }
  }
}

# Spot instances для non-critical monitoring
resource "aws_autoscaling_group" "monitoring_workers" {
  name = "monitoring-workers"
  
  mixed_instances_policy {
    launch_template {
      launch_template_specification {
        launch_template_id = aws_launch_template.monitoring.id
      }
      
      override {
        instance_type = "t3.medium"
      }
      
      override {
        instance_type = "t3a.medium"
      }
    }
    
    instances_distribution {
      on_demand_base_capacity                  = 1
      on_demand_percentage_above_base_capacity = 0
      spot_allocation_strategy                 = "capacity-optimized"
    }
  }
}
```

---

## Итоги модуля 7

После прохождения этого модуля ты должен уметь:

✅ Использовать Terraform для управления Grafana
✅ Создавать Ansible playbooks для deployment monitoring
✅ Работать с Helm charts для Kubernetes
✅ Настраивать GitOps с ArgoCD
✅ Писать CI/CD pipelines для мониторинга
✅ Создавать automated backups
✅ Тестировать monitoring конфигурацию
✅ Синхронизировать dashboards между окружениями
✅ Оптимизировать затраты на мониторинг
✅ Version control всей monitoring инфраструктуры

**Ключевые принципы:**
1. Вся инфраструктура в Git
2. Автоматизация deployment
3. Тестирование перед production
4. Регулярные backups
5. Environment parity (dev/staging/prod одинаковые)
6. Документирование изменений
7. Cost optimization

**Следующий модуль:** SLI/SLO/SLA и Error Budget - продвинутый мониторинг надежности
## Модуль 8: SLI/SLO/SLA и Error Budget - Site Reliability Engineering (40 минут)

### 🎯 Напоминалка

**SRE основные концепции:**

```
SLA (Service Level Agreement)
├─ Юридическое соглашение с клиентом
├─ Обязательство уровня сервиса
└─ Обычно: 99.9%, 99.95%, 99.99%

SLO (Service Level Objective)
├─ Внутренняя цель надежности
├─ Строже чем SLA (буфер безопасности)
└─ Используется для принятия решений

SLI (Service Level Indicator)
├─ Метрика измеряющая качество сервиса
├─ Availability, Latency, Error Rate
└─ Основа для SLO

Error Budget
├─ Допустимое количество ошибок
├─ 100% - SLO = Error Budget
└─ Баланс между надежностью и скоростью
```

**Пример расчета:**

```
SLA: 99.9% uptime в месяц
SLO: 99.95% uptime (внутренняя цель, строже SLA)

Error Budget = 100% - 99.95% = 0.05%

В месяц (30 дней):
Total time: 30 * 24 * 60 = 43,200 минут
Error Budget: 43,200 * 0.0005 = 21.6 минут downtime допустимо

Если использовали 10 минут downtime:
Remaining Budget: 21.6 - 10 = 11.6 минут
Budget consumed: 10/21.6 = 46.3%
```

**Типы SLI:**

**1. Availability (доступность):**

```
SLI = successful_requests / total_requests

Example:
- Successful: 999,500
- Failed: 500
- Total: 1,000,000
- Availability = 999,500 / 1,000,000 = 99.95%
```

**2. Latency (задержка):**

```
SLI = requests_under_threshold / total_requests

Example (threshold = 200ms):
- Under 200ms: 995,000
- Over 200ms: 5,000
- Total: 1,000,000
- Latency SLI = 995,000 / 1,000,000 = 99.5%
```

**3. Error Rate (частота ошибок):**

```
SLI = (total_requests - error_requests) / total_requests

Example:
- Total: 1,000,000
- Errors (5xx): 300
- Error Rate SLI = (1,000,000 - 300) / 1,000,000 = 99.97%
```

**4. Throughput (пропускная способность):**

```
SLI = actual_throughput >= target_throughput

Example:
- Target: 1000 req/sec
- Actual: 1050 req/sec
- SLI = 100% (meets target)
```

**SLO Types (типы целей):**

**Request-based SLO:**

yaml

```yaml
# 99.9% requests должны быть успешными
slo:
  type: request_based
  target: 99.9
  sli:
    total_query: sum(rate(http_requests_total[5m]))
    good_query: sum(rate(http_requests_total{status!~"5.."}[5m]))
```

**Window-based SLO:**

yaml

````yaml
# 99.9% времени сервис должен быть доступен
slo:
  type: window_based
  target: 99.9
  window: 30d
  sli:
    good_query: sum(up == 1)
    total_query: count(up)
```

**Multi-window SLO (Google SRE подход):**
```
Short window (1 hour):
- Alert if burn rate > 14.4x (исчерпаем budget за 2 часа)
- Severity: Critical

Medium window (6 hours):
- Alert if burn rate > 6x (исчерпаем budget за 5 дней)
- Severity: High

Long window (3 days):
- Alert if burn rate > 1x (исчерпаем budget за 30 дней)
- Severity: Warning
```

**Error Budget Policy:**
```
100% Error Budget остается:
✅ Ship новые features
✅ Делать рискованные изменения
✅ Минимальные on-call дежурства

< 50% Error Budget остается:
⚠️  Замедлить releases
⚠️  Code freeze для non-critical features
⚠️  Фокус на reliability

0% Error Budget исчерпан:
❌ Полный code freeze
❌ Только bug fixes и reliability improvements
❌ Post-mortem анализ
❌ Дополнительные on-call ресурсы
````

**SLI/SLO для разных компонентов:**

**API Service:**

yaml

```yaml
slis:
  - name: availability
    description: "Процент успешных API запросов"
    query: |
      sum(rate(http_requests_total{status!~"5.."}[5m]))
      /
      sum(rate(http_requests_total[5m]))
    
  - name: latency
    description: "P95 latency под 200ms"
    query: |
      histogram_quantile(0.95,
        rate(http_request_duration_seconds_bucket[5m])
      ) < 0.2

slos:
  - name: api_availability
    target: 99.9
    sli: availability
    window: 30d
    
  - name: api_latency
    target: 99.5
    sli: latency
    window: 30d
```

**Database:**

yaml

```yaml
slis:
  - name: query_success_rate
    query: |
      sum(rate(db_queries_total{status="success"}[5m]))
      /
      sum(rate(db_queries_total[5m]))
  
  - name: query_latency
    query: |
      histogram_quantile(0.99,
        rate(db_query_duration_seconds_bucket[5m])
      ) < 0.1

slos:
  - name: db_reliability
    target: 99.95
    sli: query_success_rate
    window: 30d
```

**Message Queue:**

yaml

```yaml
slis:
  - name: message_processing_success
    query: |
      sum(rate(queue_messages_processed_total{status="success"}[5m]))
      /
      sum(rate(queue_messages_processed_total[5m]))
  
  - name: queue_latency
    query: |
      queue_oldest_message_age_seconds < 300  # < 5 минут

slos:
  - name: queue_reliability
    target: 99.9
    sli: message_processing_success
    window: 7d
```

**Monitoring SLO Compliance:**

promql

```promql
# Текущий SLO compliance
(
  sum(rate(http_requests_total{status!~"5.."}[30d]))
  /
  sum(rate(http_requests_total[30d]))
) * 100

# Error budget remaining (в процентах)
100 - (
  (1 - (sum(rate(http_requests_total{status!~"5.."}[30d])) / sum(rate(http_requests_total[30d]))))
  /
  (1 - 0.999)  # Target SLO
) * 100

# Error budget burn rate
(
  (1 - (sum(rate(http_requests_total{status!~"5.."}[1h])) / sum(rate(http_requests_total[1h]))))
  /
  (1 - 0.999)
)
```

**Alerting на SLO нарушения:**

yaml

````yaml
# Fast burn alert (2 часа до исчерпания)
- alert: ErrorBudgetBurnRateFast
  expr: |
    (
      (1 - (sum(rate(http_requests_total{status!~"5.."}[5m])) / sum(rate(http_requests_total[5m]))))
      /
      (1 - 0.999)  # SLO target
    ) > 14.4
  for: 2m
  labels:
    severity: critical
    slo: api_availability
  annotations:
    summary: "Fast error budget burn rate detected"
    description: "At current rate, error budget will be exhausted in 2 hours"

# Slow burn alert (5 дней до исчерпания)
- alert: ErrorBudgetBurnRateSlow
  expr: |
    (
      (1 - (sum(rate(http_requests_total{status!~"5.."}[1h])) / sum(rate(http_requests_total[1h]))))
      /
      (1 - 0.999)
    ) > 6
  for: 15m
  labels:
    severity: warning
    slo: api_availability
  annotations:
    summary: "Slow error budget burn rate detected"
    description: "At current rate, error budget will be exhausted in 5 days"

# SLO violation
- alert: SLOViolation
  expr: |
    (
      sum(rate(http_requests_total{status!~"5.."}[30d]))
      /
      sum(rate(http_requests_total[30d]))
    ) < 0.999
  for: 5m
  labels:
    severity: critical
    slo: api_availability
  annotations:
    summary: "SLO violation - 30 day window"
    description: "Current availability: {{ $value | humanizePercentage }}"
```

**SLO Dashboard компоненты:**
```
1. Current SLO Status
   - Gauge: текущий SLI
   - Target line: SLO
   - Color coding: green/yellow/red

2. Error Budget
   - Gauge: оставшийся budget (%)
   - Graph: burn rate over time
   - Time to exhaustion

3. Burn Rate
   - Current burn rate (multiple windows)
   - Historical burn rate
   - Alerts status

4. Compliance History
   - 30-day rolling window
   - Daily compliance
   - Incidents impact

5. Budget Consumption
   - By incident
   - By service component
   - By time period
````

**User Journey SLO (сквозной мониторинг):**

yaml

```yaml
# Комплексный SLO для критического user journey
user_journey:
  name: "Checkout Flow"
  steps:
    - name: "Add to Cart"
      slo: 99.9
      latency_target: 200ms
      
    - name: "View Cart"
      slo: 99.9
      latency_target: 300ms
      
    - name: "Payment"
      slo: 99.95  # Строже для критической операции
      latency_target: 500ms
      
    - name: "Order Confirmation"
      slo: 99.9
      latency_target: 1000ms
  
  overall_slo: 99.7  # Композитный SLO (99.9 * 99.9 * 99.95 * 99.9)
```

**SLO Report Template:**

markdown

````markdown
# SLO Report: API Service

**Period:** 2025-01-01 to 2025-01-31

## Summary
- **SLO Target:** 99.9%
- **Actual Availability:** 99.87%
- **Status:** ⚠️ Below Target
- **Error Budget:** 100% consumed + 30% over

## SLI Breakdown

### Availability
- Target: 99.9%
- Actual: 99.87%
- Total Requests: 100,000,000
- Failed Requests: 130,000
- Success Rate: 99.87%

### Latency (P95)
- Target: < 200ms
- Actual: 185ms
- Status: ✅ Met

## Incidents

### Incident #1: Database Outage
- Date: 2025-01-15
- Duration: 45 minutes
- Impact: 100% unavailability
- Budget Consumed: 90%
- Root Cause: Primary DB failure, replication lag
- Action Items: Improve failover automation

### Incident #2: High Latency
- Date: 2025-01-22
- Duration: 2 hours
- Impact: 20% of requests over threshold
- Budget Consumed: 40%
- Root Cause: Memory leak in application
- Action Items: Add memory profiling

## Action Items
1. [ ] Implement automated database failover
2. [ ] Add continuous memory profiling
3. [ ] Increase monitoring sensitivity
4. [ ] Schedule reliability sprint

## Next Month Forecast
- If current trend continues: ❌ SLO at risk
- Recommended: Code freeze for non-critical features
```

**Tools для SLO мониторинга:**
```
1. Sloth (SLO generator)
   - Generates Prometheus rules
   - Multi-window alerts
   - Dashboard generation

2. Pyrra
   - SLO management UI
   - Error budget visualization
   - Alert configuration

3. Grafana SLO Plugin
   - Native SLO support
   - Dashboard templates
   - Integration with Prometheus

4. Google Cloud SLO Monitoring
   - Managed service
   - Built-in SLI library
   - Automated reporting

5. Datadog SLO
   - SLO tracking
   - Budget alerts
   - Integration with incidents
```

**Cost of Downtime:**
```
Расчет business impact:

E-commerce site:
- Revenue: $10M/month
- Monthly minutes: 43,200
- Revenue per minute: $231.48
- 1 hour downtime = $13,888 loss

SaaS application:
- Customers: 10,000
- Churn rate при downtime: 2%
- Average LTV: $5,000
- 1 hour downtime = 200 churned customers = $1M loss

Developer productivity:
- Developers: 50
- Hourly cost: $100/hour
- Blocked time per outage: 2 hours
- Cost: 50 * 100 * 2 = $10,000
````

### 💻 Задание

Настрой полноценный SLO мониторинг:

1. **Создай SLO конфигурацию с Sloth**:

`slos/api-service.yaml`:

yaml

```yaml
version: "prometheus/v1"
service: "api-service"
labels:
  owner: "backend-team"
  tier: "critical"
slos:
  # Availability SLO
  - name: "requests-availability"
    objective: 99.9
    description: "API requests должны быть успешными"
    sli:
      events:
        error_query: sum(rate(http_requests_total{job="api",status=~"(5..|429)"}[{{.window}}]))
        total_query: sum(rate(http_requests_total{job="api"}[{{.window}}]))
    alerting:
      name: ApiHighErrorRate
      labels:
        category: "availability"
      annotations:
        summary: "High error rate on API service"
      page_alert:
        labels:
          severity: critical
      ticket_alert:
        labels:
          severity: warning
  
  # Latency SLO
  - name: "requests-latency"
    objective: 99.5
    description: "95% requests должны выполняться за < 200ms"
    sli:
      events:
        error_query: |
          (
            sum(rate(http_request_duration_seconds_count{job="api"}[{{.window}}]))
            -
            sum(rate(http_request_duration_seconds_bucket{job="api",le="0.2"}[{{.window}}]))
          )
        total_query: sum(rate(http_request_duration_seconds_count{job="api"}[{{.window}}]))
    alerting:
      name: ApiHighLatency
      labels:
        category: "latency"
      annotations:
        summary: "High latency on API service"
      page_alert:
        labels:
          severity: critical
      ticket_alert:
        labels:
          severity: warning
```

`slos/database.yaml`:

yaml

```yaml
version: "prometheus/v1"
service: "postgresql"
labels:
  owner: "platform-team"
  tier: "critical"
slos:
  # Database availability
  - name: "connection-availability"
    objective: 99.95
    description: "Database должна быть доступна для подключений"
    sli:
      events:
        error_query: sum(rate(pg_up{job="postgres"}[{{.window}}]) == 0)
        total_query: count(pg_up{job="postgres"})
    alerting:
      name: DatabaseUnavailable
      labels:
        category: "availability"
      page_alert:
        labels:
          severity: critical
  
  # Query performance
  - name: "query-performance"
    objective: 99.9
    description: "Queries должны выполняться за < 100ms"
    sli:
      events:
        error_query: |
          sum(rate(pg_stat_statements_mean_exec_time{job="postgres"}[{{.window}}])) > 100
        total_query: sum(rate(pg_stat_statements_calls{job="postgres"}[{{.window}}]))
    alerting:
      name: DatabaseSlowQueries
      labels:
        category: "performance"
      ticket_alert:
        labels:
          severity: warning
```

2. **Генерация Prometheus rules с Sloth**:

bash

```bash
# Установка Sloth
go install github.com/slok/sloth/cmd/sloth@latest

# Генерация правил
sloth generate -i slos/api-service.yaml -o prometheus/rules/api-slo.yaml
sloth generate -i slos/database.yaml -o prometheus/rules/database-slo.yaml

# Валидация
promtool check rules prometheus/rules/*.yaml
```

3. **Создай SLO Dashboard в Grafana**:

`dashboards/slo-overview.json`:

json

```json
{
  "dashboard": {
    "title": "SLO Overview",
    "tags": ["slo", "sre"],
    "timezone": "browser",
    "rows": [
      {
        "title": "SLO Status",
        "panels": [
          {
            "id": 1,
            "title": "API Availability SLO",
            "type": "gauge",
            "targets": [
              {
                "expr": "(\n  sum(rate(http_requests_total{job=\"api\",status!~\"(5..|429)\"}[30d]))\n  /\n  sum(rate(http_requests_total{job=\"api\"}[30d]))\n) * 100",
                "legendFormat": "Current"
              }
            ],
            "fieldConfig": {
              "defaults": {
                "unit": "percent",
                "min": 99,
                "max": 100,
                "thresholds": {
                  "steps": [
                    {"value": 99, "color": "red"},
                    {"value": 99.9, "color": "yellow"},
                    {"value": 99.95, "color": "green"}
                  ]
                }
              }
            }
          },
          {
            "id": 2,
            "title": "Error Budget Remaining",
            "type": "stat",
            "targets": [
              {
                "expr": "100 - (\n  (1 - (sum(rate(http_requests_total{job=\"api\",status!~\"(5..|429)\"}[30d])) / sum(rate(http_requests_total{job=\"api\"}[30d]))))\n  /\n  (1 - 0.999)\n) * 100",
                "legendFormat": "Budget Remaining"
              }
            ],
            "fieldConfig": {
              "defaults": {
                "unit": "percent",
                "thresholds": {
                  "steps": [
                    {"value": 0, "color": "red"},
                    {"value": 25, "color": "yellow"},
                    {"value": 50, "color": "green"}
                  ]
                }
              }
            }
          },
          {
            "id": 3,
            "title": "Burn Rate (1h window)",
            "type": "timeseries",
            "targets": [
              {
                "expr": "(\n  (1 - (sum(rate(http_requests_total{job=\"api\",status!~\"(5..|429)\"}[1h])) / sum(rate(http_requests_total{job=\"api\"}[1h]))))\n  /\n  (1 - 0.999)\n)",
                "legendFormat": "Burn Rate"
              },
              {
                "expr": "14.4",
                "legendFormat": "Critical Threshold (2h to exhaustion)"
              },
              {
                "expr": "6",
                "legendFormat": "Warning Threshold (5d to exhaustion)"
              }
            ]
          }
        ]
      },
      {
        "title": "SLO Compliance History",
        "panels": [
          {
            "id": 4,
            "title": "30-Day Rolling Availability",
            "type": "timeseries",
            "targets": [
              {
                "expr": "(\n  sum(rate(http_requests_total{job=\"api\",status!~\"(5..|429)\"}[30d]))\n  /\n  sum(rate(http_requests_total{job=\"api\"}[30d]))\n) * 100",
                "legendFormat": "Availability"
              },
              {
                "expr": "99.9",
                "legendFormat": "SLO Target"
              }
            ],
            "fieldConfig": {
              "defaults": {
                "unit": "percent",
                "min": 99,
                "max": 100
              }
            }
          },
          {
            "id": 5,
            "title": "Error Budget Consumption",
            "type": "bargauge",
            "targets": [
              {
                "expr": "(\n  (1 - (sum(rate(http_requests_total{job=\"api\",status!~\"(5..|429)\"}[30d])) / sum(rate(http_requests_total{job=\"api\"}[30d]))))\n  /\n  (1 - 0.999)\n) * 100",
                "legendFormat": "Budget Used"
              }
            ],
            "fieldConfig": {
              "defaults": {
                "unit": "percent",
                "max": 100,
                "thresholds": {
                  "steps": [
                    {"value": 0, "color": "green"},
                    {"value": 75, "color": "yellow"},
                    {"value": 100, "color": "red"}
                  ]
                }
              }
            }
          }
        ]
      }
    ]
  }
}
```

4. **Создай Error Budget Policy документ**:

`docs/error-budget-policy.md`:

markdown

````markdown
# Error Budget Policy

## Overview
This document defines how we use error budgets to balance reliability and feature velocity.

## SLO Targets

| Service | SLO Target | Error Budget (30d) | Error Budget (minutes) |
|---------|------------|-------------------|----------------------|
| API Service | 99.9% | 0.1% | 43.2 minutes |
| Database | 99.95% | 0.05% | 21.6 minutes |
| Message Queue | 99.9% | 0.1% | 43.2 minutes |
| CDN | 99.99% | 0.01% | 4.3 minutes |

## Policy Levels

### 🟢 Level 1: Budget Healthy (> 50% remaining)

**Allowed Activities:**
- ✅ Normal release cadence
- ✅ Experimental features
- ✅ Performance optimizations
- ✅ Refactoring

**Requirements:**
- Standard change management process
- Automated testing
- Gradual rollouts

### 🟡 Level 2: Budget Warning (25-50% remaining)

**Allowed Activities:**
- ⚠️ Reduced release frequency
- ⚠️ Critical features only
- ⚠️ Additional testing required

**Requirements:**
- Senior engineer approval for changes
- Extended canary periods
- Increased monitoring
- Daily budget reviews

**Actions:**
- Conduct incident review
- Identify systemic issues
- Create reliability improvement tasks
- Schedule reliability sprint

### 🔴 Level 3: Budget Critical (< 25% remaining)

**Allowed Activities:**
- ❌ Feature freeze
- ✅ Bug fixes only
- ✅ Reliability improvements
- ✅ Emergency security patches

**Requirements:**
- Director-level approval for any changes
- Mandatory post-mortems for all incidents
- 24/7 on-call rotation
- Hourly budget monitoring

**Actions:**
- Emergency reliability review
- All hands on stability
- External communication about delays
- Executive escalation

### ⛔ Level 4: Budget Exhausted (0% remaining)

**Allowed Activities:**
- ❌ Complete code freeze
- ✅ Critical bug fixes only (with VP approval)
- ✅ Incident response

**Requirements:**
- VP Engineering approval for ANY change
- Full post-mortem for budget exhaustion
- Recovery plan required before resuming features
- Daily executive updates

**Actions:**
- Immediate incident declared
- Full team mobilization
- Customer communication
- Systematic root cause analysis
- Multi-week recovery plan

## Escalation
```
Budget < 50% → Team Lead notified
Budget < 25% → Engineering Manager notified
Budget < 10% → Director notified
Budget exhausted → VP Engineering notified
```

## Review Process

- **Daily:** Automated budget reports
- **Weekly:** Team review of budget trends
- **Monthly:** SLO report to stakeholders
- **Quarterly:** Policy review and adjustment

## Exceptions

Exceptions to this policy require:
1. Written justification
2. Risk assessment
3. Approval from Director of Engineering
4. Documentation in incident log

## Contact

- **Policy Owner:** SRE Team
- **Questions:** #sre-team Slack channel
- **Escalations:** oncall-sre@company.com
````

5. **Создай автоматический SLO reporter**:

`scripts/slo-report.py`:

python

```python
#!/usr/bin/env python3
"""
Automated SLO Report Generator
"""
import requests
from datetime import datetime, timedelta
import json

class SLOReporter:
    def __init__(self, prometheus_url, period_days=30):
        self.prometheus_url = prometheus_url
        self.period_days = period_days
        self.slos = self.load_slo_config()
    
    def load_slo_config(self):
        """Load SLO configuration"""
        return {
            'api-availability': {
                'name': 'API Availability',
                'target': 99.9,
                'query': '''
                    (
                      sum(rate(http_requests_total{job="api",status!~"(5..|429)"}[30d]))
                      /
                      sum(rate(http_requests_total{job="api"}[30d]))
                    ) * 100
                '''
            },
            'api-latency': {
                'name': 'API Latency (P95 < 200ms)',
                'target': 99.5,
                'query': '''
                    (
                      sum(rate(http_request_duration_seconds_bucket{job="api",le="0.2"}[30d]))
                      /
                      sum(rate(http_request_duration_seconds_count{job="api"}[30d]))
                    ) * 100
                '''
            },
            'database-availability': {
                'name': 'Database Availability',
                'target': 99.95,
                'query': '''
                    (sum(up{job="postgres"}) / count(up{job="postgres"})) * 100
                '''
            }
        }
    
    def query_prometheus(self, query):
        """Execute Prometheus query"""
        response = requests.get(
            f"{self.prometheus_url}/api/v1/query",
            params={'query': query}
        )
        data = response.json()
        
        if data['status'] == 'success' and data['data']['result']:
            return float(data['data']['result'][0]['value'][1])
        return None
    
    def calculate_error_budget(self, actual, target):
        """Calculate error budget consumption"""
        total_budget = 100 - target
        consumed = target - actual
        
        if consumed < 0:
            return 0.0  # Over-performing
        
        budget_consumed_pct = (consumed / total_budget) * 100
        return min(budget_consumed_pct, 100.0)
    
    def get_downtime_minutes(self, availability_pct):
        """Calculate downtime in minutes"""
        total_minutes = self.period_days * 24 * 60
        uptime_minutes = total_minutes * (availability_pct / 100)
        downtime_minutes = total_minutes - uptime_minutes
        return downtime_minutes
    
    def determine_status(self, actual, target):
        """Determine SLO status"""
        if actual >= target:
            return "✅ Met"
        elif actual >= target - 0.05:
            return "⚠️ At Risk"
        else:
            return "❌ Violated"
    
    def generate_report(self):
        """Generate complete SLO report"""
        report = {
            'period': f"{self.period_days} days",
            'generated_at': datetime.now().isoformat(),
            'slos': {}
        }
        
        for slo_id, slo_config in self.slos.items():
            actual = self.query_prometheus(slo_config['query'])
            
            if actual is None:
                continue
            
            target = slo_config['target']
            budget_consumed = self.calculate_error_budget(actual, target)
            status = self.determine_status(actual, target)
            downtime = self.get_downtime_minutes(actual)
            
            report['slos'][slo_id] = {
                'name': slo_config['name'],
                'target': target,
                'actual': round(actual, 4),
                'status': status,
                'error_budget_consumed': round(budget_consumed, 2),
                'error_budget_remaining': round(100 - budget_consumed, 2),
                'downtime_minutes': round(downtime, 2)
            }
        
        return report
    
    def format_markdown(self, report):
        """Format report as Markdown"""
        md = f"""# SLO Report

**Period:** {report['period']}  
**Generated:** {report['generated_at']}

## Summary

| SLO | Target | Actual | Status | Budget Used | Budget Remaining | Downtime |
|-----|--------|--------|--------|-------------|-----------------|----------|
"""
        
        for slo_id, slo_data in report['slos'].items():
            md += f"| {slo_data['name']} "
            md += f"| {slo_data['target']}% "
            md += f"| {slo_data['actual']}% " 
            md += f"| {slo_data['status']} "               
            md += f"| {slo_data['error_budget_consumed']}% " 
            md += f"| {slo_data['error_budget_remaining']}% " 
            md += f"| {slo_data['downtime_minutes']:.1f} min |\n"


    # Add recommendations
    md += "\n## Recommendations\n\n"
    
    for slo_id, slo_data in report['slos'].items():
        if slo_data['error_budget_remaining'] < 25:
            md += f"- ⚠️ **{slo_data['name']}**: Error budget critical. Implement code freeze.\n"
        elif slo_data['error_budget_remaining'] < 50:
            md += f"- ⚠️ **{slo_data['name']}**: Error budget warning. Reduce release frequency.\n"
    
    return md

def send_to_slack(self, report, webhook_url):
    """Send report to Slack"""
    # Implementation for Slack webhook
    pass


if **name** == "**main**": import argparse


parser = argparse.ArgumentParser()
parser.add_argument('--prometheus-url', default='http://localhost:9090')
parser.add_argument('--period-days', type=int, default=30)
parser.add_argument('--format', choices=['json', 'markdown'], default='markdown')
parser.add_argument('--output', help='Output file')

args = parser.parse_args()

reporter = SLOReporter(args.prometheus_url, args.period_days)
report = reporter.generate_report()

if args.format == 'json':
    output = json.dumps(report, indent=2)
else:
    output = reporter.format_markdown(report)

if args.output:
    with open(args.output, 'w') as f:
        f.write(output)
else:
    print(output)
```



Использование:
```bash
# Генерация отчета
python scripts/slo-report.py --prometheus-url http://localhost:9090

# Сохранение в файл
python scripts/slo-report.py --output reports/slo-report-$(date +%Y%m%d).md

# JSON формат
python scripts/slo-report.py --format json --output reports/slo-report.json
```

6. **Создай автоматизацию для Error Budget Policy**:

`scripts/error-budget-enforcer.py`:
```python
#!/usr/bin/env python3
"""
Error Budget Policy Enforcer
Автоматически применяет политики на основе остатка error budget
"""
import requests
import sys

class ErrorBudgetEnforcer:
    def __init__(self, prometheus_url, github_token):
        self.prometheus_url = prometheus_url
        self.github_token = github_token
        self.thresholds = {
            'healthy': 50,
            'warning': 25,
            'critical': 10,
            'exhausted': 0
        }
    
    def get_error_budget_remaining(self):
        """Get current error budget remaining percentage"""
        query = '''
            100 - (
              (1 - (sum(rate(http_requests_total{job="api",status!~"(5..|429)"}[30d])) / sum(rate(http_requests_total{job="api"}[30d]))))
              /
              (1 - 0.999)
            ) * 100
        '''
        
        response = requests.get(
            f"{self.prometheus_url}/api/v1/query",
            params={'query': query}
        )
        data = response.json()
        
        if data['status'] == 'success' and data['data']['result']:
            return float(data['data']['result'][0]['value'][1])
        return None
    
    def determine_level(self, budget_remaining):
        """Determine current policy level"""
        if budget_remaining > self.thresholds['healthy']:
            return 'healthy', '🟢'
        elif budget_remaining > self.thresholds['warning']:
            return 'warning', '🟡'
        elif budget_remaining > self.thresholds['critical']:
            return 'critical', '🔴'
        else:
            return 'exhausted', '⛔'
    
    def enable_github_protections(self, level):
        """Enable GitHub branch protections based on level"""
        # Implementation for GitHub API
        protections = {
            'healthy': {
                'required_approving_review_count': 1,
                'dismiss_stale_reviews': False
            },
            'warning': {
                'required_approving_review_count': 2,
                'dismiss_stale_reviews': True
            },
            'critical': {
                'required_approving_review_count': 3,
                'dismiss_stale_reviews': True,
                'require_code_owner_reviews': True
            },
            'exhausted': {
                'required_approving_review_count': 4,
                'dismiss_stale_reviews': True,
                'require_code_owner_reviews': True,
                'required_status_checks': ['director-approval']
            }
        }
        
        return protections.get(level, protections['healthy'])
    
    def send_notification(self, level, budget_remaining, icon):
        """Send notification to team"""
        messages = {
            'healthy': f"{icon} Error budget healthy: {budget_remaining:.1f}% remaining",
            'warning': f"{icon} Error budget warning: {budget_remaining:.1f}% remaining. Reduce release frequency.",
            'critical': f"{icon} Error budget critical: {budget_remaining:.1f}% remaining. Feature freeze recommended.",
            'exhausted': f"{icon} Error budget exhausted! Complete code freeze in effect."
        }
        
        print(messages[level])
        # Implementation for Slack/Email notifications
    
    def enforce(self):
        """Enforce error budget policy"""
        budget_remaining = self.get_error_budget_remaining()
        
        if budget_remaining is None:
            print("❌ Could not retrieve error budget data")
            sys.exit(1)
        
        level, icon = self.determine_level(budget_remaining)
        
        print(f"\n{icon} Current Error Budget: {budget_remaining:.2f}%")
        print(f"Policy Level: {level.upper()}")
        
        # Apply protections
        protections = self.enable_github_protections(level)
        print(f"\nGitHub Protections: {protections}")
        
        # Send notifications
        self.send_notification(level, budget_remaining, icon)
        
        return level, budget_remaining

if __name__ == "__main__":
    import os
    
    prometheus_url = os.getenv('PROMETHEUS_URL', 'http://localhost:9090')
    github_token = os.getenv('GITHUB_TOKEN')
    
    enforcer = ErrorBudgetEnforcer(prometheus_url, github_token)
    level, budget = enforcer.enforce()
    
    # Exit code based on level
    exit_codes = {
        'healthy': 0,
        'warning': 1,
        'critical': 2,
        'exhausted': 3
    }
    
    sys.exit(exit_codes.get(level, 0))
```

Автоматизация через cron:
```bash
# /etc/cron.d/error-budget-enforcer
*/15 * * * * /usr/local/bin/error-budget-enforcer.py >> /var/log/error-budget.log 2>&1
```

### 🚀 Бонус (новое)

**1. Composite SLO (комплексный SLO для user journey)**:
```python
# composite_slo.py
class CompositeSLO:
    """
    Вычисление composite SLO для multi-step user journey
    """
    def __init__(self, steps):
        self.steps = steps
    
    def calculate_composite_slo(self):
        """
        Composite SLO = произведение SLO всех шагов
        """
        composite = 1.0
        for step in self.steps:
            composite *= (step['slo'] / 100)
        return composite * 100
    
    def calculate_step_targets(self, target_composite_slo):
        """
        Вычислить необходимые SLO для каждого шага
        чтобы достичь целевого composite SLO
        """
        num_steps = len(self.steps)
        # Равномерное распределение
        step_slo = (target_composite_slo / 100) ** (1 / num_steps) * 100
        return step_slo

# Example
checkout_flow = CompositeSLO([
    {'name': 'Add to Cart', 'slo': 99.9},
    {'name': 'View Cart', 'slo': 99.9},
    {'name': 'Payment', 'slo': 99.95},
    {'name': 'Confirmation', 'slo': 99.9}
])

composite = checkout_flow.calculate_composite_slo()
print(f"Composite SLO: {composite:.2f}%")
# Output: 99.65%

# Если хотим 99.9% composite, какой SLO нужен на каждом шаге?
required_step_slo = checkout_flow.calculate_step_targets(99.9)
print(f"Required per-step SLO: {required_step_slo:.3f}%")
# Output: 99.975%
```

**2. SLO as Code с Terraform**:
```hcl
# terraform/slo/main.tf
resource "grafana_slo" "api_availability" {
  name        = "API Availability"
  description = "API requests должны быть успешными"
  
  query {
    type = "prometheus"
    
    # Success metric
    success {
      metric = "http_requests_total"
      filters = {
        job    = "api"
        status = "!~(5..|429)"
      }
    }
    
    # Total metric
    total {
      metric = "http_requests_total"
      filters = {
        job = "api"
      }
    }
  }
  
  objectives {
    value  = 99.9
    window = "30d"
  }
  
  alerting {
    fast_burn {
      enabled   = true
      threshold = 14.4
      window    = "1h"
    }
    
    slow_burn {
      enabled   = true
      threshold = 6
      window    = "6h"
    }
  }
  
  labels = {
    team     = "backend"
    service  = "api"
    tier     = "critical"
  }
}
```

**3. SLO Simulator для testing**:
```python
# slo_simulator.py
import random
from datetime import datetime, timedelta

class SLOSimulator:
    """
    Симулятор для тестирования SLO policies
    """
    def __init__(self, target_slo, total_requests_per_day):
        self.target_slo = target_slo
        self.total_requests_per_day = total_requests_per_day
        self.error_budget = 100 - target_slo
    
    def simulate_month(self, incident_scenarios):
        """
        Симулировать месяц работы с различными сценариями инцидентов
        """
        days = 30
        total_requests = self.total_requests_per_day * days
        allowed_failures = total_requests * (self.error_budget / 100)
        
        print(f"Simulation Parameters:")
        print(f"  Target SLO: {self.target_slo}%")
        print(f"  Error Budget: {self.error_budget}%")
        print(f"  Total Requests (30d): {total_requests:,}")
        print(f"  Allowed Failures: {allowed_failures:,.0f}")
        print()
        
        actual_failures = 0
        
        for scenario in incident_scenarios:
            duration_minutes = scenario['duration_minutes']
            failure_rate = scenario['failure_rate']
            
            requests_during_incident = (duration_minutes / 1440) * self.total_requests_per_day
            failures = requests_during_incident * failure_rate
            
            actual_failures += failures
            
            print(f"Incident: {scenario['name']}")
            print(f"  Duration: {duration_minutes} minutes")
            print(f"  Failure Rate: {failure_rate * 100}%")
            print(f"  Requests Affected: {requests_during_incident:,.0f}")
            print(f"  Failures: {failures:,.0f}")
            print()
        
        actual_slo = ((total_requests - actual_failures) / total_requests) * 100
        budget_consumed = (actual_failures / allowed_failures) * 100
        
        print(f"Results:")
        print(f"  Actual SLO: {actual_slo:.4f}%")
        print(f"  Budget Consumed: {budget_consumed:.1f}%")
        print(f"  Status: {'✅ Met' if actual_slo >= self.target_slo else '❌ Violated'}")
        
        return actual_slo, budget_consumed

# Example usage
simulator = SLOSimulator(target_slo=99.9, total_requests_per_day=10_000_000)

incidents = [
    {
        'name': 'Database Outage',
        'duration_minutes': 30,
        'failure_rate': 1.0  # 100% failure
    },
    {
        'name': 'High Latency Event',
        'duration_minutes': 120,
        'failure_rate': 0.2  # 20% failure
    },
    {
        'name': 'Partial Service Degradation',
        'duration_minutes': 60,
        'failure_rate': 0.5  # 50% failure
    }
]

simulator.simulate_month(incidents)
````

---

## Итоги модуля 8

После прохождения этого модуля ты должен уметь:

✅ Определять правильные SLI для сервисов
✅ Устанавливать реалистичные SLO targets
✅ Вычислять и мониторить Error Budget
✅ Настраивать multi-window alerting
✅ Создавать SLO dashboards
✅ Генерировать автоматические SLO reports
✅ Применять Error Budget Policy
✅ Вычислять composite SLO
✅ Балансировать reliability и velocity
✅ Принимать data-driven решения о releases

**Ключевые принципы SRE:**
1. SLO определяет target reliability
2. Error Budget позволяет балансировать риск
3. Измеряй то, что важно пользователям
4. Автоматизируй enforcement policies
5. Используй данные для принятия решений
6. Регулярно пересматривай SLO targets
7. Документируй и коммуницируй статус

**Следующий модуль:** Финальный проект - построение complete monitoring solution



## Финальный Модуль: Complete Monitoring Solution - Production-Ready Stack (90 минут)

### 🎯 Цель

Построить production-ready monitoring solution, объединяющую все знания из предыдущих модулей:

- Metrics (Prometheus)
- Logs (Loki)
- Traces (Jaeger/Tempo)
- Dashboards (Grafana)
- Alerting (Alertmanager)
- SLO Monitoring
- Infrastructure as Code
- Automation

### 📋 Архитектура проекта

```
Complete Monitoring Stack для E-commerce платформы:

┌─────────────────────────────────────────────────────────────┐
│                    Production Environment                    │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Frontend │  │   API    │  │ Payment  │  │ Inventory│   │
│  │ Service  │→→│ Gateway  │→→│ Service  │  │ Service  │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │
│       │             │              │              │          │
│       └─────────────┴──────────────┴──────────────┘          │
│                         │                                     │
│                         ↓                                     │
│              ┌──────────────────────┐                        │
│              │   PostgreSQL + Redis │                        │
│              └──────────────────────┘                        │
└─────────────────────────────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Monitoring Stack                          │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Prometheus  │  │     Loki     │  │    Tempo     │     │
│  │  + Thanos    │  │              │  │              │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                  │                  │              │
│         └──────────────────┴──────────────────┘              │
│                         │                                     │
│                         ↓                                     │
│              ┌──────────────────────┐                        │
│              │      Grafana         │                        │
│              │  + SLO Dashboards    │                        │
│              └──────────────────────┘                        │
│                                                               │
│  ┌──────────────┐           ┌──────────────┐               │
│  │ Alertmanager │──────────→│ Notifications│               │
│  │              │           │ Slack/PagerDuty              │
│  └──────────────┘           └──────────────┘               │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Проект: E-commerce Monitoring

Создадим полноценный monitoring для e-commerce платформы с:

- 4 микросервиса (Frontend, API, Payment, Inventory)
- База данных (PostgreSQL) и кэш (Redis)
- Full observability (metrics, logs, traces)
- SLO monitoring
- Automated alerting
- GitOps deployment

---

## Часть 1: Подготовка инфраструктуры (20 минут)

### 1.1 Структура проекта

bash

```bash
ecommerce-monitoring/
├── README.md
├── docker-compose.yml
├── .env.example
│
├── applications/
│   ├── frontend/
│   │   ├── Dockerfile
│   │   ├── app.py
│   │   └── requirements.txt
│   ├── api-gateway/
│   │   ├── Dockerfile
│   │   ├── app.js
│   │   └── package.json
│   ├── payment-service/
│   │   ├── Dockerfile
│   │   ├── main.go
│   │   └── go.mod
│   └── inventory-service/
│       ├── Dockerfile
│       ├── app.py
│       └── requirements.txt
│
├── monitoring/
│   ├── prometheus/
│   │   ├── prometheus.yml
│   │   ├── alerts/
│   │   │   ├── infrastructure.yml
│   │   │   ├── application.yml
│   │   │   └── slo.yml
│   │   └── recording-rules/
│   │       └── slo-rules.yml
│   ├── loki/
│   │   └── loki-config.yml
│   ├── tempo/
│   │   └── tempo-config.yml
│   ├── grafana/
│   │   ├── datasources/
│   │   │   └── datasources.yml
│   │   ├── dashboards/
│   │   │   ├── overview.json
│   │   │   ├── slo-dashboard.json
│   │   │   ├── service-mesh.json
│   │   │   └── business-metrics.json
│   │   └── provisioning/
│   ├── alertmanager/
│   │   └── alertmanager.yml
│   └── otel-collector/
│       └── otel-config.yml
│
├── slo/
│   ├── api-gateway.yaml
│   ├── payment-service.yaml
│   └── inventory-service.yaml
│
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── grafana.tf
│   └── alerts.tf
│
├── scripts/
│   ├── deploy.sh
│   ├── generate-load.sh
│   ├── backup.sh
│   ├── slo-report.py
│   └── chaos-test.sh
│
└── docs/
    ├── architecture.md
    ├── runbooks/
    │   ├── high-error-rate.md
    │   ├── database-down.md
    │   └── slo-violation.md
    └── slo-policy.md
```

### 1.2 Docker Compose - Complete Stack

`docker-compose.yml`:

yaml

```yaml
version: '3.8'

networks:
  monitoring:
    driver: bridge
  app:
    driver: bridge

volumes:
  prometheus-data:
  grafana-data:
  loki-data:
  tempo-data:
  postgres-data:
  redis-data:

services:
  # ============================================
  # Application Services
  # ============================================
  
  frontend:
    build: ./applications/frontend
    container_name: frontend
    ports:
      - "8080:8080"
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
      - OTEL_SERVICE_NAME=frontend
      - API_GATEWAY_URL=http://api-gateway:8081
    networks:
      - app
      - monitoring
    depends_on:
      - api-gateway
    restart: unless-stopped

  api-gateway:
    build: ./applications/api-gateway
    container_name: api-gateway
    ports:
      - "8081:8081"
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
      - OTEL_SERVICE_NAME=api-gateway
      - PAYMENT_SERVICE_URL=http://payment-service:8082
      - INVENTORY_SERVICE_URL=http://inventory-service:8083
    networks:
      - app
      - monitoring
    depends_on:
      - payment-service
      - inventory-service
    restart: unless-stopped

  payment-service:
    build: ./applications/payment-service
    container_name: payment-service
    ports:
      - "8082:8082"
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
      - OTEL_SERVICE_NAME=payment-service
      - DATABASE_URL=postgres://user:password@postgres:5432/ecommerce
      - REDIS_URL=redis://redis:6379
    networks:
      - app
      - monitoring
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  inventory-service:
    build: ./applications/inventory-service
    container_name: inventory-service
    ports:
      - "8083:8083"
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
      - OTEL_SERVICE_NAME=inventory-service
      - DATABASE_URL=postgres://user:password@postgres:5432/ecommerce
      - REDIS_URL=redis://redis:6379
    networks:
      - app
      - monitoring
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  # ============================================
  # Data Stores
  # ============================================

  postgres:
    image: postgres:16-alpine
    container_name: postgres
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=ecommerce
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init-db.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app
      - monitoring
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    container_name: redis
    ports:
      - "6379:6379"
    networks:
      - app
      - monitoring
    restart: unless-stopped

  # ============================================
  # Monitoring Stack
  # ============================================

  prometheus:
    image: prom/prometheus:v2.48.0
    container_name: prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
      - '--web.enable-lifecycle'
      - '--web.enable-admin-api'
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./monitoring/prometheus/alerts:/etc/prometheus/alerts
      - ./monitoring/prometheus/recording-rules:/etc/prometheus/recording-rules
      - prometheus-data:/prometheus
    networks:
      - monitoring
    restart: unless-stopped

  loki:
    image: grafana/loki:2.9.3
    container_name: loki
    command: -config.file=/etc/loki/config.yml
    ports:
      - "3100:3100"
    volumes:
      - ./monitoring/loki/loki-config.yml:/etc/loki/config.yml
      - loki-data:/loki
    networks:
      - monitoring
    restart: unless-stopped

  promtail:
    image: grafana/promtail:2.9.3
    container_name: promtail
    command: -config.file=/etc/promtail/config.yml
    volumes:
      - ./monitoring/promtail/promtail-config.yml:/etc/promtail/config.yml
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    networks:
      - monitoring
    depends_on:
      - loki
    restart: unless-stopped

  tempo:
    image: grafana/tempo:latest
    container_name: tempo
    command: [ "-config.file=/etc/tempo.yml" ]
    ports:
      - "3200:3200"   # Tempo HTTP
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
    volumes:
      - ./monitoring/tempo/tempo-config.yml:/etc/tempo.yml
      - tempo-data:/tmp/tempo
    networks:
      - monitoring
    restart: unless-stopped

  otel-collector:
    image: otel/opentelemetry-collector-contrib:0.91.0
    container_name: otel-collector
    command: ["--config=/etc/otel-collector-config.yml"]
    ports:
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
      - "8888:8888"   # Prometheus metrics
    volumes:
      - ./monitoring/otel-collector/otel-config.yml:/etc/otel-collector-config.yml
    networks:
      - monitoring
    depends_on:
      - prometheus
      - loki
      - tempo
    restart: unless-stopped

  grafana:
    image: grafana/grafana:10.2.3
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_FEATURE_TOGGLES_ENABLE=traceqlEditor,correlations
      - GF_UNIFIED_ALERTING_ENABLED=true
      - GF_INSTALL_PLUGINS=grafana-piechart-panel,grafana-worldmap-panel
    volumes:
      - grafana-data:/var/lib/grafana
      - ./monitoring/grafana/datasources:/etc/grafana/provisioning/datasources
      - ./monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./monitoring/grafana/provisioning:/etc/grafana/provisioning
    networks:
      - monitoring
    depends_on:
      - prometheus
      - loki
      - tempo
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:v0.26.0
    container_name: alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    ports:
      - "9093:9093"
    volumes:
      - ./monitoring/alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
    networks:
      - monitoring
    restart: unless-stopped

  # ============================================
  # Exporters
  # ============================================

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    command:
      - '--path.rootfs=/host'
    ports:
      - "9100:9100"
    volumes:
      - '/:/host:ro,rslave'
    pid: host
    networks:
      - monitoring
    restart: unless-stopped

  postgres-exporter:
    image: prometheuscommunity/postgres-exporter:latest
    container_name: postgres-exporter
    environment:
      - DATA_SOURCE_NAME=postgresql://user:password@postgres:5432/ecommerce?sslmode=disable
    ports:
      - "9187:9187"
    networks:
      - monitoring
    depends_on:
      - postgres
    restart: unless-stopped

  redis-exporter:
    image: oliver006/redis_exporter:latest
    container_name: redis-exporter
    environment:
      - REDIS_ADDR=redis://redis:6379
    ports:
      - "9121:9121"
    networks:
      - monitoring
    depends_on:
      - redis
    restart: unless-stopped

  # ============================================
  # Load Generator (для тестирования)
  # ============================================

  load-generator:
    image: grafana/k6:latest
    container_name: load-generator
    volumes:
      - ./scripts/load-test.js:/scripts/load-test.js
    command: run --vus 10 --duration 24h /scripts/load-test.js
    networks:
      - app
    depends_on:
      - frontend
    restart: unless-stopped
```

---

## Часть 2: Application Services с Instrumentation (25 минут)

### 2.1 Frontend Service (Python Flask)

`applications/frontend/app.py`:

python

```python
from flask import Flask, render_template_string, jsonify, request
from opentelemetry import trace, metrics
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk.metrics.export import PeriodicExportingMetricReader
from opentelemetry.exporter.otlp.proto.grpc.metric_exporter import OTLPMetricExporter
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.sdk.resources import Resource
from prometheus_client import Counter, Histogram, Gauge, generate_latest
import requests
import logging
import os
import time
import random

# Configure OpenTelemetry
resource = Resource.create({
    "service.name": os.getenv("OTEL_SERVICE_NAME", "frontend"),
    "service.version": "1.0.0"
})

# Tracing
trace_provider = TracerProvider(resource=resource)
trace_processor = BatchSpanProcessor(
    OTLPSpanExporter(endpoint=os.getenv("OTEL_EXPORTER_OTLP_ENDPOINT"))
)
trace_provider.add_span_processor(trace_processor)
trace.set_tracer_provider(trace_provider)
tracer = trace.get_tracer(__name__)

# Metrics
metric_reader = PeriodicExportingMetricReader(
    OTLPMetricExporter(endpoint=os.getenv("OTEL_EXPORTER_OTLP_ENDPOINT"))
)
meter_provider = MeterProvider(resource=resource, metric_readers=[metric_reader])
metrics.set_meter_provider(meter_provider)
meter = metrics.get_meter(__name__)

# Prometheus metrics
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)
REQUEST_DURATION = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)
ACTIVE_REQUESTS = Gauge(
    'http_requests_inprogress',
    'Active HTTP requests',
    ['method', 'endpoint']
)

# Business metrics
ORDERS_CREATED = Counter('orders_created_total', 'Total orders created')
REVENUE = Counter('revenue_total_cents', 'Total revenue in cents')

app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()

API_GATEWAY_URL = os.getenv("API_GATEWAY_URL", "http://api-gateway:8081")

# Structured logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - [trace_id=%(otelTraceID)s] - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

# HTML Template
HTML_TEMPLATE = """
<!DOCTYPE html>
<html>
<head>
    <title>E-Commerce Demo</title>
    <style>
        body { font-family: Arial; max-width: 1200px; margin: 50px auto; padding: 20px; }
        .card { border: 1px solid #ddd; padding: 20px; margin: 10px 0; border-radius: 5px; }
        button { background: #007bff; color: white; padding: 10px 20px; border: none; border-radius: 5px; cursor: pointer; }
        button:hover { background: #0056b3; }
        .metrics { background: #f8f9fa; padding: 15px; margin-top: 20px; }
    </style>
</head>
<body>
    <h1>🛒 E-Commerce Monitoring Demo</h1>
    
    <div class="card">
        <h2>Create Order</h2>
        <button onclick="createOrder()">Place Order</button>
        <button onclick="createSlowOrder()">Place Slow Order (Simulate Latency)</button>
        <button onclick="createFailingOrder()">Place Failing Order (Simulate Error)</button>
        <div id="result"></div>
    </div>
    
    <div class="card">
        <h2>Check Inventory</h2>
        <button onclick="checkInventory()">Check Stock</button>
        <div id="inventory"></div>
    </div>
    
    <div class="metrics">
        <h3>📊 Monitoring Links</h3>
        <ul>
            <li><a href="http://localhost:3000" target="_blank">Grafana Dashboards</a></li>
            <li><a href="http://localhost:9090" target="_blank">Prometheus</a></li>
            <li><a href="http://localhost:3200" target="_blank">Tempo (Traces)</a></li>
            <li><a href="http://localhost:9093" target="_blank">Alertmanager</a></li>
        </ul>
    </div>
    
    <script>
        async function createOrder() {
            const result = await fetch('/api/order', { method: 'POST' });
            const data = await result.json();
            document.getElementById('result').innerHTML = `<pre>${JSON.stringify(data, null, 2)}</pre>`;
        }
        
        async function createSlowOrder() {
            const result = await fetch('/api/order?slow=true', { method: 'POST' });
            const data = await result.json();
            document.getElementById('result').innerHTML = `<pre>${JSON.stringify(data, null, 2)}</pre>`;
        }
        
        async function createFailingOrder() {
            try {
                const result = await fetch('/api/order?fail=true', { method: 'POST' });
                const data = await result.json();
                document.getElementById('result').innerHTML = `<pre>${JSON.stringify(data, null, 2)}</pre>`;
            } catch (e) {
                document.getElementById('result').innerHTML = `<pre style="color:red">Error: ${e}</pre>`;
            }
        }
        
        async function checkInventory() {
            const result = await fetch('/api/inventory');
            const data = await result.json();
            document.getElementById('inventory').innerHTML = `<pre>${JSON.stringify(data, null, 2)}</pre>`;
        }
    </script>
</body>
</html>
"""

@app.route('/')
def index():
    """Homepage"""
    return render_template_string(HTML_TEMPLATE)

@app.route('/api/order', methods=['POST'])
def create_order():
    """Create order - calls API Gateway"""
    method = request.method
    endpoint = '/api/order'
    
    ACTIVE_REQUESTS.labels(method=method, endpoint=endpoint).inc()
    start_time = time.time()
    
    try:
        with tracer.start_as_current_span("create_order") as span:
            # Add span attributes
            span.set_attribute("http.method", method)
            span.set_attribute("http.route", endpoint)
            
            # Check for test parameters
            slow = request.args.get('slow') == 'true'
            fail = request.args.get('fail') == 'true'
            
            if slow:
                span.add_event("Simulating slow request")
            if fail:
                span.add_event("Simulating failing request")
            
            # Call API Gateway
            params = {}
            if slow:
                params['slow'] = 'true'
            if fail:
                params['fail'] = 'true'
            
            response = requests.post(
                f"{API_GATEWAY_URL}/order",
                json={"product": "Widget", "quantity": random.randint(1, 5)},
                params=params,
                timeout=10
            )
            
            # Business metrics
            if response.status_code == 200:
                data = response.json()
                ORDERS_CREATED.inc()
                REVENUE.inc(data.get('amount', 0))
                span.set_attribute("order.id", data.get('order_id'))
                span.set_attribute("order.amount", data.get('amount'))
            
            REQUEST_COUNT.labels(
                method=method,
                endpoint=endpoint,
                status=response.status_code
            ).inc()
            
            return jsonify(response.json()), response.status_code
            
    except Exception as e:
        span = trace.get_current_span()
        span.record_exception(e)
        span.set_status(trace.Status(trace.StatusCode.ERROR, str(e)))
        
        REQUEST_COUNT.labels(
            method=method,
            endpoint=endpoint,
            status=500
        ).inc()
        
        logger.error(f"Error creating order: {e}")
        return jsonify({"error": str(e)}), 500
        
    finally:
        duration = time.time() - start_time
        REQUEST_DURATION.labels(method=method, endpoint=endpoint).observe(duration)
        ACTIVE_REQUESTS.labels(method=method, endpoint=endpoint).dec()

@app.route('/api/inventory')
def get_inventory():
    """Get inventory"""
    with tracer.start_as_current_span("get_inventory"):
        response = requests.get(f"{API_GATEWAY_URL}/inventory")
        return jsonify(response.json()), response.status_code

@app.route('/health')
def health():
    """Health check"""
    return jsonify({"status": "healthy"}), 200

@app.route('/metrics')
def metrics():
    """Prometheus metrics endpoint"""
    return generate_latest()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080, debug=False)
```

`applications/frontend/Dockerfile`:

dockerfile

````dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

`applications/frontend/requirements.txt`:
```
flask==3.0.0
requests==2.31.0
opentelemetry-api==1.21.0
opentelemetry-sdk==1.21.0
opentelemetry-instrumentation-flask==0.42b0
opentelemetry-instrumentation-requests==0.42b0
opentelemetry-exporter-otlp==1.21.0
prometheus-client==0.19.0
````

### 2.2 API Gateway (Node.js)

`applications/api-gateway/app.js`:

javascript

```javascript
const express = require('express');
const axios = require('axios');
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-grpc');
const { Resource } = require('@opentelemetry/resources');
const { SemanticResourceAttributes } = require('@opentelemetry/semantic-conventions');
const { PrometheusExporter } = require('@opentelemetry/exporter-prometheus');
const client = require('prom-client');

// OpenTelemetry SDK
const sdk = new NodeSDK({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: process.env.OTEL_SERVICE_NAME || 'api-gateway',
    [SemanticResourceAttributes.SERVICE_VERSION]: '1.0.0',
  }),
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT || 'http://localhost:4317',
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();

// Prometheus metrics
const register = new client.Registry();
client.collectDefaultMetrics({ register });

const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status'],
  registers: [register]
});

const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status'],
  registers: [register]
});

const app = express();
app.use(express.json());

const PAYMENT_SERVICE_URL = process.env.PAYMENT_SERVICE_URL || 'http://payment-service:8082';
const INVENTORY_SERVICE_URL = process.env.INVENTORY_SERVICE_URL || 'http://inventory-service:8083';

// Middleware for metrics
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration.labels(req.method, req.route?.path || req.path, res.statusCode).observe(duration);
    httpRequestsTotal.labels(req.method, req.route?.path || req.path, res.statusCode).inc();
  });
  
  next();
});

// Create Order
app.post('/order', async (req, res) => {
  try {
    const { product, quantity } = req.body;
    const slow = req.query.slow === 'true';
    const fail = req.query.fail === 'true';
    
    console.log(`Creating order: product=${product}, quantity=${quantity}, slow=${slow}, fail=${fail}`);
    
    // Check inventory
    const inventoryResponse = await axios.get(`${INVENTORY_SERVICE_URL}/check`, {
      params: { product, quantity }
    });
    
    if (!inventoryResponse.data.available) {
      return res.status(400).json({ error: 'Product not available' });
    }
    
    // Process payment
    const amount = quantity * 1000; // $10 per item in cents
    
    const paymentResponse = await axios.post(`${PAYMENT_SERVICE_URL}/process`, {
      amount,
      slow,
      fail
    });
    
    if (paymentResponse.data.status !== 'success') {
      return res.status(400).json({ error: 'Payment failed' });
    }
    
    // Update inventory
    await axios.post(`${INVENTORY_SERVICE_URL}/reserve`, {
      product,
      quantity
    });
    
    const order = {
      order_id: `ORD-${Date.now()}`,
      product,
      quantity,
      amount,
      payment_id: paymentResponse.data.payment_id,
      status: 'completed'
    };
    
    console.log('Order created:', order);
    res.json(order);
    
  } catch (error) {
    console.error('Error creating order:', error.message);
    res.status(500).json({ error: error.message });
  }
});

// Get Inventory
app.get('/inventory', async (req, res) => {try { const response = await axios.get(`${INVENTORY_SERVICE_URL}/list`); res.json(response.data); } 
catch (error) { console.error('Error fetching inventory:', error.message); res.status(500).json({ error: error.message }); } });

// Health check app.get('/health', (req, res) => { res.json({ status: 'healthy' }); });

// Metrics endpoint app.get('/metrics', async (req, res) => { res.set('Content-Type', register.contentType); res.end(await register.metrics()); });

const PORT = 8081; app.listen(PORT, () => { console.log(`API Gateway listening on port ${PORT}`); });

````

`applications/api-gateway/package.json`:
```json
{
  "name": "api-gateway",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.2",
    "axios": "^1.6.2",
    "@opentelemetry/sdk-node": "^0.45.1",
    "@opentelemetry/auto-instrumentations-node": "^0.40.1",
    "@opentelemetry/exporter-trace-otlp-grpc": "^0.45.1",
    "@opentelemetry/resources": "^1.19.0",
    "@opentelemetry/semantic-conventions": "^1.19.0",
    "@opentelemetry/exporter-prometheus": "^0.45.1",
    "prom-client": "^15.0.0"
  }
}
```

`applications/api-gateway/Dockerfile`:
```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

CMD ["node", "app.js"]
```

Создаем Payment и Inventory services, и переходим к настройке monitoring stack

### 2.3 Payment Service (Go)

`applications/payment-service/main.go`:

go

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"log"
	"math/rand"
	"net/http"
	"os"
	"time"

	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
	"go.opentelemetry.io/otel"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc"
	"go.opentelemetry.io/otel/sdk/resource"
	sdktrace "go.opentelemetry.io/otel/sdk/trace"
	semconv "go.opentelemetry.io/otel/semconv/v1.17.0"
	"go.opentelemetry.io/otel/trace"
)

var (
	tracer trace.Tracer
	
	// Prometheus metrics
	requestDuration = prometheus.NewHistogramVec(
		prometheus.HistogramOpts{
			Name:    "http_request_duration_seconds",
			Help:    "HTTP request duration in seconds",
			Buckets: []float64{.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5, 10},
		},
		[]string{"method", "endpoint", "status"},
	)
	
	requestsTotal = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "http_requests_total",
			Help: "Total number of HTTP requests",
		},
		[]string{"method", "endpoint", "status"},
	)
	
	paymentsProcessed = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "payments_processed_total",
			Help: "Total number of payments processed",
		},
		[]string{"status"},
	)
	
	paymentAmount = prometheus.NewHistogram(
		prometheus.HistogramOpts{
			Name:    "payment_amount_cents",
			Help:    "Payment amount in cents",
			Buckets: prometheus.ExponentialBuckets(100, 2, 10),
		},
	)
)

func init() {
	prometheus.MustRegister(requestDuration)
	prometheus.MustRegister(requestsTotal)
	prometheus.MustRegister(paymentsProcessed)
	prometheus.MustRegister(paymentAmount)
}

type PaymentRequest struct {
	Amount int  `json:"amount"`
	Slow   bool `json:"slow"`
	Fail   bool `json:"fail"`
}

type PaymentResponse struct {
	PaymentID string `json:"payment_id"`
	Status    string `json:"status"`
	Amount    int    `json:"amount"`
}

func initTracer() func(context.Context) error {
	ctx := context.Background()

	exporter, err := otlptracegrpc.New(ctx,
		otlptracegrpc.WithEndpoint(getEnv("OTEL_EXPORTER_OTLP_ENDPOINT", "localhost:4317")),
		otlptracegrpc.WithInsecure(),
	)
	if err != nil {
		log.Fatalf("Failed to create exporter: %v", err)
	}

	res, err := resource.New(ctx,
		resource.WithAttributes(
			semconv.ServiceName(getEnv("OTEL_SERVICE_NAME", "payment-service")),
			semconv.ServiceVersion("1.0.0"),
		),
	)
	if err != nil {
		log.Fatalf("Failed to create resource: %v", err)
	}

	tp := sdktrace.NewTracerProvider(
		sdktrace.WithBatcher(exporter),
		sdktrace.WithResource(res),
	)

	otel.SetTracerProvider(tp)
	tracer = tp.Tracer("payment-service")

	return tp.Shutdown
}

func processPaymentHandler(w http.ResponseWriter, r *http.Request) {
	start := time.Now()
	ctx := r.Context()
	
	ctx, span := tracer.Start(ctx, "process_payment")
	defer span.End()

	var req PaymentRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		span.RecordError(err)
		http.Error(w, err.Error(), http.StatusBadRequest)
		recordMetrics(r.Method, "/process", "400", start)
		return
	}

	span.SetAttributes(
		attribute.Int("payment.amount", req.Amount),
		attribute.Bool("payment.slow", req.Slow),
		attribute.Bool("payment.fail", req.Fail),
	)

	log.Printf("Processing payment: amount=%d, slow=%v, fail=%v", req.Amount, req.Slow, req.Fail)

	// Simulate payment gateway call
	ctx, gatewaySpan := tracer.Start(ctx, "call_payment_gateway")
	
	if req.Slow {
		gatewaySpan.AddEvent("Simulating slow payment gateway")
		time.Sleep(time.Duration(2000+rand.Intn(3000)) * time.Millisecond)
	} else {
		time.Sleep(time.Duration(50+rand.Intn(150)) * time.Millisecond)
	}

	// Simulate random failures
	if req.Fail || rand.Float64() < 0.05 {
		gatewaySpan.AddEvent("Payment gateway failed")
		gatewaySpan.RecordError(fmt.Errorf("payment gateway error"))
		gatewaySpan.End()
		
		paymentsProcessed.WithLabelValues("failed").Inc()
		
		response := PaymentResponse{
			PaymentID: fmt.Sprintf("PAY-FAIL-%d", time.Now().Unix()),
			Status:    "failed",
			Amount:    req.Amount,
		}
		
		w.WriteHeader(http.StatusInternalServerError)
		json.NewEncoder(w).Encode(response)
		recordMetrics(r.Method, "/process", "500", start)
		return
	}
	
	gatewaySpan.AddEvent("Payment successful")
	gatewaySpan.End()

	// Record business metrics
	paymentsProcessed.WithLabelValues("success").Inc()
	paymentAmount.Observe(float64(req.Amount))

	response := PaymentResponse{
		PaymentID: fmt.Sprintf("PAY-%d", time.Now().Unix()),
		Status:    "success",
		Amount:    req.Amount,
	}

	span.SetAttributes(attribute.String("payment.id", response.PaymentID))
	span.AddEvent("Payment processed successfully")

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
	recordMetrics(r.Method, "/process", "200", start)
}

func healthHandler(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(map[string]string{"status": "healthy"})
}

func recordMetrics(method, endpoint, status string, start time.Time) {
	duration := time.Since(start).Seconds()
	requestDuration.WithLabelValues(method, endpoint, status).Observe(duration)
	requestsTotal.WithLabelValues(method, endpoint, status).Inc()
}

func getEnv(key, fallback string) string {
	if value := os.Getenv(key); value != "" {
		return value
	}
	return fallback
}

func main() {
	shutdown := initTracer()
	defer shutdown(context.Background())

	http.HandleFunc("/process", processPaymentHandler)
	http.HandleFunc("/health", healthHandler)
	http.Handle("/metrics", promhttp.Handler())

	log.Println("Payment Service listening on :8082")
	log.Fatal(http.ListenAndServe(":8082", nil))
}
```

`applications/payment-service/go.mod`:

go

```go
module payment-service

go 1.21

require (
	github.com/prometheus/client_golang v1.17.0
	go.opentelemetry.io/otel v1.21.0
	go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc v1.21.0
	go.opentelemetry.io/otel/sdk v1.21.0
)
```

`applications/payment-service/Dockerfile`:

dockerfile

```dockerfile
FROM golang:1.21-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o payment-service .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/payment-service .

EXPOSE 8082
CMD ["./payment-service"]
```

### 2.4 Inventory Service (Python)

`applications/inventory-service/app.py`:

python

````python
from flask import Flask, jsonify, request
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.sdk.resources import Resource
from prometheus_client import Counter, Gauge, Histogram, generate_latest
import psycopg2
import redis
import os
import time
import logging
import json

# OpenTelemetry setup
resource = Resource.create({
    "service.name": os.getenv("OTEL_SERVICE_NAME", "inventory-service"),
    "service.version": "1.0.0"
})

trace_provider = TracerProvider(resource=resource)
trace_processor = BatchSpanProcessor(
    OTLPSpanExporter(endpoint=os.getenv("OTEL_EXPORTER_OTLP_ENDPOINT"))
)
trace_provider.add_span_processor(trace_processor)
trace.set_tracer_provider(trace_provider)
tracer = trace.get_tracer(__name__)

# Prometheus metrics
REQUEST_COUNT = Counter('http_requests_total', 'Total requests', ['method', 'endpoint', 'status'])
REQUEST_DURATION = Histogram('http_request_duration_seconds', 'Request duration', ['method', 'endpoint'])
INVENTORY_LEVEL = Gauge('inventory_level', 'Current inventory level', ['product'])
RESERVATIONS = Counter('inventory_reservations_total', 'Total reservations', ['product'])

app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)

DATABASE_URL = os.getenv("DATABASE_URL")
REDIS_URL = os.getenv("REDIS_URL")

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Mock inventory data
INVENTORY = {
    "Widget": {"stock": 1000, "price": 1000},
    "Gadget": {"stock": 500, "price": 2000},
    "Doohickey": {"stock": 750, "price": 1500}
}

def get_redis():
    """Get Redis connection"""
    return redis.from_url(REDIS_URL)

def get_db():
    """Get database connection"""
    return psycopg2.connect(DATABASE_URL)

@app.route('/list')
def list_inventory():
    """List all inventory"""
    start = time.time()
    
    with tracer.start_as_current_span("list_inventory") as span:
        try:
            # Check cache
            r = get_redis()
            cached = r.get("inventory:list")
            
            if cached:
                span.add_event("Cache hit")
                span.set_attribute("cache.hit", True)
                inventory = json.loads(cached)
            else:
                span.add_event("Cache miss")
                span.set_attribute("cache.hit", False)
                
                # Query database (simulated with mock data)
                inventory = INVENTORY
                
                # Update Prometheus gauges
                for product, data in inventory.items():
                    INVENTORY_LEVEL.labels(product=product).set(data['stock'])
                
                # Cache result
                r.setex("inventory:list", 60, json.dumps(inventory))
            
            REQUEST_COUNT.labels(method='GET', endpoint='/list', status='200').inc()
            REQUEST_DURATION.labels(method='GET', endpoint='/list').observe(time.time() - start)
            
            return jsonify(inventory), 200
            
        except Exception as e:
            span.record_exception(e)
            logger.error(f"Error listing inventory: {e}")
            REQUEST_COUNT.labels(method='GET', endpoint='/list', status='500').inc()
            return jsonify({"error": str(e)}), 500

@app.route('/check')
def check_availability():
    """Check product availability"""
    start = time.time()
    
    with tracer.start_as_current_span("check_availability") as span:
        product = request.args.get('product')
        quantity = int(request.args.get('quantity', 1))
        
        span.set_attribute("product", product)
        span.set_attribute("quantity", quantity)
        
        try:
            if product not in INVENTORY:
                span.set_attribute("available", False)
                return jsonify({"available": False, "reason": "Product not found"}), 200
            
            available = INVENTORY[product]['stock'] >= quantity
            span.set_attribute("available", available)
            span.set_attribute("current_stock", INVENTORY[product]['stock'])
            
            REQUEST_COUNT.labels(method='GET', endpoint='/check', status='200').inc()
            REQUEST_DURATION.labels(method='GET', endpoint='/check').observe(time.time() - start)
            
            return jsonify({
                "available": available,
                "product": product,
                "requested": quantity,
                "in_stock": INVENTORY[product]['stock']
            }), 200
            
        except Exception as e:
            span.record_exception(e)
            logger.error(f"Error checking availability: {e}")
            REQUEST_COUNT.labels(method='GET', endpoint='/check', status='500').inc()
            return jsonify({"error": str(e)}), 500

@app.route('/reserve', methods=['POST'])
def reserve_inventory():
    """Reserve inventory"""
    start = time.time()
    
    with tracer.start_as_current_span("reserve_inventory") as span:
        data = request.json
        product = data.get('product')
        quantity = data.get('quantity')
        
        span.set_attribute("product", product)
        span.set_attribute("quantity", quantity)
        
        try:
            if product not in INVENTORY:
                return jsonify({"error": "Product not found"}), 404
            
            if INVENTORY[product]['stock'] < quantity:
                span.add_event("Insufficient stock")
                return jsonify({"error": "Insufficient stock"}), 400
            
            # Reserve inventory
            INVENTORY[product]['stock'] -= quantity
            INVENTORY_LEVEL.labels(product=product).set(INVENTORY[product]['stock'])
            RESERVATIONS.labels(product=product).inc()
            
            span.add_event("Inventory reserved")
            logger.info(f"Reserved {quantity} units of {product}")
            
            # Invalidate cache
            r = get_redis()
            r.delete("inventory:list")
            
            REQUEST_COUNT.labels(method='POST', endpoint='/reserve', status='200').inc()
            REQUEST_DURATION.labels(method='POST', endpoint='/reserve').observe(time.time() - start)
            
            return jsonify({
                "reserved": quantity,
                "product": product,
                "remaining_stock": INVENTORY[product]['stock']
            }), 200
            
        except Exception as e:
            span.record_exception(e)
            logger.error(f"Error reserving inventory: {e}")
            REQUEST_COUNT.labels(method='POST', endpoint='/reserve', status='500').inc()
            return jsonify({"error": str(e)}), 500

@app.route('/health')
def health():
    """Health check"""
    return jsonify({"status": "healthy"}), 200

@app.route('/metrics')
def metrics():
    """Prometheus metrics"""
    return generate_latest()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8083, debug=False)
```

`applications/inventory-service/requirements.txt`:
```
flask==3.0.0
psycopg2-binary==2.9.9
redis==5.0.1
opentelemetry-api==1.21.0
opentelemetry-sdk==1.21.0
opentelemetry-instrumentation-flask==0.42b0
opentelemetry-exporter-otlp==1.21.0
prometheus-client==0.19.0
````

`applications/inventory-service/Dockerfile`:

dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

---

## Часть 3: Monitoring Configuration (20 минут)

### 3.1 Prometheus Configuration

`monitoring/prometheus/prometheus.yml`:

yaml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'ecommerce-demo'
    environment: 'production'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

# Load rules
rule_files:
  - '/etc/prometheus/alerts/*.yml'
  - '/etc/prometheus/recording-rules/*.yml'

scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Application services
  - job_name: 'frontend'
    static_configs:
      - targets: ['frontend:8080']
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
      - target_label: service
        replacement: 'frontend'

  - job_name: 'api-gateway'
    static_configs:
      - targets: ['api-gateway:8081']
    relabel_configs:
      - target_label: service
        replacement: 'api-gateway'

  - job_name: 'payment-service'
    static_configs:
      - targets: ['payment-service:8082']
    relabel_configs:
      - target_label: service
        replacement: 'payment-service'

  - job_name: 'inventory-service'
    static_configs:
      - targets: ['inventory-service:8083']
    relabel_configs:
      - target_label: service
        replacement: 'inventory-service'

  # Infrastructure
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'postgres-exporter'
    static_configs:
      - targets: ['postgres-exporter:9187']

  - job_name: 'redis-exporter'
    static_configs:
      - targets: ['redis-exporter:9121']

  # OpenTelemetry Collector
  - job_name: 'otel-collector'
    static_configs:
      - targets: ['otel-collector:8888']
```

### 3.2 Alert Rules

`monitoring/prometheus/alerts/application.yml`:

yaml

```yaml
groups:
  - name: application_alerts
    interval: 30s
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
            /
            sum(rate(http_requests_total[5m])) by (service)
          ) > 0.05
        for: 2m
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "High error rate on {{ $labels.service }}"
          description: "Error rate is {{ $value | humanizePercentage }} on {{ $labels.service }}"
          dashboard: "http://localhost:3000/d/app-overview"
          runbook: "https://runbooks.example.com/high-error-rate"

      # High latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (service, le)
          ) > 1
        for: 5m
        labels:
          severity: warning
          team: backend
        annotations:
          summary: "High latency on {{ $labels.service }}"
          description: "P95 latency is {{ $value }}s on {{ $labels.service }}"

      # Low throughput
      - alert: LowThroughput
        expr: |
          sum(rate(http_requests_total[5m])) by (service) < 1
        for: 10m
        labels:
          severity: warning
          team: backend
        annotations:
          summary: "Low throughput on {{ $labels.service }}"
          description: "Request rate is {{ $value }} req/s"

      # Payment failures
      - alert: PaymentFailureSpike
        expr: |
          (
            sum(rate(payments_processed_total{status="failed"}[5m]))
            /
            sum(rate(payments_processed_total[5m]))
          ) > 0.1
        for: 3m
        labels:
          severity: critical
          team: payments
        annotations:
          summary: "High payment failure rate"
          description: "Payment failure rate is {{ $value | humanizePercentage }}"

      # Inventory low
      - alert: InventoryLow
        expr: inventory_level < 100
        for: 5m
        labels:
          severity: warning
          team: inventory
        annotations:
          summary: "Low inventory for {{ $labels.product }}"
          description: "Only {{ $value }} units remaining"
```

`monitoring/prometheus/alerts/infrastructure.yml`:

yaml

```yaml
groups:
  - name: infrastructure_alerts
    interval: 30s
    rules:
      # Service down
      - alert: ServiceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
          team: sre
        annotations:
          summary: "Service {{ $labels.job }} is down"
          description: "{{ $labels.instance }} has been down for more than 1 minute"

      # High CPU
      - alert: HighCPU
        expr: |
          100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
          team: infrastructure
        annotations:
          summary: "High CPU on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}%"

      # High memory
      - alert: HighMemory
        expr: |
          (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
        for: 5m
        labels:
          severity: warning
          team: infrastructure
        annotations:
          summary: "High memory on {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}%"

      # Database connection pool
      - alert: DatabaseConnectionPoolHigh
        expr: |
          pg_stat_activity_count / pg_settings_max_connections > 0.8
        for: 5m
        labels:
          severity: warning
          team: database
        annotations:
          summary: "Database connection pool near limit"
          description: "Using {{ $value | humanizePercentage }} of connections"

      # Redis memory
      - alert: RedisMemoryHigh
        expr: |
          redis_memory_used_bytes / redis_memory_max_bytes > 0.9
        for: 5m
        labels:
          severity: warning
          team: cache
        annotations:
          summary: "Redis memory usage high"
          description: "Redis using {{ $value | humanizePercentage }} of max memory"
```

### 3.3 SLO Configuration

`monitoring/prometheus/alerts/slo.yml`:

yaml

```yaml
groups:
  - name: slo_alerts
    interval: 30s
    rules:
      # Fast burn rate (2h to exhaustion)
      - alert: ErrorBudgetBurnRateFast
        expr: |
          (
            (1 - (sum(rate(http_requests_total{service="api-gateway",status!~"5.."}[5m])) / sum(rate(http_requests_total{service="api-gateway"}[5m]))))
            /
            (1 - 0.999)
          ) > 14.4
        for: 2m
        labels:
          severity: critical
          slo: api_availability
          burn_rate: fast
        annotations:
          summary: "Fast error budget burn on API Gateway"
          description: "Error budget will be exhausted in 2 hours at current rate"

      # Slow burn rate (5d to exhaustion)
      - alert: ErrorBudgetBurnRateSlow
        expr: |
          (
            (1 - (sum(rate(http_requests_total{service="api-gateway",status!~"5.."}[1h])) / sum(rate(http_requests_total{service="api-gateway"}[1h]))))
            /
            (1 - 0.999)
          ) > 6
        for: 15m
        labels:
          severity: warning
          slo: api_availability
          burn_rate: slow
        annotations:
          summary: "Slow error budget burn on API Gateway"
          description: "Error budget will be exhausted in 5 days at current rate"

      # SLO violation
      - alert: SLOViolation
        expr: |
          (
            sum(rate(http_requests_total{service="api-gateway",status!~"5.."}[30d]))
            /
            sum(rate(http_requests_total{service="api-gateway"}[30d]))
          ) < 0.999
        for: 5m
        labels:
          severity: critical
          slo: api_availability
        annotations:
          summary: "API Gateway SLO violated"
          description: "30-day availability: {{ $value | humanizePercentage }}"
```

`monitoring/prometheus/recording-rules/slo-rules.yml`:

yaml

```yaml
groups:
  - name: slo_recording_rules
    interval: 30s
    rules:
      # API Gateway availability
      - record: slo:api_gateway:availability:ratio
        expr: |
          sum(rate(http_requests_total{service="api-gateway",status!~"5.."}[5m]))
          /
          sum(rate(http_requests_total{service="api-gateway"}[5m]))

      # API Gateway error budget remaining
      - record: slo:api_gateway:error_budget:ratio
        expr: |
          1 - (
            (1 - slo:api_gateway:availability:ratio)
            /
            (1 - 0.999)
          )

      # Payment service availability
      - record: slo:payment:availability:ratio
        expr: |
          sum(rate(payments_processed_total{status="success"}[5m]))
          /
          sum(rate(payments_processed_total[5m]))
```

---

## Часть 4: Grafana Dashboards (15 минут)

### 4.1 Datasources Configuration

`monitoring/grafana/datasources/datasources.yml`:

yaml

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
    jsonData:
      httpMethod: POST
      timeInterval: 30s
      exemplarTraceIdDestinations:
        - name: trace_id
          datasourceUid: tempo

  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    editable: true
    jsonData:
      maxLines: 1000
      derivedFields:
        - datasourceUid: tempo
          matcherRegex: "trace_id=(\\w+)"
          name: TraceID
          url: "$${__value.raw}"

  - name: Tempo
    type: tempo
    access: proxy
    url: http://tempo:3200
    editable: true
    jsonData:
      tracesToLogs:
        datasourceUid: 'loki'
        tags: ['service', 'trace_id']
      serviceMap:
        datasourceUid: 'prometheus'
```

### 4.2 Dashboard Provisioning

`monitoring/grafana/provisioning/dashboards.yml`:

yaml

```yaml
apiVersion: 1

providers:
  - name: 'Default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /etc/grafana/provisioning/dashboards
```

### 4.3 Main Overview Dashboard

`monitoring/grafana/dashboards/overview.json`:

json

```json
{
  "dashboard": {
    "title": "E-Commerce Overview",
    "tags": ["overview", "production"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Request Rate",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 0},
        "targets": [
          {
            "datasource": "Prometheus",
            "expr": "sum(rate(http_requests_total[5m])) by (service)",
            "legendFormat": "{{service}}"
          }
        ]
      },
      {
        "id": 2,
        "title": "Error Rate",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 0},
        "targets": [
          {
            "datasource": "Prometheus",
            "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) by (service) / sum(rate(http_requests_total[5m])) by (service)",
            "legendFormat": "{{service}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percentunit"
          }
        }
      },
      {
        "id": 3,
        "title": "P95 Latency",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 8},
        "targets": [
          {
            "datasource": "Prometheus",
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (service, le))",
            "legendFormat": "{{service}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "s"
          }
        }
      },
      {
        "id": 4,
        "title": "Active Services",
        "type": "stat",
        "gridPos": {"h": 4, "w": 6, "x": 12, "y": 8},
        "targets": [
          {
            "datasource": "Prometheus", "expr": "count(up == 1)" } ] }, { "id": 5, "title": "Total Orders", "type": "stat", "gridPos": {"h": 4, "w": 6, "x": 18, "y": 8}, "targets": [ { "datasource": "Prometheus", "expr": "sum(orders_created_total)" } ] }, { "id": 6, "title": "Revenue (last hour)", "type": "stat", "gridPos": {"h": 4, "w": 6, "x": 12, "y": 12}, "targets": [ { "datasource": "Prometheus", "expr": "sum(increase(revenue_total_cents[1h])) / 100" } ], "fieldConfig": { "defaults": { "unit": "currencyUSD" } } }, { "id": 7, "title": "Payment Success Rate", "type": "gauge", "gridPos": {"h": 4, "w": 6, "x": 18, "y": 12}, "targets": [ { "datasource": "Prometheus", "expr": "sum(rate(payments_processed_total{status="success"}[5m])) / sum(rate(payments_processed_total[5m]))" } ], "fieldConfig": { "defaults": { "unit": "percentunit", "min": 0, "max": 1, "thresholds": { "steps": [ {"value": 0, "color": "red"}, {"value": 0.95, "color": "yellow"}, {"value": 0.99, "color": "green"} ] } } } }, { "id": 8, "title": "Service Map", "type": "nodeGraph", "gridPos": {"h": 12, "w": 24, "x": 0, "y": 16}, "targets": [ { "datasource": "Tempo", "queryType": "serviceMap" } ] } ] } }
            
            
```

### 4.4 SLO Dashboard

`monitoring/grafana/dashboards/slo-dashboard.json`:

json

```json
{
  "dashboard": {
    "title": "SLO Dashboard",
    "tags": ["slo", "sre"],
    "timezone": "browser",
    "refresh": "30s",
    "panels": [
      {
        "id": 1,
        "title": "API Gateway SLO Status",
        "type": "gauge",
        "gridPos": {"h": 8, "w": 8, "x": 0, "y": 0},
        "targets": [
          {
            "datasource": "Prometheus",
            "expr": "slo:api_gateway:availability:ratio * 100",
            "legendFormat": "Availability"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "min": 99,
            "max": 100,
            "thresholds": {
              "steps": [
                {"value": 99, "color": "red"},
                {"value": 99.9, "color": "yellow"},
                {"value": 99.95, "color": "green"}
              ]
            }
          }
        },
        "options": {
          "showThresholdLabels": true,
          "showThresholdMarkers": true
        }
      },
      {
        "id": 2,
        "title": "Error Budget Remaining",
        "type": "stat",
        "gridPos": {"h": 8, "w": 8, "x": 8, "y": 0},
        "targets": [
          {
            "datasource": "Prometheus",
            "expr": "slo:api_gateway:error_budget:ratio * 100"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "thresholds": {
              "steps": [
                {"value": 0, "color": "red"},
                {"value": 25, "color": "orange"},
                {"value": 50, "color": "yellow"},
                {"value": 75, "color": "green"}
              ]
            }
          }
        }
      },
      {
        "id": 3,
        "title": "Burn Rate (1h window)",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 8, "x": 16, "y": 0},
        "targets": [
          {
            "datasource": "Prometheus",
            "expr": "(1 - (sum(rate(http_requests_total{service=\"api-gateway\",status!~\"5..\"}[1h])) / sum(rate(http_requests_total{service=\"api-gateway\"}[1h])))) / (1 - 0.999)",
            "legendFormat": "Burn Rate"
          },
          {
            "datasource": "Prometheus",
            "expr": "14.4",
            "legendFormat": "Critical (2h)"
          },
          {
            "datasource": "Prometheus",
            "expr": "6",
            "legendFormat": "Warning (5d)"
          }
        ]
      },
      {
        "id": 4,
        "title": "30-Day Availability Trend",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 8},
        "targets": [
          {
            "datasource": "Prometheus",
            "expr": "(sum(rate(http_requests_total{service=\"api-gateway\",status!~\"5..\"}[30d])) / sum(rate(http_requests_total{service=\"api-gateway\"}[30d]))) * 100"
          },
          {
            "datasource": "Prometheus",
            "expr": "99.9",
            "legendFormat": "SLO Target"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "min": 99,
            "max": 100
          }
        }
      },
      {
        "id": 5,
        "title": "Error Budget Consumption by Day",
        "type": "bargauge",
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 8},
        "targets": [
          {
            "datasource": "Prometheus",
            "expr": "(1 - slo:api_gateway:error_budget:ratio) * 100"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "max": 100
          }
        },
        "options": {
          "orientation": "horizontal",
          "displayMode": "gradient"
        }
      },
      {
        "id": 6,
        "title": "SLI Components",
        "type": "table",
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 16},
        "targets": [
          {
            "datasource": "Prometheus",
            "expr": "slo:api_gateway:availability:ratio",
            "format": "table",
            "instant": true
          }
        ]
      }
    ]
  }
}
```

### 4.5 Business Metrics Dashboard

`monitoring/grafana/dashboards/business-metrics.json`:

json

```json
{
  "dashboard": {
    "title": "Business Metrics",
    "tags": ["business", "kpi"],
    "panels": [
      {
        "id": 1,
        "title": "Orders per Minute",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 0},
        "targets": [
          {
            "expr": "sum(rate(orders_created_total[5m])) * 60"
          }
        ]
      },
      {
        "id": 2,
        "title": "Revenue per Hour",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 0},
        "targets": [
          {
            "expr": "sum(rate(revenue_total_cents[5m])) * 3600 / 100"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "currencyUSD"
          }
        }
      },
      {
        "id": 3,
        "title": "Payment Success Rate",
        "type": "stat",
        "gridPos": {"h": 6, "w": 6, "x": 0, "y": 8},
        "targets": [
          {
            "expr": "sum(rate(payments_processed_total{status=\"success\"}[5m])) / sum(rate(payments_processed_total[5m]))"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percentunit"
          }
        }
      },
      {
        "id": 4,
        "title": "Average Order Value",
        "type": "stat",
        "gridPos": {"h": 6, "w": 6, "x": 6, "y": 8},
        "targets": [
          {
            "expr": "sum(rate(revenue_total_cents[5m])) / sum(rate(orders_created_total[5m])) / 100"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "currencyUSD"
          }
        }
      },
      {
        "id": 5,
        "title": "Inventory by Product",
        "type": "bargauge",
        "gridPos": {"h": 6, "w": 12, "x": 12, "y": 8},
        "targets": [
          {
            "expr": "inventory_level"
          }
        ],
        "options": {
          "orientation": "horizontal"
        }
      },
      {
        "id": 6,
        "title": "Top Products by Reservations",
        "type": "piechart",
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 14},
        "targets": [
          {
            "expr": "sum(rate(inventory_reservations_total[1h])) by (product)"
          }
        ]
      },
      {
        "id": 7,
        "title": "Payment Amount Distribution",
        "type": "histogram",
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 14},
        "targets": [
          {
            "expr": "payment_amount_cents"
          }
        ]
      }
    ]
  }
}
```

---

## Часть 5: OpenTelemetry Collector Configuration (10 минут)

`monitoring/otel-collector/otel-config.yml`:

yaml

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 10s
    send_batch_size: 1024

  memory_limiter:
    check_interval: 1s
    limit_mib: 512

  # Tail sampling - save errors and slow requests
  tail_sampling:
    decision_wait: 10s
    num_traces: 100
    expected_new_traces_per_sec: 10
    policies:
      - name: error-traces
        type: status_code
        status_code:
          status_codes: [ERROR]
      
      - name: slow-traces
        type: latency
        latency:
          threshold_ms: 1000
      
      - name: probabilistic-sample
        type: probabilistic
        probabilistic:
          sampling_percentage: 10

  resource:
    attributes:
      - key: environment
        value: production
        action: insert

  # Add span metrics
  spanmetrics:
    metrics_exporter: prometheus
    latency_histogram_buckets: [100ms, 250ms, 500ms, 1s, 2s, 5s, 10s]
    dimensions:
      - name: http.method
      - name: http.status_code

exporters:
  # Tempo for traces
  otlp/tempo:
    endpoint: tempo:4317
    tls:
      insecure: true

  # Prometheus for metrics
  prometheus:
    endpoint: "0.0.0.0:8889"
    namespace: otel
    const_labels:
      environment: production

  # Loki for logs
  loki:
    endpoint: http://loki:3100/loki/api/v1/push
    labels:
      attributes:
        service.name: "service"
        severity: "severity"

  # Logging for debugging
  logging:
    loglevel: info
    sampling_initial: 5
    sampling_thereafter: 200

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, tail_sampling, batch, resource, spanmetrics]
      exporters: [otlp/tempo, logging]
    
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch, resource]
      exporters: [prometheus]
    
    logs:
      receivers: [otlp]
      processors: [memory_limiter, batch, resource]
      exporters: [loki, logging]
```

---

## Часть 6: Alertmanager Configuration (5 минут)

`monitoring/alertmanager/alertmanager.yml`:

yaml

```yaml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'

templates:
  - '/etc/alertmanager/templates/*.tmpl'

route:
  receiver: 'default'
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 5m
  repeat_interval: 4h
  
  routes:
    # Critical alerts
    - match:
        severity: critical
      receiver: critical-alerts
      group_wait: 10s
      repeat_interval: 1h
      continue: true

    # SLO alerts
    - match:
        slo: api_availability
      receiver: slo-alerts
      group_wait: 30s
      repeat_interval: 2h
    
    # Business alerts
    - match_re:
        alertname: (PaymentFailureSpike|InventoryLow)
      receiver: business-alerts

    # Infrastructure alerts
    - match:
        team: infrastructure
      receiver: infrastructure-alerts

# Inhibition rules
inhibit_rules:
  # If service is down, don't alert on high error rate
  - source_match:
      alertname: ServiceDown
    target_match:
      severity: warning
    equal: ['service']

  # If fast burn rate is firing, don't alert on slow burn
  - source_match:
      burn_rate: fast
    target_match:
      burn_rate: slow
    equal: ['slo']

receivers:
  - name: 'default'
    slack_configs:
      - channel: '#monitoring'
        title: 'Alert: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'critical-alerts'
    slack_configs:
      - channel: '#critical-alerts'
        title: '🚨 CRITICAL: {{ .GroupLabels.alertname }}'
        text: |
          {{ range .Alerts }}
          *Summary:* {{ .Annotations.summary }}
          *Description:* {{ .Annotations.description }}
          *Runbook:* {{ .Annotations.runbook }}
          {{ end }}
        send_resolved: true
        color: '{{ if eq .Status "firing" }}danger{{ else }}good{{ end }}'

  - name: 'slo-alerts'
    slack_configs:
      - channel: '#slo-alerts'
        title: '📊 SLO Alert: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'business-alerts'
    slack_configs:
      - channel: '#business-alerts'
        title: '💼 Business Alert: {{ .GroupLabels.alertname }}'

  - name: 'infrastructure-alerts'
    slack_configs:
      - channel: '#infrastructure'
```

---

## Часть 7: Scripts & Automation (10 минут)

### 7.1 Deployment Script

`scripts/deploy.sh`:

bash

```bash
#!/bin/bash
set -e

echo "🚀 Deploying E-Commerce Monitoring Stack"

# Colors
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m'

# Check prerequisites
check_prerequisites() {
    echo "Checking prerequisites..."
    
    if ! command -v docker &> /dev/null; then
        echo -e "${RED}Docker is not installed${NC}"
        exit 1
    fi
    
    if ! command -v docker-compose &> /dev/null; then
        echo -e "${RED}Docker Compose is not installed${NC}"
        exit 1
    fi
    
    echo -e "${GREEN}✓ Prerequisites check passed${NC}"
}

# Initialize database
init_database() {
    echo "Initializing database..."
    cat > init-db.sql <<EOF
CREATE TABLE IF NOT EXISTS orders (
    id SERIAL PRIMARY KEY,
    product VARCHAR(100),
    quantity INT,
    amount INT,
    status VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS inventory (
    id SERIAL PRIMARY KEY,
    product VARCHAR(100),
    stock INT,
    price INT
);

INSERT INTO inventory (product, stock, price) VALUES
    ('Widget', 1000, 1000),
    ('Gadget', 500, 2000),
    ('Doohickey', 750, 1500)
ON CONFLICT DO NOTHING;
EOF
    echo -e "${GREEN}✓ Database initialized${NC}"
}

# Start services
start_services() {
    echo "Starting services..."
    docker-compose up -d
    echo -e "${GREEN}✓ Services started${NC}"
}

# Wait for services to be healthy
wait_for_services() {
    echo "Waiting for services to be healthy..."
    
    services=("prometheus:9090" "grafana:3000" "loki:3100" "tempo:3200")
    
    for service in "${services[@]}"; do
        IFS=':' read -r name port <<< "$service"
        echo -n "Waiting for $name..."
        
        max_attempts=30
        attempt=0
        
        while [ $attempt -lt $max_attempts ]; do
            if curl -s "http://localhost:$port/health" > /dev/null 2>&1 || \
               curl -s "http://localhost:$port/ready" > /dev/null 2>&1 || \
               curl -s "http://localhost:$port/" > /dev/null 2>&1; then
                echo -e " ${GREEN}✓${NC}"
                break
            fi
            
            attempt=$((attempt + 1))
            sleep 2
            echo -n "."
        done
        
        if [ $attempt -eq $max_attempts ]; then
            echo -e " ${RED}✗${NC}"
            echo -e "${RED}Failed to start $name${NC}"
        fi
    done
}

# Display access information
show_info() {
    echo ""
    echo -e "${GREEN}========================================${NC}"
    echo -e "${GREEN}Deployment Complete!${NC}"
    echo -e "${GREEN}========================================${NC}"
    echo ""
    echo "📊 Access URLs:"
    echo "  Frontend:       http://localhost:8080"
    echo "  Grafana:        http://localhost:3000 (admin/admin)"
    echo "  Prometheus:     http://localhost:9090"
    echo "  Alertmanager:   http://localhost:9093"
    echo "  Tempo:          http://localhost:3200"
    echo ""
    echo "📈 Monitoring Features:"
    echo "  - Full observability (metrics, logs, traces)"
    echo "  - SLO monitoring with error budgets"
    echo "  - Business metrics tracking"
    echo "  - Automated alerting"
    echo ""
    echo "🧪 Test the application:"
    echo "  1. Open http://localhost:8080"
    echo "  2. Click 'Place Order' to generate traffic"
    echo "  3. View metrics in Grafana"
    echo ""
}

# Main execution
main() {
    check_prerequisites
    init_database
    start_services
    wait_for_services
    show_info
}

main
```

### 7.2 Load Generator

`scripts/load-test.js`:

javascript

```javascript
import http from 'k6/http';
import { sleep, check } from 'k6';
import { Rate, Trend } from 'k6/metrics';

// Custom metrics
const errorRate = new Rate('errors');
const orderDuration = new Trend('order_duration');

export const options = {
  stages: [
    { duration: '2m', target: 10 },   // Ramp up
    { duration: '5m', target: 10 },   // Steady state
    { duration: '2m', target: 20 },   // Spike
    { duration: '5m', target: 20 },   // Steady spike
    { duration: '2m', target: 0 },    // Ramp down
  ],
  thresholds: {
    'http_req_duration': ['p(95)<2000'], // 95% of requests under 2s
    'errors': ['rate<0.05'],              // Error rate < 5%
  },
};

const BASE_URL = 'http://frontend:8080';

export default function () {
  // Normal order (80%)
  if (Math.random() < 0.8) {
    const res = http.post(`${BASE_URL}/api/order`, JSON.stringify({
      product: 'Widget',
      quantity: Math.floor(Math.random() * 5) + 1
    }), {
      headers: { 'Content-Type': 'application/json' },
    });
    
    const success = check(res, {
      'order created': (r) => r.status === 200,
    });
    
    errorRate.add(!success);
    orderDuration.add(res.timings.duration);
  }
  
  // Slow order (10%)
  else if (Math.random() < 0.1) {
    http.post(`${BASE_URL}/api/order?slow=true`);
  }
  
  // Failing order (5%)
  else if (Math.random() < 0.05) {
    http.post(`${BASE_URL}/api/order?fail=true`);
  }
  
  // Check inventory (5%)
  else {
    http.get(`${BASE_URL}/api/inventory`);
  }
  
  sleep(Math.random() * 3 + 1); // Random sleep 1-4s
}
```

### 7.3 Backup Script

`scripts/backup.sh`:

bash

```bash
#!/bin/bash

BACKUP_DIR="/backup/monitoring"
DATE=$(date +%Y%m%d-%H%M%S)
RETENTION_DAYS=7

echo "Starting backup at $(date)"

mkdir -p "$BACKUP_DIR"

# Backup Grafana
echo "Backing up Grafana..."
docker exec grafana tar czf - /var/lib/grafana > "$BACKUP_DIR/grafana-$DATE.tar.gz"

# Backup Prometheus config
echo "Backing up Prometheus config..."
tar czf "$BACKUP_DIR/prometheus-config-$DATE.tar.gz" monitoring/prometheus/

# Backup alert configuration
tar czf "$BACKUP_DIR/alertmanager-config-$DATE.tar.gz" monitoring/alertmanager/

# Cleanup old backups
echo "Cleaning up old backups..."
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup completed at $(date)"
echo "Backup location: $BACKUP_DIR"
```

### 7.4 SLO Report Generator

`scripts/generate-slo-report.sh`:

bash

```bash
#!/bin/bash

PROMETHEUS_URL="http://localhost:9090"
OUTPUT_FILE="slo-report-$(date +%Y%m%d).md"

echo "Generating SLO Report..."

python3 - <<'EOF'
import requests
from datetime import datetime

PROMETHEUS_URL = "http://localhost:9090"

def query_prometheus(query):
    response = requests.get(
        f"{PROMETHEUS_URL}/api/v1/query",
        params={'query': query}
    )
    data = response.json()
    if data['status'] == 'success' and data['data']['result']:
        return float(data['data']['result'][0]['value'][1])
    return None

# Get metrics
availability = query_prometheus('slo:api_gateway:availability:ratio * 100')
error_budget = query_prometheus('slo:api_gateway:error_budget:ratio * 100')

print(f"# SLO Report - {datetime.now().strftime('%Y-%m-%d')}")
print()
print("## API Gateway")
print(f"- **Availability**: {availability:.4f}%")
print(f"- **Target**: 99.9%")
print(f"- **Status**: {'✅ Met' if availability >= 99.9 else '❌ Violated'}")
print(f"- **Error Budget Remaining**: {error_budget:.2f}%")
print()

if error_budget < 25:
    print("⚠️ **ACTION REQUIRED**: Error budget critical. Implement code freeze.")
elif error_budget < 50:
    print("⚠️ **WARNING**: Error budget low. Reduce release frequency.")
else:
    print("✅ Error budget healthy. Normal operations.")
EOF
```

### 7.5 Chaos Testing Script

`scripts/chaos-test.sh`:

bash

```bash
#!/bin/bash

echo "🔥 Starting Chaos Engineering Tests"

# Test 1: Kill random service
chaos_kill_service() {
    services=("payment-service" "inventory-service" "api-gateway")
    service=${services[$RANDOM % ${#services[@]}]}
    
    echo "Test: Killing $service"
    docker kill $service
    sleep 30
    docker start $service
    echo "✓ Service recovered"
}

# Test 2: Network latency
chaos_network_latency() {
    echo "Test: Adding network latency"
    docker exec payment-service tc qdisc add dev eth0 root netem delay 2000ms
    sleep 60
    docker exec payment-service tc qdisc del dev eth0 root
    echo "✓ Network restored"
}

# Test 3: High CPU load
chaos_cpu_load() {
    echo "Test: Generating CPU load"
    docker exec api-gateway sh -c "yes > /dev/null &"
    PID=$!
    sleep 60
    docker exec api-gateway kill $PID
    echo "✓ CPU load removed"
}

# Run tests
chaos_kill_service
sleep 60
chaos_network_latency
sleep 60
chaos_cpu_load

echo "🎉 Chaos tests completed"
```

---

## Часть 8: Documentation (5 минут)

### 8.1 Main README

`README.md`:

markdown

````markdown
# E-Commerce Monitoring Solution

Complete production-ready monitoring stack for e-commerce platform with full observability.

## 🏗️ Architecture

- **Applications**: 4 microservices (Frontend, API Gateway, Payment, Inventory)
- **Data Stores**: PostgreSQL, Redis
- **Observability**: Prometheus, Loki, Tempo, Grafana
- **Alerting**: Alertmanager with multi-channel notifications
- **SLO Monitoring**: Automated error budget tracking

## 🚀 Quick Start
```bash
# Clone repository
git clone 
cd ecommerce-monitoring

# Deploy stack
chmod +x scripts/deploy.sh
./scripts/deploy.sh

# Access services
# Frontend: http://localhost:8080
# Grafana: http://localhost:3000 (admin/admin)
```

## 📊 Features

### Observability
- ✅ Metrics collection (Prometheus)
- ✅ Distributed tracing (Tempo)
- ✅ Log aggregation (Loki)
- ✅ Unified visualization (Grafana)

### Monitoring
- ✅ Infrastructure metrics
- ✅ Application metrics  
- ✅ Business metrics (orders, revenue)
- ✅ SLO tracking with error budgets

### Alerting
- ✅ Multi-window burn rate alerts
- ✅ Service health monitoring
- ✅ Performance degradation detection
- ✅ Business KPI alerts

## 📈 Dashboards

1. **Overview Dashboard**: High-level system health
2. **SLO Dashboard**: Error budget and compliance
3. **Business Metrics**: Revenue, orders, conversions
4. **Service Mesh**: Dependency visualization

## 🎯 SLO Targets

| Service | SLO | Error Budget |
|---------|-----|--------------|
| API Gateway | 99.9% | 43.2 min/month |
| Payment Service | 99.95% | 21.6 min/month |
| Inventory Service | 99.9% | 43.2 min/month |

## 🧪 Testing
```bash
# Generate load
./scripts/load-test.sh

# Run chaos tests
./scripts/chaos-test.sh

# Generate SLO report
./scripts/generate-slo-report.sh
```

## 📚 Documentation

- [Architecture](docs/architecture.md)
- [Runbooks](docs/runbooks/)
- [SLO Policy](docs/slo-policy.md)

## 🔧 Maintenance
```bash
# Backup
./scripts/backup.sh

# View logs
docker-compose logs -f

# Restart services
docker-compose restart

# Stop everything
docker-compose down
```

## 📞 Support

- Slack: #monitoring-support
- On-call: oncall@example.com
````

### 8.2 Runbook Example

`docs/runbooks/high-error-rate.md`:

markdown

````markdown
# Runbook: High Error Rate

## Alert: HighErrorRate

**Severity**: Critical  
**Team**: Backend

## Symptoms

- Error rate > 5% for 2 minutes
- Users experiencing 5xx errors
- SLO at risk

## Investigation Steps

### 1. Identify affected service
```bash
# Check error rate by service
curl -s 'http://localhost:9090/api/v1/query?query=rate(http_requests_total{status=~"5.."}[5m])' | jq
```

### 2. Check recent deployments
```bash
# View recent changes
git log --since="1 hour ago"

# Check deployment times
kubectl get events --sort-by='.lastTimestamp'
```

### 3. Review traces

- Open Grafana → Explore → Tempo
- Filter by: `status=error` 
- Look for common patterns

### 4. Check dependencies
```promql
# Database errors
rate(pg_stat_database_errors[5m])

# Redis errors  
rate(redis_commands_failed_total[5m])

# External API errors
rate(http_requests_total{endpoint="external"}[5m])
```

## Resolution

### If deployment related:
```bash
# Rollback
kubectl rollout undo deployment/
```

### If database related:
```bash
# Check connections
SELECT count(*) FROM pg_stat_activity;

# Check slow queries
SELECT * FROM pg_stat_statements ORDER BY mean_exec_time DESC LIMIT 10;
```

### If external dependency:
- Enable circuit breaker
- Switch to fallback service
- Contact vendor

## Prevention

- [ ] Add integration tests
- [ ] Improve monitoring
- [ ] Update runbook
- [ ] Schedule post-mortem
````

---

## Часть 9: Финальное тестирование (5 минут)

### 9.1 Verification Checklist

`VERIFICATION.md`:

markdown

```markdown
# Verification Checklist

## Services Health

- [ ] All containers running: `docker-compose ps`
- [ ] Frontend accessible: http://localhost:8080
- [ ] Grafana accessible: http://localhost:3000
- [ ] Prometheus accessible: http://localhost:9090
- [ ] Alertmanager accessible: http://localhost:9093

## Metrics Collection

- [ ] Prometheus targets all UP
- [ ] Metrics flowing from all services
- [ ] Node exporter metrics visible
- [ ] Application metrics visible

## Logging

- [ ] Logs visible in Loki
- [ ] Log correlation with traces working
- [ ] Log levels correct

## Tracing

- [ ] Traces visible in Tempo
- [ ] Service map populated
- [ ] Trace→Log correlation working
- [ ] Span attributes correct

## Dashboards

- [ ] Overview dashboard loading
- [ ] SLO dashboard showing data
- [ ] Business metrics dashboard working
- [ ] All panels displaying correctly

## Alerting

- [ ] Alertmanager config loaded
- [ ] Test alert fires correctly
- [ ] Notifications working
- [ ] Inhibition rules working

## SLO Monitoring

- [ ] SLO metrics calculating correctly
- [ ] Error budget visible
- [ ] Burn rate alerts configured
- [ ] Recording rules working

## End-to-End Test

1. Place order through frontend
2. Verify trace appears in Tempo
3. Check logs in Loki
4. Verify metrics in Prometheus
5. See order count increase in
````markdown
1. Place order through frontend
2. Verify trace appears in Tempo
3. Check logs in Loki
4. Verify metrics in Prometheus
5. See order count increase in Grafana
6. Verify alert doesn't fire (healthy state)
7. Trigger slow order, verify latency alert
8. Trigger failing order, verify error alert

## Load Testing

- [ ] Load generator running
- [ ] Request rate stable
- [ ] Error rate within SLO
- [ ] Latency within targets
- [ ] No memory leaks over time

## SLO Compliance

- [ ] Availability > 99.9%
- [ ] Error budget > 50%
- [ ] Burn rate < warning threshold
- [ ] 30-day trend positive

## Documentation

- [ ] All runbooks accessible
- [ ] Architecture diagram accurate
- [ ] SLO policy documented
- [ ] Backup procedures tested

## Troubleshooting Common Issues

### Issue: Service not showing metrics
```bash
# Check service is running
docker ps | grep 

# Check metrics endpoint
curl http://localhost:/metrics

# Check Prometheus targets
curl http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | select(.health != "up")'
```

### Issue: Traces not appearing
```bash
# Check OTLP endpoint
curl http://localhost:4317

# Verify collector config
docker logs otel-collector | grep -i error

# Check trace in Tempo
curl http://localhost:3200/api/search
```

### Issue: Dashboards not loading
```bash
# Check Grafana logs
docker logs grafana

# Verify datasource connection
curl -u admin:admin http://localhost:3000/api/datasources

# Re-provision dashboards
docker restart grafana
```

### Issue: Alerts not firing
```bash
# Check Prometheus rules
curl http://localhost:9090/api/v1/rules

# Verify Alertmanager config
curl http://localhost:9093/api/v2/status

# Test alert manually
curl -X POST http://localhost:9093/api/v2/alerts
```

### 9.2 Integration Test Script

`scripts/integration-test.sh`:

bash

```bash
#!/bin/bash
set -e

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

FAILED=0

test_service() {
    local name=$1
    local url=$2
    local expected=$3
    
    echo -n "Testing $name... "
    
    response=$(curl -s -o /dev/null -w "%{http_code}" "$url")
    
    if [ "$response" -eq "$expected" ]; then
        echo -e "${GREEN}✓${NC}"
        return 0
    else
        echo -e "${RED}✗ (got $response, expected $expected)${NC}"
        FAILED=$((FAILED + 1))
        return 1
    fi
}

test_metrics() {
    local name=$1
    local url=$2
    
    echo -n "Testing $name metrics... "
    
    if curl -s "$url" | grep -q "^# HELP"; then
        echo -e "${GREEN}✓${NC}"
        return 0
    else
        echo -e "${RED}✗${NC}"
        FAILED=$((FAILED + 1))
        return 1
    fi
}

test_prometheus_target() {
    local job=$1
    
    echo -n "Testing Prometheus target $job... "
    
    result=$(curl -s "http://localhost:9090/api/v1/targets" | \
        jq -r ".data.activeTargets[] | select(.labels.job==\"$job\") | .health")
    
    if [ "$result" = "up" ]; then
        echo -e "${GREEN}✓${NC}"
        return 0
    else
        echo -e "${RED}✗ (status: $result)${NC}"
        FAILED=$((FAILED + 1))
        return 1
    fi
}

test_grafana_datasource() {
    local name=$1
    
    echo -n "Testing Grafana datasource $name... "
    
    result=$(curl -s -u admin:admin "http://localhost:3000/api/datasources/name/$name" | \
        jq -r '.basicAuth')
    
    if [ "$result" != "null" ] || [ -n "$result" ]; then
        echo -e "${GREEN}✓${NC}"
        return 0
    else
        echo -e "${RED}✗${NC}"
        FAILED=$((FAILED + 1))
        return 1
    fi
}

echo "========================================="
echo "Integration Tests"
echo "========================================="
echo ""

echo "## Service Health Checks"
test_service "Frontend" "http://localhost:8080/health" 200
test_service "API Gateway" "http://localhost:8081/health" 200
test_service "Payment Service" "http://localhost:8082/health" 200
test_service "Inventory Service" "http://localhost:8083/health" 200
echo ""

echo "## Metrics Endpoints"
test_metrics "Frontend" "http://localhost:8080/metrics"
test_metrics "API Gateway" "http://localhost:8081/metrics"
test_metrics "Payment Service" "http://localhost:8082/metrics"
test_metrics "Inventory Service" "http://localhost:8083/metrics"
echo ""

echo "## Monitoring Stack"
test_service "Prometheus" "http://localhost:9090/-/healthy" 200
test_service "Grafana" "http://localhost:3000/api/health" 200
test_service "Alertmanager" "http://localhost:9093/-/healthy" 200
test_service "Loki" "http://localhost:3100/ready" 200
test_service "Tempo" "http://localhost:3200/ready" 200
echo ""

echo "## Prometheus Targets"
test_prometheus_target "frontend"
test_prometheus_target "api-gateway"
test_prometheus_target "payment-service"
test_prometheus_target "inventory-service"
echo ""

echo "## Grafana Datasources"
test_grafana_datasource "Prometheus"
test_grafana_datasource "Loki"
test_grafana_datasource "Tempo"
echo ""

echo "## End-to-End Test"
echo -n "Creating test order... "
response=$(curl -s -X POST "http://localhost:8080/api/order" \
    -H "Content-Type: application/json" \
    -d '{"product":"Widget","quantity":1}')

if echo "$response" | jq -e '.order_id' > /dev/null 2>&1; then
    echo -e "${GREEN}✓${NC}"
    order_id=$(echo "$response" | jq -r '.order_id')
    echo "  Order ID: $order_id"
else
    echo -e "${RED}✗${NC}"
    FAILED=$((FAILED + 1))
fi
echo ""

# Wait for metrics to be scraped
echo "Waiting for metrics propagation (15s)..."
sleep 15

echo -n "Verifying order metrics... "
order_count=$(curl -s "http://localhost:9090/api/v1/query?query=orders_created_total" | \
    jq -r '.data.result[0].value[1]')

if [ -n "$order_count" ] && [ "$order_count" != "null" ]; then
    echo -e "${GREEN}✓${NC}"
    echo "  Total orders: $order_count"
else
    echo -e "${RED}✗${NC}"
    FAILED=$((FAILED + 1))
fi
echo ""

echo "========================================="
if [ $FAILED -eq 0 ]; then
    echo -e "${GREEN}All tests passed!${NC}"
    exit 0
else
    echo -e "${RED}$FAILED test(s) failed${NC}"
    exit 1
fi
```

---

## Часть 10: Production Deployment Guide (5 минут)

### 10.1 Production Checklist

`docs/PRODUCTION.md`:

markdown

````markdown
# Production Deployment Guide

## Pre-Deployment Checklist

### Infrastructure
- [ ] Sufficient resources allocated (CPU, Memory, Disk)
- [ ] High availability configured (3+ replicas)
- [ ] Backup strategy in place
- [ ] Disaster recovery plan documented
- [ ] Network policies configured
- [ ] SSL/TLS certificates installed

### Security
- [ ] Default passwords changed
- [ ] Access control configured
- [ ] Secrets stored securely (Vault/AWS Secrets Manager)
- [ ] API keys rotated
- [ ] Network segmentation implemented
- [ ] Security scanning completed

### Monitoring
- [ ] All exporters configured
- [ ] Alert rules validated
- [ ] Notification channels tested
- [ ] SLO targets defined
- [ ] Dashboards reviewed
- [ ] Runbooks updated

### Data Retention
- [ ] Prometheus retention: 30 days
- [ ] Loki retention: 7 days
- [ ] Tempo retention: 7 days
- [ ] Long-term storage configured (Thanos/Mimir)
- [ ] Backup retention: 30 days

## Deployment Steps

### 1. Environment Preparation
```bash
# Set production environment
export ENVIRONMENT=production
export CLUSTER_NAME=prod-cluster

# Update configurations
sed -i 's/development/production/g' monitoring/*/config.yml
```

### 2. Resource Sizing

**Prometheus**
```yaml
resources:
  requests:
    memory: 8Gi
    cpu: 2
  limits:
    memory: 16Gi
    cpu: 4
```

**Grafana**
```yaml
resources:
  requests:
    memory: 2Gi
    cpu: 1
  limits:
    memory: 4Gi
    cpu: 2
```

**Loki**
```yaml
resources:
  requests:
    memory: 4Gi
    cpu: 2
  limits:
    memory: 8Gi
    cpu: 4
```

### 3. High Availability Setup

**Prometheus HA**
```yaml
# Use Thanos for HA
services:
  prometheus-1:
    image: prom/prometheus
    command:
      - --web.enable-lifecycle
      - --storage.tsdb.min-block-duration=2h
      - --storage.tsdb.max-block-duration=2h
    
  prometheus-2:
    image: prom/prometheus
    command:
      - --web.enable-lifecycle
      - --storage.tsdb.min-block-duration=2h
      - --storage.tsdb.max-block-duration=2h
    
  thanos-sidecar-1:
    image: thanosio/thanos
    command:
      - sidecar
      - --prometheus.url=http://prometheus-1:9090
      - --tsdb.path=/prometheus
      - --objstore.config-file=/etc/thanos/bucket.yml
    
  thanos-query:
    image: thanosio/thanos
    command:
      - query
      - --store=thanos-sidecar-1:10901
      - --store=thanos-sidecar-2:10901
```

### 4. Secrets Management

**Using Kubernetes Secrets**
```bash
# Create secrets
kubectl create secret generic grafana-admin \
  --from-literal=username=admin \
  --from-literal=password=$(openssl rand -base64 32)

kubectl create secret generic slack-webhook \
  --from-literal=url=$SLACK_WEBHOOK_URL

kubectl create secret generic pagerduty-key \
  --from-literal=key=$PAGERDUTY_INTEGRATION_KEY
```

**Using AWS Secrets Manager**
```bash
# Store secret
aws secretsmanager create-secret \
  --name monitoring/grafana-admin \
  --secret-string '{"username":"admin","password":"xxx"}'

# Retrieve in application
aws secretsmanager get-secret-value \
  --secret-id monitoring/grafana-admin \
  --query SecretString \
  --output text
```

### 5. TLS Configuration
```yaml
# Grafana with TLS
services:
  grafana:
    environment:
      - GF_SERVER_PROTOCOL=https
      - GF_SERVER_CERT_FILE=/etc/grafana/ssl/cert.pem
      - GF_SERVER_CERT_KEY=/etc/grafana/ssl/key.pem
    volumes:
      - ./ssl:/etc/grafana/ssl:ro
```

### 6. Deploy to Production
```bash
# Deploy with production values
./scripts/deploy.sh --env production

# Verify deployment
./scripts/integration-test.sh

# Enable monitoring
./scripts/enable-monitoring.sh
```

## Post-Deployment

### 1. Smoke Tests
```bash
# Run comprehensive tests
./scripts/smoke-test.sh

# Verify SLOs
./scripts/verify-slo.sh

# Check alert rules
./scripts/test-alerts.sh
```

### 2. Initial Monitoring
- Monitor for 24 hours
- Review all dashboards
- Verify alerts fire correctly
- Check resource usage
- Validate backup jobs

### 3. Team Handoff
- [ ] Documentation shared
- [ ] Runbooks reviewed
- [ ] On-call rotation set up
- [ ] Access granted
- [ ] Training completed

## Scaling Guidelines

### Horizontal Scaling
```yaml
# Prometheus
replicas: 3
sharding:
  enabled: true
  shards: 3

# Loki
replicas: 3
querier:
  replicas: 3
ingester:
  replicas: 3
```

### Vertical Scaling
Monitor these metrics:
- Prometheus: `prometheus_tsdb_head_series`
- Grafana: `grafana_api_response_time_seconds`
- Loki: `loki_ingester_memory_chunks`

Scale when:
- Memory > 80%
- CPU > 70%
- Disk > 85%
- Query latency > 5s

## Troubleshooting Production Issues

### High Memory Usage
```bash
# Check series cardinality
curl http://prometheus:9090/api/v1/status/tsdb | jq '.data.seriesCountByMetricName'

# Reduce cardinality
# - Add relabel_configs to drop high-cardinality labels
# - Increase scrape_interval
# - Use recording rules
```

### Slow Queries
```bash
# Enable query logging
--query.log-file=/var/log/prometheus/query.log

# Analyze slow queries
cat query.log | jq 'select(.duration > 5)'

# Optimize with recording rules
```

### Disk Space Issues
```bash
# Check disk usage
df -h /prometheus

# Reduce retention
--storage.tsdb.retention.time=15d
--storage.tsdb.retention.size=50GB

# Setup long-term storage
# Use Thanos/Mimir for historical data
```

## Maintenance Windows

### Planned Maintenance
```bash
# 1. Create silence in Alertmanager
amtool silence add --duration=2h --comment="Planned maintenance"

# 2. Notify stakeholders
./scripts/notify-maintenance.sh

# 3. Perform maintenance
./scripts/maintenance.sh

# 4. Verify system
./scripts/integration-test.sh

# 5. Remove silence
amtool silence expire 
```

### Emergency Rollback
```bash
# Quick rollback
docker-compose down
docker-compose -f docker-compose.yml -f docker-compose.backup.yml up -d

# Or use Kubernetes
kubectl rollout undo deployment/prometheus
```

## Support Contacts

- **On-Call**: oncall@example.com
- **Slack**: #monitoring-support
- **Escalation**: sre-lead@example.com
- **Emergency**: +1-555-0100
````

---

## Часть 11: Graduation & Next Steps (5 минут)

### 11.1 Course Completion Certificate

`docs/COMPLETION.md`:

markdown

````markdown
# 🎓 Course Completion

Congratulations! You've completed the **Complete Monitoring for DevOps** course.

## What You've Learned

### Core Concepts ✅
- Four Golden Signals (Latency, Traffic, Errors, Saturation)
- USE and RED methods
- Observability pillars (Metrics, Logs, Traces)
- SLI/SLO/SLA and Error Budgets

### Technical Skills ✅
- Prometheus configuration and PromQL
- Grafana dashboards and alerting
- Loki for log aggregation
- Jaeger/Tempo for distributed tracing
- Alertmanager for notifications
- OpenTelemetry instrumentation

### Advanced Topics ✅
- Multi-window multi-burn-rate alerting
- Error Budget policies
- Infrastructure as Code for monitoring
- GitOps workflows
- SLO-based alerting
- Composite SLOs

### Production Readiness ✅
- High availability setup
- Security best practices
- Scaling strategies
- Disaster recovery
- Runbook creation
- Incident management

## Your Monitoring Stack

You now have a production-ready monitoring solution featuring:
- 📊 Complete observability (metrics, logs, traces)
- 🎯 SLO monitoring with error budgets
- 🚨 Intelligent alerting with multiple channels
- 📈 Business metrics tracking
- 🔄 Automated deployment via IaC
- 📚 Comprehensive documentation

## Next Steps

### Immediate (This Week)
1. **Deploy to staging**: Test in your staging environment
2. **Customize dashboards**: Adapt to your specific needs
3. **Define SLOs**: Set realistic targets for your services
4. **Write runbooks**: Document procedures for common issues

### Short-term (This Month)
1. **Production deployment**: Roll out with proper planning
2. **Team training**: Educate your team on the stack
3. **Alert tuning**: Reduce noise, improve signal
4. **Integration**: Connect with existing tools (Slack, PagerDuty)

### Long-term (This Quarter)
1. **Capacity planning**: Use metrics for resource forecasting
2. **Cost optimization**: Identify and reduce waste
3. **Advanced features**: Implement anomaly detection, AI-powered insights
4. **Continuous improvement**: Regular review and enhancement

## Advanced Learning Paths

### Path 1: Cloud-Native Monitoring
- Kubernetes monitoring with kube-prometheus-stack
- Service mesh observability (Istio, Linkerd)
- Cloud provider monitoring (AWS CloudWatch, GCP Monitoring)
- Multi-cluster monitoring with Thanos

**Resources**:
- Kubernetes Monitoring Guide: https://kubernetes.io/docs/tasks/debug/
- Istio Observability: https://istio.io/latest/docs/tasks/observability/
- Thanos Documentation: https://thanos.io/

### Path 2: Advanced Observability
- OpenTelemetry advanced features
- Continuous profiling (Pyroscope, Parca)
- Real User Monitoring (RUM)
- Synthetic monitoring
- Chaos engineering with monitoring

**Resources**:
- OpenTelemetry docs: https://opentelemetry.io/docs/
- Grafana Tempo deep dive: https://grafana.com/docs/tempo/
- Chaos Engineering book: "Chaos Engineering" by Casey Rosenthal

### Path 3: SRE Mastery
- SLO engineering at scale
- Error budget policies
- Incident management
- Post-mortem culture
- Toil reduction

**Resources**:
- "Site Reliability Engineering" (Google SRE book)
- "The Site Reliability Workbook"
- SLO workshop materials: https://sre.google/workbook/

### Path 4: AI/ML in Monitoring
- Anomaly detection
- Predictive alerting
- Intelligent incident management
- Auto-remediation
- AIOps platforms

**Resources**:
- Prometheus ML: https://github.com/grafana/cortex-tools
- Anomaly detection patterns
- AIOps platforms evaluation

## Community & Resources

### Official Documentation
- **Prometheus**: https://prometheus.io/docs/
- **Grafana**: https://grafana.com/docs/
- **OpenTelemetry**: https://opentelemetry.io/docs/

### Communities
- **CNCF Slack**: cloud-native.slack.com
- **Prometheus Users**: groups.google.com/forum/#!forum/prometheus-users
- **r/devops**: reddit.com/r/devops
- **SRE Weekly**: sreweekly.com

### Conferences
- **KubeCon + CloudNativeCon**
- **Observability Meetups**
- **SREcon**
- **Monitorama**

### Certifications
- **Certified Kubernetes Administrator (CKA)**
- **Prometheus Certified Associate**
- **AWS/GCP/Azure Monitoring Certifications**
- **Site Reliability Engineering Certification**

## Your Monitoring Journey
```
Where you started               Where you are now              Where you're going
     ❓                              ✅                            🚀
                                    
Basic monitoring           Complete observability        Industry expert
Manual alerts              Intelligent alerting          Thought leader
Reactive approach          SLO-based decisions          Proactive optimization
Limited visibility         Full stack observability      Predictive systems
```

## Final Checklist

Before considering yourself "done", ensure:
- [ ] Complete monitoring stack running
- [ ] All tests passing
- [ ] Documentation complete
- [ ] Team trained
- [ ] Production deployed
- [ ] SLOs defined and tracked
- [ ] Runbooks written
- [ ] Alerts tuned and tested
- [ ] Backup and DR tested
- [ ] On-call rotation established

## Staying Current

Monitoring is an evolving field. Stay updated:
- Subscribe to monitoring blogs
- Follow industry leaders on Twitter/LinkedIn
- Participate in online communities
- Attend webinars and conferences
- Read new releases and changelogs
- Experiment with emerging tools

## Give Back

Help others on their monitoring journey:
- Write blog posts about your experience
- Contribute to open-source projects
- Share dashboards on grafana.com
- Answer questions in forums
- Mentor junior engineers
- Speak at meetups

## Thank You!

Thank you for completing this course. Remember:

> "Monitoring is not about the tools, it's about the insights and actions they enable."

Now go build amazing, reliable systems! 🚀

---

**Course Author**: [Your Name]  
**Version**: 1.0  
**Last Updated**: 2025-01-05

**Feedback**: We'd love to hear about your experience!  
- Email: monitoring-course@example.com
- GitHub: github.com/your-org/monitoring-course
````

### 11.2 Final Exercise

`docs/FINAL_EXERCISE.md`:

markdown

```markdown
# Final Exercise: Build Your Own Monitoring Solution

## Objective
Apply everything you've learned to monitor a real application of your choice.

## Requirements

### 1. Choose Your Application
Select one of:
- Your existing production application
- Open-source project (e.g., WordPress, GitLab, Jenkins)
- Sample e-commerce/blog platform
- Your side project

### 2. Implement Complete Observability

#### Metrics (Required)
- [ ] Install Prometheus
- [ ] Add at least 3 exporters
- [ ] Create 5+ custom metrics
- [ ] Set up recording rules
- [ ] Configure scrape targets

#### Logs (Required)
- [ ] Set up Loki
- [ ] Configure log collection
- [ ] Parse structured logs
- [ ] Create log-based alerts
- [ ] Set up retention policies

#### Traces (Required)
- [ ] Implement distributed tracing
- [ ] Instrument your application
- [ ] Set up Tempo/Jaeger
- [ ] Create service dependency map
- [ ] Link traces with logs

#### Dashboards (Required)
- [ ] Create overview dashboard
- [ ] Create service-specific dashboards
- [ ] Add business metrics dashboard
- [ ] Include SLO dashboard
- [ ] Make them production-ready

#### Alerting (Required)
- [ ] Define 10+ alert rules
- [ ] Set up Alertmanager
- [ ] Configure notification channels
- [ ] Create inhibition rules
- [ ] Test all alerts

#### SLO Monitoring (Required)
- [ ] Define 3+ SLOs
- [ ] Set error budgets
- [ ] Create burn rate alerts
- [ ] Build SLO dashboard
- [ ] Document SLO policy

### 3. Documentation

Create:
- Architecture diagram
- Monitoring strategy document
- 5+ runbooks for common issues
- On-call procedures
- Disaster recovery plan

### 4. Infrastructure as Code

Use:
- Terraform for Grafana configuration
- Ansible for deployment, OR
- Helm charts for Kubernetes, OR
- Docker Compose for simple setups

### 5. Advanced Features (Choose 2)

- [ ] Implement continuous profiling
- [ ] Add synthetic monitoring
- [ ] Set up anomaly detection
- [ ] Create custom exporters
- [ ] Implement chaos testing
- [ ] Build cost tracking dashboard
- [ ] Add security monitoring
- [ ] Implement capacity planning

## Deliverables

Submit:
1. **GitHub repository** with:
   - All configuration files
   - Application code with instrumentation
   - Documentation
   - IaC scripts
   
2. **Video demo** (5-10 minutes) showing:
   - System overview
   - Key dashboards
   - Alert demonstration
   - Trace walkthrough
   
3. **Written report** including:
   - Design decisions
   - Challenges faced
   - SLO definitions
   - Future improvements

## Evaluation Criteria

### Technical Implementation (40%)
- Completeness of observability
- Quality of instrumentation
- Proper use of tools
- Security considerations

### Design & Architecture (20%)
- Scalability
- High availability
- Maintainability
- Best practices

### Documentation (20%)
- Clarity and completeness
- Runbooks quality
- Architecture diagrams
- Code comments

### Operational Readiness (20%)
- Production-ready setup
- Disaster recovery plan
- Alert quality
- SLO definitions

## Timeline

Recommended schedule:
- **Week 1**: Planning and setup
- **Week 2**: Implementation
- **Week 3**: Testing and tuning
- **Week 4**: Documentation and demo

## Getting Help

- Review course materials
- Check documentation
- Ask in community forums
- Consult with peers

## Bonus Challenges

Go above and beyond:
- Multi-region monitoring
- Custom AI/ML models
- Integration with CI/CD
- Automated remediation
- Cost optimization dashboard
- Mobile app monitoring
- User experience monitoring

## Success Stories

Share your project:
- Blog about your experience
- Present at a meetup
- Submit to CNCF showcase
- Add to your portfolio

Good luck! 🚀
```

---

## 🎉 Поздравляем с завершением курса!

Вы прошли полный путь от базовых метрик до production-ready monitoring solution. Теперь у вас есть:

1. ✅ **Полноценный monitoring stack** с метриками, логами и трейсами
2. ✅ **SLO мониторинг** с error budget tracking
3. ✅ **Интеллектуальный alerting** с multi-window burn rates
4. ✅ **Production-ready инфраструктура** с HA и DR
5. ✅ **Автоматизация** через IaC и GitOps
6. ✅ **Полная документация** с runbooks и procedures

**Следующие шаги:**

1. Примените знания к вашим реальным проектам
2. Поделитесь опытом с командой
3. Продолжайте учиться и совершенствоваться
4. Помогайте другим в их monitoring journey

**Помните:** Мониторинг - это непрерывный процесс улучшения. Удачи! 🚀
