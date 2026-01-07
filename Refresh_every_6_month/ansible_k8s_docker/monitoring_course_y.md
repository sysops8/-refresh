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
import os
import json

# =========================
# OpenTelemetry setup
# =========================

resource = Resource.create({
    "service.name": os.getenv("OTEL_SERVICE_NAME", "backend"),
    "service.version": "1.0.0",
    "deployment.environment": "development",
})

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

# =========================
# Flask app
# =========================

app = Flask(__name__)

FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
Psycopg2Instrumentor().instrument()
RedisInstrumentor().instrument()

# =========================
# Connections
# =========================

DATABASE_URL = os.getenv(
    "DATABASE_URL",
    "postgresql://user:password@localhost:5432/demo",
)
REDIS_URL = os.getenv("REDIS_URL", "redis://localhost:6379")


def get_db_connection():
    return psycopg2.connect(DATABASE_URL)


def get_redis_connection():
    return redis.from_url(REDIS_URL)


# =========================
# DB init
# =========================

def init_db():
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
                    amount DECIMAL(10,2),
                    status VARCHAR(20),
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            """)
        conn.commit()


# =========================
# Routes
# =========================

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
    with tracer.start_as_current_span("get_user") as span:
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
            span.set_attribute("http.status_code", 404)
            return jsonify({"error": "User not found"}), 404

        return jsonify({
            "id": row[0],
            "name": row[1],
            "email": row[2],
        }), 200


@app.route("/api/users", methods=["POST"])
def create_user():
    with tracer.start_as_current_span("create_user") as span:
        data = request.json or {}

        if not data.get("name") or not data.get("email"):
            span.set_attribute("error", True)
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

        span.add_event("User created", {"user.id": user_id})

        return jsonify({
            "id": user_id,
            "name": data["name"],
            "email": data["email"],
        }), 201


@app.route("/api/orders", methods=["POST"])
def create_order():
    with tracer.start_as_current_span("create_order") as span:
        data = request.json or {}

        user_id = data.get("user_id")
        product = data.get("product")
        amount = data.get("amount")

        if not all([user_id, product, amount]):
            span.set_attribute("error", True)
            return jsonify({"error": "Missing required fields"}), 400

        # Check user
        with get_db_connection() as conn:
            with conn.cursor() as cur:
                cur.execute("SELECT id FROM users WHERE id = %s", (user_id,))
                if not cur.fetchone():
                    return jsonify({"error": "User not found"}), 404

        # Payment simulation
        delay = random.uniform(0.1, 0.5)
        time.sleep(delay)

        # Save order
        with get_db_connection() as conn:
            with conn.cursor() as cur:
                cur.execute("""
                    INSERT INTO orders (user_id, product, amount, status)
                    VALUES (%s, %s, %s, %s)
                    RETURNING id
                """, (user_id, product, amount, "completed"))
                order_id = cur.fetchone()[0]
            conn.commit()

        return jsonify({
            "id": order_id,
            "user_id": user_id,
            "product": product,
            "amount": amount,
            "status": "completed",
        }), 201


@app.route("/api/orders/<int:order_id>", methods=["GET"])
def get_order(order_id):
    with tracer.start_as_current_span("get_order") as span:
        span.set_attribute("order.id", order_id)
        time.sleep(random.uniform(0.01, 0.05))

        with get_db_connection() as conn:
            with conn.cursor() as cur:
                cur.execute("""
                    SELECT id, user_id, product, amount, status
                    FROM orders WHERE id = %s
                """, (order_id,))
                row = cur.fetchone()

        if not row:
            return jsonify({"error": "Order not found"}), 404

        return jsonify({
            "id": row[0],
            "user_id": row[1],
            "product": row[2],
            "amount": float(row[3]),
            "status": row[4],
        }), 200


@app.route("/api/slow")
def slow_endpoint():
    with tracer.start_as_current_span("slow_endpoint") as span:
        delay = random.uniform(2, 5)
        span.set_attribute("delay.seconds", delay)
        time.sleep(delay)
        return jsonify({"delay": delay}), 200


@app.route("/api/error")
def error_endpoint():
    with tracer.start_as_current_span("error_endpoint") as span:
        try:
            raise Exception("Simulated error")
        except Exception as e:
            span.record_exception(e)
            span.set_status(trace.Status(trace.StatusCode.ERROR, str(e)))
            return jsonify({"error": str(e)}), 500


# =========================
# Entry point
# =========================

if __name__ == "__main__":
    init_db()
    app.run(host="0.0.0.0", port=5000, debug=False)

```
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

## Модуль 7: Kubernetes Monitoring - мониторинг контейнеров и оркестрации (45 минут)

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
```
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
```
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

```yaml
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
```

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

```yaml
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

## Итоги модуля 7

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

```
Cluster:  Nodes ready, API latency, etcd health
Nodes:    CPU/Memory usage, disk pressure
Pods:     Restarts, OOM kills, phase
Workload: Replicas mismatch, HPA status
Network:  Traffic, errors, latency
```


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


## Модуль 8: SRE Практики - SLO/SLI/SLA и Error Budget (40 минут)

### 🎯 Напоминалка

**SRE (Site Reliability Engineering) - что это:**

```
SRE = Software Engineering + Operations

Цель: Баланс между скоростью разработки и надежностью системы

Ключевые принципы:
1. Измеряемая надежность (SLI/SLO/SLA)
2. Error Budget - можем рисковать
3. Автоматизация - toil reduction
4. Постмортемы без обвинений
5. Мониторинг как код
```

**SLI/SLO/SLA - разница:**

```
SLI (Service Level Indicator) - ЧТО мы измеряем
├─ Availability: % успешных запросов
├─ Latency: время ответа (p50, p95, p99)
├─ Throughput: запросов в секунду
├─ Durability: % сохраненных данных
└─ Correctness: % правильных ответов

SLO (Service Level Objective) - ЦЕЛЬ которую хотим достичь
├─ "99.9% запросов успешны"
├─ "95% запросов < 200ms"
├─ "99.95% данных сохранены"
└─ Внутренняя цель команды

SLA (Service Level Agreement) - КОНТРАКТ с клиентом
├─ "99.5% uptime гарантируем"
├─ Если нарушаем → штраф/компенсация
└─ SLA ≤ SLO (оставляем себе запас)

Правило: SLI ← SLO ← SLA
         Измерение ← Цель ← Договор
```

**Пример SLI/SLO/SLA:**

```
Сервис: API для платежей

SLI (измерения):
- Availability = successful_requests / total_requests
- Latency (p99) = 99-й перцентиль времени ответа
- Error rate = failed_requests / total_requests

SLO (цели):
- Availability ≥ 99.9% (monthly)
- Latency p99 ≤ 500ms (monthly)
- Error rate ≤ 0.1% (monthly)

SLA (контракт с клиентом):
- Availability ≥ 99.5% (monthly)
- Если < 99.5% → возврат 10% стоимости
- Если < 99.0% → возврат 25% стоимости
```

**Error Budget - революционная концепция:**

```
Error Budget = 100% - SLO

Пример:
SLO = 99.9% → Error Budget = 0.1%

Что это значит:
- 0.1% запросов МОГУТ фейлиться
- За месяц (30 дней) = 43 минуты downtime
- Это НЕ плохо - это БЮДЖЕТ на эксперименты!

Как использовать:
✅ Error budget есть → можем:
   - Деплоить чаще
   - Экспериментировать
   - Рисковать с новыми фичами

❌ Error budget исчерпан → нужно:
   - Freeze на новые фичи
   - Фокус на stability
   - Баг-фиксы и reliability work
```

**Error Budget calculation:**

```
Monthly SLO = 99.9%
Total time = 30 days = 43,200 minutes

Allowed downtime = 43,200 * (1 - 0.999) = 43.2 minutes

Текущий месяц:
- Прошло: 15 дней
- Downtime: 20 минут
- Budget used: 20 / 43.2 = 46.3%
- Budget remaining: 53.7% (23.2 минуты)

Статус: ✅ OK - можем продолжать деплоить
```

**Burn Rate - скорость сжигания бюджета:**

```
Burn Rate = как быстро тратим error budget

Normal rate = 1.0
- Тратим ровно столько, сколько заложено
- 1% бюджета в час при 100 часах в бюджете

High burn rate > 1.0 - проблема!
- Burn rate = 10 → бюджет закончится за 1/10 времени
- Нужны немедленные действия

Пример:
30-day budget = 43.2 minutes
Current burn rate = 5.0
→ Бюджет закончится за 6 дней вместо 30!
```

**Multi-window multi-burn-rate alerts:**

```
Идея от Google SRE:
Разные окна для разных burn rates

Fast burn (критично):
- 5min window + 1h window
- Burn rate > 14.4
- Alert: страница в PagerDuty
- Действие: немедленное

Slow burn (важно):
- 30min window + 6h window
- Burn rate > 6
- Alert: ticket в Jira
- Действие: в течение дня

Пример rule:
(
  error_rate[5m] > 14.4 * (1 - SLO)  # 5-минутное окно
  AND
  error_rate[1h] > 14.4 * (1 - SLO)  # 1-часовое окно
)
→ Page on-call engineer
```

**SLO tiers (разные сервисы = разные SLO):**

```
Tier 1 - Critical (99.95%)
├─ Payment processing
├─ Authentication
└─ User-facing API

Tier 2 - Important (99.9%)
├─ Analytics API
├─ Recommendations
└─ Search

Tier 3 - Best effort (99.0%)
├─ Admin panel
├─ Internal tools
└─ Batch jobs

Tier 4 - Eventual (95.0%)
├─ Data backfill
├─ ML training
└─ Reports
```

**SLO document template:**

yaml

````yaml
service: payment-api
owner: payments-team
tier: tier-1

slo:
  availability:
    target: 99.95%
    window: 30d
    measurement:
      sli: |
        sum(rate(http_requests_total{status!~"5.."}[5m]))
        /
        sum(rate(http_requests_total[5m]))
      
  latency:
    target: 95%  # 95% запросов < 200ms
    threshold: 200ms
    window: 30d
    measurement:
      sli: |
        histogram_quantile(0.95,
          rate(http_request_duration_seconds_bucket[5m])
        )

error_budget:
  policy:
    - remaining > 50%: Normal deployment cadence
    - remaining 25-50%: Review each deployment
    - remaining < 25%: Freeze deployments, focus on reliability
    - remaining < 0%: Incident, all hands on deck

dependencies:
  - service: auth-service
    criticality: hard  # Cannot work without it
  - service: cache-service
    criticality: soft  # Degraded without it

alerts:
  - name: ErrorBudgetFastBurn
    severity: page
    condition: burn_rate[5m] > 14.4 AND burn_rate[1h] > 14.4
  
  - name: ErrorBudgetSlowBurn
    severity: ticket
    condition: burn_rate[30m] > 6 AND burn_rate[6h] > 6

runbook: https://wiki.company.com/runbooks/payment-api
dashboard: https://grafana.company.com/d/payment-api
````

**Choosing good SLIs:**
```
✅ Good SLI:
- Измеряет user experience
- Легко измерить
- Имеет бизнес-impact
- Можно влиять на него

❌ Bad SLI:
- Внутренняя метрика (queue depth, CPU)
- Не связана с user experience
- Не можем контролировать
- Слишком сложно измерить

Примеры:

✅ "95% search queries return in < 100ms"
   → User experience, measurable, controllable

❌ "Average server CPU < 70%"
   → Internal metric, не связано с UX

✅ "99.9% of payments processed successfully"
   → Business impact, clear

❌ "Database connection pool utilization < 80%"
   → Internal, не user-facing
```

**SLO для разных типов сервисов:**
```
Request/Response (API):
- Availability: % успешных запросов
- Latency: p50, p95, p99
- Throughput: requests/sec

Data Processing (Pipeline):
- Freshness: задержка обработки
- Coverage: % обработанных данных
- Correctness: % правильных результатов

Storage:
- Durability: % сохраненных данных
- Availability: % успешных read/write
- Latency: time to first byte

Batch Jobs:
- Success rate: % успешных jobs
- Throughput: records/hour
- Freshness: время до completion
```

**Toil - что это и как измерить:**
```
Toil = Ручная, повторяющаяся, автоматизируемая работа

Признаки toil:
✓ Manual - требует человека
✓ Repetitive - делается снова и снова
✓ Automatable - можно автоматизировать
✓ Tactical - не стратегическая работа
✓ No enduring value - не создает ценность
✓ Linear scaling - растет с нагрузкой

Примеры toil:
- Ручной restart сервисов
- Очистка логов вручную
- Ручное scaling
- Checking alerts manually
- Ручной deployment

Цель SRE: < 50% времени на toil

Измерение:
- Трекай время на каждую задачу
- Категоризуй: engineering vs toil
- Target: 50% engineering work
```

**Постмортем (Post-mortem) - учимся на ошибках:**
```
Правило #1: Blameless - не виним людей!

Структура постмортема:

1. Summary
   - Что случилось (1-2 предложения)
   - Impact: затронутые пользователи, downtime
   - Начало: время старта инцидента
   - Конец: время разрешения
   - Duration: общее время

2. Timeline
   - [10:00] Deploy v2.3.4
   - [10:15] Error rate spike to 15%
   - [10:20] Rollback initiated
   - [10:25] Error rate back to normal

3. Root Cause
   - Техническая причина (не "кто-то ошибся")
   - Configuration bug in new version
   - Missing validation in deployment pipeline

4. Impact
   - 15 minutes of degraded service
   - 5% of users affected
   - 0 data loss
   - $5000 estimated revenue loss

5. What Went Well
   - Fast detection (5 minutes)
   - Clear runbook followed
   - Good communication

6. What Went Wrong
   - No canary deployment
   - Testing didn't catch the bug
   - Monitoring delayed alert

7. Action Items
   - [P0] Add canary stage to pipeline (Owner: Alice, ETA: 1 week)
   - [P1] Improve test coverage (Owner: Bob, ETA: 2 weeks)
   - [P2] Tune alert thresholds (Owner: Carol, ETA: 3 days)

8. Lessons Learned
   - Always canary deploy
   - Integration tests needed
   - Monitoring can be improved
```

**On-call rotation best practices:**
```
Структура:
- Primary on-call: первая линия
- Secondary on-call: backup
- Escalation: если primary не отвечает

Shifts:
- 7 дней на человека (не больше!)
- Handoff meeting: передача контекста
- Follow-the-sun: разные timezone

Compensation:
- Компенсация за on-call время
- Time off после тяжелого инцидента
- Rotation должен быть fair

Tools:
- PagerDuty / Opsgenie / VictorOps
- Clear escalation policy
- Runbooks для всех alerts

Health:
- Limit pages: < 2 per night acceptable
- If more → система нездорова
- Fix или adjust alerts
```

**Reliability hierarchy:**
```
Level 4: Self-healing systems
         ├─ Auto-remediation
         └─ Zero human intervention

Level 3: Automated response
         ├─ Auto-scaling
         ├─ Auto-rollback
         └─ Circuit breakers

Level 2: Good observability
         ├─ Metrics, logs, traces
         ├─ Dashboards
         └─ Alerts with runbooks

Level 1: Manual operations
         ├─ SSH into servers
         ├─ Manual restarts
         └─ No monitoring

Target: Level 3-4 для production систем
```

**Capacity planning:**
```
Правило: Plan for 2x current peak

Steps:
1. Measure current usage
   - Peak requests/sec
   - Peak CPU/Memory
   - Peak disk I/O

2. Forecast growth
   - Historical trends
   - Business projections
   - Seasonal patterns

3. Calculate headroom
   - Current capacity: 1000 req/s
   - Peak usage: 600 req/s
   - Headroom: 40%
   - Target: 50%+ headroom

4. Plan ahead
   - When hits 80% → order more capacity
   - Lead time: 3-6 months for hardware
   - Cloud: easier, but still plan

5. Load testing
   - Regular load tests
   - Verify capacity estimates
   - Find bottlenecks early
```

**Change management:**
```
Change Types:

Low risk:
- Config updates (reviewed)
- Scaling up
- Monitoring changes
→ Normal approval

Medium risk:
- Code deployments
- Database schema changes
- Infrastructure updates
→ Change review + testing

High risk:
- Data migrations
- Major architecture changes
- Dependency upgrades
→ Change advisory board + extensive testing

Change windows:
- Production: Tue-Thu, 10am-4pm
- No Fridays (weekend recovery)
- No holidays
- Emergency: any time (with approval)

Rollback plan:
- Every change needs rollback plan
- Test rollback procedure
- Time limit: can rollback in < 10 min
```

**Golden Signals (4 основные метрики):**
```
1. Latency (Задержка)
   - Время обработки запроса
   - p50, p95, p99
   - Separate success vs error latency

2. Traffic (Трафик)
   - Запросов в секунду
   - Transactions per second
   - Network I/O

3. Errors (Ошибки)
   - Rate of failed requests
   - 5xx errors, timeouts
   - % of total requests

4. Saturation (Насыщение)
   - Resource utilization
   - CPU, Memory, Disk
   - Queue depth, thread pool

If you can only monitor 4 things, monitor these!
````

**PromQL для SLO:**

promql

```promql
# Availability SLI
sum(rate(http_requests_total{status!~"5.."}[30d]))
/
sum(rate(http_requests_total[30d]))

# Error budget remaining (30 days)
1 - (
  (1 - availability_sli)  # actual error rate
  /
  (1 - 0.999)             # target error rate (SLO = 99.9%)
)

# Burn rate (5 minutes)
(
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
)
/
(1 - 0.999)  # Normalize to SLO

# Latency SLI (95% < 200ms)
histogram_quantile(0.95,
  sum(rate(http_request_duration_seconds_bucket[30d])) by (le)
) < 0.2

# Error budget consumption rate
rate(http_requests_total{status=~"5.."}[1h])
/
(
  sum(rate(http_requests_total[30d])) * (1 - 0.999) / 24 / 30
)
# Если > 1.0 → тратим быстрее, чем планировали
```

### 💻 Задание

Создай полноценную SRE систему с SLO мониторингом:

1. **Создай SLO configuration**:

`slo-config.yaml`:

yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: slo-config
  namespace: monitoring
data:
  slo-config.yaml: |
    # Payment API SLO
    payment-api:
      tier: tier-1
      owner: payments-team
      
      availability:
        target: 0.999  # 99.9%
        window: 30d
        sli_query: |
          sum(rate(http_requests_total{service="payment-api",status!~"5.."}[5m]))
          /
          sum(rate(http_requests_total{service="payment-api"}[5m]))
      
      latency:
        target: 0.95  # 95% requests < 200ms
        threshold_ms: 200
        window: 30d
        sli_query: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket{service="payment-api"}[5m])) by (le)
          )
      
      error_budget_policy:
        - remaining: "> 75%"
          action: "Normal operations"
          deployment_frequency: "Multiple per day"
        
        - remaining: "50-75%"
          action: "Cautious"
          deployment_frequency: "Once per day"
        
        - remaining: "25-50%"
          action: "Review each deployment"
          deployment_frequency: "As needed only"
        
        - remaining: "< 25%"
          action: "Freeze deployments"
          deployment_frequency: "Emergency fixes only"
        
        - remaining: "< 0%"
          action: "Incident mode"
          deployment_frequency: "Reliability work only"
    
    # User API SLO
    user-api:
      tier: tier-2
      owner: user-team
      
      availability:
        target: 0.99  # 99%
        window: 30d
        sli_query: |
          sum(rate(http_requests_total{service="user-api",status!~"5.."}[5m]))
          /
          sum(rate(http_requests_total{service="user-api"}[5m]))
      
      latency:
        target: 0.90  # 90% requests < 500ms
        threshold_ms: 500
        window: 30d
        sli_query: |
          histogram_quantile(0.90,
            sum(rate(http_request_duration_seconds_bucket{service="user-api"}[5m])) by (le)
          )
```

2. **Создай Prometheus recording rules для SLO**:

`k8s-manifests/slo-recording-rules.yaml`:

yaml

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: slo-recording-rules
  namespace: monitoring
  labels:
    prometheus: kube-prometheus
    role: slo-rules
spec:
  groups:
  - name: slo.rules
    interval: 30s
    rules:
    # Payment API SLI
    - record: sli:availability:payment_api
      expr: |
        sum(rate(http_requests_total{service="payment-api",status!~"5.."}[5m]))
        /
        sum(rate(http_requests_total{service="payment-api"}[5m]))
    
    - record: sli:latency:payment_api:p95
      expr: |
        histogram_quantile(0.95,
          sum(rate(http_request_duration_seconds_bucket{service="payment-api"}[5m])) by (le)
        )
    
    - record: sli:latency:payment_api:p99
      expr: |
        histogram_quantile(0.99,
          sum(rate(http_request_duration_seconds_bucket{service="payment-api"}[5m])) by (le)
        )
    
    # Error Budget (30 days)
    - record: slo:error_budget:payment_api:30d
      expr: |
        1 - (
          (
            1 - (
              sum(rate(http_requests_total{service="payment-api",status!~"5.."}[30d]))
              /
              sum(rate(http_requests_total{service="payment-api"}[30d]))
            )
          )
          /
          (1 - 0.999)  # SLO target
        )
    
    # Burn Rate (multiple windows)
    - record: slo:burn_rate:payment_api:5m
      expr: |
        (
          sum(rate(http_requests_total{service="payment-api",status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total{service="payment-api"}[5m]))
        )
        /
        (1 - 0.999)
    
    - record: slo:burn_rate:payment_api:1h
      expr: |
        (
          sum(rate(http_requests_total{service="payment-api",status=~"5.."}[1h]))
          /
          sum(rate(http_requests_total{service="payment-api"}[1h]))
        )
        /
        (1 - 0.999)
    
    - record: slo:burn_rate:payment_api:6h
      expr: |
        (
          sum(rate(http_requests_total{service="payment-api",status=~"5.."}[6h]))
          /
          sum(rate(http_requests_total{service="payment-api"}[6h]))
        )
        /
        (1 - 0.999)
    
    # Time to exhaustion (hours)
    - record: slo:error_budget:payment_api:time_to_exhaustion_hours
      expr: |
        (
          slo:error_budget:payment_api:30d
          /
          max_over_time(slo:burn_rate:payment_api:1h[1h])
        ) * 24 * 30
```

3. **Создай SLO alert rules**:

`k8s-manifests/slo-alert-rules.yaml`:

yaml

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: slo-alert-rules
  namespace: monitoring
  labels:
    prometheus: kube-prometheus
    role: slo-alerts
spec:
  groups:
  - name: slo.alerts
    interval: 30s
    rules:
    # Fast burn - Page immediately
    - alert: ErrorBudgetFastBurn
      expr: |
        slo:burn_rate:payment_api:5m > 14.4
        and
        slo:burn_rate:payment_api:1h > 14.4
      for: 2m
      labels:
        severity: page
        service: payment-api
        slo_type: availability
      annotations:
        summary: "Payment API burning error budget too fast"
        description: |
          Error budget will be exhausted in {{ $value | humanizeDuration }}.
          Current burn rate: {{ printf "%.2f" $value }}x
          5m error rate: {{ with query "rate(http_requests_total{service='payment-api',status=~'5..'}[5m]) / rate(http_requests_total{service='payment-api'}[5m])" }}{{ . | first | value | humanizePercentage }}{{ end }}
        dashboard: "http://grafana:3000/d/slo-payment-api"
        runbook: "https://wiki.example.com/runbooks/error-budget-fast-burn"
        playbook: |
          1. Check recent deployments
          2. Check error logs
          3. Consider rollback if deployment related
          4. Page team lead if not resolved in 15 min
    
    # Slow burn - Create ticket
    - alert: ErrorBudgetSlowBurn
      expr: |
        slo:burn_rate:payment_api:30m > 6
        and
        slo:burn_rate:payment_api:6h > 6
      for: 15m
      labels:
        severity: ticket
        service: payment-api
        slo_type: availability
      annotations:
        summary: "Payment API burning error budget steadily"
        description: |
          Error budget will be exhausted in {{ $value | humanizeDuration }}.
          Current burn rate: {{ printf "%.2f" $value }}x
          30m error rate: {{ with query "rate(http_requests_total{service='payment-api',status=~'5..'}[30m]) / rate(http_requests_total{service='payment-api'}[30m])" }}{{ . | first | value | humanizePercentage }}{{ end }}
        dashboard: "http://grafana:3000/d/slo-payment-api"
    
    # Error budget nearly exhausted
    - alert: ErrorBudgetNearlyExhausted
      expr: |
        slo:error_budget:payment_api:30d < 0.25
        and
        slo:error_budget:payment_api:30d > 0
      for: 5m
      labels:
        severity: warning
        service: payment-api
      annotations:
        summary: "Payment API error budget nearly exhausted"
        description: |
          Only {{ $value | humanizePercentage }} of 30-day error budget remaining.
          Consider deployment freeze and focus on reliability.
        dashboard: "http://grafana:3000/d/slo-payment-api"
    
    # Error budget exhausted - deployment freeze!
    - alert: ErrorBudgetExhausted
      expr: |
        slo:error_budget:payment_api:30d <= 0
      for: 5m
      labels:
        severity: critical
        service: payment-api
      annotations:
        summary: "Payment API error budget EXHAUSTED"
        description: |
          30-day error budget is exhausted!
          FREEZE all deployments except emergency fixes.
          Focus on reliability work only.
        dashboard: "http://grafana:3000/d/slo-payment-api"
        runbook: "https://wiki.example.com/runbooks/error-budget-exhausted"
    
    # Latency SLO violation
    - alert: LatencySLOViolation
      expr: |
        sli:latency:payment_api:p95 > 0.2  # 200ms
      for: 10m
      labels:
        severity: warning
        service: payment-api
        slo_type: latency
      annotations:
        summary: "Payment API latency SLO violation"
        description: |
          P95 latency is {{ $value | humanizeDuration }}, exceeding 200ms target.
        dashboard: "http://grafana:3000/d/slo-payment-api"
    
    # SLO at risk (trending)
    - alert: SLOAtRisk
      expr: |
        predict_linear(slo:error_budget:payment_api:30d[6h], 7*24*3600) < 0
      for: 1h
      labels:
        severity: warning
        service: payment-api
      annotations:
        summary: "Payment API SLO at risk"
        description: |
          Based on current trends, error budget will be exhausted in < 7 days.
          Current remaining: {{ $value | humanizePercentage }}
        dashboard: "http://grafana:3000/d/slo-payment-api"
```

4. **Создай demo app с настраиваемой надежностью**:

`demo-app/sre-demo-app.py`:

python

```python
from flask import Flask, request, jsonify
from prometheus_client import Counter, Histogram, Gauge, generate_latest
import random
import time
import os

# =========================
# App
# =========================

app = Flask(__name__)

# =========================
# Prometheus metrics
# =========================

REQUEST_COUNT = Counter(
    "http_requests_total",
    "Total HTTP requests",
    ["method", "endpoint", "status", "service"],
)

REQUEST_DURATION = Histogram(
    "http_request_duration_seconds",
    "HTTP request duration",
    ["method", "endpoint", "service"],
    buckets=[0.01, 0.05, 0.1, 0.2, 0.5, 1.0, 2.0, 5.0],
)

ERROR_BUDGET = Gauge(
    "error_budget_remaining",
    "Remaining error budget (0-1)",
    ["service", "window"],
)

# =========================
# Config
# =========================

SERVICE_NAME = os.getenv("SERVICE_NAME", "payment-api")
ERROR_RATE = float(os.getenv("ERROR_RATE", "0.001"))  # 0.1%
SLOW_REQUEST_RATE = float(os.getenv("SLOW_REQUEST_RATE", "0.05"))  # 5%
SLOW_REQUEST_DURATION = float(os.getenv("SLOW_REQUEST_DURATION", "2.0"))  # seconds

# =========================
# Routes
# =========================

@app.route("/health")
def health():
    return jsonify({"status": "healthy"}), 200


@app.route("/api/payment", methods=["POST"])
def process_payment():
    start_time = time.time()

    try:
        # Simulate error
        if random.random() < ERROR_RATE:
            REQUEST_COUNT.labels(
                method="POST",
                endpoint="/api/payment",
                status="500",
                service=SERVICE_NAME,
            ).inc()
            return jsonify({"error": "Payment processing failed"}), 500

        # Simulate latency
        if random.random() < SLOW_REQUEST_RATE:
            time.sleep(SLOW_REQUEST_DURATION)
        else:
            time.sleep(random.uniform(0.01, 0.1))

        REQUEST_COUNT.labels(
            method="POST",
            endpoint="/api/payment",
            status="200",
            service=SERVICE_NAME,
        ).inc()

        return jsonify({
            "status": "success",
            "transaction_id": f"txn_{random.randint(1000, 9999)}",
        }), 200

    finally:
        duration = time.time() - start_time
        REQUEST_DURATION.labels(
            method="POST",
            endpoint="/api/payment",
            service=SERVICE_NAME,
        ).observe(duration)


@app.route("/api/user/<user_id>")
def get_user(user_id):
    start_time = time.time()

    try:
        # Simulate error (lower rate)
        if random.random() < ERROR_RATE * 0.5:
            REQUEST_COUNT.labels(
                method="GET",
                endpoint="/api/user",
                status="500",
                service=SERVICE_NAME,
            ).inc()
            return jsonify({"error": "User not found"}), 500

        time.sleep(random.uniform(0.01, 0.05))

        REQUEST_COUNT.labels(
            method="GET",
            endpoint="/api/user",
            status="200",
            service=SERVICE_NAME,
        ).inc()

        return jsonify({
            "user_id": user_id,
            "name": f"User {user_id}",
            "email": f"user{user_id}@example.com",
        }), 200

    finally:
        duration = time.time() - start_time
        REQUEST_DURATION.labels(
            method="GET",
            endpoint="/api/user",
            service=SERVICE_NAME,
        ).observe(duration)


# =========================
# Chaos endpoints
# =========================

@app.route("/chaos/increase_errors")
def increase_errors():
    """Simulate incident by increasing error rate"""
    global ERROR_RATE
    ERROR_RATE = min(ERROR_RATE * 2, 0.5)

    return jsonify({
        "message": "Error rate increased",
        "new_rate": ERROR_RATE,
    }), 200


@app.route("/chaos/decrease_errors")
def decrease_errors():
    """Recover from incident"""
    global ERROR_RATE
    ERROR_RATE = max(ERROR_RATE / 2, 0.001)

    return jsonify({
        "message": "Error rate decreased",
        "new_rate": ERROR_RATE,
    }), 200


@app.route("/chaos/slow_down")
def slow_down():
    """Simulate performance degradation"""
    global SLOW_REQUEST_RATE, SLOW_REQUEST_DURATION

    SLOW_REQUEST_RATE = min(SLOW_REQUEST_RATE * 2, 0.5)
    SLOW_REQUEST_DURATION = min(SLOW_REQUEST_DURATION * 1.5, 10.0)

    return jsonify({
        "message": "Service slowed down",
        "slow_rate": SLOW_REQUEST_RATE,
        "slow_duration": SLOW_REQUEST_DURATION,
    }), 200


@app.route("/chaos/speed_up")
def speed_up():
    """Recover performance"""
    global SLOW_REQUEST_RATE, SLOW_REQUEST_DURATION

    SLOW_REQUEST_RATE = max(SLOW_REQUEST_RATE / 2, 0.01)
    SLOW_REQUEST_DURATION = max(SLOW_REQUEST_DURATION / 1.5, 0.1)

    return jsonify({
        "message": "Service sped up",
        "slow_rate": SLOW_REQUEST_RATE,
        "slow_duration": SLOW_REQUEST_DURATION,
    }), 200


# =========================
# Metrics
# =========================

@app.route("/metrics")
def metrics():
    return generate_latest(), 200, {
        "Content-Type": "text/plain; charset=utf-8",
    }


# =========================
# Entry point
# =========================

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

`demo-app/Dockerfile`:
````dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN pip install flask prometheus-client

COPY sre-demo-app.py .

EXPOSE 5000

CMD ["python", "sre-demo-app.py"]
````

5. **Deploy demo app в Kubernetes**:

`k8s-manifests/sre-demo-app.yaml`:
````yaml
apiVersion: v1
kind: Namespace
metadata:
  name: sre-demo

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-api
  namespace: sre-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment-api
  template:
    metadata:
      labels:
        app: payment-api
        service: payment-api
    spec:
      containers:
      - name: app
        image: sre-demo-app:latest
        imagePullPolicy: Never  # For local kind cluster
        ports:
        - containerPort: 5000
        env:
        - name: SERVICE_NAME
          value: "payment-api"
        - name: ERROR_RATE
          value: "0.001"  # 0.1%
        - name: SLOW_REQUEST_RATE
          value: "0.05"    # 5%
        - name: SLOW_REQUEST_DURATION
          value: "2.0"
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: payment-api
  namespace: sre-demo
  labels:
    app: payment-api
spec:
  selector:
    app: payment-api
  ports:
  - port: 5000
    targetPort: 5000
    name: http

---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: payment-api
  namespace: sre-demo
spec:
  selector:
    matchLabels:
      app: payment-api
  endpoints:
  - port: http
    path: /metrics
    interval: 15s
````

Build and load image to kind:
````bash
# Build image
cd demo-app
docker build -t sre-demo-app:latest .

# Load to kind cluster
kind load docker-image sre-demo-app:latest --name monitoring-cluster

# Deploy
kubectl apply -f ../k8s-manifests/sre-demo-app.yaml
kubectl apply -f ../k8s-manifests/slo-recording-rules.yaml
kubectl apply -f ../k8s-manifests/slo-alert-rules.yaml

# Check
kubectl get pods -n sre-demo
kubectl logs -n sre-demo -l app=payment-api
````

6. **Создай load generator**:

`load-generator.sh`:
````bash
#!/bin/bash

PAYMENT_API="http://localhost:30003"  # Adjust NodePort

echo "Starting load generation..."
echo "Payment API: $PAYMENT_API"

# Normal load
normal_load() {
    while true; do
        curl -s -X POST $PAYMENT_API/api/payment \
          -H "Content-Type: application/json" \
          -d '{"amount": 100, "currency": "USD"}' \
          > /dev/null 2>&1
        
        sleep 0.1
    done
}

# Start multiple workers
for i in {1..5}; do
    normal_load &
done

echo "Load generation started (5 workers)"
echo "Press Ctrl+C to stop"

wait
````

7. **Создай Grafana dashboard для SLO**:

Import в Grafana (`slo-dashboard.json`):
````json
{
  "dashboard": {
    "title": "SLO Dashboard - Payment API",
    "tags": ["slo", "sre", "payment-api"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "gridPos": {"h": 6, "w": 8, "x": 0, "y": 0},
        "type": "stat",
        "title": "Availability SLI (30d)",
        "targets": [
          {
            "expr": "sli:availability:payment_api",
            "legendFormat": "Availability"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percentunit",
            "thresholds": {
              "steps": [
                {"value": 0, "color": "red"},
                {"value": 0.995, "color": "yellow"},
                {"value": 0.999, "color": "green"}
              ]
            },
            "mappings": [],
            "min": 0.99,
            "max": 1.0
          }
        }
      },
      {
        "id": 2,
        "gridPos": {"h": 6, "w": 8, "x": 8, "y": 0},
        "type": "stat",
        "title": "Error Budget Remaining",
        "targets": [
          {
            "expr": "slo:error_budget:payment_api:30d",
            "legendFormat": "Budget"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percentunit",
            "thresholds": {
              "steps": [
                {"value": 0, "color": "red"},
                {"value": 0.25, "color": "orange"},
                {"value": 0.5, "color": "yellow"},
                {"value": 0.75, "color": "green"}
              ]
            },
            "min": 0,
            "max": 1.0
          }
        }
      },
      {
        "id": 3,
        "gridPos": {"h": 6, "w": 8, "x": 16, "y": 0},
        "type": "stat",
        "title": "P95 Latency",
        "targets": [
          {
            "expr": "sli:latency:payment_api:p95",
            "legendFormat": "P95"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "s",
            "thresholds": {
              "steps": [
                {"value": 0, "color": "green"},
                {"value": 0.2, "color": "yellow"},
                {"value": 0.5, "color": "red"}
              ]
            }
          }
        }
      },
      {
        "id": 4,
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 6},
        "type": "timeseries",
        "title": "Error Budget Over Time",
        "targets": [
          {
            "expr": "slo:error_budget:payment_api:30d",
            "legendFormat": "Error Budget"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percentunit",
            "custom": {
              "lineWidth": 2
            }
          },
          "overrides": [
            {
              "matcher": {"id": "byName", "options": "Error Budget"},
              "properties": [
                {
                  "id": "color",
                  "value": {"mode": "thresholds"}
                },
                {
                  "id": "thresholds",
                  "value": {
                    "steps": [
                      {"value": 0, "color": "red"},
                      {"value": 0.25, "color": "orange"},
                      {"value": 0.5, "color": "yellow"},
                      {"value": 0.75, "color": "green"}
                    ]
                  }
                }
              ]
            }
          ]
        }
      },
      {
        "id": 5,
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 6},
        "type": "timeseries",
        "title": "Burn Rate (Multiple Windows)",
        "targets": [
          {
            "expr": "slo:burn_rate:payment_api:5m",
            "legendFormat": "5m window"
          },
          {
            "expr": "slo:burn_rate:payment_api:1h",
            "legendFormat": "1h window"
          },
          {
            "expr": "slo:burn_rate:payment_api:6h",
            "legendFormat": "6h window"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "custom": {
              "lineWidth": 2
            }
          }
        }
      },
      {
        "id": 6,
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 14},
        "type": "timeseries",
        "title": "Request Rate and Error Rate",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{service='payment-api'}[5m]))",
            "legendFormat": "Total Requests/s"
          },
          {
            "expr": "sum(rate(http_requests_total{service='payment-api',status=~'5..'}[5m]))",
            "legendFormat": "Errors/s"
          }
        ]
      },
      {
        "id": 7,
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 22},
        "type": "timeseries",
        "title": "Latency Percentiles",
        "targets": [
          {
            "expr": "sli:latency:payment_api:p95",
            "legendFormat": "P95"
          },
          {
            "expr": "sli:latency:payment_api:p99",
            "legendFormat": "P99"
          },
          {
            "expr": "histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket{service='payment-api'}[5m])) by (le))",
            "legendFormat": "P50"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "s"
          }
        }
      }
    ],
    "refresh": "30s"
  }
}
````

8. **Создай инструменты для incident simulation**:

`simulate-incident.sh`:
````bash
#!/bin/bash

API_URL="http://localhost:30003"  # Adjust

case "$1" in
  "start-error-spike")
    echo "🔥 Starting error spike incident..."
    curl -s $API_URL/chaos/increase_errors
    echo "Error rate increased. Watch SLO dashboard!"
    ;;
  
  "stop-error-spike")
    echo "✅ Recovering from error spike..."
    curl -s $API_URL/chaos/decrease_errors
    echo "Error rate decreased."
    ;;
  
  "start-latency-spike")
    echo "🐌 Starting latency spike incident..."
    curl -s $API_URL/chaos/slow_down
    echo "Service slowed down. Watch latency metrics!"
    ;;
  
  "stop-latency-spike")
    echo "⚡ Recovering from latency spike..."
    curl -s $API_URL/chaos/speed_up
    echo "Service sped up."
    ;;
  
  "burndown-test")
    echo "🔥🔥🔥 BURN DOWN TEST - This will exhaust error budget!"
    echo "Are you sure? (yes/no)"
    read confirmation
    if [ "$confirmation" = "yes" ]; then
      # Dramatically increase errors
      for i in {1..5}; do
        curl -s $API_URL/chaos/increase_errors
      done
      echo "Error budget burning fast! Watch alerts fire!"
    fi
    ;;
  
  *)
    echo "Usage: $0 {start-error-spike|stop-error-spike|start-latency-spike|stop-latency-spike|burndown-test}"
    exit 1
    ;;
esac
````

9. **Тестирование SLO системы**:
````bash
# Expose service
kubectl port-forward -n sre-demo svc/payment-api 30003:5000 &

# Start load
./load-generator.sh &

# Watch SLO metrics
watch -n 5 'kubectl exec -n monitoring prometheus-kube-prometheus-prometheus-0 -- promtool query instant http://localhost:9090 "slo:error_budget:payment_api:30d"'

# Simulate incident
./simulate-incident.sh start-error-spike

# Watch alerts fire
kubectl logs -n monitoring -l app.kubernetes.io/name=alertmanager -f

# Check Grafana dashboard
open http://localhost:3000/d/slo-payment-api

# Recover
./simulate-incident.sh stop-error-spike

# Check error budget recovered
````

### 🚀 Бонус (новое)

**1. Создай SLO report generator**:

`slo-report.py`:
````python
#!/usr/bin/env python3
"""
SLO Monthly Report Generator
"""
import requests
from datetime import datetime, timedelta
import json

PROMETHEUS_URL = "http://localhost:9090"

def query_prometheus(query):
    """Query Prometheus"""
    response = requests.get(
        f"{PROMETHEUS_URL}/api/v1/query",
        params={'query': query}
    )
    return response.json()['data']['result']

def generate_slo_report(service_name, days=30):
    """Generate SLO report for a service"""
    
    print(f"=" * 60)
    print(f"SLO REPORT: {service_name}")
    print(f"Period: Last {days} days")
    print(f"Generated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print(f"=" * 60)
    print()
    
    # Availability SLI
    availability_query = f'sli:availability:{service_name}'
    availability_result = query_prometheus(availability_query)
    if availability_result:
        availability = float(availability_result[0]['value'][1])
        print(f"📊 Availability SLI: {availability*100:.3f}%")
        print(f"   Target: 99.900%")
        if availability >= 0.999:
            print(f"   Status: ✅ Meeting SLO")
        else:
            print(f"   Status: ❌ Violating SLO")
        print()
    
    # Error Budget
    budget_query = f'slo:error_budget:{service_name}:30d'
    budget_result = query_prometheus(budget_query)
    if budget_result:
        budget = float(budget_result[0]['value'][1])
        print(f"💰 Error Budget Remaining: {budget*100:.1f}%")
        
        if budget > 0.75:
            status = "✅ Healthy - Normal operations"
        elif budget > 0.5:
            status = "⚠️  Caution - Review deployments"
        elif budget > 0.25:
            status = "🔶 Warning - Slow down deployments"
        elif budget > 0:
            status = "🔴 Critical - Freeze deployments"
        else:
            status = "💀 EXHAUSTED - Incident mode"
        
        print(f"   Status: {status}")
        print()
    
    # Latency
    latency_p95_query = f'sli:latency:{service_name}:p95'
    latency_p95_result = query_prometheus(latency_p95_query)
    if latency_p95_result:
        latency_p95 = float(latency_p95_result[0]['value'][1])
        print(f"⏱️  Latency P95: {latency_p95*1000:.0f}ms")
        print(f"   Target: 200ms")
        if latency_p95 <= 0.2:
            print(f"   Status: ✅ Meeting SLO")
        else:
            print(f"   Status: ❌ Violating SLO")
        print()
    
    # Error rate
    error_rate_query = f'sum(rate(http_requests_total{{service="{service_name}",status=~"5.."}}[{days}d])) / sum(rate(http_requests_total{{service="{service_name}"}}[{days}d]))'
    error_rate_result = query_prometheus(error_rate_query)
    if error_rate_result:
        error_rate = float(error_rate_result[0]['value'][1])
        print(f"❌ Error Rate: {error_rate*100:.3f}%")
        print(f"   Target: < 0.100%")
        print()
    
    # Recommendations
    print("📋 Recommendations:")
    if budget > 0.75:
        print("   • Continue normal deployment cadence")
        print("   • Consider experimenting with new features")
    elif budget > 0.5:
        print("   • Review each deployment carefully")
        print("   • Increase test coverage")
    elif budget > 0.25:
        print("   • Slow down deployment frequency")
        print("   • Focus on bug fixes")
        print("   • Conduct reliability review")
    elif budget > 0:
        print("   • FREEZE deployments except emergency fixes")
        print("   • All hands on reliability")
        print("   • Schedule incident review")
    else:
        print("   • INCIDENT MODE - Error budget exhausted")
        print("   • No deployments allowed")
        print("   • Emergency reliability work only")
    
    print()
    print("=" * 60)

if __name__ == "__main__":
    import sys
    
    service = sys.argv[1] if len(sys.argv) > 1 else "payment_api"
    days = int(sys.argv[2]) if len(sys.argv) > 2 else 30
    
    generate_slo_report(service, days)
````

**2. Post-mortem template**:

`postmortem-template.md`:
````markdown
# Post-Mortem: [Incident Title]

**Date:** YYYY-MM-DD
**Duration:** X hours Y minutes
**Severity:** P0 / P1 / P2
**Status:** Draft / Under Review / Closed

## Summary
<!-- 1-2 sentences describing what happened -->

## Impact
- **Users Affected:** X% / X users
- **Services Affected:** [list services]
- **Duration:** From HH:MM to HH:MM UTC
- **Estimated Revenue Loss:** $X
- **SLO Impact:** X% error budget consumed

## Timeline (All times UTC)
- **HH:MM** - Initial symptoms detected
- **HH:MM** - Alert fired
- **HH:MM** - Engineer acknowledged
- **HH:MM** - Root cause identified
- **HH:MM** - Fix deployed
- **HH:MM** - Service recovered
- **HH:MM** - Incident closed

## Root Cause
<!-- Technical root cause - what broke and why -->

## Detection
- **How was it detected?** Alert / User report / Monitoring
- **Time to detect:** X minutes from start
- **What worked well?**
- **What could be improved?**

## Response
- **Time to acknowledge:** X minutes
- **Time to mitigate:** X minutes
- **Time to resolve:** X minutes
- **What worked well?**
- **What could be improved?**

## Resolution
<!-- How was it fixed -->

## Five Whys
1. Why did X happen? Because Y
2. Why did Y happen? Because Z
3. Why did Z happen? Because...
4. ...
5. Root cause: ...

## What Went Well
-
-

## What Went Wrong
-
-

## Action Items
| Action | Owner | Priority | ETA | Status |
|--------|-------|----------|-----|--------|
| [P0] Implement circuit breaker | Alice | High | 2024-01-15 | In Progress |
| [P1] Add integration tests | Bob | Medium | 2024-01-20 | Not Started |
| [P2] Improve monitoring | Carol | Low | 2024-01-25 | Not Started |

## Lessons Learned
-
-

## Appendix
### Relevant Logs
````
[paste relevant logs]
`````

### Metrics/Graphs

Show Image Show Image

### Related Incidents

- [INC-123] Similar issue on...

`````

**3. On-call runbook template**:

`runbook-template.md`:
````markdown
# Runbook: [Alert Name]

## Alert Details
- **Alert Name:** ErrorBudgetFastBurn
- **Severity:** Critical / Warning / Info
- **Service:** payment-api
- **SLO Impact:** High / Medium / Low

## What This Means
<!-- Explain in plain English what this alert means -->
The service is consuming error budget at 14x normal rate. If this continues, the monthly error budget will be exhausted in < 2 days.

## Impact
- **User Impact:** Users seeing 5xx errors, payment failures
- **Business Impact:** Revenue loss, customer complaints
- **Dependencies:** May affect downstream services

## Quick Checks
```bash
# Check error rate
kubectl logs -n prod -l app=payment-api --tail=100 | grep ERROR

# Check recent deployments
kubectl rollout history deployment/payment-api -n prod

# Check pod status
kubectl get pods -n prod -l app=payment-api
```

## Common Causes
1. Recent deployment with bugs
   - **Check:** Recent deployments in last hour
   - **Fix:** Rollback deployment

2. Dependency failure
   - **Check:** Database/cache/external API status
   - **Fix:** Restart dependency or switch to backup

3. Resource exhaustion
   - **Check:** CPU/Memory usage
   - **Fix:** Scale up replicas

4. Traffic spike
   - **Check:** Request rate
   - **Fix:** Enable rate limiting or scale

## Investigation Steps
1. Check Grafana dashboard: [link]
2. Check recent changes: `kubectl rollout history`
3. Check error logs: `kubectl logs -n prod -l app=payment-api`
4. Check dependencies: [list monitoring links]
5. Check metrics: [Prometheus queries]

## Resolution Steps
### If caused by recent deployment:
```bash
# Rollback
kubectl rollout undo deployment/payment-api -n prod

# Verify
kubectl rollout status deployment/payment-api -n prod

# Check metrics recovering
# [Prometheus/Grafana link]
```

### If caused by dependency:
```bash
# Check dependency health
curl https://dependency-api/health

# If unhealthy, restart
kubectl rollout restart deployment/dependency -n prod
```

### If caused by resource exhaustion:
```bash
# Scale up
kubectl scale deployment/payment-api -n prod --replicas=10

# Check HPA
kubectl get hpa -n prod
```

## Escalation
- **Primary:** @payments-team
- **Secondary:** @platform-team
- **Manager:** @engineering-manager

## Communication
- **Slack:** #incidents
- **Status Page:** Update at status.company.com
- **Template:** "We're investigating issues with [service]. ETA for resolution: [time]"

## Post-Incident
- [ ] Create Jira ticket
- [ ] Schedule post-mortem
- [ ] Update runbook with learnings
- [ ] Review SLO impact

## Related Links
- Dashboard: [Grafana link]
- Logs: [Loki/Kibana link]
- Traces: [Jaeger link]
- Playbook: [Confluence link]
- Previous incidents: [Links]
````

---

## Итоги модуля 8

После прохождения этого модуля ты должен уметь:

✅ Понимать разницу между SLI/SLO/SLA
✅ Рассчитывать и мониторить Error Budget
✅ Создавать multi-window multi-burn-rate alerts
✅ Выбирать правильные SLI для сервисов
✅ Настраивать SLO-based alerting в Prometheus
✅ Создавать SLO dashboards в Grafana
✅ Писать эффективные post-mortem документы
✅ Использовать Error Budget для принятия решений
✅ Измерять и снижать toil
✅ Проводить reliability reviews
✅ Создавать runbooks для on-call
✅ Применять SRE принципы на практике

**Ключевые принципы SRE:**
1. Измеряй надежность объективно (SLI/SLO)
2. Error Budget = право на ошибку
3. Баланс: velocity vs stability
4. Автоматизация > ручная работа
5. Постмортемы без обвинений
6. Toil < 50% времени
7. On-call должен быть sustainable

**SLO Formula:**

Availability SLO = successful_requests / total_requests >= target
Error Budget = 1 - SLO Example: 99.9% SLO → 0.1% error budget → 43.2 min/month downtime
Burn Rate = actual_error_rate / budgeted_error_rate

**Production Checklist:**
- ✅ SLO определены для всех критичных сервисов
- ✅ SLI метрики собираются и мониторятся
- ✅ Error budget tracking dashboard
- ✅ Multi-window burn rate alerts
- ✅ Runbooks для всех alerts
- ✅ Post-mortem процесс налажен
- ✅ On-call rotation справедливый
- ✅ Toil измеряется и снижается
- ✅ Incident response процесс документирован
- ✅ Regular SLO reviews



## Модуль 9: Security Monitoring и Audit Logging - безопасность и комплаенс (45 минут)

### 🎯 Напоминалка

**Security Monitoring - что это:**

```
Security Monitoring = Обнаружение и предотвращение угроз

Основные направления:
1. Access Control - кто и куда заходит
2. Vulnerability Detection - поиск уязвимостей
3. Intrusion Detection - обнаружение вторжений
4. Compliance - соответствие стандартам
5. Audit Logging - журнал всех действий
6. Threat Detection - выявление угроз
```

**CIA Triad - основа информационной безопасности:**

```
┌──────────────────────────────────┐
│   Confidentiality (Конфиденц.)   │  - Только авторизованный доступ
│          ▲                       │
│          │                       │
│          │                       │
│  Integrity ◄────► Availability   │  - Целостность ◄──► Доступность
│                                  │
└──────────────────────────────────┘

Мониторинг должен обеспечивать все три аспекта!
```

**Типы security событий для мониторинга:**

```
Authentication Events:
- Failed login attempts
- Brute force attacks
- Privilege escalation
- Account lockouts
- Password changes

Authorization Events:
- Unauthorized access attempts
- Permission changes
- Role modifications
- Policy violations

Network Events:
- Port scanning
- DDoS attacks
- Unusual traffic patterns
- Connection from blacklisted IPs
- Data exfiltration attempts

Application Events:
- SQL injection attempts
- XSS attacks
- API abuse
- Unusual API calls
- File upload attacks

System Events:
- Root/admin logins
- System file modifications
- Service starts/stops
- Configuration changes
- Package installations

Data Events:
- Sensitive data access
- Data exports
- Encryption key usage
- Database queries with PII
- File downloads
```

**SIEM (Security Information and Event Management):**

```
SIEM = Centralized security monitoring

Компоненты:
┌────────────────────────────────────┐
│   Log Collection                   │  - Сбор логов со всех источников
├────────────────────────────────────┤
│   Normalization                    │  - Приведение к единому формату
├────────────────────────────────────┤
│   Correlation                      │  - Корреляция событий
├────────────────────────────────────┤
│   Analysis                         │  - Анализ на угрозы
├────────────────────────────────────┤
│   Alerting                         │  - Оповещения о инцидентах
├────────────────────────────────────┤
│   Reporting                        │  - Отчеты для compliance
└────────────────────────────────────┘

Популярные SIEM:
- Splunk (commercial)
- Elastic Security (ELK)
- Wazuh (open source)
- Graylog (open source)
- IBM QRadar (commercial)
```

**Audit Logging - что логировать:**

```
WHO - Кто выполнил действие
├─ User ID
├─ Session ID
├─ IP Address
└─ User Agent

WHAT - Что было сделано
├─ Action type (CREATE, READ, UPDATE, DELETE)
├─ Resource type (user, file, database)
├─ Resource ID
└─ Operation status (success/failure)

WHEN - Когда произошло
├─ Timestamp (UTC)
├─ Duration
└─ Sequence number

WHERE - Где произошло
├─ Service/application
├─ Server/container
├─ Geographic location
└─ Network zone

WHY - Контекст
├─ Request ID
├─ Session context
├─ Business reason
└─ Approval ID (if applicable)

RESULT - Результат
├─ Status code
├─ Error message
├─ Data changed (before/after)
└─ Side effects
```

**Audit log format (JSON):**

json

````json
{
  "timestamp": "2025-01-15T10:30:00.000Z",
  "event_id": "evt_abc123",
  "event_type": "user.login.success",
  "severity": "info",
  
  "actor": {
    "user_id": "user_123",
    "username": "john.doe",
    "email": "john.doe@example.com",
    "ip_address": "192.168.1.100",
    "user_agent": "Mozilla/5.0...",
    "session_id": "sess_xyz789"
  },
  
  "action": {
    "type": "authentication",
    "operation": "login",
    "resource_type": "session",
    "resource_id": "sess_xyz789",
    "status": "success"
  },
  
  "context": {
    "service": "auth-service",
    "server": "auth-01",
    "region": "eu-west-1",
    "environment": "production",
    "request_id": "req_456"
  },
  
  "metadata": {
    "mfa_used": true,
    "login_method": "password",
    "previous_login": "2025-01-14T08:00:00Z",
    "risk_score": 0.1
  }
}
```

**Compliance стандарты:**
```
PCI DSS (Payment Card Industry):
- Логи хранятся минимум 1 год
- Защита данных карт
- Регулярные security аудиты
- Access control и мониторинг

GDPR (General Data Protection Regulation):
- Право на удаление данных
- Data breach notification (72 hours)
- Audit trail всех операций с PII
- Encryption at rest and in transit

HIPAA (Healthcare):
- Защита медицинских данных
- Audit logs всех доступов к PHI
- Encryption required
- Access control

SOC 2:
- Security controls
- Availability monitoring
- Confidentiality
- Processing integrity

ISO 27001:
- Information security management
- Risk assessment
- Incident management
- Continuous monitoring
````

**Authentication monitoring:**

promql

````promql
# Failed login attempts (possible brute force)
sum(rate(auth_login_attempts_total{status="failed"}[5m])) by (username, ip) > 5

# Successful login from new location
auth_login_success{location!~"known_locations"}

# Multiple failed logins followed by success (credential stuffing)
(
  sum(increase(auth_login_attempts_total{status="failed"}[5m])) by (username) > 10
  and
  sum(increase(auth_login_attempts_total{status="success"}[5m])) by (username) > 0
)

# Login from multiple IPs simultaneously
count(auth_login_success) by (username) > 2

# Privileged account access
auth_login_success{role=~"admin|root|superuser"}
```

**API security monitoring:**
```
Метрики для мониторинга:

Rate Limiting:
- Requests per IP
- Requests per user
- Requests per endpoint

Anomaly Detection:
- Unusual request patterns
- Spike in errors
- New endpoints accessed
- Unusual request size

Authentication:
- Invalid token attempts
- Expired token usage
- Missing authentication
- Token reuse

Authorization:
- 403 Forbidden errors
- Permission escalation attempts
- Cross-tenant access attempts

Input Validation:
- SQL injection patterns
- XSS attempts
- Path traversal
- Command injection
````

**Network security monitoring:**

bash

````bash
# Метрики для мониторинга сети:

Connection tracking:
- New connections per second
- Connection states (ESTABLISHED, SYN_SENT, etc)
- Connections per IP
- Unusual ports accessed

Traffic analysis:
- Bandwidth usage
- Protocol distribution (HTTP, HTTPS, SSH, etc)
- Traffic to/from suspicious IPs
- DNS queries patterns

Firewall events:
- Blocked connections
- Rule violations
- Port scan attempts
- Failed connection attempts

IDS/IPS events:
- Signature matches
- Anomaly detection
- Blocked attacks
- False positives
```

**Container security:**
```
Security scanning:
┌─────────────────────────────────┐
│   Image Scanning                │
│   - Trivy, Clair, Anchore       │
│   - CVE detection               │
│   - License compliance          │
├─────────────────────────────────┤
│   Runtime Security              │
│   - Falco (syscall monitoring)  │
│   - AppArmor/SELinux            │
│   - Pod Security Standards      │
├─────────────────────────────────┤
│   Network Policies              │
│   - Ingress/Egress rules        │
│   - Service mesh security       │
│   - mTLS enforcement            │
├─────────────────────────────────┤
│   Secrets Management            │
│   - Vault, Sealed Secrets       │
│   - Rotation policies           │
│   - Access auditing             │
└─────────────────────────────────┘
````

**Falco - runtime security для Kubernetes:**

yaml

```yaml
# Falco правила

# Detect shell in container
- rule: Terminal shell in container
  desc: A shell was used as the entrypoint/exec point
  condition: >
    spawned_process and container
    and shell_procs
    and proc.tty != 0
  output: >
    Shell spawned in container
    (user=%user.name container_id=%container.id
    container_name=%container.name shell=%proc.name
    parent=%proc.pname cmdline=%proc.cmdline)
  priority: WARNING

# Detect sensitive file access
- rule: Read sensitive file
  desc: Attempt to read sensitive files
  condition: >
    open_read and container
    and fd.name in (sensitive_files)
  output: >
    Sensitive file opened for reading
    (user=%user.name file=%fd.name
    container=%container.name)
  priority: WARNING

# Detect privilege escalation
- rule: Privilege Escalation
  desc: Detect setuid or setgid
  condition: >
    spawned_process and container
    and (proc.name in (setuid_binaries))
  output: >
    Privilege escalation detected
    (user=%user.name proc=%proc.name
    container=%container.name)
  priority: CRITICAL
```

**Vulnerability scanning:**

bash

```bash
# Trivy - сканирование образов
trivy image nginx:latest

# Результат:
Total: 145 (UNKNOWN: 0, LOW: 45, MEDIUM: 75, HIGH: 20, CRITICAL: 5)

# Сканирование filesystem
trivy fs /path/to/project

# Kubernetes сканирование
trivy k8s --report summary cluster

# Integration в CI/CD
trivy image --severity CRITICAL,HIGH --exit-code 1 myapp:latest
```

**Security metrics (Prometheus):**

promql

````promql
# Failed authentication rate
rate(auth_failed_attempts_total[5m])

# 403 Forbidden rate (authorization failures)
rate(http_requests_total{status="403"}[5m])

# 401 Unauthorized rate
rate(http_requests_total{status="401"}[5m])

# Suspicious IPs
count(http_requests_total{ip=~"suspicious_ip_list"})

# Certificate expiration
(cert_expiry_timestamp_seconds - time()) / 86400 < 30  # < 30 days

# Vulnerability count by severity
sum(trivy_vulnerabilities) by (severity)

# Security scan failures
rate(security_scan_failures_total[5m])

# WAF blocks
rate(waf_blocked_requests_total[5m])

# Rate limit violations
rate(rate_limit_exceeded_total[5m])
```

**WAF (Web Application Firewall) monitoring:**
```
ModSecurity/OWASP Core Rule Set:

Типы атак для обнаружения:
- SQL Injection
- XSS (Cross-Site Scripting)
- CSRF (Cross-Site Request Forgery)
- Path Traversal
- Command Injection
- XXE (XML External Entity)
- File Upload attacks
- Session Hijacking

Метрики WAF:
- Blocked requests by rule
- False positive rate
- Top attacking IPs
- Attack type distribution
- Response time impact
````

**Secrets detection:**

bash

````bash
# Detect secrets in code/logs
trufflehog git https://github.com/example/repo

# Common patterns to detect:
- AWS keys: AKIA[0-9A-Z]{16}
- GitHub tokens: ghp_[0-9a-zA-Z]{36}
- Private keys: -----BEGIN PRIVATE KEY-----
- Passwords in code: password = "..."
- API keys: api_key = "..."
- Database URLs: postgres://user:pass@host

# Gitleaks - alternative
gitleaks detect --source . --verbose

# Integration в pre-commit hook
```

**Data Loss Prevention (DLP):**
```
DLP Monitoring:

Sensitive data patterns:
- Credit card numbers (PCI)
- Social Security Numbers
- Email addresses
- Phone numbers
- API keys / tokens
- Personal Health Information (PHI)

Monitoring points:
- File uploads
- API responses
- Database queries
- Email attachments
- Logs (prevent leaking secrets)
- External integrations

Actions:
- Block transmission
- Mask/redact data
- Alert security team
- Log incident
````

**Compliance reporting:**

sql

````sql
-- Audit queries for compliance reports

-- Who accessed user data (GDPR)
SELECT 
  timestamp,
  actor_user_id,
  action_type,
  resource_type,
  resource_id,
  status
FROM audit_logs
WHERE resource_type = 'user_data'
  AND timestamp > NOW() - INTERVAL '90 days'
ORDER BY timestamp DESC;

-- Failed access attempts (PCI DSS)
SELECT 
  DATE_TRUNC('day', timestamp) as date,
  COUNT(*) as failed_attempts,
  actor_ip_address
FROM audit_logs
WHERE action_type = 'authentication'
  AND status = 'failed'
  AND timestamp > NOW() - INTERVAL '1 year'
GROUP BY date, actor_ip_address
HAVING COUNT(*) > 5
ORDER BY failed_attempts DESC;

-- Privileged access report (SOC 2)
SELECT 
  actor_user_id,
  COUNT(*) as access_count,
  MAX(timestamp) as last_access
FROM audit_logs
WHERE actor_role IN ('admin', 'root', 'superuser')
  AND timestamp > NOW() - INTERVAL '30 days'
GROUP BY actor_user_id
ORDER BY access_count DESC;

-- Data modification audit (ISO 27001)
SELECT 
  timestamp,
  actor_user_id,
  resource_type,
  resource_id,
  action_type,
  metadata->>'before' as before_value,
  metadata->>'after' as after_value
FROM audit_logs
WHERE action_type IN ('UPDATE', 'DELETE')
  AND timestamp > NOW() - INTERVAL '7 days'
ORDER BY timestamp DESC;
```

**Incident response workflow:**
```
Detection → Triage → Investigation → Containment → Eradication → Recovery → Post-Incident

1. Detection (Automated)
   ├─ SIEM alert fires
   ├─ Anomaly detected
   └─ User report

2. Triage (5-15 min)
   ├─ Verify alert (true positive?)
   ├─ Assess severity
   ├─ Assign incident owner
   └─ Create incident ticket

3. Investigation (15-60 min)
   ├─ Analyze logs
   ├─ Check timeline
   ├─ Identify affected systems
   ├─ Determine attack vector
   └─ Assess impact

4. Containment (Immediate)
   ├─ Isolate affected systems
   ├─ Block malicious IPs
   ├─ Disable compromised accounts
   ├─ Prevent spread
   └─ Preserve evidence

5. Eradication (Hours-Days)
   ├─ Remove malware
   ├─ Close vulnerabilities
   ├─ Patch systems
   └─ Reset credentials

6. Recovery (Hours-Days)
   ├─ Restore from backups
   ├─ Verify system integrity
   ├─ Monitor for reinfection
   └─ Resume normal operations

7. Post-Incident (Days-Weeks)
   ├─ Incident report
   ├─ Lessons learned
   ├─ Update runbooks
   ├─ Improve detection
   └─ Train team
```

**Security alert priorities:**
```
P0 - Critical (Page immediately)
├─ Active breach detected
├─ Data exfiltration in progress
├─ Ransomware detected
├─ Root/admin compromise
└─ DDoS attack

P1 - High (Alert within 15 min)
├─ Multiple failed auth from same IP
├─ Privilege escalation attempt
├─ Suspicious admin activity
├─ Vulnerability exploitation attempt
└─ Certificate about to expire

P2 - Medium (Alert within 1 hour)
├─ Port scan detected
├─ Unusual API usage
├─ Policy violation
├─ Configuration drift
└─ Suspicious login location

P3 - Low (Daily digest)
├─ Informational security events
├─ Compliance audit findings
├─ Low severity vulnerabilities
└─ Best practice recommendations
````

### 💻 Задание

Создай полноценную систему security monitoring:

1. **Установи Falco для runtime security**:

bash

```bash
# Установка Falco в Kubernetes
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update

kubectl create namespace falco

helm install falco falcosecurity/falco \
  --namespace falco \
  --set falcosidekick.enabled=true \
  --set falcosidekick.webui.enabled=true

# Проверка
kubectl get pods -n falco
```

2. **Создай custom Falco rules**:

`k8s-manifests/falco-custom-rules.yaml`:

yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: falco-custom-rules
  namespace: falco
data:
  custom-rules.yaml: |
    # Custom security rules
    
    - rule: Unauthorized Process in Container
      desc: Detect unauthorized process execution
      condition: >
        spawned_process and container
        and not proc.name in (allowed_processes)
      output: >
        Unauthorized process started in container
        (user=%user.name process=%proc.name
        container=%container.name
        parent=%proc.pname cmdline=%proc.cmdline)
      priority: WARNING
      tags: [container, process]
    
    - rule: Sensitive File Access
      desc: Detect access to sensitive files
      condition: >
        open_read and container
        and (fd.name startswith /etc/shadow or
             fd.name startswith /etc/passwd or
             fd.name contains /id_rsa or
             fd.name contains /authorized_keys)
      output: >
        Sensitive file accessed
        (user=%user.name file=%fd.name
        container=%container.name
        process=%proc.name)
      priority: CRITICAL
      tags: [file, credentials]
    
    - rule: Reverse Shell Detected
      desc: Detect reverse shell attempts
      condition: >
        spawned_process and container
        and proc.name in (shell_binaries)
        and (proc.args contains ">" or proc.args contains "&")
        and fd.name contains "/dev/tcp"
      output: >
        Potential reverse shell detected
        (user=%user.name container=%container.name
        process=%proc.name args=%proc.args)
      priority: CRITICAL
      tags: [shell, network]
    
    - rule: Crypto Mining Activity
      desc: Detect cryptocurrency mining
      condition: >
        spawned_process and container
        and (proc.name in (xmrig, minerd, ccminer, ethminer) or
             proc.cmdline contains "stratum+tcp" or
             proc.cmdline contains "pool.minergate.com")
      output: >
        Cryptocurrency mining detected
        (user=%user.name container=%container.name
        process=%proc.name cmdline=%proc.cmdline)
      priority: CRITICAL
      tags: [malware, mining]
    
    - rule: Container Drift Detected
      desc: Detect executable created in container
      condition: >
        container and (
          open_write and
          fd.name startswith /bin/ or
          fd.name startswith /usr/bin/
        )
      output: >
        Executable file created in container (drift)
        (user=%user.name file=%fd.name
        container=%container.name)
      priority: ERROR
      tags: [container, drift]
    
    - rule: Privileged Container Launch
      desc: Detect privileged container
      condition: >
        container_started and container.privileged=true
      output: >
        Privileged container started
        (user=%user.name container=%container.name
        image=%container.image.repository)
      priority: WARNING
      tags: [container, privileges]
    
    - rule: SSH Connection from Container
      desc: Detect outbound SSH from container
      condition: >
        outbound and container
        and fd.sport != 22
        and fd.dport = 22
      output: >
        Outbound SSH connection from container
        (user=%user.name container=%container.name
        dest_ip=%fd.rip dest_port=%fd.rport)
      priority: WARNING
      tags: [network, ssh]
    
    - rule: Package Manager in Container
      desc: Detect package installation in running container
      condition: >
        spawned_process and container
        and proc.name in (apt, apt-get, yum, dnf, apk, pip, npm)
      output: >
        Package manager executed in container
        (user=%user.name container=%container.name
        process=%proc.name args=%proc.args)
      priority: ERROR
      tags: [container, package]
```

3. **Создай audit logging system**:

`audit-logging/audit-logger.py`:

python

```python
from flask import Flask, request, g
from functools import wraps
import json
import time
import hashlib
from datetime import datetime
import logging

app = Flask(__name__)

# Audit log configuration
audit_logger = logging.getLogger('audit')
audit_logger.setLevel(logging.INFO)
handler = logging.FileHandler('/var/log/audit/audit.log')
formatter = logging.Formatter('%(message)s')
handler.setFormatter(formatter)
audit_logger.addHandler(handler)

# Prometheus metrics
from prometheus_client import Counter, Histogram

AUDIT_EVENTS = Counter(
    'audit_events_total',
    'Total audit events',
    ['event_type', 'status', 'severity']
)

AUDIT_DURATION = Histogram(
    'audit_event_duration_seconds',
    'Audit event processing time',
    ['event_type']
)

def get_client_ip():
    """Get real client IP"""
    if request.headers.get('X-Forwarded-For'):
        return request.headers.get('X-Forwarded-For').split(',')[0]
    return request.remote_addr

def get_user_context():
    """Get current user context"""
    # В реальном приложении это из JWT token или session
    return {
        'user_id': g.get('user_id', 'anonymous'),
        'username': g.get('username', 'anonymous'),
        'email': g.get('email', None),
        'roles': g.get('roles', []),
        'session_id': g.get('session_id', None)
    }

def create_audit_event(event_type, action, resource_type, resource_id, 
                       status, severity='info', metadata=None):
    """Create structured audit log entry"""
    
    user_context = get_user_context()
    
    audit_event = {
        'timestamp': datetime.utcnow().isoformat() + 'Z',
        'event_id': hashlib.sha256(
            f"{time.time()}{event_type}{user_context['user_id']}".encode()
        ).hexdigest()[:16],
        'event_type': event_type,
        'severity': severity,
        
        'actor': {
            'user_id': user_context['user_id'],
            'username': user_context['username'],
            'email': user_context['email'],
            'roles': user_context['roles'],
            'ip_address': get_client_ip(),
            'user_agent': request.headers.get('User-Agent'),
            'session_id': user_context['session_id']
        },
        
        'action': {
            'type': action,
            'resource_type': resource_type,
            'resource_id': resource_id,
            'status': status
        },
        
        'context': {
            'service': 'audit-api',
            'request_id': g.get('request_id'),
            'request_method': request.method,
            'request_path': request.path,
            'request_query': dict(request.args)
        },
        
        'metadata': metadata or {}
    }
    
    # Log to file
    audit_logger.info(json.dumps(audit_event))
    
    # Metrics
    AUDIT_EVENTS.labels(
        event_type=event_type,
        status=status,
        severity=severity
    ).inc()
    
    return audit_event

def audit_log(event_type, resource_type):
    """Decorator for automatic audit logging"""
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            start_time = time.time()
            resource_id = kwargs.get('resource_id', 'unknown')
            
            try:
                result = f(*args, **kwargs)
                
                # Success audit
                create_audit_event(
                    event_type=event_type,
                    action=request.method,
                    resource_type=resource_type,
                    resource_id=resource_id,
                    status='success',
                    severity='info',
                    metadata={
                        'duration_ms': (time.time() - start_time) * 1000
                    }
                )
                
                return result
                
            except Exception as e:
                # Failure audit
                create_audit_event(
                    event_type=event_type,
                    action=request.method,
                    resource_type=resource_type,
                    resource_id=resource_id,
                    status='failure',
                    severity='error',
                    metadata={
                        'error': str(e),
                        'duration_ms': (time.time() - start_time) * 1000
                    }
                )
                raise
            
            finally:
                AUDIT_DURATION.labels(event_type=event_type).observe(
                    time.time() - start_time
                )
        
        return decorated_function
    return decorator

# Example protected endpoints

@app.route('/api/user/<user_id>', methods=['GET'])
@audit_log(event_type='user.read', resource_type='user')
def get_user(user_id):
    """Get user - audited endpoint"""
    # Check if accessing sensitive PII
    if 'ssn' in request.args or 'credit_card' in request.args:
        create_audit_event(
            event_type='pii.access',
            action='READ',
            resource_type='user',
            resource_id=user_id,
            status='success',
            severity='warning',
            metadata={'fields_accessed': list(request.args.keys())}
        )
    
    return {'user_id': user_id, 'name': 'Test User'}

@app.route('/api/user/<user_id>', methods=['PUT'])
@audit_log(event_type='user.update', resource_type='user')
def update_user(user_id):
    """Update user - audited with before/after"""
    data = request.json
    
    # Log before and after for critical changes
    if 'email' in data or 'role' in data:
        create_audit_event(
            event_type='user.critical_update',
            action='UPDATE',
            resource_type='user',
            resource_id=user_id,
            status='success',
            severity='warning',
            metadata={
                'changes': data,
                'before': {'email': 'old@example.com', 'role': 'user'},
                'after': data
            }
        )
    
    return {'status': 'updated'}

@app.route('/api/user/<user_id>', methods=['DELETE'])
@audit_log(event_type='user.delete', resource_type='user')
def delete_user(user_id):
    """Delete user - critical audit event"""
    create_audit_event(
        event_type='user.delete',
        action='DELETE',
        resource_type='user',
        resource_id=user_id,
        status='success',
        severity='critical',
        metadata={
            'permanent': True,
            'reason': request.args.get('reason', 'not_specified')
        }
    )
    
    return {'status': 'deleted'}

@app.route('/api/admin/role', methods=['POST'])
@audit_log(event_type='role.grant', resource_type='permission')
def grant_role():
    """Grant admin role - security critical"""
    data = request.json
    
    create_audit_event(
        event_type='privilege.escalation',
        action='GRANT',
        resource_type='permission',
        resource_id=data.get('role'),
        status='success',
        severity='critical',
        metadata={
            'target_user': data.get('user_id'),
            'role_granted': data.get('role'),
            'granted_by': get_user_context()['user_id']
        }
    )
    
    return {'status': 'role_granted'}

@app.route('/api/export/data', methods=['POST'])
@audit_log(event_type='data.export', resource_type='data')
def export_data():
    """Data export - DLP monitoring"""
    data = request.json
    
    create_audit_event(
        event_type='data.export',
        action='EXPORT',
        resource_type='data',
        resource_id='bulk_export',
        status='success',
        severity='warning',
        metadata={
            'record_count': data.get('count', 0),
            'data_types': data.get('types', []),
            'destination': data.get('destination'),
            'format': data.get('format')
        }
    )
    
    return {'status': 'exported'}

# Security events

@app.route('/auth/login', methods=['POST'])
def login():
    """Login with audit"""
    data = request.json
    username = data.get('username')
    
    # Simulate authentication
    import random
    success = random.random() > 0.1  # 90% success rate
    
    if success:
        create_audit_event(
            event_type='auth.login.success',
            action='AUTHENTICATE',
            resource_type='session',
            resource_id='sess_' + hashlib.md5(username.encode()).hexdigest()[:8],
            status='success',
            severity='info',
            metadata={
                'method': 'password',
                'mfa_used': data.get('mfa', False)
            }
        )
        return {'status': 'success', 'token': 'fake_token'}
    else:
        create_audit_event(
            event_type='auth.login.failure',
            action='AUTHENTICATE',
            resource_type='session',
            resource_id='unknown',
            status='failure',
            severity='warning',
			metadata={ 'method': 'password', 'reason': 'invalid_credentials', 'username_attempted': username } ) return {'status': 'failed', 'error': 'Invalid credentials'}, 401

@app.route('/api/config', methods=['PUT']) @audit_log(event_type='config.change', resource_type='configuration') def update_config(): """Configuration change - critical audit""" data = request.json


create_audit_event(
    event_type='config.change',
    action='UPDATE',
    resource_type='configuration',
    resource_id=data.get('key'),
    status='success',
    severity='critical',
    metadata={
        'key': data.get('key'),
        'old_value': '***REDACTED***',  # Never log secrets
        'new_value': '***REDACTED***',
        'change_type': 'manual'
    }
)

return {'status': 'config_updated'}


if **name** == '**main**': app.run(host='0.0.0.0', port=5000)

````

4. **Создай security monitoring dashboards**:

`k8s-manifests/security-prometheus-rules.yaml`:
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: security-alerts
  namespace: monitoring
spec:
  groups:
  - name: security.rules
    interval: 30s
    rules:
    # Authentication alerts
    - alert: BruteForceAttempt
      expr: |
        sum(rate(audit_events_total{event_type="auth.login.failure"}[5m])) by (actor_ip_address) > 10
      for: 2m
      labels:
        severity: critical
        category: security
      annotations:
        summary: "Brute force attack detected"
        description: "IP {{ $labels.actor_ip_address }} has {{ $value }} failed login attempts/sec"
        runbook: "Block IP and investigate"
    
    - alert: MultipleFailedLoginsBeforeSuccess
      expr: |
        (
          sum(increase(audit_events_total{event_type="auth.login.failure"}[5m])) by (actor_user_id) > 5
          and
          sum(increase(audit_events_total{event_type="auth.login.success"}[5m])) by (actor_user_id) > 0
        )
      for: 1m
      labels:
        severity: warning
        category: security
      annotations:
        summary: "Possible credential stuffing"
        description: "User {{ $labels.actor_user_id }} had multiple failed attempts then success"
    
    # Privilege escalation
    - alert: PrivilegeEscalation
      expr: |
        sum(rate(audit_events_total{event_type="privilege.escalation"}[5m])) > 0
      for: 1m
      labels:
        severity: critical
        category: security
      annotations:
        summary: "Privilege escalation detected"
        description: "Admin role granted to user"
    
    # Data exfiltration
    - alert: MassiveDataExport
      expr: |
        sum(rate(audit_events_total{event_type="data.export"}[5m])) > 10
      for: 5m
      labels:
        severity: critical
        category: security
      annotations:
        summary: "Possible data exfiltration"
        description: "Unusual volume of data exports detected"
    
    # PII access
    - alert: UnauthorizedPIIAccess
      expr: |
        sum(rate(audit_events_total{event_type="pii.access",status="failure"}[5m])) > 0
      for: 1m
      labels:
        severity: critical
        category: security
      annotations:
        summary: "Unauthorized PII access attempt"
        description: "Failed attempt to access sensitive personal data"
    
    # Configuration changes
    - alert: CriticalConfigurationChange
      expr: |
        sum(increase(audit_events_total{event_type="config.change",severity="critical"}[5m])) > 0
      for: 1m
      labels:
        severity: warning
        category: security
      annotations:
        summary: "Critical configuration changed"
        description: "System configuration was modified"
    
    # Anomalous behavior
    - alert: UnusualAPIActivity
      expr: |
        sum(rate(http_requests_total[5m])) by (user_id) >
        (avg_over_time(sum(rate(http_requests_total[5m])) by (user_id)[1h]) * 3)
      for: 10m
      labels:
        severity: warning
        category: security
      annotations:
        summary: "Unusual API activity detected"
        description: "User {{ $labels.user_id }} has 3x normal API activity"
    
    # Certificate expiration
    - alert: CertificateExpiringSoon
      expr: |
        (ssl_certificate_expiry_timestamp - time()) / 86400 < 30
      for: 1h
      labels:
        severity: warning
        category: security
      annotations:
        summary: "SSL certificate expiring soon"
        description: "Certificate {{ $labels.cn }} expires in {{ $value }} days"
    
    # Falco security events
    - alert: FalcoSecurityEvent
      expr: |
        sum(rate(falco_events_total{priority=~"Critical|Error"}[5m])) > 0
      for: 1m
      labels:
        severity: critical
        category: security
      annotations:
        summary: "Falco security event detected"
        description: "Runtime security violation: {{ $labels.rule }}"
    
    # WAF blocks
    - alert: HighWAFBlockRate
      expr: |
        sum(rate(waf_blocked_requests_total[5m])) > 10
      for: 5m
      labels:
        severity: warning
        category: security
      annotations:
        summary: "High WAF block rate"
        description: "WAF is blocking {{ $value }} requests/sec"
```

5. **Deploy audit system**:
```bash
# Build audit logger
cd audit-logging
docker build -t audit-logger:latest .
kind load docker-image audit-logger:latest --name monitoring-cluster

# Deploy
kubectl create namespace security
kubectl apply -f k8s-manifests/audit-logger-deployment.yaml
kubectl apply -f k8s-manifests/security-prometheus-rules.yaml
kubectl apply -f k8s-manifests/falco-custom-rules.yaml
```

`k8s-manifests/audit-logger-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: audit-logger
  namespace: security
spec:
  replicas: 2
  selector:
    matchLabels:
      app: audit-logger
  template:
    metadata:
      labels:
        app: audit-logger
    spec:
      containers:
      - name: app
        image: audit-logger:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 5000
        volumeMounts:
        - name: audit-logs
          mountPath: /var/log/audit
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
      volumes:
      - name: audit-logs
        persistentVolumeClaim:
          claimName: audit-logs-pvc

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: audit-logs-pvc
  namespace: security
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

---
apiVersion: v1
kind: Service
metadata:
  name: audit-logger
  namespace: security
spec:
  selector:
    app: audit-logger
  ports:
  - port: 5000
    targetPort: 5000

---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: audit-logger
  namespace: security
spec:
  selector:
    matchLabels:
      app: audit-logger
  endpoints:
  - port: metrics
    path: /metrics
    interval: 30s
```

6. **Тестирование security monitoring**:
```bash
# Test authentication failures (brute force)
for i in {1..20}; do
  curl -X POST http://localhost:5000/auth/login \
    -H "Content-Type: application/json" \
    -d '{"username":"admin","password":"wrong"}' &
done

# Watch alerts
kubectl logs -n monitoring -l app.kubernetes.io/name=alertmanager -f

# Check Falco events
kubectl logs -n falco -l app.kubernetes.io/name=falco -f

# Test privilege escalation
curl -X POST http://localhost:5000/api/admin/role \
  -H "Content-Type: application/json" \
  -d '{"user_id":"user123","role":"admin"}'

# Test data export
curl -X POST http://localhost:5000/api/export/data \
  -H "Content-Type: application/json" \
  -d '{"count":10000,"types":["user","payment"],"destination":"s3"}'

# Check audit logs
kubectl exec -n security audit-logger-xxx -- tail -f /var/log/audit/audit.log | jq
```

### 🚀 Бонус (новое)

**1. Vulnerability scanning automation**:

`vulnerability-scanner.yaml`:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: trivy-scanner
  namespace: security
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: trivy
            image: aquasec/trivy:latest
            args:
            - "image"
            - "--severity"
            - "CRITICAL,HIGH"
            - "--format"
            - "json"
            - "--output"
            - "/reports/trivy-report.json"
            - "myapp:latest"
            volumeMounts:
            - name: reports
              mountPath: /reports
          volumes:
          - name: reports
            persistentVolumeClaim:
              claimName: scan-reports
          restartPolicy: OnFailure
```

**2. Secret scanning in CI/CD**:

`.github/workflows/security-scan.yml`:
```yaml
name: Security Scan

on: [push, pull_request]

jobs:
  secret-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0
    
    - name: TruffleHog Secret Scan
      uses: trufflesecurity/trufflehog@main
      with:
        path: ./
        base: main
        head: HEAD
    
    - name: Gitleaks Scan
      uses: gitleaks/gitleaks-action@v2
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  
  vulnerability-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Run Trivy
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        format: 'sarif'
        output: 'trivy-results.sarif'
        severity: 'CRITICAL,HIGH'
    
    - name: Upload to Security Tab
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'
```

**3. Compliance report generator**:

`compliance-reporter.py`:
```python
#!/usr/bin/env python3
"""
Compliance Report Generator
Supports: PCI DSS, GDPR, HIPAA, SOC 2
"""
import psycopg2
from datetime import datetime, timedelta
import json

class ComplianceReporter:
    def __init__(self, db_conn):
        self.conn = db_conn
    
    def generate_pci_dss_report(self, start_date, end_date):
        """PCI DSS Compliance Report"""
        report = {
            'standard': 'PCI DSS v4.0',
            'period': f"{start_date} to {end_date}",
            'generated': datetime.now().isoformat(),
            'findings': []
        }
        
        # Requirement 10: Track and monitor all access
        cursor = self.conn.cursor()
        
        # 10.2.1: User access to cardholder data
        cursor.execute("""
            SELECT COUNT(*), actor_user_id
            FROM audit_logs
            WHERE event_type LIKE 'payment%'
              AND timestamp BETWEEN %s AND %s
            GROUP BY actor_user_id
        """, (start_date, end_date))
        
        report['findings'].append({
            'requirement': '10.2.1',
            'description': 'User access to cardholder data',
            'access_count': cursor.fetchall()
        })
        
        # 10.2.2: Administrative actions
        cursor.execute("""
            SELECT COUNT(*)
            FROM audit_logs
            WHERE actor_role IN ('admin', 'root')
              AND timestamp BETWEEN %s AND %s
        """, (start_date, end_date))
        
        report['findings'].append({
            'requirement': '10.2.2',
            'description': 'Administrative actions',
            'count': cursor.fetchone()[0]
        })
        
        # 10.2.4: Invalid access attempts
        cursor.execute("""
            SELECT COUNT(*), actor_ip_address
            FROM audit_logs
            WHERE status = 'failure'
              AND event_type LIKE 'auth%'
              AND timestamp BETWEEN %s AND %s
            GROUP BY actor_ip_address
            HAVING COUNT(*) > 5
        """, (start_date, end_date))
        
        report['findings'].append({
            'requirement': '10.2.4',
            'description': 'Invalid access attempts',
            'suspicious_ips': cursor.fetchall()
        })
        
        return report
    
    def generate_gdpr_report(self, start_date, end_date):
        """GDPR Compliance Report"""
        cursor = self.conn.cursor()
        
        report = {
            'regulation': 'GDPR',
            'period': f"{start_date} to {end_date}",
            'generated': datetime.now().isoformat(),
            'data_processing': {}
        }
        
        # Personal data access
        cursor.execute("""
            SELECT 
              COUNT(*) as access_count,
              actor_user_id,
              resource_type
            FROM audit_logs
            WHERE event_type IN ('pii.access', 'user.read')
              AND timestamp BETWEEN %s AND %s
            GROUP BY actor_user_id, resource_type
        """, (start_date, end_date))
        
        report['data_processing']['access'] = cursor.fetchall()
        
        # Data deletions (right to be forgotten)
        cursor.execute("""
            SELECT COUNT(*), metadata->>'reason'
            FROM audit_logs
            WHERE event_type = 'user.delete'
              AND timestamp BETWEEN %s AND %s
            GROUP BY metadata->>'reason'
        """, (start_date, end_date))
        
        report['data_processing']['deletions'] = cursor.fetchall()
        
        # Data breaches (must report within 72h)
        cursor.execute("""
            SELECT *
            FROM audit_logs
            WHERE event_type LIKE 'security.breach%'
              AND timestamp BETWEEN %s AND %s
        """, (start_date, end_date))
        
        report['data_breaches'] = cursor.fetchall()
        
        return report

if __name__ == "__main__":
    conn = psycopg2.connect("postgresql://user:pass@localhost/audit")
    reporter = ComplianceReporter(conn)
    
    end_date = datetime.now()
    start_date = end_date - timedelta(days=30)
    
    # Generate reports
    pci_report = reporter.generate_pci_dss_report(start_date, end_date)
    gdpr_report = reporter.generate_gdpr_report(start_date, end_date)
    
    print(json.dumps(pci_report, indent=2))
    print(json.dumps(gdpr_report, indent=2))
```

**4. Security dashboard в Grafana**:

Import dashboard `security-dashboard.json`:
```json
{
  "dashboard": {
    "title": "Security Monitoring Dashboard",
    "tags": ["security", "audit"],
    "panels": [
      {
        "id": 1,
        "title": "Authentication Failures",
        "targets": [
          {
            "expr": "sum(rate(audit_events_total{event_type='auth.login.failure'}[5m])) by (actor_ip_address)",
            "legendFormat": "{{ actor_ip_address }}"
          }
        ],
        "type": "timeseries"
      },
      {
        "id": 2,
        "title": "Falco Security Events",
        "targets": [
          {
            "expr": "sum(rate(falco_events_total[5m])) by (rule, priority)",
            "legendFormat": "{{ rule }} ({{ priority }})"
          }
        ],
        "type": "timeseries"
      },
      {
        "id": 3,
        "title": "Privilege Escalations",
        "targets": [
          {
            "expr": "sum(increase(audit_events_total{event_type='privilege.escalation'}[1h]))"
          }
        ],
        "type": "stat"
      },
      {
        "id": 4,
        "title": "Top Failed Auth IPs",
        "targets": [
          {
            "expr": "topk(10, sum(increase(audit_events_total{event_type='auth.login.failure'}[24h])) by (actor_ip_address))",
            "format": "table"
          }
        ],
        "type": "table"
      },
      {
        "id": 5,
        "title": "Vulnerability Scan Results",
        "targets": [
          {
            "expr": "sum(trivy_vulnerabilities) by (severity)",
            "legendFormat": "{{ severity }}"
          }
        ],
        "type": "piechart"
      }
    ]
  }
}
```

---

## Итоги модуля 9

После прохождения этого модуля ты должен уметь:

✅ Понимать основы security monitoring
✅ Настраивать Falco для runtime security
✅ Создавать structured audit logs
✅ Мониторить authentication и authorization события
✅ Обнаруживать security threats в реальном времени
✅ Настраивать vulnerability scanning
✅ Генерировать compliance reports
✅ Создавать security alerts в Prometheus
✅ Использовать SIEM концепции
✅ Проводить incident response
✅ Защищать secrets и sensitive data
✅ Мониторить container security

**Security Monitoring Pillars:**
```

1. Prevention - Prevent attacks before they happen
2. Detection - Detect threats in real-time
3. Response - Respond quickly to incidents
4. Recovery - Recover and learn from incidents
5. Compliance - Meet regulatory requirements

```

**Key Security Metrics:**
```

- Failed authentication rate
- Privilege escalation attempts
- Unusual access patterns
- Vulnerability count by severity
- Certificate expiration
- Security scan failures
- Falco rule violations
- WAF block rate

```

**Production Security Checklist:**
- ✅ Audit logging для всех критичных операций
- ✅ Falco или аналог для runtime security
- ✅ Vulnerability scanning в CI/CD
- ✅ Secret scanning перед commit
- ✅ Security alerts в Prometheus/Alertmanager
- ✅ SIEM или centralized log analysis
- ✅ Regular compliance reports
- ✅ Incident response runbooks
- ✅ Security dashboard в Grafana
- ✅ Encryption at rest and in transit
- ✅ Regular security audits
- ✅ Penetration testing schedule


## Модуль 10: Advanced Topics - Observability as Code и Автоматизация (50 минут)

### 🎯 Напоминалка

**Observability as Code - что это:**

```
Observability as Code = Infrastructure as Code для мониторинга

Принципы:
1. Версионирование - все в Git
2. Переиспользование - DRY principle
3. Тестирование - validate перед apply
4. Automation - CI/CD для мониторинга
5. Self-service - команды сами управляют своим мониторингом

Что хранить как код:
├─ Dashboards (Grafana JSON)
├─ Alerts (Prometheus rules)
├─ Recording rules
├─ SLO definitions
├─ Runbooks (Markdown)
└─ Monitoring configuration
```

**GitOps для мониторинга:**

```
┌─────────────────────────────────────┐
│   Git Repository                    │
│   ├── dashboards/                   │
│   ├── alerts/                       │
│   ├── recording-rules/              │
│   └── slo/                          │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   CI/CD Pipeline                    │
│   ├── Validate syntax               │
│   ├── Test queries                  │
│   ├── Check best practices          │
│   └── Deploy to clusters            │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   Monitoring Stack                  │
│   ├── Prometheus                    │
│   ├── Grafana                       │
│   └── Alertmanager                  │
└─────────────────────────────────────┘
```

**Jsonnet для Grafana dashboards:**

jsonnet

```jsonnet
// Reusable dashboard library
local grafana = import 'grafonnet/grafana.libsonnet';
local dashboard = grafana.dashboard;
local prometheus = grafana.prometheus;

// Функция для создания стандартного dashboard
local createServiceDashboard(service_name) = 
  dashboard.new(
    'Service Dashboard - ' + service_name,
    tags=['service', service_name],
    refresh='30s',
  )
  .addPanel(
    // CPU panel
    grafana.graphPanel.new(
      'CPU Usage',
      datasource='Prometheus',
      format='percent',
    )
    .addTarget(
      prometheus.target(
        'rate(container_cpu_usage_seconds_total{service="' + service_name + '"}[5m]) * 100'
      )
    ), gridPos={x: 0, y: 0, w: 12, h: 8}
  )
  .addPanel(
    // Memory panel
    grafana.graphPanel.new(
      'Memory Usage',
      datasource='Prometheus',
      format='bytes',
    )
    .addTarget(
      prometheus.target(
        'container_memory_working_set_bytes{service="' + service_name + '"}'
      )
    ), gridPos={x: 12, y: 0, w: 12, h: 8}
  );

// Generate dashboards для всех сервисов
{
  'payment-api-dashboard.json': createServiceDashboard('payment-api'),
  'user-api-dashboard.json': createServiceDashboard('user-api'),
  'order-api-dashboard.json': createServiceDashboard('order-api'),
}
```

**Terraform для Grafana:**

hcl

```hcl
# Provider configuration
terraform {
  required_providers {
    grafana = {
      source = "grafana/grafana"
      version = "~> 2.0"
    }
  }
}

provider "grafana" {
  url  = "http://grafana.example.com"
  auth = var.grafana_api_key
}

# Datasource
resource "grafana_data_source" "prometheus" {
  type = "prometheus"
  name = "Prometheus"
  url  = "http://prometheus:9090"
  
  json_data_encoded = jsonencode({
    httpMethod    = "POST"
    timeInterval  = "30s"
  })
}

# Dashboard from file
resource "grafana_dashboard" "service_overview" {
  config_json = file("${path.module}/dashboards/service-overview.json")
  
  folder = grafana_folder.services.id
}

# Folder
resource "grafana_folder" "services" {
  title = "Service Dashboards"
}

# Alert notification channel
resource "grafana_notification_channel" "slack" {
  name = "Slack Alerts"
  type = "slack"
  
  settings = {
    url         = var.slack_webhook_url
    recipient   = "#alerts"
    uploadImage = true
  }
}

# Variables
variable "grafana_api_key" {
  type      = string
  sensitive = true
}

variable "slack_webhook_url" {
  type      = string
  sensitive = true
}
```

**Prometheus Operator CRDs:**

yaml

```yaml
# PrometheusRule - alerts as code
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: app-alerts
  namespace: monitoring
spec:
  groups:
  - name: app.rules
    interval: 30s
    rules:
    - alert: HighErrorRate
      expr: |
        rate(http_requests_total{status=~"5.."}[5m]) 
        / 
        rate(http_requests_total[5m]) > 0.05
      for: 5m
      labels:
        severity: warning
        team: backend
      annotations:
        summary: "High error rate on {{ $labels.service }}"

# ServiceMonitor - auto-discovery
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-metrics
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics

# PodMonitor - для pods без service
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: pod-metrics
  namespace: monitoring
spec:
  selector:
    matchLabels:
      monitoring: enabled
  podMetricsEndpoints:
  - port: metrics
    interval: 30s
```

**Monitoring configuration validation:**

python

```python
#!/usr/bin/env python3
"""
Validate monitoring configuration before deployment
"""
import yaml
import json
import sys
from pathlib import Path

class MonitoringValidator:
    def __init__(self):
        self.errors = []
        self.warnings = []
    
    def validate_prometheus_rules(self, rules_file):
        """Validate Prometheus alert rules"""
        with open(rules_file) as f:
            rules = yaml.safe_load(f)
        
        for group in rules.get('groups', []):
            # Check group name
            if not group.get('name'):
                self.errors.append(f"Group missing name in {rules_file}")
            
            for rule in group.get('rules', []):
                # Check required fields
                if 'alert' in rule:
                    self._validate_alert(rule, rules_file)
                elif 'record' in rule:
                    self._validate_recording_rule(rule, rules_file)
    
    def _validate_alert(self, alert, filename):
        """Validate alert rule"""
        required = ['alert', 'expr', 'labels', 'annotations']
        for field in required:
            if field not in alert:
                self.errors.append(
                    f"Alert {alert.get('alert', 'unknown')} missing {field} in {filename}"
                )
        
        # Check severity label
        if 'labels' in alert:
            if 'severity' not in alert['labels']:
                self.warnings.append(
                    f"Alert {alert['alert']} missing severity label"
                )
            
            valid_severities = ['critical', 'warning', 'info']
            if alert['labels'].get('severity') not in valid_severities:
                self.errors.append(
                    f"Alert {alert['alert']} has invalid severity"
                )
        
        # Check for duration
        if 'for' not in alert:
            self.warnings.append(
                f"Alert {alert['alert']} missing 'for' clause - will fire immediately"
            )
        
        # Check annotations
        if 'annotations' in alert:
            required_annotations = ['summary', 'description']
            for anno in required_annotations:
                if anno not in alert['annotations']:
                    self.warnings.append(
                        f"Alert {alert['alert']} missing '{anno}' annotation"
                    )
    
    def _validate_recording_rule(self, rule, filename):
        """Validate recording rule"""
        if not rule.get('record'):
            self.errors.append(f"Recording rule missing name in {filename}")
        
        if not rule.get('expr'):
            self.errors.append(f"Recording rule missing expr in {filename}")
        
        # Check naming convention
        record_name = rule.get('record', '')
        if not ':' in record_name:
            self.warnings.append(
                f"Recording rule {record_name} doesn't follow naming convention (level:metric:operations)"
            )
    
    def validate_grafana_dashboard(self, dashboard_file):
        """Validate Grafana dashboard"""
        with open(dashboard_file) as f:
            dashboard = json.load(f)
        
        # Check required fields
        if 'dashboard' in dashboard:
            dash = dashboard['dashboard']
        else:
            dash = dashboard
        
        if not dash.get('title'):
            self.errors.append(f"Dashboard missing title in {dashboard_file}")
        
        if not dash.get('panels'):
            self.warnings.append(f"Dashboard has no panels in {dashboard_file}")
        
        # Check panel queries
        for panel in dash.get('panels', []):
            if 'targets' in panel:
                for target in panel['targets']:
                    if not target.get('expr'):
                        self.warnings.append(
                            f"Panel {panel.get('title', 'unknown')} has empty query"
                        )
    
    def validate_slo_config(self, slo_file):
        """Validate SLO configuration"""
        with open(slo_file) as f:
            slo = yaml.safe_load(f)
        
        required_fields = ['service', 'slo', 'error_budget_policy']
        for field in required_fields:
            if field not in slo:
                self.errors.append(f"SLO config missing {field} in {slo_file}")
        
        # Validate SLO targets
        if 'slo' in slo:
            for slo_type, config in slo['slo'].items():
                if 'target' not in config:
                    self.errors.append(f"SLO {slo_type} missing target")
                
                target = config.get('target', 0)
                if not (0 < target <= 1):
                    self.errors.append(f"SLO {slo_type} target must be between 0 and 1")
    
    def report(self):
        """Print validation report"""
        print("\n" + "="*60)
        print("MONITORING CONFIGURATION VALIDATION REPORT")
        print("="*60 + "\n")
        
        if self.errors:
            print(f"❌ ERRORS ({len(self.errors)}):")
            for error in self.errors:
                print(f"  - {error}")
            print()
        
        if self.warnings:
            print(f"⚠️  WARNINGS ({len(self.warnings)}):")
            for warning in self.warnings:
                print(f"  - {warning}")
            print()
        
        if not self.errors and not self.warnings:
            print("✅ All validations passed!")
        
        print("="*60 + "\n")
        
        return len(self.errors) == 0

# Usage
if __name__ == "__main__":
    validator = MonitoringValidator()
    
    # Validate all configs
    for rules_file in Path('alerts/').glob('*.yaml'):
        validator.validate_prometheus_rules(rules_file)
    
    for dashboard_file in Path('dashboards/').glob('*.json'):
        validator.validate_grafana_dashboard(dashboard_file)
    
    for slo_file in Path('slo/').glob('*.yaml'):
        validator.validate_slo_config(slo_file)
    
    # Report and exit
    success = validator.report()
    sys.exit(0 if success else 1)
```

**CI/CD для мониторинга:**

`.github/workflows/monitoring-ci.yml`:

yaml

```yaml
name: Monitoring CI/CD

on:
  pull_request:
    paths:
      - 'monitoring/**'
  push:
    branches:
      - main
    paths:
      - 'monitoring/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    - name: Install dependencies
      run: |
        pip install pyyaml jsonschema promtool-cli
    
    - name: Validate Prometheus rules
      run: |
        promtool check rules monitoring/alerts/*.yaml
    
    - name: Validate dashboards
      run: |
        python scripts/validate_monitoring.py
    
    - name: Lint Jsonnet
      uses: jsonnet-libs/jsonnet-action@main
      with:
        files: monitoring/dashboards/*.jsonnet
    
    - name: Test queries
      run: |
        python scripts/test_queries.py
  
  preview:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
    - uses: actions/checkout@v3
    
    - name: Generate dashboard previews
      run: |
        docker run --rm -v $PWD:/workspace \
          grafana/grafonnet-lib:latest \
          jsonnet -J /workspace/vendor \
          /workspace/monitoring/dashboards/*.jsonnet
    
    - name: Comment PR with changes
      uses: actions/github-script@v6
      with:
        script: |
          const fs = require('fs');
          const changes = fs.readFileSync('changes.txt', 'utf8');
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: `## Monitoring Changes\n\n${changes}`
          });
  
  deploy-staging:
    needs: validate
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: staging
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to staging
      run: |
        kubectl apply -f monitoring/alerts/ --context=staging
        
        # Reload Prometheus
        kubectl rollout restart deployment/prometheus -n monitoring --context=staging
    
    - name: Verify deployment
      run: |
        sleep 30
        # Check Prometheus is healthy
        kubectl exec -n monitoring prometheus-0 --context=staging -- \
          wget -q -O - http://localhost:9090/-/healthy
  
  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to production
      run: |
        kubectl apply -f monitoring/alerts/ --context=production
        kubectl rollout restart deployment/prometheus -n monitoring --context=production
    
    - name: Verify and rollback if needed
      run: |
        sleep 60
        if ! kubectl exec -n monitoring prometheus-0 --context=production -- \
          wget -q -O - http://localhost:9090/-/healthy; then
          echo "Deployment failed, rolling back"
          git revert HEAD
          kubectl apply -f monitoring/alerts/ --context=production
          exit 1
        fi
```

**Automated dashboard generation:**

python

```python
#!/usr/bin/env python3
"""
Generate Grafana dashboards from service metadata
"""
import json
import yaml
from typing import Dict, List

class DashboardGenerator:
    def __init__(self):
        self.dashboard_id = 0
        self.panel_id = 0
    
    def generate_service_dashboard(self, service_config: Dict) -> Dict:
        """Generate dashboard from service configuration"""
        service_name = service_config['name']
        
        dashboard = {
            'dashboard': {
                'id': None,
                'uid': f"{service_name}-overview",
                'title': f"{service_name.title()} - Service Overview",
                'tags': ['generated', 'service', service_name],
                'timezone': 'browser',
                'refresh': '30s',
                'time': {
                    'from': 'now-6h',
                    'to': 'now'
                },
                'panels': []
            }
        }
        
        y_pos = 0
        
        # Add standard panels
        dashboard['dashboard']['panels'].append(
            self._create_request_rate_panel(service_name, y_pos)
        )
        y_pos += 8
        
        dashboard['dashboard']['panels'].append(
            self._create_error_rate_panel(service_name, y_pos)
        )
        y_pos += 8
        
        dashboard['dashboard']['panels'].append(
            self._create_latency_panel(service_name, y_pos)
        )
        y_pos += 8
        
        # Add custom metrics from config
        for metric in service_config.get('custom_metrics', []):
            dashboard['dashboard']['panels'].append(
                self._create_custom_panel(metric, service_name, y_pos)
            )
            y_pos += 8
        
        return dashboard
    
    def _create_request_rate_panel(self, service: str, y_pos: int) -> Dict:
        """Create request rate panel"""
        self.panel_id += 1
        return {
            'id': self.panel_id,
            'gridPos': {'x': 0, 'y': y_pos, 'w': 12, 'h': 8},
            'type': 'timeseries',
            'title': 'Request Rate',
            'targets': [{
                'expr': f'sum(rate(http_requests_total{{service="{service}"}}[5m]))',
                'legendFormat': 'Requests/sec',
                'refId': 'A'
            }],
            'fieldConfig': {
                'defaults': {
                    'unit': 'reqps',
                    'custom': {
                        'lineWidth': 2,
                        'fillOpacity': 10
                    }
                }
            }
        }
    
    def _create_error_rate_panel(self, service: str, y_pos: int) -> Dict:
        """Create error rate panel"""
        self.panel_id += 1
        return {
            'id': self.panel_id,
            'gridPos': {'x': 12, 'y': y_pos, 'w': 12, 'h': 8},
            'type': 'timeseries',
            'title': 'Error Rate',
            'targets': [{
                'expr': f'''
                    sum(rate(http_requests_total{{service="{service}",status=~"5.."}}[5m]))
                    /
                    sum(rate(http_requests_total{{service="{service}"}}[5m]))
                ''',
                'legendFormat': 'Error %',
                'refId': 'A'
            }],
            'fieldConfig': {
                'defaults': {
                    'unit': 'percentunit',
                    'thresholds': {
                        'mode': 'absolute',
                        'steps': [
                            {'value': 0, 'color': 'green'},
                            {'value': 0.01, 'color': 'yellow'},
                            {'value': 0.05, 'color': 'red'}
                        ]
                    }
                }
            }
        }
    
    def _create_latency_panel(self, service: str, y_pos: int) -> Dict:
        """Create latency panel"""
        self.panel_id += 1
        return {
            'id': self.panel_id,
            'gridPos': {'x': 0, 'y': y_pos, 'w': 24, 'h': 8},
            'type': 'timeseries',
            'title': 'Latency Percentiles',
            'targets': [
                {
                    'expr': f'histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket{{service="{service}"}}[5m])) by (le))',
                    'legendFormat': 'P50',
                    'refId': 'A'
                },
                {
                    'expr': f'histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{{service="{service}"}}[5m])) by (le))',
                    'legendFormat': 'P95',
                    'refId': 'B'
                },
                {
                    'expr': f'histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket{{service="{service}"}}[5m])) by (le))',
                    'legendFormat': 'P99',
                    'refId': 'C'
                }
            ],
            'fieldConfig': {
                'defaults': {
                    'unit': 's'
                }
            }
        }
    
    def _create_custom_panel(self, metric: Dict, service: str, y_pos: int) -> Dict:
        """Create custom metric panel"""
        self.panel_id += 1
        return {
            'id': self.panel_id,
            'gridPos': {'x': 0, 'y': y_pos, 'w': 12, 'h': 8},
            'type': metric.get('type', 'timeseries'),
            'title': metric['title'],
            'targets': [{
                'expr': metric['query'].replace('$service', service),
                'legendFormat': metric.get('legend', 'Value'),
                'refId': 'A'
            }],
            'fieldConfig': {
                'defaults': {
                    'unit': metric.get('unit', 'short')
                }
            }
        }

# Usage
if __name__ == "__main__":
    # Load service configurations
    with open('services.yaml') as f:
        services = yaml.safe_load(f)
    
    generator = DashboardGenerator()
    
    # Generate dashboards for all services
    for service in services['services']:
        dashboard = generator.generate_service_dashboard(service)
        
        # Save to file
        filename = f"dashboards/generated/{service['name']}-dashboard.json"
        with open(filename, 'w') as f:
            json.dump(dashboard, f, indent=2)
        
        print(f"✅ Generated dashboard: {filename}")
```

`services.yaml` example:

yaml

```yaml
services:
  - name: payment-api
    type: http
    slo:
      availability: 0.999
      latency_p95: 200ms
    custom_metrics:
      - title: "Payment Success Rate"
        query: 'sum(rate(payments_success_total{service="$service"}[5m])) / sum(rate(payments_total{service="$service"}[5m]))'
        type: "gauge"
        unit: "percentunit"
      
      - title: "Transaction Amount"
        query: 'sum(rate(payment_amount_total{service="$service"}[5m]))'
        type: "timeseries"
        unit: "currencyUSD"
  
  - name: user-api
    type: http
    slo:
      availability: 0.99
      latency_p95: 500ms
    custom_metrics:
      - title: "Active Sessions"
        query: 'count(user_sessions{service="$service",status="active"})'
        type: "stat"
```

**Testing Prometheus queries:**

python

```python
#!/usr/bin/env python3
"""
Test Prometheus queries before deployment
"""
import requests
import yaml
from datetime import datetime

class QueryTester:
    def __init__(self, prometheus_url):
        self.prometheus_url = prometheus_url
    
    def test_query(self, query: str) -> tuple:
        """Test if query is valid and returns data"""
        try:
            response = requests.get(
                f"{self.prometheus_url}/api/v1/query",
                params={'query': query}
            )
            
            if response.status_code != 200:
                return False, f"HTTP {response.status_code}"
            
            data = response.json()
            
            if data['status'] != 'success':
                return False, data.get('error', 'Unknown error')
            
            if not data['data']['result']:
                return False, "No data returned"
            
            return True, "OK"
        
        except Exception as e:
            return False, str(e)
    
    def test_alert_rules(self, rules_file: str):
        """Test all queries in alert rules"""
        with open(rules_file) as f:
            rules = yaml.safe_load(f)
        
        results = []
        
        for group in rules.get('groups', []):
            for rule in group.get('rules', []):
                query = rule.get('expr')
                rule_name = rule.get('alert') or rule.get('record')
                
                success, message = self.test_query(query)
                
                results.append({
                    'rule': rule_name,
                    'success': success,
                    'message': message,
                    'query': query
                })
        
        return results
    
    def report(self, results):
        """Print test report"""
        print("\n" + "="*60)
        print("QUERY TESTING REPORT")
        print("="*60 + "\n")
        
        passed = sum(1 for r in results if r['success'])
        failed = len(results) - passed
        
        print(f"Total: {len(results)} | Passed: {passed} | Failed: {failed}\n")
        
        if failed > 0:
            print("❌ FAILED QUERIES:\n")
            for result in results:
                if not result['success']:
                    print(f"  Rule: {result['rule']}")
                    print(f"  Error: {result['message']}")
                    print(f"  Query: {result['query'][:100]}...")
                    print()
        
        print("="*60 + "\n")
        
        return failed == 0

# Usage
if __name__ == "__main__":
    tester = QueryTester("http://localhost:9090")
    
    results = tester.test_alert_rules("alerts/app-alerts.yaml")
    success = tester.report(results)
    
    exit(0 if success else 1)
```

### 💻 Задание

Создай полноценную систему Observability as Code:

1. **Структура проекта**:

bash

```bash
monitoring-as-code/
├── .github/
│   └── workflows/
│       └── monitoring-ci.yml
├── alerts/
│   ├── infrastructure.yaml
│   ├── applications.yaml
│   └── slo.yaml
├── dashboards/
│   ├── jsonnet/
│   │   ├── lib/
│   │   │   └── common.libsonnet
│   │   └── services/
│   │       ├── payment-api.jsonnet
│   │       └── user-api.jsonnet
│   └── generated/
├── recording-rules/
│   ├── slo-rules.yaml
│   └── performance-rules.yaml
├── slo/
│   ├── payment-api.yaml
│   └── user-api.yaml
├── scripts/
│   ├── validate.py
│   ├── test-queries.py
│   └── generate-dashboards.py
├── terraform/
│   ├── grafana.tf
│   └── prometheus.tf
└── README.md
```

2. **Создай reusable dashboard library**:

`dashboards/jsonnet/lib/common.libsonnet`:

jsonnet

```jsonnet
{
  // Standard панель для CPU
  cpuPanel(service, y_pos=0)::
    {
      id: 1,
      gridPos: { x: 0, y: y_pos, w: 12, h: 8 },
      type: 'timeseries',
      title: 'CPU Usage',
      targets: [{
        expr: 'rate(container_cpu_usage_seconds_total{service="' + service + '"}[5m]) * 100',
        legendFormat: 'CPU %',
        refId: 'A',
      }],
      fieldConfig: {
        defaults: {
          unit: 'percent',
          thresholds: {
            steps: [
              { value: 0, color: 'green' },
              { value: 70, color: 'yellow' },
              { value: 90, color: 'red' },
            ],
          },
        },
      },
    },

  // Standard панель для Memory
  memoryPanel(service, y_pos=0)::
    {
      id: 2,
      gridPos: { x: 12, y: y_pos, w: 12, h: 8 },
      type: 'timeseries',
      title: 'Memory Usage',
      targets: [{
        expr: 'container_memory_working_set_bytes{service="' + service + '"}',
        legendFormat: 'Memory',
        refId: 'A',
      }],
      fieldConfig: {
        defaults: {
          unit: 'bytes',
        },
      },
    },

  // Standard панель для Request Rate
  requestRatePanel(service, y_pos=0)::
    {
      id: 3,
      gridPos: { x: 0, y: y_pos, w: 12, h: 8 },
      type: 'timeseries',
      title: 'Request Rate',
      targets: [{
        expr: 'sum(rate(http_requests_total{service="' + service + '"}[5m]))',
        legendFormat: 'Req/s',
        refId: 'A',
      }],
      fieldConfig: {
        defaults: {
          unit: 'reqps',
        },
      },
    },

  // Standard панель для Error Rate
  errorRatePanel(service, y_pos=0)::
    {
      id: 4,
      gridPos: { x: 12, y: y_pos, w: 12, h: 8 },
      type: 'timeseries',
      title: 'Error Rate',
      targets: [{
        expr: |||
          sum(rate(http_requests_total{service="%s",status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total{service="%s"}[5m]))
        ||| % [service, service],
        legendFormat: 'Error %',
        refId: 'A',
      }],
      fieldConfig: {
        defaults: {
          unit: 'percentunit',
          thresholds: {
            steps: [
              { value: 0, color: 'green' },
              { value: 0.01, color: 'yellow' },
              { value: 0.05, color: 'red' }, ], }, }, }, },

// Template для полного service dashboard serviceDashboard(name, panels=[]):: { dashboard: { title: name + ' - Service Dashboard', tags: ['service', name, 'generated'], timezone: 'browser', refresh: '30s', panels: panels, }, }, }

````

3. **Используй library для генерации dashboards**:

`dashboards/jsonnet/services/payment-api.jsonnet`:
```jsonnet
local common = import '../lib/common.libsonnet';

local service = 'payment-api';

common.serviceDashboard(service, [
  common.requestRatePanel(service, y_pos=0),
  common.errorRatePanel(service, y_pos=0),
  common.cpuPanel(service, y_pos=8),
  common.memoryPanel(service, y_pos=8),
  
  // Custom панель для payment-specific метрик
  {
    id: 5,
    gridPos: { x: 0, y: 16, w: 24, h: 8 },
    type: 'timeseries',
    title: 'Payment Success Rate',
    targets: [{
      expr: 'sum(rate(payments_success_total{service="' + service + '"}[5m])) / sum(rate(payments_total{service="' + service + '"}[5m]))',
      legendFormat: 'Success Rate',
      refId: 'A',
    }],
    fieldConfig: {
      defaults: {
        unit: 'percentunit',
        min: 0.9,
        max: 1.0,
      },
    },
  },
])
```

4. **Build script для Jsonnet**:

`scripts/build-dashboards.sh`:
```bash
#!/bin/bash

set -e

JSONNET_DIR="dashboards/jsonnet"
OUTPUT_DIR="dashboards/generated"

mkdir -p "$OUTPUT_DIR"

echo "Building Grafana dashboards from Jsonnet..."

for jsonnet_file in $JSONNET_DIR/services/*.jsonnet; do
  filename=$(basename "$jsonnet_file" .jsonnet)
  output_file="$OUTPUT_DIR/${filename}-dashboard.json"
  
  echo "  Building $filename..."
  jsonnet -J "$JSONNET_DIR/lib" "$jsonnet_file" > "$output_file"
done

echo "✅ All dashboards built successfully!"
```

5. **Automated alert generation from SLO**:

`scripts/generate-slo-alerts.py`:
```python
#!/usr/bin/env python3
"""
Generate Prometheus alert rules from SLO definitions
"""
import yaml
from pathlib import Path

def generate_burn_rate_alerts(service_name, slo_config):
    """Generate multi-window multi-burn-rate alerts"""
    target = slo_config['availability']['target']
    error_budget = 1 - target
    
    alerts = []
    
    # Fast burn (14.4x)
    alerts.append({
        'alert': f'{service_name.replace("-", "_")}_ErrorBudgetFastBurn',
        'expr': f'''
            (
              sum(rate(http_requests_total{{service="{service_name}",status=~"5.."}}[5m]))
              /
              sum(rate(http_requests_total{{service="{service_name}"}}[5m]))
            ) / {error_budget} > 14.4
            and
            (
              sum(rate(http_requests_total{{service="{service_name}",status=~"5.."}}[1h]))
              /
              sum(rate(http_requests_total{{service="{service_name}"}}[1h]))
            ) / {error_budget} > 14.4
        ''',
        'for': '2m',
        'labels': {
            'severity': 'critical',
            'service': service_name,
            'slo_type': 'availability'
        },
        'annotations': {
            'summary': f'{service_name} burning error budget too fast',
            'description': 'Error budget will be exhausted in < 2 days',
            'runbook': f'https://runbooks.example.com/{service_name}/error-budget-burn'
        }
    })
    
    # Slow burn (6x)
    alerts.append({
        'alert': f'{service_name.replace("-", "_")}_ErrorBudgetSlowBurn',
        'expr': f'''
            (
              sum(rate(http_requests_total{{service="{service_name}",status=~"5.."}}[30m]))
              /
              sum(rate(http_requests_total{{service="{service_name}"}}[30m]))
            ) / {error_budget} > 6
            and
            (
              sum(rate(http_requests_total{{service="{service_name}",status=~"5.."}}[6h]))
              /
              sum(rate(http_requests_total{{service="{service_name}"}}[6h]))
            ) / {error_budget} > 6
        ''',
        'for': '15m',
        'labels': {
            'severity': 'warning',
            'service': service_name,
            'slo_type': 'availability'
        },
        'annotations': {
            'summary': f'{service_name} burning error budget steadily',
            'description': 'Error budget will be exhausted in < 1 week'
        }
    })
    
    return alerts

def generate_latency_alerts(service_name, slo_config):
    """Generate latency SLO alerts"""
    if 'latency' not in slo_config:
        return []
    
    latency_config = slo_config['latency']
    threshold = latency_config['threshold_ms'] / 1000  # Convert to seconds
    
    return [{
        'alert': f'{service_name.replace("-", "_")}_LatencySLOViolation',
        'expr': f'''
            histogram_quantile(0.95,
              sum(rate(http_request_duration_seconds_bucket{{service="{service_name}"}}[5m])) by (le)
            ) > {threshold}
        ''',
        'for': '10m',
        'labels': {
            'severity': 'warning',
            'service': service_name,
            'slo_type': 'latency'
        },
        'annotations': {
            'summary': f'{service_name} latency SLO violation',
            'description': f'P95 latency is above {latency_config["threshold_ms"]}ms'
        }
    }]

def main():
    output_file = 'alerts/generated-slo-alerts.yaml'
    all_alerts = []
    
    # Load all SLO configs
    for slo_file in Path('slo/').glob('*.yaml'):
        with open(slo_file) as f:
            slo_config = yaml.safe_load(f)
        
        service_name = slo_config['service']
        print(f"Generating alerts for {service_name}...")
        
        # Generate alerts
        alerts = []
        alerts.extend(generate_burn_rate_alerts(service_name, slo_config))
        alerts.extend(generate_latency_alerts(service_name, slo_config))
        
        all_alerts.extend(alerts)
    
    # Write to file
    output = {
        'groups': [{
            'name': 'generated_slo_alerts',
            'interval': '30s',
            'rules': all_alerts
        }]
    }
    
    Path('alerts').mkdir(exist_ok=True)
    with open(output_file, 'w') as f:
        yaml.dump(output, f, default_flow_style=False, sort_keys=False)
    
    print(f"\n✅ Generated {len(all_alerts)} alerts → {output_file}")

if __name__ == "__main__":
    main()
```

6. **Complete CI/CD pipeline**:
```bash
# Makefile для удобства

.PHONY: all validate test build deploy clean

all: validate test build

validate:
	@echo "Validating monitoring configuration..."
	python scripts/validate.py
	promtool check rules alerts/*.yaml
	promtool check rules recording-rules/*.yaml

test:
	@echo "Testing Prometheus queries..."
	python scripts/test-queries.py

build:
	@echo "Building dashboards from Jsonnet..."
	./scripts/build-dashboards.sh
	
	@echo "Generating SLO alerts..."
	python scripts/generate-slo-alerts.py

deploy-staging:
	@echo "Deploying to staging..."
	kubectl apply -f alerts/ --context=staging --namespace=monitoring
	kubectl apply -f recording-rules/ --context=staging --namespace=monitoring
	./scripts/upload-dashboards.sh staging

deploy-production:
	@echo "Deploying to production..."
	kubectl apply -f alerts/ --context=production --namespace=monitoring
	kubectl apply -f recording-rules/ --context=production --namespace=monitoring
	./scripts/upload-dashboards.sh production

clean:
	rm -rf dashboards/generated/*
	rm -f alerts/generated-*.yaml
```

7. **Testing**:
```bash
# Clone repo
git clone <monitoring-as-code-repo>
cd monitoring-as-code

# Validate
make validate

# Test queries
make test

# Build dashboards
make build

# Check generated files
ls dashboards/generated/
ls alerts/generated-*

# Deploy to staging
make deploy-staging

# If all good, deploy to production
make deploy-production
```

### 🚀 Бонус (новое)

**1. Auto-scaling monitoring infrastructure**:

`terraform/prometheus-autoscaling.tf`:
```hcl
resource "kubernetes_horizontal_pod_autoscaler_v2" "prometheus" {
  metadata {
    name      = "prometheus"
    namespace = "monitoring"
  }

  spec {
    scale_target_ref {
      api_version = "apps/v1"
      kind        = "Deployment"
      name        = "prometheus"
    }

    min_replicas = 2
    max_replicas = 10

    metric {
      type = "Resource"
      resource {
        name = "cpu"
        target {
          type                = "Utilization"
          average_utilization = 70
        }
      }
    }

    metric {
      type = "Resource"
      resource {
        name = "memory"
        target {
          type                = "Utilization"
          average_utilization = 80
        }
      }
    }

    behavior {
      scale_up {
        stabilization_window_seconds = 60
        select_policy                = "Max"
        policy {
          type          = "Percent"
          value         = 100
          period_seconds = 60
        }
      }

      scale_down {
        stabilization_window_seconds = 300
        select_policy                = "Min"
        policy {
          type          = "Percent"
          value         = 10
          period_seconds = 60
        }
      }
    }
  }
}
```

**2. Monitoring cost optimization**:

`scripts/analyze-costs.py`:
```python
#!/usr/bin/env python3
"""
Analyze monitoring costs and suggest optimizations
"""
import requests
from datetime import datetime, timedelta

class CostAnalyzer:
    def __init__(self, prometheus_url):
        self.prom = prometheus_url
    
    def analyze_metric_cardinality(self):
        """Find high cardinality metrics"""
        query = 'count by (__name__) ({__name__!=""})'
        response = requests.get(
            f"{self.prom}/api/v1/query",
            params={'query': query}
        )
        
        metrics = response.json()['data']['result']
        high_cardinality = [
            m for m in metrics 
            if int(m['value'][1]) > 10000
        ]
        
        print("\n🔍 High Cardinality Metrics (>10k series):")
        for metric in sorted(high_cardinality, 
                           key=lambda x: int(x['value'][1]), 
                           reverse=True)[:10]:
            print(f"  {metric['metric']['__name__']}: {metric['value'][1]} series")
    
    def analyze_unused_metrics(self):
        """Find metrics that are never queried"""
        # This would require query logs analysis
        print("\n📊 Metrics Analysis:")
        print("  Recommendation: Enable Prometheus query logging")
        print("  kubectl edit prometheus -n monitoring")
        print("  Add: --enable-feature=query-logging")
    
    def estimate_storage_costs(self):
        """Estimate storage costs"""
        # Get TSDB size
        response = requests.get(f"{self.prom}/api/v1/status/tsdb")
        tsdb_stats = response.json()['data']
        
        total_series = tsdb_stats['seriesCountByMetricName']
        total_count = sum(total_series[i]['value'] for i in range(len(total_series)))
        
        # Rough estimate: 1-2 bytes per sample
        samples_per_day = total_count * 86400 / 15  # 15s scrape interval
        storage_per_day_gb = (samples_per_day * 1.5) / 1024 / 1024 / 1024
        
        print(f"\n💰 Storage Cost Estimate:")
        print(f"  Total series: {total_count:,}")
        print(f"  Est. storage/day: {storage_per_day_gb:.2f} GB")
        print(f"  Est. storage/month: {storage_per_day_gb * 30:.2f} GB")
        print(f"  Est. cost/month: ${storage_per_day_gb * 30 * 0.10:.2f} (at $0.10/GB)")

if __name__ == "__main__":
    analyzer = CostAnalyzer("http://localhost:9090")
    
    analyzer.analyze_metric_cardinality()
    analyzer.analyze_unused_metrics()
    analyzer.estimate_storage_costs()
```

---

## Итоги модуля 10

После прохождения этого модуля ты должен уметь:

✅ Применять Observability as Code подход
✅ Хранить мониторинг конфигурацию в Git
✅ Использовать Jsonnet для генерации dashboards
✅ Автоматизировать создание alerts из SLO
✅ Настраивать CI/CD для мониторинга
✅ Валидировать конфигурацию перед deploy
✅ Тестировать Prometheus queries
✅ Использовать Terraform для Grafana
✅ Генерировать dashboards программно
✅ Оптимизировать затраты на мониторинг
✅ Автоматизировать routine задачи
✅ Масштабировать monitoring infrastructure

**Observability as Code Benefits:**
```

1. Версионирование - history и rollback
2. Переиспользование - DRY principle
3. Консистентность - одинаково везде
4. Тестирование - catch bugs early
5. Автоматизация - less manual work
6. Документация - код = документация
7. Collaboration - code review процесс

```

**Production Checklist:**
- ✅ Все dashboards в Git
- ✅ Все alerts в Git
- ✅ CI/CD pipeline для мониторинга
- ✅ Validation перед deployment
- ✅ Query testing automated
- ✅ Dashboard generation automated
- ✅ SLO-based alert generation
- ✅ Terraform для infrastructure
- ✅ Rollback mechanism
- ✅ Cost monitoring enabled
- ✅ Documentation as code
- ✅ Regular reviews и cleanup


## Модуль 11: Database и Storage Monitoring - мониторинг данных и производительности (45 минут)

### 🎯 Напоминалка

**Почему Database Monitoring критичен:**

```
Database = Сердце приложения

Проблемы с БД → Проблемы везде:
- Медленные запросы → Slow API
- Высокая нагрузка → Timeouts
- Deadlocks → Failed transactions
- Полный диск → Downtime
- Connection pool exhausted → Service unavailable

Database downtime = Application downtime
```

**Основные метрики баз данных:**

```
┌─────────────────────────────────────┐
│   Performance Metrics               │
├─────────────────────────────────────┤
│ • Query response time (latency)     │
│ • Queries per second (throughput)   │
│ • Slow queries count                │
│ • Cache hit ratio                   │
│ • Index usage                       │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│   Resource Metrics                  │
├─────────────────────────────────────┤
│ • CPU usage                         │
│ • Memory usage                      │
│ • Disk I/O (IOPS, throughput)       │
│ • Network I/O                       │
│ • Connection count                  │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│   Availability Metrics              │
├─────────────────────────────────────┤
│ • Uptime                            │
│ • Replication lag                   │
│ • Failed connections                │
│ • Lock wait time                    │
│ • Deadlocks                         │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│   Capacity Metrics                  │
├─────────────────────────────────────┤
│ • Disk space usage                  │
│ • Table/index sizes                 │
│ • Transaction log size              │
│ • Connection pool usage             │
│ • Buffer pool usage                 │
└─────────────────────────────────────┘
```

**PostgreSQL мониторинг:**

**Ключевые метрики PostgreSQL:**

sql

```sql
-- Active connections
SELECT count(*) FROM pg_stat_activity WHERE state = 'active';

-- Long running queries
SELECT 
  pid,
  now() - query_start AS duration,
  query,
  state
FROM pg_stat_activity
WHERE state != 'idle'
  AND now() - query_start > interval '5 minutes'
ORDER BY duration DESC;

-- Database size
SELECT 
  pg_database.datname,
  pg_size_pretty(pg_database_size(pg_database.datname)) AS size
FROM pg_database
ORDER BY pg_database_size(pg_database.datname) DESC;

-- Table sizes
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;

-- Cache hit ratio (should be > 99%)
SELECT 
  sum(heap_blks_read) as heap_read,
  sum(heap_blks_hit) as heap_hit,
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) * 100 AS cache_hit_ratio
FROM pg_statio_user_tables;

-- Index usage
SELECT 
  schemaname,
  tablename,
  indexname,
  idx_scan as index_scans,
  idx_tup_read as tuples_read,
  idx_tup_fetch as tuples_fetched
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Unused indexes (candidates for deletion)
SELECT 
  schemaname,
  tablename,
  indexname,
  pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelid NOT IN (
    SELECT indexrelid FROM pg_index WHERE indisprimary
  )
ORDER BY pg_relation_size(indexrelid) DESC;

-- Bloat estimation
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
  pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - 
                 pg_relation_size(schemaname||'.'||tablename)) AS index_size
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;

-- Locks
SELECT 
  locktype,
  relation::regclass,
  mode,
  transactionid AS tid,
  virtualtransaction AS vtid,
  pid,
  granted
FROM pg_catalog.pg_locks
WHERE NOT granted;

-- Replication lag (if using replication)
SELECT 
  client_addr,
  state,
  pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) / 1024 / 1024 AS lag_mb
FROM pg_stat_replication;

-- Vacuum and analyze stats
SELECT 
  schemaname,
  tablename,
  last_vacuum,
  last_autovacuum,
  last_analyze,
  last_autoanalyze,
  n_dead_tup
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 10;
```

**PostgreSQL Exporter для Prometheus:**

yaml

```yaml
# postgres_exporter configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-exporter-queries
  namespace: monitoring
data:
  queries.yaml: |
    # Custom queries для postgres_exporter
    
    pg_database:
      query: |
        SELECT 
          datname,
          pg_database_size(datname) as size_bytes,
          numbackends as connections
        FROM pg_database
      metrics:
        - datname:
            usage: "LABEL"
            description: "Database name"
        - size_bytes:
            usage: "GAUGE"
            description: "Database size in bytes"
        - connections:
            usage: "GAUGE"
            description: "Number of backends currently connected"
    
    pg_slow_queries:
      query: |
        SELECT 
          COUNT(*) as count
        FROM pg_stat_activity
        WHERE state != 'idle'
          AND now() - query_start > interval '5 seconds'
      metrics:
        - count:
            usage: "GAUGE"
            description: "Number of queries running longer than 5 seconds"
    
    pg_cache_hit_ratio:
      query: |
        SELECT 
          sum(heap_blks_hit) / NULLIF(sum(heap_blks_hit) + sum(heap_blks_read), 0) * 100 as ratio
        FROM pg_statio_user_tables
      metrics:
        - ratio:
            usage: "GAUGE"
            description: "Cache hit ratio percentage"
    
    pg_replication_lag:
      query: |
        SELECT 
          client_addr,
          pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) as lag_bytes
        FROM pg_stat_replication
      metrics:
        - client_addr:
            usage: "LABEL"
            description: "Replica address"
        - lag_bytes:
            usage: "GAUGE"
            description: "Replication lag in bytes"
    
    pg_table_bloat:
      query: |
        SELECT 
          schemaname || '.' || tablename as table_name,
          n_dead_tup as dead_tuples,
          n_live_tup as live_tuples,
          CASE 
            WHEN n_live_tup > 0 
            THEN n_dead_tup::float / n_live_tup::float 
            ELSE 0 
          END as bloat_ratio
        FROM pg_stat_user_tables
        WHERE n_live_tup > 0
      metrics:
        - table_name:
            usage: "LABEL"
            description: "Table name"
        - dead_tuples:
            usage: "GAUGE"
            description: "Number of dead tuples"
        - live_tuples:
            usage: "GAUGE"
            description: "Number of live tuples"
        - bloat_ratio:
            usage: "GAUGE"
            description: "Ratio of dead to live tuples"
```

**MySQL мониторинг:**

sql

```sql
-- Connection stats
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';
SHOW VARIABLES LIKE 'max_connections';

-- Query performance
SELECT 
  DIGEST_TEXT as query,
  COUNT_STAR as exec_count,
  AVG_TIMER_WAIT/1000000000000 as avg_time_sec,
  SUM_ROWS_EXAMINED as rows_examined,
  SUM_ROWS_SENT as rows_sent
FROM performance_schema.events_statements_summary_by_digest
ORDER BY AVG_TIMER_WAIT DESC
LIMIT 10;

-- Slow queries
SELECT 
  sql_text,
  current_schema,
  rows_examined,
  rows_sent,
  created
FROM performance_schema.events_statements_history
WHERE timer_wait > 5000000000000  -- 5 seconds in picoseconds
ORDER BY timer_wait DESC;

-- Table sizes
SELECT 
  table_schema,
  table_name,
  ROUND((data_length + index_length) / 1024 / 1024, 2) AS size_mb,
  table_rows
FROM information_schema.TABLES
WHERE table_schema NOT IN ('mysql', 'information_schema', 'performance_schema')
ORDER BY (data_length + index_length) DESC
LIMIT 10;

-- InnoDB buffer pool hit ratio
SHOW STATUS LIKE 'Innodb_buffer_pool_read_requests';
SHOW STATUS LIKE 'Innodb_buffer_pool_reads';
-- Hit ratio = (read_requests - reads) / read_requests * 100
-- Should be > 99%

-- Lock waits
SELECT 
  r.trx_id waiting_trx_id,
  r.trx_mysql_thread_id waiting_thread,
  r.trx_query waiting_query,
  b.trx_id blocking_trx_id,
  b.trx_mysql_thread_id blocking_thread,
  b.trx_query blocking_query
FROM information_schema.innodb_lock_waits w
INNER JOIN information_schema.innodb_trx b ON b.trx_id = w.blocking_trx_id
INNER JOIN information_schema.innodb_trx r ON r.trx_id = w.requesting_trx_id;

-- Replication status
SHOW SLAVE STATUS\G
-- Check: Seconds_Behind_Master should be 0 or low
```

**Redis мониторинг:**

bash

```bash
# Redis INFO command sections
redis-cli INFO

# Key metrics:
redis-cli INFO stats | grep -E 'total_commands_processed|instantaneous_ops_per_sec'
redis-cli INFO memory | grep -E 'used_memory|used_memory_peak|mem_fragmentation_ratio'
redis-cli INFO replication | grep -E 'role|connected_slaves|master_repl_offset'
redis-cli INFO clients | grep -E 'connected_clients|blocked_clients'
redis-cli INFO persistence | grep -E 'rdb_last_save_time|aof_enabled'

# Slowlog
redis-cli SLOWLOG GET 10

# Connected clients
redis-cli CLIENT LIST

# Key statistics
redis-cli --bigkeys

# Memory usage by key pattern
redis-cli --memkeys

# Hit rate
redis-cli INFO stats | grep keyspace_hits
redis-cli INFO stats | grep keyspace_misses
# Hit rate = hits / (hits + misses)
```

**MongoDB мониторинг:**

javascript

```javascript
// Connection stats
db.serverStatus().connections

// Current operations
db.currentOp()

// Slow queries (from profiler)
db.system.profile.find({millis: {$gt: 100}}).sort({ts: -1}).limit(10)

// Database stats
db.stats()

// Collection stats
db.collection.stats()

// Index usage
db.collection.aggregate([
  { $indexStats: {} }
])

// Replication lag
rs.status()
rs.printSlaveReplicationInfo()

// Lock statistics
db.serverStatus().locks

// WiredTiger cache
db.serverStatus().wiredTiger.cache
```

**PromQL для Database мониторинг:**

promql

```promql
# PostgreSQL

## Connection usage
pg_stat_database_numbackends / pg_settings_max_connections * 100

## Cache hit ratio
rate(pg_stat_database_blks_hit[5m]) 
/ 
(rate(pg_stat_database_blks_hit[5m]) + rate(pg_stat_database_blks_read[5m])) * 100

## Active queries
pg_stat_activity_count{state="active"}

## Long running queries
pg_slow_queries_count

## Replication lag
pg_replication_lag_bytes / 1024 / 1024  # Convert to MB

## Deadlocks
rate(pg_stat_database_deadlocks[5m])

## Transaction rate
rate(pg_stat_database_xact_commit[5m]) + rate(pg_stat_database_xact_rollback[5m])

## Disk usage
pg_database_size_bytes / 1024 / 1024 / 1024  # Convert to GB

# MySQL

## Connection usage
mysql_global_status_threads_connected / mysql_global_variables_max_connections * 100

## Buffer pool hit ratio
(
  mysql_global_status_innodb_buffer_pool_read_requests
  - 
  mysql_global_status_innodb_buffer_pool_reads
)
/
mysql_global_status_innodb_buffer_pool_read_requests * 100

## Query rate
rate(mysql_global_status_queries[5m])

## Slow queries
rate(mysql_global_status_slow_queries[5m])

## Replication lag
mysql_slave_status_seconds_behind_master

## Table locks
rate(mysql_global_status_table_locks_waited[5m])

# Redis

## Memory usage
redis_memory_used_bytes / redis_memory_max_bytes * 100

## Hit rate
rate(redis_keyspace_hits_total[5m])
/
(rate(redis_keyspace_hits_total[5m]) + rate(redis_keyspace_misses_total[5m])) * 100

## Connected clients
redis_connected_clients

## Commands per second
rate(redis_commands_processed_total[5m])

## Evicted keys
rate(redis_evicted_keys_total[5m])

## Replication lag
redis_master_repl_offset - redis_slave_repl_offset

# MongoDB

## Connection usage
mongodb_connections{state="current"} / mongodb_connections{state="available"} * 100

## Operation rate
rate(mongodb_op_counters_total[5m])

## Page faults
rate(mongodb_extra_info_page_faults[5m])

## Replication lag
mongodb_mongod_replset_member_replication_lag

## Lock queue
mongodb_locks_timeAcquiringMicros{type="Global"}
```

**Storage мониторинг (Disk I/O):**

promql

```promql
# Disk usage
(
  node_filesystem_size_bytes{mountpoint="/"}
  -
  node_filesystem_avail_bytes{mountpoint="/"}
)
/
node_filesystem_size_bytes{mountpoint="/"} * 100

# Disk I/O rate
rate(node_disk_read_bytes_total[5m])
rate(node_disk_written_bytes_total[5m])

# IOPS
rate(node_disk_reads_completed_total[5m])
rate(node_disk_writes_completed_total[5m])

# Disk latency
rate(node_disk_read_time_seconds_total[5m]) 
/ 
rate(node_disk_reads_completed_total[5m])

rate(node_disk_write_time_seconds_total[5m])
/
rate(node_disk_writes_completed_total[5m])

# Disk queue length
node_disk_io_time_weighted_seconds_total

# Inode usage
(
  node_filesystem_files{mountpoint="/"}
  -
  node_filesystem_files_free{mountpoint="/"}
)
/
node_filesystem_files{mountpoint="/"} * 100
```

### 💻 Задание

Настрой мониторинг PostgreSQL и Redis:

1. **Deploy PostgreSQL с мониторингом**:

`k8s-manifests/postgres-with-monitoring.yaml`:

yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: database

---
# PostgreSQL
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: database
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:16-alpine
        ports:
        - containerPort: 5432
          name: postgres
        env:
        - name: POSTGRES_DB
          value: "testdb"
        - name: POSTGRES_USER
          value: "postgres"
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
          limits:
            cpu: 2
            memory: 4Gi
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi

---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: database
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  clusterIP: None

---
# PostgreSQL Exporter
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-exporter
  namespace: database
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres-exporter
  template:
    metadata:
      labels:
        app: postgres-exporter
    spec:
      containers:
      - name: postgres-exporter
        image: prometheuscommunity/postgres-exporter:latest
        ports:
        - containerPort: 9187
          name: metrics
        env:
        - name: DATA_SOURCE_NAME
          value: "postgresql://postgres:password@postgres:5432/testdb?sslmode=disable"
        - name: PG_EXPORTER_EXTEND_QUERY_PATH
          value: "/etc/postgres-exporter/queries.yaml"
        volumeMounts:
        - name: queries
          mountPath: /etc/postgres-exporter
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
      volumes:
      - name: queries
        configMap:
          name: postgres-exporter-queries

---
apiVersion: v1
kind: Service
metadata:
  name: postgres-exporter
  namespace: database
  labels:
    app: postgres-exporter
spec:
  selector:
    app: postgres-exporter
  ports:
  - port: 9187
    targetPort: 9187
    name: metrics

---
# ServiceMonitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: postgres-exporter
  namespace: database
spec:
  selector:
    matchLabels:
      app: postgres-exporter
  endpoints:
  - port: metrics
    interval: 30s

---
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: database
type: Opaque
stringData:
  password: "your-secure-password"
```

2. **Deploy Redis с мониторингом**:

`k8s-manifests/redis-with-monitoring.yaml`:

yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: database
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
          name: redis
        command:
        - redis-server
        - "--maxmemory"
        - "512mb"
        - "--maxmemory-policy"
        - "allkeys-lru"
        resources:
          requests:
            cpu: 100m
            memory: 512Mi
          limits:
            cpu: 500m
            memory: 1Gi
      
      - name: redis-exporter
        image: oliver006/redis_exporter:latest
        ports:
        - containerPort: 9121
          name: metrics
        env:
        - name: REDIS_ADDR
          value: "localhost:6379"
        resources:
          requests:
            cpu: 50m
            memory: 64Mi
          limits:
            cpu: 100m
            memory: 128Mi

---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: database
  labels:
    app: redis
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
    name: redis
  - port: 9121
    targetPort: 9121
    name: metrics

---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: redis
  namespace: database
spec:
  selector:
    matchLabels:
      app: redis
  endpoints:
  - port: metrics
    interval: 30s
```

3. **Prometheus alert rules для баз данных**:

`k8s-manifests/database-alerts.yaml`:

yaml

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: database-alerts
  namespace: monitoring
spec:
  groups:
  - name: postgresql.rules
    interval: 30s
    rules:
    # Connection pool near limit
    - alert: PostgreSQLConnectionPoolNearLimit
      expr: |
        pg_stat_database_numbackends / pg_settings_max_connections * 100 > 80
      for: 5m
      labels:
        severity: warning
        component: database
      annotations:
        summary: "PostgreSQL connection pool near limit"
        description: "{{ $labels.instance }} using {{ $value | humanize }}% of connections"
    
    # Cache hit ratio low
    - alert: PostgreSQLLowCacheHitRatio
      expr: |
        rate(pg_stat_database_blks_hit[5m]) 
        / 
        (rate(pg_stat_database_blks_hit[5m]) + rate(pg_stat_database_blks_read[5m])) * 100 < 90
      for: 10m
      labels:
        severity: warning
        component: database
      annotations:
        summary: "PostgreSQL low cache hit ratio"
        description: "Cache hit ratio is {{ $value | humanize }}% (should be > 99%)"
    
    # Too many slow queries
    - alert: PostgreSQLTooManySlowQueries
      expr: |
        pg_slow_queries_count > 10
      for: 5m
      labels:
        severity: warning
        component: database
      annotations:
        summary: "PostgreSQL has many slow queries"
        description: "{{ $value }} queries running longer than 5 seconds"
    
    # Replication lag high
    - alert: PostgreSQLReplicationLagHigh
      expr: |
        pg_replication_lag_bytes / 1024 / 1024 > 100
      for: 5m
      labels:
        severity: critical
        component: database
      annotations:
        summary: "PostgreSQL replication lag high"
        description: "Replication lag is {{ $value | humanize }} MB"
    
    # Database size growing fast
    - alert: PostgreSQLDatabaseGrowingFast
      expr: |
        predict_linear(pg_database_size_bytes[1h], 24*3600) > 
        pg_database_size_bytes * 1.5
      for: 1h
      labels:
        severity: warning
        component: database
      annotations:
        summary: "PostgreSQL database growing fast"
        description: "Database {{ $labels.datname }} will grow 50% in 24h"
    
    # Deadlocks detected
    - alert: PostgreSQLDeadlocksDetected
      expr: |
        rate(pg_stat_database_deadlocks[5m]) > 0
      for: 1m
      labels:
        severity: warning
        component: database
      annotations:
        summary: "PostgreSQL deadlocks detected"
        description: "{{ $value }} deadlocks/sec in {{ $labels.datname }}"
  
  - name: redis.rules
    interval: 30s
    rules:
    # Memory usage high
    - alert: RedisMemoryUsageHigh
      expr: |
        redis_memory_used_bytes / redis_memory_max_bytes * 100 > 90
      for: 5m
      labels:
        severity: warning
        component: database
      annotations:
        summary: "Redis memory usage high"
        description: "{{ $labels.instance }} using {{ $value | humanize }}% memory"
    
    # Hit rate low
    - alert: RedisHitRateLow
      expr: |
        rate(redis_keyspace_hits_total[5m])
        /
        (rate(redis_keyspace_hits_total[5m]) + rate(redis_keyspace_misses_total[5m])) * 100 < 80
      for: 10m
      labels:
        severity: warning
        component: database
      annotations:
        summary: "Redis hit rate low"
        description: "Hit rate is {{ $value | humanize }}% (should be > 95%)"
    
    # Too many evicted keys
    - alert: RedisTooManyEvictions
      expr: |
        rate(redis_evicted_keys_total[5m]) > 10
      for: 5m
      labels:
        severity: warning
        component: database
      annotations:
        summary: "Redis evicting too many keys"
        description: "{{ $value }} keys/sec being evicted"
    
    # Replication lag
    - alert: RedisReplicationLag
      expr: |
        redis_master_repl_offset - redis_slave_repl_offset > 1000000
      for: 5m
      labels:
        severity: warning
        component: database
      annotations:
        summary: "Redis replication lag high"
        description: "Slave is {{ $value }} bytes behind master"
    
    # Too many connected clients
    - alert: RedisTooManyClients
      expr: |
        redis_connected_clients > redis_config_maxclients * 0.8
      for: 5m
      labels:
        severity: warning
        component: database
      annotations:
        summary: "Redis too many clients"
        description: "{{ $value }} clients connected (80% of max)"
```

4. **Load generator для тестирования**:

`test/database-load-generator.py`:

python

```python
#!/usr/bin/env python3
"""
Database load generator for testing monitoring
"""
import psycopg2
import redis
import time
import random
from threading import Thread

class PostgreSQLLoadGenerator:
    def __init__(self, host, database, user, password):
        self.conn_params = {
            'host': host,
            'database': database,
            'user': user,
            'password': password
        }
    
    def create_test_data(self):
        """Create test tables and data"""
        conn = psycopg2.connect(**self.conn_params)
        cur = conn.cursor()
        
        # Create test table
        cur.execute("""
            CREATE TABLE IF NOT EXISTS test_data (
                id SERIAL PRIMARY KEY,
                name VARCHAR(100),
                value INTEGER,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
        
        # Create index
        cur.execute("""
            CREATE INDEX IF NOT EXISTS idx_test_data_value 
            ON test_data(value)
        """)
        
        # Insert test data
        for i in range(10000):
            cur.execute(
                "INSERT INTO test_data (name, value) VALUES (%s, %s)",
                (f"test_{i}", random.randint(1, 1000))
            )
        
        conn.commit()
        cur.close()
        conn.close()
        print("✅ PostgreSQL test data created")
    
    def generate_normal_load(self):
        """Generate normal database load"""
        while True:
            conn = psycopg2.connect(**self.conn_params)
            cur = conn.cursor()
            
            # Random queries
            query_type = random.choice(['select', 'insert', 'update'])
            
            if query_type == 'select':
                cur.execute(
                    "SELECT * FROM test_data WHERE value > %s LIMIT 10",
                    (random.randint(1, 1000),)
                )
                cur.fetchall()
            
            elif query_type == 'insert':
                cur.execute(
                    "INSERT INTO test_data (name, value) VALUES (%s, %s)",
                    (f"load_test_{random.randint(1, 10000)}", random.randint(1, 1000))
                )
                conn.commit()
            
            elif query_type == 'update':
                cur.execute(
                    "UPDATE test_data SET value = %s WHERE id = %s",
                    (random.randint(1, 1000), random.randint(1, 10000))
                )
                conn.commit()
            
            cur.close()
            conn.close()
            
            time.sleep(random.uniform(0.1, 0.5))
    
    def generate_slow_queries(self):
        """Generate intentionally slow queries"""
        while True:
            conn = psycopg2.connect(**self.conn_params)
            cur = conn.cursor()
            
            # Slow query without index
            cur.execute("""
                SELECT * FROM test_data 
                WHERE name LIKE '%test%' 
                ORDER BY created_at DESC
            """)
            cur.fetchall()
            
            cur.close()
            conn.close()
            
            time.sleep(random.uniform(5, 10))

class RedisLoadGenerator:
    def __init__(self, host, port=6379):
        self.redis_client = redis.Redis(host=host, port=port, decode_responses=True)
    
    def generate_load(self):
        """Generate Redis load"""
        while True:
            operation = random.choice(['set', 'get', 'delete'])
            key = f"test_key_{random.randint(1, 1000)}"
  if operation == 'set':
            self.redis_client.set(key, f"value_{random.randint(1, 10000)}", ex=300)
        
        elif operation == 'get':
            self.redis_client.get(key)
        
        elif operation == 'delete':
            self.redis_client.delete(key)
        
        time.sleep(random.uniform(0.01, 0.1))
# Usage

if **name** == "**main**": # PostgreSQL pg_gen = PostgreSQLLoadGenerator( host='localhost', database='testdb', user='postgres', password='password' )

# Create test data
pg_gen.create_test_data()

# Start load generators
Thread(target=pg_gen.generate_normal_load, daemon=True).start()
Thread(target=pg_gen.generate_slow_queries, daemon=True).start()

# Redis
redis_gen = RedisLoadGenerator(host='localhost')
Thread(target=redis_gen.generate_load, daemon=True).start()

print("🔥 Load generation started. Press Ctrl+C to stop...")

try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    print("\n⏹️  Stopped")
```


5. **Deploy и тестирование**:
```bash
# Deploy databases
kubectl apply -f k8s-manifests/postgres-with-monitoring.yaml
kubectl apply -f k8s-manifests/redis-with-monitoring.yaml
kubectl apply -f k8s-manifests/database-alerts.yaml

# Check pods
kubectl get pods -n database

# Port forward для доступа
kubectl port-forward -n database svc/postgres 5432:5432 &
kubectl port-forward -n database svc/redis 6379:6379 &

# Run load generator
python test/database-load-generator.py

# Check metrics
curl http://localhost:9187/metrics | grep pg_
curl http://localhost:9121/metrics | grep redis_

# Check Grafana dashboards
open http://localhost:3000
```

### 🚀 Бонус (новое)

**1. Slow query analyzer**:

`scripts/analyze-slow-queries.py`:
```python
#!/usr/bin/env python3
"""
Analyze slow queries from PostgreSQL logs
"""
import psycopg2
from collections import Counter
import re

def analyze_slow_queries(conn_params, threshold_ms=100):
    """Analyze and report slow queries"""
    conn = psycopg2.connect(**conn_params)
    cur = conn.cursor()
    
    # Get slow queries from pg_stat_statements
    # (requires pg_stat_statements extension)
    cur.execute("""
        SELECT 
          query,
          calls,
          total_exec_time / calls as avg_time_ms,
          total_exec_time,
          rows
        FROM pg_stat_statements
        WHERE total_exec_time / calls > %s
        ORDER BY avg_time_ms DESC
        LIMIT 20
    """, (threshold_ms,))
    
    print(f"\n{'='*80}")
    print(f"SLOW QUERIES REPORT (> {threshold_ms}ms)")
    print(f"{'='*80}\n")
    
    for row in cur.fetchall():
        query, calls, avg_time, total_time, rows = row
        
        # Clean query
        clean_query = re.sub(r'\s+', ' ', query).strip()[:100]
        
        print(f"Query: {clean_query}...")
        print(f"  Calls: {calls:,}")
        print(f"  Avg time: {avg_time:.2f}ms")
        print(f"  Total time: {total_time/1000:.2f}s")
        print(f"  Rows: {rows:,}")
        print()
    
    cur.close()
    conn.close()

if __name__ == "__main__":
    analyze_slow_queries({
        'host': 'localhost',
        'database': 'testdb',
        'user': 'postgres',
        'password': 'password'
    })
```

**2. Index recommendation tool**:
```python
#!/usr/bin/env python3
"""
Recommend indexes based on query patterns
"""
import psycopg2

def recommend_indexes(conn_params):
    """Analyze and recommend missing indexes"""
    conn = psycopg2.connect(**conn_params)
    cur = conn.cursor()
    
    print("\n🔍 INDEX RECOMMENDATIONS\n")
    
    # Find tables with sequential scans
    cur.execute("""
        SELECT 
          schemaname,
          tablename,
          seq_scan,
          seq_tup_read,
          idx_scan,
          seq_tup_read / NULLIF(seq_scan, 0) as avg_seq_read
        FROM pg_stat_user_tables
        WHERE seq_scan > 0
        ORDER BY seq_tup_read DESC
        LIMIT 10
    """)
    
    print("Tables with high sequential scans (might need indexes):")
    for row in cur.fetchall():
        schema, table, seq_scan, seq_tup_read, idx_scan, avg = row
        if idx_scan is None or seq_scan > idx_scan * 10:
            print(f"\n  ⚠️  {schema}.{table}")
            print(f"     Sequential scans: {seq_scan:,}")
            print(f"     Tuples read: {seq_tup_read:,}")
            print(f"     Index scans: {idx_scan or 0:,}")
            print(f"     💡 Consider adding index on frequently queried columns")
    
    # Find unused indexes
    cur.execute("""
        SELECT 
          schemaname,
          tablename,
          indexname,
          pg_size_pretty(pg_relation_size(indexrelid)) AS size
        FROM pg_stat_user_indexes
        WHERE idx_scan = 0
          AND indexrelid NOT IN (
            SELECT indexrelid FROM pg_index WHERE indisprimary
          )
        ORDER BY pg_relation_size(indexrelid) DESC
        LIMIT 10
    """)
    
    print("\n\nUnused indexes (candidates for removal):")
    for row in cur.fetchall():
        schema, table, index, size = row
        print(f"\n  ❌ {index} on {schema}.{table}")
        print(f"     Size: {size}")
        print(f"     💡 DROP INDEX IF EXISTS {schema}.{index};")
    
    cur.close()
    conn.close()

if __name__ == "__main__":
    recommend_indexes({
        'host': 'localhost',
        'database': 'testdb',
        'user': 'postgres',
        'password': 'password'
    })
```

**3. Backup monitoring**:

`k8s-manifests/backup-monitoring.yaml`:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: postgres-backup
  namespace: database
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:16-alpine
            command:
            - /bin/sh
            - -c
            - |
              set -e
              
              BACKUP_FILE="/backups/backup-$(date +%Y%m%d-%H%M%S).sql.gz"
              
              echo "Starting backup..."
              pg_dump -h postgres -U postgres testdb | gzip > $BACKUP_FILE
              
              # Push metric to Pushgateway
              cat <<EOF | curl --data-binary @- http://pushgateway:9091/metrics/job/postgres_backup
              # TYPE postgres_backup_timestamp gauge
              postgres_backup_timestamp $(date +%s)
              # TYPE postgres_backup_size_bytes gauge
              postgres_backup_size_bytes $(stat -f%z $BACKUP_FILE)
              # TYPE postgres_backup_success gauge
              postgres_backup_success 1
              EOF
              
              echo "Backup completed: $BACKUP_FILE"
              
              # Cleanup old backups (keep last 7 days)
              find /backups -name "backup-*.sql.gz" -mtime +7 -delete
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
            volumeMounts:
            - name: backups
              mountPath: /backups
          volumes:
          - name: backups
            persistentVolumeClaim:
              claimName: postgres-backups
          restartPolicy: OnFailure

---
# Alert на failed backup
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: backup-alerts
  namespace: monitoring
spec:
  groups:
  - name: backup.rules
    rules:
    - alert: PostgreSQLBackupFailed
      expr: |
        time() - postgres_backup_timestamp > 86400 * 2  # No backup for 2 days
      for: 1h
      labels:
        severity: critical
      annotations:
        summary: "PostgreSQL backup failed or not running"
        description: "Last successful backup was {{ $value | humanizeDuration }} ago"
    
    - alert: PostgreSQLBackupSizeAnomaly
      expr: |
        postgres_backup_size_bytes < 
        avg_over_time(postgres_backup_size_bytes[7d]) * 0.5
      for: 1h
      labels:
        severity: warning
      annotations:
        summary: "PostgreSQL backup size anomaly"
        description: "Backup size is significantly smaller than usual"
```

---

## Итоги модуля 11

После прохождения этого модуля ты должен уметь:

✅ Мониторить PostgreSQL (connections, queries, cache, replication)
✅ Мониторить MySQL/MariaDB
✅ Мониторить Redis (memory, hit rate, clients)
✅ Мониторить MongoDB
✅ Настраивать exporters для баз данных
✅ Создавать alerts для database метрик
✅ Анализировать slow queries
✅ Рекомендовать indexes
✅ Мониторить disk I/O и storage
✅ Отслеживать backup успешность
✅ Оптимизировать производительность БД
✅ Детектировать проблемы до их возникновения

**Key Database Metrics:**

Performance: Query time, throughput, cache hit ratio Resources: CPU, Memory, Disk I/O, connections Availability: Uptime, replication lag, deadlocks Capacity: Disk space, table sizes, connection pool


**Production Checklist:**
- ✅ Exporters для всех баз данных
- ✅ Alerts на критичные метрики
- ✅ Slow query monitoring
- ✅ Connection pool monitoring
- ✅ Replication lag alerts
- ✅ Backup monitoring
- ✅ Disk space alerts
- ✅ Index optimization
- ✅ Query performance tracking
- ✅ Capacity planning
- ✅ Regular vacuum/analyze (PostgreSQL)
- ✅ Automated backups with verification

## Модуль 12: Заключение - Production-Ready Monitoring и Best Practices (30 минут)

### 🎯 Итоговый обзор

**Что мы изучили в курсе:**

```
Модуль 1:  Основы мониторинга (Metrics, Golden Signals, USE/RED)
Модуль 2:  Prometheus (TSDB, PromQL, scraping)
Модуль 3:  Grafana (Dashboards, визуализация)
Модуль 4:  Логирование (Loki, ELK, централизация)
Модуль 5:  Alerting (Alertmanager, notification)
Модуль 6:  Distributed Tracing (Jaeger, OpenTelemetry, APM)
Модуль 7:  Kubernetes Monitoring (kube-state-metrics, pods, nodes)
Модуль 8:  SRE практики (SLO/SLI/SLA, Error Budget)
Модуль 9:  Security Monitoring (Falco, audit logs, compliance)
Модуль 10: Observability as Code (GitOps, automation)
Модуль 11: Database Monitoring (PostgreSQL, Redis, slow queries)
```

**The Observability Pyramid:**

```
                    ▲
                   ╱ ╲
                  ╱   ╲
                 ╱ SRE ╲
                ╱Practices╲
               ╱───────────╲
              ╱             ╲
             ╱   Security   ╲
            ╱   Monitoring   ╲
           ╱─────────────────╲
          ╱                   ╲
         ╱  Distributed Trace  ╲
        ╱                       ╲
       ╱─────────────────────────╲
      ╱                           ╲
     ╱      Logs & Metrics         ╲
    ╱                               ╲
   ╱─────────────────────────────────╲
  ╱                                   ╲
 ╱        Infrastructure              ╲
╱                                       ╲
─────────────────────────────────────────
```

---

## Production-Ready Monitoring Checklist

### 📊 1. Metrics & Monitoring

**Infrastructure Level:**

```
✅ Node Exporter на всех серверах
✅ cAdvisor для Docker/Kubernetes
✅ kube-state-metrics для K8s objects
✅ Prometheus с HA setup (минимум 2 replicas)
✅ Long-term storage (Thanos/Cortex/VictoriaMetrics)
✅ Retention policy определена (15-30 дней local, 1+ год remote)
```

**Application Level:**

```
✅ /metrics endpoint на всех сервисах
✅ Custom business metrics экспортируются
✅ ServiceMonitor/PodMonitor для auto-discovery
✅ Structured logging (JSON формат)
✅ Correlation IDs во всех логах
✅ Distributed tracing настроен
```

**Database Level:**

```
✅ PostgreSQL/MySQL/Redis exporters
✅ Slow query monitoring
✅ Connection pool monitoring
✅ Replication lag alerts
✅ Backup monitoring
```

### 🚨 2. Alerting

**Alert Quality:**

```
✅ Каждый alert имеет:
   - Понятное имя
   - Severity (critical/warning/info)
   - Summary и description
   - Runbook URL
   - Dashboard link
   
✅ Alerts следуют принципу:
   - Actionable (требует действий)
   - Relevant (важен для бизнеса)
   - Timely (срабатывает вовремя)

✅ Alert fatigue prevention:
   - Grouping настроен
   - Inhibition rules работают
   - For clause предотвращает flapping
   - Severity levels правильные
```

**Alert Coverage:**

```
✅ Infrastructure alerts:
   - Node down
   - High CPU/Memory/Disk
   - Network issues

✅ Application alerts:
   - High error rate
   - High latency
   - API down
   - Service degradation

✅ SLO alerts:
   - Error budget burn rate
   - SLO violation
   - Latency SLO breach

✅ Security alerts:
   - Authentication failures
   - Unauthorized access
   - Security scan failures

✅ Database alerts:
   - Connection pool exhausted
   - Replication lag
   - Slow queries
   - Backup failures
```

### 📈 3. Dashboards

**Standard Dashboards:**

```
✅ Cluster overview (nodes, pods, resources)
✅ Service dashboards (RED metrics)
✅ Database dashboards
✅ SLO dashboards (error budget tracking)
✅ Security dashboards
✅ Cost dashboards (FinOps)
```

**Dashboard Best Practices:**

```
✅ Используй variables для фильтрации
✅ Добавляй links на runbooks
✅ Группируй панели логически (rows)
✅ Используй consistent naming
✅ Добавляй descriptions к панелям
✅ Настраивай refresh intervals
✅ Используй appropriate visualizations
✅ Храни dashboards в Git
```

### 🔍 4. Logging

**Log Infrastructure:**

```
✅ Centralized logging (Loki/ELK)
✅ Structured logs (JSON)
✅ Log retention policy
✅ Log rotation configured
✅ Log levels правильные
```

**Log Content:**

```
✅ Timestamp (UTC)
✅ Log level
✅ Service name
✅ Correlation ID / Trace ID
✅ Error stack traces
✅ Context информация
✅ No sensitive data (passwords, tokens)
```

### 🔐 5. Security

**Security Monitoring:**

```
✅ Falco или аналог для runtime security
✅ Audit logging для критичных операций
✅ Vulnerability scanning в CI/CD
✅ Secret scanning
✅ WAF monitoring
✅ Authentication/Authorization monitoring
```

**Compliance:**

```
✅ PCI DSS / GDPR / HIPAA requirements
✅ Audit trails
✅ Access logs
✅ Data retention policies
✅ Regular security reports
```

### 🎯 6. SRE Practices

**SLO/SLI:**

```
✅ SLO определены для критичных сервисов
✅ SLI метрики собираются
✅ Error budget tracking
✅ Multi-window multi-burn-rate alerts
✅ SLO review процесс
```

**On-Call:**

```
✅ On-call rotation справедливая
✅ Runbooks для всех alerts
✅ Escalation policy
✅ Post-mortem процесс
✅ Incident response playbooks
```

---

## Best Practices Summary

### 🎨 Design Principles

**1. Start with SLIs/SLOs**

```
❌ Мониторить всё подряд
✅ Начинать с user-facing metrics

Вопросы:
- Что важно для пользователей?
- Какой уровень надёжности нужен?
- Какой error budget допустим?
```

**2. Monitor Symptoms, Not Causes**

```
❌ Alert: "CPU > 80%"
✅ Alert: "API latency > 1s"

Принцип: Алертить на то, что видит пользователь
```

**3. Reduce Alert Fatigue**

```
Правило: Если не требует действий → не alert, а dashboard

Техники:
- Grouping похожих alerts
- Inhibition для зависимых alerts
- For clause для предотвращения flapping
- Правильные thresholds
```

**4. Make Alerts Actionable**

```
Каждый alert должен отвечать на:
- Что случилось?
- Почему это важно?
- Что делать? (runbook)
- Где смотреть? (dashboard)
```

**5. Automate Everything**

```
✅ Dashboard generation
✅ Alert creation from SLO
✅ Configuration validation
✅ Deployment через CI/CD
✅ Cost optimization
```

### 📏 Metric Guidelines

**Naming Conventions:**

```
<namespace>_<name>_<unit>_<type>

Examples:
- http_requests_total (counter)
- http_request_duration_seconds (histogram)
- process_cpu_seconds_total (counter)
- node_memory_MemAvailable_bytes (gauge)
```

**Cardinality Management:**

```
❌ High cardinality labels:
   - user_id
   - request_id
   - timestamp
   - IP address (full)

✅ Low cardinality labels:
   - service
   - environment
   - status_code
   - method
   - region
```

**Label Best Practices:**

```
✅ Use labels for dimensions you'll filter/aggregate by
✅ Keep label values bounded (< 100 unique values)
✅ Don't use labels for high cardinality data
✅ Use consistent label names across metrics
```

### 🎓 Query Optimization

**PromQL Performance:**

```
❌ Slow:
rate(http_requests_total[5m])

✅ Fast (with recording rule):
job:http_requests:rate5m

Принцип: Pre-compute expensive queries
```

**Recording Rules:**

yaml

````yaml
groups:
  - name: performance
    interval: 30s
    rules:
    # Pre-compute commonly used aggregations
    - record: job:http_requests:rate5m
      expr: sum(rate(http_requests_total[5m])) by (job)
    
    - record: job:http_request_duration:p95
      expr: histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, job))
