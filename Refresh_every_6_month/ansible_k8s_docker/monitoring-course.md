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
```
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
````

**Log best practices:**
````
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
````
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
````

**4. Создай log analysis dashboard**:

Grafana panels для анализа логов:
````
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
````

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
````

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
````
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
import requests

app = Flask(__name__)

@app.route("/webhook/jira", methods=["POST"])
def jira_webhook():
    """
    Создает Jira ticket для критичных алертов
    (status=firing, severity=critical)
    """
    data = request.json or {}

    if data.get("status") == "firing":
        for alert in data.get("alerts", []):
            if alert.get("labels", {}).get("severity") == "critical":
                create_jira_ticket(alert)

    return jsonify({"status": "ok"}), 200

def create_jira_ticket(alert):
    """Создает Jira ticket через API"""

    jira_url = "https://your-jira.atlassian.net/rest/api/2/issue"

    ticket = {
        "fields": {
            "project": {"key": "OPS"},
            "summary": f"[ALERT] {alert['labels'].get('alertname', 'Unknown alert')}",
            "description": alert.get("annotations", {}).get(
                "description", "No description provided"
            ),
            "issuetype": {"name": "Incident"},
            "priority": {"name": "Critical"},
            "labels": ["alert", "monitoring"],
        }
    }

    response = requests.post(
        jira_url,
        json=ticket,
        auth=("user@example.com", "jira-api-token"),
        headers={"Content-Type": "application/json"},
        timeout=10,
    )

    if response.ok:
        print(f"Jira ticket created: {response.json().get('key')}")
    else:
        print(f"Failed to create Jira ticket: {response.text}")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
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
````

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
````
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
````

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
````

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
```

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
````

**Best practices:**
````
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
````

`demo-app/backend/requirements.txt`:
````
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
from opentelemetry.trace.status import Status, StatusCode

import psycopg2
import redis
import time
import random
import os
import json

# ---------------------------------------------------------------------
# OpenTelemetry configuration
# ---------------------------------------------------------------------

resource = Resource.create(
    {
        "service.name": os.getenv("OTEL_SERVICE_NAME", "backend"),
        "service.version": "1.0.0",
        "deployment.environment": "development",
    }
)

provider = TracerProvider(resource=resource)
processor = BatchSpanProcessor(
    OTLPSpanExporter(
        endpoint=os.getenv("OTEL_EXPORTER_OTLP_ENDPOINT", "http://localhost:4317"),
        insecure=True,
    )
)
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)

tracer = trace.get_tracer(__name__)

# ---------------------------------------------------------------------
# Flask app
# ---------------------------------------------------------------------

app = Flask(__name__)

FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
Psycopg2Instrumentor().instrument()
RedisInstrumentor().instrument()

# ---------------------------------------------------------------------
# Connections
# ---------------------------------------------------------------------

DATABASE_URL = os.getenv(
    "DATABASE_URL", "postgresql://user:password@localhost:5432/demo"
)
REDIS_URL = os.getenv("REDIS_URL", "redis://localhost:6379")


def get_db_connection():
    return psycopg2.connect(DATABASE_URL)


def get_redis_connection():
    return redis.from_url(REDIS_URL)


# ---------------------------------------------------------------------
# Database init
# ---------------------------------------------------------------------

def init_db():
    with get_db_connection() as conn:
        with conn.cursor() as cur:
            cur.execute(
                """
                CREATE TABLE IF NOT EXISTS users (
                    id SERIAL PRIMARY KEY,
                    name VARCHAR(100),
                    email VARCHAR(100),
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
                """
            )
            cur.execute(
                """
                CREATE TABLE IF NOT EXISTS orders (
                    id SERIAL PRIMARY KEY,
                    user_id INTEGER REFERENCES users(id),
                    product VARCHAR(100),
                    amount DECIMAL(10, 2),
                    status VARCHAR(20),
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
                """
            )
            conn.commit()


# ---------------------------------------------------------------------
# Routes
# ---------------------------------------------------------------------

@app.route("/health")
def health():
    return jsonify({"status": "healthy"}), 200


@app.route("/api/users", methods=["GET"])
def get_users():
    with tracer.start_as_current_span("get_users") as span:
        time.sleep(random.uniform(0.01, 0.1))

        r = get_redis_connection()
        cached = r.get("users:all")

        if cached:
            span.set_attribute("cache.hit", True)
            return jsonify(json.loads(cached)), 200

        span.set_attribute("cache.hit", False)

        with get_db_connection() as conn:
            with conn.cursor() as cur:
                cur.execute("SELECT id, name, email FROM users")
                users = [
                    {"id": r[0], "name": r[1], "email": r[2]}
                    for r in cur.fetchall()
                ]

        r.setex("users:all", 60, json.dumps(users))
        return jsonify(users), 200


@app.route("/api/users/<int:user_id>", methods=["GET"])
def get_user(user_id):
    with tracer.start_as_current_span("get_user_by_id") as span:
        span.set_attribute("user.id", user_id)
        time.sleep(random.uniform(0.01, 0.05))

        with get_db_connection() as conn:
            with conn.cursor() as cur:
                cur.execute(
                    "SELECT id, name, email FROM users WHERE id = %s",
                    (user_id,),
                )
                row = cur.fetchone()

        if not row:
            return jsonify({"error": "User not found"}), 404

        return jsonify(
            {"id": row[0], "name": row[1], "email": row[2]}
        ), 200


@app.route("/api/users", methods=["POST"])
def create_user():
    with tracer.start_as_current_span("create_user") as span:
        data = request.json or {}

        if not data.get("name") or not data.get("email"):
            span.set_status(Status(StatusCode.ERROR))
            return jsonify({"error": "Name and email required"}), 400

        time.sleep(random.uniform(0.05, 0.15))

        with get_db_connection() as conn:
            with conn.cursor() as cur:
                cur.execute(
                    "INSERT INTO users (name, email) VALUES (%s, %s) RETURNING id",
                    (data["name"], data["email"]),
                )
                user_id = cur.fetchone()[0]
                conn.commit()

        get_redis_connection().delete("users:all")

        return jsonify(
            {"id": user_id, "name": data["name"], "email": data["email"]}
        ), 201


@app.route("/api/orders", methods=["POST"])
def create_order():
    with tracer.start_as_current_span("create_order") as span:
        data = request.json or {}

        user_id = data.get("user_id")
        product = data.get("product")
        amount = data.get("amount")

        if not all([user_id, product, amount]):
            span.set_status(Status(StatusCode.ERROR))
            return jsonify({"error": "Missing required fields"}), 400

        # Check user
        with get_db_connection() as conn:
            with conn.cursor() as cur:
                cur.execute("SELECT id FROM users WHERE id = %s", (user_id,))
                if not cur.fetchone():
                    return jsonify({"error": "User not found"}), 404

        # Payment simulation
        time.sleep(random.uniform(0.1, 0.5))

        # Save order
        with get_db_connection() as conn:
            with conn.cursor() as cur:
                cur.execute(
                    """
                    INSERT INTO orders (user_id, product, amount, status)
                    VALUES (%s, %s, %s, %s)
                    RETURNING id
                    """,
                    (user_id, product, amount, "completed"),
                )
                order_id = cur.fetchone()[0]
                conn.commit()

        return jsonify(
            {
                "id": order_id,
                "user_id": user_id,
                "product": product,
                "amount": amount,
                "status": "completed",
            }
        ), 201


@app.route("/api/orders/<int:order_id>", methods=["GET"])
def get_order(order_id):
    with tracer.start_as_current_span("get_order") as span:
        time.sleep(random.uniform(0.01, 0.05))

        with get_db_connection() as conn:
            with conn.cursor() as cur:
                cur.execute(
                    """
                    SELECT id, user_id, product, amount, status
                    FROM orders WHERE id = %s
                    """,
                    (order_id,),
                )
                row = cur.fetchone()

        if not row:
            return jsonify({"error": "Order not found"}), 404

        return jsonify(
            {
                "id": row[0],
                "user_id": row[1],
                "product": row[2],
                "amount": float(row[3]),
                "status": row[4],
            }
        ), 200


@app.route("/api/slow")
def slow_endpoint():
    with tracer.start_as_current_span("slow_endpoint") as span:
        delay = random.uniform(2, 5)
        time.sleep(delay)
        return jsonify({"delay": delay}), 200


@app.route("/api/error")
def error_endpoint():
    with tracer.start_as_current_span("error_endpoint") as span:
        try:
            raise RuntimeError("Simulated error for testing")
        except Exception as e:
            span.record_exception(e)
            span.set_status(Status(StatusCode.ERROR, str(e)))
            return jsonify({"error": str(e)}), 500


# ---------------------------------------------------------------------
# Entry point - Запуск приложения
# ---------------------------------------------------------------------

if __name__ == "__main__":
    init_db()
    app.run(host="0.0.0.0", port=5000)
```
```
````

4. **Создай demo-app/frontend (простой HTML + JS)**:

`demo-app/frontend/Dockerfile`:
```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 8080
```

`demo-app/frontend/index.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tracing Demo App</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 50px auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            background: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        h1 {
            color: #333;
            border-bottom: 2px solid #007bff;
            padding-bottom: 10px;
        }
        .section {
            margin: 20px 0;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        button {
            background-color: #007bff;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            margin: 5px;
        }
        button:hover {
            background-color: #0056b3;
        }
        .error {
            background-color: #dc3545;
        }
        .error:hover {
            background-color: #c82333;
        }
        .slow {
            background-color: #ffc107;
        }
        .slow:hover {
            background-color: #e0a800;
        }
        #output {
            margin-top: 20px;
            padding: 15px;
            background-color: #f8f9fa;
            border-radius: 4px;
            min-height: 100px;
            white-space: pre-wrap;
            font-family: monospace;
        }
        input {
            padding: 8px;
            margin: 5px;
            border: 1px solid #ddd;
            border-radius: 4px;
            width: 200px;
        }
        .links {
            margin-top: 30px;
            padding: 20px;
            background-color: #e9ecef;
            border-radius: 4px;
        }
        .links a {
            display: block;
            margin: 10px 0;
            color: #007bff;
            text-decoration: none;
        }
        .links a:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🔍 Distributed Tracing Demo</h1>
        
        <div class="section">
            <h2>User Operations</h2>
            <button onclick="getUsers()">Get All Users</button>
            <button onclick="createUser()">Create Random User</button>
            <br>
            <input type="number" id="userId" placeholder="User ID">
            <button onclick="getUser()">Get User by ID</button>
        </div>
        
        <div class="section">
            <h2>Order Operations</h2>
            <input type="number" id="orderUserId" placeholder="User ID">
            <input type="text" id="product" placeholder="Product">
            <input type="number" id="amount" placeholder="Amount">
            <button onclick="createOrder()">Create Order</button>
            <br><br>
            <input type="number" id="orderId" placeholder="Order ID">
            <button onclick="getOrder()">Get Order by ID</button>
        </div>
        
        <div class="section">
            <h2>Test Scenarios</h2>
            <button class="slow" onclick="testSlow()">Test Slow Endpoint (2-5s)</button>
            <button class="error" onclick="testError()">Test Error Endpoint</button>
            <button onclick="stressTest()">Stress Test (10 requests)</button>
        </div>
        
        <div id="output">Response will appear here...</div>
        
        <div class="links">
            <h3>📊 Monitoring Links</h3>
            <a href="http://localhost:16686" target="_blank">🔍 Jaeger UI - View Traces</a>
            <a href="http://localhost:3000" target="_blank">📈 Grafana - Metrics & Traces</a>
            <a href="http://localhost:5000/health" target="_blank">💚 Backend Health Check</a>
        </div>
    </div>

    <script>
        const API_URL = 'http://localhost:5000/api';
        const output = document.getElementById('output');

        function log(message, data = null) {
            const timestamp = new Date().toISOString();
            let logMessage = `[${timestamp}] ${message}`;
            if (data) {
                logMessage += '\n' + JSON.stringify(data, null, 2);
            }
            output.textContent = logMessage;
            console.log(message, data);
        }

        async function getUsers() {
            try {
                log('Fetching all users...');
                const response = await fetch(`${API_URL}/users`);
                const data = await response.json();
                log('✅ Users retrieved:', data);
            } catch (error) {
                log('❌ Error:', error.message);
            }
        }

        async function getUser() {
            const userId = document.getElementById('userId').value;
            if (!userId) {
                log('❌ Please enter a user ID');
                return;
            }
            
            try {
                log(`Fetching user ${userId}...`);
                const response = await fetch(`${API_URL}/users/${userId}`);
                const data = await response.json();
                
                if (response.ok) {
                    log('✅ User retrieved:', data);
                } else {
                    log('❌ Error:', data);
                }
            } catch (error) {
                log('❌ Error:', error.message);
            }
        }

        async function createUser() {
            const randomNum = Math.floor(Math.random() * 1000);
            const userData = {
                name: `User ${randomNum}`,
                email: `user${randomNum}@example.com`
            };
            
            try {
                log('Creating user...', userData);
                const response = await fetch(`${API_URL}/users`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify(userData)
                });
                const data = await response.json();
                log('✅ User created:', data);
            } catch (error) {
                log('❌ Error:', error.message);
            }
        }

        async function createOrder() {
            const orderData = {
                user_id: parseInt(document.getElementById('orderUserId').value),
                product: document.getElementById('product').value || 'Product',
                amount: parseFloat(document.getElementById('amount').value) || 99.99
            };
            
            if (!orderData.user_id) {
                log('❌ Please enter a user ID');
                return;
            }
            
            try {
                log('Creating order...', orderData);
                const response = await fetch(`${API_URL}/orders`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify(orderData)
                });
                const data = await response.json();
                
                if (response.ok) {
                    log('✅ Order created:', data);
                } else {
                    log('❌ Error:', data);
                }
            } catch (error) {
                log('❌ Error:', error.message);
            }
        }

        async function getOrder() {
            const orderId = document.getElementById('orderId').value;
            if (!orderId) {
                log('❌ Please enter an order ID');
                return;
            }
            
            try {
                log(`Fetching order ${orderId}...`);
                const response = await fetch(`${API_URL}/orders/${orderId}`);
                const data = await response.json();
                
                if (response.ok) {
                    log('✅ Order retrieved:', data);
                } else {
                    log('❌ Error:', data);
                }
            } catch (error) {
                log('❌ Error:', error.message);
            }
        }

        async function testSlow() {
            try {
                log('⏳ Testing slow endpoint (this will take 2-5 seconds)...');
                const start = Date.now();
                const response = await fetch(`${API_URL}/slow`);
                const data = await response.json();
                const duration = ((Date.now() - start) / 1000).toFixed(2);
                log(`✅ Slow endpoint completed in ${duration}s:`, data);
            } catch (error) {
                log('❌ Error:', error.message);
            }
        }

        async function testError() {
            try {
                log('💥 Testing error endpoint...');
                const response = await fetch(`${API_URL}/error`);
                const data = await response.json();
                log('❌ Expected error:', data);
            } catch (error) {
                log('❌ Error:', error.message);
            }
        }

        async function stressTest() {
            log('🔥 Starting stress test with 10 parallel requests...');
            const promises = [];
            
            for (let i = 0; i < 10; i++) {
                promises.push(fetch(`${API_URL}/users`));
            }
            
            try {
                const start = Date.now();
                await Promise.all(promises);
                const duration = ((Date.now() - start) / 1000).toFixed(2);
                log(`✅ Stress test completed in ${duration}s (10 requests)`);
            } catch (error) {
                log('❌ Stress test failed:', error.message);
            }
        }

        // Initial message
        log('👋 Welcome! Click any button to start generating traces.');
    </script>
</body>
</html>
```

`demo-app/frontend/nginx.conf`:
```nginx
server {
    listen 8080;
    server_name localhost;
    
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
    
    # CORS для API запросов
    location /api {
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'Content-Type';
            return 204;
        }
    }
}
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
      tracesToLogsV2:
        datasourceUid: 'loki'
        spanStartTimeShift: '-1h'
        spanEndTimeShift: '1h'
        filterByTraceID: true
        filterBySpanID: false
```

6. **Запусти и протестируй**:
```bash
# Создай директории
mkdir -p demo-app/frontend demo-app/backend

# Запусти stack
docker-compose up -d

# Проверка статуса
docker-compose ps

# Проверка логов
docker-compose logs -f backend

# Проверка Jaeger
curl http://localhost:16686

# Проверка backend health
curl http://localhost:5000/health
```

7. **Открой UI и тестируй**:
```bash
# Frontend demo app
open http://localhost:8080

# Jaeger UI
open http://localhost:16686

# Grafana
open http://localhost:3000

# Создай тестовые данные
curl -X POST http://localhost:5000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Test User", "email": "test@example.com"}'

curl -X POST http://localhost:5000/api/orders \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1, "product": "Test Product", "amount": 99.99}'
```

8. **Анализ трейсов в Jaeger**:
````

