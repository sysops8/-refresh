# Zabbix Refresh Course для DevOps инженера

## Цель курса
Освежить знания по Zabbix каждые 6-12 месяцев для уверенного прохождения собеседований и эффективной работы с системой мониторинга.

---

## 1. Основы Zabbix

### Что такое Zabbix?
Zabbix - это enterprise-класса open source решение для мониторинга инфраструктуры, приложений и сервисов.

### Архитектура Zabbix
- **Zabbix Server** - центральный компонент, обрабатывает данные
- **Zabbix Database** - хранилище данных (MySQL, PostgreSQL, Oracle)
- **Zabbix Web Interface** - frontend на PHP
- **Zabbix Proxy** - опциональный компонент для распределенного мониторинга
- **Zabbix Agent** - агент на целевых хостах (активный/пассивный режим)
- **Zabbix Sender** - утилита для отправки данных
- **Zabbix Get** - утилита для получения данных с агента

### Порты по умолчанию
```
10050 - Zabbix Agent (пассивный режим)
10051 - Zabbix Server/Proxy
80/443 - Web Interface
```

---

## 2. Установка и Настройка

### Быстрая установка Zabbix Server (Ubuntu/Debian)
```bash
# Добавление репозитория
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
apt update

# Установка компонентов
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent

# Создание БД
mysql -uroot -p
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user 'zabbix'@'localhost' identified by 'password';
grant all privileges on zabbix.* to 'zabbix'@'localhost';
quit;

# Импорт схемы
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix

# Настройка конфига
vi /etc/zabbix/zabbix_server.conf
# DBPassword=password

# Запуск сервисов
systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2
```

### Установка Zabbix Agent
```bash
# Ubuntu/Debian
apt install zabbix-agent

# CentOS/RHEL
yum install zabbix-agent

# Настройка
vi /etc/zabbix/zabbix_agentd.conf
Server=<Zabbix Server IP>
ServerActive=<Zabbix Server IP>
Hostname=<Hostname>

systemctl restart zabbix-agent
systemctl enable zabbix-agent
```

### Docker Compose установка
```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_password
    volumes:
      - mysql-data:/var/lib/mysql

  zabbix-server:
    image: zabbix/zabbix-server-mysql:latest
    environment:
      DB_SERVER_HOST: mysql
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_password
    ports:
      - "10051:10051"
    depends_on:
      - mysql

  zabbix-web:
    image: zabbix/zabbix-web-nginx-mysql:latest
    environment:
      DB_SERVER_HOST: mysql
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_password
      ZBX_SERVER_HOST: zabbix-server
      PHP_TZ: Europe/Moscow
    ports:
      - "8080:8080"
    depends_on:
      - mysql
      - zabbix-server

volumes:
  mysql-data:
```

---

## 3. Основные Концепции

### Host (Узел)
Устройство или сервер, который вы хотите мониторить.

### Item (Элемент данных)
Отдельная метрика для сбора (CPU load, memory usage, disk space).

**Типы Items:**
- Zabbix agent
- SNMP agent
- IPMI agent
- Simple check
- Internal check
- Zabbix trapper
- Calculated
- JMX monitoring
- HTTP agent
- Database monitor

**Типы значений:**
- Numeric (float/unsigned)
- Character
- Log
- Text

### Trigger (Триггер)
Логическое выражение, определяющее проблемное состояние.

```
Примеры триггеров:
{host:system.cpu.load[percpu,avg1].last()}>5
{host:vfs.fs.size[/,pfree].last()}<10
{host:net.if.in[eth0].avg(300)}>1000000
```

### Template (Шаблон)
Набор элементов данных, триггеров и графиков для переиспользования.

### Action (Действие)
Автоматическая реакция на события (отправка уведомлений, выполнение команд).

### Media Type (Тип оповещения)
Канал доставки уведомлений (Email, SMS, Slack, Telegram, PagerDuty).

---

## 4. Типы Мониторинга

### Passive Check (Пассивная проверка)
Zabbix Server запрашивает данные у агента.
```
Server -> "дай данные" -> Agent
Agent -> "вот данные" -> Server
```

### Active Check (Активная проверка)
Агент сам отправляет данные на сервер.
```
Agent -> "получить список items" -> Server
Agent -> "отправка данных" -> Server
```

### SNMP Monitoring
Мониторинг сетевого оборудования через SNMP.

### IPMI Monitoring
Мониторинг аппаратного обеспечения (температура, вентиляторы).

### JMX Monitoring
Мониторинг Java приложений.

### HTTP/API Monitoring
Мониторинг веб-сервисов и REST API.

---

## 5. Практические Примеры Items

### System Metrics
```
CPU:
system.cpu.load[percpu,avg1]
system.cpu.util[,user]

Memory:
vm.memory.size[available]
vm.memory.size[total]

Disk:
vfs.fs.size[/,used]
vfs.fs.size[/,pfree]
vfs.fs.inode[/,pfree]

Network:
net.if.in[eth0]
net.if.out[eth0]
```

### Application Metrics
```
Web Check:
net.tcp.service[http]
web.page.regexp[,<pattern>,<port>,<path>]

Process:
proc.num[nginx]
proc.mem[nginx]

Custom Script:
system.run[/path/to/script.sh]
```

### Log Monitoring
```
log[/var/log/nginx/error.log,"error"]
logrt["/var/log/app/*.log","ERROR"]
```

---

## 6. Triggers - Продвинутые Примеры

