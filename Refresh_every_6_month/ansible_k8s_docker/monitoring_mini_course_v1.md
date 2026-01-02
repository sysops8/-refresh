
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
```json
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
Loki           - Хранение логов
Promtail       - Агент сбора
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
```

**Полезные команды для логов:**
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

# Продолжение курса: Модуль 4 - Логирование

```bash
# Традиционные логи
tail -f /var/log/syslog
tail -f /var/log/nginx/access.log
grep "ERROR" /var/log/app.log
grep -i "error" /var/log/* | tail -20

# Анализ логов
awk '{print $1}' access.log | sort | uniq -c | sort -rn  # Top IP
awk '{print $9}' access.log | sort | uniq -c | sort -rn  # HTTP codes
```

**LogQL (Loki Query Language):**
```logql
# Stream selector
{job="varlogs"}
{job="varlogs", level="error"}

# Line filter
{job="varlogs"} |= "error"
{job="varlogs"} |~ "error|fail"
{job="varlogs"} != "debug"

# JSON parsing
{job="app"} | json | level="error"

# Rate
rate({job="varlogs"}[5m])

# Count
count_over_time({job="varlogs"}[5m])
```

### 💻 Задание

Настрой централизованное логирование с Loki:

1. **Добавь Loki stack в docker-compose.yml**:
```yaml
  loki:
    image: grafana/loki:latest
    container_name: loki
    ports:
      - "3100:3100"
    volumes:
      - ./loki-config.yml:/etc/loki/local-config.yaml
      - loki-data:/loki
    command: -config.file=/etc/loki/local-config.yaml
    restart: unless-stopped

  promtail:
    image: grafana/promtail:latest
    container_name: promtail
    volumes:
      - ./promtail-config.yml:/etc/promtail/config.yml
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    command: -config.file=/etc/promtail/config.yml
    restart: unless-stopped
    depends_on:
      - loki

volumes:
  loki-data:
```

2. **Создай loki-config.yml**:
```yaml
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
  chunk_idle_period: 5m
  chunk_retain_period: 30s

schema_config:
  configs:
    - from: 2024-01-01
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /loki/boltdb-shipper-active
    cache_location: /loki/boltdb-shipper-cache
    shared_store: filesystem
  filesystem:
    directory: /loki/chunks

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h

chunk_store_config:
  max_look_back_period: 0s

table_manager:
  retention_deletes_enabled: false
  retention_period: 0s
```

3. **Создай promtail-config.yml**:
```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  # System logs
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*log

  # Docker containers
  - job_name: containers
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s
    relabel_configs:
      - source_labels: ['__meta_docker_container_name']
        regex: '/(.*)'
        target_label: 'container'
      - source_labels: ['__meta_docker_container_log_stream']
        target_label: 'stream'
```

4. **Запусти Loki stack**:
```bash
docker-compose up -d loki promtail

# Проверка
curl http://localhost:3100/ready
docker logs promtail
```

5. **Добавь Loki в Grafana**:
- Configuration → Data Sources → Add data source
- Выбери Loki
- URL: http://loki:3100
- Save & Test

6. **Создай Log Dashboard в Grafana**:
- Create → Dashboard → Add new panel
- Data source: Loki
- Query: `{job="varlogs"}`
- Visualization: Logs

**Попробуй LogQL запросы:**
```logql
# Все логи
{job="varlogs"}

# Только ошибки
{job="varlogs"} |= "error"

# Docker логи конкретного контейнера
{container="prometheus"}

# Rate логов
rate({job="varlogs"}[5m])

# Количество логов
count_over_time({job="varlogs"}[1h])

# Топ IP адресов
topk(10, sum by (ip) (rate({job="nginx"}[5m])))
```

### 🚀 Бонус (новое)

**Создай приложение с structured logging**:

**app-with-logs.py**:
```python
import logging
import json
import time
from datetime import datetime
from flask import Flask, request
import random

# Structured JSON logging
class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_obj = {
            "timestamp": datetime.utcnow().isoformat(),
            "level": record.levelname,
            "message": record.getMessage(),
            "service": "demo-app",
        }
        if hasattr(record, 'request_id'):
            log_obj['request_id'] = record.request_id
        if hasattr(record, 'user_id'):
            log_obj['user_id'] = record.user_id
        if hasattr(record, 'duration_ms'):
            log_obj['duration_ms'] = record.duration_ms
        return json.dumps(log_obj)

# Setup logging
logger = logging.getLogger()
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)

app = Flask(__name__)

@app.route('/')
def home():
    request_id = f"req-{random.randint(1000, 9999)}"
    start = time.time()
    
    logger.info("Request received", extra={
        'request_id': request_id,
        'path': request.path
    })
    
    # Simulate work
    time.sleep(random.uniform(0.1, 0.5))
    
    # Random error
    if random.random() < 0.1:
        logger.error("Internal error occurred", extra={
            'request_id': request_id,
            'error': "Database connection failed"
        })
        return "Error", 500
    
    duration = (time.time() - start) * 1000
    logger.info("Request completed", extra={
        'request_id': request_id,
        'duration_ms': duration
    })
    
    return "OK", 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Dockerfile**:
```dockerfile
FROM python:3.9-slim
WORKDIR /app
RUN pip install flask
COPY app-with-logs.py .
CMD ["python", "app-with-logs.py"]
```

**Добавь в docker-compose.yml**:
```yaml
  demo-app:
    build: ./demo-app
    container_name: demo-app
    ports:
      - "5000:5000"
    restart: unless-stopped
```

**Генерируй трафик**:
```bash
# Install hey (HTTP load generator)
# или используй curl в цикле
while true; do curl http://localhost:5000; sleep 0.5; done
```

**Анализируй логи в Grafana**:
```logql
# Error rate
sum(rate({container="demo-app"} |= "ERROR" [5m]))