1. Открой Jaeger UI: [http://localhost:16686](http://localhost:16686)
2. Search traces:
    - Service: backend
    - Operation: create_order
    - Min Duration: 1s (для медленных)
    - Tags: error=true (для ошибок)
3. Анализируй:
    - Timeline view - где время тратится
    - Span details - атрибуты, события, ошибки
    - Service graph - карта зависимостей
    - Trace comparison - сравнение быстрых и медленных

````

### 🚀 Бонус (новое)

**1. Интеграция Tempo (альтернатива Jaeger)**:

Добавь в `docker-compose.yml`:
```yaml
  tempo:
    image: grafana/tempo:2.3.1
    container_name: tempo
    command: ["-config.file=/etc/tempo.yaml"]
    volumes:
      - ./tempo.yaml:/etc/tempo.yaml
      - tempo-data:/tmp/tempo
    ports:
      - "3200:3200"   # Tempo UI
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
    restart: unless-stopped

volumes:
  tempo-data:
```

`tempo.yaml`:
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

metrics_generator:
  registry:
    external_labels:
      source: tempo
  storage:
    path: /tmp/tempo/generator/wal
  traces_storage:
    path: /tmp/tempo/generator/traces
```

**2. Создай Python скрипт для load testing с трейсингом**:

`load_test.py`:
```python
#!/usr/bin/env python3
"""
Load testing с генерацией трейсов
"""
import concurrent.futures
import requests
import time
import random
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.resources import Resource

# Setup tracing
resource = Resource.create({"service.name": "load-tester"})
provider = TracerProvider(resource=resource)
processor = BatchSpanProcessor(
    OTLPSpanExporter(endpoint="http://localhost:4317", insecure=True)
)
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

API_URL = "http://localhost:5000/api"

def make_request(endpoint, method="GET", data=None):
    """Делает запрос с трейсингом"""
    with tracer.start_as_current_span(f"{method} {endpoint}") as span:
        span.set_attribute("http.method", method)
        span.set_attribute("http.url", f"{API_URL}{endpoint}")
        
        try:
            if method == "GET":
                response = requests.get(f"{API_URL}{endpoint}")
            else:
                response = requests.post(
                    f"{API_URL}{endpoint}",
                    json=data,
                    headers={"Content-Type": "application/json"}
                )
            
            span.set_attribute("http.status_code", response.status_code)
            
            if response.status_code >= 400:
                span.set_attribute("error", True)
                
            return response
            
        except Exception as e:
            span.record_exception(e)
            span.set_attribute("error", True)
            raise

def user_flow():
    """Симулирует типичный user flow"""
    with tracer.start_as_current_span("user_flow") as span:
        # 1. Создаем пользователя
        user_data = {
            "name": f"LoadTest User {random.randint(1, 1000)}",
            "email": f"test{random.randint(1, 1000)}@example.com"
        }
        response = make_request("/users", "POST", user_data)
        
        if response.status_code != 201:
            span.set_attribute("flow.failed", True)
            return
        
        user_id = response.json()["id"]
        span.set_attribute("user.id", user_id)
        
        # 2. Получаем пользователя
        time.sleep(random.uniform(0.1, 0.5))
        make_request(f"/users/{user_id}")
        
        # 3. Создаем заказ
        time.sleep(random.uniform(0.1, 0.5))
        order_data = {
            "user_id": user_id,
            "product": f"Product {random.randint(1, 100)}",
            "amount": round(random.uniform(10, 500), 2)
        }
        response = make_request("/orders", "POST", order_data)
        
        if response.status_code == 201:
            order_id = response.json()["id"]
            span.set_attribute("order.id", order_id)
            
            # 4. Получаем заказ
            time.sleep(random.uniform(0.1, 0.5))
            make_request(f"/orders/{order_id}")
        
        span.set_attribute("flow.completed", True)

def run_load_test(num_users=10, concurrent=5):
    """Запускает load test"""
    print(f"Starting load test: {num_users} users, {concurrent} concurrent")
    
    start_time = time.time()
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=concurrent) as executor:
        futures = [executor.submit(user_flow) for _ in range(num_users)]
        
        for future in concurrent.futures.as_completed(futures):
            try:
                future.result()
            except Exception as e:
                print(f"Error: {e}")
    
    duration = time.time() - start_time
    print(f"Load test completed in {duration:.2f}s")
    print(f"Average: {duration/num_users:.2f}s per user")
    print(f"Throughput: {num_users/duration:.2f} users/sec")

if __name__ == "__main__":
    import argparse
    
    parser = argparse.ArgumentParser(description='Load testing with tracing')
    parser.add_argument('--users', type=int, default=10, help='Number of users')
    parser.add_argument('--concurrent', type=int, default=5, help='Concurrent requests')
    
    args = parser.parse_args()
    
    run_load_test(num_users=args.users, concurrent=args.concurrent)
```

**3. Создай dashboard для APM в Grafana**:

`grafana-dashboards/apm-dashboard.json`:
```json
{
  "dashboard": {
    "title": "Application Performance Monitoring",
    "panels": [
      {
        "id": 1,
        "title": "Request Rate",
        "targets": [
          {
            "expr": "sum(rate(traces_spanmetrics_calls_total[5m])) by (service_name)"
          }
        ],
        "type": "timeseries"
      },
      {
        "id": 2,
        "title": "Error Rate",
        "targets": [
          {
            "expr": "sum(rate(traces_spanmetrics_calls_total{status_code=\"STATUS_CODE_ERROR\"}[5m])) by (service_name)"
          }
        ],
        "type": "timeseries"
      },
      {
        "id": 3,
        "title": "Latency (p95)",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(traces_spanmetrics_latency_bucket[5m])) by (le, service_name))"
          }
        ],
        "type": "timeseries"
      },
      {
        "id": 4,
        "title": "Service Map",
        "type": "nodeGraph",
        "targets": [
          {
            "queryType": "serviceMap"
          }
        ]
      }
    ]
  }
}
```

**4. Continuous Profiling с Pyroscope**:

Добавь в `docker-compose.yml`:
```yaml
  pyroscope:
    image: grafana/pyroscope:latest
    container_name: pyroscope
    ports:
      - "4040:4040"
    restart: unless-stopped
```

Обнови Python app для профилирования:
```python
import pyroscope

pyroscope.configure(
    application_name="backend",
    server_address="http://pyroscope:4040",
    tags={
        "environment": "development",
    }
)
```

---

## Итоги модуля 6

После прохождения этого модуля ты должен уметь:

✅ Понимать концепции distributed tracing (trace, span, context)
✅ Настраивать OpenTelemetry в приложениях
✅ Использовать Jaeger для анализа трейсов
✅ Интегрировать трейсы с логами и метриками
✅ Настраивать sampling для оптимизации хранения
✅ Анализировать performance bottlenecks
✅ Использовать Service Map для визуализации зависимостей
✅ Интегрировать continuous profiling
✅ Создавать APM dashboards
✅ Отлаживать проблемы в распределенных системах

**Ключевые takeaways:**
1. Трейсинг критичен для микросервисов - без него невозможно отладить проблемы
2. OpenTelemetry - современный стандарт, используй его
3. Всегда сохраняй ошибки и медленные запросы (tail sampling)
4. Связывай трейсы с логами через trace_id
5. Используй semantic conventions для консистентности
6. Service Map помогает понять архитектуру системы
7. Профилирование дополняет трейсинг для deep analysis
8. Правильный sampling экономит деньги и storage


## Модуль 7: Application Performance Monitoring (APM) (30 минут)

### 🎯 Напоминалка

**APM - что мониторим:**

```
┌─────────────────────────────────────┐
│ User Experience                     │
├─────────────────────────────────────┤
│ • Page Load Time                    │
│ • Time to First Byte (TTFB)        │
│ • First Contentful Paint (FCP)     │
│ • Largest Contentful Paint (LCP)   │
│ • Cumulative Layout Shift (CLS)    │
└─────────────────────────────────────┘
         ▼
┌─────────────────────────────────────┐
│ Application Layer                   │
├─────────────────────────────────────┤
│ • Request Rate                      │
│ • Error Rate                        │
│ • Response Time (p50, p95, p99)    │
│ • Throughput                        │
│ • Apdex Score                       │
└─────────────────────────────────────┘
         ▼
┌─────────────────────────────────────┐
│ Code Level                          │
├─────────────────────────────────────┤
│ • Function Execution Time           │
│ • Database Query Performance        │
│ • External API Calls                │
│ • Memory Allocations                │
│ • CPU Profiling                     │
└─────────────────────────────────────┘
```

**Ключевые метрики APM:**

**1. Apdex Score** (Application Performance Index):

```
Apdex = (Satisfied + Tolerating/2) / Total Requests

Satisfied:   Response time ≤ T (target)
Tolerating:  T < Response time ≤ 4T
Frustrated:  Response time > 4T

Пример: T = 0.5s
- 0.3s  → Satisfied
- 1.2s  → Tolerating
- 3.0s  → Frustrated

Score: 0.0-1.0 (1.0 = идеально)
```

**2. Percentiles:**

```
p50 (median)  - 50% requests быстрее
p95           - 95% requests быстрее (хорошо для SLA)
p99           - 99% requests быстрее (tail latency)
p99.9         - для критичных систем

Почему не среднее?
Average: [10ms, 10ms, 10ms, 5000ms] = 1257ms
p95:     [10ms, 10ms, 10ms, 5000ms] = 10ms
```

**3. Service Level Objectives (SLO):**

yaml

```yaml
SLI (Indicator):   Availability = successful_requests / total_requests
SLO (Objective):   99.9% availability
SLA (Agreement):   99.9% or credit

Error Budget:      0.1% = 43 minutes/month downtime
```

**4. Golden Signals для APM:**

yaml

````yaml
Latency:      How long to process requests
Traffic:      How many requests
Errors:       Rate of failed requests  
Saturation:   How "full" your service is
````

**Инструменты APM:**
````
Commercial:
- New Relic
- Datadog APM
- Dynatrace
- AppDynamics

Open Source:
- Elastic APM
- SigNoz
- Grafana Tempo + Prometheus
- Jaeger
````

### 💻 Задание

Настрой полноценный APM с Elastic Stack:

1. **Добавь Elastic APM в docker-compose.yml**:

yaml

```yaml
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    restart: unless-stopped

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch
    restart: unless-stopped

  apm-server:
    image: docker.elastic.co/apm/apm-server:8.11.0
    container_name: apm-server
    ports:
      - "8200:8200"
    command: >
      apm-server -e
        -E apm-server.rum.enabled=true
        -E apm-server.host=0.0.0.0:8200
        -E output.elasticsearch.hosts=["elasticsearch:9200"]
        -E apm-server.kibana.enabled=true
        -E apm-server.kibana.host=kibana:5601
    depends_on:
      - elasticsearch
      - kibana
    restart: unless-stopped

volumes:
  elasticsearch-data:
```

2. **Создай приложение с Elastic APM instrumentation**:

**Python/Flask example:**

python

```python
# app_with_apm.py
from flask import Flask, request, jsonify
from elasticapm.contrib.flask import ElasticAPM
import time
import random
import psycopg2
from redis import Redis

app = Flask(__name__)

# Конфигурация Elastic APM
app.config['ELASTIC_APM'] = {
    'SERVICE_NAME': 'my-flask-app',
    'SERVER_URL': 'http://apm-server:8200',
    'ENVIRONMENT': 'production',
    'CAPTURE_BODY': 'all',
    'TRANSACTION_SAMPLE_RATE': 1.0,  # 100% в dev, 0.1 в prod
}

apm = ElasticAPM(app)
redis_client = Redis(host='redis', port=6379, decode_responses=True)

@app.route('/')
def index():
    return jsonify({
        'status': 'ok',
        'service': 'my-flask-app'
    })

@app.route('/api/users')
def get_users():
    """Endpoint с различными операциями для APM"""
    
    # Custom span для кеша
    with apm.capture_span('check_cache', span_type='cache'):
        cache_key = 'users:all'
        cached = redis_client.get(cache_key)
        
        if cached:
            apm.tag(cache='hit')
            return jsonify({'users': eval(cached), 'source': 'cache'})
        
        apm.tag(cache='miss')
    
    # Database query (автоматически отслеживается)
    users = fetch_users_from_db()
    
    # External API call
    with apm.capture_span('enrich_user_data', span_type='external.http'):
        enriched = enrich_users(users)
    
    # Сохраняем в кеш
    with apm.capture_span('save_to_cache', span_type='cache'):
        redis_client.setex(cache_key, 300, str(enriched))
    
    return jsonify({'users': enriched, 'source': 'database'})

@app.route('/api/slow')
def slow_endpoint():
    """Медленный endpoint для тестирования"""
    # Симуляция медленной операции
    time.sleep(random.uniform(2, 5))
    return jsonify({'message': 'This was slow'})

@app.route('/api/error')
def error_endpoint():
    """Endpoint с ошибками"""
    if random.random() < 0.5:
        raise Exception("Random error occurred!")
    return jsonify({'message': 'Success'})

@app.route('/api/transactions')
def complex_transaction():
    """Сложная транзакция с множеством операций"""
    
    # Шаг 1: Валидация
    with apm.capture_span('validate_request', span_type='app'):
        time.sleep(0.05)
        apm.label(validation='passed')
    
    # Шаг 2: Получение данных
    with apm.capture_span('fetch_data', span_type='db.postgresql'):
        conn = get_db_connection()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM transactions LIMIT 10")
        data = cursor.fetchall()
        cursor.close()
        conn.close()
    
    # Шаг 3: Обработка
    with apm.capture_span('process_data', span_type='app'):
        time.sleep(0.1)
        processed = [{'id': row[0], 'amount': row[1]} for row in data]
    
    # Шаг 4: Сохранение результата
    with apm.capture_span('save_result', span_type='cache'):
        redis_client.setex(f'tx:result:{random.randint(1,1000)}', 
                          600, 
                          str(processed))
    
    return jsonify({'transactions': processed, 'count': len(processed)})

def fetch_users_from_db():
    """Получение пользователей из БД"""
    # Автоматическая инструментация DB queries
    conn = get_db_connection()
    cursor = conn.cursor()
    
    # Медленный запрос
    time.sleep(random.uniform(0.1, 0.3))
    cursor.execute("SELECT id, name, email FROM users")
    
    users = [
        {'id': row[0], 'name': row[1], 'email': row[2]}
        for row in cursor.fetchall()
    ]
    
    cursor.close()
    conn.close()
    return users

def enrich_users(users):
    """Обогащение данных пользователей"""
    import requests
    
    # APM автоматически отследит HTTP requests
    for user in users:
        try:
            # Внешний API вызов
            response = requests.get(
                f'http://external-api:8000/user/{user["id"]}/details',
                timeout=1
            )
            if response.ok:
                user['details'] = response.json()
        except Exception as e:
            apm.capture_exception()
            user['details'] = None
    
    return users

def get_db_connection():
    """Получение подключения к БД"""
    return psycopg2.connect(
        host='postgres',
        database='mydb',
        user='user',
        password='password'
    )