````

### 🔧 Troubleshooting Guide

**Common Problems & Solutions:**

**1. High Cardinality**
```
Problem: Prometheus running out of memory
Причина: Metrics with too many unique label values

Solution:
- Найти high cardinality metrics: 
  topk(10, count by (__name__)({__name__!=""}))
- Remove или aggregate labels
- Use recording rules
- Adjust retention
```

**2. Missing Metrics**
```
Problem: Metrics not appearing in Prometheus
Checks:
1. Is target up? Check /targets
2. Is ServiceMonitor correct? Check labels
3. Is exporter working? curl /metrics
4. Are firewall rules correct?
5. Check Prometheus logs
```

**3. Alert Flapping**
```
Problem: Alert firing and resolving repeatedly
Solution:
- Add "for" clause (e.g., for: 5m)
- Adjust threshold
- Add hysteresis (different thresholds for firing/resolving)
```

**4. Dashboard Loading Slow**
```
Problem: Grafana dashboard takes long to load
Solutions:
- Use recording rules for expensive queries
- Reduce time range
- Reduce refresh frequency
- Use query caching
- Optimize PromQL queries (avoid regex, use recording rules)
```

### 💰 Cost Optimization

**Reduce Storage Costs:**
````
1. Adjust retention:
   - Local: 15-30 days
   - Remote: 1-2 years with downsampling

