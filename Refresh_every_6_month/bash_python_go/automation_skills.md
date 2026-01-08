# Коллекция DevOps-скриптов для тренировки автоматизма

## Как использовать эту коллекцию

### Метод повторения (доказанная эффективность)

1. **Набери скрипт 3-5 раз вручную** (не copy-paste!)
2. **Запускай после каждого набора** — проверяй, что работает
3. **Разбери каждую строку** — понимай, что делает код
4. **Модифицируй постепенно** — добавляй новые фичи
5. **Повторяй, пока не напишешь без подглядывания**

### Порядок изучения

**Неделя 1:** Скрипты 1-3 (Bash основы)  
**Неделя 2:** Скрипты 4-7 (Python основы)  
**Неделя 3:** Скрипты 8-10 (Комплексные задачи)

---

## Скрипт 1: Backup с ротацией

**Задача:** Создать архив директории и удалить старые копии.

bash

```bash
#!/bin/bash

# Backup script with rotation
BACKUP_DIR="/backup"
SOURCE_DIR="/data"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_$DATE.tar.gz"
RETENTION_DAYS=7

# Создаём директорию если не существует
mkdir -p "$BACKUP_DIR"

# Создаём архив
echo "Creating backup: $BACKUP_FILE"
tar -czf "$BACKUP_DIR/$BACKUP_FILE" "$SOURCE_DIR" 2>/dev/null

# Проверяем успешность
if [ $? -eq 0 ]; then
    echo "Backup completed successfully"
else
    echo "Backup failed!"
    exit 1
fi

# Удаляем старые backup'ы
echo "Removing backups older than $RETENTION_DAYS days"
find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "Done!"
```

### Задания для повторения

**Повтор 1:** Набери скрипт, запусти с тестовой директорией  
**Повтор 2:** Добавь проверку существования SOURCE_DIR перед backup  
**Повтор 3:** Добавь логирование в файл `/var/log/backup.log`  
**Повтор 4:** Добавь аргументы командной строки для SOURCE_DIR  
**Повтор 5:** Добавь отправку email при ошибке

---

## Скрипт 2: Мониторинг дискового пространства

**Задача:** Проверить свободное место и предупредить при превышении порога.

bash

```bash
#!/bin/bash

# Disk space monitoring
THRESHOLD=80
EMAIL="admin@example.com"

# Проверяем каждую файловую систему
df -H | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{ print $5 " " $1 }' | while read output;
do
    usage=$(echo $output | awk '{ print $1}' | sed 's/%//g')
    partition=$(echo $output | awk '{ print $2 }')
    
    if [ $usage -ge $THRESHOLD ]; then
        echo "WARNING: Partition $partition is ${usage}% full"
        # Здесь можно добавить отправку email
        # echo "Disk space alert on $partition" | mail -s "Disk Alert" $EMAIL
    else
        echo "OK: Partition $partition is ${usage}% full"
    fi
done
```

### Задания для повторения

**Повтор 1:** Набери и запусти, посмотри на вывод  
**Повтор 2:** Измени THRESHOLD на 50% и проверь срабатывание  
**Повтор 3:** Добавь запись в syslog при превышении порога  
**Повтор 4:** Добавь исключение конкретных разделов из проверки  
**Повтор 5:** Сохраняй историю использования в CSV файл

---

## Скрипт 3: Парсер логов Nginx

**Задача:** Найти все ошибки 5xx за последний час и показать топ URL.

bash

```bash
#!/bin/bash

# Nginx log parser for 5xx errors
LOG_FILE="/var/log/nginx/access.log"
HOUR_AGO=$(date -d '1 hour ago' '+%d/%b/%Y:%H')

echo "=== 5xx Errors in the last hour ==="

# Фильтруем логи за последний час с ошибками 5xx
grep "$HOUR_AGO" "$LOG_FILE" | grep ' 5[0-9][0-9] ' | \
awk '{print $7}' | sort | uniq -c | sort -rn | head -10

echo ""
echo "=== Total 5xx errors ==="
grep "$HOUR_AGO" "$LOG_FILE" | grep -c ' 5[0-9][0-9] '
```

### Задания для повторения

**Повтор 1:** Набери и запусти (можешь создать тестовый лог)  
**Повтор 2:** Модифицируй для поиска 4xx ошибок  
**Повтор 3:** Добавь группировку по IP-адресам  
**Повтор 4:** Сохраняй результат в JSON формат  
**Повтор 5:** Добавь фильтр по конкретному URL pattern