### Базовые операторы
```
> < = <> >= <=
and or
```

### Функции триггеров
```
last() - последнее значение
avg(период) - среднее за период
min(период) - минимум
max(период) - максимум
count(период) - количество
diff() - изменилось значение
nodata(период) - нет данных
```

### Примеры триггеров
```
# CPU высокая нагрузка 5 минут
{host:system.cpu.load[percpu,avg1].avg(5m)}>5

# Свободно меньше 10% места на диске
{host:vfs.fs.size[/,pfree].last()}<10

# Процесс не запущен
{host:proc.num[nginx].last()}=0

# Нет данных 5 минут
{host:agent.ping.nodata(5m)}=1

# Высокий трафик 15 минут
{host:net.if.in[eth0].avg(15m)}>100M

# HTTP недоступен
{host:net.tcp.service[http].last()}=0

# Изменился конфиг файл
{host:vfs.file.cksum[/etc/nginx/nginx.conf].diff()}=1
```

---

## 7. Zabbix API

### Аутентификация
```bash
curl -X POST http://zabbix-server/api_jsonrpc.php \
-H "Content-Type: application/json-rpc" \
-d '{
    "jsonrpc": "2.0",
    "method": "user.login",
    "params": {
        "user": "Admin",
        "password": "zabbix"
    },
    "id": 1
}'
```

### Получение списка хостов
```bash
curl -X POST http://zabbix-server/api_jsonrpc.php \
-H "Content-Type: application/json-rpc" \
-d '{
    "jsonrpc": "2.0",
    "method": "host.get",
    "params": {
        "output": ["hostid", "host"]
    },
    "auth": "<AUTH_TOKEN>",
    "id": 2
}'
```

### Создание хоста через API (Python)
```python
import requests
import json

url = 'http://zabbix-server/api_jsonrpc.php'
headers = {'Content-Type': 'application/json-rpc'}

# Логин
auth_data = {
    "jsonrpc": "2.0",
    "method": "user.login",
    "params": {
        "user": "Admin",
        "password": "zabbix"
    },
    "id": 1
}

response = requests.post(url, data=json.dumps(auth_data), headers=headers)
auth_token = response.json()['result']

# Создание хоста
host_data = {
    "jsonrpc": "2.0",
    "method": "host.create",
    "params": {
        "host": "webserver01",
        "interfaces": [{
            "type": 1,
            "main": 1,
            "useip": 1,
            "ip": "192.168.1.10",
            "dns": "",
            "port": "10050"
        }],
        "groups": [{"groupid": "2"}],
        "templates": [{"templateid": "10001"}]
    },
    "auth": auth_token,
    "id": 2
}

response = requests.post(url, data=json.dumps(host_data), headers=headers)
print(response.json())
```

---

## 8. Zabbix Sender

### Отправка данных в Zabbix
```bash
# Простая отправка
zabbix_sender -z zabbix-server -s "hostname" -k custom.metric -o 123

# Отправка из файла
zabbix_sender -z zabbix-server -i /tmp/metrics.txt

# Формат файла metrics.txt:
# hostname custom.metric1 123
# hostname custom.metric2 456
```

### Пример скрипта с sender
```bash
#!/bin/bash
SERVER="zabbix-server"
HOST="webserver01"

# Получаем метрику
VALUE=$(curl -s http://localhost/metrics | jq '.active_connections')

# Отправляем в Zabbix
zabbix_sender -z $SERVER -s $HOST -k custom.active_connections -o $VALUE
```

---

## 9. User Parameters (Пользовательские метрики)

### Настройка в zabbix_agentd.conf
```bash
# Простой параметр
UserParameter=custom.ping.count[*],ping -c $1 $2 | grep transmitted | awk '{print $4}'

# Скрипт
UserParameter=mysql.queries,mysql -uroot -ppassword -e "show status like 'Queries'" | tail -1 | awk '{print $2}'

# С параметрами
UserParameter=docker.containers[*],docker ps -f name=$1 --format "{{.Status}}"

# JSON данные
UserParameter=app.status,/opt/scripts/check_app.sh
```

### Тестирование user parameter
```bash
zabbix_get -s 192.168.1.10 -k "custom.ping.count[5,google.com]"
```

---

## 10. Ansible для автоматизации Zabbix

### Установка роли
```bash
ansible-galaxy install community.zabbix.zabbix_server
ansible-galaxy install community.zabbix.zabbix_agent
```

### Playbook для установки агента
```yaml
---
- name: Install Zabbix Agent
  hosts: all
  become: yes
  vars:
    zabbix_agent_server: "192.168.1.100"
    zabbix_agent_serveractive: "192.168.1.100"
  
  tasks:
    - name: Install Zabbix Agent
      apt:
        name: zabbix-agent
        state: present
        update_cache: yes
    
    - name: Configure Zabbix Agent
      template:
        src: zabbix_agentd.conf.j2
        dest: /etc/zabbix/zabbix_agentd.conf
      notify: restart zabbix-agent
    
    - name: Start and enable Zabbix Agent
      systemd:
        name: zabbix-agent
        state: started
        enabled: yes
  
  handlers:
    - name: restart zabbix-agent
      systemd:
        name: zabbix-agent
        state: restarted
```