# Average duration
avg_over_time({container="demo-app"} | json | unwrap duration_ms [5m])

# Request count by level
sum by (level) (count_over_time({container="demo-app"} [5m]))
```

---

## Модуль 5: Алертинг и On-Call (25 минут)

### 🎯 Напоминалка

**Alertmanager архитектура:**
```
┌────────────┐
│ Prometheus │──┐
└────────────┘  │
                │  ┌──────────────┐    ┌──────────┐
┌────────────┐  ├─►│ Alertmanager │───►│ Receivers│
│ Grafana    │──┘  └──────────────┘    └──────────┘
└────────────┘           │                    │
                         │              Email, Slack
                         │              PagerDuty
                         ▼              Webhook
                    Grouping
                    Throttling
                    Silencing
```

**Alert states:**
```
Inactive  - Условие не выполнено
Pending   - Условие выполнено, но меньше "for:"
Firing    - Условие выполнено дольше "for:"
```

**Alert best practices:**
```
1. Alert на симптомы, не на причины
2. Каждый alert должен быть actionable
3. Включай контекст в annotations
4. Используй severity levels
5. Документируй runbook
6. Тестируй alerts
7. Избегай alert fatigue
```

**Severity levels:**
```
critical  - Нужно немедленное действие (pager)
warning   - Нужно действие в рабочее время
info      - Для информации, действие не требуется
```

**Good vs Bad alerts:**
```
✅ GOOD:
- "API error rate > 5% for 5 minutes"
- "Disk will be full in 4 hours"
- "Payment processing failed"

❌ BAD:
- "CPU usage > 80%"  (не actionable)
- "Log contains ERROR"  (слишком шумно)
- "Service restarted"  (если это норма)
```

**Alertmanager routing:**
```yaml
route:
  group_by: ['alertname', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'default'
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty'
    - match:
        severity: warning
      receiver: 'slack'
```

### 💻 Задание

Настрой полноценный Alertmanager:

1. **Добавь Alertmanager в docker-compose.yml**:
```yaml
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

volumes:
  alertmanager-data:
```

2. **Создай alertmanager.yml**:
```yaml
global:
  resolve_timeout: 5m
  slack_api_url: 'YOUR_SLACK_WEBHOOK_URL'

route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'default'
  routes:
    # Critical alerts
    - match:
        severity: critical
      receiver: 'critical-alerts'
      continue: true
    
    # Warning alerts
    - match:
        severity: warning
      receiver: 'warning-alerts'
      continue: true

receivers:
  - name: 'default'
    webhook_configs:
      - url: 'http://localhost:5001/webhook'
        send_resolved: true

  - name: 'critical-alerts'
    slack_configs:
      - channel: '#alerts-critical'
        title: '🚨 Critical Alert'
        text: "{{ range .Alerts }}{{ .Annotations.summary }}\n{{ .Annotations.description }}\n{{ end }}"
        send_resolved: true

  - name: 'warning-alerts'
    slack_configs:
      - channel: '#alerts-warning'
        title: '⚠️ Warning Alert'
        text: "{{ range .Alerts }}{{ .Annotations.summary }}\n{{ .Annotations.description }}\n{{ end }}"
        send_resolved: true

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

3. **Обнови prometheus.yml**:
```yaml
# Добавь в секцию alerting
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

4. **Создай более продвинутые alerts** (обнови `alerts.yml`):
```yaml
groups:
  - name: slo_alerts
    rules:
    # SLO: 99.9% availability
    - alert: HighErrorRate
      expr: |
        (
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m]))
        ) > 0.001
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High error rate detected"
        description: "Error rate is {{ $value | humanizePercentage }} (threshold: 0.1%)"
        runbook: "https://runbook.example.com/high-error-rate"

    # SLO: p99 latency < 500ms
    - alert: HighLatency
      expr: |
        histogram_quantile(0.99,
          rate(http_request_duration_seconds_bucket[5m])
        ) > 0.5
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: "High latency detected"
        description: "p99 latency is {{ $value }}s (threshold: 0.5s)"

  - name: capacity_alerts
    rules:
    # Predict disk full
    - alert: DiskWillFillIn4Hours
      expr: |
        predict_linear(
          node_filesystem_avail_bytes{mountpoint="/"}[1h], 
          4 * 3600
        ) < 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Disk will be full soon"
        description: "Disk on {{ $labels.instance }} will be full in ~4 hours"
        action: "Clean up logs or expand disk"

    # Memory pressure
    - alert: HighMemoryPressure
      expr: |
        (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) > 0.95
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High memory pressure"
        description: "Memory usage is {{ $value | humanizePercentage }} on {{ $labels.instance }}"

  - name: service_alerts
    rules:
    # Service down
    - alert: ServiceDown
      expr: up == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Service {{ $labels.job }} is down"
        description: "{{ $labels.instance }} has been down for more than 1 minute"
        action: "Check service status and logs"

    # Container restarting
    - alert: ContainerRestarting
      expr: |
        rate(container_last_seen{name!=""}[5m]) > 0
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Container {{ $labels.name }} is restarting"
        description: "Container has restarted {{ $value }} times in the last 5 minutes"
```

5. **Запусти и протестируй**:
```bash
docker-compose up -d alertmanager

# Reload Prometheus config
curl -X POST http://localhost:9090/-/reload

# Проверь Alertmanager UI
http://localhost:9093

