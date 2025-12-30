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
```

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
```

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

✅ Понимать различные подходы к логированию ✅ Настраивать Loki + Promtail + Grafana ✅ Писать LogQL запросы ✅ Парсить различные форматы логов ✅ Создавать дашборды для анализа логов ✅ Настраивать алерты на основе логов ✅ Управлять retention и rotation ✅ Сравнивать Loki и ELK стеки


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
            "issuetype": {"name": "Incident"}, "priority": {"name": "Critical"}, "labels": ["alert", "monitoring"] } }
```
# Отправка в Jira
response = requests.post(
    jira_url,
    json=ticket,
    auth=('user@example.com', 'jira-api-token'),
    headers={'Content-Type': 'application/json'}
)

print(f"Jira ticket created: {response.json().get('key')}")
```

if **name** == '**main**': app.run(host='0.0.0.0', port=5000)

````

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
````
```