### Создание хоста через Ansible
```yaml
---
- name: Add host to Zabbix
  hosts: localhost
  tasks:
    - name: Create a new host
      community.zabbix.zabbix_host:
        server_url: "http://zabbix.example.com"
        login_user: Admin
        login_password: zabbix
        host_name: webserver01
        visible_name: Web Server 01
        host_groups:
          - Linux servers
        link_templates:
          - Template OS Linux
        interfaces:
          - type: 1
            main: 1
            useip: 1
            ip: 192.168.1.10
            dns: ""
            port: 10050
        status: enabled
        state: present
```

---

## 11. Мониторинг Kubernetes с Zabbix

### Используя встроенный шаблон
```yaml
# Создать service account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: zabbix
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: zabbix-monitoring
rules:
- apiGroups: [""]
  resources: ["nodes", "pods", "services", "endpoints"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: zabbix-monitoring
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: zabbix-monitoring
subjects:
- kind: ServiceAccount
  name: zabbix
  namespace: default
```

### HTTP Agent для K8s API
```
# Item Key:
kubernetes.api.get["/api/v1/nodes"]

# URL:
https://kubernetes.default.svc/api/v1/nodes

# Authentication:
Bearer token from service account
```

---

## 12. Low-Level Discovery (LLD)

### Встроенные LLD правила
```
# Обнаружение файловых систем
vfs.fs.discovery

# Обнаружение сетевых интерфейсов
net.if.discovery

# Обнаружение CPU
system.cpu.discovery
```

### Пользовательское LLD правило
```bash
# Script: /usr/local/bin/docker_discovery.sh
#!/bin/bash
echo "{"
echo "  \"data\":["
docker ps --format "{{.Names}}" | while read container; do
    echo "    {"
    echo "      \"{#CONTAINER}\":\"$container\""
    echo "    },"
done | sed '$ s/,$//'
echo "  ]"
echo "}"
```

### User Parameter для LLD
```
UserParameter=docker.discovery,/usr/local/bin/docker_discovery.sh
```

### Item Prototype
```
# Key:
docker.container.status[{#CONTAINER}]

# Type:
Zabbix agent

# Command:
docker inspect --format='{{.State.Status}}' {#CONTAINER}
```

---

## 13. Zabbix Proxy

### Когда использовать Proxy?
- Мониторинг удаленных локаций
- Распределение нагрузки
- Упрощение настройки firewall
- Мониторинг через NAT

### Установка Zabbix Proxy
```bash
# Установка
apt install zabbix-proxy-mysql zabbix-sql-scripts

# Создание БД
mysql -uroot -p
create database zabbix_proxy character set utf8mb4 collate utf8mb4_bin;
create user 'zabbix'@'localhost' identified by 'password';
grant all privileges on zabbix_proxy.* to 'zabbix'@'localhost';
quit;

# Импорт схемы
cat /usr/share/zabbix-sql-scripts/mysql/proxy.sql | mysql -uzabbix -p zabbix_proxy

# Настройка
vi /etc/zabbix/zabbix_proxy.conf
Server=<Zabbix Server IP>
Hostname=ZabbixProxy01
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=password

# Запуск
systemctl restart zabbix-proxy
systemctl enable zabbix-proxy
```

---

## 14. Best Practices

### Производительность
- Используйте partitioning для БД
- Настройте housekeeping правильно
- Используйте Proxy для больших инфраструктур
- Оптимизируйте интервалы сбора данных
- Используйте тренды для исторических данных

### Безопасность
- Измените пароль Admin сразу после установки
- Используйте HTTPS для веб-интерфейса
- Настройте PSK шифрование для агентов
- Ограничьте доступ к Zabbix Server/Proxy через firewall
- Используйте отдельного пользователя БД с минимальными правами

### Мониторинг
- Начинайте с готовых templates
- Используйте value mapping для читаемости
- Группируйте хосты логически
- Настройте maintenance периоды
- Используйте теги для фильтрации

### Уведомления
- Настройте эскалацию
- Используйте разные media types для разных severity
- Настройте recovery messages
- Используйте macros в уведомлениях
- Настройте quiet periods для уведомлений

---

## 15. Типичные Вопросы на Собеседованиях

### Q: В чем разница между активными и пассивными проверками?
**A:** При пассивных проверках Zabbix Server запрашивает данные у агента. При активных - агент сам отправляет данные на сервер. Активные проверки эффективнее при большом количестве хостов.

### Q: Что такое Zabbix Proxy и когда его использовать?
**A:** Zabbix Proxy - это промежуточный сервер между Server и Agent. Используется для мониторинга удаленных локаций, распределения нагрузки и упрощения сетевой конфигурации.

### Q: Как масштабировать Zabbix?
**A:** 
- Использовать Zabbix Proxy
- Оптимизировать БД (partitioning, indexes)
- Настроить housekeeping
- Использовать кэширование
- Распределить нагрузку (active checks)
- Настроить value cache

### Q: Как мониторить Docker контейнеры?
**A:** Несколько способов:
- Zabbix Agent внутри контейнера
- Docker API через HTTP agent
- Zabbix Sender из контейнера
- cAdvisor + Zabbix
- LLD для автообнаружения контейнеров

### Q: Что такое LLD и как его использовать?
**A:** Low-Level Discovery - автоматическое обнаружение объектов (файловых систем, сетевых интерфейсов, Docker контейнеров). Использует JSON формат для возврата списка обнаруженных объектов, для которых затем создаются items, triggers и graphs.

### Q: Как отправлять кастомные метрики в Zabbix?
**A:** 
- User Parameters в zabbix_agentd.conf
- Zabbix Sender
- Zabbix Trapper items
- HTTP Agent с REST API