2. Reduce scrape frequency:
   - Non-critical: 60s instead of 15s
   - Development: 120s

3. Drop unused metrics:
   metric_relabel_configs:
     - source_labels: [__name__]
       regex: 'unused_metric.*'
       action: drop

4. Use recording rules:
   - Pre-aggregate expensive queries
   - Reduce query cost

5. Implement sampling:
   - Not all metrics need 100% samples
````

**Monitor Monitoring Costs:**

promql

````promql
# Prometheus storage size
prometheus_tsdb_storage_blocks_bytes

# Number of time series
prometheus_tsdb_head_series

# Cardinality per metric
count by (__name__) ({__name__!=""})

# Sample ingestion rate
rate(prometheus_tsdb_head_samples_appended_total[5m])

# Estimate cost
# samples_per_day * retention_days * cost_per_sample
````

---

## Production Deployment Architecture

**Recommended Setup:**
```
┌─────────────────────────────────────────────┐
│            Load Balancer                    │
└──────────────┬──────────────────────────────┘
               │
     ┌─────────┴──────────┐
     │                    │
┌────▼─────┐        ┌─────▼────┐
│Prometheus│        │Prometheus│  (HA pair)
│ Primary  │        │Secondary │
└────┬─────┘        └─────┬────┘
     │                    │
     └─────────┬──────────┘
               │
     ┌─────────▼──────────┐
     │                    │
┌────▼─────┐        ┌─────▼────┐
│  Thanos  │        │  Thanos  │  (Long-term storage)
│  Store   │        │  Store   │
└──────────┘        └──────────┘
     │                    │
     └─────────┬──────────┘
               │
         ┌─────▼─────┐
         │  S3/GCS   │  (Object storage)
         │  Bucket   │
         └───────────┘

┌─────────────────────────────────────────────┐
│            Grafana HA                       │
│  ┌──────────┐         ┌──────────┐         │
│  │ Grafana  │         │ Grafana  │         │
│  │ Instance │         │ Instance │         │
│  └──────────┘         └──────────┘         │
│       │                     │               │
│       └──────────┬──────────┘               │
│                  │                          │
│            ┌─────▼─────┐                    │
│            │ PostgreSQL│  (Shared DB)       │
│            └───────────┘                    │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│         Alertmanager Cluster                │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐    │
│  │  Alert   │ │  Alert   │ │  Alert   │    │
│  │ manager  │ │ manager  │ │ manager  │    │
│  │    1     │ │    2     │ │    3     │    │
│  └──────────┘ └──────────┘ └──────────┘    │
└─────────────────────────────────────────────┘
```