# Проверь alerts в Prometheus
http://localhost:9090/alerts
```

6. **Создай webhook receiver для тестирования** (`webhook_receiver.py`):
```python
from flask import Flask, request
import json

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def webhook():
    data = request.json
    print("=" * 50)
    print("Alert received:")
    print(json.dumps(data, indent=2))
    print("=" * 50)
    
    for alert in data.get('alerts', []):
        status = alert['status']
        labels = alert['labels']
        annotations = alert['annotations']
        
        print(f"\nStatus: {status}")
        print(f"Alert: {labels.get('alertname')}")
        print(f"Severity: {labels.get('severity')}")
        print(f"Summary: {annotations.get('summary')}")
        print(f"Description: {annotations.get('description')}")
    
    return "OK", 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

### 🚀 Бонус (новое)

**Настрой Silences и Inhibit rules**:

**Создай silence через API**:
```bash
# Silence alert на 2 часа
curl -XPOST http://localhost:9093/api/v2/silences \
  -H "Content-Type: application/json" \
  -d '{
    "matchers": [
      {
        "name": "alertname",
        "value": "HighCPUUsage",
        "isRegex": false
      }
    ],
    "startsAt": "2025-01-15T10:00:00Z",
    "endsAt": "2025-01-15T12:00:00Z",
    "createdBy": "admin",
    "comment": "Maintenance window"
  }'
```

**Продвинутые inhibit rules**:
```yaml
inhibit_rules:
  # Если сервис down, не алертить про его метрики
  - source_match:
      alertname: 'ServiceDown'
    target_match_re:
      alertname: '.*'
    equal: ['instance']
  
  # Если critical alert firing, подавить warning
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'cluster', 'service']
```

**Настрой PagerDuty integration**:
```yaml
receivers:
  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_SERVICE_KEY'
        description: '{{ .GroupLabels.alertname }}: {{ .CommonAnnotations.summary }}'
        details:
          firing: '{{ .Alerts.Firing | len }}'
          resolved: '{{ .Alerts.Resolved | len }}'
```

---

## Модуль 6: Синтетический мониторинг и Uptime (20 минут)

### 🎯 Напоминалка

**Synthetic monitoring** - проактивная проверка доступности и производительности:
```
Real User Monitoring (RUM)  - Реальные пользователи
Synthetic Monitoring        - Роботы/боты проверяют
```

**Типы проверок:**
```
- HTTP/HTTPS endpoints
- SSL certificate expiration
- DNS resolution
- TCP port availability
- API response validation
- Multi-step transactions
- Browser scenarios (Selenium)
```

**Blackbox Exporter** - для проверки endpoints:
```
Protocols:
- HTTP/HTTPS
- TCP
- ICMP (ping)
- DNS
- GRPC
```

### 💻 Задание

Настрой синтетический мониторинг:

1. **Добавь Blackbox Exporter в docker-compose.yml**:
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
```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_status_codes: [200]
      method: GET
      follow_redirects: true
      preferred_ip_protocol: "ip4"

  http_post_2xx:
    prober: http
    timeout: 5s
    http:
      method: POST
      headers:
        Content-Type: application/json
      body: '{"test": "data"}'

  tcp_connect:
    prober: tcp
    timeout: 5s

  icmp:
    prober: icmp
    timeout: 5s
    icmp:
      preferred_ip_protocol: "ip4"

  dns_example:
    prober: dns
    timeout: 5s
    dns:
      query_name: "example.com"
      query_type: "A"

  ssl_expire:
    prober: http
    timeout: 5s
    http:
      method: GET
      tls_config:
        insecure_skip_verify: false
```

3. **Добавь в prometheus.yml**:
```yaml
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - https://google.com
        - https://github.com
        - http://localhost:3000  # Grafana
        - http://localhost:9090  # Prometheus
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115

  - job_name: 'blackbox-ssl'
    metrics_path: /probe
    params:
      module: [ssl_expire]
    static_configs:
      - targets:
        - https://google.com
        - https://github.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```

4. **Создай uptime alerts** (добавь в `alerts.yml`):
```yaml
groups:
  - name: uptime_alerts
    rules:
    # Endpoint down
    - alert: EndpointDown
      expr: probe_success == 0
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "Endpoint {{ $labels.instance }} is down"
        description: "Endpoint has been down for more than 2 minutes"

    # SSL certificate expires soon
    - alert: SSLCertExpiringSoon
      expr: (probe_ssl_earliest_cert_expiry - time()) / 86400 < 30
      for: 1h
      labels:
        severity: warning
      annotations:
        summary: "SSL certificate expiring soon"
        description: "SSL certificate for {{ $labels.instance }} expires in {{ $value }} days"

    # Slow response
    - alert: SlowResponse
      expr: probe_duration_seconds > 5
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Slow response from {{ $labels.instance }}"
        description: "Response time is {{ $value }}s (threshold: 5s)"

    # HTTP status code error
    - alert: HTTPStatusError
      expr: probe_http_status_code >= 400
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "HTTP error on {{ $labels.instance }}"
        description: "HTTP status code is {{ $value }}"
```

5. **Запусти и проверь**:
```bash
docker-compose up -d blackbox-exporter

# Проверь metrics
curl "http://localhost:9115/probe?target=https://google.com&module=http_2xx"

# Reload Prometheus
curl -X POST http://localhost:9090/-/reload
```

6. **Создай Uptime Dashboard в Grafana**:

**Panel 1: Service Availability**
```promql
avg_over_time(probe_success[24h]) * 100
```
Visualization: Stat
Unit: Percent (0-100)

**Panel 2: Response Time**
```promql
probe_duration_seconds
```
Visualization: Time series

**Panel 3: SSL Certificate Days Left**
```promql
(probe_ssl_earliest_cert_expiry - time()) / 86400
```
Visualization: Gauge
Thresholds: Green >30, Yellow 15-30, Red <15

**Panel 4: Uptime Table**
```promql
probe_success
```
Visualization: Table
Transform: Add field from calculation → "Uptime" → `$probe_success * 100`

### 🚀 Бонус (новое)