# Custom метрики
@app.before_request
def before_request():
    """Добавляем custom tags к транзакции"""
    elasticapm.set_user_context(
        user_id=request.headers.get('X-User-ID'),
        username=request.headers.get('X-Username')
    )
    
    elasticapm.set_custom_context({
        'user_agent': request.headers.get('User-Agent'),
        'request_id': request.headers.get('X-Request-ID'),
        'api_version': 'v1'
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=False)
```

**Node.js/Express example:**

javascript

```javascript
// app.js
const express = require('express');
const apm = require('elastic-apm-node').start({
  serviceName: 'my-node-app',
  serverUrl: 'http://apm-server:8200',
  environment: 'production',
  captureBody: 'all',
  transactionSampleRate: 1.0
});

const app = express();
const redis = require('redis');
const { Pool } = require('pg');

const redisClient = redis.createClient({
  host: 'redis',
  port: 6379
});

const pgPool = new Pool({
  host: 'postgres',
  database: 'mydb',
  user: 'user',
  password: 'password'
});

app.use(express.json());

app.get('/api/products', async (req, res) => {
  // Custom span
  const span = apm.startSpan('fetch_products', 'db');
  
  try {
    // Check cache
    const cached = await redisClient.get('products:all');
    if (cached) {
      apm.setLabel('cache', 'hit');
      if (span) span.end();
      return res.json(JSON.parse(cached));
    }
    
    apm.setLabel('cache', 'miss');
    
    // Fetch from DB
    const result = await pgPool.query('SELECT * FROM products LIMIT 100');
    const products = result.rows;
    
    // Save to cache
    await redisClient.setex('products:all', 300, JSON.stringify(products));
    
    if (span) span.end();
    res.json(products);
    
  } catch (error) {
    apm.captureError(error);
    if (span) span.end();
    res.status(500).json({ error: error.message });
  }
});

app.get('/api/checkout', async (req, res) => {
  const transaction = apm.currentTransaction;
  
  // Set custom context
  transaction.setCustomContext({
    cart_items: req.query.items,
    payment_method: req.query.payment
  });
  
  // Multiple spans
  const validateSpan = apm.startSpan('validate_cart', 'app');
  await simulateWork(100);
  if (validateSpan) validateSpan.end();
  
  const inventorySpan = apm.startSpan('check_inventory', 'db');
  await simulateWork(200);
  if (inventorySpan) inventorySpan.end();
  
  const paymentSpan = apm.startSpan('process_payment', 'external');
  await simulateWork(500);
  if (paymentSpan) paymentSpan.end();
  
  res.json({ order_id: Math.random().toString(36) });
});

function simulateWork(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

3. **Запусти Elastic APM Stack**:

bash

````bash
# Запусти все сервисы
docker-compose up -d elasticsearch kibana apm-server

# Подожди пока поднимутся (30-60 сек)
docker logs -f kibana

# Запусти приложение
docker-compose up -d app

# Сгенерируй трафик
for i in {1..100}; do
  curl http://localhost:5000/api/users
  curl http://localhost:5000/api/transactions
  curl http://localhost:5000/api/slow
  curl http://localhost:5000/api/error || true
  sleep 0.5
done
````

4. **Открой Kibana и настрой APM**:
````
1. Открой: http://localhost:5601
2. Перейди в: Observability → APM
3. Выбери сервис: my-flask-app
4. Изучи:
   - Transactions: распределение времени ответа
   - Errors: все ошибки с stacktrace
   - Metrics: throughput, latency, error rate
   - Service Map: зависимости между сервисами
````

5. **Создай дашборд для APM метрик в Grafana**:

bash

```bash
# APM метрики доступны через Elasticsearch
# Настрой Elasticsearch data source в Grafana
# URL: http://elasticsearch:9200
# Index: apm-*
```

Queries для Grafana:

json

```json
// Average response time
{
  "query": {
    "bool": {
      "filter": [
        {"range": {"@timestamp": {"gte": "now-15m"}}},
        {"term": {"processor.event": "transaction"}}
      ]
    }
  },
  "aggs": {
    "response_time": {
      "date_histogram": {"field": "@timestamp", "interval": "1m"},
      "aggs": {
        "avg_duration": {"avg": {"field": "transaction.duration.us"}}
      }
    }
  }
}

// Error rate
{
  "query": {
    "bool": {
      "filter": [
        {"range": {"@timestamp": {"gte": "now-15m"}}},
        {"term": {"processor.event": "error"}}
      ]
    }
  },
  "aggs": {
    "errors_over_time": {
      "date_histogram": {"field": "@timestamp", "interval": "1m"},
      "aggs": {
        "error_count": {"value_count": {"field": "error.id"}}
      }
    }
  }
}
```

### 🚀 Бонус (новое)

**1. Настрой Real User Monitoring (RUM)**:

**Frontend instrumentation:**

html

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>My App with RUM</title>
    <script src="https://unpkg.com/@elastic/apm-rum@5.12.0/dist/bundles/elastic-apm-rum.umd.min.js"></script>
    <script>
        // Инициализация RUM
        var apm = window.elasticApm.init({
            serviceName: 'my-frontend',
            serverUrl: 'http://localhost:8200',
            serviceVersion: '1.0.0',
            environment: 'production'
        });
        
        // Custom transaction
        function searchProducts(query) {
            var transaction = apm.startTransaction('Search Products', 'custom');
            
            // Span для API call
            var span = apm.startSpan('API Call', 'external.http');
            
            fetch('/api/search?q=' + query)
                .then(response => response.json())
                .then(data => {
                    if (span) span.end();
                    if (transaction) transaction.end();
                    displayResults(data);
                })
                .catch(error => {
                    apm.captureError(error);
                    if (span) span.end();
                    if (transaction) transaction.end();
                });
        }
        
        // Track user actions
        document.addEventListener('click', function(e) {
            if (e.target.matches('.product-item')) {
                apm.setCustomContext({
                    product_id: e.target.dataset.productId,
                    product_name: e.target.dataset.productName
                });
            }
        });
        
        // Web Vitals
        apm.observe('longtask', function(list) {
            list.getEntries().forEach(function(entry) {
                apm.captureError(new Error('Long Task detected: ' + entry.duration + 'ms'));
            });
        });
    </script>
</head>
<body>
    <h1>My Application</h1>
    <input type="text" id="search" placeholder="Search products...">
    <div id="results"></div>
    
    <script>
        document.getElementById('search').addEventListener('input', function(e) {
            searchProducts(e.target.value);
        });
    </script>
</body>
</html>
```

**2. Создай custom metrics в приложении**:

python

```python
from elasticapm import Client

apm_client = Client({
    'SERVICE_NAME': 'my-app',
    'SERVER_URL': 'http://apm-server:8200'
})

# Custom метрики
class MetricsCollector:
    def __init__(self, apm_client):
        self.apm = apm_client
    
    def track_business_metric(self, metric_name, value, labels=None):
        """Отправка бизнес-метрики"""
        self.apm.gauge(metric_name, value, labels=labels)
    
    def track_cart_value(self, user_id, cart_total):
        """Трекинг стоимости корзины"""
        self.apm.gauge('cart.total', cart_total, labels={
            'user_id': user_id
        })
    
    def track_conversion(self, funnel_step, success):
        """Трекинг конверсии"""
        self.apm.counter('conversion.steps', labels={
            'step': funnel_step,
            'success': str(success)
        })

metrics = MetricsCollector(apm_client)

@app.route('/api/add-to-cart')
def add_to_cart():
    # ... добавление в корзину
    
    # Трекинг метрики
    metrics.track_cart_value(
        user_id=request.headers.get('X-User-ID'),
        cart_total=calculate_cart_total()
    )
    
    metrics.track_conversion('add_to_cart', True)
    
    return jsonify({'success': True})
```

**3. Настрой SLO мониторинг**:

python

```python
# slo_monitor.py
from elasticapm import Client
import time

class SLOMonitor:
    """Мониторинг SLO/SLA"""
    
    def __init__(self, apm_client):
        self.apm = apm_client
        self.slo_target = 0.999  # 99.9%
        self.latency_target_ms = 500
    
    def check_availability_slo(self):
        """Проверка SLO по доступности"""
        total_requests = self.get_total_requests()
        successful_requests = self.get_successful_requests()
        
        availability = successful_requests / total_requests if total_requests > 0 else 1.0
        error_budget_remaining = (self.slo_target - availability) * total_requests
        
        self.apm.gauge('slo.availability', availability)
        self.apm.gauge('slo.error_budget_remaining', error_budget_remaining)
        
        if availability < self.slo_target:
            self.apm.capture_message(
                f'SLO violation: Availability {availability:.4f} below target {self.slo_target}',
                level='warning'
            )
        
        return {
            'availability': availability,
            'target': self.slo_target,
            'error_budget': error_budget_remaining,
            'in_compliance': availability >= self.slo_target
        }
    
    def check_latency_slo(self):
        """Проверка SLO по латентности"""
        p95_latency = self.get_p95_latency()
        
        self.apm.gauge('slo.latency_p95', p95_latency)
        
        if p95_latency > self.latency_target_ms:
            self.apm.capture_message(
                f'SLO violation: p95 latency {p95_latency}ms above target {self.latency_target_ms}ms',
                level='warning'
            )
        
        return {
            'p95_latency': p95_latency,
            'target': self.latency_target_ms,
            'in_compliance': p95_latency <= self.latency_target_ms
        }

# Периодическая проверка SLO
def monitor_slo():
    monitor = SLOMonitor(apm_client)
    
    while True:
        availability_status = monitor.check_availability_slo()
        latency_status = monitor.check_latency_slo()
        
        print(f"Availability SLO: {availability_status}")
        print(f"Latency SLO: {latency_status}")
        
        time.sleep(60)  # Каждую минуту
```

**4. Профилирование производительности**:

python

````python
from elasticapm.contrib.flask import ElasticAPM
import cProfile
import pstats
from io import StringIO

@app.route('/api/profile')
def profile_endpoint():
    """Endpoint с профилированием"""
    
    # CPU profiling
    profiler = cProfile.Profile()
    profiler.enable()
    
    # Код для профилирования
    result = expensive_operation()
    
    profiler.disable()
    
    # Сохраняем результаты в APM
    s = StringIO()
    stats = pstats.Stats(profiler, stream=s)
    stats.sort_stats('cumulative')
    stats.print_stats(20)
    
    apm.capture_message(
        'Profile results',
        custom={'profile': s.getvalue()}
    )
    
    return result

# Memory profiling
from memory_profiler import profile

@profile  # Декоратор для memory profiling
def memory_intensive_operation():
    large_list = [i for i in range(1000000)]
    return sum(large_list)
````

---

## Модуль 8: Синтетический мониторинг и Uptime (25 минут)

### 🎯 Напоминалка

**Синтетический мониторинг vs Real User Monitoring:**
````
Synthetic Monitoring:
✓ Проактивный (обнаруживает проблемы до пользователей)
✓ Контролируемые условия
✓ Регулярные проверки 24/7
✓ Географическое распределение
✗ Не отражает реальный user experience

Real User Monitoring (RUM):
✓ Реальный user experience
✓ Реальные устройства и сети
✓ Бизнес-метрики
✗ Реактивный (пользователи уже пострадали)
````

**Типы синтетических проверок:**

yaml

````yaml
HTTP/HTTPS Check:
  - Status code
  - Response time
  - SSL certificate
  - Response body contains text

TCP Check:
  - Port availability
  - Connection time

DNS Check:
  - DNS resolution time
  - Correct IP returned

Browser Check (Headless):
  - Full page load
  - JavaScript execution
  - Form submission
  - Multi-step transactions
````

**Локации для проверок:**
````
Multiple Geographic Locations:
- North America (US-East, US-West)
- Europe (London, Frankfurt)
- Asia (Tokyo, Singapore)
- South America (São Paulo)

Цель: Обнаружить региональные проблемы
````

### 💻 Задание

Настрой синтетический мониторинг с Blackbox Exporter и Uptime Kuma:

1. **Добавь Blackbox Exporter в docker-compose.yml**:

yaml

```yaml
  blackbox-exporter:
    image: prom/blackbox-exporter:latest
    container_name: blackbox-exporter
    ports:
      - "9115:9115"
    volumes:
      - ./blackbox.yml:/etc/blackbox_exporter/config.yml
    command:
      - '--config.file=/etc/blackbox_exporter/config.yml'
    restart: unless-stopped
```

2. **Создай blackbox.yml**:

yaml

```yaml
modules:
  # HTTP 2xx check
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
      valid_status_codes: []  # defaults to 2xx
      method: GET
      preferred_ip_protocol: "ip4"
      follow_redirects: true
      fail_if_ssl: false
      fail_if_not_ssl: false

  # HTTP check with POST
  http_post_2xx:
    prober: http
    http:
      method: POST
      headers:
        Content-Type: application/json
      body: '{"key": "value"}'

  # HTTP check с проверкой содержимого
  http_content_check:
    prober: http
    http:
      fail_if_body_not_matches_regexp:
        - "Welcome"
        - "Status: OK"
      fail_if_body_matches_regexp:
        - "Error"
        - "Exception"

  # HTTPS с проверкой SSL
  https_ssl_check:
    prober: http
    timeout: 5s
    http:
      valid_status_codes: [200]
      fail_if_ssl: false
      fail_if_not_ssl: true
      tls_config:
        insecure_skip_verify: false

  # TCP check
  tcp_connect:
    prober: tcp
    timeout: 5s

  # ICMP (ping) check
  icmp:
    prober: icmp
    timeout: 5s
    icmp:
      preferred_ip_protocol: "ip4"

  # DNS check
  dns_check:
    prober: dns
    timeout: 5s
    dns:
      query_name: "example.com"
      query_type: "A"

  # SSH check
  ssh_banner:
    prober: tcp
    timeout: 5s
    tcp:
      query_response:
        - expect: "^SSH-2.0-"

  # PostgreSQL check
  postgres_check:
    prober: tcp
    timeout: 5s
    tcp:
      query_response:
        - send: "\x00\x00\x00\x08\x04\xd2\x16\x2f"
```

3. **Обнови prometheus.yml** для Blackbox:

yaml

```yaml
scrape_configs:
  # ... существующие jobs

  # HTTP endpoints
  - job_name: 'blackbox-http'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://example.com
          - https://api.example.com
          - http://localhost:5000
          - http://localhost:3000
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115

  # TCP ports
  - job_name: 'blackbox-tcp'
    metrics_path: /probe
    params:
      module: [tcp_connect]
    static_configs:
      - targets:
          - localhost:5432  # PostgreSQL
          - localhost:6379  # Redis
          - localhost:9090  # Prometheus
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115

  # ICMP (ping)
  - job_name: 'blackbox-icmp'
    metrics_path: /probe
    params:
      module: [icmp]
    static_configs:
      - targets:
          - 8.8.8.8        # Google DNS
          - 1.1.1.1        # Cloudflare DNS
          - example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115

```

4. **Создай алерты для синтетических проверок**:
```yaml
# synthetic_alerts.yml
groups:
  - name: blackbox_alerts
    rules:
      # Endpoint недоступен
      - alert: EndpointDown
        expr: probe_success == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Endpoint {{ $labels.instance }} is down"
          description: "{{ $labels.instance }} has been down for more than 5 minutes"
          impact: "Service unavailable for users"

      # Медленный ответ
      - alert: SlowResponse
        expr: probe_duration_seconds > 3
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Slow response from {{ $labels.instance }}"
          description: "Response time is {{ $value }}s (threshold: 3s)"

      # SSL сертификат истекает
      - alert: SSLCertExpiringSoon
        expr: (probe_ssl_earliest_cert_expiry - time()) / 86400 < 30
        for: 1h
        labels:
          severity: warning
        annotations:
          summary: "SSL certificate expiring soon for {{ $labels.instance }}"
          description: "SSL certificate expires in {{ $value }} days"
          action: "Renew SSL certificate"

      # SSL сертификат истек
      - alert: SSLCertExpired
        expr: probe_ssl_earliest_cert_expiry - time() <= 0
        labels:
          severity: critical
        annotations:
          summary: "SSL certificate expired for {{ $labels.instance }}"
          description: "SSL certificate has expired"
          impact: "HTTPS connections will fail"

      # HTTP status code не 2xx
      - alert: HTTPStatusCode5xx
        expr: probe_http_status_code >= 500
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "HTTP 5xx error on {{ $labels.instance }}"
          description: "Status code: {{ $value }}"

      - alert: HTTPStatusCode4xx
        expr: probe_http_status_code >= 400 and probe_http_status_code < 500
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "HTTP 4xx error on {{ $labels.instance }}"
          description: "Status code: {{ $value }}"
```

5. **Добавь Uptime Kuma** (красивый UI для uptime мониторинга):
```yaml
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    ports:
      - "3001:3001"
    volumes:
      - uptime-kuma-data:/app/data
    restart: unless-stopped

volumes:
  uptime-kuma-data:
```

6. **Запусти и настрой**:
```bash
# Запусти сервисы
docker-compose up -d blackbox-exporter uptime-kuma

# Проверь Blackbox
curl "http://localhost:9115/probe?module=http_2xx&target=https://example.com"

# Настрой Uptime Kuma
# Открой: http://localhost:3001
# Создай аккаунт
# Добавь мониторы:
#   - HTTP(s) для веб-сайтов
#   - TCP для портов
#   - Ping для серверов
```

7. **Создай дашборд для синтетического мониторинга в Grafana**:

Импортируй готовый дашборд: ID 7587 (Prometheus Blackbox Exporter)

Или создай свой с панелями:
```

Panel 1: Uptime % Query: avg_over_time(probe_success[24h]) * 100

Panel 2: Response Time Query: probe_duration_seconds

Panel 3: SSL Certificate Days Left Query: (probe_ssl_earliest_cert_expiry - time()) / 86400

Panel 4: HTTP Status Codes Query: probe_http_status_code

Panel 5: Availability Map Type: Status History Query: probe_success

```

### 🚀 Бонус (новое)

**1. Создай Multi-Step Browser Check с Playwright**:
```python
# synthetic_browser_check.py
from playwright.sync_api import sync_playwright
from prometheus_client import Gauge, Counter, start_http_server
import time

# Метрики
check_duration = Gauge('synthetic_check_duration_seconds', 
                       'Duration of synthetic check', 
                       ['check_name', 'step'])
check_success = Gauge('synthetic_check_success', 
                     'Success status of synthetic check',
                     ['check_name'])
check_errors = Counter('synthetic_check_errors_total',
                      'Total errors in synthetic checks',
                      ['check_name', 'error_type'])

def run_login_flow_check():
    """Multi-step синтетическая проверка: логин и покупка"""
    
    check_name = 'ecommerce_purchase_flow'
    start_time = time.time()
    
    try:
        with sync_playwright() as p:
            browser = p.chromium.launch(headless=True)
            page = browser.new_page()
            
            # Step 1: Загрузка главной страницы
            step_start = time.time()
            page.goto('https://example-shop.com')
            check_duration.labels(check_name=check_name, step='homepage').set(
                time.time() - step_start
            )
            
            # Step 2: Поиск товара
            step_start = time.time()
            page.fill('input[name="search"]', 'laptop')
            page.click('button[type="submit"]')
            page.wait_for_selector('.product-list')
            check_duration.labels(check_name=check_name, step='search').set(
                time.time() - step_start
            )
            
            # Step 3: Добавление в корзину
            step_start = time.time()
            page.click('.product-item:first-child .add-to-cart')
            page.wait_for_selector('.cart-notification')
            check_duration.labels(check_name=check_name, step='add_to_cart').set(
                time.time() - step_start
            )
            
            # Step 4: Checkout
            step_start = time.time()
            page.click('a[href="/cart"]')
            page.wait_for_selector('.checkout-button')
            page.click('.checkout-button')
            page.wait_for_url('**/checkout')
            check_duration.labels(check_name=check_name, step='checkout').set(
                time.time() - step_start
            )
            
            # Step 5: Заполнение формы
            step_start = time.time()
            page.fill('input[name="email"]', 'test@example.com')
            page.fill('input[name="card_number"]', '4242424242424242')
            page.fill('input[name="exp_date"]', '12/25')
            page.fill('input[name="cvv"]', '123')
            check_duration.labels(check_name=check_name, step='fill_form').set(
                time.time() - step_start
            )
            
            # Step 6: Submit и проверка успеха
            step_start = time.time()
            page.click('button[type="submit"]')
            page.wait_for_selector('.order-success', timeout=10000)
            check_duration.labels(check_name=check_name, step='submit').set(
                time.time() - step_start
            )
            
            browser.close()
            
            # Успех
            check_success.labels(check_name=check_name).set(1)
            
            total_duration = time.time() - start_time
            print(f"✓ Check {check_name} passed in {total_duration:.2f}s")
            
    except Exception as e:
        check_success.labels(check_name=check_name).set(0)
        check_errors.labels(
            check_name=check_name,
            error_type=type(e).__name__
        ).inc()
        print(f"✗ Check {check_name} failed: {e}")

def run_api_workflow_check():
    """API workflow проверка"""
    import requests
    
    check_name = 'api_workflow'
    
    try:
        # Step 1: Get auth token
        step_start = time.time()
        auth_response = requests.post('https://api.example.com/auth', json={
            'username': 'test',
            'password': 'test123'
        }, timeout=5)
        check_duration.labels(check_name=check_name, step='auth').set(
            time.time() - step_start
        )
        
        if auth_response.status_code != 200:
            raise Exception(f"Auth failed: {auth_response.status_code}")
        
        token = auth_response.json()['token']
        
        # Step 2: Create resource
        step_start = time.time()
        create_response = requests.post(
            'https://api.example.com/resources',
            headers={'Authorization': f'Bearer {token}'},
            json={'name': 'test-resource'},
            timeout=5
        )
        check_duration.labels(check_name=check_name, step='create').set(
            time.time() - step_start
        )
        
        resource_id = create_response.json()['id']
        
        # Step 3: Get resource
        step_start = time.time()
        get_response = requests.get(
            f'https://api.example.com/resources/{resource_id}',
            headers={'Authorization': f'Bearer {token}'},
            timeout=5
        )
        check_duration.labels(check_name=check_name, step='get').set(
            time.time() - step_start
        )
        
        # Step 4: Delete resource
        step_start = time.time()
        delete_response = requests.delete(
            f'https://api.example.com/resources/{resource_id}',
            headers={'Authorization': f'Bearer {token}'},
            timeout=5
        )
        check_duration.labels(check_name=check_name, step='delete').set(
            time.time() - step_start
        )
        
        check_success.labels(check_name=check_name).set(1)
        print(f"✓ API workflow check passed")
        
    except Exception as e:
        check_success.labels(check_name=check_name).set(0)
        check_errors.labels(
            check_name=check_name,
            error_type=type(e).__name__
        ).inc()
        print(f"✗ API workflow check failed: {e}")

if __name__ == '__main__':
    # Запусти Prometheus metrics server
    start_http_server(8000)
    print("Metrics available at http://localhost:8000")
    
    # Запускай проверки каждые 5 минут
    while True:
        print(f"\n{'='*50}")
        print(f"Running synthetic checks at {time.strftime('%Y-%m-%d %H:%M:%S')}")
        print(f"{'='*50}")
        
        run_login_flow_check()
        run_api_workflow_check()
        
        time.sleep(300)  # 5 минут
```

**2. Создай Geographic Distributed Monitoring**:
```yaml
# docker-compose для разных регионов
# us-east.docker-compose.yml
version: '3.8'

services:
  blackbox-us-east:
    image: prom/blackbox-exporter:latest
    container_name: blackbox-us-east
    ports:
      - "9116:9115"
    volumes:
      - ./blackbox.yml:/etc/blackbox_exporter/config.yml
    environment:
      - LOCATION=us-east
    restart: unless-stopped
```

Prometheus config для нескольких локаций:
```yaml
scrape_configs:
  - job_name: 'blackbox-us-east'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: instance
        replacement: example.com
      - target_label: region
        replacement: us-east
      - target_label: __address__
        replacement: blackbox-us-east:9115

  - job_name: 'blackbox-eu-west'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: instance
        replacement: example.com
      - target_label: region
        replacement: eu-west
      - target_label: __address__
        replacement: blackbox-eu-west:9115
```

**3. Настрой Status Page** с использованием Cachet:
```yaml
  cachet:
    image: cachethq/docker:latest
    container_name: cachet
    ports:
      - "8001:8000"
    environment:
      - DB_DRIVER=sqlite
      - APP_KEY=base64:yourapplicationkey
      - APP_URL=http://localhost:8001
    volumes:
      - cachet-data:/var/www/html/database
    restart: unless-stopped

volumes:
  cachet-data:
```

Или используй **Upptime** (GitHub-based):
```yaml
# .github/workflows/upptime.yml
name: Upptime CI
on:
  schedule:
    - cron: "*/5 * * * *"
  workflow_dispatch:

jobs:
  release:
    name: Check status
    runs-on: ubuntu-latest
    steps:
      - uses: upptime/upptime@v1.28.0
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

`.upptimerc.yml`:
```yaml
owner: your-username
repo: upptime

sites:
  - name: Website
    url: https://example.com
    maxResponseTime: 5000
  - name: API
    url: https://api.example.com
    maxResponseTime: 3000
  - name: Blog
    url: https://blog.example.com

status-website:
  cname: status.example.com
  name: Status Page
  introTitle: "Service Status"
  introMessage: Real-time status and uptime monitoring
```

---
## Модуль 9: SLI/SLO/SLA и Error Budget - Site Reliability Engineering (40 минут)

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
````

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
````

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
            m
```

Продолжи создание документа

14:06

+= f"| {slo_data['actual']}% " md += f"| {slo_data['status']} " md += f"| {slo_data['error_budget_consumed']}% " md += f"| {slo_data['error_budget_remaining']}% " md += f"| {slo_data['downtime_minutes']:.1f} min |\n"

```
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
```

if **name** == "**main**": import argparse

```
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

````

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
```

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
````
---
## Модуль 10: Infrastructure as Code для мониторинга (35 минут)

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

## Итоги модуля 8

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



## Модуль 11: Kubernetes Monitoring - мониторинг контейнеров и оркестрации (45 минут)

### 🎯 Напоминалка

**Kubernetes архитектура и компоненты:**

```
┌─────────────────────────────────────────┐
│           Control Plane                 │
│  ┌──────────┐  ┌──────────┐            │
│  │   API    │  │  etcd    │            │
│  │  Server  │  │          │            │
│  └──────────┘  └──────────┘            │
│  ┌──────────┐  ┌──────────┐            │
│  │Scheduler │  │Controller│            │
│  │          │  │ Manager  │            │
│  └──────────┘  └──────────┘            │
└─────────────────────────────────────────┘
              │
    ┌─────────┴─────────┐
    │                   │
┌───▼────┐         ┌────▼───┐
│ Node 1 │         │ Node 2 │
│        │         │        │
│ ┌────┐ │         │ ┌────┐ │
│ │Pod │ │         │ │Pod │ │
│ │    │ │         │ │    │ │
│ └────┘ │         │ └────┘ │
│ ┌────┐ │         │ ┌────┐ │
│ │Pod │ │         │ │Pod │ │
│ └────┘ │         │ └────┘ │
│        │         │        │
│kubelet │         │kubelet │
└────────┘         └────────┘
```

**Уровни мониторинга K8s:**

```
┌──────────────────────────────────────┐
│  Application Level                   │  - Бизнес метрики
│  (ваше приложение)                   │  - Custom metrics
├──────────────────────────────────────┤
│  Container Level                     │  - CPU, Memory, Network
│  (Docker/containerd)                 │  - Restart count
├──────────────────────────────────────┤
│  Pod Level                           │  - Pod status
│  (K8s workload)                      │  - Resource limits
├──────────────────────────────────────┤
│  Node Level                          │  - Node resources
│  (Worker nodes)                      │  - Disk, Network
├──────────────────────────────────────┤
│  Cluster Level                       │  - API server health
│  (Control plane)                     │  - etcd, scheduler
└──────────────────────────────────────┘
```

**Ключевые метрики K8s:**

**Cluster metrics:**

```
- Общее количество nodes
- Nodes ready/not ready
- Total CPU/Memory capacity
- Total CPU/Memory usage
- API server request rate
- API server latency
- etcd latency
- Scheduler latency
```

**Node metrics:**

```
- CPU usage/limits
- Memory usage/limits
- Disk usage/IOPS
- Network traffic
- Pod count per node
- Node conditions (Ready, DiskPressure, MemoryPressure)
```

**Pod metrics:**

```
- CPU usage/requests/limits
- Memory usage/requests/limits
- Restart count
- Pod phase (Pending, Running, Failed, Succeeded)
- Container state
- Network I/O
```

**Container metrics:**

```
- CPU usage
- Memory usage (RSS, cache, swap)
- Disk I/O
- Network I/O
- OOM kills
```

**Важные K8s состояния:**

```
Pod Phases:
- Pending     - Ждет scheduling
- Running     - Запущен на node
- Succeeded   - Все контейнеры успешно завершились
- Failed      - Хотя бы один контейнер failed
- Unknown     - Состояние неизвестно

Container States:
- Waiting     - Ждет запуска
- Running     - Выполняется
- Terminated  - Завершен

Node Conditions:
- Ready              - Node готов принимать pods
- MemoryPressure     - Мало памяти
- DiskPressure       - Мало места на диске
- PIDPressure        - Много процессов
- NetworkUnavailable - Проблемы с сетью
```

**Prometheus в Kubernetes:**

```
┌─────────────────────────────────────────┐
│         Prometheus Operator             │
│                                         │
│  Автоматизирует:                        │
│  - Deployment Prometheus                │
│  - Service Discovery                    │
│  - Scrape configuration                 │
│  - Alert rules                          │
│  - ServiceMonitor CRDs                  │
└─────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│      Prometheus Server(s)               │
│                                         │
│  Собирает метрики с:                    │
│  - kubelet (cAdvisor)                   │
│  - API server                           │
│  - Node exporters                       │
│  - Application pods                     │
└─────────────────────────────────────────┘
```

**Kube-state-metrics vs Metrics Server:**

```
Metrics Server:
- Базовые CPU/Memory метрики
- Для Horizontal Pod Autoscaler (HPA)
- Для kubectl top
- Real-time данные
- Не хранит историю

Kube-state-metrics:
- Метрики о K8s объектах (Deployments, Pods, etc)
- Состояние кластера
- Для мониторинга и alerting
- Экспортирует в Prometheus формате
- Дополняет Metrics Server
```

**ServiceMonitor и PodMonitor:**

yaml

```yaml
# ServiceMonitor - автоматический scraping через Service
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app
  labels:
    team: backend
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics

# PodMonitor - прямой scraping pods
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: my-app-pods
spec:
  selector:
    matchLabels:
      app: my-app
  podMetricsEndpoints:
  - port: metrics
    interval: 30s
```

**Resource Requests и Limits:**

yaml

```yaml
resources:
  requests:
    cpu: 100m        # Гарантированно получит
    memory: 128Mi
  limits:
    cpu: 500m        # Максимум может использовать
    memory: 512Mi    # OOM kill если превысит

QoS Classes:
1. Guaranteed  - requests == limits (приоритет highest)
2. Burstable   - requests < limits
3. BestEffort  - нет requests/limits (приоритет lowest)
```

**Horizontal Pod Autoscaler (HPA):**

yaml

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Target 70% CPU
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
```

**Vertical Pod Autoscaler (VPA):**

yaml

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"  # Auto, Recreate, Initial, Off
  resourcePolicy:
    containerPolicies:
    - containerName: app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 2Gi
```

**Важные PromQL запросы для K8s:**

promql

````promql
# CPU
## Node CPU usage
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

## Pod CPU usage
sum(rate(container_cpu_usage_seconds_total{pod!=""}[5m])) by (pod, namespace)

## CPU throttling
rate(container_cpu_cfs_throttled_seconds_total[5m]) > 0

# Memory
## Pod memory usage
sum(container_memory_working_set_bytes{pod!=""}) by (pod, namespace)

## Memory usage vs limit
sum(container_memory_working_set_bytes{pod!=""}) by (pod)
/
sum(container_spec_memory_limit_bytes{pod!=""}) by (pod) * 100

## OOM kills
rate(container_oom_events_total[5m]) > 0

# Disk
## Disk usage per node
(1 - (node_filesystem_avail_bytes{mountpoint="/"} 
/ node_filesystem_size_bytes{mountpoint="/"})) * 100

## Pod disk I/O
rate(container_fs_reads_bytes_total[5m])
rate(container_fs_writes_bytes_total[5m])

# Network
## Pod network traffic
rate(container_network_receive_bytes_total[5m])
rate(container_network_transmit_bytes_total[5m])

## Network errors
rate(container_network_receive_errors_total[5m])
rate(container_network_transmit_errors_total[5m])

# Kubernetes objects
## Pods not ready
kube_pod_status_phase{phase!~"Running|Succeeded"} > 0

## Deployment replicas mismatch
kube_deployment_spec_replicas != kube_deployment_status_replicas_available

## Pod restarts
rate(kube_pod_container_status_restarts_total[15m]) > 0

## Failed pods
kube_pod_status_phase{phase="Failed"} > 0

## Pending pods (долго)
kube_pod_status_phase{phase="Pending"} > 0

# Resources
## CPU requests vs limits
sum(kube_pod_container_resource_requests{resource="cpu"})
/
sum(kube_pod_container_resource_limits{resource="cpu"})

## Memory requests vs limits
sum(kube_pod_container_resource_requests{resource="memory"})
/
sum(kube_pod_container_resource_limits{resource="memory"})

## Node capacity vs allocatable
sum(kube_node_status_capacity{resource="cpu"})
sum(kube_node_status_allocatable{resource="cpu"})

# API Server
## Request rate
rate(apiserver_request_total[5m])

## Request latency
histogram_quantile(0.99, 
  rate(apiserver_request_duration_seconds_bucket[5m])
)

## Request errors
rate(apiserver_request_total{code=~"5.."}[5m])

# etcd
## Leader changes
rate(etcd_server_leader_changes_seen_total[5m])

## Proposal failures
rate(etcd_server_proposals_failed_total[5m])

## DB size
etcd_mvcc_db_total_size_in_bytes

# Scheduler
## Scheduling latency
histogram_quantile(0.99,
  rate(scheduler_scheduling_duration_seconds_bucket[5m])
)

## Pending pods in queue
scheduler_pending_pods

# HPA
## Current replicas vs desired
kube_horizontalpodautoscaler_status_current_replicas
vs
kube_horizontalpodautoscaler_status_desired_replicas

## HPA metric value
kube_horizontalpodautoscaler_status_current_metrics_value
````

**Лучшие практики K8s мониторинга:**
````
1. ✅ Всегда устанавливай resource requests/limits
2. ✅ Мониторь все уровни: cluster → node → pod → container
3. ✅ Используй ServiceMonitor для auto-discovery
4. ✅ Настрой alerting на Pod restarts и OOM kills
5. ✅ Отслеживай CPU throttling
6. ✅ Мониторь kube-state-metrics для объектов K8s
7. ✅ Используй HPA для auto-scaling
8. ✅ Настрой PodDisruptionBudget для availability
9. ✅ Мониторь control plane компоненты
10. ✅ Используй namespace для изоляции и multi-tenancy
11. ✅ Экспортируй метрики приложений через /metrics endpoint
12. ✅ Используй labels для организации и filtering
````

**Namespace isolation:**

yaml

```yaml
# ResourceQuota - ограничения на namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: production
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    persistentvolumeclaims: "10"
    pods: "50"

# LimitRange - default limits для pods
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: production
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container
```

**PodDisruptionBudget (для HA):**

yaml

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 2  # или maxUnavailable: 1
  selector:
    matchLabels:
      app: my-app
```

**Liveness и Readiness проbes:**

yaml

````yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 2

# Startup probe (для медленных приложений)
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 0
  periodSeconds: 10
  failureThreshold: 30  # 300s total
````

**Типичные проблемы и их диагностика:**
```
Проблема: Pod постоянно restarts
Диагностика:
- kubectl describe pod <pod-name>
- kubectl logs <pod-name> --previous
- Проверь liveness probe
- Проверь OOM kills: kube_pod_container_status_terminated_reason{reason="OOMKilled"}

Проблема: High CPU throttling
Диагностика:
- rate(container_cpu_cfs_throttled_seconds_total[5m])
- Увеличь CPU limits или оптимизируй код

Проблема: Pod Pending долго
Диагностика:
- kubectl describe pod <pod-name>
- Проверь Events
- Причины: insufficient resources, node selector mismatch, PVC issues

Проблема: High memory usage
Диагностика:
- container_memory_working_set_bytes
- Проверь memory leaks
- Настрой VPA для автоматической оптимизации

Проблема: Slow API requests
Диагностика:
- apiserver_request_duration_seconds
- Проверь etcd latency
- Масштабируй API server replicas
```

**Grafana dashboards для K8s:**
````
Рекомендуемые community dashboards:

1. Kubernetes Cluster Monitoring (315)
   - Общий обзор кластера
   - Nodes, Pods, CPU, Memory

2. Kubernetes / Compute Resources / Cluster (7249)
   - Resource usage по namespace
   - Requests vs Limits

3. Kubernetes / Compute Resources / Namespace (Pods) (7630)
   - Детальная информация по pods

4. Node Exporter Full (1860)
   - Детали по nodes

5. Kubernetes apiserver (12006)
   - API server metrics

Импорт: Grafana → Dashboards → Import → ID
````

### 💻 Задание

Настрой полноценный мониторинг Kubernetes кластера:

1. **Создай локальный K8s кластер с kind**:

`kind-config.yaml`:

yaml

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: monitoring-cluster
nodes:
  - role: control-plane
    image: kindest/node:v1.29.0
    extraPortMappings:
      - containerPort: 30000
        hostPort: 9090
        protocol: TCP
      - containerPort: 30001
        hostPort: 3000
        protocol: TCP
      - containerPort: 30002
        hostPort: 16686
        protocol: TCP
  - role: worker
    image: kindest/node:v1.29.0
  - role: worker
    image: kindest/node:v1.29.0
```

Создай кластер:

bash

```bash
# Установи kind если нет
# Mac: brew install kind
# Linux: curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64

# Создай кластер
kind create cluster --config kind-config.yaml

# Проверь
kubectl cluster-info
kubectl get nodes
```

2. **Установи kube-prometheus-stack (Prometheus Operator)**:

bash

```bash
# Добавь Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Создай namespace
kubectl create namespace monitoring

# Установи kube-prometheus-stack
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.service.type=NodePort \
  --set prometheus.service.nodePort=30000 \
  --set grafana.service.type=NodePort \
  --set grafana.service.nodePort=30001 \
  --set grafana.adminPassword=admin

# Проверь установку
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

3. **Создай demo приложение с метриками**:

`k8s-manifests/demo-app-deployment.yaml`:

yaml

````yaml
apiVersion: v1
kind: Namespace
metadata:
  name: demo-app

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
  namespace: demo-app
  labels:
    app: demo-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: app
        image: quay.io/brancz/prometheus-example-app:v0.5.0
        ports:
        - containerPort: 8080
          name: metrics
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 3

---
apiVersion: v1
kind: Service
metadata:
  name: demo-app
  namespace: demo-app
  labels:
    app: demo-app
spec:
  selector:
    app: demo-app
  ports:
  - port: 8080
    targetPort: 8080
    name: metrics
  type: ClusterIP

---
# ServiceMonitor для автоматического scraping
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: demo-app
  namespace: demo-app
  labels:
    app: demo-app
spec:
  selector:
    matchLabels:
      app: demo-app
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics

---
# HPA для автомасштабирования
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: demo-app-hpa
  namespace: demo-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: demo-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60

---
# PodDisruptionBudget для HA
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: demo-app-pdb
  namespace: demo-app
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: demo-app

---
# ResourceQuota для namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: demo-app-quota
  namespace: demo-app
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "20"
````

Примени:

bash

```bash
kubectl apply -f k8s-manifests/demo-app-deployment.yaml

# Проверь
kubectl get all -n demo-app
kubectl get servicemonitor -n demo-app
kubectl get hpa -n demo-app
```

4. **Создай PrometheusRule для alerting**:

`k8s-manifests/prometheus-rules.yaml`:

yaml

````yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: kubernetes-alerts
  namespace: monitoring
  labels:
    prometheus: kube-prometheus
spec:
  groups:
  - name: kubernetes.rules
    interval: 30s
    rules:
    # Pod alerts
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
      for: 5m
      labels:
        severity: critical
        component: pod
      annotations:
        summary: "Pod {{ $labels.namespace }}/{{ $labels.pod }} is crash looping"
        description: "Pod has restarted {{ $value }} times in the last 15 minutes"
        dashboard: "http://localhost:3000/d/kubernetes-pods"

    - alert: PodNotReady
      expr: |
        sum by (namespace, pod) (
          kube_pod_status_phase{phase!~"Running|Succeeded"}
        ) > 0
      for: 10m
      labels:
        severity: warning
        component: pod
      annotations:
        summary: "Pod {{ $labels.namespace }}/{{ $labels.pod }} not ready"
        description: "Pod has been in {{ $labels.phase }} state for more than 10 minutes"

    - alert: PodOOMKilled
      expr: |
        sum by (namespace, pod) (
          rate(kube_pod_container_status_terminated_reason{reason="OOMKilled"}[5m])
        ) > 0
      for: 1m
      labels:
        severity: critical
        component: pod
      annotations:
        summary: "Pod {{ $labels.namespace }}/{{ $labels.pod }} OOMKilled"
        description: "Pod was killed due to out of memory"
        runbook: "Increase memory limits or fix memory leak"

    # Container alerts
    - alert: ContainerCPUThrottling
      expr: |
        rate(container_cpu_cfs_throttled_seconds_total[5m]) > 0.5
      for: 10m
      labels:
        severity: warning
        component: container
      annotations:
        summary: "Container {{ $labels.namespace }}/{{ $labels.pod }}/{{ $labels.container }} CPU throttling"
        description: "Container is being throttled {{ $value | humanizePercentage }}"
        runbook: "Increase CPU limits"

    - alert: ContainerHighMemoryUsage
      expr: |
        (
          sum by (namespace, pod, container) (container_memory_working_set_bytes)
          /
          sum by (namespace, pod, container) (container_spec_memory_limit_bytes)
        ) > 0.9
      for: 5m
      labels:
        severity: warning
        component: container
      annotations:
        summary: "Container {{ $labels.namespace }}/{{ $labels.pod }}/{{ $labels.container }} high memory"
        description: "Memory usage is {{ $value | humanizePercentage }}"

    # Deployment alerts
    - alert: DeploymentReplicasMismatch
      expr: |
        kube_deployment_spec_replicas != kube_deployment_status_replicas_available
      for: 10m
      labels:
        severity: warning
        component: deployment
      annotations:
        summary: "Deployment {{ $labels.namespace }}/{{ $labels.deployment }} replicas mismatch"
        description: "Desired: {{ $value }}, Available: {{ $labels.replicas_available }}"

    # Node alerts
    - alert: NodeNotReady
      expr: kube_node_status_condition{condition="Ready",status="true"} == 0
      for: 5m
      labels:
        severity: critical
        component: node
      annotations:
        summary: "Node {{ $labels.node }} not ready"
        description: "Node has been unready for more than 5 minutes"

    - alert: NodeHighCPUUsage
      expr: |
        100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
      for: 10m
      labels:
        severity: warning
        component: node
      annotations:
        summary: "Node {{ $labels.instance }} high CPU"
        description: "CPU usage is {{ $value | humanize }}%"

    - alert: NodeHighMemoryUsage
      expr: |
        (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
      for: 5m
      labels:
        severity: warning
        component: node
      annotations:
        summary: "Node {{ $labels.instance }} high memory"
        description: "Memory usage is {{ $value | humanize }}%"

    - alert: NodeDiskPressure
      expr: kube_node_status_condition{condition="DiskPressure",status="true"} == 1
      for: 5m
      labels:
        severity: critical
        component: node
      annotations:
        summary: "Node {{ $labels.node }} disk pressure"
        description: "Node is experiencing disk pressure"

    # HPA alerts
    - alert: HPAMaxedOut
      expr: |
        kube_horizontalpodautoscaler_status_current_replicas
        ==
        kube_horizontalpodautoscaler_spec_max_replicas
      for: 15m
      labels:
        severity: warning
        component: hpa
      annotations:
        summary: "HPA {{ $labels.namespace }}/{{ $labels.horizontalpodautoscaler }} maxed out"
        description: "HPA has been at max replicas ({{ $value }}) for 15 minutes"
        runbook: "Consider increasing max replicas"

    - alert: HPAScalingDisabled
      expr: |
        kube_horizontalpodautoscaler_status_condition{condition="ScalingActive",status="false"} == 1
      for: 5m
      labels:
        severity: warning
        component: hpa
      annotations:
        summary: "HPA {{ $labels.namespace }}/{{ $labels.horizontalpodautoscaler }} scaling disabled"
        description: "HPA is unable to compute metrics"

    # Control plane alerts
    - alert: APIServerHighLatency
      expr: |
        histogram_quantile(0.99,
          sum by (le) (rate(apiserver_request_duration_seconds_bucket[5m]))
        ) > 1
      for: 5m
      labels:
        severity: warning
        component: apiserver
      annotations:
        summary: "API Server high latency"
        description: "P99 latency is {{ $value }}s"

    - alert: APIServerErrorRate
      expr: |
        sum(rate(apiserver_request_total{code=~"5.."}[5m]))
        /
        sum(rate(apiserver_request_total[5m])) > 0.05
      for: 5m
      labels:
        severity: critical
        component: apiserver
      annotations:
        summary: "API Server high error rate"
        description: "Error rate is {{ $value | humanizePercentage }}"

    - alert: EtcdHighLatency
      expr: |
        histogram_quantile(0.99,
          rate(etcd_disk_wal_fsync_duration_seconds_bucket[5m])
        ) > 0.5
      for: 5m
      labels:
        severity: warning
        component: etcd
      annotations:
        summary: "etcd high latency"
        description: "P99 fsync latency is {{ $value }}s"

    # PersistentVolume alerts
    - alert: PersistentVolumeFillingUp
      expr: |
        (
          kubelet_volume_stats_available_bytes
          /
          kubelet_volume_stats_capacity_bytes
        ) < 0.1
      for: 5m
      labels:
        severity: warning
        component: pv
      annotations:
        summary: "PV {{ $labels.persistentvolumeclaim }} filling up"
        description: "Only {{ $value | humanizePercentage }} available"

````

Примени:
```bash
kubectl apply -f k8s-manifests/prometheus-rules.yaml

# Проверь rules в Prometheus
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
# Открой http://localhost:9090/rules
```

5. **Создай load generator для тестирования**:

`k8s-manifests/load-generator.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: load-generator
  namespace: demo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: load-generator
  template:
    metadata:
      labels:
        app: load-generator
    spec:
      containers:
      - name: load-generator
        image: busybox:latest
        command:
        - /bin/sh
        - -c
        - |
          while true; do
            # Нормальные запросы
            for i in $(seq 1 10); do
              wget -q -O- http://demo-app.demo-app.svc.cluster.local:8080/ > /dev/null 2>&1
              sleep 0.1
            done
            
            # Случайные медленные запросы
            if [ $((RANDOM % 10)) -eq 0 ]; then
              echo "Generating slow request..."
              wget -q -O- http://demo-app.demo-app.svc.cluster.local:8080/?sleep=3 > /dev/null 2>&1
            fi
            
            sleep 1
          done
        resources:
          requests:
            cpu: 50m
            memory: 64Mi
          limits:
            cpu: 100m
            memory: 128Mi

---
# Job для стресс-теста
apiVersion: batch/v1
kind: Job
metadata:
  name: stress-test
  namespace: demo-app
spec:
  parallelism: 5
  completions: 5
  template:
    spec:
      containers:
      - name: stress
        image: busybox:latest
        command:
        - /bin/sh
        - -c
        - |
          echo "Starting stress test..."
          for i in $(seq 1 100); do
            wget -q -O- http://demo-app.demo-app.svc.cluster.local:8080/ > /dev/null 2>&1 &
          done
          wait
          echo "Stress test complete"
      restartPolicy: Never
  backoffLimit: 4
```

Примени:
```bash
kubectl apply -f k8s-manifests/load-generator.yaml

# Запусти стресс-тест
kubectl apply -f k8s-manifests/load-generator.yaml

# Наблюдай за HPA
watch kubectl get hpa -n demo-app

# Проверь pods
watch kubectl get pods -n demo-app
```

6. **Доступ к UI**:
```bash
# Prometheus
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
# http://localhost:9090

# Grafana (admin/admin)
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
# http://localhost:3000

# Или через NodePort (если kind с portMapping)
# Prometheus: http://localhost:9090
# Grafana: http://localhost:3000
```

7. **Создай custom Grafana dashboard**:

Сохрани как `k8s-dashboard.json` и импортируй в Grafana:
```json
{
  "dashboard": {
    "title": "Kubernetes Cluster Overview",
    "tags": ["kubernetes", "cluster"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 0},
        "type": "stat",
        "title": "Cluster Status",
        "targets": [
          {
            "expr": "sum(kube_node_status_condition{condition=\"Ready\",status=\"true\"})",
            "legendFormat": "Ready Nodes"
          },
          {
            "expr": "sum(kube_pod_status_phase{phase=\"Running\"})",
            "legendFormat": "Running Pods"
          }
        ]
      },
      {
        "id": 2,
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 0},
        "type": "timeseries",
        "title": "Cluster CPU Usage",
        "targets": [
          {
            "expr": "sum(rate(container_cpu_usage_seconds_total[5m])) by (namespace)",
            "legendFormat": "{{ namespace }}"
          }
        ]
      },
      {
        "id": 3,
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 8},
        "type": "timeseries",
        "title": "Cluster Memory Usage",
        "targets": [
          {
            "expr": "sum(container_memory_working_set_bytes) by (namespace)",
            "legendFormat": "{{ namespace }}"
          }
        ]
      },
      {
        "id": 4,
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 8},
        "type": "table",
        "title": "Top Pods by CPU",
        "targets": [
          {
            "expr": "topk(10, sum(rate(container_cpu_usage_seconds_total[5m])) by (namespace, pod))",
            "format": "table",
            "instant": true
          }
        ]
      }
    ]
  }
}
```

8. **Тестирование и валидация**:
```bash
# Проверь все метрики собираются
kubectl exec -n monitoring prometheus-kube-prometheus-prometheus-0 -- \
  promtool query instant http://localhost:9090 'up'

# Проверь ServiceMonitor обнаружен
kubectl get servicemonitor -A

# Проверь targets в Prometheus
# http://localhost:9090/targets

# Проверь alerts
# http://localhost:9090/alerts

# Симулируй проблемы
# OOMKill
kubectl run oom-test --image=polinux/stress --restart=Never -- \
  stress --vm 1 --vm-bytes 1G --timeout 10s

# CPU stress для HPA
kubectl run cpu-stress --image=polinux/stress --restart=Never -- \
  stress --cpu 4 --timeout 60s

# Наблюдай за scaling
watch kubectl get hpa -n demo-app
watch kubectl get pods -n demo-app
```

### 🚀 Бонус (новое)

**1. Установи Metrics Server для kubectl top**:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Для kind нужен патч (insecure TLS)
kubectl patch deployment metrics-server -n kube-system --type='json' \
  -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'

# Проверь
kubectl top nodes
kubectl top pods -A
```

**2. Vertical Pod Autoscaler**:
```bash
# Установи VPA
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh

# Создай VPA для demo-app
cat <<EOF | kubectl apply -f -
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: demo-app-vpa
  namespace: demo-app
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: demo-app
  updatePolicy:
    updateMode: "Off"  # Рекомендации без автоприменения
  resourcePolicy:
    containerPolicies:
    - containerName: app
      minAllowed:
        cpu: 50m
        memory: 64Mi
      maxAllowed:
        cpu: 1
        memory: 1Gi
EOF

# Проверь рекомендации
kubectl describe vpa demo-app-vpa -n demo-app
```

**3. Kube-state-metrics custom metrics**:

Создай ConfigMap с custom resource state metrics:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-state-metrics-customresourcestate-config
  namespace: monitoring
data:
  config.yaml: |
    kind: CustomResourceStateMetrics
    spec:
      resources:
        - groupVersionKind:
            group: "apps"
            version: "v1"
            kind: "Deployment"
          metricNamePrefix: "kube_deployment"
          metrics:
            - name: "replicas_custom"
              help: "Custom deployment replicas metric"
              each:
                type: Gauge
                gauge:
                  path: [spec, replicas]
```

**4. Cost monitoring с OpenCost**:
```bash
# Установи OpenCost
helm install opencost opencost/opencost \
  --namespace opencost --create-namespace \
  --set prometheus.internal.enabled=false \
  --set prometheus.external.url=http://prometheus-kube-prometheus-prometheus.monitoring:9090

# Port-forward
kubectl port-forward -n opencost svc/opencost 9090:9090

# Открой UI
# http://localhost:9090
```

**5. Cluster autoscaler (для cloud)**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
      - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.29.0
        name: cluster-autoscaler
        command:
        - ./cluster-autoscaler
        - --cloud-provider=aws  # или gce, azure
        - --nodes=2:10:worker-nodes
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        resources:
          limits:
            cpu: 100m
            memory: 300Mi
          requests:
            cpu: 100m
            memory: 300Mi
```

**6. Network Policy monitoring**:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: demo-app-netpol
  namespace: demo-app
spec:
  podSelector:
    matchLabels:
      app: demo-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: demo-app
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 53  # DNS
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 443  # HTTPS
```

**7. Создай script для анализа проблем**:

`k8s-troubleshoot.sh`:
```bash
#!/bin/bash

echo "=== Kubernetes Cluster Health Check ==="
echo ""

# Nodes
echo "📦 Nodes Status:"
kubectl get nodes -o wide
echo ""

echo "⚠️  Not Ready Nodes:"
kubectl get nodes --field-selector spec.unschedulable=false | grep -v "Ready" || echo "All nodes ready"
echo ""

# Pods
echo "🔴 Failed/Pending Pods:"
kubectl get pods -A --field-selector status.phase!=Running,status.phase!=Succeeded
echo ""

echo "🔄 Restarting Pods (last hour):"
kubectl get pods -A -o json | jq -r '.items[] | select(.status.containerStatuses[]?.restartCount > 0) | "\(.metadata.namespace)/\(.metadata.name): \(.status.containerStatuses[0].restartCount) restarts"'
echo ""

# Resources
echo "📊 Top Resource Consumers:"
echo "CPU:"
kubectl top pods -A --sort-by=cpu | head -10
echo ""
echo "Memory:"
kubectl top pods -A --sort-by=memory | head -10
echo ""

# Events
echo "⚡ Recent Events (errors):"
kubectl get events -A --sort-by='.lastTimestamp' | grep -i "error\|fail\|warning" | tail -20
echo ""

# HPA Status
echo "📈 HPA Status:"
kubectl get hpa -A
echo ""

# PVC Status
echo "💾 PVC Status:"
kubectl get pvc -A
echo ""

echo "=== Health Check Complete ==="
```

**8. Monitoring Helm chart values для production**:

`prometheus-values-prod.yaml`:
```yaml
prometheus:
  prometheusSpec:
    retention: 30d
    retentionSize: "50GB"
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 100Gi
    resources:
      requests:
        cpu: 1
        memory: 2Gi
      limits:
        cpu: 2
        memory: 4Gi
    
    # High availability
    replicas: 2
    
    # Remote write для long-term storage
    remoteWrite:
    - url: "http://thanos-receive:19291/api/v1/receive"
    
    # Service monitors
    serviceMonitorSelectorNilUsesHelmValues: false
    podMonitorSelectorNilUsesHelmValues: false

alertmanager:
  alertmanagerSpec:
    replicas: 3
    storage:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 10Gi

grafana:
  replicas: 2
  persistence:
    enabled: true
    size: 10Gi
  
  # SSO integration
  grafana.ini:
    auth.generic_oauth:
      enabled: true
      name: OAuth
      allow_sign_up: true
      client_id: your-client-id
      client_secret: your-client-secret
      scopes: openid profile email
      auth_url: https://auth.example.com/authorize
      token_url: https://auth.example.com/token
      api_url: https://auth.example.com/userinfo

# Node exporter на всех nodes
prometheus-node-exporter:
  tolerations:
  - effect: NoSchedule
    operator: Exists

# Kube-state-metrics
kube-state-metrics:
  replicas: 2
  resources:
    requests:
      cpu: 100m
      memory: 256Mi
    limits:
      cpu: 200m
      memory: 512Mi
```

---

## Итоги модуля 9

После прохождения этого модуля ты должен уметь:

✅ Понимать архитектуру Kubernetes и уровни мониторинга
✅ Устанавливать и настраивать kube-prometheus-stack
✅ Создавать ServiceMonitor для auto-discovery
✅ Писать PrometheusRule для K8s alerting
✅ Настраивать HPA и VPA для autoscaling
✅ Мониторить control plane компоненты
✅ Анализировать pod restarts, OOM kills, CPU throttling
✅ Создавать Grafana dashboards для K8s
✅ Использовать kubectl top и Metrics Server
✅ Настраивать ResourceQuota и LimitRange
✅ Troubleshooting проблем в кластере
✅ Интегрировать cost monitoring

**Ключевые метрики K8s:**

**Cluster:** Nodes ready, API latency, etcd health 
**Nodes:** CPU/Memory usage, disk pressure 
**Pods:** Restarts, OOM kills, phase 
**Workload:** Replicas mismatch, HPA status 
**Network:** Traffic, errors, latency


**Production checklist:**
- ✅ Настроены resource requests/limits для всех pods
- ✅ HPA для критичных сервисов
- ✅ PodDisruptionBudget для HA
- ✅ Liveness/Readiness probes
- ✅ Monitoring всех уровней (cluster → node → pod → container)
- ✅ Alerting на критичные события
- ✅ Grafana dashboards для всей команды
- ✅ ServiceMonitor для всех приложений
- ✅ ResourceQuota для namespaces
- ✅ Network policies для security
- ✅ Regular backup etcd
- ✅ Cost tracking и optimization


## Модуль 12: Мониторинг инфраструктуры и облачных сервисов (30 минут)

### 🎯 Напоминалка

**Уровни мониторинга инфраструктуры:**

```
┌─────────────────────────────────────┐
│ Cloud Services (AWS/GCP/Azure)      │
│ - EC2, S3, RDS, Lambda              │
│ - Billing, Quotas, API limits       │
└─────────────────────────────────────┘
         ▼
┌─────────────────────────────────────┐
│ Kubernetes / Orchestration          │
│ - Cluster health                    │
│ - Pod/Node metrics                  │
│ - Resource quotas                   │
└─────────────────────────────────────┘
         ▼
┌─────────────────────────────────────┐
│ Containers (Docker)                 │
│ - Container metrics                 │
│ - Image vulnerabilities             │
└─────────────────────────────────────┘
         ▼
┌─────────────────────────────────────┐
│ Infrastructure (VMs, Bare Metal)    │
│ - CPU, Memory, Disk, Network        │
│ - Hardware health                   │
└─────────────────────────────────────┘
         ▼
┌─────────────────────────────────────┐
│ Network                             │
│ - Switches, Routers                 │
│ - Bandwidth, Latency                │
│ - SNMP, NetFlow                     │
└─────────────────────────────────────┘
```

**Kubernetes мониторинг - ключевые компоненты:**

yaml

```yaml
kube-state-metrics:
  # Метрики о состоянии K8s объектов
  - Deployments: replicas, available, unavailable
  - Pods: phase, restarts, conditions
  - Nodes: capacity, allocatable, conditions
  - PersistentVolumes: phase, capacity
  - Jobs: succeeded, failed, active

metrics-server:
  # Реальные метрики использования ресурсов
  - CPU usage (по контейнерам/подам/нодам)
  - Memory usage
  - Используется для HPA

cAdvisor:
  # Container-level метрики
  - CPU/Memory usage
  - Network I/O
  - Filesystem I/O
  - Встроен в kubelet
```

**Cloud Provider метрики:**

yaml

````yaml
AWS CloudWatch:
  - EC2: CPU, Network, Disk I/O
  - RDS: Connections, IOPS, Storage
  - S3: Requests, Bandwidth, Storage
  - Lambda: Invocations, Duration, Errors
  - ELB: Request Count, Latency, HTTP codes
  - Billing: Estimated charges

GCP Monitoring:
  - Compute Engine: CPU, Disk, Network
  - Cloud SQL: Queries, Connections
  - Cloud Storage: Operations, Bandwidth
  - Cloud Functions: Executions, Memory
  - Load Balancer: Request rate, Latency

Azure Monitor:
  - Virtual Machines: CPU, Memory, Disk
  - SQL Database: DTU, Storage, Connections
  - Storage: Transactions, Ingress/Egress
  - Functions: Execution count, Duration
  - Application Insights: APM метрики
````

**SNMP мониторинг (Network devices):**
````
SNMP OIDs (Object Identifiers):
- .1.3.6.1.2.1.1.1.0        # System description
- .1.3.6.1.2.1.1.3.0        # Uptime
- .1.3.6.1.2.1.2.2.1.10.*   # Interface inbound octets
- .1.3.6.1.2.1.2.2.1.16.*   # Interface outbound octets
- .1.3.6.1.2.1.25.1.1.0     # Host CPU load

SNMP Versions:
v1: Basic, no encryption
v2c: Community strings, better performance
v3: Authentication + Encryption (рекомендуется)
````

**Cost Monitoring:**

yaml

```yaml
Cloud Cost Metrics:
  - Daily/Monthly spend by service
  - Cost per application/team
  - Unutilized resources
  - Reserved vs On-Demand usage
  - Savings opportunities

Infrastructure Cost:
  - Compute: Instance types, utilization
  - Storage: Type, size, IOPS
  - Network: Data transfer, NAT gateways
  - Managed Services: RDS, Lambda, etc.
```

### 💻 Задание

Настрой комплексный мониторинг Kubernetes кластера:

1. **Установи kube-state-metrics**:

yaml

```yaml
# kube-state-metrics-deployment.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-state-metrics
  namespace: monitoring
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kube-state-metrics
rules:
  - apiGroups: [""]
    resources:
      - configmaps
      - secrets
      - nodes
      - pods
      - services
      - resourcequotas
      - replicationcontrollers
      - limitranges
      - persistentvolumeclaims
      - persistentvolumes
      - namespaces
      - endpoints
    verbs: ["list", "watch"]
  - apiGroups: ["apps"]
    resources:
      - statefulsets
      - daemonsets
      - deployments
      - replicasets
    verbs: ["list", "watch"]
  - apiGroups: ["batch"]
    resources:
      - cronjobs
      - jobs
    verbs: ["list", "watch"]
  - apiGroups: ["autoscaling"]
    resources:
      - horizontalpodautoscalers
    verbs: ["list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kube-state-metrics
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-state-metrics
subjects:
  - kind: ServiceAccount
    name: kube-state-metrics
    namespace: monitoring
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-state-metrics
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kube-state-metrics
  template:
    metadata:
      labels:
        app: kube-state-metrics
    spec:
      serviceAccountName: kube-state-metrics
      containers:
        - name: kube-state-metrics
          image: registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.10.1
          ports:
            - name: http-metrics
              containerPort: 8080
            - name: telemetry
              containerPort: 8081
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 5
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /
              port: 8081
            initialDelaySeconds: 5
            timeoutSeconds: 5
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 200m
              memory: 256Mi
---
apiVersion: v1
kind: Service
metadata:
  name: kube-state-metrics
  namespace: monitoring
  labels:
    app: kube-state-metrics
spec:
  ports:
    - name: http-metrics
      port: 8080
      targetPort: http-metrics
    - name: telemetry
      port: 8081
      targetPort: telemetry
  selector:
    app: kube-state-metrics
```

Примени:

bash

```bash
kubectl create namespace monitoring
kubectl apply -f kube-state-metrics-deployment.yaml
```

2. **Настрой Prometheus для Kubernetes мониторинга**:

Обнови `prometheus.yml`:

yaml

```yaml
scrape_configs:
  # ... существующие jobs

  # Kubernetes API server
  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
      - role: endpoints
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https

  # Kubernetes nodes
  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
      - role: node
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)

  # Kubernetes nodes (Kubelet)
  - job_name: 'kubernetes-nodes-kubelet'
    kubernetes_sd_configs:
      - role: node
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics

  # Kubernetes pods
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name

  # kube-state-metrics
  - job_name: 'kube-state-metrics'
    static_configs:
      - targets: ['kube-state-metrics.monitoring.svc.cluster.local:8080']

  # cAdvisor (встроен в kubelet)
  - job_name: 'kubernetes-cadvisor'
    kubernetes_sd_configs:
      - role: node
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
```

3. **Создай Kubernetes-specific алерты**:

yaml

```yaml
# kubernetes_alerts.yml
groups:
  - name: kubernetes_cluster
    rules:
      # Нода недоступна
      - alert: KubernetesNodeNotReady
        expr: kube_node_status_condition{condition="Ready",status="true"} == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Kubernetes node not ready"
          description: "Node {{ $labels.node }} has been unready for more than 5 minutes"

      # Высокое использование CPU на ноде
      - alert: KubernetesNodeHighCPU
        expr: |
          (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)) * 100 > 80
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "High CPU on node {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}%"

      # Высокое использование памяти на ноде
      - alert: KubernetesNodeHighMemory
        expr: |
          (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High memory on node {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}%"

      # Pod в CrashLoopBackOff
      - alert: KubernetesPodCrashLooping
        expr: |
          rate(kube_pod_container_status_restarts_total[15m]) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Pod {{ $labels.namespace }}/{{ $labels.pod }} crash looping"
          description: "Pod is restarting frequently"

      # Pod не может запуститься
      - alert: KubernetesPodNotReady
        expr: |
          sum by (namespace, pod) (kube_pod_status_phase{phase=~"Pending|Unknown"}) > 0
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Pod {{ $labels.namespace }}/{{ $labels.pod }} not ready"
          description: "Pod has been in {{ $labels.phase }} state for more than 10 minutes"

      # Deployment replicas mismatch
      - alert: KubernetesDeploymentReplicasMismatch
        expr: |
          kube_deployment_spec_replicas != kube_deployment_status_replicas_available
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Deployment {{ $labels.namespace }}/{{ $labels.deployment }} replicas mismatch"
          description: "Desired: {{ $value }}, Available: {{ $labels.replicas_available }}"

      # StatefulSet replicas mismatch
      - alert: KubernetesStatefulSetReplicasMismatch
        expr: |
          kube_statefulset_status_replicas_ready != kube_statefulset_status_replicas
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "StatefulSet {{ $labels.namespace }}/{{ $labels.statefulset }} replicas mismatch"

      # DaemonSet pods не на всех нодах
      - alert: KubernetesDaemonSetRolloutStuck
        expr: |
          kube_daemonset_status_number_ready / kube_daemonset_status_desired_number_scheduled * 100 < 100
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "DaemonSet {{ $labels.namespace }}/{{ $labels.daemonset }} rollout stuck"

      # Job failed
      - alert: KubernetesJobFailed
        expr: |
          kube_job_status_failed > 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Job {{ $labels.namespace }}/{{ $labels.job_name }} failed"

      # PVC pending
      - alert: KubernetesPersistentVolumeClaimPending
        expr: |
          kube_persistentvolumeclaim_status_phase{phase="Pending"} == 1
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "PVC {{ $labels.namespace }}/{{ $labels.persistentvolumeclaim }} pending"

      # Container OOMKilled
      - alert: KubernetesContainerOOMKilled
        expr: |
          (kube_pod_container_status_restarts_total - kube_pod_container_status_restarts_total offset 10m >= 1)
          and ignoring (reason) min_over_time(kube_pod_container_status_last_terminated_reason{reason="OOMKilled"}[10m]) == 1
        labels:
          severity: warning
        annotations:
          summary: "Container OOMKilled in {{ $labels.namespace }}/{{ $labels.pod }}"
          description: "Container {{ $labels.container }} was OOMKilled"

  - name: kubernetes_resources
    rules:
      # Высокое использование CPU контейнером
      - alert: KubernetesContainerHighCPU
        expr: |
          sum(rate(container_cpu_usage_seconds_total{container!=""}[5m])) by (namespace, pod, container)
          / 
          sum(kube_pod_container_resource_limits{resource="cpu"}) by (namespace, pod, container)
          * 100 > 80
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage in container"
          description: "Container {{ $labels.namespace }}/{{ $labels.pod }}/{{ $labels.container }} using {{ $value }}% of limit"

      # Высокое использование памяти контейнером
      - alert: KubernetesContainerHighMemory
        expr: |
          sum(container_memory_working_set_bytes{container!=""}) by (namespace, pod, container)
          /
          sum(kube_pod_container_resource_limits{resource="memory"}) by (namespace, pod, container)
          * 100 > 80
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage in container"
          description: "Container {{ $labels.namespace }}/{{ $labels.pod }}/{{ $labels.container }} using {{ $value }}% of limit"

      # Недостаточно ресурсов на ноде
      - alert: KubernetesNodeResourcePressure
        expr: |
          kube_node_status_condition{condition=~"MemoryPressure|DiskPressure|PIDPressure",status="true"} == 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Node {{ $labels.node }} under {{ $labels.condition }}"
```

4. **Создай комплексный Kubernetes дашборд в Grafana**:

Импортируй готовые дашборды:

- **Kubernetes Cluster Monitoring**: ID 7249
- **Kubernetes Pod Resources**: ID 6417
- **Node Exporter Full**: ID 1860

Или создай свой с панелями:

**Panel 1: Cluster Overview**

promql

```promql
# Всего нод
count(kube_node_info)

# Нод Ready
sum(kube_node_status_condition{condition="Ready",status="true"})

# Всего Pods
count(kube_pod_info)

# Running Pods
count(kube_pod_status_phase{phase="Running"})
```

**Panel 2: Resource Usage**

promql

```promql
# CPU Requests vs Allocatable
sum(kube_pod_container_resource_requests{resource="cpu"}) 
/ 
sum(kube_node_status_allocatable{resource="cpu"}) * 100

# Memory Requests vs Allocatable
sum(kube_pod_container_resource_requests{resource="memory"}) 
/ 
sum(kube_node_status_allocatable{resource="memory"}) * 100
```

**Panel 3: Pod Status by Phase**

promql

```promql
sum(kube_pod_status_phase{phase="Running"})
sum(kube_pod_status_phase{phase="Pending"})
sum(kube_pod_status_phase{phase="Failed"})
```

**Panel 4: Top Pods by CPU**

promql

```promql
topk(10, 
  sum(rate(container_cpu_usage_seconds_total{container!=""}[5m])) by (namespace, pod)
)
```

**Panel 5: Top Pods by Memory**

promql

```promql
topk(10,
  sum(container_memory_working_set_bytes{container!=""}) by (namespace, pod)
)
```

**Panel 6: Network Traffic**

promql

```promql
# Inbound
sum(rate(container_network_receive_bytes_total[5m])) by (namespace, pod)

# Outbound
sum(rate(container_network_transmit_bytes_total[5m])) by (namespace, pod)
```

5. **Тестирование мониторинга**:

bash

```bash
# Создай тестовое приложение
kubectl create deployment test-app --image=nginx --replicas=3

# Посмотри метрики
kubectl port-forward -n monitoring svc/prometheus 9090:9090

# Открой Prometheus
http://localhost:9090

# Попробуй запросы:
# - kube_deployment_status_replicas{deployment="test-app"}
# - kube_pod_info{created_by_name="test-app"}
# - sum(rate(container_cpu_usage_seconds_total{pod=~"test-app.*"}[5m]))

# Симулируй проблему
kubectl scale deployment test-app --replicas=10
kubectl delete pod -l app=test-app --force --grace-period=0

# Смотри алерты в Alertmanager
http://localhost:9093
```

### 🚀 Бонус (новое)

**1. Настрой AWS CloudWatch Exporter** для мониторинга AWS ресурсов:

yaml

```yaml
  cloudwatch-exporter:
    image: prom/cloudwatch-exporter:latest
    container_name: cloudwatch-exporter
    ports:
      - "9106:9106"
    volumes:
      - ./cloudwatch-exporter.yml:/config/config.yml
      - ~/.aws:/root/.aws:ro
    command:
      - '/bin/cloudwatch_exporter'
      - '/config/config.yml'
    restart: unless-stopped
```

`cloudwatch-exporter.yml`:

yaml

```yaml
region: us-east-1
metrics:
  # EC2 Instances
  - aws_namespace: AWS/EC2
    aws_metric_name: CPUUtilization
    aws_dimensions:
      - InstanceId
    aws_statistics:
      - Average
    period_seconds: 300
    range_seconds: 600

  - aws_namespace: AWS/EC2
    aws_metric_name: NetworkIn
    aws_dimensions:
      - InstanceId
    aws_statistics:
      - Sum
    period_seconds: 300

  # RDS
  - aws_namespace: AWS/RDS
    aws_metric_name: DatabaseConnections
    aws_dimensions:
      - DBInstanceIdentifier
    aws_statistics:
      - Average
    period_seconds: 300

  - aws_namespace: AWS/RDS
    aws_metric_name: ReadLatency
    aws_dimensions:
      - DBInstanceIdentifier
    aws_statistics:
      - Average
    period_seconds: 300

  # ELB
  - aws_namespace: AWS/ELB
    aws_metric_name: RequestCount
    aws_dimensions:
      - LoadBalancerName
    aws_statistics:
      - Sum
    period_seconds: 300

  - aws_namespace: AWS/ELB
    aws_metric_name: Latency
    aws_dimensions:
      - LoadBalancerName
    aws_statistics:
      - Average
    period_seconds: 300

  # Lambda
  - aws_namespace: AWS/Lambda
    aws_metric_name: Invocations
    aws_dimensions:
      - FunctionName
    aws_statistics:
      - Sum
    period_seconds: 300

  - aws_namespace: AWS/Lambda
    aws_metric_name: Duration
    aws_dimensions:
      - FunctionName
    aws_statistics:
      - Average
    period_seconds: 300

  - aws_namespace: AWS/Lambda
    aws_metric_name: Errors
    aws_dimensions:
      - FunctionName
    aws_statistics:
      - Sum
    period_seconds: 300

  # S3
  - aws_namespace: AWS/S3
    aws_metric_name: NumberOfObjects
    aws_dimensions:
      - BucketName
      - StorageType
    aws_statistics:
      - Average
    period_seconds: 86400  # Once per day

  # Billing
  - aws_namespace: AWS/Billing
    aws_metric_name: EstimatedCharges
    aws_dimensions:
      - Currency
    aws_statistics:
      - Maximum
    period_seconds: 86400
```

Добавь в `prometheus.yml`:

yaml

```yaml
scrape_configs:
  - job_name: 'cloudwatch'
    static_configs:
      - targets: ['cloudwatch-exporter:9106']
```

**2. Мониторинг Network с SNMP Exporter**:

yaml

```yaml
  snmp-exporter:
    image: prom/snmp-exporter:latest
    container_name: snmp-exporter
    ports:
      - "9116:9116"
    volumes:
      - ./snmp.yml:/etc/snmp_exporter/snmp.yml
    command:
      - '--config.file=/etc/snmp_exporter/snmp.yml'
    restart: unless-stopped
```

`snmp.yml` (пример для Cisco):

yaml

```yaml
auths:
  public_v2:
    community: public
    security_level: noAuthNoPriv
    auth_protocol: MD5
    priv_protocol: DES
    version: 2

modules:
  if_mib:
    walk:
      - 1.3.6.1.2.1.2.2.1.2   # ifDescr
      - 1.3.6.1.2.1.2.2.1.10  # ifInOctets
      - 1.3.6.1.2.1.2.2.1.16  # ifOutOctets
      - 1.3.6.1.2.1.2.2.1.8   # ifOperStatus
    
    lookups:
      - source_indexes: [ifIndex]
        lookup: ifDescr
    
    overrides:
      ifDescr:
        type: DisplayString
      ifOperStatus:
        type: gauge
```

Prometheus config:

yaml

```yaml
scrape_configs:
  - job_name: 'snmp'
    static_configs:
      - targets:
          - 192.168.1.1  # Switch IP
          - 192.168.1.2  # Router IP
    metrics_path: /snmp
    params:
      module: [if_mib]
      auth: [public_v2]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: snmp-exporter:9116
```

**3. Cost Monitoring Dashboard**:

Создай панели в Grafana:

promql

```promql
# AWS Estimated Charges (last value)
aws_billing_estimated_charges_maximum{currency="USD"}

# Daily cost trend
increase(aws_billing_estimated_charges_maximum{currency="USD"}[1d])

# Cost by service (requires detailed billing)
sum by (service) (aws_cloudwatch_billing_estimated_charges_average)

# Top 10 most expensive resources
topk(10, 
  sum by (resource_id) (aws_resource_cost_daily)
)

# Unutilized resources cost
sum(aws_ec2_cpu_utilization_average < 10) * avg(aws_ec2_pricing_hourly)

# Kubernetes cost by namespace
sum by (namespace) (
  avg_over_time(container_cpu_usage_seconds_total[1h]) * 
  scalar(aws_ec2_pricing_hourly / 8)  # assuming 8 vCPUs per instance
) * 24 * 30  # Monthly estimate
```

**4. Infrastructure as Code Monitoring**:

Мониторинг Terraform state:

python

```python
# terraform_exporter.py
from prometheus_client import start_http_server, Gauge
import json
import subprocess
import time

# Метрики
terraform_resource_count = Gauge('terraform_resource_count', 
                                 'Number of resources in Terraform state',
                                 ['workspace', 'type'])
terraform_drift_detected = Gauge('terraform_drift_detected',
                                'Drift detected in Terraform',
                                ['workspace'])

def check_terraform_state():
    """Проверка Terraform state"""
    try:
        # Get resource count
        result = subprocess.run(
            ['terraform', 'state', 'list'],
            capture_output=True,
            text=True,
            check=True
        )
        
        resources = result.stdout.strip().split('\n')
        resource_types = {}
        
        for resource in resources:
            if resource:
                resource_type = resource.split('.')[0]
                resource_types[resource_type] = resource_types.get(resource_type, 0) + 1
        
        workspace = subprocess.run(
            ['terraform', 'workspace', 'show'],
            capture_output=True,
            text=True,
            check=True
        ).stdout.strip()
        
        # Update metrics
        for rtype, count in resource_types.items():
            terraform_resource_count.labels(
                workspace=workspace,
                type=rtype
            ).set(count)
        
        # Check for drift
        plan_result = subprocess.run(
            ['terraform', 'plan', '-detailed-exitcode'],
            capture_output=True
        )
        
        # Exit code 2 means changes detected
        if plan_result.returncode == 2:
            terraform_drift_detected.labels(workspace=workspace).set(1)
            print(f"⚠️  Drift detected in {workspace}")
        else:
            terraform_drift_detected.labels(workspace=workspace).set(0)
            print(f"✓ No drift in {workspace}")
            
    except Exception as e:
        print(f"Error checking Terraform: {e}")

if __name__ == '__main__':
    start_http_server(8000)
    print("Terraform exporter started on :8000")
    
    while True:
        check_terraform_state()
        time.sleep(300)  # Check every 5 minutes
```

---

**Чеклист модуля 11:**

- ✅ Настроил kube-state-metrics
- ✅ Создал Kubernetes алерты
- ✅ Построил K8s дашборды
- ✅  Интегрировал cloud monitoring (AWS/GCP/Azure)
- ✅ Настроил network monitoring (SNMP)
- ✅  Мониторинг стоимости инфраструктуры

## Модуль 13: Мониторинг в продакшене - Best Practices (15 минут)

### 🎯 Напоминалка

**SRE принципы:**

```
SLI (Service Level Indicator)   - Метрика качества сервиса
SLO (Service Level Objective)   - Целевое значение SLI
SLA (Service Level Agreement)   - Договорное обязательство
Error Budget                     - Допустимое количество ошибок
```

**Примеры SLI/SLO:**

```
SLI: Availability
SLO: 99.9% uptime (43.2 min downtime/month)

SLI: Latency
SLO: 95% requests < 100ms

SLI: Error Rate
SLO: < 0.1% error rate

SLI: Throughput
SLO: Handle 10,000 req/s
```

**Error Budget:**

```
Uptime SLO: 99.9%
Allowed downtime: 0.1% = 43.2 min/month

If error budget > 0:
  → Can take risks, deploy faster
  
If error budget = 0:
  → Focus on reliability, slow deploys
```

**Monitoring Best Practices:**

```
✅ DO:
- Monitor symptoms, not causes
- Alert on SLO violations
- Use runbooks for alerts
- Test alerts regularly
- Keep dashboards simple
- Document everything
- Use labels consistently
- Set up test environments
- Automate alert remediation where possible
- Review alerts quarterly

❌ DON'T:
- Alert on everything
- Set alert thresholds too tight
- Ignore alert fatigue
- Monitor without context
- Create dashboards without purpose
- Alert without actionable next steps
```

**Dashboard hierarchy:**

```
Level 1: Overview (C-level)
- Overall system health
- Key business metrics
- High-level SLOs

Level 2: Service (Team leads)
- Per-service metrics
- RED/USE metrics
- Resource utilization

Level 3: Detailed (Engineers)
- Detailed metrics
- Debug information
- Deep-dive panels
```

**On-call best practices:**

```
1. Clear escalation paths
2. Comprehensive runbooks
3. Postmortems for incidents
4. Fair rotation schedule
5. Context in alerts
6. Response time SLOs
7. Blameless culture
8. Regular drills
```

**Incident Response Process:**

```
1. Detection    - Alert fires
2. Triage       - Assess severity
3. Mitigation   - Stop the bleeding
4. Investigation - Find root cause
5. Resolution   - Fix permanently
6. Postmortem   - Learn & improve
```

**Cost optimization:**

```
- Use recording rules for expensive queries
- Set appropriate retention periods
- Use downsampling for old data
- Archive cold data
- Monitor cardinality
- Use relabeling to drop unnecessary metrics
- Implement metric limits
```

### 💻 Задание

Создай production-ready monitoring setup:

1. **Определи SLIs/SLOs для сервиса**:

**slo_config.yml**:

yaml

```yaml
slos:
  - name: api_availability
    description: "API должен быть доступен 99.9% времени"
    target: 0.999
    window: 30d
    sli:
      error_ratio_query: |
        sum(rate(http_requests_total{job="frontend",status=~"5.."}[5m]))
        /
        sum(rate(http_requests_total{job="frontend"}[5m]))

  - name: api_latency
    description: "95% запросов должны обрабатываться < 200ms"
    target: 0.95
    window: 30d
    sli:
      latency_query: |
        histogram_quantile(0.95,
          rate(http_request_duration_seconds_bucket{job="frontend"}[5m])
        ) < 0.2

  - name: error_rate
    description: "Процент ошибок < 0.1%"
    target: 0.999
    window: 30d
    sli:
      error_ratio_query: |
        (
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m]))
        ) < 0.001
```

2. **Создай SLO alerts** (добавь в `alerts.yml`):

yaml

```yaml
groups:
  - name: slo_alerts
    rules:
    # Error Budget Alert
    - alert: ErrorBudgetBurn
      expr: |
        (
          1 - (
            sum(rate(http_requests_total{status=~"2.."}[1h]))
            /
            sum(rate(http_requests_total[1h]))
          )
        ) > 0.001
      for: 5m
      labels:
        severity: critical
        slo: availability
      annotations:
        summary: "Error budget burning too fast"
        description: "Current error rate {{ $value | humanizePercentage }} exceeds budget"
        runbook: "https://runbook.example.com/error-budget"

    # Latency SLO violation
    - alert: LatencySLOViolation
      expr: |
        histogram_quantile(0.95,
          rate(http_request_duration_seconds_bucket[5m])
        ) > 0.2
      for: 10m
      labels:
        severity: warning
        slo: latency
      annotations:
        summary: "Latency SLO violation"
        description: "p95 latency is {{ $value }}s (SLO: 0.2s)"
        impact: "Users experiencing slow responses"
        action: "Check service performance and database queries"

    # Multi-window burn rate
    - alert: ErrorBudgetFastBurn
      expr: |
        (
          (1 - avg_over_time(up[1h]) < 0.999)
          and
          (1 - avg_over_time(up[5m]) < 0.999)
        )
      labels:
        severity: critical
        burn_rate: fast
      annotations:
        summary: "Error budget burning at fast rate"
        description: "Both short and long windows show SLO violations"
```

3. **Создай comprehensive dashboard** (`production-overview.json`):

json

```json
{
  "dashboard": {
    "title": "Production Overview",
    "tags": ["production", "slo"],
    "rows": [
      {
        "title": "SLOs",
        "panels": [
          {
            "title": "Availability (30d SLO: 99.9%)",
            "targets": [{
              "expr": "avg_over_time(up[30d]) * 100"
            }],
            "type": "stat",
            "thresholds": [
              {"value": 99.9, "color": "green"},
              {"value": 99.5, "color": "yellow"},
              {"value": 0, "color": "red"}
            ]
          },
          {
            "title": "Error Budget Remaining",
            "targets": [{
              "expr": "(0.999 - (1 - avg_over_time(up[30d]))) / 0.001 * 100"
            }],
            "type": "gauge"
          }
        ]
      },
      {
        "title": "Golden Signals",
        "panels": [
          {
            "title": "Request Rate",
            "targets": [{
              "expr": "sum(rate(http_requests_total[5m]))"
            }]
          },
          {
            "title": "Error Rate",
            "targets": [{
              "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100"
            }]
          },
          {
            "title": "Latency (p50, p95, p99)",
            "targets": [
              {"expr": "histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))", "legendFormat": "p50"},
              {"expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))", "legendFormat": "p95"},
              {"expr": "histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))", "legendFormat": "p99"}
            ]
          }
        ]
      }
    ]
  }
}
```

4. **Создай runbook template** (`runbooks/high-error-rate.md`):

markdown

````markdown
# Runbook: High Error Rate

## Alert
**Name:** HighErrorRate
**Severity:** Critical
**SLO Impact:** Availability

## Symptoms
- Error rate > 1% for more than 5 minutes
- Users experiencing 5xx errors
- Error budget burning

## Impact
- **Users:** Cannot complete requests
- **Business:** Loss of revenue/trust
- **SLO:** Burns error budget

## Diagnosis

### Step 1: Verify the alert
```bash
# Check current error rate
curl -G http://prometheus:9090/api/v1/query \
  --data-urlencode 'query=rate(http_requests_total{status=~"5.."}[5m])/rate(http_requests_total[5m])'
```

### Step 2: Identify affected services
```bash
# Check which services are returning errors
# Grafana → Explore → Prometheus
sum by (job) (rate(http_requests_total{status=~"5.."}[5m]))
```

### Step 3: Check logs
```bash
# Grafana → Explore → Loki
{job="frontend"} |= "ERROR" | json
```

### Step 4: Check recent deployments
```bash
# Check if error rate increased after deployment
kubectl get pods -o wide
kubectl describe deployment frontend
```

### Step 5: Check dependencies
```bash
# Database
psql -c "SELECT pg_is_in_recovery();"

# Cache
redis-cli ping

# External APIs
curl https://api.external.com/health
```

## Mitigation

### Immediate (Stop the bleeding)
1. **Rollback deployment** if recent:
```bash
kubectl rollout undo deployment/frontend
```

2. **Scale up** if resource constrained:
```bash
kubectl scale deployment/frontend --replicas=10
```

3. **Enable circuit breaker** for failing dependency:
```bash
# Update config to bypass failing service
```

4. **Put up maintenance page** if critical:
```bash
# Route traffic to maintenance page
```

### Short-term (Stabilize)
1. Investigate root cause
2. Apply proper fix
3. Deploy with gradual rollout
4. Monitor closely

## Resolution
- [ ] Error rate back to < 0.1%
- [ ] Root cause identified
- [ ] Fix deployed and verified
- [ ] Monitoring confirms stability
- [ ] Postmortem scheduled

## Escalation
- **L1:** On-call engineer (You)
- **L2:** Team lead (after 15 min)
- **L3:** Engineering manager (after 30 min)
- **L4:** VP Engineering (critical)

## References
- Dashboard: http://grafana/d/prod-overview
- Logs: http://grafana/explore?loki
- Traces: http://jaeger:16686
- Slack: #incidents

## Related Runbooks
- [Database Connection Issues](./db-connection.md)
- [High Latency](./high-latency.md)
- [Service Down](./service-down.md)
````

5. **Создай incident response script** (`scripts/incident_response.sh`):

bash

```bash
#!/bin/bash

# Incident Response Helper Script

set -e

ALERT_NAME=$1
SEVERITY=$2

if [ -z "$ALERT_NAME" ]; then
  echo "Usage: ./incident_response.sh <alert_name> <severity>"
  exit 1
fi

echo "🚨 Incident Response Started"
echo "Alert: $ALERT_NAME"
echo "Severity: $SEVERITY"
echo "Time: $(date)"
echo ""

# 1. Gather context
echo "📊 Gathering context..."
echo ""

echo "Current Metrics:"
curl -s -G http://localhost:9090/api/v1/query \
  --data-urlencode 'query=up' | jq '.data.result[] | {job: .metric.job, status: .value[1]}'

echo ""
echo "Recent Errors (last 5 min):"
curl -s -G http://localhost:9090/api/v1/query \
  --data-urlencode 'query=sum(rate(http_requests_total{status=~"5.."}[5m]))' | jq '.data.result[0].value[1]'

echo ""
echo "Active Alerts:"
curl -s http://localhost:9090/api/v1/alerts | jq '.data.alerts[] | select(.state=="firing") | {alert: .labels.alertname, severity: .labels.severity}'

# 2. Check recent changes
echo ""
echo "🔍 Recent changes..."
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"

# 3. Generate incident report
INCIDENT_ID="INC-$(date +%Y%m%d-%H%M%S)"
echo ""
echo "📝 Creating incident report: $INCIDENT_ID"

cat > "incidents/${INCIDENT_ID}.md" <<EOF
# Incident Report: $INCIDENT_ID

## Summary
- **Alert:** $ALERT_NAME
- **Severity:** $SEVERITY
- **Start Time:** $(date)
- **Status:** Investigating

## Timeline
- $(date +%H:%M:%S) - Alert fired
- $(date +%H:%M:%S) - Investigation started

## Impact
- [ ] Users affected: TBD
- [ ] Services affected: TBD
- [ ] Revenue impact: TBD

## Actions Taken
- Gathered initial context
- Reviewed metrics and logs

## Next Steps
1. Identify root cause
2. Implement mitigation
3. Verify resolution
4. Schedule postmortem

## Notes
EOF

echo "Incident report created: incidents/${INCIDENT_ID}.md"
echo ""
echo "✅ Context gathered. Next: Check runbook at runbooks/${ALERT_NAME}.md"
```

### 🚀 Бонус (новое)

**Создай automated remediation** для частых проблем:

**auto_remediation.py**:

python

```python
import requests
import time
from datetime import datetime

PROMETHEUS_URL = "http://localhost:9090"
ALERTMANAGER_URL = "http://localhost:9093"

def check_alerts():
    """Check for active alerts"""
    response = requests.get(f"{PROMETHEUS_URL}/api/v1/alerts")
    alerts = response.json()['data']['alerts']
    
    firing_alerts = [a for a in alerts if a['state'] == 'firing']
    return firing_alerts

def auto_remediate(alert):
    """Attempt automatic remediation"""
    alert_name = alert['labels']['alertname']
    
    print(f"[{datetime.now()}] Attempting auto-remediation for: {alert_name}")
    
    if alert_name == "HighMemoryUsage":
        # Clear caches
        print("  → Clearing application caches")
        requests.post("http://localhost:5000/admin/clear-cache")
        
    elif alert_name == "HighCPUUsage":
        # Scale up service
        print("  → Scaling up service")
        # kubectl scale deployment --replicas=+2
        
    elif alert_name == "DiskSpaceLow":
        # Clean old logs
        print("  → Cleaning old logs")
        import subprocess
        subprocess.run(["find", "/var/log", "-name", "*.log.gz", "-mtime", "+7", "-delete"])
    
    else:
        print(f"  ⚠️  No auto-remediation for {alert_name}")
        return False
    
    print(f"  ✅ Auto-remediation completed")
    return True

def create_silence(alert, duration_hours=1):
    """Create silence after remediation"""
    silence = {
        "matchers": [
            {
                "name": "alertname",
                "value": alert['labels']['alertname'],
                "isRegex": False
            }
        ],
        "startsAt": datetime.utcnow().isoformat() + "Z",
        "endsAt": datetime.utcnow().isoformat() + "Z",  # +duration
        "createdBy": "auto-remediation",
        "comment": f"Auto-remediated at {datetime.now()}"
    }
    
    requests.post(f"{ALERTMANAGER_URL}/api/v2/silences", json=silence)

if __name__ == '__main__':
    print("🤖 Auto-remediation service started")
    
    while True:
        alerts = check_alerts()
        
        for alert in alerts:
            if auto_remediate(alert):
                create_silence(alert, duration_hours=1)
        
        time.sleep(60)
```


### 🚀 Бонус: Chaos Engineering

**Настрой chaos engineering для тестирования мониторинга**:

**chaos_test.sh**:

bash

```bash
#!/bin/bash

echo "🔥 Starting Chaos Engineering Tests"
echo "Testing monitoring system resilience..."
echo ""

# Test 1: Kill random container
echo "Test 1: Container Failure"
CONTAINER=$(docker ps --format "{{.Names}}" | grep -E "frontend|auth|business" | shuf -n 1)
echo "  → Killing: $CONTAINER"
docker kill $CONTAINER
echo "  → Waiting 30 seconds..."
sleep 30
echo "  → Restoring container..."
docker-compose up -d $CONTAINER
echo "  ✅ Test 1 complete"
echo ""

# Test 2: Simulate high CPU
echo "Test 2: High CPU Load"
echo "  → Starting CPU stress on frontend..."
docker exec frontend sh -c "for i in 1 2 3 4; do yes > /dev/null & done" 2>/dev/null
echo "  → Stress running for 60 seconds..."
sleep 60
echo "  → Stopping stress..."
docker exec frontend sh -c "pkill yes" 2>/dev/null
echo "  ✅ Test 2 complete"
echo ""

# Test 3: Simulate network latency
echo "Test 3: Network Latency"
echo "  → Adding 200ms latency..."
docker exec frontend sh -c "tc qdisc add dev eth0 root netem delay 200ms" 2>/dev/null || echo "  ⚠️  tc not available"
sleep 60
echo "  → Removing latency..."
docker exec frontend sh -c "tc qdisc del dev eth0 root" 2>/dev/null
echo "  ✅ Test 3 complete"
echo ""

# Test 4: Disk space pressure
echo "Test 4: Disk Space Pressure"
echo "  → Creating 1GB file..."
docker exec frontend sh -c "dd if=/dev/zero of=/tmp/fillfile bs=1M count=1000" 2>/dev/null
sleep 30
echo "  → Removing file..."
docker exec frontend sh -c "rm /tmp/fillfile" 2>/dev/null
echo "  ✅ Test 4 complete"
echo ""

# Test 5: Memory pressure
echo "Test 5: Memory Pressure"
echo "  → Allocating 512MB..."
docker exec frontend sh -c "stress --vm 1 --vm-bytes 512M --timeout 60s" 2>/dev/null || echo "  ⚠️  stress tool not available"
echo "  ✅ Test 5 complete"
echo ""

# Test 6: Database connection failure
echo "Test 6: Database Connection Failure"
echo "  → Stopping PostgreSQL..."
docker-compose stop postgres
sleep 30
echo "  → Restarting PostgreSQL..."
docker-compose start postgres
sleep 10
echo "  ✅ Test 6 complete"
echo ""

# Test 7: Random 500 errors
echo "Test 7: Random Application Errors"
echo "  → Injecting errors for 60 seconds..."
# Здесь можно добавить код для инъекции ошибок в приложение
for i in {1..20}; do
    curl -X POST http://localhost:5000/api/order 2>/dev/null
    sleep 3
done
echo "  ✅ Test 7 complete"
echo ""

echo "🎉 All chaos tests completed!"
echo ""
echo "📊 Check your monitoring:"
echo "  Prometheus Alerts: http://localhost:9090/alerts"
echo "  Grafana Dashboards: http://localhost:3000"
echo "  Alertmanager: http://localhost:9093"
echo "  Jaeger Traces: http://localhost:16686"
echo ""
echo "Questions to verify:"
echo "  ✓ Did alerts fire as expected?"
echo "  ✓ Were all incidents visible in dashboards?"
echo "  ✓ Did traces show the failures?"
echo "  ✓ Were logs properly collected?"
echo "  ✓ Did services recover automatically?"
```

**Запуск:**

bash

```bash
chmod +x chaos_test.sh
./chaos_test.sh
```
## Модуль 14: Финальный проект и карьера (30 минут)

### 🎯 Финальный проект: E-Commerce Monitoring Stack

Создай production-ready мониторинг для интернет-магазина.

**Архитектура проекта:**

```
┌──────────────────── Frontend Layer ─────────────────────┐
│  NGINX (Load Balancer) → Frontend Services (x3)         │
└────────────────────────┬────────────────────────────────┘
                         │
┌──────────────────── API Layer ──────────────────────────┐
│  API Gateway → Authentication → Rate Limiting            │
└────────────────────────┬────────────────────────────────┘
                         │
┌──────────────────── Service Layer ──────────────────────┐
│  Product Service │ Order Service │ Payment Service       │
│  Inventory Svc   │ User Service  │ Notification Svc     │
└────────────────────────┬────────────────────────────────┘
                         │
┌──────────────────── Data Layer ─────────────────────────┐
│  PostgreSQL (Primary + Replica) │ Redis Cache │ S3       │
└──────────────────────────────────────────────────────────┘

┌──────────────────── Monitoring Layer ───────────────────┐
│ Metrics: Prometheus + Grafana                            │
│ Logs: Loki + Promtail                                    │
│ Traces: Tempo + Jaeger                                   │
│ Alerts: Alertmanager → PagerDuty/Slack                   │
└──────────────────────────────────────────────────────────┘
```

### 💻 Задание: Полная реализация

**Шаг 1: Клонируй структуру проекта**

bash

```bash
mkdir ecommerce-monitoring
cd ecommerce-monitoring

# Структура
mkdir -p {services/{frontend,api-gateway,product,order,payment},monitoring/{prometheus,grafana,loki,alertmanager},scripts,docs}
```

**Шаг 2: Создай docker-compose-final.yml**

yaml

```yaml
version: '3.8'

networks:
  frontend:
  backend:
  monitoring:

services:
  # === Load Balancer ===
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    networks:
      - frontend
      - monitoring
    depends_on:
      - frontend

  # === Application Services ===
  frontend:
    build: ./services/frontend
    deploy:
      replicas: 3
    environment:
      API_URL: http://api-gateway:8080
      JAEGER_AGENT_HOST: jaeger
    networks:
      - frontend
      - monitoring

  api-gateway:
    build: ./services/api-gateway
    environment:
      PRODUCT_SERVICE: http://product-service:8081
      ORDER_SERVICE: http://order-service:8082
      PAYMENT_SERVICE: http://payment-service:8083
      REDIS_URL: redis://redis:6379
    networks:
      - frontend
      - backend
      - monitoring

  product-service:
    build: ./services/product
    environment:
      DATABASE_URL: postgresql://postgres:password@postgres:5432/products
      REDIS_URL: redis://redis:6379
    networks:
      - backend
      - monitoring

  order-service:
    build: ./services/order
    environment:
      DATABASE_URL: postgresql://postgres:password@postgres:5432/orders
      PAYMENT_SERVICE: http://payment-service:8083
    networks:
      - backend
      - monitoring

  payment-service:
    build: ./services/payment
    environment:
      DATABASE_URL: postgresql://postgres:password@postgres:5432/payments
    networks:
      - backend
      - monitoring

  # === Data Layer ===
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_DB: ecommerce
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - backend
      - monitoring

  redis:
    image: redis:7-alpine
    networks:
      - backend
      - monitoring

  # === Monitoring Stack ===
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=30d'
      - '--web.enable-lifecycle'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
      GF_INSTALL_PLUGINS: grafana-piechart-panel
    volumes:
      - ./monitoring/grafana:/etc/grafana/provisioning
      - grafana-data:/var/lib/grafana
    networks:
      - monitoring

  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    volumes:
      - ./monitoring/loki:/etc/loki
      - loki-data:/loki
    networks:
      - monitoring

  promtail:
    image: grafana/promtail:latest
    volumes:
      - ./monitoring/promtail:/etc/promtail
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    networks:
      - monitoring

  tempo:
    image: grafana/tempo:latest
    ports:
      - "3200:3200"
      - "4317:4317"
    volumes:
      - ./monitoring/tempo:/etc/tempo
      - tempo-data:/tmp/tempo
    networks:
      - monitoring

  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"
      - "14268:14268"
    environment:
      SPAN_STORAGE_TYPE: badger
      BADGER_EPHEMERAL: "false"
      BADGER_DIRECTORY_VALUE: /badger/data
      BADGER_DIRECTORY_KEY: /badger/key
    volumes:
      - jaeger-data:/badger
    networks:
      - monitoring

  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./monitoring/alertmanager:/etc/alertmanager
    networks:
      - monitoring

volumes:
  postgres-data:
  prometheus-data:
  grafana-data:
  loki-data:
  tempo-data:
  jaeger-data:
```

**Шаг 3: Определи SLIs и SLOs**

**docs/slo-definitions.md**:

markdown

```markdown
# Service Level Objectives (SLOs)

## 1. Availability SLO
**SLI:** Percentage of successful requests
**SLO:** 99.9% (43.2 minutes downtime/month)
**Measurement:** `sum(rate(http_requests{status=~"2.."}[30d])) / sum(rate(http_requests[30d]))`

## 2. Latency SLO
**SLI:** 95th percentile response time
**SLO:** < 500ms for 95% of requests
**Measurement:** `histogram_quantile(0.95, rate(http_duration_bucket[5m]))`

## 3. Error Budget
**Calculation:** (1 - SLO) × Total requests
**30-day budget:** 0.1% × requests = allowed errors
**Burn rate alerting:**
- Fast burn: 2% budget in 1 hour → Page
- Slow burn: 10% budget in 6 hours → Ticket

## 4. Business SLOs

### Order Processing
- **SLO:** 99.5% orders processed successfully
- **Target:** < 1 minute processing time

### Payment Success
- **SLO:** 99.9% payment success rate
- **Target:** < 3 seconds payment confirmation

### Search Response
- **SLO:** 95% searches return results
- **Target:** < 200ms search response time
```

**Шаг 4: Создай Production Dashboards**

**monitoring/grafana/dashboards/01-executive-overview.json**:

json

```json
{
  "dashboard": {
    "title": "Executive Overview",
    "tags": ["business", "executive"],
    "panels": [
      {
        "title": "System Health Score",
        "type": "gauge",
        "gridPos": {"h": 8, "w": 6, "x": 0, "y": 0},
        "targets": [{
          "expr": "avg((up{job=~\".*service\"} == 1) * 100)"
        }],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "min": 0,
            "max": 100,
            "thresholds": {
              "steps": [
                {"value": 0, "color": "red"},
                {"value": 95, "color": "yellow"},
                {"value": 99, "color": "green"}
              ]
            }
          }
        }
      },
      {
        "title": "Orders per Hour",
        "type": "stat",
        "gridPos": {"h": 4, "w": 6, "x": 6, "y": 0},
        "targets": [{
          "expr": "sum(increase(orders_total[1h]))"
        }],
        "fieldConfig": {
          "defaults": {
            "unit": "short",
            "color": {"mode": "thresholds"},
            "thresholds": {
              "steps": [
                {"value": 0, "color": "red"},
                {"value": 100, "color": "yellow"},
                {"value": 500, "color": "green"}
              ]
            }
          }
        }
      },
      {
        "title": "Revenue Today",
        "type": "stat",
        "gridPos": {"h": 4, "w": 6, "x": 12, "y": 0},
        "targets": [{
          "expr": "sum(increase(payment_amount_total[24h]))"
        }],
        "fieldConfig": {
          "defaults": {
            "unit": "currencyUSD",
            "decimals": 2
          }
        }
      },
      {
        "title": "Active Users",
        "type": "graph",
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 8},
        "targets": [{
          "expr": "sum(rate(http_requests_total{endpoint=\"/\"}[5m])) * 60",
          "legendFormat": "Active Users"
        }]
      }
    ]
  }
}
```

**Шаг 5: Comprehensive Alerts**

**monitoring/prometheus/alerts/production.yml**:

yaml

```yaml
groups:
  - name: critical_slo_violations
    interval: 30s
    rules:
    # Multi-window burn rate (Google SRE)
    - alert: ErrorBudgetCriticalBurn
      expr: |
        (
          sum(rate(http_requests{status=~"5.."}[1h]))
          / sum(rate(http_requests[1h]))
        ) > (14.4 * 0.001)  # 2% of monthly budget in 1 hour
        and
        (
          sum(rate(http_requests{status=~"5.."}[5m]))
          / sum(rate(http_requests[5m]))
        ) > (14.4 * 0.001)
      labels:
        severity: page
        team: sre
      annotations:
        summary: "🚨 Critical error budget burn"
        description: "Burning 2% of 30-day error budget per hour"
        runbook: "https://runbook.company.com/error-budget-burn"
        action: "Page on-call engineer immediately"

    - alert: ServiceDown
      expr: up{job=~".*-service"} == 0
      for: 1m
      labels:
        severity: page
        team: platform
      annotations:
        summary: "🔴 Service {{ $labels.job }} is DOWN"
        description: "{{ $labels.instance }} unreachable for 1+ minutes"
        impact: "Service unavailable to users"

  - name: slo_approaching_violations
    interval: 1m
    rules:
    - alert: LatencySLOAtRisk
      expr: |
        histogram_quantile(0.95,
          sum(rate(http_duration_bucket[30m])) by (le)
        ) > 0.45  # 90% of 500ms threshold
      for: 15m
      labels:
        severity: warning
        team: backend
      annotations:
        summary: "⚠️ Latency approaching SLO limit"
        description: "p95 latency: {{ $value }}s (SLO: 0.5s)"

  - name: business_kpis
    interval: 5m
    rules:
    - alert: OrderRateDropCritical
      expr: |
        (
          sum(rate(orders_total[10m]))
          /
          sum(rate(orders_total[10m] offset 1h))
        ) < 0.5
      for: 10m
      labels:
        severity: page
        team: business
      annotations:
        summary: "📉 Order rate dropped 50%+"
        description: "Current: {{ $value | humanizePercentage }} of normal"
        impact: "Severe revenue impact"

    - alert: PaymentFailureSpike
      expr: |
        sum(rate(payments_total{status="failed"}[5m]))
        / sum(rate(payments_total[5m]))
        > 0.05
      for: 5m
      labels:
        severity: page
        team: payments
      annotations:
        summary: "💳 Payment failure rate > 5%"
        description: "{{ $value | humanizePercentage }} payments failing"

  - name: infrastructure
    interval: 1m
    rules:
    - alert: HighMemoryPressure
      expr: |
        (
          1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)
        ) > 0.90
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Memory usage > 90%"

    - alert: DiskFillPrediction
      expr: |
        predict_linear(node_filesystem_avail_bytes[1h], 4*3600) < 0
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Disk will fill in ~4 hours"
        action: "Clean logs or expand disk"

    - alert: DatabaseConnectionPoolExhausted
      expr: |
        pg_stat_database_numbackends
        / pg_settings_max_connections
        > 0.8
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "DB connection pool 80%+ utilized"
```

**Шаг 6: Runbooks**

**docs/runbooks/high-error-rate.md**:

markdown

````markdown
# Runbook: High Error Rate

## Alert Details
- **Alert:** HighErrorRate  
- **Severity:** Critical (Page)
- **SLO Impact:** Availability

## Symptoms
- Error rate > 1% for 5+ minutes
- Users seeing 5xx errors
- Error budget burning fast

## Initial Response (First 5 minutes)

### 1. Acknowledge alert
```bash
# Silence alert while investigating
amtool silence add alertname=HighErrorRate --duration=30m --author=oncall --comment="Investigating"
```

### 2. Check overall system health
- Grafana: http://grafana.company.com/d/overview
- Look for: spike in errors, latency, resource usage

### 3. Identify affected service(s)
```promql
# Which service has errors?
topk(5, sum by (service) (rate(http_requests{status=~"5.."}[5m])))
```

### 4. Check recent changes
```bash
# Recent deployments
kubectl get events --sort-by='.lastTimestamp' | head -20

# Recent config changes
git log --since="1 hour ago" --oneline
```

## Diagnosis

### Check application logs
```logql
# Loki query
{service="api"} |= "ERROR" | json | line_format "{{.level}}: {{.message}}"
```

### Check traces
- Jaeger: http://jaeger.company.com
- Search for failing requests
- Look for slow/failing spans

### Check dependencies
```bash
# Database
pg_isready -h postgres-primary
SELECT count(*) FROM pg_stat_activity WHERE state = 'active';

# Redis
redis-cli -h redis ping

# External APIs
curl -I https://payment-gateway.external.com/health
```

## Common Causes & Solutions

### 1. Bad Deployment
**Symptoms:** Errors started after recent deploy

**Solution:**
```bash
# Immediate rollback
kubectl rollout undo deployment/api-service

# Verify
kubectl rollout status deployment/api-service
```

### 2. Database Issues
**Symptoms:** Slow queries, timeouts

**Solution:**
```sql
-- Check long-running queries
SELECT pid, age(clock_timestamp(), query_start), query
FROM pg_stat_activity
WHERE state = 'active' AND query_start < now() - interval '1 minute'
ORDER BY query_start;

-- Kill if needed
SELECT pg_terminate_backend(pid);
```

### 3. Resource Exhaustion
**Symptoms:** High CPU/memory, OOMKills

**Solution:**
```bash
# Scale up immediately
kubectl scale deployment/api-service --replicas=10

# Check resource usage
kubectl top pods
```

### 4. External API Failure
**Symptoms:** Timeout errors, circuit breaker open

**Solution:**
```bash
# Enable fallback/cache
kubectl set env deployment/api-service USE_CACHE=true

# Bypass failing dependency if non-critical
kubectl set env deployment/api-service FEATURE_X_ENABLED=false
```

## Mitigation Strategy

### Immediate (Stop the bleeding)
1. Rollback bad deployment
2. Scale up if resource constrained
3. Enable circuit breakers
4. Route to healthy instances

### Short-term (Stabilize)
1. Apply proper fix
2. Gradual rollout with monitoring
3. Load test before full deployment

### Long-term (Prevent)
1. Add pre-deployment tests
2. Improve monitoring/alerting
3. Implement gradual rollouts
4. Add chaos testing

## Verification

- [ ] Error rate < 0.1%
- [ ] Latency back to normal
- [ ] No active alerts
- [ ] Users not reporting issues

## Communication

### During incident
````

Slack: #incidents "Investigating high error rate on API service. ETA for resolution: 15 minutes. Status page: [https://status.company.com](https://status.company.com)"

```

### After resolution
```

"Issue resolved. Root cause: [X]. Total impact: [Y] minutes. Postmortem scheduled for [date]."

```

## Escalation Path
1. **L1** (0-5 min): On-call engineer
2. **L2** (5-15 min): Team lead
3. **L3** (15-30 min): Engineering manager
4. **L4** (30+ min): VP Engineering + CTO

## Postmortem
Schedule within 24 hours. Template: docs/postmortem-template.md

## Related Runbooks
- [Service Down](./service-down.md)
- [High Latency](./high-latency.md)
- [Database Issues](./database-issues.md)
```

### 🎯 Критерии успешной сдачи проекта

**Must Have (обязательно):**

- [ ]  Все сервисы запускаются одной командой
- [ ]  Prometheus собирает метрики со всех компонентов
- [ ]  3+ dashboard в Grafana (Business, Technical, Infrastructure)
- [ ]  Loki собирает логи в JSON формате
- [ ]  Distributed tracing работает через Jaeger/Tempo
- [ ]  10+ production alerts настроены
- [ ]  3+ SLO определены и измеряются
- [ ]  Runbooks для критических alerts
- [ ]  Load testing показывает стабильность
- [ ]  Documentation (README, architecture, SLOs)

**Nice to Have (дополнительно):**

- [ ]  Multi-environment (dev/staging/prod)
- [ ]  Automated remediation
- [ ]  Chaos engineering suite
- [ ]  Cost analysis dashboard
- [ ]  Security monitoring
- [ ]  Capacity planning dashboard
- [ ]  Custom exporters
- [ ]  Integration tests
- [ ]  Performance benchmarks
- [ ]  Postmortem examples

---

## 📚 Карьерный путь DevOps/SRE

### Junior DevOps/Monitoring Engineer (0-2 года)

**Навыки:**

- Linux basics
- Docker basics
- Basic monitoring (Prometheus, Grafana)
- Log aggregation basics
- Alert configuration
- Dashboard creation

**Зарплата:** $40k-70k

### Middle DevOps/SRE (2-4 года)

**Навыки:**

- Advanced Prometheus (recording rules, federation)
- Distributed tracing
- SLI/SLO management
- Incident response
- CI/CD integration
- Infrastructure as Code

**Зарплата:** $70k-120k

### Senior SRE (4-7 лет)

**Навыки:**

- System design для observability
- Multi-cloud monitoring
- Capacity planning
- Cost optimization
- Team leadership
- On-call strategy

**Зарплата:** $120k-180k

### Staff/Principal SRE (7+ лет)

**Навыки:**

- Organization-wide observability strategy
- Tooling development
- SLO framework design
- Incident management process
- Technical leadership

**Зарплата:** $180k-300k+

### Популярные вопросы на собеседованиях

**Технические:**

1. **Explain the difference between monitoring and observability**
2. **How would you monitor a microservices architecture?**
3. **What is high cardinality and why is it a problem?**
4. **Design an alerting strategy that avoids alert fatigue**
5. **How do you calculate error budget for 99.9% SLO?**
6. **Explain push vs pull monitoring models**
7. **How would you debug a memory leak in production?**
8. **What metrics would you track for a database?**

**Ситуационные:**

1. **Production is down, walk me through your process**
2. **You're getting 100 alerts per minute, what do you do?**
3. **Disk is 99% full but you can't find large files**
4. **Latency increased 10x after deployment, how to investigate?**
5. **Your monitoring system is down, how do you monitor?**

**System Design:**

1. **Design monitoring for a global CDN**
2. **Design alerting for 10,000 microservices**
3. **How would you monitor a mobile app backend?**

---

## 🏆 Финальный экзамен

### Часть 1: Теория (30 баллов)

**Вопрос 1 (10 баллов):** Объясни концепцию Error Budget и как его использовать для балансировки скорости разработки и надежности.

**Вопрос 2 (10 баллов):** Сравни USE и RED методологии. Когда использовать каждую?

**Вопрос 3 (10 баллов):** Что такое "high cardinality" в контексте метрик? Почему это проблема и как её решить?

### Часть 2: Практика (70 баллов)

**Задание 1: Incident Response (25 баллов)**

```
Сценарий:
- 02:00 AM: PagerDuty alert "API Error Rate High"
- Current error rate: 15% (normal: 0.1%)
- Last deployment: 4 hours ago
- Affected: Payment service

Задачи:
1. Напиши последовательность действий (first 10 minutes)
2. Какие метрики/логи/traces проверишь?
3. 3 наиболее вероятные причины
4. Mitigation strategy для каждой
5. Communication plan
```

**Задание 2: Monitoring Design (25 баллов)**

```
Спроектируй мониторинг для:
- Video streaming platform
- 10M users
- 1M concurrent streams
- Multi-region deployment

Определи:
1. Key metrics (минимум 15)
2. SLIs и SLOs (минимум 5)
3. Critical alerts (минимум 8)
4. Dashboard structure
5. Cost estimation
```

**Задание 3: PromQL Challenge (20 баллов)**

Напиши запросы для:

```
1. CPU usage per pod (excluding idle)
2. p99 latency for last 24 hours
3. Error rate by endpoint (last 5 min)
4. Predict when disk will be full
5. Cache hit ratio trending down
6. Requests per second by service
7. Top 5 slowest endpoints
8. Database connection pool utilization
9. Apdex score (T=300ms)
10. Memory usage forecast (next
```