**Key Components:**

1. **Prometheus HA:**
   - Минимум 2 replicas
   - Одинаковые scrape configs
   - Load balancer перед ними

2. **Long-term Storage:**
   - Thanos/Cortex/VictoriaMetrics
   - S3/GCS для хранения
   - Downsampling для старых данных

3. **Grafana HA:**
   - Минимум 2 instances
   - Shared PostgreSQL/MySQL
   - Session storage в DB

4. **Alertmanager Cluster:**
   - 3+ instances (odd number)
   - Gossip protocol для sync
   - Deduplication автоматическая

---

## Migration Strategy

**Переход на Production Monitoring:**

**Phase 1: Foundation (Week 1-2)**
```
✅ Deploy Prometheus + Grafana
✅ Setup node-exporter
✅ Basic infrastructure dashboards
✅ Critical alerts only
✅ Document everything
```

**Phase 2: Application Monitoring (Week 3-4)**
```
✅ Add /metrics endpoints
✅ ServiceMonitors для всех сервисов
✅ Application dashboards
✅ SLO definition
✅ Application alerts
```

**Phase 3: Advanced (Week 5-6)**
```
✅ Distributed tracing
✅ Security monitoring
✅ Database monitoring
✅ Cost tracking
✅ Automation (GitOps)
```