---

## Скрипт 4: Healthcheck с retry (Python)

**Задача:** Проверить доступность списка сервисов с повторными попытками.

python

```python
#!/usr/bin/env python3

import requests
import time
from datetime import datetime

# Список сервисов для проверки
SERVICES = [
    "https://api.example.com/health",
    "https://app.example.com/ping",
    "https://db.example.com/status"
]

MAX_RETRIES = 3
TIMEOUT = 5
RETRY_DELAY = 2

def check_service(url, retries=MAX_RETRIES):
    """Проверка доступности сервиса с retry"""
    for attempt in range(retries):
        try:
            response = requests.get(url, timeout=TIMEOUT)
            if response.status_code == 200:
                return True, response.status_code, None
            else:
                return False, response.status_code, None
        except requests.RequestException as e:
            if attempt < retries - 1:
                print(f"  Retry {attempt + 1}/{retries - 1} for {url}")
                time.sleep(RETRY_DELAY)
            else:
                return False, None, str(e)
    return False, None, "Max retries reached"

def main():
    print(f"=== Health Check started at {datetime.now()} ===\n")
    
    all_healthy = True
    
    for service in SERVICES:
        print(f"Checking: {service}")
        is_healthy, status_code, error = check_service(service)
        
        if is_healthy:
            print(f"  ✓ OK (Status: {status_code})")
        else:
            all_healthy = False
            if status_code:
                print(f"  ✗ FAILED (Status: {status_code})")
            else:
                print(f"  ✗ FAILED (Error: {error})")
        print()
    
    if all_healthy:
        print("All services are healthy!")
        exit(0)
    else:
        print("Some services are down!")
        exit(1)

if __name__ == "__main__":
    main()
```

### Задания для повторения

**Повтор 1:** Набери скрипт, запусти с реальными URL  
**Повтор 2:** Добавь чтение списка URL из файла  
**Повтор 3:** Добавь логирование результатов в файл  
**Повтор 4:** Добавь отправку уведомления в Slack при падении  
**Повтор 5:** Добавь измерение времени ответа каждого сервиса

---

## Скрипт 5: Парсер JSON конфигов (Python)

**Задача:** Прочитать JSON конфиг и извлечь нужные параметры.

python

```python
#!/usr/bin/env python3

import json
import sys
from pathlib import Path

def load_config(config_file):
    """Загрузка JSON конфигурации"""
    try:
        with open(config_file, 'r') as f:
            return json.load(f)
    except FileNotFoundError:
        print(f"Error: Config file '{config_file}' not found")
        sys.exit(1)
    except json.JSONDecodeError as e:
        print(f"Error: Invalid JSON in config file: {e}")
        sys.exit(1)

def validate_config(config):
    """Проверка обязательных параметров"""
    required_fields = ['app_name', 'version', 'database']
    missing = [field for field in required_fields if field not in config]
    
    if missing:
        print(f"Error: Missing required fields: {', '.join(missing)}")
        sys.exit(1)
    
    return True

def print_config_summary(config):
    """Вывод краткой информации о конфиге"""
    print("=== Configuration Summary ===")
    print(f"App Name: {config.get('app_name', 'N/A')}")
    print(f"Version: {config.get('version', 'N/A')}")
    print(f"Environment: {config.get('environment', 'production')}")
    
    if 'database' in config:
        db = config['database']
        print(f"\nDatabase:")
        print(f"  Host: {db.get('host', 'N/A')}")
        print(f"  Port: {db.get('port', 'N/A')}")
        print(f"  Name: {db.get('name', 'N/A')}")
    
    if 'features' in config:
        print(f"\nEnabled Features: {', '.join(config['features'])}")

def main():
    if len(sys.argv) < 2:
        print("Usage: python script.py <config.json>")
        sys.exit(1)
    
    config_file = sys.argv[1]
    config = load_config(config_file)
    validate_config(config)
    print_config_summary(config)

if __name__ == "__main__":
    main()
```

**Пример config.json:**

json

```json
{
  "app_name": "MyApp",
  "version": "1.2.3",
  "environment": "production",
  "database": {
    "host": "localhost",
    "port": 5432,
    "name": "mydb"
  },
  "features": ["auth", "api", "cache"]
}
```