**Создай multi-step transaction check** с использованием Selenium:

**synthetic_test.py**:
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from prometheus_client import Gauge, Counter, start_http_server
import time

# Metrics
transaction_duration = Gauge('transaction_duration_seconds', 'Transaction duration')
transaction_success = Counter('transaction_success_total', 'Successful transactions')
transaction_failure = Counter('transaction_failure_total', 'Failed transactions')

def run_transaction():
    """Simulate user transaction"""
    start = time.time()
    
    try:
        options = Options()
        options.add_argument('--headless')
        driver = webdriver.Chrome(options=options)
        
        # Step 1: Navigate
        driver.get('https://example.com')
        time.sleep(1)
        
        # Step 2: Find element
        element = driver.find_element(By.TAG_NAME, 'h1')
        assert 'Example' in element.text
        
        # Step 3: Click (if needed)
        # link = driver.find_element(By.LINK_TEXT, 'More information...')
        # link.click()
        
        driver.quit()
        
        duration = time.time() - start
        transaction_duration.set(duration)
        transaction_success.inc()
        print(f"✅ Transaction successful in {duration:.2f}s")
        
    except Exception as e:
        transaction_failure.inc()
        print(f"❌ Transaction failed: {e}")

if __name__ == '__main__':
    start_http_server(8001)
    print("Synthetic monitoring started on port 8001")
    
    while True:
        run_transaction()
        time.sleep(60)  # Run every minute
```

**Создай API endpoint checker**:
```python
# api_checker.py
import requests
import time
from prometheus_client import Histogram, Counter, Gauge, start_http_server

api_request_duration = Histogram('api_request_duration_seconds', 
                                 'API request duration',
                                 ['endpoint', 'method'])
api_request_total = Counter('api_request_total',
                           'Total API requests',
                           ['endpoint', 'method', 'status'])
api_request_errors = Counter('api_request_errors_total',
                            'Total API errors',
                            ['endpoint', 'error_type'])

endpoints = [
    {'url': 'https://api.github.com', 'method': 'GET'},
    {'url': 'https://jsonplaceholder.typicode.com/posts/1', 'method': 'GET'},
]

def check_endpoint(endpoint):
    url = endpoint['url']
    method = endpoint['method']
    
    try:
        start = time.time()
        
        if method == 'GET':
            response = requests.get(url, timeout=10)
        elif method == 'POST':
            response = requests.post(url, json={}, timeout=10)
        
        duration = time.time() - start
        
        api_request_duration.labels(endpoint=url, method=method).observe(duration)
        api_request_total.labels(endpoint=url, method=method, status=response.status_code).inc()
        
        print(f"✅ {method} {url}: {response.status_code} ({duration:.2f}s)")
        
    except requests.Timeout:
        api_request_errors.labels(endpoint=url, error_type='timeout').inc()
        print(f"⏱️ {method} {url}: Timeout")
    except requests.ConnectionError:
        api_request_errors.labels(endpoint=url, error_type='connection').inc()
        print(f"🔌 {method} {url}: Connection error")
    except Exception as e:
        api_request_errors.labels(endpoint=url, error_type='other').inc()
        print(f"❌ {method} {url}: {e}")

if __name__ == '__main__':
    start_http_server(8002)
    print("API checker started on port 8002")
    
    while True:
        for endpoint in endpoints:
            check_endpoint(endpoint)
        time.sleep(30)
```

---

## Модуль 7: Application Performance Monitoring (APM) (20 минут)

### 🎯 Напоминалка

**APM компоненты:**
```
- Distributed Tracing  - Трассировка запросов между сервисами
- Profiling           - Анализ производительности кода
- Error Tracking      - Отслеживание ошибок
- Transaction Monitoring - Мониторинг транзакций
- Real User Monitoring  - Мониторинг реальных пользователей
```

**Distributed Tracing концепции:**
```
Trace    - Полный путь запроса через систему
Span     - Отдельная операция внутри trace
Context  - Передача trace ID между сервисами

┌─────────────────────────────────────────┐
│ Trace ID: abc123                        │
├─────────────────────────────────────────┤
│ Span 1: API Gateway [200ms]             │
│   └─ Span 2: Auth Service [50ms]        │
│   └─ Span 3: Business Logic [100ms]     │
│       └─ Span 4: Database Query [80ms]  │
│   └─ Span 5: Cache Lookup [20ms]        │
└─────────────────────────────────────────┘
```

**OpenTelemetry (OTEL):**
```
Стандарт для инструментирования приложений
- Traces
- Metrics
- Logs

Компоненты:
- SDK        - Библиотеки для языков
- Collector  - Агрегация и экспорт
- Exporters  - Отправка в backends (Jaeger, Tempo)
```

**Jaeger архитектура:**
```
┌──────────┐
│   App    │──► Jaeger Agent ──► Jaeger Collector ──► Storage
└──────────┘                                             (Cassandra/
                                                          Elasticsearch/
                                                          Badger)
                                                             │
                                                             ▼
                                                        Jaeger Query
                                                             │
                                                             ▼
                                                         Jaeger UI
```

**Ключевые метрики APM:**
```
- Request rate (req/s)
- Error rate (%)
- Duration (p50, p95, p99)
- Throughput
- Apdex score (Application Performance Index)
- Service dependencies
- Slow queries
- N+1 queries
```

**Apdex Score:**
```
Apdex = (Satisfied + Tolerating/2) / Total Requests

Satisfied:   Response time ≤ T
Tolerating:  Response time > T and ≤ 4T
Frustrated:  Response time > 4T