### Q: Как обеспечить безопасность Zabbix?
**A:**
- PSK шифрование для агентов
- HTTPS для веб-интерфейса
- Firewall правила
- Регулярные обновления
- Безопасные пароли
- Минимальные права для БД

### Q: Как работает housekeeping в Zabbix?
**A:** Housekeeping - процесс очистки старых данных из БД. Настраивается глобально или для каждого item отдельно. Определяет как долго хранить history и trends.

---

## 16. Полезные Команды

### Проверка статуса
```bash
# Статус сервисов
systemctl status zabbix-server
systemctl status zabbix-agent

# Логи
tail -f /var/log/zabbix/zabbix_server.log
tail -f /var/log/zabbix/zabbix_agentd.log

# Проверка порта
netstat -tulpn | grep zabbix
ss -tulpn | grep 10050
```

### Тестирование агента
```bash
# Получить значение item
zabbix_get -s 192.168.1.10 -k system.cpu.load[percpu,avg1]

# Проверить конфигурацию
zabbix_agentd -t system.cpu.load[percpu,avg1]

# Список активных проверок
zabbix_agentd -p
```

### Работа с БД
```sql
-- Размер таблиц
SELECT table_name, 
       ROUND(((data_length + index_length) / 1024 / 1024), 2) AS "Size (MB)"
FROM information_schema.TABLES 
WHERE table_schema = "zabbix"
ORDER BY (data_length + index_length) DESC;

-- Количество хостов
SELECT COUNT(*) FROM hosts WHERE status=0;

-- Количество items
SELECT COUNT(*) FROM items WHERE status=0;

-- Активные триггеры
SELECT COUNT(*) FROM triggers WHERE status=0 AND value=1;
```

---

## 17. Troubleshooting

### Агент не подключается
```bash
# Проверить firewall
iptables -L -n | grep 10050
firewall-cmd --list-all

# Проверить конфиг
cat /etc/zabbix/zabbix_agentd.conf | grep -v '^#' | grep -v '^$'

# Проверить доступность
telnet zabbix-server 10051

# Проверить из сервера
zabbix_get -s agent-host -k agent.ping
```

### Высокая нагрузка на сервер
```bash
# Проверить очередь
zabbix_server -R config_cache_reload

# В веб-интерфейсе: Administration -> Queue
# Проверить:
# - Количество items
# - Частоту опроса
# - Размер БД
# - Value cache utilization
```

### Проблемы с БД
```bash
# Проверить соединения
mysql -e "SHOW PROCESSLIST;"

# Оптимизация таблиц
mysqlcheck -o zabbix

# Проверить размер
du -sh /var/lib/mysql/zabbix/
```

---

## 18. Интеграции

### Grafana + Zabbix
```bash
# Установка плагина
grafana-cli plugins install alexanderzobnin-zabbix-app

# Настройка в Grafana:
# Configuration -> Plugins -> Zabbix
# Add Data Source:
# - URL: http://zabbix-server/api_jsonrpc.php
# - Username/Password
```

### Slack интеграция
```javascript
// Media Type: Webhook
// Script:
try {
    var params = JSON.parse(value);
    var req = new HttpRequest();
    req.addHeader('Content-Type: application/json');
    
    var payload = {
        "text": params.subject,
        "attachments": [{
            "color": "danger",
            "text": params.message
        }]
    };
    
    req.post('https://hooks.slack.com/services/YOUR/WEBHOOK/URL',
        JSON.stringify(payload));
    
    return 'OK';
} catch (error) {
    throw 'Slack notification failed: ' + error;
}
```

### Telegram бот
```bash
# User Parameter
UserParameter=telegram.send[*],curl -s "https://api.telegram.org/bot<TOKEN>/sendMessage?chat_id=$1&text=$2"
```

---

## 19. Мониторинг популярных сервисов

### Nginx
```bash
# Items:
web.page.get[localhost,/nginx_status,80]
web.page.regexp[localhost,/nginx_status,80,"Active connections: ([0-9]+)",\1]

# Nginx конфиг:
location /nginx_status {
    stub_status on;
    access_log off;
    allow 127.0.0.1;
    deny all;
}
```

### MySQL/MariaDB
```bash
# Template: MySQL by Zabbix agent
# Custom check:
UserParameter=mysql.queries,mysql -umonitor -ppassword -e "SHOW GLOBAL STATUS LIKE 'Queries'" | tail -1 | awk '{print $2}'
```

### PostgreSQL
```bash
# Template: PostgreSQL by Zabbix agent
UserParameter=pgsql.connections,psql -U zabbix -h localhost -t -c "SELECT count(*) FROM pg_stat_activity"
```

### Redis
```bash
UserParameter=redis.info[*],redis-cli INFO | grep $1 | awk -F: '{print $2}'
# Usage: redis.info[used_memory]
```

### MongoDB
```bash
UserParameter=mongodb.status[*],mongo --quiet --eval "db.serverStatus().$1"
```

---

## 20. Резервное копирование

### Backup конфигурации
```bash
#!/bin/bash
# backup_zabbix.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/zabbix"

# Backup БД
mysqldump -uzabbix -ppassword zabbix | gzip > $BACKUP_DIR/zabbix_db_$DATE.sql.gz

# Backup конфигов
tar -czf $BACKUP_DIR/zabbix_conf_$DATE.tar.gz \
    /etc/zabbix/ \
    /usr/share/zabbix/

# Удаление старых бэкапов (старше 30 дней)
find $BACKUP_DIR -name "zabbix_*" -mtime +30 -delete
```