**Phase 4: Optimization (Week 7-8)**
```
✅ Alert tuning
✅ Dashboard refinement
✅ Performance optimization
✅ Documentation update
✅ Training team
```

---

## Team Structure & Responsibilities

**Who Does What:**
```
SRE Team:
├─ Maintain monitoring infrastructure
├─ Create platform dashboards
├─ Define SLO framework
├─ Incident response coordination
└─ Post-mortem facilitation

Development Teams:
├─ Instrument applications
├─ Create service dashboards
├─ Define service SLOs
├─ Respond to service alerts
└─ Fix reliability issues

Security Team:
├─ Security monitoring rules
├─ Audit log analysis
├─ Compliance reports
└─ Security incident response

Platform Team:
├─ K8s monitoring
├─ Infrastructure alerts
├─ Capacity planning
└─ Cost optimization
```

---

## Learning Resources

**Books:**
```
📚 "Site Reliability Engineering" - Google
📚 "The Site Reliability Workbook" - Google
📚 "Database Reliability Engineering" - O'Reilly
📚 "Prometheus: Up & Running" - O'Reilly
📚 "Observability Engineering" - O'Reilly
```

**Online:**
```
🌐 Prometheus Documentation - prometheus.io
🌐 Grafana Tutorials - grafana.com/tutorials
🌐 SRE Weekly Newsletter - sreweekly.com
🌐 Cloud Native Computing Foundation - cncf.io
🌐 PromCon Talks - youtube.com/@PrometheusIo
```