### Задания для повторения

**Повтор 1:** Набери скрипт, создай тестовый JSON, запусти  
**Повтор 2:** Добавь обработку вложенных конфигов (config.database.replicas)  
**Повтор 3:** Добавь экспорт параметров в переменные окружения  
**Повтор 4:** Добавь merge двух конфигов (base + environment)  
**Повтор 5:** Конвертируй JSON в YAML формат

---

## Скрипт 6: Обработка логов с ротацией (Python)

**Задача:** Читать лог-файл, искать паттерны, группировать ошибки.

python

```python
#!/usr/bin/env python3

import re
from collections import Counter
from datetime import datetime

LOG_FILE = "/var/log/app/application.log"

def parse_log_line(line):
    """Парсинг строки лога"""
    # Формат: 2024-01-15 10:30:45 [ERROR] Message here
    pattern = r'(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) \[(\w+)\] (.+)'
    match = re.match(pattern, line)
    
    if match:
        return {
            'timestamp': match.group(1),
            'level': match.group(2),
            'message': match.group(3)
        }
    return None

def analyze_logs(log_file, level='ERROR'):
    """Анализ логов по уровню"""
    errors = []
    
    try:
        with open(log_file, 'r') as f:
            for line in f:
                parsed = parse_log_line(line.strip())
                if parsed and parsed['level'] == level:
                    errors.append(parsed['message'])
    except FileNotFoundError:
        print(f"Log file not found: {log_file}")
        return []
    
    return errors

def main():
    print(f"=== Log Analysis for {LOG_FILE} ===\n")
    
    # Находим все ошибки
    errors = analyze_logs(LOG_FILE, level='ERROR')
    
    if not errors:
        print("No errors found!")
        return
    
    print(f"Total errors: {len(errors)}\n")
    
    # Группируем одинаковые ошибки
    error_counts = Counter(errors)
    
    print("=== Top 10 Most Frequent Errors ===")
    for error, count in error_counts.most_common(10):
        print(f"{count:4d}x  {error[:80]}")

if __name__ == "__main__":
    main()
```

### Задания для повторения

**Повтор 1:** Набери скрипт, создай тестовый лог-файл  
**Повтор 2:** Добавь фильтр по временному диапазону (последний час)  
**Повтор 3:** Добавь экспорт результата в JSON  
**Повтор 4:** Добавь поиск по регулярному выражению  
**Повтор 5:** Добавь real-time мониторинг (tail -f аналог)

---

## Скрипт 7: Проверка процессов и перезапуск (Python)

**Задача:** Проверить, запущен ли процесс, и перезапустить при необходимости.

python

```python
#!/usr/bin/env python3

import subprocess
import sys
import time

PROCESS_NAME = "nginx"
RESTART_COMMAND = "sudo systemctl restart nginx"
CHECK_INTERVAL = 60  # секунд

def is_process_running(process_name):
    """Проверка запущен ли процесс"""
    try:
        # Используем pgrep для поиска процесса
        result = subprocess.run(
            ['pgrep', '-x', process_name],
            capture_output=True,
            text=True
        )
        return result.returncode == 0
    except Exception as e:
        print(f"Error checking process: {e}")
        return False

def restart_process(command):
    """Перезапуск процесса"""
    try:
        print(f"Restarting process with command: {command}")
        result = subprocess.run(
            command.split(),
            capture_output=True,
            text=True
        )
        
        if result.returncode == 0:
            print("Process restarted successfully")
            return True
        else:
            print(f"Failed to restart: {result.stderr}")
            return False
    except Exception as e:
        print(f"Error restarting process: {e}")
        return False

def monitor_process(process_name, restart_cmd, check_interval):
    """Мониторинг процесса"""
    print(f"Starting process monitor for: {process_name}")
    print(f"Check interval: {check_interval} seconds")
    print("Press Ctrl+C to stop\n")
    
    try:
        while True:
            if is_process_running(process_name):
                print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] ✓ {process_name} is running")
            else:
                print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] ✗ {process_name} is NOT running")
                restart_process(restart_cmd)
                
                # Ждём немного и проверяем запустился ли
                time.sleep(5)
                if is_process_running(process_name):
                    print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] ✓ {process_name} started successfully")
                else:
                    print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] ✗ Failed to start {process_name}")
            
            time.sleep(check_interval)
    
    except KeyboardInterrupt:
        print("\nMonitoring stopped")
        sys.exit(0)

def main():
    monitor_process(PROCESS_NAME, RESTART_COMMAND, CHECK_INTERVAL)

if __name__ == "__main__":
    main()
```