### Восстановление
```bash
# Восстановление БД
gunzip < zabbix_db_20240101_120000.sql.gz | mysql -uzabbix -ppassword zabbix

# Восстановление конфигов
tar -xzf zabbix_conf_20240101_120000.tar.gz -C /
```

---

## Заключение

### Что нужно помнить для собеседования:
1. Архитектура Zabbix (Server, Agent, Proxy, Web)
2. Разница между активными и пассивными проверками
3. Основные концепции: Host, Item, Trigger, Template, Action
4. Типы мониторинга и когда их использовать
5. Как создавать кастомные метрики (User Parameters, Sender)
6. Low-Level Discovery для автообнаружения
7. Zabbix API для автоматизации
8. Best practices и оптимизация
9. Troubleshooting основных проблем
10. Интеграция с другими инструментами

### Полезные ссылки:
- Официальная документация: https://www.zabbix.com/documentation
- Zabbix Share (Templates): https://share.zabbix.com/
- GitHub: https://github.com/zabbix/zabbix
- Community: https://www.zabbix.com/forum/

---

**Рекомендация:** Пройдите этот курс за 2-3 часа, настройте тестовую среду через Docker Compose, создайте несколько кастомных метрик и протестируйте API.

---

## 21. Zabbix Macros

### Встроенные макросы
```
{HOST.NAME} - имя хоста
{HOST.IP} - IP адрес хоста
{HOST.CONN} - соединение с хостом (IP или DNS)
{ITEM.VALUE} - текущее значение item
{ITEM.LASTVALUE} - последнее значение
{TRIGGER.STATUS} - статус триггера
{TRIGGER.SEVERITY} - severity триггера
{EVENT.DATE} - дата события
{EVENT.TIME} - время события
{$MACRO} - пользовательский макрос
```

### Пользовательские макросы
```
# Глобальный уровень
Administration -> General -> Macros
{$DISK.WARN} = 20
{$DISK.CRIT} = 10
{$CPU.MAX} = 80

# Уровень хоста
Configuration -> Hosts -> Macros
{$DB.PORT} = 5432
{$APP.PATH} = /opt/myapp
```

### Использование в триггерах
```
{host:vfs.fs.size[/,pfree].last()}<{$DISK.WARN}
{host:system.cpu.load[percpu,avg1].avg(5m)}>{$CPU.MAX}
```

### Context macros (Zabbix 6.0+)
```
{$LOW_SPACE_LIMIT:"/"}=10
{$LOW_SPACE_LIMIT:"/home"}=5
{$LOW_SPACE_LIMIT:"/var"}=15
```

---

## 22. Zabbix Maps

### Создание Network Map
```
Configuration -> Maps -> Create map

Элементы:
- Host
- Map
- Trigger
- Host group
- Image

Связи:
- Links between elements
- Link indicators (triggers)
```

### Автоматический map через API
```python
import requests
import json

def create_map(auth_token, map_name):
    url = 'http://zabbix/api_jsonrpc.php'
    
    map_data = {
        "jsonrpc": "2.0",
        "method": "map.create",
        "params": {
            "name": map_name,
            "width": 800,
            "height": 600,
            "selements": [
                {
                    "elementtype": 0,  # host
                    "elements": [{"hostid": "10084"}],
                    "x": 100,
                    "y": 100
                }
            ]
        },
        "auth": auth_token,
        "id": 1
    }
    
    response = requests.post(url, json=map_data)
    return response.json()
```

---

## 23. Value Mapping

### Примеры маппинга
```
# Service state
0 → Down
1 → Up

# Process state
0 → Stopped
1 → Running

# Boolean
0 → False
1 → True
```

### Создание value map
```
Administration -> General -> Value mapping

Name: Service Status
Mappings:
  0 = Down
  1 = Up
```

### Применение к item
```
Configuration -> Hosts -> Items -> Item
Show value: Service Status
```

---

## 24. Maintenance Mode

### Создание maintenance периода
```
Configuration -> Maintenance -> Create maintenance period

Параметры:
- Name: Server Maintenance
- Type: With data collection / No data collection
- Active since: 2024-12-28 22:00
- Active till: 2024-12-29 06:00
- Period: One time only / Daily / Weekly / Monthly

Hosts/Groups:
- Select hosts or host groups
```

### Maintenance через API
```python
maintenance_data = {
    "jsonrpc": "2.0",
    "method": "maintenance.create",
    "params": {
        "name": "Sunday maintenance",
        "active_since": 1640995200,
        "active_till": 1641024000,
        "hostids": ["10084"],
        "timeperiods": [{
            "timeperiod_type": 0,
            "period": 3600
        }]
    },
    "auth": auth_token,
    "id": 1
}
```

### Скрипт для автоматического maintenance
```bash
#!/bin/bash
# Создание maintenance перед деплоем

HOST_ID="10084"
START=$(date +%s)
END=$((START + 3600))  # 1 час

curl -X POST http://zabbix/api_jsonrpc.php \
-H "Content-Type: application/json" \
-d "{
    \"jsonrpc\": \"2.0\",
    \"method\": \"maintenance.create\",
    \"params\": {
        \"name\": \"Deploy maintenance\",
        \"active_since\": $START,
        \"active_till\": $END,
        \"hostids\": [\"$HOST_ID\"],
        \"timeperiods\": [{
            \"timeperiod_type\": 0,
            \"period\": 3600
        }]
    },
    \"auth\": \"$AUTH_TOKEN\",
    \"id\": 1
}"
```