**Community:**
```
💬 CNCF Slack - cloud-native.slack.com
💬 Prometheus Users - groups.google.com/g/prometheus-users
💬 Reddit r/devops, r/sre
💬 SRE Conferences: SREcon, KubeCon
```

---

## Career Path

**Monitoring & SRE Career:**
```
Junior DevOps/SRE
├─ Setup basic monitoring
├─ Create dashboards
├─ Respond to alerts
└─ Learn PromQL

    ↓

Mid-Level SRE
├─ Design monitoring architecture
├─ Implement SLO/SLI
├─ Incident response lead
├─ Automation
└─ Mentor juniors

    ↓

Senior SRE
├─ Define SRE strategy
├─ Platform reliability
├─ Capacity planning
├─ Cross-team leadership
└─ Incident commander

    ↓

Staff/Principal SRE
├─ Company-wide observability
├─ SRE practices evangelism
├─ Architecture decisions
└─ Industry thought leadership
```

**Skills to Develop:**
```
Technical:
✅ Prometheus, Grafana, Loki
✅ Kubernetes deep knowledge
✅ PromQL mastery
✅ Distributed systems
✅ Performance analysis
✅ Automation (Python, Go)

Soft Skills:
✅ Incident management
✅ Communication (written & verbal)
✅ Blameless post-mortems
✅ Stakeholder management
✅ Teaching/mentoring
✅ On-call leadership
```