Score: 0.0 (worst) to 1.0 (best)
```

### 💻 Задание

Настрой distributed tracing с Jaeger:

1. **Добавь Jaeger в docker-compose.yml**:
```yaml
  jaeger:
    image: jaegertracing/all-in-one:latest
    container_name: jaeger
    ports:
      - "5775:5775/udp"    # Zipkin compatible
      - "6831:6831/udp"    # Jaeger compact thrift
      - "6832:6832/udp"    # Jaeger binary thrift
      - "5778:5778"        # Configs
      - "16686:16686"      # Jaeger UI
      - "14268:14268"      # Jaeger collector
      - "14250:14250"      # Jaeger gRPC
      - "9411:9411"        # Zipkin compatible
    environment:
      - COLLECTOR_ZIPKIN_HOST_PORT=:9411
    restart: unless-stopped
```

2. **Создай микросервисное приложение с трассировкой**:

**frontend.py** (сервис 1):
```python
from flask import Flask, jsonify
import requests
import time
import random
from jaeger_client import Config
from flask_opentracing import FlaskTracing
from opentracing_instrumentation.client_hooks import install_all_patches

app = Flask(__name__)

# Jaeger configuration
config = Config(
    config={
        'sampler': {'type': 'const', 'param': 1},
        'logging': True,
        'local_agent': {
            'reporting_host': 'jaeger',
            'reporting_port': 6831,
        },
    },
    service_name='frontend',
    validate=True,
)
jaeger_tracer = config.initialize_tracer()
tracing = FlaskTracing(jaeger_tracer, True, app)

# Instrument requests library
install_all_patches()

@app.route('/')
def index():
    return jsonify({"service": "frontend", "status": "ok"})

@app.route('/api/order', methods=['POST'])
def create_order():
    with jaeger_tracer.start_active_span('create_order') as scope:
        scope.span.set_tag('http.method', 'POST')
        scope.span.log_kv({'event': 'order_created'})
        
        # Call auth service
        with jaeger_tracer.start_active_span('call_auth', child_of=scope.span):
            auth_response = requests.get('http://auth-service:5001/verify')
            time.sleep(random.uniform(0.01, 0.05))
        
        if auth_response.status_code != 200:
            scope.span.set_tag('error', True)
            return jsonify({"error": "Auth failed"}), 401
        
        # Call business service
        with jaeger_tracer.start_active_span('call_business', child_of=scope.span):
            business_response = requests.post('http://business-service:5002/process')
            time.sleep(random.uniform(0.05, 0.15))
        
        return jsonify({
            "order_id": "ORD-" + str(random.randint(1000, 9999)),
            "status": "created"
        })