### Задания для повторения

**Повтор 1:** Набери и запусти (измени PROCESS_NAME на существующий)  
**Повтор 2:** Добавь логирование в файл  
**Повтор 3:** Добавь мониторинг нескольких процессов  
**Повтор 4:** Добавь уведомление при перезапуске  
**Повтор 5:** Добавь ограничение числа перезапусков (circuit breaker)

---

## Скрипт 8: Деплой-скрипт с откатом (Bash)

**Задача:** Развернуть приложение с возможностью отката к предыдущей версии.

bash

```bash
#!/bin/bash

# Deployment script with rollback capability
APP_NAME="myapp"
DEPLOY_DIR="/opt/$APP_NAME"
RELEASES_DIR="$DEPLOY_DIR/releases"
CURRENT_LINK="$DEPLOY_DIR/current"
BACKUP_DIR="$DEPLOY_DIR/backups"
VERSION=$(date +%Y%m%d_%H%M%S)
KEEP_RELEASES=5

# Цвета для вывода
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log() {
    echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')]${NC} $1"
}

error() {
    echo -e "${RED}[$(date +'%Y-%m-%d %H:%M:%S')] ERROR:${NC} $1"
}

warning() {
    echo -e "${YELLOW}[$(date +'%Y-%m-%d %H:%M:%S')] WARNING:${NC} $1"
}

# Создаём необходимые директории
setup_directories() {
    log "Setting up directories..."
    mkdir -p "$RELEASES_DIR"
    mkdir -p "$BACKUP_DIR"
}

# Деплой новой версии
deploy() {
    local release_dir="$RELEASES_DIR/$VERSION"
    
    log "Deploying version: $VERSION"
    
    # Создаём директорию релиза
    mkdir -p "$release_dir"
    
    # Копируем файлы (здесь должна быть ваша логика копирования)
    log "Copying application files..."
    # cp -r /path/to/source/* "$release_dir/"
    echo "Application files" > "$release_dir/app.txt"
    
    # Переключаем симлинк
    log "Switching to new version..."
    ln -sfn "$release_dir" "$CURRENT_LINK"
    
    # Перезапускаем сервис
    log "Restarting service..."
    # systemctl restart $APP_NAME
    
    log "Deployment completed successfully!"
}

# Откат к предыдущей версии
rollback() {
    local releases=($(ls -t "$RELEASES_DIR"))
    
    if [ ${#releases[@]} -lt 2 ]; then
        error "No previous release to rollback to"
        exit 1
    fi
    
    local previous_release="${releases[1]}"
    warning "Rolling back to: $previous_release"
    
    ln -sfn "$RELEASES_DIR/$previous_release" "$CURRENT_LINK"
    
    # Перезапускаем сервис
    # systemctl restart $APP_NAME
    
    log "Rollback completed!"
}

# Очистка старых релизов
cleanup() {
    log "Cleaning up old releases..."
    
    local releases=($(ls -t "$RELEASES_DIR"))
    local count=${#releases[@]}
    
    if [ $count -gt $KEEP_RELEASES ]; then
        local to_remove=$((count - KEEP_RELEASES))
        log "Removing $to_remove old release(s)..."
        
        for ((i=$KEEP_RELEASES; i<$count; i++)); do
            rm -rf "$RELEASES_DIR/${releases[$i]}"
            log "Removed: ${releases[$i]}"
        done
    else
        log "No old releases to remove"
    fi
}

# Показать текущую версию
show_current() {
    if [ -L "$CURRENT_LINK" ]; then
        local current=$(readlink "$CURRENT_LINK" | xargs basename)
        log "Current version: $current"
    else
        warning "No current version deployed"
    fi
}

# Главная функция
main() {
    case "${1:-}" in
        deploy)
            setup_directories
            deploy
            cleanup
            ;;
        rollback)
            rollback
            ;;
        current)
            show_current
            ;;
        *)
            echo "Usage: $0 {deploy|rollback|current}"
            exit 1
            ;;
    esac
}

main "$@"
```

### Задания для повторения