---

## 25. Distributed Monitoring

### Архитектура с Proxy
```
┌─────────────────┐
│  Zabbix Server  │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
┌───▼───┐ ┌──▼────┐
│Proxy 1│ │Proxy 2│
└───┬───┘ └──┬────┘
    │        │
┌───▼───┐ ┌──▼────┐
│Agents │ │Agents │
└───────┘ └───────┘
```

### HA Setup (Zabbix 6.0+)
```bash
# Node 1
vi /etc/zabbix/zabbix_server.conf
HANodeName=zabbix-node1
NodeAddress=192.168.1.10:10051

# Node 2
vi /etc/zabbix/zabbix_server.conf
HANodeName=zabbix-node2
NodeAddress=192.168.1.11:10051

# Database должна быть общая (PostgreSQL/MySQL cluster)
```

### Load Balancing с Proxy
```
# Несколько Proxy для одной сети
Proxy1 → Hosts 1-100
Proxy2 → Hosts 101-200
Proxy3 → Hosts 201-300

# Настройка в Zabbix:
Configuration -> Hosts -> Host
Monitored by proxy: ProxyName
```

---

## 26. Zabbix и Prometheus

### Использование Prometheus exporter
```yaml
# docker-compose.yml
services:
  node-exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"
```

### HTTP Agent для Prometheus metrics
```
# Item configuration
Name: Prometheus Node CPU
Type: HTTP agent
URL: http://node-exporter:9100/metrics
Request type: GET
Preprocessing:
  - Prometheus pattern
  - Pattern: node_cpu_seconds_total{mode="idle"}
  - Function: avg
```

### Парсинг Prometheus в Zabbix
```
# Preprocessing steps:
1. Prometheus to JSON
2. JSONPath: $.node_cpu_seconds_total[?(@.mode=="idle")].value
3. Custom multiplier: 1
```

---

## 27. Alert Escalation

### Пример эскалации
```
Escalation step 1 (0-10 min):
  - Send to: DevOps team (Email)
  
Escalation step 2 (10-30 min):
  - Send to: DevOps team (Slack)
  - Send to: On-call engineer (SMS)
  
Escalation step 3 (30+ min):
  - Send to: Manager (Email + SMS)
  - Execute: /scripts/critical_alert.sh
```

### Настройка в Action
```
Configuration -> Actions -> Create action

Conditions:
  - Trigger severity >= High
  
Operations:
  Step 1: 1-1 (0-10 min)
    Send message to: DevOps via Email
    
  Step 2: 2-3 (10-30 min)
    Send message to: DevOps via Slack
    Send message to: On-call via SMS
    
  Step 3: 4-0 (30 min until resolved)
    Send message to: Manager via Email
    Run command: critical_alert.sh
```

---

## 28. Zabbix Database Partitioning

### Преимущества партиционирования
- Быстрое удаление старых данных
- Улучшенная производительность запросов
- Оптимизация housekeeping

### Пример партиционирования (MySQL)
```sql
-- Партиционирование таблицы history
ALTER TABLE history PARTITION BY RANGE (clock) (
    PARTITION p2024_01 VALUES LESS THAN (UNIX_TIMESTAMP('2024-02-01')),
    PARTITION p2024_02 VALUES LESS THAN (UNIX_TIMESTAMP('2024-03-01')),
    PARTITION p2024_03 VALUES LESS THAN (UNIX_TIMESTAMP('2024-04-01')),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

### Автоматизация через скрипт
```bash
#!/bin/bash
# partition_maintenance.sh

MYSQL="mysql -uzabbix -ppassword zabbix"

# Создание партиции на следующий месяц
NEXT_MONTH=$(date -d "next month" +%Y-%m-01)
NEXT_MONTH_UNIX=$(date -d "$NEXT_MONTH" +%s)
PARTITION_NAME="p$(date -d "next month" +%Y_%m)"

$MYSQL -e "ALTER TABLE history 
    REORGANIZE PARTITION p_future INTO (
        PARTITION $PARTITION_NAME VALUES LESS THAN ($NEXT_MONTH_UNIX),
        PARTITION p_future VALUES LESS THAN MAXVALUE
    );"

# Удаление старых партиций (старше 90 дней)
OLD_DATE=$(date -d "90 days ago" +%Y_%m)
$MYSQL -e "ALTER TABLE history DROP PARTITION p$OLD_DATE;" 2>/dev/null
```

---

## 29. Custom Dashboards

### Создание dashboard
```
Monitoring -> Dashboards -> Create dashboard

Widgets:
- Graph (classic)
- Graph prototype
- Plain text
- URL
- Problems
- Problems by severity
- Top hosts
- Map
- Item history
```

### Dynamic dashboard через API
```python
def create_dashboard(auth_token, name):
    dashboard_data = {
        "jsonrpc": "2.0",
        "method": "dashboard.create",
        "params": {
            "name": name,
            "pages": [{
                "widgets": [
                    {
                        "type": "problems",
                        "name": "Current Problems",
                        "x": 0,
                        "y": 0,
                        "width": 12,
                        "height": 5,
                        "fields": [{
                            "type": "INTEGER",
                            "name": "show_tags",
                            "value": "3"
                        }]
                    },
                    {
                        "type": "graph",
                        "name": "CPU Usage",
                        "x": 0,
                        "y": 5,
                        "width": 6,
                        "height": 5,
                        "fields": [{
                            "type": "ITEM",
                            "name": "itemid",
                            "value": "23295"
                        }]
                    }
                ]
            }]
        },
        "auth": auth_token,
        "id": 1
    }
    
    response = requests.post(url, json=dashboard_data)
    return response.json()