---

## Final Checklist

**Before Going to Production:**
```
Infrastructure:
□ Prometheus HA setup
□ Long-term storage configured
□ Backup strategy defined
□ Disaster recovery tested

Monitoring:
□ All critical services monitored
□ SLOs defined and tracked
□ Dashboards created and shared
□ Alerts tuned and documented

Process:
□ On-call rotation established
□ Runbooks written
□ Post-mortem process defined
□ Training completed

Security:
□ Access control configured
□ Audit logging enabled
□ Compliance requirements met
□ Security scanning automated

Documentation:
□ Architecture documented
□ Runbooks updated
□ Dashboards documented
□ Contact list current
```

---

## 🎓 Поздравляю!

Ты прошёл полный курс по мониторингу для DevOps!

**Что ты теперь умеешь:**
- ✅ Строить production-ready monitoring systems
- ✅ Настраивать Prometheus, Grafana, Loki
- ✅ Создавать эффективные alerts
- ✅ Применять SRE практики
- ✅ Мониторить безопасность
- ✅ Автоматизировать observability
- ✅ Оптимизировать costs
- ✅ Troubleshoot production issues

**Следующие шаги:**
1. Примени знания в реальном проекте
2. Поделись опытом с командой
3. Продолжай учиться (SRE - это journey, не destination)
4. Присоединяйся к community
5. Помогай другим учиться

**Remember:**
```
"Hope is not a strategy, but monitoring is!"
"You can't fix what you can't see!"
"Measure twice, deploy once!"
````

Удачи в твоём SRE/DevOps путешествии!