**Повтор 1:** Набери скрипт, запусти `deploy` несколько раз  
**Повтор 2:** Добавь healthcheck после деплоя  
**Повтор 3:** Добавь автоматический rollback при ошибке  
**Повтор 4:** Добавь загрузку артефакта из S3/registry  
**Повтор 5:** Добавь уведомления в Slack о деплое

---

## Скрипт 9: Сбор метрик системы (Python)

**Задача:** Собрать CPU, RAM, Disk метрики и сохранить в JSON.

python

```python
#!/usr/bin/env python3

import psutil
import json
from datetime import datetime

def get_cpu_metrics():
    """Получить метрики CPU"""
    return {
        'usage_percent': psutil.cpu_percent(interval=1),
        'count': psutil.cpu_count(),
        'load_avg': psutil.getloadavg() if hasattr(psutil, 'getloadavg') else None
    }

def get_memory_metrics():
    """Получить метрики памяти"""
    mem = psutil.virtual_memory()
    swap = psutil.swap_memory()
    
    return {
        'total_mb': mem.total / (1024 ** 2),
        'used_mb': mem.used / (1024 ** 2),
        'available_mb': mem.available / (1024 ** 2),
        'usage_percent': mem.percent,
        'swap_used_mb': swap.used / (1024 ** 2),
        'swap_percent': swap.percent
    }

def get_disk_metrics():
    """Получить метрики диска"""
    partitions = []
    
    for partition in psutil.disk_partitions():
        try:
            usage = psutil.disk_usage(partition.mountpoint)
            partitions.append({
                'device': partition.device,
                'mountpoint': partition.mountpoint,
                'total_gb': usage.total / (1024 ** 3),
                'used_gb': usage.used / (1024 ** 3),
                'free_gb': usage.free / (1024 ** 3),
                'usage_percent': usage.percent
            })
        except PermissionError:
            continue
    
    return partitions

def get_network_metrics():
    """Получить метрики сети"""
    net_io = psutil.net_io_counters()
    
    return {
        'bytes_sent_mb': net_io.bytes_sent / (1024 ** 2),
        'bytes_recv_mb': net_io.bytes_recv / (1024 ** 2),
        'packets_sent': net_io.packets_sent,
        'packets_recv': net_io.packets_recv
    }

def collect_metrics():
    """Собрать все метрики"""
    metrics = {
        'timestamp': datetime.now().isoformat(),
        'hostname': psutil.os.uname().nodename if hasattr(psutil.os, 'uname') else 'unknown',
        'cpu': get_cpu_metrics(),
        'memory': get_memory_metrics(),
        'disk': get_disk_metrics(),
        'network': get_network_metrics()
    }
    
    return metrics

def print_metrics(metrics):
    """Вывести метрики в читаемом виде"""
    print(f"=== System Metrics at {metrics['timestamp']} ===\n")
    
    # CPU
    cpu = metrics['cpu']
    print(f"CPU Usage: {cpu['usage_percent']}%")
    print(f"CPU Count: {cpu['count']}")
    
    # Memory
    mem = metrics['memory']
    print(f"\nMemory:")
    print(f"  Used: {mem['used_mb']:.1f} MB / {mem['total_mb']:.1f} MB ({mem['usage_percent']}%)")
    print(f"  Swap: {mem['swap_used_mb']:.1f} MB ({mem['swap_percent']}%)")
    
    # Disk
    print(f"\nDisk Usage:")
    for disk in metrics['disk']:
        print(f"  {disk['mountpoint']}: {disk['used_gb']:.1f} GB / {disk['total_gb']:.1f} GB ({disk['usage_percent']}%)")
    
    # Network
    net = metrics['network']
    print(f"\nNetwork:")
    print(f"  Sent: {net['bytes_sent_mb']:.1f} MB")
    print(f"  Received: {net['bytes_recv_mb']:.1f} MB")

def save_metrics(metrics, filename='metrics.json'):
    """Сохранить метрики в файл"""
    with open(filename, 'w') as f:
        json.dump(metrics, f, indent=2)
    print(f"\nMetrics saved to {filename}")

def main():
    metrics = collect_metrics()
    print_metrics(metrics)
    save_metrics(metrics)

if __name__ == "__main__":
    main()
```

### Задания для повторения