```

---

## 30. Performance Tuning

### Оптимизация Zabbix Server
```bash
# /etc/zabbix/zabbix_server.conf

# Cache
CacheSize=128M
HistoryCacheSize=64M
TrendCacheSize=32M
ValueCacheSize=256M

# Processes
StartPollers=50
StartPollersUnreachable=10
StartTrappers=20
StartPingers=10
StartDiscoverers=10
StartHTTPPollers=10

# Timeouts
Timeout=10
TrapperTimeout=300
UnreachablePeriod=120
UnavailableDelay=60

# Database
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=password
DBSocket=/var/run/mysqld/mysqld.sock

# Connection pool
MaxHousekeeperDelete=5000
```

### Оптимизация MySQL/MariaDB
```ini
# /etc/mysql/my.cnf

[mysqld]
innodb_buffer_pool_size=4G
innodb_log_file_size=512M
innodb_flush_log_at_trx_commit=2
innodb_flush_method=O_DIRECT

max_connections=500
tmp_table_size=64M
max_heap_table_size=64M

query_cache_size=0
query_cache_type=OFF

# Для больших таблиц
innodb_file_per_table=1
```

### Оптимизация PostgreSQL
```ini
# /etc/postgresql/14/main/postgresql.conf

shared_buffers=4GB
effective_cache_size=12GB
maintenance_work_mem=1GB
work_mem=64MB

checkpoint_completion_target=0.9
wal_buffers=16MB

max_connections=500

# Autovacuum
autovacuum=on
autovacuum_max_workers=4
```

### Мониторинг производительности Zabbix
```bash
# Internal items для мониторинга самого Zabbix
zabbix[wcache,values,all]
zabbix[wcache,history,pused]
zabbix[queue,10m]
zabbix[process,poller,avg,busy]

# Проверка очереди
zabbix_server -R config_cache_reload

# В веб-интерфейсе
Reports -> System information
Administration -> Queue
```

---

## 31. Security Hardening

### TLS/PSK шифрование агентов
```bash
# Генерация PSK ключа
openssl rand -hex 32 > /etc/zabbix/zabbix_agentd.psk

# Конфигурация агента
vi /etc/zabbix/zabbix_agentd.conf
TLSConnect=psk
TLSAccept=psk
TLSPSKIdentity=PSK001
TLSPSKFile=/etc/zabbix/zabbix_agentd.psk

# В веб-интерфейсе (Host configuration)
Encryption:
  Connections to host: PSK
  Connections from host: PSK
  PSK identity: PSK001
  PSK: [содержимое файла zabbix_agentd.psk]
```

### TLS с сертификатами
```bash
# Генерация сертификатов
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/zabbix/zabbix_agentd.key \
  -out /etc/zabbix/zabbix_agentd.crt

# Конфигурация
TLSConnect=cert
TLSAccept=cert
TLSCAFile=/etc/zabbix/ca.crt
TLSCertFile=/etc/zabbix/zabbix_agentd.crt
TLSKeyFile=/etc/zabbix/zabbix_agentd.key
```

### Web интерфейс HTTPS
```bash
# Apache
a2enmod ssl
a2ensite default-ssl

# Nginx
server {
    listen 443 ssl;
    ssl_certificate /etc/ssl/certs/zabbix.crt;
    ssl_certificate_key /etc/ssl/private/zabbix.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    
    location / {
        proxy_pass http://zabbix-web:8080;
    }
}
```

### Firewall правила
```bash
# iptables
iptables -A INPUT -p tcp --dport 10051 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 10051 -j DROP

# firewalld
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="10051" accept'
firewall-cmd --reload

# ufw
ufw allow from 192.168.1.0/24 to any port 10051
```

---

## 32. Monitoring as Code

### Terraform для Zabbix
```hcl
# main.tf
terraform {
  required_providers {
    zabbix = {
      source = "tpretz/zabbix"
      version = "0.16.0"
    }
  }
}

provider "zabbix" {
  url      = "http://zabbix.example.com/api_jsonrpc.php"
  username = "Admin"
  password = var.zabbix_password
}

resource "zabbix_host" "webserver" {
  host   = "webserver01"
  name   = "Web Server 01"
  groups = ["2"]  # Linux servers

  interface {
    ip   = "192.168.1.10"
    main = true
    type = 1  # Agent
    port = 10050
  }

  template_id = ["10001"]  # Template OS Linux
}

resource "zabbix_trigger" "cpu_high" {
  description = "CPU usage is too high"
  expression  = "{webserver01:system.cpu.load[percpu,avg1].avg(5m)}>5"
  priority    = "high"
  enabled     = true
}
```

### GitOps для Zabbix конфигурации
```yaml
# .gitlab-ci.yml
stages:
  - validate
  - deploy

validate:
  stage: validate
  script:
    - terraform init
    - terraform validate
    - terraform plan

deploy:
  stage: deploy
  script:
    - terraform init
    - terraform apply -auto-approve
  only:
    - main