@app.route('/health')
def health():
    return jsonify({"status": "healthy"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**auth_service.py** (сервис 2):
```python
from flask import Flask, jsonify
import time
import random
from jaeger_client import Config
from flask_opentracing import FlaskTracing

app = Flask(__name__)

config = Config(
    config={
        'sampler': {'type': 'const', 'param': 1},
        'logging': True,
        'local_agent': {
            'reporting_host': 'jaeger',
            'reporting_port': 6831,
        },
    },
    service_name='auth-service',
    validate=True,
)
jaeger_tracer = config.initialize_tracer()
tracing = FlaskTracing(jaeger_tracer, True, app)

@app.route('/verify')
def verify():
    with jaeger_tracer.start_active_span('verify_token') as scope:
        scope.span.set_tag('auth.method', 'token')
        
        # Simulate auth check
        time.sleep(random.uniform(0.02, 0.08))
        
        # Random auth failure
        if random.random() < 0.05:
            scope.span.set_tag('error', True)
            scope.span.log_kv({'event': 'auth_failed'})
            return jsonify({"error": "Unauthorized"}), 401
        
        scope.span.log_kv({'event': 'auth_success'})
        return jsonify({"status": "verified"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

**business_service.py** (сервис 3):
```python
from flask import Flask, jsonify
import time
import random
from jaeger_client import Config
from flask_opentracing import FlaskTracing

app = Flask(__name__)

config = Config(
    config={
        'sampler': {'type': 'const', 'param': 1},
        'logging': True,
        'local_agent': {
            'reporting_host': 'jaeger',
            'reporting_port': 6831,
        },
    },
    service_name='business-service',
    validate=True,
)
jaeger_tracer = config.initialize_tracer()
tracing = FlaskTracing(jaeger_tracer, True, app)

@app.route('/process', methods=['POST'])
def process():
    with jaeger_tracer.start_active_span('process_order') as scope:
        # Simulate database query
        with jaeger_tracer.start_active_span('db_query', child_of=scope.span):
            scope.span.set_tag('db.type', 'postgresql')
            scope.span.set_tag('db.statement', 'INSERT INTO orders...')
            time.sleep(random.uniform(0.05, 0.15))
        
        # Simulate cache lookup
        with jaeger_tracer.start_active_span('cache_lookup', child_of=scope.span):
            scope.span.set_tag('cache.hit', random.choice([True, False]))
            time.sleep(random.uniform(0.01, 0.03))
        
        return jsonify({"status": "processed"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5002)
```

3. **Создай requirements.txt**:
```txt
flask==2.3.0
requests==2.31.0
jaeger-client==4.8.0
flask-opentracing==1.1.0
opentracing-instrumentation==3.3.1
```

4. **Создай Dockerfile для сервисов**:
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY *.py .
CMD ["python"]
```

5. **Обнови docker-compose.yml** для сервисов:
```yaml
  frontend:
    build: ./services
    container_name: frontend
    command: python frontend.py
    ports:
      - "5000:5000"
    depends_on:
      - jaeger
      - auth-service
      - business-service
    restart: unless-stopped

  auth-service:
    build: ./services
    container_name: auth-service
    command: python auth_service.py
    depends_on:
      - jaeger
    restart: unless-stopped

  business-service:
    build: ./services
    container_name: business-service
    command: python business_service.py
    depends_on:
      - jaeger
    restart: unless-stopped
```

6. **Запусти и протестируй**:
```bash
# Создай директорию для сервисов
mkdir -p services
# Скопируй файлы в services/

# Запусти
docker-compose up -d jaeger frontend auth-service business-service

# Генерируй трафик
for i in {1..50}; do
  curl -X POST http://localhost:5000/api/order
  sleep 0.5
done

# Открой Jaeger UI
http://localhost:16686
```

7. **Изучи Jaeger UI**:
- Service → frontend
- Operation → POST /api/order
- Find Traces
- Изучи waterfall view
- Посмотри на зависимости сервисов

### 🚀 Бонус (новое)

**Добавь Grafana Tempo для long-term storage**:

```yaml
  tempo:
    image: grafana/tempo:latest
    container_name: tempo
    command: [ "-config.file=/etc/tempo.yaml" ]
    volumes:
      - ./tempo.yaml:/etc/tempo.yaml
      - tempo-data:/tmp/tempo
    ports:
      - "3200:3200"   # tempo
      - "4317:4317"   # otlp grpc
      - "4318:4318"   # otlp http
    restart: unless-stopped

volumes:
  tempo-data:
```

**tempo.yaml**:
```yaml
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
        http:

storage:
  trace:
    backend: local
    local:
      path: /tmp/tempo/blocks

query_frontend:
  search:
    duration_slo: 5s
    throughput_bytes_slo: 1.073741824e+09
  trace_by_id:
    duration_slo: 5s
```

**Создай exemplars связку между metrics и traces**:
```python
from prometheus_client import Histogram, Info
from opentracing.ext import tags

# Histogram with exemplar support
request_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

@app.route('/api/order', methods=['POST'])
def create_order():
    with request_duration.labels(method='POST', endpoint='/api/order').time():
        with jaeger_tracer.start_active_span('create_order') as scope:
            # Add trace ID to exemplar
            trace_id = scope.span.context.trace_id
            # Process request
            return process_order(trace_id)
```

**Настрой Service Map в Grafana**:
1. Add Tempo data source в Grafana
2. URL: http://tempo:3200
3. Explore → Tempo
4. Search → Service Graph

---

## Модуль 8: Мониторинг в продакшене - Best Practices (15 минут)

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
```markdown
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
```

5. **Создай incident response script** (`scripts/incident_response.sh`):
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

**Настрой chaos engineering для тестирования мониторинга**:

**chaos_test.sh**:
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
```bash
chmod +x chaos_test.sh
./chaos_test.sh
```

---

## Модуль 9: Финальный проект и карьера (30 минут)

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
```bash
mkdir ecommerce-monitoring
cd ecommerce-monitoring

# Структура
mkdir -p {services/{frontend,api-gateway,product,order,payment},monitoring/{prometheus,grafana,loki,alertmanager},scripts,docs}
```

**Шаг 2: Создай docker-compose-final.yml**
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
```markdown
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
```
Slack: #incidents
"Investigating high error rate on API service. 
ETA for resolution: 15 minutes. 
Status page: https://status.company.com"
```

### After resolution
```
"Issue resolved. Root cause: [X]. 
Total impact: [Y] minutes. 
Postmortem scheduled for [date]."
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
- [ ] Все сервисы запускаются одной командой
- [ ] Prometheus собирает метрики со всех компонентов
- [ ] 3+ dashboard в Grafana (Business, Technical, Infrastructure)
- [ ] Loki собирает логи в JSON формате
- [ ] Distributed tracing работает через Jaeger/Tempo
- [ ] 10+ production alerts настроены
- [ ] 3+ SLO определены и измеряются
- [ ] Runbooks для критических alerts
- [ ] Load testing показывает стабильность
- [ ] Documentation (README, architecture, SLOs)

**Nice to Have (дополнительно):**
- [ ] Multi-environment (dev/staging/prod)
- [ ] Automated remediation
- [ ] Chaos engineering suite
- [ ] Cost analysis dashboard
- [ ] Security monitoring
- [ ] Capacity planning dashboard
- [ ] Custom exporters
- [ ] Integration tests
- [ ] Performance benchmarks
- [ ] Postmortem examples

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
10. Memory usage forecast (next 2 hours)


---

## 🎓 Сертификат о прохождении

```

╔════════════════════════════════════════════════════════════╗
║                                                            ║
║            🎓 СЕРТИФИКАТ О ПРОХОЖДЕНИИ КУРСА               ║
║                                                            ║
║                 "Мониторинг для DevOps"                    ║
║                                                            ║
║  Настоящий сертификат подтверждает, что                   ║
║                                                            ║
║                    _______________________                 ║
║                                                            ║
║  успешно прошёл(ла) интенсивный курс по DevOps             ║
║  мониторингу и продемонстрировал(а) глубокие               ║
║  практические навыки в следующих областях:                 ║
║                                                            ║
║  ✓ Prometheus & PromQL                                     ║
║  ✓ Grafana Dashboards & Visualization                      ║
║  ✓ Distributed Tracing (Jaeger/Tempo)                      ║
║  ✓ Centralized Logging (Loki)                              ║
║  ✓ Alerting & Incident Response                            ║
║  ✓ SLI/SLO/SLA Management                                  ║
║  ✓ Production Best Practices                               ║
║  ✓ Chaos Engineering                                       ║
║                                                            ║
║  Дата выдачи: _______________                              ║
║                                                            ║
║  Подпись инструктора: _______________                      ║
║                                                            ║
╚════════════════════════════════════════════════════════════╝
```

---

## 📖 Полезные ресурсы для дальнейшего обучения

### Книги 📚

**Обязательно к прочтению:**
1. **"Site Reliability Engineering"** - Google
   - Библия SRE, бесплатно онлайн
   - https://sre.google/books/

2. **"The Site Reliability Workbook"** - Google
   - Практические примеры SRE практик

3. **"Prometheus: Up & Running"** - Brian Brazil
   - Глубокое погружение в Prometheus

4. **"Observability Engineering"** - Charity Majors et al.
   - Современный подход к наблюдаемости

5. **"Database Reliability Engineering"** - Laine Campbell
   - Мониторинг БД в деталях

**Дополнительно:**
- "The Phoenix Project" - Gene Kim (DevOps культура)
- "Systems Performance" - Brendan Gregg (Performance analysis)
- "Designing Data-Intensive Applications" - Martin Kleppmann

### Онлайн курсы 🎓

**Бесплатные:**
- **Prometheus Official Tutorial** - prometheus.io/docs/tutorials
- **Grafana Fundamentals** - grafana.com/tutorials
- **Google SRE Course** - linkedin.com/learning
- **Linux Performance** - brendangregg.com

**Платные:**
- **Coursera**: "Site Reliability Engineering"
- **Udemy**: "Prometheus | The Complete Hands-On for Monitoring"
- **A Cloud Guru**: "Monitoring and Observability"
- **Linux Foundation**: "Prometheus Certified Associate (PCA)"

### Интерактивные лабы 🔬

```
killercoda.com          - Interactive Kubernetes & monitoring labs
katacoda.com            - Free interactive scenarios
play-with-docker.com    - Docker playground
instruqt.com            - Hands-on labs
```

### Сообщества 👥

**Slack:**
- CNCF Slack (#prometheus, #grafana)
- Kubernetes Slack
- SRE Community

**Reddit:**
- r/devops
- r/sre
- r/kubernetes
- r/prometheus

**Telegram (русскоязычные):**
- DevOps Russia
- SRE Russia
- Kubernetes Russia

**Discord:**
- DevOps Chat
- Cloud Native Computing

### YouTube каналы 📺

- **TechWorld with Nana** - DevOps tutorials
- **That DevOps Guy** - Practical DevOps
- **DevOps Toolkit** - Advanced topics
- **KodeKloud** - Structured courses
- **Brendan Gregg** - Performance analysis
- **Google Cloud Tech** - SRE practices

### Блоги и сайты 🌐

**Обязательно читать:**
- https://sre.google - Google SRE
- https://brendangregg.com - Performance
- https://prometheus.io/blog - Prometheus news
- https://grafana.com/blog - Grafana insights
- https://www.honeycomb.io/blog - Observability

**Русскоязычные:**
- habr.com (тег #devops, #monitoring)
- highload.ru
- devops.ru

### Конференции 🎤

**Международные:**
- **KubeCon + CloudNativeCon** - CNCF flagship
- **SREcon** - USENIX SRE conference
- **GrafanaCON** - Observability
- **PromCon** - Prometheus conference
- **Monitorama** - Monitoring practices

**Русскоязычные:**
- **HighLoad++** - Москва
- **DevOops** - Санкт-Петербург
- **Saint TeamLead Conf**

### Инструменты для практики 🛠️

**Monitoring Playground:**
```bash
# Быстрый старт с готовым стеком
git clone https://github.com/vegasbrianc/prometheus
cd prometheus
docker-compose up -d

# Demo приложения для мониторинга
https://github.com/prometheus/demo
https://github.com/grafana/tns
```

**Генераторы нагрузки:**
- **k6** - Modern load testing (k6.io)
- **hey** - HTTP load generator
- **ab** - Apache Benchmark
- **wrk** - HTTP benchmark tool
- **locust** - Python load testing

**Chaos Engineering:**
- **Chaos Mesh** - Kubernetes chaos
- **Gremlin** - Chaos as a Service
- **Pumba** - Docker chaos
- **Chaos Toolkit** - Generic chaos

---

## 🎯 30-дневный план развития после курса

### Неделя 1: Закрепление основ
**День 1-2:** Повтори модули 1-3
- Установи Prometheus + Grafana локально
- Создай 3 custom dashboards

**День 3-4:** Практика PromQL
- Решай задачи на promql.io
- Напиши 50 различных запросов

**День 5-7:** Pet project
- Выбери свой проект для мониторинга
- Настрой полный мониторинг стек

### Неделя 2: Углубление
**День 8-10:** Distributed Tracing
- Добавь tracing в свой проект
- Изучи все возможности Jaeger UI

**День 11-12:** Логирование
- Настрой structured logging
- Практика LogQL запросов

**День 13-14:** Alerting
- Создай 20 production-ready alerts
- Напиши runbooks для каждого

### Неделя 3: Production практики
**День 15-17:** SLO/SLI
- Определи SLO для своего проекта
- Настрой error budget tracking
- Создай SLO dashboard

**День 18-19:** Chaos Engineering
- Проведи 10 chaos experiments
- Проверь, что все alerts срабатывают

**День 20-21:** Performance
- Оптимизируй Prometheus (cardinality)
- Добавь recording rules
- Настрой federation (если нужно)

### Неделя 4: Реальные проекты
**День 22-24:** Open Source вклад
- Найди issue в Prometheus/Grafana
- Или создай полезный dashboard
- Поделись в сообществе

**День 25-27:** Портфолио
- Задокументируй все проекты
- Выложи на GitHub
- Напиши статью на Habr/Medium

**День 28-30:** Подготовка к собеседованиям
- Повтори теорию
- Практикуй ответы на вопросы
- Mock interviews

---

## 🚀 Следующие шаги в карьере

### Сертификации

**Prometheus & Kubernetes:**
- **Prometheus Certified Associate (PCA)** - $300
  - Официальная сертификация CNCF
  - Экзамен: 90 минут, multiple choice
  
- **Certified Kubernetes Administrator (CKA)** - $395
  - Основа для SRE роли
  - Практический экзамен: 2 часа

- **Certified Kubernetes Application Developer (CKAD)** - $395

**Cloud сертификации:**
- **AWS Certified Solutions Architect**
- **Google Cloud Professional Cloud Architect**
- **Azure Administrator**

### Построение портфолио

**Что включить:**

1. **GitHub репозитории**
```
monitoring-stack/           - Ваш monitoring setup
├── prometheus/
├── grafana/
├── alerts/
└── README.md              - Детальное описание

custom-exporters/          - Ваши exporters
slo-dashboards/            - Production dashboards
chaos-tests/               - Chaos experiments
```

2. **Блог/статьи**
- Medium/Habr статьи
- Технические туториалы
- Case studies из практики

3. **Выступления**
- Митапы
- Конференции
- YouTube видео

### Специализации

После базового DevOps можно специализироваться:

**1. Site Reliability Engineering (SRE)**
- Фокус на reliability и scale
- SLO/SLA management
- Incident management
- Capacity planning

**2. Platform Engineering**
- Внутренние инструменты
- Developer experience
- Self-service платформы

**3. Security Engineering (DevSecOps)**
- Security monitoring
- SIEM
- Compliance
- Vulnerability management

**4. Cloud Architect**
- Multi-cloud solutions
- Cost optimization
- Architecture design

**5. Observability Engineer**
- Advanced tracing
- AIOps
- Custom instrumentation

---

## 💡 Pro Tips от опытных SRE

### Культура мониторинга

**DO:**
✅ Начинай с бизнес-метрик
✅ Мониторь user experience
✅ Делай постмортемы без обвинений
✅ Автоматизируй рутину
✅ Документируй всё
✅ Делись знаниями с командой
✅ Тестируй alerts регулярно

**DON'T:**
❌ Не мониторь ради мониторинга
❌ Не создавай alerts без action
❌ Не игнорируй alert fatigue
❌ Не делай dashboards "для галочки"
❌ Не храни логи вечно (cost!)
❌ Не забывай про security

### Производительность

```promql
# Оптимизация PromQL
# ❌ Медленно
sum(rate(http_requests_total[5m])) by (method, status, path)

# ✅ Быстрее (меньше labels)
sum(rate(http_requests_total[5m])) by (method, status)

# ✅ Ещё быстрее (recording rule)
job:http_requests:rate5m
```

### Карьерные советы

1. **Soft skills важны**
   - Communication
   - Incident management
   - Teaching others
   - Documentation

2. **Не зацикливайся на инструментах**
   - Понимай концепции
   - Инструменты меняются
   - Принципы остаются

3. **Участвуй в on-call**
   - Лучшее обучение
   - Понимаешь систему глубже
   - Растёшь профессионально

4. **Пиши код**
   - Automation scripts
   - Custom exporters
   - Internal tools

5. **Оставайся в курсе**
   - CNCF projects
   - New monitoring tools
   - Industry trends

---

## 🎉 Поздравляем с завершением курса!

Вы прошли путь от основ мониторинга до production-ready решений. Теперь у вас есть:

✅ Понимание принципов observability
✅ Практические навыки с Prometheus, Grafana, Loki, Jaeger
✅ Опыт создания dashboards и alerts
✅ Знание SRE практик и SLO management
✅ Готовность к production incidents
✅ Портфолио проектов

### Что дальше?

1. **Практикуйся ежедневно** - мониторь свои проекты
2. **Вноси вклад в Open Source** - помогай сообществу
3. **Учи других** - лучший способ закрепить знания
4. **Оставайся любопытным** - технологии развиваются

### Обратная связь

Мы будем рады услышать:
- Что помогло больше всего?
- Что можно улучшить?
- Какие темы добавить?

**Контакты:**
- GitHub: [ссылка]
- Email: monitoring-course@example.com
- Telegram: @devops_monitoring_course

---

## 📜 Лицензия и использование

Этот курс распространяется под лицензией **MIT**.

Вы можете:
✅ Использовать для личного обучения
✅ Делиться с коллегами
✅ Адаптировать под свои нужды
✅ Использовать в корпоративном обучении

Просим только:
- Сохранять ссылку на оригинал
- Делиться улучшениями
- Не продавать курс

---

## 🙏 Благодарности

Спасибо:
- **Prometheus community** - за отличный инструмент
- **Grafana Labs** - за визуализацию
- **Google SRE team** - за практики и книги
- **CNCF** - за экосистему
- **Всем контрибьюторам Open Source**

---

## 📊 Статистика курса

```
📚 Модулей: 9
⏱️ Общее время: ~3-4 часа
💻 Практических заданий: 27
🎯 Бонусных заданий: 18
📖 Строк кода: 5000+
🎓 Навыков: 50+
```

---

## 🔥 Мотивационные цитаты

> "You can't improve what you don't measure."  
> — Peter Drucker

> "Hope is not a strategy."  
> — Traditional SRE wisdom

> "If you can't monitor it, you can't manage it."  
> — Unknown

> "The best monitoring is the one that alerts you before your users do."  
> — SRE Proverb

> "Measure twice, deploy once."  
> — DevOps wisdom

---

## 🌟 Финальное слово

Мониторинг - это не просто инструменты. Это **культура**, **мышление** и **практики**, которые помогают создавать надёжные системы.

Ваше путешествие только начинается. Впереди:
- Реальные production инциденты
- Масштабирование на миллионы пользователей
- Оптимизация систем
- Построение команды
- Влияние на продукт

**Удачи в вашем DevOps/SRE путешествии! 🚀**

---

**Версия:** 1.0  
**Дата:** Январь 2025  
**Автор:** DevOps Monitoring Course Team  
**Лицензия:** MIT

**Happy Monitoring! 📊🎯🔥**