**Повтор 1:** Набери скрипт, установи psutil (`pip install psutil`), запусти  
**Повтор 2:** Добавь сохранение метрик с timestamp в имени файла  
**Повтор 3:** Добавь отправку метрик в Prometheus/InfluxDB  
**Повтор 4:** Добавь алерты при превышении порогов  
**Повтор 5:** Добавь график изменения метрик во времени

---

## Скрипт 10: Управление Docker контейнерами (Python)

**Задача:** Проверить статус контейнеров, перезапустить упавшие.

python

```python
#!/usr/bin/env python3

import subprocess
import json
import sys

def run_command(command):
    """Выполнить shell команду"""
    try:
        result = subprocess.run(
            command,
            shell=True,
            capture_output=True,
            text=True,
            check=True
        )
        return result.stdout.strip()
    except subprocess.CalledProcessError as e:
        print(f"Error executing command: {e}")
        return None

def get_containers():
    """Получить список всех контейнеров"""
    output = run_command("docker ps -a --format '{{json .}}'")
    
    if not output:
        return []
    
    containers = []
    for line in output.split('\n'):
        if line:
            containers.append(json.loads(line))
    
    return containers

def check_container_health(container):
    """Проверить здоровье контейнера"""
    name = container['Names']
    status = container['Status']
    state = container['State']
    
    return {
        'name': name,
        'status': status,
        'state': state,
        'is_running': state == 'running',
        'is_healthy': 'healthy' in status.lower() if 'health' in status.lower() else None
    }

def restart_container(name):
    """Перезапустить контейнер"""
    print(f"Restarting container: {name}")
    result = run_command(f"docker restart {name}")
    
    if result is not None:
        print(f"  ✓ Container {name} restarted successfully")
        return True
    else:
        print(f"  ✗ Failed to restart {name}")
        return False

def get_container_logs(name, lines=20):
    """Получить последние логи контейнера"""
    return run_command(f"docker logs --tail {lines} {name}")

def monitor_containers():
    """Мониторинг контейнеров"""
    print("=== Docker Container Monitor ===\n")
    
    containers = get_containers()
    
    if not containers:
        print("No containers found")
        return
    
    print(f"Total containers: {len(containers)}\n")
    
    unhealthy_containers = []
    
    for container in containers:
        health = check_container_health(container)
        
        status_icon = "✓" if health['is_running'] else "✗"
        print(f"{status_icon} {health['name']}")
        print(f"  State: {health['state']}")
        print(f"  Status: {health['status']}")
        
        if not health['is_running']:
            unhealthy_containers.append(health['name'])
        
        print()
    
    # Перезапуск упавших контейнеров
    if unhealthy_containers:
        print(f"\n=== Found {len(unhealthy_containers)} stopped container(s) ===")
        
        for name in unhealthy_containers:
            restart_container(name)
    else:
        print("All containers are running!")

def show_container_stats():
    """Показать статистику использования ресурсов"""
    print("\n=== Container Resource Usage ===\n")
    output = run_command("docker stats --no-stream --format 'table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}'")
    print(output)

def main():
    if len(sys.argv) > 1:
        command = sys.argv[1]
        
        if command == "monitor":
            monitor_containers()
        elif command == "stats":
            show_container_stats()
        elif command == "restart":
            if len(sys.argv) < 3:
                print("Usage: python script.py restart <container_name>")
                sys.exit(1)
            restart_container(sys.argv[2])
        else:
            print("Unknown command. Use: monitor, stats, or restart")
            sys.exit(1)
    else:
        monitor_containers()

if __name__ == "__main__":
    main()
```

### Задания для повторения

**Повтор 1:** Набери скрипт, запусти (нужен Docker)  
**Повтор 2:** Добавь фильтр по меткам контейнеров  
**Повтор 3:** Добавь проверку healthcheck endpoint'ов  
**Повтор 4:** Добавь экспорт метрик в JSON  
**Повтор 5:** Добавь автоматический cleanup старых контейнеров

---

## План тренировок на 3 недели

### Неделя 1: Bash Foundation

**День 1-2:** Скрипт #1 (Backup)

- 5 повторов базового скрипта
- Все модификации из заданий

**День 3-4:** Скрипт #2 (Disk monitoring)

- 5 повторов
- Интеграция с первым скриптом

**День 5-7:** Скрипт #3 (Log parsing) + Скрипт #8 (Deploy)