```

---

## 33. Advanced Monitoring Scenarios

### Мониторинг SSL сертификатов
```bash
# User Parameter
UserParameter=ssl.cert.expire[*],echo | openssl s_client -servername $1 -connect $1:443 2>/dev/null | openssl x509 -noout -enddate | cut -d= -f2 | xargs -I {} date -d "{}" +%s

# Item
Name: SSL Certificate Expiry
Key: ssl.cert.expire[example.com]
Type: Zabbix agent

# Trigger
{host:ssl.cert.expire[example.com].last()}-{host:system.localtime.last()}<30*24*3600
```

### Мониторинг API endpoints
```
# HTTP Agent item
Name: API Health Check
Type: HTTP agent
URL: https://api.example.com/health
Request type: GET
Headers: Authorization: Bearer {$API_TOKEN}
Required status codes: 200

# Preprocessing
JSONPath: $.status
```

### Мониторинг log файлов с регулярками
```bash
# Item
Key: log[/var/log/application.log,"ERROR.*database connection",,,skip]
Type: Zabbix agent (active)

# Trigger
{host:log[/var/log/application.log,"ERROR.*database connection",,,skip].str(ERROR)}=1
```

### Мониторинг Jenkins jobs
```bash
# User Parameter
UserParameter=jenkins.job.status[*],curl -s "http://jenkins:8080/job/$1/lastBuild/api/json" | jq -r '.result'

# Trigger
{host:jenkins.job.status[deploy-prod].str(FAILURE)}=1
```

---

## 34. Calculated Items

### Примеры вычисляемых items
```
# Процент использования диска
100-last(vfs.fs.size[/,pfree])

# Средняя загрузка CPU за 3 периода
(last(system.cpu.load[percpu,avg1])+last(system.cpu.load[percpu,avg5])+last(system.cpu.load[percpu,avg15]))/3

# Входящий + исходящий трафик
last(net.if.in[eth0])+last(net.if.out[eth0])

# Если сервис не работает, умножить на -1
last(net.tcp.service[http])*(-1)

# Процент доступности за день
avg(net.tcp.service[http],1d)*100
```

### Dependent Items
```
Master Item:
  Key: web.page.get[api.example.com,/metrics]
  Type: HTTP agent

Dependent Item 1:
  Master item: web.page.get[...]
  Preprocessing: JSONPath: $.cpu_usage

Dependent Item 2:
  Master item: web.page.get[...]
  Preprocessing: JSONPath: $.memory_usage
```

---

## 35. Zabbix Scripts и Remote Commands

### Создание скрипта
```
Administration -> Scripts -> Create script

Name: Restart Apache
Type: Script
Execute on: Zabbix server
Commands:
  ssh root@{HOST.CONN} "systemctl restart apache2"
  
User group: Zabbix administrators
Host group: Web servers
```

### Remote commands в Actions
```
Configuration -> Actions -> Create action

Operations:
  Remote command:
    Target: Current host
    Type: Custom script
    Commands:
      /usr/local/bin/restart_service.sh {HOST.NAME}
```

### Безопасное выполнение команд
```bash
# /etc/sudoers.d/zabbix
zabbix ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart apache2
zabbix ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

# Скрипт
#!/bin/bash
# /usr/local/bin/restart_service.sh

SERVICE=$1
ALLOWED_SERVICES="apache2 nginx mysql"

if echo $ALLOWED_SERVICES | grep -q $SERVICE; then
    sudo systemctl restart $SERVICE
    echo "Service $SERVICE restarted"
else
    echo "Service $SERVICE not allowed"
    exit 1
fi
```

---

## Финальный Чеклист для Собеседований

### Теория
- [ ] Объяснить архитектуру Zabbix
- [ ] Разница active/passive checks
- [ ] Когда использовать Proxy
- [ ] Что такое LLD и как работает
- [ ] Типы items и их применение
- [ ] Функции триггеров
- [ ] Как работает housekeeping

### Практика
- [ ] Установить Zabbix Server + Agent
- [ ] Настроить мониторинг CPU/Memory/Disk
- [ ] Создать custom user parameter
- [ ] Настроить trigger с эскалацией
- [ ] Использовать zabbix_sender
- [ ] Работать с Zabbix API
- [ ] Создать LLD правило
- [ ] Настроить интеграцию (Slack/Telegram)

### Автоматизация
- [ ] Ansible playbook для установки
- [ ] Terraform для конфигурации
- [ ] Скрипты для backup/restore
- [ ] CI/CD интеграция

### Production Ready
- [ ] Security hardening (PSK/TLS)
- [ ] Performance tuning
- [ ] Database optimization
- [ ] HA setup
- [ ] Monitoring самого Zabbix

---

## Дополнительные Ресурсы

### Официальные
- Documentation: https://www.zabbix.com/documentation/current/
- Blog: https://blog.zabbix.com/
- Forum: https://www.zabbix.com/forum/
- GitHub: https://github.com/zabbix/zabbix

### Community
- Zabbix Share: https://share.zabbix.com/
- Reddit: r/zabbix
- Awesome Zabbix: https://github.com/zabbix/awesome-zabbix

### Обучение
- Zabbix Certified Specialist
- Zabbix Certified Professional
- Zabbix Summit (ежегодная конференция)

---

**Последний совет:** Держите локальное окружение Zabbix на Docker Compose. Регулярно (раз в 6 месяцев) проходите этот курс, экспериментируйте с новыми features, читайте release notes. Удачи на собеседованиях! 🚀