- По 3 повтора каждого
- Комбинированное задание: деплой с проверкой логов

### Неделя 2: Python Essentials

**День 1-2:** Скрипт #4 (Healthcheck)

- 5 повторов базового
- Все модификации

**День 3-4:** Скрипт #5 (JSON) + Скрипт #6 (Logs)

- По 3 повтора каждого
- Комбинированное: чтение конфига и парсинг логов

**День 5-7:** Скрипт #7 (Process monitor)

- 5 повторов
- Интеграция с healthcheck

### Неделя 3: Advanced Integration

**День 1-3:** Скрипт #9 (Metrics)

- 3 повтора базового
- Все модификации
- Интеграция с алертингом

**День 4-5:** Скрипт #10 (Docker)

- 3 повтора
- Модификации

**День 6-7:** Симуляция собеседования

- Комбинированные задачи без подглядывания
- Код-ревью своих решений

---

## Чеклист прогресса

Отмечай выполненное:

### Bash скрипты

- [ ]  Backup (5 повторов + модификации)
- [ ]  Disk monitoring (5 повторов + модификации)
- [ ]  Log parsing (3 повтора + модификации)
- [ ]  Deploy script (3 повтора + модификации)

### Python скрипты

- [ ]  Healthcheck (5 повторов + модификации)
- [ ]  JSON parser (3 повтора + модификации)
- [ ]  Log analyzer (3 повтора + модификации)
- [ ]  Process monitor (5 повторов + модификации)
- [ ]  Metrics collector (3 повтора + модификации)
- [ ]  Docker manager (3 повтора + модификации)

### Финальные навыки

- [ ]  Пишу любой скрипт без подглядывания в документацию
- [ ]  Добавляю error handling автоматически
- [ ]  Могу объяснить код вслух во время написания
- [ ]  Комбинирую несколько скриптов в единое решение

---

## Советы по эффективной тренировке

### 1. Создай рабочее окружение

bash

```bash
# Структура для практики
~/devops-practice/
├── bash/
│   ├── 01-backup/
│   ├── 02-monitoring/
│   └── 03-logs/
├── python/
│   ├── 04-healthcheck/
│   ├── 05-config/
│   └── 06-logs/
└── test-data/
    ├── logs/
    ├── configs/
    └── backups/
```

### 2. Используй таймер

- **1-й повтор:** не ограничивай время, разбирайся
- **2-3 повтор:** засекай время, старайся ускориться
- **4-5 повтор:** пиши на скорость без ошибок

### 3. Код-ревью после каждого скрипта

Задай себе вопросы:

- Что произойдёт если файл не существует?
- Что если нет прав доступа?
- Что если процесс упадёт в середине выполнения?
- Можно ли этот код сделать проще?

### 4. Веди дневник ошибок

Записывай:

- Какую ошибку сделал
- Как исправил
- Как избежать в будущем

### 5. Симулируй собеседование

После 2 недель тренировок:

- Поставь таймер на 30 минут
- Возьми случайный скрипт
- Напиши его с нуля, объясняя вслух

---

## Дополнительные ресурсы

### Документация (держи открытой)

- **Bash:** [https://www.gnu.org/software/bash/manual/](https://www.gnu.org/software/bash/manual/)
- **Python:** [https://docs.python.org/3/](https://docs.python.org/3/)
- **Docker:** [https://docs.docker.com/](https://docs.docker.com/)

### Инструменты для практики

- **ShellCheck** — линтер для bash скриптов
- **pylint** / **black** — форматирование Python
- **tmux** — для работы с несколькими терминалами

### Тестовые данные

Создай реалистичные тестовые данные:

- Логи с разными форматами
- JSON конфиги разной сложности
- Файлы больших размеров

---

## Критерии готовности к собесу

Ты готов, когда:

✅ Пишешь любой из 10 скриптов за 15-20 минут  
✅ Автоматически добавляешь обработку ошибок  
✅ Можешь модифицировать скрипт под новые требования за 5 минут  
✅ Объясняешь код вслух без запинок  
✅ Знаешь типичные edge cases и обрабатываешь их  
✅ Комбинируешь Bash и Python для комплексных задач

---

## Главное правило

**Не читай — пиши!**

Каждый скрипт должен пройти через твои руки минимум 3 раза. Только так появится автоматизм.

Начни с первого скрипта прямо сейчас. Удачи! 🚀
