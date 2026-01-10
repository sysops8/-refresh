# Ansible Refresh: Ежедневный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые концепции Ansible за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:

1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная задача, которую нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Предварительные требования:**

- Linux/Unix базовые знания
- SSH доступ к тестовым серверам (или Docker контейнеры)
- Базовые знания YAML
- Python установлен (для Ansible)

---

## Модуль 1: Базовая архитектура и установка (20 минут)

### 🎯 Напоминалка

**Ansible - что это:**

```
Ansible
├── Агентless (без агентов на целевых хостах)
├── Использует SSH для подключения
├── Декларативный подход (описываем желаемое состояние)
├── Idempotent (можно запускать многократно)
├── Push model (запускается с control node)
└── Python-based
```

**Ключевые компоненты:**

bash

```bash
Control Node          # Машина, с которой запускается Ansible
├── Inventory         # Список управляемых хостов
├── Playbooks        # YAML файлы с задачами
├── Modules          # Готовые блоки функциональности
├── Roles            # Переиспользуемая структура
├── Variables        # Параметры конфигурации
└── Templates        # Jinja2 шаблоны

Managed Nodes        # Целевые серверы
├── Python требуется (обычно уже есть)
├── SSH доступ
└── Sudo/root права (для большинства задач)
```

**Установка:**

bash

````bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible -y

# RHEL/CentOS/Rocky
sudo dnf install ansible -y

# macOS
brew install ansible

# pip (универсально)
pip install ansible

# Проверка
ansible --version
ansible localhost -m ping
````

**Базовая структура проекта:**
````
ansible-project/
├── ansible.cfg           # Конфигурация Ansible
├── inventory/
│   ├── production       # Production хосты
│   └── staging          # Staging хосты
├── group_vars/          # Переменные для групп
│   ├── all.yml
│   ├── webservers.yml
│   └── databases.yml
├── host_vars/           # Переменные для хостов
│   └── server1.yml
├── playbooks/           # Плейбуки
│   ├── site.yml
│   └── webserver.yml
├── roles/               # Роли
│   ├── common/
│   ├── nginx/
│   └── mysql/
└── files/               # Статические файлы
````

**ansible.cfg (базовая конфигурация):**

ini

```ini
[defaults]
inventory = ./inventory/production
remote_user = ansible
host_key_checking = False
retry_files_enabled = False
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 3600

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```

**Inventory (базовый формат):**

ini

```ini
# inventory/production

# Отдельные хосты
server1.example.com
192.168.1.10

# Группы
[webservers]
web1.example.com
web2.example.com ansible_host=192.168.1.20

[databases]
db1.example.com ansible_host=192.168.1.30
db2.example.com ansible_host=192.168.1.31

# Группа групп
[production:children]
webservers
databases

# Переменные для группы
[webservers:vars]
http_port=80
max_clients=200

[databases:vars]
mysql_port=3306
```

**Inventory YAML формат (современный):**

yaml

```yaml
# inventory/production.yml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
          ansible_host: 192.168.1.20
      vars:
        http_port: 80
        max_clients: 200
    
    databases:
      hosts:
        db1.example.com:
          ansible_host: 192.168.1.30
        db2.example.com:
          ansible_host: 192.168.1.31
      vars:
        mysql_port: 3306
    
    production:
      children:
        - webservers
        - databases
```

**Ad-hoc команды (для быстрых задач):**

bash

```bash
# Ping всех хостов
ansible all -m ping

# Проверка uptime
ansible all -m command -a "uptime"

# Установка пакета
ansible webservers -m apt -a "name=nginx state=present" -b

# Копирование файла
ansible all -m copy -a "src=/local/file dest=/remote/file"

# Перезапуск сервиса
ansible webservers -m service -a "name=nginx state=restarted" -b

# Получение фактов
ansible server1 -m setup

# Фильтрация фактов
ansible server1 -m setup -a "filter=ansible_distribution*"

# Выполнение shell команды
ansible all -m shell -a "df -h | grep /dev/sda"

# Проверка синтаксиса без выполнения
ansible-playbook playbook.yml --syntax-check

# Dry-run (check mode)
ansible-playbook playbook.yml --check

# С указанием конкретного инвентори
ansible all -i inventory/staging -m ping
```

**Основные модули (must-know):**

bash

```bash
# Управление пакетами
apt, yum, dnf, package     # Установка пакетов

# Управление файлами
copy, template, file       # Копирование, шаблоны, создание
fetch                      # Скачивание с удаленного хоста
lineinfile, blockinfile   # Редактирование файлов

# Управление сервисами
service, systemd          # Управление сервисами

# Выполнение команд
command                   # Простые команды (без shell features)
shell                     # С shell features (pipes, redirects)
script                    # Запуск локального скрипта на удаленном хосте

# Управление пользователями
user, group              # Пользователи и группы

# Git
git                      # Git операции

# Архивы
unarchive, archive       # Распаковка/создание архивов

# Сеть
uri                      # HTTP запросы
get_url                  # Скачивание файлов

# Debug
debug                    # Вывод информации
assert                   # Проверка условий
```

### 💻 Задание

Подготовь тестовое окружение и выполни базовые операции:

1. **Установи Ansible:**

bash

```bash
# Проверь версию
ansible --version

# Должна быть >= 2.12 (лучше 2.15+)
```

2. **Создай тестовое окружение с Docker:**

bash

```bash
# Создай docker-compose.yml для тестовых хостов
cat > docker-compose.yml <<EOF
version: '3'
services:
  web1:
    image: ubuntu:22.04
    container_name: ansible-web1
    command: |
      bash -c "apt-get update && 
               apt-get install -y openssh-server python3 sudo && 
               mkdir -p /var/run/sshd && 
               echo 'root:password' | chpasswd && 
               sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && 
               service ssh start && 
               tail -f /dev/null"
    ports:
      - "2221:22"
  
  web2:
    image: ubuntu:22.04
    container_name: ansible-web2
    command: |
      bash -c "apt-get update && 
               apt-get install -y openssh-server python3 sudo && 
               mkdir -p /var/run/sshd && 
               echo 'root:password' | chpasswd && 
               sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && 
               service ssh start && 
               tail -f /dev/null"
    ports:
      - "2222:22"
  
  db1:
    image: ubuntu:22.04
    container_name: ansible-db1
    command: |
      bash -c "apt-get update && 
               apt-get install -y openssh-server python3 sudo && 
               mkdir -p /var/run/sshd && 
               echo 'root:password' | chpasswd && 
               sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && 
               service ssh start && 
               tail -f /dev/null"
    ports:
      - "2223:22"
EOF

# Запусти контейнеры
docker-compose up -d

# Подожди пока запустятся
sleep 10
```

3. **Создай inventory:**

bash

```bash
mkdir -p ansible-practice
cd ansible-practice

cat > inventory.ini <<EOF
[webservers]
web1 ansible_host=localhost ansible_port=2221
web2 ansible_host=localhost ansible_port=2222

[databases]
db1 ansible_host=localhost ansible_port=2223

[all:vars]
ansible_user=root
ansible_password=password
ansible_connection=ssh
ansible_python_interpreter=/usr/bin/python3
EOF
```

4. **Создай ansible.cfg:**

bash

```bash
cat > ansible.cfg <<EOF
[defaults]
inventory = ./inventory.ini
host_key_checking = False
retry_files_enabled = False
EOF
```

5. **Протестируй подключение:**

bash

```bash
# Ping всех хостов
ansible all -m ping

# Должен вернуть SUCCESS для всех

# Проверь базовую информацию
ansible all -m setup -a "filter=ansible_distribution*"

# Выполни команду
ansible all -m command -a "hostname"
ansible all -m shell -a "cat /etc/os-release | grep PRETTY_NAME"

# Установи пакет
ansible webservers -m apt -a "name=vim state=present update_cache=yes"

# Создай файл
ansible all -m file -a "path=/tmp/test.txt state=touch"

# Проверь создание
ansible all -m command -a "ls -la /tmp/test.txt"
```

6. **Работа с группами и переменными:**

bash

```bash
# Проверь какие хосты в группе
ansible webservers --list-hosts
ansible databases --list-hosts

# Выполни команду только на web серверах
ansible webservers -m command -a "echo 'I am webserver'"

# Проверь все переменные хоста
ansible web1 -m debug -a "var=hostvars[inventory_hostname]"
```

### 🚀 Бонус (новое)

**1. Используй динамический inventory:**

bash

```bash
# Создай скрипт динамического inventory
cat > dynamic_inventory.py <<'EOF'
#!/usr/bin/env python3
import json

inventory = {
    "webservers": {
        "hosts": ["web1", "web2"],
        "vars": {
            "http_port": 80
        }
    },
    "databases": {
        "hosts": ["db1"],
        "vars": {
            "mysql_port": 3306
        }
    },
    "_meta": {
        "hostvars": {
            "web1": {
                "ansible_host": "localhost",
                "ansible_port": 2221
            },
            "web2": {
                "ansible_host": "localhost",
                "ansible_port": 2222
            },
            "db1": {
                "ansible_host": "localhost",
                "ansible_port": 2223
            }
        }
    }
}

print(json.dumps(inventory, indent=2))
EOF

chmod +x dynamic_inventory.py

# Используй динамический inventory
ansible all -i dynamic_inventory.py -m ping
```

**2. Ansible Vault для секретов:**

bash

```bash
# Создай encrypted файл
ansible-vault create secrets.yml
# Введи пароль: testpass123
# Добавь содержимое:
database_password: "SuperSecretPassword123"
api_key: "secret-api-key-12345"

# Просмотр
ansible-vault view secrets.yml

# Редактирование
ansible-vault edit secrets.yml

# Использование в ad-hoc
ansible all -m debug -a "var=database_password" -e @secrets.yml --ask-vault-pass

# Или создай файл с паролем
echo "testpass123" > .vault_pass
chmod 600 .vault_pass

# Добавь в ansible.cfg
echo "vault_password_file = ./.vault_pass" >> ansible.cfg

# Теперь можно без --ask-vault-pass
ansible all -m debug -a "var=database_password" -e @secrets.yml
```

**3. Ansible facts и custom facts:**

bash

```bash
# Собери все facts
ansible web1 -m setup > web1_facts.json

# Custom facts
ansible web1 -m shell -a "mkdir -p /etc/ansible/facts.d"
ansible web1 -m copy -a "content='#!/bin/bash\necho {\"custom_fact\": \"my_value\"}' dest=/etc/ansible/facts.d/custom.fact mode=0755"

# Обнови facts
ansible web1 -m setup -a "filter=ansible_local"
```

---

## Модуль 2: Playbooks - основа автоматизации (30 минут)

### 🎯 Напоминалка

**Структура Playbook:**

yaml

```yaml
---
- name: Описание playbook
  hosts: целевая_группа
  become: yes              # Использовать sudo
  vars:                    # Переменные
    var_name: value
  
  tasks:                   # Задачи
    - name: Описание задачи
      module_name:
        parameter: value
      
    - name: Другая задача
      another_module:
        param: value
```

**Базовый playbook:**

yaml

```yaml
---
- name: Install and configure nginx
  hosts: webservers
  become: yes
  
  vars:
    http_port: 80
    server_name: "{{ ansible_hostname }}"
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Copy config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx
  
  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

**Handlers (обработчики событий):**

yaml

```yaml
tasks:
  - name: Copy nginx config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify:
      - Restart nginx
      - Reload nginx

handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
  
  - name: Reload nginx
    service:
      name: nginx
      state: reloaded
```

**Условия (when):**

yaml

```yaml
tasks:
  - name: Install Apache (Debian)
    apt:
      name: apache2
      state: present
    when: ansible_os_family == "Debian"
  
  - name: Install Apache (RedHat)
    yum:
      name: httpd
      state: present
    when: ansible_os_family == "RedHat"
  
  - name: Start service if enabled
    service:
      name: nginx
      state: started
    when: 
      - nginx_enabled | default(false)
      - ansible_distribution == "Ubuntu"
```

**Циклы (loop):**

yaml

```yaml
tasks:
  # Простой цикл
  - name: Install packages
    apt:
      name: "{{ item }}"
      state: present
    loop:
      - nginx
      - vim
      - git
  
  # Цикл со словарями
  - name: Create users
    user:
      name: "{{ item.name }}"
      state: present
      groups: "{{ item.groups }}"
    loop:
      - { name: 'alice', groups: 'wheel' }
      - { name: 'bob', groups: 'users' }
  
  # Цикл с переменной
  - name: Install from variable
    apt:
      name: "{{ item }}"
      state: present
    loop: "{{ packages }}"
```

**Переменные:**

yaml

```yaml
# В playbook
vars:
  http_port: 80
  server_name: example.com

# Из файла
vars_files:
  - vars/main.yml
  - vars/{{ ansible_distribution }}.yml

# Extra vars (highest priority)
# ansible-playbook playbook.yml -e "http_port=8080"

# Facts (автоматически собираются)
- debug:
    msg: "OS is {{ ansible_distribution }} {{ ansible_distribution_version }}"

# Registered variables
- name: Check if file exists
  stat:
    path: /etc/nginx/nginx.conf
  register: nginx_config

- name: Print result
  debug:
    var: nginx_config.stat.exists
```

**Templates (Jinja2):**

jinja

```jinja
{# nginx.conf.j2 #}
server {
    listen {{ http_port }};
    server_name {{ server_name }};
    
    {% if ssl_enabled %}
    ssl_certificate {{ ssl_cert_path }};
    ssl_certificate_key {{ ssl_key_path }};
    {% endif %}
    
    {% for location in locations %}
    location {{ location.path }} {
        proxy_pass {{ location.backend }};
    }
    {% endfor %}
}
```

**Блоки и обработка ошибок:**

yaml

```yaml
- name: Handle errors
  block:
    - name: Task that might fail
      command: /usr/bin/might_fail
    
    - name: This runs if above succeeds
      debug:
        msg: "Success!"
  
  rescue:
    - name: This runs on failure
      debug:
        msg: "Something failed"
  
  always:
    - name: This always runs
      debug:
        msg: "Cleanup"
```

**Tags (для выборочного выполнения):**

yaml

```yaml
tasks:
  - name: Install nginx
    apt:
      name: nginx
    tags:
      - packages
      - nginx
  
  - name: Configure nginx
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    tags:
      - config
      - nginx

# Запуск только определенных тегов
# ansible-playbook playbook.yml --tags "config"
# ansible-playbook playbook.yml --skip-tags "packages"
```

### 💻 Задание

Создай функциональный playbook для настройки веб-сервера:

1. **Создай структуру проекта:**

bash

```bash
mkdir -p playbooks/templates playbooks/files
cd playbooks
```

2. **Базовый playbook для веб-сервера:**

yaml

```yaml
# playbooks/webserver.yml
---
- name: Setup web server
  hosts: webservers
  become: yes
  
  vars:
    http_port: 8080
    server_name: "{{ ansible_hostname }}.local"
    app_user: webadmin
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
      tags: packages
    
    - name: Install required packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - python3-pip
        - git
      tags: packages
    
    - name: Create app user
      user:
        name: "{{ app_user }}"
        state: present
        shell: /bin/bash
        create_home: yes
      tags: users
    
    - name: Create web directory
      file:
        path: /var/www/html
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: '0755'
      tags: files
    
    - name: Copy index.html
      copy:
        content: |
          <html>
          <head><title>{{ server_name }}</title></head>
          <body>
            <h1>Welcome to {{ server_name }}</h1>
            <p>Served by {{ ansible_hostname }}</p>
            <p>IP: {{ ansible_default_ipv4.address }}</p>
          </body>
          </html>
        dest: /var/www/html/index.html
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
      tags: files
    
    - name: Configure nginx
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
        owner: root
        group: root
        mode: '0644'
      notify: Restart nginx
      tags: config
    
    - name: Ensure nginx is started
      service:
        name: nginx
        state: started
        enabled: yes
      tags: service
    
    - name: Check nginx status
      command: systemctl status nginx
      register: nginx_status
      changed_when: false
      tags: check
    
    - name: Display nginx status
      debug:
        var: nginx_status.stdout_lines
      tags: check
  
  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

3. **Создай template для nginx:**

jinja

```jinja
{# playbooks/templates/nginx.conf.j2 #}
server {
    listen {{ http_port }} default_server;
    listen [::]:{{ http_port }} default_server;
    
    root /var/www/html;
    index index.html index.htm;
    
    server_name {{ server_name }};
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    # Generated by Ansible on {{ ansible_date_time.iso8601 }}
    # Host: {{ ansible_hostname }}
    # Distribution: {{ ansible_distribution }} {{ ansible_distribution_version }}
}
```

4. **Запусти playbook:**

bash

```bash
# Синтаксис проверка
ansible-playbook webserver.yml --syntax-check

# Dry-run
ansible-playbook webserver.yml --check

# Реальный запуск
ansible-playbook webserver.yml

# С verbose выводом
ansible-playbook webserver.yml -v

# Только определенные теги
ansible-playbook webserver.yml --tags "config"

# Пропустить теги
ansible-playbook webserver.yml --skip-tags "packages"
```

5. **Создай playbook с обработкой ошибок:**

yaml

```yaml
# playbooks/with_error_handling.yml
---
- name: Playbook with error handling
  hosts: webservers
  become: yes
  
  tasks:
    - name: Try to execute block
      block:
        - name: Check if service exists
          command: systemctl status nginx
          register: service_status
          changed_when: false
          failed_when: false
        
        - name: Print service status
          debug:
            msg: "Nginx is {{ 'running' if service_status.rc == 0 else 'not running' }}"
        
        - name: Fail if nginx not running
          fail:
            msg: "Nginx is not running!"
          when: service_status.rc != 0
      
      rescue:
        - name: Install nginx if not present
          apt:
            name: nginx
            state: present
        
        - name: Start nginx
          service:
            name: nginx
            state: started
      
      always:
        - name: Final check
          debug:
            msg: "Block execution completed"
```

6. **Playbook с переменными и условиями:**

yaml

```yaml
# playbooks/conditional.yml
---
- name: Conditional playbook
  hosts: all
  become: yes
  
  vars:
    install_nginx: true
    install_apache: false
    custom_packages:
      - vim
      - curl
      - wget
  
  tasks:
    - name: Install nginx (if enabled)
      apt:
        name: nginx
        state: present
      when: 
        - install_nginx
        - ansible_os_family == "Debian"
    
    - name: Install apache (if enabled)
      apt:
        name: apache2
        state: present
      when:
        - install_apache
        - ansible_os_family == "Debian"
    
    - name: Install custom packages
      apt:
        name: "{{ item }}"
        state: present
      loop: "{{ custom_packages }}"
    
    - name: Different actions per distribution
      debug:
        msg: "This is {{ ansible_distribution }}"
    
    - name: Ubuntu specific task
      debug:
        msg: "Running on Ubuntu"
      when: ansible_distribution == "Ubuntu"
    
    - name: Check memory and warn if low
      debug:
        msg: "WARNING: Low memory! Only {{ ansible_memtotal_mb }}MB"
      when: ansible_memtotal_mb | int < 1024
```

### 🚀 Бонус (новое)

**1. Используй роли для переиспользуемости:**

bash

```bash
# Создай структуру роли
ansible-galaxy init roles/nginx

# Структура будет:
# roles/nginx/
# ├── defaults/      # Переменные по умолчанию
# ├── files/         # Статические файлы
# ├── handlers/      # Обработчики
# ├── tasks/         # Задачи
# ├── templates/     # Шаблоны
# ├── tests/         # Тесты
# └── vars/          # Переменные

# roles/nginx/tasks/main.yml
---
- name: Install nginx
  apt:
    name: nginx
    state: present

- name: Configure nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

# roles/nginx/handlers/main.yml
---
- name: restart nginx
  service:
    name: nginx
    state: restarted

# Используй роль в playbook
---
- name: Setup servers
  hosts: webservers
  become: yes
  
  roles:
    - nginx
    - { role: mysql, when: install_database }
```

**2. Ansible-lint для проверки качества:**

bash

```bash
# Установи ansible-lint
pip install ansible-lint

# Проверь playbook
ansible-lint webserver.yml

# С игнорированием определенных правил
ansible-lint -x 301,303 webserver.yml

# Создай .ansible-lint конфиг
cat > .ansible-lint <<EOF
skip_list:
  - '301'  # Commands should not change things
  - '303'  # Using command rather than module
warn_list:
  - experimental
EOF
```

**3. Используй ansible-playbook с callback plugins:**

bash

````bash
# Установи ara (Ansible Run Analysis)
pip install ara

# Настрой в ansible.cfg
cat >> ansible.cfg <<EOF
[defaults]
callback_plugins = $(python3 -m ara.setup.callback_plugins)
EOF

# Запусти playbook
ansible-playbook webserver.yml

# Просмотр результатов
ara playbook list
ara playbook show <id>

# Web интерфейс
ara-manage runserver
# Открой http://localhost:8000
````

---

## Модуль 3: Роли и Galaxy (25 минут)

### 🎯 Напоминалка

**Структура роли:**
```
roles/
└── роль_name/
    ├── defaults/
    │   └── main.yml      # Переменные по умолчанию (lowest priority)
    ├── vars/
    │   └── main.yml      # Переменные роли (higher priority)
    ├── tasks/
    │   └── main.yml      # Основные задачи
    ├── files/
    │   └── file.txt      # Статические файлы для copy
    ├── templates/
    │   └── config.j2     # Jinja2 шаблоны
    ├── handlers/
    │   └── main.yml      # Обработчики событий
    ├── meta/
    │   └── main.yml      # Метаданные и зависимости
    ├── tests/
    │   ├── inventory     # Тестовый inventory
    │   └── test.yml      # Тестовый playbook
    └── README.md         # Документация
````

**Создание роли:**

bash

```bash
# Автоматическое создание структуры
ansible-galaxy init роль_name

# Создать в определенной директории
ansible-galaxy role init --init-path roles/ nginx

# С force (перезаписать существующую)
ansible-galaxy init nginx --force
```

**Использование ролей в playbook:**

yaml

```yaml
---
- name: Configure servers
  hosts: all
  become: yes
  
  roles:
    # Простое использование
    - common
    - nginx
    
    # С переменными
    - role: nginx
      vars:
        http_port: 8080
        server_name: custom.example.com
    
    # С условием
    - role: mysql
      when: inventory_hostname in groups['databases']
    
    # С тегами
    - role: monitoring
      tags: ['monitoring']
```

**Alternative: tasks:**

yaml

```yaml
---
- name: Configure servers
  hosts: all
  become: yes
  
  tasks:
    - name: Include common role
      include_role:
        name: common
    
    - name: Include nginx with vars
      include_role:
        name: nginx
        vars_from: production
```

Продолжи курс

28 дек. 2025 г.

.yml vars: http_port: 8080 when: install_nginx

```
- name: Import role statically
  import_role:
    name: security
```

````

**Ansible Galaxy:**
```bash
# Поиск ролей
ansible-galaxy search nginx
ansible-galaxy search elasticsearch --platforms EL

# Информация о роли
ansible-galaxy info geerlingguy.nginx

# Установка роли
ansible-galaxy install geerlingguy.nginx

# В определенную директорию
ansible-galaxy install geerlingguy.nginx -p ./roles

# Установка конкретной версии
ansible-galaxy install geerlingguy.nginx,2.8.0

# Из requirements.yml
ansible-galaxy install -r requirements.yml

# Список установленных
ansible-galaxy list

# Удаление роли
ansible-galaxy remove geerlingguy.nginx
```

**requirements.yml:**
```yaml
# requirements.yml

# Из Galaxy
- name: geerlingguy.nginx
  version: 2.8.0

- name: geerlingguy.mysql
  version: 4.3.0

# Из Git
- name: custom-role
  src: https://github.com/user/custom-role.git
  version: main

- name: monitoring
  src: git+https://gitlab.com/company/monitoring-role.git
  scm: git
  version: v1.2.3

# Локальная роль
- src: ../local-roles/my-role
  name: my-role
```

**meta/main.yml (зависимости и метаданные):**
```yaml
# roles/nginx/meta/main.yml
---
galaxy_info:
  role_name: nginx
  author: Your Name
  description: Install and configure nginx
  company: Your Company
  license: MIT
  min_ansible_version: 2.12
  
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
    - name: Debian
      versions:
        - bullseye
  
  galaxy_tags:
    - web
    - nginx
    - webserver

dependencies:
  - role: common
    vars:
      install_basics: true
  
  - role: firewall
    when: configure_firewall
```

**defaults/main.yml:**
```yaml
# roles/nginx/defaults/main.yml
---
# Nginx settings
nginx_version: latest
nginx_http_port: 80
nginx_https_port: 443
nginx_user: www-data
nginx_worker_processes: auto
nginx_worker_connections: 1024

# SSL settings
nginx_ssl_enabled: false
nginx_ssl_cert_path: /etc/ssl/certs/server.crt
nginx_ssl_key_path: /etc/ssl/private/server.key

# Virtual hosts
nginx_vhosts: []
# Example:
# - server_name: example.com
#   root: /var/www/example
#   index: index.html

# Modules
nginx_modules_enabled:
  - ssl
  - gzip
```

**tasks/main.yml (с includes):**
```yaml
# roles/nginx/tasks/main.yml
---
- name: Include OS-specific variables
  include_vars: "{{ ansible_os_family }}.yml"

- name: Include installation tasks
  include_tasks: install.yml
  tags: install

- name: Include configuration tasks
  include_tasks: configure.yml
  tags: configure

- name: Include SSL setup
  include_tasks: ssl.yml
  when: nginx_ssl_enabled
  tags: ssl
```

**handlers/main.yml:**
```yaml
# roles/nginx/handlers/main.yml
---
- name: restart nginx
  service:
    name: nginx
    state: restarted

- name: reload nginx
  service:
    name: nginx
    state: reloaded

- name: validate nginx config
  command: nginx -t
  changed_when: false
```

### 💻 Задание

Создай реюзабельную роль для настройки веб-приложения:

1. **Создай роль webapp:**
```bash
mkdir -p roles
cd roles
ansible-galaxy init webapp
cd ..
```

2. **roles/webapp/defaults/main.yml:**
```yaml
---
# Application settings
app_name: myapp
app_user: "{{ app_name }}"
app_group: "{{ app_name }}"
app_port: 5000
app_dir: "/opt/{{ app_name }}"

# Python settings
python_version: "3.10"
virtualenv_path: "{{ app_dir }}/venv"

# Dependencies
system_packages:
  - python3
  - python3-pip
  - python3-venv
  - git

pip_packages:
  - flask
  - gunicorn
  - requests

# Service settings
service_enabled: true
service_state: started
```

3. **roles/webapp/tasks/main.yml:**
```yaml
---
- name: Create application user
  user:
    name: "{{ app_user }}"
    group: "{{ app_group }}"
    shell: /bin/bash
    create_home: yes
    home: "{{ app_dir }}"
  tags: users

- name: Install system packages
  apt:
    name: "{{ system_packages }}"
    state: present
    update_cache: yes
  tags: packages

- name: Create application directory
  file:
    path: "{{ app_dir }}"
    state: directory
    owner: "{{ app_user }}"
    group: "{{ app_group }}"
    mode: '0755'
  tags: setup

- name: Create Python virtual environment
  command: "python3 -m venv {{ virtualenv_path }}"
  args:
    creates: "{{ virtualenv_path }}/bin/activate"
  become_user: "{{ app_user }}"
  tags: setup

- name: Install Python packages
  pip:
    name: "{{ pip_packages }}"
    virtualenv: "{{ virtualenv_path }}"
  become_user: "{{ app_user }}"
  tags: packages

- name: Create simple Flask app
  copy:
    dest: "{{ app_dir }}/app.py"
    owner: "{{ app_user }}"
    group: "{{ app_group }}"
    mode: '0644'
    content: |
      from flask import Flask
      import socket
      
      app = Flask(__name__)
      
      @app.route('/')
      def hello():
          return f'''
          <html>
            <head><title>{{ app_name }}</title></head>
            <body>
              <h1>{{ app_name }} is running!</h1>
              <p>Hostname: {socket.gethostname()}</p>
              <p>Port: {{ app_port }}</p>
            </body>
          </html>
          '''
      
      if __name__ == '__main__':
          app.run(host='0.0.0.0', port={{ app_port }})
  notify: restart webapp
  tags: deploy

- name: Create systemd service
  template:
    src: webapp.service.j2
    dest: "/etc/systemd/system/{{ app_name }}.service"
    mode: '0644'
  notify:
    - reload systemd
    - restart webapp
  tags: service

- name: Ensure service is {{ service_state }}
  service:
    name: "{{ app_name }}"
    state: "{{ service_state }}"
    enabled: "{{ service_enabled }}"
  tags: service
```

4. **roles/webapp/templates/webapp.service.j2:**
```jinja
[Unit]
Description={{ app_name }} Web Application
After=network.target

[Service]
Type=simple
User={{ app_user }}
Group={{ app_group }}
WorkingDirectory={{ app_dir }}
Environment="PATH={{ virtualenv_path }}/bin"
ExecStart={{ virtualenv_path }}/bin/python {{ app_dir }}/app.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

5. **roles/webapp/handlers/main.yml:**
```yaml
---
- name: reload systemd
  systemd:
    daemon_reload: yes

- name: restart webapp
  service:
    name: "{{ app_name }}"
    state: restarted

- name: stop webapp
  service:
    name: "{{ app_name }}"
    state: stopped
```

6. **roles/webapp/meta/main.yml:**
```yaml
---
galaxy_info:
  role_name: webapp
  author: DevOps Team
  description: Deploy Python Flask web application
  min_ansible_version: 2.12
  
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy

dependencies: []
```

7. **Создай playbook используя роль:**
```yaml
# playbooks/deploy_webapp.yml
---
- name: Deploy web application
  hosts: webservers
  become: yes
  
  roles:
    - role: webapp
      vars:
        app_name: mywebapp
        app_port: 5000
        pip_packages:
          - flask==2.3.0
          - gunicorn==21.2.0
```

8. **Запусти:**
```bash
ansible-playbook playbooks/deploy_webapp.yml

# Проверь что приложение работает
ansible webservers -m uri -a "url=http://localhost:5000 return_content=yes"

# Или через curl напрямую
curl http://localhost:5000  # через проброшенный порт
```

### 🚀 Бонус (новое)

**1. Создай multi-role playbook с зависимостями:**
```bash
# Создай несколько ролей
ansible-galaxy init roles/common
ansible-galaxy init roles/security
ansible-galaxy init roles/monitoring
```
```yaml
# roles/common/meta/main.yml
dependencies: []

# roles/security/meta/main.yml
dependencies:
  - role: common

# roles/monitoring/meta/main.yml
dependencies:
  - role: common
  - role: security

# playbooks/full_setup.yml
---
- name: Full server setup
  hosts: all
  become: yes
  
  roles:
    - common      # Выполнится первым
    - security    # Выполнит common если еще не выполнялась
    - monitoring  # Выполнит common и security если еще не выполнялись
```

**2. Используй Galaxy коллекции:**
```bash
# Установи коллекцию
ansible-galaxy collection install community.general

# В requirements.yml
---
collections:
  - name: community.general
    version: ">=5.0.0"
  - name: ansible.posix
  - name: community.docker

roles:
  - name: geerlingguy.nginx
  - name: geerlingguy.mysql

# Установка всего
ansible-galaxy install -r requirements.yml
```
```yaml
# Использование модулей из коллекции
---
- name: Use collection modules
  hosts: all
  
  tasks:
    - name: Add user to docker group
      community.general.user:
        name: myuser
        groups: docker
        append: yes
    
    - name: Install docker
      ansible.builtin.include_role:
        name: community.docker.docker
```

**3. Создай свою коллекцию:**
```bash
# Создай структуру коллекции
ansible-galaxy collection init mycompany.devops

# Структура:
# mycompany/devops/
# ├── docs/
# ├── plugins/
# │   ├── modules/
# │   └── inventory/
# ├── roles/
# └── galaxy.yml

# Собери коллекцию
ansible-galaxy collection build

# Установи локально
ansible-galaxy collection install mycompany-devops-1.0.0.tar.gz
```

---

Продолжить с модулем 4?
````

Продолжай

28 дек. 2025 г.

## Модуль 4: Переменные, Facts и Jinja2 Templates (30 минут)

### 🎯 Напоминалка

**Приоритет переменных (от низкого к высокому):**

```
1. role defaults (roles/role_name/defaults/main.yml)
2. inventory file or script group vars
3. inventory group_vars/all
4. inventory group_vars/*
5. playbook group_vars/all
6. playbook group_vars/*
7. inventory file or script host vars
8. inventory host_vars/*
9. playbook host_vars/*
10. host facts
11. play vars
12. play vars_prompt
13. play vars_files
14. role vars (roles/role_name/vars/main.yml)
15. block vars
16. task vars
17. include_vars
18. set_facts
19. registered vars
20. extra vars (ansible-playbook -e "var=value")
```

**Определение переменных:**

yaml

```yaml
# В playbook
---
- hosts: webservers
  vars:
    http_port: 80
    server_name: example.com
  
  vars_files:
    - vars/external_vars.yml
  
  vars_prompt:
    - name: username
      prompt: "Enter username"
      private: no
    
    - name: password
      prompt: "Enter password"
      private: yes

# В inventory (INI)
[webservers]
web1 http_port=80 server_name=web1.example.com
web2 http_port=8080 server_name=web2.example.com

[webservers:vars]
nginx_version=1.18

# В inventory (YAML)
all:
  children:
    webservers:
      hosts:
        web1:
          http_port: 80
      vars:
        nginx_version: 1.18
```

**group_vars и host_vars:**

bash

```bash
# Структура
inventory/
├── production
group_vars/
├── all.yml              # Для всех хостов
├── webservers.yml       # Для группы webservers
└── databases.yml        # Для группы databases
host_vars/
├── web1.example.com.yml # Для конкретного хоста
└── db1.example.com.yml
```

yaml

```yaml
# group_vars/all.yml
---
ntp_servers:
  - 0.pool.ntp.org
  - 1.pool.ntp.org
timezone: UTC
admin_email: admin@example.com

# group_vars/webservers.yml
---
http_port: 80
https_port: 443
max_clients: 200

# host_vars/web1.example.com.yml
---
server_id: 1
backup_enabled: true
```

**Ansible Facts (автоматически собираемая информация):**

yaml

```yaml
# Основные facts
ansible_hostname           # Hostname
ansible_fqdn              # FQDN
ansible_distribution      # Ubuntu, CentOS, etc
ansible_distribution_version  # 20.04, 8, etc
ansible_os_family         # Debian, RedHat, etc
ansible_architecture      # x86_64, aarch64, etc
ansible_processor_cores   # Количество ядер
ansible_memtotal_mb       # RAM в MB
ansible_default_ipv4.address  # IP адрес

# Использование
- debug:
    msg: "Host {{ ansible_hostname }} has {{ ansible_processor_cores }} cores"

# Отключить сбор facts (быстрее)
- hosts: all
  gather_facts: no

# Собрать только нужные facts
- hosts: all
  gather_facts: yes
  gather_subset:
    - '!all'
    - '!min'
    - network
```

**Custom Facts:**

bash

```bash
# На управляемом хосте создай
mkdir -p /etc/ansible/facts.d

# Создай custom fact (executable или .fact файл)
cat > /etc/ansible/facts.d/custom.fact <<EOF
#!/bin/bash
echo '{"app_version": "1.2.3", "environment": "production"}'
EOF
chmod +x /etc/ansible/facts.d/custom.fact

# Или JSON файл
cat > /etc/ansible/facts.d/app.fact <<EOF
{
  "version": "2.0.0",
  "deployed_by": "ansible"
}
EOF
```

yaml

```yaml
# Использование custom facts
- hosts: all
  tasks:
    - name: Display custom fact
      debug:
        msg: "App version: {{ ansible_local.custom.app_version }}"
```

**Jinja2 Templates - базовый синтаксис:**

jinja

```jinja
{# Комментарии #}

{# Переменные #}
{{ variable }}
{{ variable | default('default_value') }}
{{ variable | upper }}

{# Условия #}
{% if condition %}
    something
{% elif other_condition %}
    something else
{% else %}
    default
{% endif %}

{# Циклы #}
{% for item in list %}
    {{ item }}
{% endfor %}

{% for key, value in dict.items() %}
    {{ key }}: {{ value }}
{% endfor %}

{# Фильтры #}
{{ variable | upper }}
{{ variable | lower }}
{{ variable | capitalize }}
{{ list | join(', ') }}
{{ number | int }}
{{ string | length }}
{{ list | first }}
{{ list | last }}
{{ dict | to_json }}
{{ dict | to_yaml }}
```

**Полезные Jinja2 фильтры:**

jinja

```jinja
{# Строки #}
{{ 'hello' | upper }}                    # HELLO
{{ 'HELLO' | lower }}                    # hello
{{ 'hello world' | capitalize }}         # Hello world
{{ 'hello world' | title }}              # Hello World
{{ '  spaces  ' | trim }}                # spaces
{{ 'hello' | length }}                   # 5

{# Списки #}
{{ [1,2,3] | join(',') }}               # 1,2,3
{{ [1,2,3] | first }}                   # 1
{{ [1,2,3] | last }}                    # 3
{{ [1,2,3] | length }}                  # 3
{{ [1,2,3] | min }}                     # 1
{{ [1,2,3] | max }}                     # 3
{{ [1,2,3,2,1] | unique }}             # [1,2,3]
{{ [1,2,3] | reverse }}                # [3,2,1]

{# Словари #}
{{ dict | to_json }}                    # JSON
{{ dict | to_yaml }}                    # YAML
{{ dict | to_nice_json }}               # Pretty JSON
{{ dict.keys() | list }}                # Ключи
{{ dict.values() | list }}              # Значения

{# Числа #}
{{ 3.14159 | round }}                   # 3.0
{{ 3.14159 | round(2) }}                # 3.14
{{ '10' | int }}                        # 10
{{ 10.5 | int }}                        # 10

{# Условные #}
{{ variable | default('default') }}
{{ variable | default(omit) }}          # Не передавать если не задано
{{ variable | mandatory }}              # Ошибка если не задано

{# Регулярки #}
{{ string | regex_search('pattern') }}
{{ string | regex_replace('old', 'new') }}

{# IP адреса #}
{{ '192.168.1.0/24' | ipaddr }}
{{ '192.168.1.1' | ipaddr('address') }}
{{ '192.168.1.0/24' | ipaddr('network') }}
```

**Template пример (nginx.conf.j2):**

jinja

```jinja
# Generated by Ansible on {{ ansible_date_time.iso8601 }}
# Hostname: {{ ansible_hostname }}

user {{ nginx_user }};
worker_processes {{ nginx_worker_processes }};
pid /run/nginx.pid;

events {
    worker_connections {{ nginx_worker_connections }};
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    
    {% if nginx_gzip_enabled %}
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    {% endif %}
    
    {% for vhost in nginx_vhosts %}
    server {
        listen {{ vhost.port | default(80) }};
        server_name {{ vhost.server_name }};
        root {{ vhost.root }};
        
        {% if vhost.ssl_enabled | default(false) %}
        ssl_certificate {{ vhost.ssl_cert }};
        ssl_certificate_key {{ vhost.ssl_key }};
        {% endif %}
        
        location / {
            try_files $uri $uri/ =404;
        }
        
        {% if vhost.locations is defined %}
        {% for location in vhost.locations %}
        location {{ location.path }} {
            {{ location.config }}
        }
        {% endfor %}
        {% endif %}
    }
    {% endfor %}
}
```

**Registered variables:**

yaml

```yaml
- name: Check if file exists
  stat:
    path: /etc/myapp/config.yml
  register: config_file

- name: Print result
  debug:
    var: config_file
    verbosity: 1

- name: Use registered variable
  debug:
    msg: "Config exists!"
  when: config_file.stat.exists

- name: Get service status
  command: systemctl is-active nginx
  register: nginx_status
  changed_when: false
  failed_when: false

- name: Show status
  debug:
    msg: "Nginx is {{ 'running' if nginx_status.rc == 0 else 'stopped' }}"
```

**Set_fact для создания переменных:**

yaml

```yaml
- name: Set facts
  set_fact:
    custom_var: "value"
    calculated_var: "{{ ansible_memtotal_mb / 1024 }}"
    cacheable: yes  # Сохранить между запусками

- name: Use in next task
  debug:
    msg: "Custom var: {{ custom_var }}"

# Более сложный пример
- name: Build server list
  set_fact:
    server_list: "{{ groups['webservers'] | map('extract', hostvars, 'ansible_default_ipv4') | map(attribute='address') | list }}"

- name: Show servers
  debug:
    var: server_list
```

### 💻 Задание

Создай сложную конфигурацию используя переменные и шаблоны:

1. **Создай структуру с переменными:**

bash

```bash
mkdir -p practice-vars/{group_vars,host_vars,templates}
cd practice-vars
```

2. **group_vars/all.yml:**

yaml

```yaml
---
# Global variables
company_name: "ACME Corp"
admin_email: "admin@acme.com"
timezone: "UTC"

# NTP configuration
ntp_servers:
  - 0.pool.ntp.org
  - 1.pool.ntp.org
  - 2.pool.ntp.org

# Monitoring
monitoring_enabled: true
monitoring_port: 9100

# Security
ssh_port: 22
firewall_enabled: true
fail2ban_enabled: true

# Common packages
common_packages:
  - vim
  - curl
  - wget
  - git
  - htop
  - net-tools
```

3. **group_vars/webservers.yml:**

yaml

```yaml
---
# Web server configuration
web_server: nginx
http_port: 80
https_port: 443

# SSL configuration
ssl_enabled: true
ssl_protocols:
  - TLSv1.2
  - TLSv1.3

# Application settings
app_name: webapp
app_port: 5000
app_workers: 4

# Virtual hosts
vhosts:
  - name: "www.example.com"
    root: "/var/www/example"
    ssl: true
    locations:
      - path: "/api"
        proxy_pass: "http://localhost:5000"
      - path: "/static"
        root: "/var/www/example/static"
  
  - name: "blog.example.com"
    root: "/var/www/blog"
    ssl: false

# Load balancing
backend_servers:
  - { host: "app1.internal", port: 5000, weight: 1 }
  - { host: "app2.internal", port: 5000, weight: 2 }
```

4. **host_vars/web1.yml:**

yaml

```yaml
---
server_id: 1
is_primary: true
max_connections: 1000

# Host-specific overrides
app_workers: 8
```

5. **templates/nginx-vhost.conf.j2:**

jinja

```jinja
# {{ ansible_managed }}
# Virtual host configuration for {{ item.name }}
# Generated on {{ ansible_date_time.iso8601 }}

{% if item.ssl | default(false) %}
# HTTPS Server
server {
    listen {{ https_port }} ssl http2;
    server_name {{ item.name }};
    
    ssl_certificate /etc/ssl/certs/{{ item.name }}.crt;
    ssl_certificate_key /etc/ssl/private/{{ item.name }}.key;
    ssl_protocols {{ ssl_protocols | join(' ') }};
    
    root {{ item.root }};
    index index.html index.htm;
    
    {% if item.locations is defined %}
    {% for location in item.locations %}
    location {{ location.path }} {
        {% if location.proxy_pass is defined %}
        proxy_pass {{ location.proxy_pass }};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        {% endif %}
        {% if location.root is defined %}
        root {{ location.root }};
        {% endif %}
    }
    {% endfor %}
    {% endif %}
    
    access_log /var/log/nginx/{{ item.name }}_access.log;
    error_log /var/log/nginx/{{ item.name }}_error.log;
}

# Redirect HTTP to HTTPS
server {
    listen {{ http_port }};
    server_name {{ item.name }};
    return 301 https://$server_name$request_uri;
}
{% else %}
# HTTP Server
server {
    listen {{ http_port }};
    server_name {{ item.name }};
    
    root {{ item.root }};
    index index.html index.htm;
    
    {% if item.locations is defined %}
    {% for location in item.locations %}
    location {{ location.path }} {
        {% if location.proxy_pass is defined %}
        proxy_pass {{ location.proxy_pass }};
        {% endif %}
    }
    {% endfor %}
    {% endif %}
    
    access_log /var/log/nginx/{{ item.name }}_access.log;
    error_log /var/log/nginx/{{ item.name }}_error.log;
}
{% endif %}
```

6. **templates/monitoring-config.j2:**

jinja

```jinja
# {{ ansible_managed }}
# Monitoring configuration for {{ ansible_hostname }}

[global]
hostname = {{ ansible_hostname }}
fqdn = {{ ansible_fqdn }}
environment = {{ environment | default('production') }}
server_id = {{ server_id | default('unknown') }}

[system]
total_memory_mb = {{ ansible_memtotal_mb }}
cpu_cores = {{ ansible_processor_cores }}
architecture = {{ ansible_architecture }}
os_family = {{ ansible_os_family }}
distribution = {{ ansible_distribution }} {{ ansible_distribution_version }}

[network]
default_ipv4 = {{ ansible_default_ipv4.address | default('N/A') }}
{% if ansible_all_ipv4_addresses is defined %}
all_ipv4_addresses = {{ ansible_all_ipv4_addresses | join(', ') }}
{% endif %}

[monitoring]
enabled = {{ monitoring_enabled | lower }}
port = {{ monitoring_port }}
{% if monitoring_enabled %}
endpoints = 
{% for endpoint in monitoring_endpoints | default([]) %}
    - {{ endpoint }}
{% endfor %}
{% endif %}

[application]
{% if 'webservers' in group_names %}
type = webserver
web_server = {{ web_server }}
http_port = {{ http_port }}
https_port = {{ https_port }}
app_name = {{ app_name }}
app_workers = {{ app_workers }}
{% elif 'databases' in group_names %}
type = database
db_engine = {{ db_engine | default('postgresql') }}
db_port = {{ db_port | default(5432) }}
{% else %}
type = generic
{% endif %}

[facts]
# Custom facts
{% if ansible_local is defined %}
{% for fact_file, fact_data in ansible_local.items() %}
{{ fact_file }} = {{ fact_data | to_json }}
{% endfor %}
{% endif %}
```

7. **playbook-vars.yml:**

yaml

```yaml
---
- name: Deploy configuration with variables and templates
  hosts: webservers
  become: yes
  
  vars:
    environment: staging
    deploy_timestamp: "{{ ansible_date_time.epoch }}"
  
  pre_tasks:
    - name: Display host information
      debug:
        msg: |
          Hostname: {{ ansible_hostname }}
          Distribution: {{ ansible_distribution }} {{ ansible_distribution_version }}
          IP: {{ ansible_default_ipv4.address }}
          Memory: {{ ansible_memtotal_mb }}MB
          CPUs: {{ ansible_processor_cores }}
      tags: info
    
    - name: Set custom facts
      set_fact:
        deployment_id: "{{ ansible_date_time.epoch }}-{{ ansible_hostname }}"
        is_production: "{{ environment == 'production' }}"
        server_role: "{{ 'primary' if is_primary | default(false) else 'secondary' }}"
      tags: always
  
  tasks:
    - name: Create required directories
      file:
        path: "{{ item }}"
        state: directory
        mode: '0755'
      loop:
        - /etc/nginx/sites-available
        - /etc/nginx/sites-enabled
        - /var/www
      tags: setup
    
    - name: Create virtual host directories
      file:
        path: "{{ item.root }}"
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'
      loop: "{{ vhosts }}"
      tags: setup
    
    - name: Deploy virtual host configurations
      template:
        src: templates/nginx-vhost.conf.j2
        dest: "/etc/nginx/sites-available/{{ item.name }}.conf"
        mode: '0644'
      loop: "{{ vhosts }}"
      notify: reload nginx
      tags: config
    
    - name: Enable virtual hosts
      file:
        src: "/etc/nginx/sites-available/{{ item.name }}.conf"
        dest: "/etc/nginx/sites-enabled/{{ item.name }}.conf"
        state: link
      loop: "{{ vhosts }}"
      notify: reload nginx
      tags: config
    
    - name: Deploy monitoring configuration
      template:
        src: templates/monitoring-config.j2
        dest: /etc/monitoring/config.ini
        mode: '0644'
      tags: monitoring
    
    - name: Create deployment info file
      copy:
        dest: /etc/deployment-info.json
        mode: '0644'
        content: |
          {
            "deployment_id": "{{ deployment_id }}",
            "timestamp": "{{ ansible_date_time.iso8601 }}",
            "deployed_by": "{{ lookup('env', 'USER') }}",
            "environment": "{{ environment }}",
            "server_role": "{{ server_role }}",
            "ansible_version": "{{ ansible_version.full }}",
            "hostname": "{{ ansible_hostname }}",
            "variables": {
              "app_name": "{{ app_name }}",
              "app_workers": {{ app_workers }},
              "http_port": {{ http_port }},
              "ssl_enabled": {{ ssl_enabled | lower }}
            }
          }
      tags: info
    
    - name: Display final configuration
      debug:
        msg: |
          Deployment completed!
          ID: {{ deployment_id }}
          Role: {{ server_role }}
          Vhosts: {{ vhosts | length }}
          App workers: {{ app_workers }}
      tags: info
  
  handlers:
    - name: reload nginx
      service:
        name: nginx
        state: reloaded
```

8. **Запусти с разными переменными:**

bash

```bash
# Базовый запуск
ansible-playbook playbook-vars.yml

# С extra vars (переопределят все остальные)
ansible-playbook playbook-vars.yml -e "environment=production app_workers=16"

# С конкретным хостом
ansible-playbook playbook-vars.yml --limit web1

# Только определенные теги
ansible-playbook playbook-vars.yml --tags config

# Dry-run для проверки
ansible-playbook playbook-vars.yml --check --diff
```

9. **Проверка переменных:**

bash

```bash
# Посмотреть все переменные для хоста
ansible web1 -m debug -a "var=hostvars[inventory_hostname]"

# Конкретную переменную
ansible web1 -m debug -a "var=app_workers"

# Проверить откуда берется переменная
ansible-playbook playbook-vars.yml --tags info -vvv
```

### 🚀 Бонус (новое)

**1. Использование lookup plugins:**

yaml

```yaml
---
- name: Advanced lookups
  hosts: localhost
  
  tasks:
    - name: Read file content
      debug:
        msg: "{{ lookup('file', '/etc/hostname') }}"
    
    - name: Get environment variable
      debug:
        msg: "User: {{ lookup('env', 'USER') }}"
    
    - name: Read from URL
      debug:
        msg: "{{ lookup('url', 'https://api.github.com/repos/ansible/ansible') }}"
    
    - name: Generate password
      debug:
        msg: "{{ lookup('password', '/tmp/passwordfile chars=ascii_letters,digits length=16') }}"
    
    - name: Pipe command output
      debug:
        msg: "{{ lookup('pipe', 'date +%Y-%m-%d') }}"
    
    - name: Read CSV
      debug:
        msg: "{{ lookup('csvfile', 'user1 file=users.csv delimiter=,') }}"
    
    - name: Template with lookup
      debug:
        msg: "{{ lookup('template', 'template.j2') }}"
```

**2. Комплексные вычисления в Jinja2:**

yaml

```yaml
# templates/complex-config.j2
{% set total_memory_gb = (ansible_memtotal_mb / 1024) | round(2) %}
{% set memory_per_worker = (total_memory_gb / app_workers) | round(2) %}

# Memory configuration
total_memory = {{ total_memory_gb }}GB
workers = {{ app_workers }}
memory_per_worker = {{ memory_per_worker }}GB

# Calculate optimal settings
{% if total_memory_gb > 16 %}
max_connections = {{ (total_memory_gb * 100) | int }}
buffer_size = large
{% elif total_memory_gb > 8 %}
max_connections = {{ (total_memory_gb * 75) | int }}
buffer_size = medium
{% else %}
max_connections = {{ (total_memory_gb * 50) | int }}
buffer_size = small
{% endif %}

# Dynamic backend configuration
{% for server in backend_servers %}
backend_{{ loop.index }} {
    host = {{ server.host }}
    port = {{ server.port }}
    weight = {{ server.weight }}
    # {{ ((server.weight / (backend_servers | sum(attribute='weight'))) * 100) | round(1) }}% of traffic
}
{% endfor %}

# Conditional includes
{% if ansible_os_family == "Debian" %}
include /etc/myapp/debian.conf
{% elif ansible_os_family == "RedHat" %}
include /etc/myapp/redhat.conf
{% endif %}
```

**3. Использование filters для IP адресов:**

yaml

```yaml
---
- name: IP address manipulation
  hosts: localhost
  vars:
    subnet: "192.168.1.0/24"
    ip_list:
      - 192.168.1.10
      - 192.168.1.20
      - 10.0.0.5
  
  tasks:
    - name: Get network address
      debug:
        msg: "Network: {{ subnet | ipaddr('network') }}"
    
    - name: Get broadcast address
      debug:
        msg: "Broadcast: {{ subnet | ipaddr('broadcast') }}"
    
    - name: Get netmask
      debug:
        msg: "Netmask: {{ subnet | ipaddr('netmask') }}"
    
    - name: Filter IPs in subnet
      debug:
        msg: "{{ ip_list | ipaddr(subnet) }}"
    
    - name: Get host range
      debug:
        msg: "Range: {{ subnet | ipaddr('1') }} - {{ subnet | ipaddr('-2') }}"
```

---

## Модуль 5: Advanced Playbooks и Best Practices (25 минут)

### 🎯 Напоминалка

**Блоки для группировки задач:**

yaml

```yaml
- name: Install and configure
  block:
    - name: Install package
      apt:
        name: nginx
        state: present
    
    - name: Start service
      service:
        name: nginx
        state: started
  
  rescue:
    - name: Handle failure
      debug:
        msg: "Installation failed, rolling back"
    
    - name: Remove package
      apt:
        name: nginx
        state: absent
  
  always:
    - name: Log attempt
      lineinfile:
        path: /var/log/deployment.log
        line: "Deployment attempt at {{ ansible_date_time.iso8601 }}"
        create: yes
  
  when: install_nginx | default(true)
  become: yes
  tags: nginx
```

**Делегирование задач:**

yaml

```yaml
# Выполнить на другом хосте
- name: Add to load balancer
  command: /usr/local/bin/add_to_lb {{ inventory_hostname }}
  delegate_to: loadbalancer.example.com

# Выполнить локально (на control node)
- name: Generate configuration
  template:
    src: config.j2
    dest: /tmp/config.txt
  delegate_to: localhost

# Run once (только на одном хосте из группы)
- name: Create database
  postgresql_db:
    name: myapp
    state: present
  run_once: true
  delegate_to: "{{ groups['databases'][0] }}"
```

**Асинхронные задачи:**

yaml

```yaml
# Запустить задачу асинхронно
- name: Long running task
  command: /usr/bin/long_script.sh
  async: 3600  # Таймаут в секундах
  poll: 0      # Не ждать завершения
  register: long_task

# Проверить статус позже
- name: Check on task
  async_status:
    jid: "{{ long_task.ansible_job_id }}"
  register: job_result
  until: job_result.finished
  retries: 30
  delay: 10

# Или запустить и ждать
- name: Long task with polling
  command: /usr/bin/another_script.sh
  async: 600
  poll: 10  # Проверять каждые 10 секунд
```

**Стратегии выполнения:**

yaml

```yaml
---
- name: Deployment with strategy
  hosts: webservers
  strategy: free  # Варианты: linear (default), free, debug
  serial: 2       # По 2 хоста за раз
  max_fail_percentage: 25  # Остановить если > 25% failed
  
  tasks:
    - name: Deploy application
      # ...
```

yaml

```yaml
# Rolling update pattern
---
- name: Rolling update
  hosts: webservers
  serial: "30%"  # По 30% хостов за раз
  
  pre_tasks:
    - name: Remove from load balancer
      haproxy:
        host: "{{ inventory_hostname }}"
        state: disabled
      delegate_to: loadbalancer
  
  tasks:
    - name: Update application
      # ...
  
  post_tasks:
    - name: Add back to load balancer
      haproxy:
        host: "{{ inventory_hostname }}"
        state: enabled
      delegate_to: loadbalancer
    
    - name: Wait for health check
      uri:
        url: "http://{{ inventory_hostname }}/health"
        status_code: 200
      retries: 5
      delay: 3
```

**Include vs Import:**

yaml

```yaml
# Import (статический, во время parse)
- import_tasks: tasks/setup.yml
- import_playbook: playbooks/common.yml
- import_role:
    name: common

# Include (динамический, во время runtime)
- include_tasks: tasks/{{ ansible_os_family }}.yml
- include_vars: vars/{{ environment }}.yml
- include_role:
    name: "{{ role_name }}"
  when: some_condition
```

**Loops с различными структурами:**

yaml

```yaml
# Простой список
- name: Create users
  user:
    name: "{{ item }}"
  loop:
    - alice
    - bob
    - charlie

# Вложенные списки
- name: Create directories
  file:
    path: "{{ item.0 }}/{{ item.1 }}"
    state: directory
  loop: "{{ ['app1', 'app2'] | product(['logs', 'data']) }}"

# Словари
- name: Configure apps
  template:
    src: "{{ item.key }}.conf.j2"
    dest: "/etc/{{ item.key }}/config.conf"
  loop: "{{ applications | dict2items }}"

# С индексом
- name: Create with index
  debug:
    msg: "Item {{ item.0 }}: {{ item.1 }}"
  loop: "{{ my_list | zip_longest(range(my_list|length)) | list }}"

# Until (retry loop)
- name: Wait for service
  uri:
    url: http://localhost:8080/health
    status_code: 200
  register: result
  until: result.status == 200
  retries: 10
  delay: 5
```

**Работа с ошибками:**

yaml

```yaml
# Игнорировать ошибки
- name: This might fail
  command: /usr/bin/might_fail
  ignore_errors: yes

# Собственное условие ошибки
- name: Check return code
  command: /usr/bin/my_script
  register: result
  failed_when: result.rc != 0 and result.rc != 2

# Изменить когда считать changed
- name: Check status
  command: systemctl is-active nginx
  register: result
  changed_when: false
  failed_when: result.rc > 1

# Any_errors_fatal
---
- hosts: webservers
  any_errors_fatal: true  # Остановить playbook если любая
```

Продолжи созание курса по модулям, допиши модуль 5 и далее начни модуль 6. По завершению модуля 6, я далее дам отдельную инструкцию

29 дек. 2025 г.

ошибка tasks: # ...

````

**Тестирование и проверки:**
```yaml
# Assert для проверок
- name: Validate configuration
  assert:
    that:
      - ansible_memtotal_mb >= 1024
      - ansible_distribution == "Ubuntu"
      - app_version is version('2.0.0', '>=')
    fail_msg: "Server doesn't meet requirements"
    success_msg: "All checks passed"

# Check mode (dry-run)
- name: Task that always runs in check mode
  debug:
    msg: "This runs even with --check"
  check_mode: no

- name: Task that never runs in check mode
  command: /usr/bin/make_changes
  check_mode: yes

# Diff mode
- name: Show changes
  template:
    src: config.j2
    dest: /etc/app/config
  diff: yes
```

**Tags лучшие практики:**
```yaml
tasks:
  - name: Install packages
    apt:
      name: nginx
    tags:
      - packages
      - nginx
      - never  # Никогда не выполнять unless explicitly tagged

  - name: Configure
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    tags:
      - config
      - nginx
      - always  # Всегда выполнять

# Запуск:
# ansible-playbook playbook.yml --tags "nginx"
# ansible-playbook playbook.yml --tags "config,packages"
# ansible-playbook playbook.yml --skip-tags "packages"
# ansible-playbook playbook.yml --tags "never,backup"
```

### 💻 Задание

Создай production-ready playbook с best practices:

1. **Структура проекта:**
```bash
mkdir -p advanced-playbook/{inventories/{production,staging},group_vars,roles,playbooks}
cd advanced-playbook
```

2. **ansible.cfg:**
```ini
[defaults]
inventory = ./inventories/staging
roles_path = ./roles
host_key_checking = False
retry_files_enabled = False
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 3600
interpreter_python = auto_silent
stdout_callback = yaml
callbacks_enabled = profile_tasks, timer

[privilege_escalation]
become = True
become_method = sudo
become_user = root

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=300s
```

3. **playbooks/deploy.yml (production-ready):**
```yaml
---
- name: Pre-deployment checks
  hosts: all
  gather_facts: yes
  tags: always
  
  tasks:
    - name: Validate requirements
      assert:
        that:
          - ansible_version.full is version('2.12.0', '>=')
          - ansible_distribution in ['Ubuntu', 'Debian', 'CentOS', 'Rocky']
        fail_msg: "Requirements not met"
        success_msg: "Pre-checks passed"
    
    - name: Check connectivity
      wait_for_connection:
        timeout: 30
    
    - name: Set deployment facts
      set_fact:
        deployment_id: "{{ ansible_date_time.epoch }}"
        deployment_user: "{{ lookup('env', 'USER') }}"
        deployment_timestamp: "{{ ansible_date_time.iso8601 }}"
        cacheable: yes

- name: Backup current state
  hosts: webservers
  serial: "{{ serial_count | default('100%') }}"
  tags: backup
  
  tasks:
    - name: Create backup directory
      file:
        path: "/backup/{{ deployment_id }}"
        state: directory
        mode: '0755'
      delegate_to: localhost
      run_once: true
    
    - name: Backup configuration files
      block:
        - name: Archive configs
          archive:
            path:
              - /etc/nginx
              - /etc/myapp
            dest: "/tmp/config-backup-{{ inventory_hostname }}.tar.gz"
        
        - name: Fetch backup
          fetch:
            src: "/tmp/config-backup-{{ inventory_hostname }}.tar.gz"
            dest: "/backup/{{ deployment_id }}/"
            flat: yes
      
      rescue:
        - name: Backup failed
          debug:
            msg: "WARNING: Backup failed for {{ inventory_hostname }}"

- name: Deploy application
  hosts: webservers
  serial: "{{ batch_size | default(1) }}"
  max_fail_percentage: 20
  
  pre_tasks:
    - name: Check disk space
      assert:
        that:
          - item.size_available > 1000000000  # 1GB free
        fail_msg: "Insufficient disk space on {{ item.mount }}"
      loop: "{{ ansible_mounts }}"
      when: item.mount == '/'
      tags: preflight
    
    - name: Remove from load balancer
      shell: |
        echo "Removing {{ inventory_hostname }} from LB"
        # actual LB removal command
      delegate_to: localhost
      when: use_load_balancer | default(false)
      tags: lb
  
  tasks:
    - name: Stop application
      service:
        name: myapp
        state: stopped
      register: stop_result
      failed_when: false
      tags: deploy
    
    - name: Wait for application to stop
      wait_for:
        port: "{{ app_port }}"
        state: stopped
        timeout: 30
      when: stop_result is changed
      tags: deploy
    
    - name: Deploy new version
      block:
        - name: Copy application files
          synchronize:
            src: /path/to/app/
            dest: /opt/myapp/
            delete: yes
            rsync_opts:
              - "--exclude=logs"
              - "--exclude=data"
        
        - name: Install dependencies
          pip:
            requirements: /opt/myapp/requirements.txt
            virtualenv: /opt/myapp/venv
        
        - name: Run database migrations
          command: /opt/myapp/venv/bin/python manage.py migrate
          run_once: true
          delegate_to: "{{ groups['webservers'][0] }}"
      
      rescue:
        - name: Deployment failed
          debug:
            msg: "Deployment failed, initiating rollback"
        
        - name: Restore from backup
          unarchive:
            src: "/backup/{{ deployment_id }}/config-backup-{{ inventory_hostname }}.tar.gz"
            dest: /
            remote_src: no
        
        - name: Fail deployment
          fail:
            msg: "Deployment failed and rolled back"
      
      always:
        - name: Ensure application is running
          service:
            name: myapp
            state: started
            enabled: yes
      
      tags: deploy
  
  post_tasks:
    - name: Wait for application
      wait_for:
        port: "{{ app_port }}"
        state: started
        timeout: 60
      tags: healthcheck
    
    - name: Health check
      uri:
        url: "http://localhost:{{ app_port }}/health"
        status_code: 200
      register: health_result
      until: health_result.status == 200
      retries: 10
      delay: 5
      tags: healthcheck
    
    - name: Add back to load balancer
      shell: |
        echo "Adding {{ inventory_hostname }} back to LB"
        # actual LB add command
      delegate_to: localhost
      when: use_load_balancer | default(false)
      tags: lb
    
    - name: Wait after adding to LB
      pause:
        seconds: 10
      when: use_load_balancer | default(false)
      tags: lb

- name: Post-deployment tasks
  hosts: webservers
  run_once: true
  tags: post-deploy
  
  tasks:
    - name: Clear cache
      command: /usr/local/bin/clear_cache.sh
      async: 300
      poll: 0
      register: cache_clear
    
    - name: Record deployment
      lineinfile:
        path: /var/log/deployments.log
        line: "{{ deployment_timestamp }} | {{ deployment_user }} | {{ deployment_id }} | SUCCESS"
        create: yes
      delegate_to: localhost
    
    - name: Send notification
      debug:
        msg: |
          Deployment completed successfully!
          ID: {{ deployment_id }}
          Time: {{ deployment_timestamp }}
          User: {{ deployment_user }}
          Hosts: {{ groups['webservers'] | join(', ') }}

- name: Verify deployment
  hosts: webservers
  tags: verify
  
  tasks:
    - name: Run smoke tests
      script: scripts/smoke_test.sh
      register: smoke_test
      failed_when: smoke_test.rc != 0
    
    - name: Check logs for errors
      shell: |
        tail -100 /var/log/myapp/app.log | grep -i error || true
      register: log_check
      changed_when: false
    
    - name: Report log errors
      debug:
        var: log_check.stdout_lines
      when: log_check.stdout != ""
```

4. **Создай скрипт для удобного запуска:**
```bash
# deploy.sh
#!/bin/bash

set -e

INVENTORY="${INVENTORY:-staging}"
TAGS="${TAGS:-all}"
LIMIT="${LIMIT:-all}"
CHECK_MODE="${CHECK_MODE:-}"

echo "=== Ansible Deployment ==="
echo "Inventory: $INVENTORY"
echo "Tags: $TAGS"
echo "Limit: $LIMIT"
echo "======================="

# Syntax check
echo "Checking syntax..."
ansible-playbook playbooks/deploy.yml --syntax-check

# Lint check (if installed)
if command -v ansible-lint &> /dev/null; then
    echo "Running ansible-lint..."
    ansible-lint playbooks/deploy.yml || true
fi

# Confirm
read -p "Proceed with deployment? (yes/no): " confirm
if [ "$confirm" != "yes" ]; then
    echo "Deployment cancelled"
    exit 0
fi

# Run playbook
ANSIBLE_OPTS="-i inventories/$INVENTORY"
[ -n "$TAGS" ] && [ "$TAGS" != "all" ] && ANSIBLE_OPTS="$ANSIBLE_OPTS --tags $TAGS"
[ -n "$LIMIT" ] && [ "$LIMIT" != "all" ] && ANSIBLE_OPTS="$ANSIBLE_OPTS --limit $LIMIT"
[ -n "$CHECK_MODE" ] && ANSIBLE_OPTS="$ANSIBLE_OPTS --check --diff"

echo "Running: ansible-playbook playbooks/deploy.yml $ANSIBLE_OPTS"
ansible-playbook playbooks/deploy.yml $ANSIBLE_OPTS

echo "=== Deployment Complete ==="
```
```bash
chmod +x deploy.sh

# Использование:
./deploy.sh  # Staging
INVENTORY=production ./deploy.sh  # Production
TAGS=deploy ./deploy.sh  # Только deploy таски
CHECK_MODE=1 ./deploy.sh  # Dry-run
```

### 🚀 Бонус (новое)

**1. Используй mitogen для ускорения:**
```bash
# Установка
pip install mitogen

# ansible.cfg
[defaults]
strategy_plugins = /path/to/mitogen/ansible_mitogen/plugins/strategy
strategy = mitogen_linear

# Или mitogen_free для parallel execution
```

**2. Ansible Test Kitchen для тестирования:**
```bash
# Установка
gem install test-kitchen kitchen-ansible kitchen-docker

# .kitchen.yml
---
driver:
  name: docker

provisioner:
  name: ansible_playbook
  playbook: playbooks/deploy.yml
  ansible_cfg_path: ansible.cfg

platforms:
  - name: ubuntu-22.04
  - name: centos-8

suites:
  - name: default
    provisioner:
      playbook: test/integration/default/default.yml

# Команды
kitchen create   # Создать инстансы
kitchen converge # Применить playbook
kitchen verify   # Запустить тесты
kitchen destroy  # Удалить инстансы
kitchen test     # Все вместе
```

**3. Ansible Molecule для тестирования ролей:**
```bash
# Установка
pip install molecule molecule-docker

# В директории роли
cd roles/myapp
molecule init scenario -d docker

# molecule/default/molecule.yml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: instance
    image: ubuntu:22.04
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: ansible

# Команды
molecule create    # Создать контейнер
molecule converge  # Применить роль
molecule verify    # Запустить верификацию
molecule test      # Полный цикл тестирования
molecule destroy   # Удалить контейнер
```

---

Продолжить с модулем 6?
````

Давай модуль 6

7 янв.

## Модуль 6: Безопасность, Vault и Production Practices (30 минут)

### 🎯 Напоминалка

**Ansible Vault - шифрование секретов:**

bash

```bash
# Создание encrypted файла
ansible-vault create secrets.yml
# Введи пароль и содержимое

# Редактирование
ansible-vault edit secrets.yml

# Просмотр
ansible-vault view secrets.yml

# Шифрование существующего файла
ansible-vault encrypt vars/database.yml

# Расшифровка
ansible-vault decrypt vars/database.yml

# Перешифровка с новым паролем
ansible-vault rekey secrets.yml

# Шифрование строки
ansible-vault encrypt_string 'secret_password' --name 'db_password'
```

**Использование Vault файлов:**

bash

```bash
# С запросом пароля
ansible-playbook playbook.yml --ask-vault-pass

# С файлом пароля
ansible-playbook playbook.yml --vault-password-file .vault_pass

# В ansible.cfg
[defaults]
vault_password_file = ./.vault_pass

# Или скрипт для получения пароля
vault_password_file = ./get_vault_pass.sh
```

**Multiple Vault IDs (Ansible 2.4+):**

bash

```bash
# Создать с vault ID
ansible-vault create --vault-id prod@prompt secrets_prod.yml
ansible-vault create --vault-id dev@prompt secrets_dev.yml

# Использовать
ansible-playbook playbook.yml --vault-id prod@prompt --vault-id dev@prompt

# В ansible.cfg
[defaults]
vault_identity_list = prod@.vault_pass_prod, dev@.vault_pass_dev
```

**Структура vault файлов:**

yaml

```yaml
# group_vars/all/vault.yml (encrypted)
---
vault_db_password: "SuperSecretPassword123!"
vault_api_key: "secret-api-key-xyz"
vault_ssl_key: |
  -----BEGIN PRIVATE KEY-----
  MIIEvQIBADANBgkqhkiG9w0BAQE...
  -----END PRIVATE KEY-----

# group_vars/all/vars.yml (plain)
---
db_password: "{{ vault_db_password }}"
api_key: "{{ vault_api_key }}"
ssl_key: "{{ vault_ssl_key }}"
```

**Inline encrypted strings:**

yaml

```yaml
# vars.yml
---
db_host: localhost
db_user: admin
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          66386439653831303037353831643731336337373232366665323830313564393834373739666338
          6536373633623830393536623831663634376264656662370a626334653239346330376665393735
          61383766633464373733666633326665626565653034336563646136616237643339303838306331
          3434653535653030650a376434383537396338656165616461656661626634323261396434303836
          3564
```

**SSH ключи и безопасность:**

ini

```ini
# ansible.cfg
[defaults]
private_key_file = ~/.ssh/ansible_key
remote_user = ansible

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=300s -o StrictHostKeyChecking=no
pipelining = True
```

**Настройка sudo без пароля:**

bash

```bash
# На управляемых хостах
# /etc/sudoers.d/ansible
ansible ALL=(ALL) NOPASSWD: ALL

# Или с ограничениями
ansible ALL=(ALL) NOPASSWD: /usr/bin/systemctl, /usr/bin/apt-get

# Проверка
sudo -l -U ansible
```

**Безопасное хранение паролей:**

bash

```bash
# Использование внешних систем
# 1. HashiCorp Vault
ansible-playbook playbook.yml -e "@<(vault kv get -format=json secret/ansible | jq .data.data)"

# 2. AWS Secrets Manager
aws secretsmanager get-secret-value --secret-id ansible/prod --query SecretString

# 3. 1Password CLI
op item get "Ansible Prod" --fields password

# 4. pass (password store)
pass show ansible/prod/db_password
```

**Lookup plugins для секретов:**

yaml

```yaml
---
- name: Use external secrets
  hosts: localhost
  
  tasks:
    # HashiCorp Vault
    - name: Get secret from Vault
      debug:
        msg: "{{ lookup('community.hashi_vault.hashi_vault', 'secret=secret/data/ansible:password') }}"
    
    # AWS SSM Parameter Store
    - name: Get from SSM
      debug:
        msg: "{{ lookup('aws_ssm', '/ansible/prod/db_password') }}"
    
    # Environment variable
    - name: Get from env
      debug:
        msg: "{{ lookup('env', 'DB_PASSWORD') }}"
    
    # Password manager
    - name: Get from pass
      debug:
        msg: "{{ lookup('passwordstore', 'ansible/prod/api_key') }}"
```

**Security best practices:**

yaml

```yaml
# 1. Минимальные привилегии
- name: Create service user
  user:
    name: appuser
    shell: /usr/sbin/nologin
    system: yes
    create_home: no

# 2. Файловые права
- name: Secure configuration
  file:
    path: /etc/app/config.yml
    owner: appuser
    group: appuser
    mode: '0600'

# 3. Очистка sensitive data
- name: Remove temporary files
  file:
    path: "{{ item }}"
    state: absent
  loop:
    - /tmp/database_dump.sql
    - /tmp/ssl_keys.tar.gz
  no_log: true

# 4. No_log для sensitive tasks
- name: Set database password
  mysql_user:
    name: dbuser
    password: "{{ db_password }}"
    state: present
  no_log: true

# 5. Проверка SSH fingerprint
- name: Verify SSH host key
  known_hosts:
    name: example.com
    key: "example.com ssh-rsa AAAAB3NzaC1yc2E..."
    state: present
```

**Firewall конфигурация:**

yaml

```yaml
# UFW (Ubuntu)
- name: Configure UFW
  block:
    - name: Install UFW
      apt:
        name: ufw
        state: present
    
    - name: Set default policies
      ufw:
        direction: "{{ item.direction }}"
        policy: "{{ item.policy }}"
      loop:
        - { direction: 'incoming', policy: 'deny' }
        - { direction: 'outgoing', policy: 'allow' }
    
    - name: Allow SSH
      ufw:
        rule: allow
        port: '22'
        proto: tcp
    
    - name: Allow HTTP/HTTPS
      ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop:
        - '80'
        - '443'
    
    - name: Enable UFW
      ufw:
        state: enabled

# firewalld (RHEL/CentOS)
- name: Configure firewalld
  block:
    - name: Install firewalld
      yum:
        name: firewalld
        state: present
    
    - name: Start firewalld
      service:
        name: firewalld
        state: started
        enabled: yes
    
    - name: Allow services
      firewalld:
        service: "{{ item }}"
        permanent: yes
        state: enabled
      loop:
        - http
        - https
        - ssh
      notify: reload firewalld
    
    - name: Allow custom port
      firewalld:
        port: "{{ app_port }}/tcp"
        permanent: yes
        state: enabled
      notify: reload firewalld
```

**SELinux configuration:**

yaml

```yaml
- name: Configure SELinux
  block:
    - name: Set SELinux mode
      selinux:
        policy: targeted
        state: enforcing
    
    - name: Set file contexts
      sefcontext:
        target: '/opt/myapp(/.*)?'
        setype: httpd_sys_content_t
        state: present
    
    - name: Apply contexts
      command: restorecon -Rv /opt/myapp
    
    - name: Allow httpd network connect
      seboolean:
        name: httpd_can_network_connect
        state: yes
        persistent: yes
```

**Fail2ban для защиты:**

yaml

```yaml
- name: Install and configure fail2ban
  block:
    - name: Install fail2ban
      apt:
        name: fail2ban
        state: present
    
    - name: Configure fail2ban
      template:
        src: jail.local.j2
        dest: /etc/fail2ban/jail.local
      notify: restart fail2ban
    
    - name: Start fail2ban
      service:
        name: fail2ban
        state: started
        enabled: yes

# templates/jail.local.j2
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
destemail = {{ admin_email }}
action = %(action_mwl)s

[sshd]
enabled = true
port = {{ ssh_port }}
logpath = /var/log/auth.log

[nginx-limit-req]
enabled = true
filter = nginx-limit-req
logpath = /var/log/nginx/error.log
```

**Audit logging:**

yaml

```yaml
- name: Setup audit logging
  block:
    - name: Install auditd
      package:
        name: "{{ 'auditd' if ansible_os_family == 'RedHat' else 'auditd' }}"
        state: present
    
    - name: Configure audit rules
      lineinfile:
        path: /etc/audit/rules.d/ansible.rules
        line: "{{ item }}"
        create: yes
      loop:
        - "-w /etc/passwd -p wa -k identity"
        - "-w /etc/sudoers -p wa -k sudoers"
        - "-w /var/log/ansible -p wa -k ansible_logs"
      notify: restart auditd
```

### 💻 Задание

Создай безопасный playbook с использованием Vault и best practices:

1. **Создай структуру:**

bash

```bash
mkdir -p secure-playbook/{group_vars/all,roles/secure_server/{tasks,templates,handlers}}
cd secure-playbook
```

2. **Создай vault файл с секретами:**

bash

```bash
# Создай файл пароля
echo "VaultPassword123" > .vault_pass
chmod 600 .vault_pass

# Создай encrypted файл
ansible-vault create group_vars/all/vault.yml --vault-password-file .vault_pass
```

yaml

```yaml
# Содержимое group_vars/all/vault.yml
---
# Database credentials
vault_db_root_password: "RootPass123!@#"
vault_db_app_password: "AppPass456!@#"
vault_db_backup_password: "BackupPass789!@#"

# API keys
vault_api_key_github: "ghp_xxxxxxxxxxxxxxxxxxxx"
vault_api_key_slack: "xoxb-xxxxxxxxxxxx"

# SSL certificates
vault_ssl_cert: |
  -----BEGIN CERTIFICATE-----
  MIIDXTCCAkWgAwIBAgIJAKL0UG+mRKF0MA0GCSqGSIb3DQEBCwUAMEUxCzAJBgNV
  ... (dummy cert)
  -----END CERTIFICATE-----

vault_ssl_key: |
  -----BEGIN PRIVATE KEY-----
  MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQDXYZ/hGPvqHEKx
  ... (dummy key)
  -----END PRIVATE KEY-----

# SSH keys
vault_deploy_ssh_key: |
  -----BEGIN OPENSSH PRIVATE KEY-----
  b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdz
  ... (dummy key)
  -----END OPENSSH PRIVATE KEY-----

# Encrypted strings inline
vault_admin_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          63383962633464626164316530613332643739393138323765613937373634383935303165373166
          3662306566663032646638303066323937613738643638310a373135386666303162333766323536
          35663061353962636530323061633437396232636266336263396466663665666436376363386635
          3365373538343837660a316662623762336264353931306264373062306138343263356634303264
          6232
```

3. **Создай plain vars файл:**

yaml

```yaml
# group_vars/all/vars.yml
---
# Reference vault variables
db_root_password: "{{ vault_db_root_password }}"
db_app_password: "{{ vault_db_app_password }}"
db_backup_password: "{{ vault_db_backup_password }}"

api_key_github: "{{ vault_api_key_github }}"
api_key_slack: "{{ vault_api_key_slack }}"

ssl_cert_content: "{{ vault_ssl_cert }}"
ssl_key_content: "{{ vault_ssl_key }}"

deploy_ssh_key: "{{ vault_deploy_ssh_key }}"
admin_password: "{{ vault_admin_password }}"

# Non-sensitive variables
app_name: myapp
app_user: appuser
app_port: 5000

# Security settings
ssh_port: 22
firewall_enabled: true
fail2ban_enabled: true
selinux_enabled: false  # Set to true for RHEL/CentOS

# Allowed IPs
allowed_ips:
  - 192.168.1.0/24
  - 10.0.0.0/8

# Admin users
admin_users:
  - name: admin
    ssh_key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC..."
  - name: deploy
    ssh_key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD..."
```

4. **ansible.cfg:**

ini

```ini
[defaults]
inventory = ./inventory.ini
vault_password_file = ./.vault_pass
host_key_checking = False
retry_files_enabled = False
roles_path = ./roles
nocows = 1

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=300s
```

5. **roles/secure_server/tasks/main.yml:**

yaml

```yaml
---
- name: Include OS-specific variables
  include_vars: "{{ ansible_os_family }}.yml"
  tags: always

- name: Update system
  apt:
    update_cache: yes
    upgrade: dist
    autoremove: yes
  when: ansible_os_family == "Debian"
  tags: system

- name: Install security packages
  package:
    name: "{{ security_packages }}"
    state: present
  tags: packages

- name: Create admin users
  user:
    name: "{{ item.name }}"
    shell: /bin/bash
    groups: sudo
    append: yes
    create_home: yes
  loop: "{{ admin_users }}"
  no_log: true
  tags: users

- name: Add SSH keys for admin users
  authorized_key:
    user: "{{ item.name }}"
    key: "{{ item.ssh_key }}"
    state: present
  loop: "{{ admin_users }}"
  no_log: true
  tags: users

- name: Create application user
  user:
    name: "{{ app_user }}"
    shell: /usr/sbin/nologin
    system: yes
    create_home: no
  tags: users

- name: Configure SSH
  template:
    src: sshd_config.j2
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: '0600'
    validate: '/usr/sbin/sshd -t -f %s'
  notify: restart ssh
  tags: ssh

- name: Configure firewall
  include_tasks: firewall.yml
  when: firewall_enabled
  tags: firewall

- name: Configure fail2ban
  include_tasks: fail2ban.yml
  when: fail2ban_enabled
  tags: fail2ban

- name: Deploy SSL certificates
  block:
    - name: Create SSL directory
      file:
        path: /etc/ssl/{{ app_name }}
        state: directory
        owner: root
        group: root
        mode: '0700'
    
    - name: Deploy SSL certificate
      copy:
        content: "{{ ssl_cert_content }}"
        dest: "/etc/ssl/{{ app_name }}/cert.pem"
        owner: root
        group: root
        mode: '0644'
      no_log: true
    
    - name: Deploy SSL private key
      copy:
        content: "{{ ssl_key_content }}"
        dest: "/etc/ssl/{{ app_name }}/key.pem"
        owner: root
        group: root
        mode: '0600'
      no_log: true
  tags: ssl

- name: Create application directories
  file:
    path: "{{ item }}"
    state: directory
    owner: "{{ app_user }}"
    group: "{{ app_user }}"
    mode: '0750'
  loop:
    - "/opt/{{ app_name }}"
    - "/opt/{{ app_name }}/logs"
    - "/opt/{{ app_name }}/data"
  tags: app

- name: Deploy application secrets
  template:
    src: app_secrets.yml.j2
    dest: "/opt/{{ app_name }}/.secrets.yml"
    owner: "{{ app_user }}"
    group: "{{ app_user }}"
    mode: '0600'
  no_log: true
  tags: app

- name: Configure audit logging
  include_tasks: audit.yml
  tags: audit

- name: Setup log rotation
  template:
    src: logrotate.j2
    dest: "/etc/logrotate.d/{{ app_name }}"
    owner: root
    group: root
    mode: '0644'
  tags: logging
```

6. **roles/secure_server/tasks/firewall.yml:**

yaml

```yaml
---
- name: Configure UFW (Debian/Ubuntu)
  block:
    - name: Install UFW
      apt:
        name: ufw
        state: present
    
    - name: Reset UFW
      ufw:
        state: reset
      when: reset_firewall | default(false)
    
    - name: Set UFW defaults
      ufw:
        direction: "{{ item.direction }}"
        policy: "{{ item.policy }}"
      loop:
        - { direction: 'incoming', policy: 'deny' }
        - { direction: 'outgoing', policy: 'allow' }
    
    - name: Allow SSH
      ufw:
        rule: allow
        port: "{{ ssh_port }}"
        proto: tcp
    
    - name: Allow application ports
      ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop:
        - '80'
        - '443'
        - "{{ app_port }}"
    
    - name: Allow from specific IPs
      ufw:
        rule: allow
        from_ip: "{{ item }}"
      loop: "{{ allowed_ips }}"
    
    - name: Enable UFW
      ufw:
        state: enabled
        logging: 'on'
  
  when: ansible_os_family == "Debian"

- name: Configure firewalld (RedHat/CentOS)
  block:
    - name: Install firewalld
      yum:
        name: firewalld
        state: present
    
    - name: Start firewalld
      service:
        name: firewalld
        state: started
        enabled: yes
    
    - name: Configure firewalld services
      firewalld:
        service: "{{ item }}"
        permanent: yes
        state: enabled
        immediate: yes
      loop:
        - ssh
        - http
        - https
    
    - name: Configure firewalld ports
      firewalld:
        port: "{{ item }}/tcp"
        permanent: yes
        state: enabled
        immediate: yes
      loop:
        - "{{ app_port }}"
  
  when: ansible_os_family == "RedHat"
```

7. **roles/secure_server/tasks/fail2ban.yml:**

yaml

```yaml
---
- name: Install fail2ban
  package:
    name: fail2ban
    state: present

- name: Create fail2ban local config
  template:
    src: jail.local.j2
    dest: /etc/fail2ban/jail.local
    owner: root
    group: root
    mode: '0644'
  notify: restart fail2ban

- name: Create custom filters
  copy:
    dest: "/etc/fail2ban/filter.d/{{ app_name }}.conf"
    content: |
      [Definition]
      failregex = ^.*Failed login attempt from <HOST>.*$
                  ^.*Authentication failure for .* from <HOST>.*$
      ignoreregex =
    owner: root
    group: root
    mode: '0644'
  notify: restart fail2ban

- name: Enable and start fail2ban
  service:
    name: fail2ban
    state: started
    enabled: yes
```

8. **roles/secure_server/templates/sshd_config.j2:**

jinja

```jinja
# {{ ansible_managed }}
# SSH Server Configuration

Port {{ ssh_port }}
Protocol 2

# Authentication
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no

# Security
X11Forwarding no
MaxAuthTries 3
MaxSessions 5

# Logging
SyslogFacility AUTH
LogLevel VERBOSE

# Allow only specific users
AllowUsers {{ admin_users | map(attribute='name') | join(' ') }}

# Crypto
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512

# Connection
ClientAliveInterval 300
ClientAliveCountMax 2
LoginGraceTime 60
```

9. **roles/secure_server/templates/jail.local.j2:**

jinja

```jinja
# {{ ansible_managed }}
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
destemail = {{ admin_email | default('root@localhost') }}
action = %(action_mwl)s

[sshd]
enabled = true
port = {{ ssh_port }}
filter = sshd
logpath = /var/log/auth.log
maxretry = 3

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
logpath = /var/log/nginx/error.log
maxretry = 5

[nginx-noscript]
enabled = true
filter = nginx-noscript
logpath = /var/log/nginx/access.log
maxretry = 6

[{{ app_name }}]
enabled = true
filter = {{ app_name }}
logpath = /opt/{{ app_name }}/logs/app.log
maxretry = 5
```

10. **playbooks/deploy_secure.yml:**

yaml

```yaml
---
- name: Security audit
  hosts: all
  gather_facts: yes
  
  pre_tasks:
    - name: Check if running as root
      fail:
        msg: "Do not run this playbook as root user"
      when: ansible_user_id == 'root'
      tags: always
    
    - name: Validate vault variables are encrypted
      assert:
        that:
          - vault_db_root_password is defined
          - vault_api_key_github is defined
        fail_msg: "Vault variables are not properly loaded"
      tags: always

- name: Deploy secure server configuration
  hosts: webservers
  become: yes
  
  roles:
    - role: secure_server
      tags: security

- name: Post-deployment security checks
  hosts: webservers
  become: yes
  
  tasks:
    - name: Check SSH configuration
      command: sshd -t
      changed_when: false
      tags: verify
    
    - name: Verify firewall is active
      command: "{{ 'ufw status' if ansible_os_family == 'Debian' else 'firewall-cmd --state' }}"
      register: firewall_status
      changed_when: false
      failed_when: "'active' not in firewall_status.stdout.lower()"
      tags: verify
    
    - name: Check fail2ban status
      command: fail2ban-client status
      register: fail2ban_status
      changed_when: false
      tags: verify
    
    - name: Verify SSL files exist
      stat:
        path: "{{ item }}"
      register: ssl_files
      failed_when: not ssl_files.stat.exists
      loop:
        - "/etc/ssl/{{ app_name }}/cert.pem"
        - "/etc/ssl/{{ app_name }}/key.pem"
      no_log: true
      tags: verify
    
    - name: Check application secrets file permissions
      stat:
        path: "/opt/{{ app_name }}/.secrets.yml"
      register: secrets_file
      failed_when: secrets_file.stat.mode != '0600'
      tags: verify
    
    - name: Security check summary
      debug:
        msg: |
          Security Configuration Complete
          ================================
          Firewall: {{ 'Active' if firewall_enabled else 'Disabled' }}
          Fail2ban: {{ 'Active' if fail2ban_enabled else 'Disabled' }}
          SSH Port: {{ ssh_port }}
          SSL: Configured
          Secrets: Protected
      tags: verify
```

11. **Запуск с проверками:**

bash

```bash
# Проверка синтаксиса
ansible-playbook playbooks/deploy_secure.yml --syntax-check

# Проверка vault файлов
ansible-vault view group_vars/all/vault.yml --vault-password-file .vault_pass

# Dry-run
ansible-playbook playbooks/deploy_secure.yml --check --diff

# Реальный запуск
ansible-playbook playbooks/deploy_secure.yml

# С дополнительным verbose для debug
ansible-playbook playbooks/deploy_secure.yml -vv

# Только теги безопасности
ansible-playbook playbooks/deploy_secure.yml --tags security,verify
```

12. **Создай скрипт для ротации секретов:**

bash

```bash
# rotate_secrets.sh
#!/bin/bash

set -e

echo "=== Secret Rotation Script ==="

# Backup current vault
cp group_vars/all/vault.yml group_vars/all/vault.yml.backup.$(date +%Y%m%d)

# Edit vault
ansible-vault edit group_vars/all/vault.yml --vault-password-file .vault_pass

# Confirm
read -p "Deploy new secrets? (yes/no): " confirm
if [ "$confirm" = "yes" ]; then
    ansible-playbook playbooks/deploy_secure.yml --tags app
    echo "Secrets rotated successfully"
else
    echo "Cancelled"
fi
```

### 🚀 Бонус (новое)

**1. Интеграция с HashiCorp Vault:**

bash

```bash
# Установка collection
ansible-galaxy collection install community.hashi_vault

# Использование
```

yaml

```yaml
---
- name: Use HashiCorp Vault
  hosts: localhost
  
  vars:
    vault_url: "https://vault.example.com:8200"
    vault_token: "{{ lookup('env', 'VAULT_TOKEN') }}"
  
  tasks:
    - name: Get secret from Vault
      set_fact:
        db_password: "{{ lookup('community.hashi_vault.hashi_vault', 'secret=secret/data/database:password token={{ vault_token }} url={{ vault_url }}') }}"
      no_log: true
    
    - name: Use secret
      debug:
        msg: "Password retrieved (hidden)"
```

**2. Автоматический audit trail:**

yaml

```yaml
# roles/audit/tasks/main.yml
---
- name: Create audit log directory
  file:
    path: /var/log/ansible-audit
    state: directory
    mode: '0750'

- name: Log playbook execution
  lineinfile:
    path: /var/log/ansible-audit/executions.log
    line: "{{ ansible_date_time.iso8601 }} | {{ lookup('env', 'USER') }} | {{ ansible_play_name }} | {{ inventory_hostname }}"
    create: yes
  delegate_to: localhost
  run_once: true

- name: Log changes
  shell: |
    echo "{{ ansible_date_time.iso8601 }} | Change: {{ item.key }} | {{ item.value }}" >> /var/log/ansible-audit/changes.log
  loop: "{{ ansible_diff_mode | default([]) }}"
  when: ansible_diff_mode is defined
  no_log: true
```

**3. Security scanning после deployment:**

yaml

```yaml
# playbooks/security_scan.yml
---
- name: Run security scan
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install security tools
      apt:
        name:
          - lynis
          - rkhunter
          - aide
        state: present
    
    - name: Run Lynis audit
      command: lynis audit system --quick
      register: lynis_result
      changed_when: false
    
    - name: Save Lynis report
      copy:
        content: "{{ lynis_result.stdout }}"
        dest: "/var/log/lynis-{{ ansible_date_time.date }}.log"
    
    - name: Check for rootkits
      command: rkhunter --check --skip-keypress
      register: rkhunter_result
      changed_when: false
      failed_when: false
    
    - name: Report findings
      debug:
        msg: "Security scan complete. Check /var/log/ for details"
```

---

Курс завершен! Готов к следующим инструкциям.

Дальше модулей нет?

7 янв.

Вы правы! Давайте продолжим курс. Добавлю еще модули для полноты картины.

---

## Модуль 7: CI/CD интеграция и автоматизация (25 минут)

### 🎯 Напоминалка

**GitLab CI/CD с Ansible:**

yaml

```yaml
# .gitlab-ci.yml
stages:
  - validate
  - test
  - deploy

variables:
  ANSIBLE_FORCE_COLOR: "true"
  ANSIBLE_HOST_KEY_CHECKING: "false"

before_script:
  - apt-get update -qq
  - apt-get install -y python3-pip
  - pip3 install ansible ansible-lint
  - eval $(ssh-agent -s)
  - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
  - mkdir -p ~/.ssh
  - chmod 700 ~/.ssh

# Validation stage
syntax-check:
  stage: validate
  script:
    - ansible-playbook playbooks/*.yml --syntax-check
  only:
    - merge_requests
    - main

lint-check:
  stage: validate
  script:
    - ansible-lint playbooks/
  allow_failure: true
  only:
    - merge_requests
    - main

# Test stage
test-staging:
  stage: test
  script:
    - ansible-playbook -i inventories/staging playbooks/site.yml --check --diff
  only:
    - merge_requests
  environment:
    name: staging

# Deploy stage
deploy-staging:
  stage: deploy
  script:
    - ansible-playbook -i inventories/staging playbooks/site.yml
  only:
    - main
  environment:
    name: staging
  when: manual

deploy-production:
  stage: deploy
  script:
    - ansible-playbook -i inventories/production playbooks/site.yml
  only:
    - main
  environment:
    name: production
  when: manual
  needs:
    - deploy-staging
```

**GitHub Actions с Ansible:**

yaml

```yaml
# .github/workflows/ansible.yml
name: Ansible CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install Ansible
        run: |
          pip install ansible ansible-lint
      
      - name: Ansible version
        run: ansible --version
      
      - name: Syntax check
        run: |
          ansible-playbook playbooks/*.yml --syntax-check
      
      - name: Ansible lint
        run: |
          ansible-lint playbooks/
        continue-on-error: true

  test:
    runs-on: ubuntu-latest
    needs: validate
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install dependencies
        run: |
          pip install ansible
      
      - name: Run molecule tests
        run: |
          pip install molecule molecule-docker
          cd roles/myapp
          molecule test

  deploy-staging:
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    environment: staging
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install Ansible
        run: pip install ansible
      
      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          echo "${{ secrets.VAULT_PASSWORD }}" > .vault_pass
          chmod 600 .vault_pass
      
      - name: Deploy to staging
        run: |
          ansible-playbook -i inventories/staging playbooks/site.yml

  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install Ansible
        run: pip install ansible
      
      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          echo "${{ secrets.VAULT_PASSWORD }}" > .vault_pass
          chmod 600 .vault_pass
      
      - name: Deploy to production
        run: |
          ansible-playbook -i inventories/production playbooks/site.yml \
            -e "deployment_version=${{ github.sha }}"
```

**Jenkins Pipeline:**

groovy

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        ANSIBLE_FORCE_COLOR = 'true'
        ANSIBLE_HOST_KEY_CHECKING = 'false'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Validate') {
            parallel {
                stage('Syntax Check') {
                    steps {
                        sh 'ansible-playbook playbooks/*.yml --syntax-check'
                    }
                }
                stage('Lint') {
                    steps {
                        sh 'ansible-lint playbooks/ || true'
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    ansible-playbook -i inventories/staging \
                    playbooks/site.yml --check --diff
                '''
            }
        }
        
        stage('Deploy Staging') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'ansible-ssh-key', keyFileVariable: 'SSH_KEY'),
                    string(credentialsId: 'vault-password', variable: 'VAULT_PASS')
                ]) {
                    sh '''
                        echo "$VAULT_PASS" > .vault_pass
                        chmod 600 .vault_pass
                        ansible-playbook -i inventories/staging \
                        playbooks/site.yml \
                        --private-key=$SSH_KEY
                        rm .vault_pass
                    '''
                }
            }
        }
        
        stage('Approval') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to Production?', ok: 'Deploy'
            }
        }
        
        stage('Deploy Production') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'ansible-ssh-key', keyFileVariable: 'SSH_KEY'),
                    string(credentialsId: 'vault-password', variable: 'VAULT_PASS')
                ]) {
                    sh '''
                        echo "$VAULT_PASS" > .vault_pass
                        chmod 600 .vault_pass
                        ansible-playbook -i inventories/production \
                        playbooks/site.yml \
                        --private-key=$SSH_KEY
                        rm .vault_pass
                    '''
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            slackSend(
                color: 'good',
                message: "Deployment successful: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Deployment failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
            )
        }
    }
}
```

**AWX/Ansible Tower интеграция:**

bash

```bash
# awx-cli для автоматизации
pip install awxkit

# Настройка
export TOWER_HOST=https://awx.example.com
export TOWER_USERNAME=admin
export TOWER_PASSWORD=password

# Запуск job template
awx job_templates launch <template_id> \
  --extra_vars '{"environment": "production"}' \
  --monitor

# Через API
curl -X POST https://awx.example.com/api/v2/job_templates/7/launch/ \
  -H "Authorization: Bearer $TOWER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "extra_vars": {
      "deployment_version": "v1.2.3"
    }
  }'
```

**Ansible Pull для self-service:**

yaml

```yaml
# Playbook для ansible-pull
# playbooks/pull.yml
---
- name: Configure server via pull
  hosts: localhost
  connection: local
  become: yes
  
  tasks:
    - name: Update system
      apt:
        update_cache: yes
        upgrade: dist
    
    - name: Install base packages
      apt:
        name:
          - vim
          - git
          - htop
        state: present
    
    - name: Configure monitoring
      include_role:
        name: monitoring
```

bash

```bash
# На сервере настроить cron для ansible-pull
ansible-pull \
  -U https://github.com/company/ansible-repo.git \
  -i localhost, \
  playbooks/pull.yml

# Cron job
0 */4 * * * ansible-pull -U https://github.com/company/ansible-repo.git -i localhost, playbooks/pull.yml >> /var/log/ansible-pull.log 2>&1
```

**Terraform + Ansible интеграция:**

hcl

```hcl
# main.tf
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.medium"
  key_name      = "ansible-key"
  
  tags = {
    Name = "web-${count.index + 1}"
    Role = "webserver"
  }
  
  provisioner "local-exec" {
    command = "echo '${self.public_ip}' >> inventory/dynamic_hosts"
  }
}

# Создать inventory для Ansible
resource "local_file" "ansible_inventory" {
  content = templatefile("${path.module}/templates/inventory.tpl", {
    web_ips = aws_instance.web[*].public_ip
  })
  filename = "${path.module}/inventory/production"
}

# Запустить Ansible после создания
resource "null_resource" "ansible_provision" {
  depends_on = [aws_instance.web]
  
  provisioner "local-exec" {
    command = "ansible-playbook -i inventory/production playbooks/site.yml"
  }
}
```

### 💻 Задание

Создай CI/CD pipeline для Ansible:

1. **Структура проекта с CI/CD:**

bash

```bash
mkdir -p ansible-cicd/{.github/workflows,playbooks,roles,inventories/{staging,production},tests}
cd ansible-cicd
```

2. **Создай тестовый playbook:**

yaml

```yaml
# playbooks/webserver.yml
---
- name: Deploy webserver
  hosts: webservers
  become: yes
  
  vars:
    app_version: "{{ deployment_version | default('latest') }}"
    deployment_timestamp: "{{ ansible_date_time.epoch }}"
  
  pre_tasks:
    - name: Validate deployment
      assert:
        that:
          - app_version is defined
          - ansible_distribution in ['Ubuntu', 'Debian']
        fail_msg: "Validation failed"
  
  tasks:
    - name: Create deployment marker
      copy:
        content: |
          Version: {{ app_version }}
          Timestamp: {{ deployment_timestamp }}
          Deployed by: {{ lookup('env', 'USER') | default('automation') }}
        dest: /etc/deployment-info
        mode: '0644'
    
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: Deploy application
      copy:
        content: |
          <html>
          <head><title>Deployment {{ app_version }}</title></head>
          <body>
            <h1>Application {{ app_version }}</h1>
            <p>Deployed at: {{ deployment_timestamp }}</p>
            <p>Host: {{ ansible_hostname }}</p>
          </body>
          </html>
        dest: /var/www/html/index.html
    
    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
  
  post_tasks:
    - name: Verify deployment
      uri:
        url: http://localhost/
        return_content: yes
      register: health_check
      failed_when: app_version not in health_check.content
```

3. **Создай GitHub Actions workflow:**

yaml

```yaml
# .github/workflows/deploy.yml
name: Deploy with Ansible

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

env:
  ANSIBLE_FORCE_COLOR: "1"

jobs:
  validate:
    name: Validate Playbooks
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          pip install ansible ansible-lint yamllint
      
      - name: YAML Lint
        run: |
          yamllint .
        continue-on-error: true
      
      - name: Ansible Lint
        run: |
          ansible-lint playbooks/
        continue-on-error: true
      
      - name: Syntax Check
        run: |
          for playbook in playbooks/*.yml; do
            ansible-playbook "$playbook" --syntax-check
          done

  test:
    name: Test Playbooks
    runs-on: ubuntu-latest
    needs: validate
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install dependencies
        run: |
          pip install ansible molecule molecule-docker docker
      
      - name: Run Molecule tests
        run: |
          # Если есть molecule тесты
          if [ -d "molecule" ]; then
            molecule test
          fi
        continue-on-error: true
      
      - name: Test with check mode
        run: |
          echo "localhost ansible_connection=local" > test_inventory
          ansible-playbook -i test_inventory playbooks/webserver.yml \
            --check --diff
        env:
          ANSIBLE_HOST_KEY_CHECKING: "False"

  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main' || github.event.inputs.environment == 'staging'
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install Ansible
        run: pip install ansible
      
      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H ${{ secrets.STAGING_HOST }} >> ~/.ssh/known_hosts
      
      - name: Setup Vault
        run: |
          echo "${{ secrets.VAULT_PASSWORD }}" > .vault_pass
          chmod 600 .vault_pass
      
      - name: Deploy to Staging
        run: |
          ansible-playbook -i inventories/staging \
            playbooks/webserver.yml \
            -e "deployment_version=${{ github.sha }}" \
            -e "deployed_by=${{ github.actor }}"
      
      - name: Cleanup
        if: always()
        run: |
          rm -f ~/.ssh/id_rsa .vault_pass

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.ref == 'refs/heads/main' || github.event.inputs.environment == 'production'
    environment:
      name: production
      url: https://example.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install Ansible
        run: pip install ansible
      
      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H ${{ secrets.PRODUCTION_HOST }} >> ~/.ssh/known_hosts
      
      - name: Setup Vault
        run: |
          echo "${{ secrets.VAULT_PASSWORD }}" > .vault_pass
          chmod 600 .vault_pass
      
      - name: Deploy to Production
        run: |
          ansible-playbook -i inventories/production \
            playbooks/webserver.yml \
            -e "deployment_version=${{ github.sha }}" \
            -e "deployed_by=${{ github.actor }}"
      
      - name: Cleanup
        if: always()
        run: |
          rm -f ~/.ssh/id_rsa .vault_pass
      
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: release-${{ github.sha }}
          release_name: Production Release ${{ github.sha }}
          body: |
            Deployed to production
            - Version: ${{ github.sha }}
            - Deployed by: ${{ github.actor }}
            - Timestamp: ${{ github.event.head_commit.timestamp }}

  notify:
    name: Send Notifications
    runs-on: ubuntu-latest
    needs: [deploy-staging, deploy-production]
    if: always()
    
    steps:
      - name: Slack Notification
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: |
            Deployment Status: ${{ job.status }}
            Repository: ${{ github.repository }}
            Branch: ${{ github.ref }}
            Commit: ${{ github.sha }}
            Actor: ${{ github.actor }}
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        if: always()
```

4. **Создай pre-commit hooks:**

yaml

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
  
  - repo: https://github.com/ansible/ansible-lint
    rev: v6.14.0
    hooks:
      - id: ansible-lint
        files: \.(yaml|yml)$
```

bash

```bash
# Установка
pip install pre-commit
pre-commit install

# Запуск вручную
pre-commit run --all-files
```

5. **Создай Makefile для удобства:**

makefile

```makefile
# Makefile
.PHONY: help validate test deploy-staging deploy-production

help:
	@echo "Available targets:"
	@echo "  validate         - Run validation checks"
	@echo "  test            - Run tests"
	@echo "  deploy-staging  - Deploy to staging"
	@echo "  deploy-production - Deploy to production"

validate:
	@echo "Running validation..."
	ansible-playbook playbooks/*.yml --syntax-check
	ansible-lint playbooks/
	yamllint .

test:
	@echo "Running tests..."
	ansible-playbook -i inventories/staging playbooks/webserver.yml --check --diff

deploy-staging:
	@echo "Deploying to staging..."
	ansible-playbook -i inventories/staging playbooks/webserver.yml \
		-e "deployment_version=$$(git rev-parse HEAD)"

deploy-production:
	@echo "Deploying to production..."
	@read -p "Are you sure? [y/N] " -n 1 -r; \
	echo; \
	if [[ $$REPLY =~ ^[Yy]$$ ]]; then \
		ansible-playbook -i inventories/production playbooks/webserver.yml \
			-e "deployment_version=$$(git rev-parse HEAD)"; \
	fi

clean:
	@echo "Cleaning up..."
	find . -name "*.retry" -delete
	find . -name "__pycache__" -type d -exec rm -rf {} +
```

### 🚀 Бонус (новое)

**1. Blue-Green Deployment:**

yaml

```yaml
# playbooks/blue-green-deploy.yml
---
- name: Blue-Green Deployment
  hosts: localhost
  gather_facts: no
  
  vars:
    blue_servers: "{{ groups['blue'] }}"
    green_servers: "{{ groups['green'] }}"
    active_env: blue  # Current active environment
  
  tasks:
    - name: Determine target environment
      set_fact:
        target_env: "{{ 'green' if active_env == 'blue' else 'blue' }}"
        target_servers: "{{ green_servers if active_env == 'blue' else blue_servers }}"
    
    - name: Deploy to {{ target_env }} environment
      include_tasks: deploy_app.yml
      vars:
        target_hosts: "{{ target_env }}"
    
    - name: Run smoke tests on {{ target_env }}
      uri:
        url: "http://{{ item }}:8080/health"
        status_code: 200
      loop: "{{ target_servers }}"
      register: health_checks
    
    - name: Switch load balancer to {{ target_env }}
      community.general.haproxy:
        backend: "{{ target_env }}"
        state: enabled
      when: health_checks is success
    
    - name: Disable {{ active_env }} in load balancer
      community.general.haproxy:
        backend: "{{ active_env }}"
        state: disabled
      when: health_checks is success
    
    - name: Update active environment marker
      copy:
        content: "{{ target_env }}"
        dest: /etc/active_environment
      when: health_checks is success
```

**2. Canary Deployment:**

yaml

```yaml
# playbooks/canary-deploy.yml
---
- name: Canary Deployment
  hosts: webservers
  serial:
    - 1          # First server (canary)
    - 25%        # Then 25%
    - 50%        # Then 50%
    - 100%       # Finally all
  max_fail_percentage: 0
  
  tasks:
    - name: Deploy new version
      include_role:
        name: deploy_app
    
    - name: Wait for stabilization
      pause:
        minutes: 5
      when: inventory_hostname == groups['webservers'][0]
    
    - name: Check error rate
      uri:
        url: "http://monitoring.example.com/api/error_rate?host={{ inventory_hostname }}"
        return_content: yes
      register: error_rate
      failed_when: error_rate.json.rate > 1.0
    
    - name: Check response time
      uri:
        url: "http://monitoring.example.com/api/response_time?host={{ inventory_hostname }}"
        return_content: yes
      register: response_time
      failed_when: response_time.json.p95 > 500
    
    - name: Continue or rollback
      debug:
        msg: "Canary looks good, continuing deployment"
```

**3. Rollback mechanism:**

yaml

```yaml
# playbooks/rollback.yml
---
- name: Rollback Deployment
  hosts: webservers
  become: yes
  
  vars:
    rollback_version: "{{ deployment_version | default('previous') }}"
  
  tasks:
    - name: Get current version
      slurp:
        src: /etc/deployment-info
      register: current_deployment
    
    - name: Parse current version
      set_fact:
        current_version: "{{ (current_deployment.content | b64decode).split('\n')[0].split(':')[1].strip() }}"
    
    - name: Get previous version from backup
      find:
        paths: /backup/deployments
        patterns: "deployment-*.tar.gz"
      register: backups
    
    - name: Sort backups by date
      set_fact:
        latest_backup: "{{ backups.files | sort(attribute='mtime', reverse=true) | first }}"
    
    - name: Restore from backup
      unarchive:
        src: "{{ latest_backup.path }}"
        dest: /opt/app
        remote_src: yes
    
    - name: Restart application
      service:
        name: myapp
        state: restarted
    
    - name: Verify rollback
      uri:
        url: http://localhost:8080/health
        status_code: 200
      register: health_check
      until: health_check.status == 200
      retries: 5
      delay: 10
    
    - name: Update deployment info
      copy:
        content: |
          Version: {{ rollback_version }}
          Rolled back from: {{ current_version }}
          Timestamp: {{ ansible_date_time.epoch }}
        dest: /etc/deployment-info
```

---

## Модуль 8: Мониторинг, Логирование и Troubleshooting (20 минут)

### 🎯 Напоминалка

**Ansible callback plugins для мониторинга:**

ini

```ini
# ansible.cfg
[defaults]
callback_whitelist = profile_tasks, timer, mail
callbacks_enabled = profile_tasks, timer

[callback_profile_tasks]
task_output_limit = 20
sort_order = descending

[callback_mail]
to = devops@example.com
sender = ansible@example.com
```

**Логирование Ansible:**

ini

```ini
# ansible.cfg
[defaults]
log_path = /var/log/ansible/ansible.log
display_skipped_hosts = False
display_ok_hosts = True

# Подробный вывод
stdout_callback = yaml
bin_ansible_callbacks = True
```

**Custom callback plugin:**

python

```python
# callback_plugins/custom_logger.py
from ansible.plugins.callback import CallbackBase
from datetime import datetime
import json

class CallbackModule(CallbackBase):
    """
    Custom callback plugin for detailed logging
    """
    CALLBACK_VERSION = 2.0
    CALLBACK_TYPE = 'notification'
    CALLBACK_NAME = 'custom_logger'
    
    def __init__(self):
        super(CallbackModule, self).__init__()
        self.log_file = '/var/log/ansible/detailed.json'
    
    def v2_playbook_on_start(self, playbook):
        self.playbook = playbook._file_name
        self.log_event('playbook_start', {
            'playbook': self.playbook,
            'timestamp': datetime.now().isoformat()
        })
    
    def v2_playbook_on_task_start(self, task, is_conditional):
        self.log_event('task_start', {
            'task': task.get_name(),
            'timestamp': datetime.now().isoformat()
        })
    
    def v2_runner_on_ok(self, result):
        self.log_event('task_ok', {
            'host': result._host.get_name(),
            'task': result._task.get_name(),
            'result': result._result,
            'timestamp': datetime.now().isoformat()
        })
    
    def v2_runner_on_failed(self, result, ignore_errors=False):
        self.log_event('task_failed', {
            'host': result._host.get_name(),
            'task': result._task.get_name(),
            'error': result._result.get('msg', ''),
            'timestamp': datetime.now().isoformat()
        })
    
    def log_event(self, event_type, data):
        with open(self.log_file, 'a') as f:
            log_entry = {
                'event': event_type,
                'data': data
            }
            f.write(json.dumps(log_entry) + '\n')
```

**Prometheus метрики для Ansible:**

yaml

```yaml
# playbooks/prometheus_exporter.yml
---
- name: Setup Prometheus Node Exporter
  hosts: all
  become: yes
  
  tasks:
    - name: Download node exporter
      get_url:
        url: https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
        dest: /tmp/node_exporter.tar.gz
    
    - name: Extract node exporter
      unarchive:
        src: /tmp/node_exporter.tar.gz
        dest: /opt
        remote_src: yes
    
    - name: Create node exporter service
      copy:
        content: |
          [Unit]
          Description=Node Exporter
          After=network.target
          
          [Service]
          Type=simple
          ExecStart=/opt/node_exporter-1.6.0.linux-amd64/node_exporter
          
          [Install]
          WantedBy=multi-user.target
        dest: /etc/systemd/system/node_exporter.service
    
    - name: Start node exporter
      systemd:
        name: node_exporter
        state: started
        enabled: yes
        daemon_reload: yes
```

**ELK Stack для централизованных логов:**

yaml

```yaml
# roles/filebeat/tasks/main.yml
---
- name: Install Filebeat
  apt:
    deb: https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.8.0-amd64.deb
    state: present

- name: Configure Filebeat
  template:
    src: filebeat.yml.j2
    dest: /etc/filebeat/filebeat.yml
  notify: restart filebeat

- name: Enable Filebeat modules
  command: filebeat modules enable {{ item }}
  loop:
    - system
    - nginx
    - mysql
  notify: restart filebeat

-
```

Continue

8 янв.

name: Start Filebeat service: name: filebeat state: started enabled: yes

````
```jinja
# templates/filebeat.yml.j2
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/ansible/*.log
    - /opt/{{ app_name }}/logs/*.log
  fields:
    environment: {{ environment }}
    application: {{ app_name }}
    
output.elasticsearch:
  hosts: ["{{ elasticsearch_host }}:9200"]
  username: "{{ elk_username }}"
  password: "{{ elk_password }}"

setup.kibana:
  host: "{{ kibana_host }}:5601"
```

**Troubleshooting playbook:**
```yaml
# playbooks/troubleshoot.yml
---
- name: System Diagnostics
  hosts: problematic_host
  become: yes
  gather_facts: yes
  
  tasks:
    - name: Check system resources
      debug:
        msg: |
          CPU Cores: {{ ansible_processor_cores }}
          Memory: {{ ansible_memtotal_mb }}MB
          Disk Space: {{ ansible_mounts | json_query('[?mount==`/`].size_available') | first }}
    
    - name: Check disk usage
      shell: df -h
      register: disk_usage
    
    - name: Display disk usage
      debug:
        var: disk_usage.stdout_lines
    
    - name: Check memory usage
      shell: free -h
      register: memory_usage
    
    - name: Display memory usage
      debug:
        var: memory_usage.stdout_lines
    
    - name: Check running processes
      shell: ps aux --sort=-%mem | head -20
      register: top_processes
    
    - name: Display top processes
      debug:
        var: top_processes.stdout_lines
    
    - name: Check system load
      shell: uptime
      register: system_load
    
    - name: Display system load
      debug:
        var: system_load.stdout
    
    - name: Check network connections
      shell: netstat -tuln | grep LISTEN
      register: network_connections
    
    - name: Display network connections
      debug:
        var: network_connections.stdout_lines
    
    - name: Check recent logs
      shell: journalctl -n 50 --no-pager
      register: recent_logs
    
    - name: Display recent logs
      debug:
        var: recent_logs.stdout_lines
    
    - name: Check failed services
      shell: systemctl --failed
      register: failed_services
    
    - name: Display failed services
      debug:
        var: failed_services.stdout_lines
    
    - name: Generate diagnostic report
      template:
        src: diagnostic_report.j2
        dest: "/tmp/diagnostic_{{ inventory_hostname }}_{{ ansible_date_time.epoch }}.txt"
      delegate_to: localhost
```

Продолжить с заключением и финальным проектом?
````

Продолжи создание

8 янв.

jinja

```jinja
# templates/diagnostic_report.j2
=== System Diagnostic Report ===
Generated: {{ ansible_date_time.iso8601 }}
Host: {{ inventory_hostname }}
IP: {{ ansible_default_ipv4.address }}

=== System Information ===
Distribution: {{ ansible_distribution }} {{ ansible_distribution_version }}
Kernel: {{ ansible_kernel }}
Architecture: {{ ansible_architecture }}
Hostname: {{ ansible_hostname }}
FQDN: {{ ansible_fqdn }}

=== Hardware ===
CPU Cores: {{ ansible_processor_cores }}
CPU Model: {{ ansible_processor[2] }}
Total Memory: {{ ansible_memtotal_mb }} MB
Free Memory: {{ ansible_memfree_mb }} MB
Swap Total: {{ ansible_swaptotal_mb }} MB
Swap Free: {{ ansible_swapfree_mb }} MB

=== Disk Usage ===
{% for mount in ansible_mounts %}
{{ mount.mount }}: {{ (mount.size_available / 1024 / 1024 / 1024) | round(2) }}GB free of {{ (mount.size_total / 1024 / 1024 / 1024) | round(2) }}GB
{% endfor %}

=== Network ===
{% for interface in ansible_interfaces %}
{{ interface }}: {{ hostvars[inventory_hostname]['ansible_' + interface].get('ipv4', {}).get('address', 'N/A') }}
{% endfor %}

=== Services Status ===
{{ failed_services.stdout }}

=== Top Processes ===
{{ top_processes.stdout }}

=== Recent Logs ===
{{ recent_logs.stdout }}

=== Network Connections ===
{{ network_connections.stdout }}
```

**Ansible Tower/AWX API для мониторинга:**

python

```python
# scripts/monitor_ansible_jobs.py
#!/usr/bin/env python3

import requests
import json
from datetime import datetime, timedelta

class AnsibleTowerMonitor:
    def __init__(self, tower_url, token):
        self.tower_url = tower_url
        self.headers = {
            'Authorization': f'Bearer {token}',
            'Content-Type': 'application/json'
        }
    
    def get_recent_jobs(self, hours=24):
        """Get jobs from last N hours"""
        url = f"{self.tower_url}/api/v2/jobs/"
        since = datetime.now() - timedelta(hours=hours)
        
        params = {
            'created__gte': since.isoformat(),
            'order_by': '-created'
        }
        
        response = requests.get(url, headers=self.headers, params=params)
        return response.json()
    
    def get_failed_jobs(self):
        """Get all failed jobs"""
        url = f"{self.tower_url}/api/v2/jobs/"
        params = {'status': 'failed'}
        
        response = requests.get(url, headers=self.headers, params=params)
        return response.json()
    
    def get_job_details(self, job_id):
        """Get detailed job information"""
        url = f"{self.tower_url}/api/v2/jobs/{job_id}/"
        response = requests.get(url, headers=self.headers)
        return response.json()
    
    def get_job_events(self, job_id):
        """Get job events (tasks executed)"""
        url = f"{self.tower_url}/api/v2/jobs/{job_id}/job_events/"
        response = requests.get(url, headers=self.headers)
        return response.json()
    
    def generate_report(self):
        """Generate monitoring report"""
        recent_jobs = self.get_recent_jobs()
        failed_jobs = self.get_failed_jobs()
        
        total = recent_jobs['count']
        failed = failed_jobs['count']
        success_rate = ((total - failed) / total * 100) if total > 0 else 0
        
        report = {
            'timestamp': datetime.now().isoformat(),
            'total_jobs_24h': total,
            'failed_jobs': failed,
            'success_rate': f"{success_rate:.2f}%",
            'failed_job_details': []
        }
        
        for job in failed_jobs['results'][:10]:  # Last 10 failed
            report['failed_job_details'].append({
                'id': job['id'],
                'name': job['name'],
                'created': job['created'],
                'status': job['status']
            })
        
        return report

# Usage
if __name__ == '__main__':
    monitor = AnsibleTowerMonitor(
        tower_url='https://awx.example.com',
        token='your-api-token'
    )
    
    report = monitor.generate_report()
    print(json.dumps(report, indent=2))
```

**Health check playbook:**

yaml

```yaml
# playbooks/health_check.yml
---
- name: Infrastructure Health Check
  hosts: all
  gather_facts: yes
  
  vars:
    health_checks:
      disk_threshold: 90  # %
      memory_threshold: 90  # %
      cpu_load_threshold: 5.0
      service_list:
        - nginx
        - mysql
        - redis
  
  tasks:
    - name: Check disk usage
      set_fact:
        disk_usage_percent: "{{ (ansible_mounts | selectattr('mount', 'equalto', '/') | map(attribute='size_available') | first / ansible_mounts | selectattr('mount', 'equalto', '/') | map(attribute='size_total') | first * 100) | round(0) | int }}"
    
    - name: Disk usage warning
      debug:
        msg: "WARNING: Disk usage is {{ 100 - disk_usage_percent }}%"
      when: (100 - disk_usage_percent | int) > health_checks.disk_threshold
    
    - name: Check memory usage
      set_fact:
        memory_usage_percent: "{{ ((ansible_memtotal_mb - ansible_memfree_mb) / ansible_memtotal_mb * 100) | round(0) | int }}"
    
    - name: Memory usage warning
      debug:
        msg: "WARNING: Memory usage is {{ memory_usage_percent }}%"
      when: memory_usage_percent | int > health_checks.memory_threshold
    
    - name: Check CPU load
      shell: cat /proc/loadavg | awk '{print $1}'
      register: cpu_load
      changed_when: false
    
    - name: CPU load warning
      debug:
        msg: "WARNING: CPU load is {{ cpu_load.stdout }}"
      when: cpu_load.stdout | float > health_checks.cpu_load_threshold
    
    - name: Check critical services
      service_facts:
    
    - name: Verify services are running
      assert:
        that:
          - ansible_facts.services[item + '.service'].state == 'running'
        fail_msg: "Service {{ item }} is not running!"
        success_msg: "Service {{ item }} is healthy"
      loop: "{{ health_checks.service_list }}"
      when: item + '.service' in ansible_facts.services
    
    - name: Check port accessibility
      wait_for:
        host: "{{ inventory_hostname }}"
        port: "{{ item }}"
        timeout: 5
      loop:
        - 22
        - 80
        - 443
      delegate_to: localhost
    
    - name: Test HTTP endpoint
      uri:
        url: "http://{{ inventory_hostname }}/health"
        status_code: 200
        timeout: 10
      register: http_check
      failed_when: false
      when: "'webservers' in group_names"
    
    - name: HTTP check result
      debug:
        msg: "HTTP health check: {{ 'PASSED' if http_check.status == 200 else 'FAILED' }}"
      when: "'webservers' in group_names"
    
    - name: Create health report
      set_fact:
        health_status:
          host: "{{ inventory_hostname }}"
          timestamp: "{{ ansible_date_time.iso8601 }}"
          disk_usage: "{{ 100 - disk_usage_percent }}%"
          memory_usage: "{{ memory_usage_percent }}%"
          cpu_load: "{{ cpu_load.stdout }}"
          services_ok: "{{ health_checks.service_list | length }}"
          overall_status: "{{ 'HEALTHY' if (100 - disk_usage_percent | int) < health_checks.disk_threshold and memory_usage_percent | int < health_checks.memory_threshold else 'WARNING' }}"
    
    - name: Save health report
      copy:
        content: "{{ health_status | to_nice_json }}"
        dest: "/tmp/health_report_{{ inventory_hostname }}.json"
      delegate_to: localhost

- name: Aggregate health reports
  hosts: localhost
  gather_facts: no
  
  tasks:
    - name: Collect all health reports
      set_fact:
        all_reports: "{{ lookup('fileglob', '/tmp/health_report_*.json', wantlist=True) }}"
    
    - name: Generate summary
      debug:
        msg: "Health check completed for {{ groups['all'] | length }} hosts"
```

### 💻 Задание

Создай comprehensive мониторинг и troubleshooting систему:

1. **Создай monitoring role:**

bash

```bash
ansible-galaxy init roles/monitoring
```

2. **roles/monitoring/tasks/main.yml:**

yaml

```yaml
---
- name: Install monitoring tools
  apt:
    name:
      - htop
      - iotop
      - nethogs
      - sysstat
      - dstat
    state: present
  tags: packages

- name: Setup Prometheus Node Exporter
  include_tasks: node_exporter.yml
  tags: prometheus

- name: Setup log forwarding
  include_tasks: logging.yml
  tags: logging

- name: Configure health checks
  include_tasks: health_checks.yml
  tags: health

- name: Setup alerting
  include_tasks: alerting.yml
  tags: alerting
```

3. **roles/monitoring/tasks/node_exporter.yml:**

yaml

```yaml
---
- name: Create node_exporter user
  user:
    name: node_exporter
    system: yes
    shell: /bin/false
    create_home: no

- name: Download Node Exporter
  get_url:
    url: "https://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_version }}/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz"
    dest: /tmp/node_exporter.tar.gz
    mode: '0644'

- name: Extract Node Exporter
  unarchive:
    src: /tmp/node_exporter.tar.gz
    dest: /usr/local/bin
    remote_src: yes
    extra_opts:
      - --strip-components=1
      - --wildcards
      - '*/node_exporter'
    owner: node_exporter
    group: node_exporter
    mode: '0755'

- name: Create Node Exporter systemd service
  template:
    src: node_exporter.service.j2
    dest: /etc/systemd/system/node_exporter.service
    mode: '0644'
  notify: restart node_exporter

- name: Enable and start Node Exporter
  systemd:
    name: node_exporter
    enabled: yes
    state: started
    daemon_reload: yes

- name: Verify Node Exporter is running
  uri:
    url: http://localhost:9100/metrics
    status_code: 200
  register: ne_check
  until: ne_check.status == 200
  retries: 5
  delay: 2
```

4. **roles/monitoring/templates/node_exporter.service.j2:**

jinja

```jinja
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
Type=simple
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/bin/node_exporter \
  --collector.filesystem.mount-points-exclude=^/(dev|proc|sys|var/lib/docker/.+)($|/) \
  --collector.netclass.ignored-devices=^(veth.*)$

SyslogIdentifier=node_exporter
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

5. **roles/monitoring/tasks/logging.yml:**

yaml

```yaml
---
- name: Install rsyslog
  apt:
    name: rsyslog
    state: present

- name: Configure rsyslog for application logs
  template:
    src: rsyslog_app.conf.j2
    dest: /etc/rsyslog.d/50-app.conf
    mode: '0644'
  notify: restart rsyslog

- name: Create log directory
  file:
    path: /var/log/{{ app_name }}
    state: directory
    owner: syslog
    group: adm
    mode: '0755'

- name: Setup logrotate
  template:
    src: logrotate.j2
    dest: /etc/logrotate.d/{{ app_name }}
    mode: '0644'
```

6. **roles/monitoring/templates/logrotate.j2:**

jinja

```jinja
/var/log/{{ app_name }}/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 {{ app_user }} {{ app_user }}
    sharedscripts
    postrotate
        systemctl reload {{ app_name }} > /dev/null 2>&1 || true
    endscript
}
```

7. **roles/monitoring/tasks/health_checks.yml:**

yaml

```yaml
---
- name: Create health check script
  template:
    src: health_check.sh.j2
    dest: /usr/local/bin/health_check.sh
    mode: '0755'

- name: Setup health check cron
  cron:
    name: "System health check"
    minute: "*/15"
    job: "/usr/local/bin/health_check.sh"
    user: root

- name: Create health check endpoint
  copy:
    content: |
      #!/bin/bash
      echo "Content-Type: application/json"
      echo ""
      
      LOAD=$(cat /proc/loadavg | awk '{print $1}')
      MEMORY=$(free | grep Mem | awk '{printf "%.0f", $3/$2 * 100}')
      DISK=$(df -h / | tail -1 | awk '{print $5}' | sed 's/%//')
      
      cat << EOF
      {
        "status": "healthy",
        "timestamp": "$(date -Iseconds)",
        "metrics": {
          "load": $LOAD,
          "memory_percent": $MEMORY,
          "disk_percent": $DISK
        }
      }
      EOF
    dest: /var/www/html/health
    mode: '0755'
  when: "'webservers' in group_names"
```

8. **roles/monitoring/tasks/alerting.yml:**

yaml

```yaml
---
- name: Install mailutils for alerts
  apt:
    name: mailutils
    state: present

- name: Create alerting script
  template:
    src: send_alert.sh.j2
    dest: /usr/local/bin/send_alert.sh
    mode: '0755'

- name: Configure alert thresholds
  copy:
    content: |
      DISK_THRESHOLD=90
      MEMORY_THRESHOLD=90
      CPU_THRESHOLD=5.0
      ALERT_EMAIL="{{ alert_email | default('admin@example.com') }}"
    dest: /etc/monitoring/thresholds.conf
    mode: '0644'
```

9. **playbooks/monitoring_setup.yml:**

yaml

```yaml
---
- name: Setup Comprehensive Monitoring
  hosts: all
  become: yes
  
  vars:
    node_exporter_version: "1.6.0"
    alert_email: "devops@example.com"
  
  roles:
    - role: monitoring

  post_tasks:
    - name: Run initial health check
      command: /usr/local/bin/health_check.sh
      register: health_result
      changed_when: false
    
    - name: Display health status
      debug:
        var: health_result.stdout_lines

- name: Configure Prometheus Server
  hosts: monitoring_server
  become: yes
  
  tasks:
    - name: Install Prometheus
      include_role:
        name: prometheus_server
    
    - name: Add monitoring targets
      template:
        src: prometheus_targets.yml.j2
        dest: /etc/prometheus/targets.yml
      notify: reload prometheus
```

10. **Troubleshooting playbook с детальной диагностикой:**

yaml

```yaml
# playbooks/deep_troubleshoot.yml
---
- name: Deep System Troubleshooting
  hosts: "{{ target_host | default('all') }}"
  become: yes
  gather_facts: yes
  
  vars:
    output_dir: "/tmp/troubleshoot_{{ ansible_date_time.epoch }}"
  
  tasks:
    - name: Create output directory
      file:
        path: "{{ output_dir }}"
        state: directory
      delegate_to: localhost
      run_once: true
    
    - name: Collect system information
      block:
        - name: Get system info
          shell: |
            echo "=== System Info ==="
            uname -a
            cat /etc/os-release
          register: sys_info
        
        - name: Get resource usage
          shell: |
            echo "=== CPU Info ==="
            lscpu
            echo ""
            echo "=== Memory Info ==="
            free -h
            echo ""
            echo "=== Disk Info ==="
            df -h
            echo ""
            echo "=== Load Average ==="
            uptime
          register: resource_info
        
        - name: Get network info
          shell: |
            echo "=== Network Interfaces ==="
            ip addr show
            echo ""
            echo "=== Listening Ports ==="
            netstat -tulpn
            echo ""
            echo "=== Active Connections ==="
            netstat -an | grep ESTABLISHED | wc -l
          register: network_info
        
        - name: Get process info
          shell: |
            echo "=== Top Memory Consumers ==="
            ps aux --sort=-%mem | head -20
            echo ""
            echo "=== Top CPU Consumers ==="
            ps aux --sort=-%cpu | head -20
          register: process_info
        
        - name: Get service status
          shell: |
            echo "=== Failed Services ==="
            systemctl --failed
            echo ""
            echo "=== Service Status ==="
            systemctl status nginx mysql redis || true
          register: service_info
        
        - name: Get logs
          shell: |
            echo "=== System Logs (last 100 lines) ==="
            journalctl -n 100 --no-pager
            echo ""
            echo "=== Auth Logs ==="
            tail -50 /var/log/auth.log 2>/dev/null || echo "Not available"
          register: log_info
        
        - name: Get disk I/O
          shell: |
            echo "=== Disk I/O Stats ==="
            iostat -x 1 5
          register: io_info
        
        - name: Check for issues
          shell: |
            echo "=== Checking for common issues ==="
            echo "Disk full partitions:"
            df -h | awk '$5 ~ /9[0-9]%/ || $5 ~ /100%/ {print}'
            echo ""
            echo "Out of memory killer activity:"
            grep -i "out of memory" /var/log/syslog 2>/dev/null | tail -5 || echo "None"
            echo ""
            echo "Zombie processes:"
            ps aux | awk '$8=="Z" {print}'
          register: issue_check
      
      always:
        - name: Compile troubleshooting report
          copy:
            content: |
              ========================================
              TROUBLESHOOTING REPORT
              ========================================
              Host: {{ inventory_hostname }}
              Date: {{ ansible_date_time.iso8601 }}
              Generated by: {{ lookup('env', 'USER') }}
              
              {{ sys_info.stdout }}
              
              {{ resource_info.stdout }}
              
              {{ network_info.stdout }}
              
              {{ process_info.stdout }}
              
              {{ service_info.stdout }}
              
              {{ log_info.stdout }}
              
              {{ io_info.stdout }}
              
              {{ issue_check.stdout }}
              
              ========================================
              END OF REPORT
              ========================================
            dest: "{{ output_dir }}/{{ inventory_hostname }}_report.txt"
          delegate_to: localhost
        
        - name: Create summary
          debug:
            msg: |
              Troubleshooting complete for {{ inventory_hostname }}
              Report saved to: {{ output_dir }}/{{ inventory_hostname }}_report.txt
              
              Quick Summary:
              - CPU Load: {{ ansible_facts.loadavg.1m }}
              - Memory Free: {{ ansible_memfree_mb }}MB / {{ ansible_memtotal_mb }}MB
              - Root Disk Free: {{ ansible_mounts | selectattr('mount', 'equalto', '/') | map(attribute='size_available') | first | filesizeformat }}

- name: Generate Master Report
  hosts: localhost
  gather_facts: no
  
  tasks:
    - name: Combine all reports
      shell: |
        cat {{ output_dir }}/*_report.txt > {{ output_dir }}/MASTER_REPORT.txt
        echo "Master report created at {{ output_dir }}/MASTER_REPORT.txt"
      args:
        warn: false
```

### 🚀 Бонус (новое)

**1. Grafana Dashboard автоматизация:**

yaml

```yaml
# playbooks/setup_grafana_dashboards.yml
---
- name: Setup Grafana Dashboards
  hosts: monitoring_server
  become: yes
  
  tasks:
    - name: Install Grafana
      apt:
        deb: https://dl.grafana.com/oss/release/grafana_10.0.0_amd64.deb
        state: present
    
    - name: Start Grafana
      service:
        name: grafana-server
        state: started
        enabled: yes
    
    - name: Wait for Grafana to start
      uri:
        url: http://localhost:3000/api/health
        status_code: 200
      register: result
      until: result.status == 200
      retries: 10
      delay: 5
    
    - name: Add Prometheus datasource
      uri:
        url: http://localhost:3000/api/datasources
        method: POST
        user: admin
        password: admin
        force_basic_auth: yes
        body_format: json
        body:
          name: Prometheus
          type: prometheus
          url: http://localhost:9090
          access: proxy
          isDefault: true
        status_code: [200, 409]  # 409 if already exists
    
    - name: Import Node Exporter dashboard
      uri:
        url: http://localhost:3000/api/dashboards/import
        method: POST
        user: admin
        password: admin
        force_basic_auth: yes
        body_format: json
        body:
          dashboard:
            id: 1860  # Node Exporter Full dashboard
            title: "Node Exporter Full"
          overwrite: true
          inputs:
            - name: "DS_PROMETHEUS"
              type: "datasource"
              pluginId: "prometheus"
              value: "Prometheus"
        status_code: [200, 412]
```

**2. Automated incident response:**

yaml

```yaml
# playbooks/incident_response.yml
---
- name: Automated Incident Response
  hosts: "{{ incident_host }}"
  become: yes
  gather_facts: yes
  
  vars:
    incident_type: "{{ incident_type | default('unknown') }}"
    incident_id: "{{ ansible_date_time.epoch }}"
  
  tasks:
    - name: Create incident directory
      file:
        path: "/var/log/incidents/{{ incident_id }}"
        state: directory
        mode: '0755'
    
    - name: Log incident start
      lineinfile:
        path: "/var/log/incidents/{{ incident_id }}/incident.log"
        line: "{{ ansible_date_time.iso8601 }} | Incident {{ incident_id }} started | Type: {{ incident_type }}"
        create: yes
    
    - name: Collect incident data
      shell: |
        ps aux > /var/log/incidents/{{ incident_id }}/processes.txt
        netstat -tulpn > /var/log/incidents/{{ incident_id }}/network.txt
        journalctl -n 1000 > /var/log/incidents/{{ incident_id }}/logs.txt
        df -h > /var/log/incidents/{{ incident_id }}/disk.txt
    
    - name: Response based on incident type
      include_tasks: "responses/{{ incident_type }}.yml"
      when: incident_type != 'unknown'
    
    - name: Notify team
      uri:
        url: "{{ slack_webhook_url }}"
        method: POST
        body_format: json
        body:
          text: |
            🚨 Incident Response Activated
            Host: {{ inventory_hostname }}
            Type: {{ incident_type }}
            ID: {{ incident_id }}
            Time: {{ ansible_date_time.iso8601 }}
        status_code: 200
      delegate_to: localhost
      when: slack_webhook_url is defined
```

**3. Performance baseline collection:**

yaml

````yaml
# playbooks/collect_baseline.yml
---
- name: Collect Performance Baseline
  hosts: all
  become: yes
  serial: 5  # Don't overload
  
  tasks:
    - name: Install performance tools
      apt:
        name:
          - sysstat
          - iotop
          - dstat
        state: present
    
    - name: Collect CPU baseline
      shell: |
        mpstat 1 60 > /tmp/cpu_baseline.txt
      async: 120
      poll: 0
      register: cpu_job
    
    - name: Collect disk I/O baseline
      shell: |
        iostat -x 1 60 > /tmp/io_baseline.txt
      async: 120
      poll: 0
      register: io_job
    
    - name: Collect network baseline
      shell: |
        sar -n DEV 1 60 > /tmp/network_baseline.txt
      async: 120
      poll: 0
      register: network_job
    
    - name: Wait for collection to complete
      async_status:
        jid: "{{ item.ansible_job_id }}"
      register: job_result
      until: job_result.finished
      retries: 150
      delay: 1
      loop:
        - "{{ cpu_job }}"
        - "{{ io_job }}"
        - "{{ network_job }}"
    
    - name: Fetch baseline data
      fetch:
        src: "{{ item }}"
        dest: "baselines/{{ inventory_hostname }}/"
        flat: yes
      loop:
        - /tmp/cpu_baseline.txt
        - /tmp/io_baseline.txt
        - /tmp/network_baseline.txt
```

---

## Финальный проект: Полный Production Deployment (60+ минут)

### Задача

Создай complete production-ready инфраструктуру со всеми best practices:

**Требования:**
- Multi-tier application (web, app, db, cache)
- High Availability с Sentinel
- Load balancing
- Automated backups
- Monitoring и alerting
- Security hardening
- CI/CD pipeline
- Disaster recovery plan
- Documentation

**Структура финального проекта:**
```
production-infrastructure/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy-staging.yml
│       └── deploy-production.yml
├── ansible.cfg
├── inventories/
│   ├── production/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   └── host_vars/
│   └── staging/
│       ├── hosts.yml
│       └── group_vars/
├── group_vars/
│   ├── all/
│   │   ├── vars.yml
│   │   └── vault.yml
│   ├── webservers.yml
│   ├── appservers.yml
│   └── databases.yml
├── roles/
│   ├── common/
│   ├── security/
│   ├── nginx/
│   ├── application/
│   ├── postgresql/
│   ├── redis/
│   ├── monitoring/
│   └── backup/
├── playbooks/
│   ├── site.yml
│   ├── deploy.yml
│   ├── backup.yml
│   ├── restore.yml
│   ├── rollback.yml
│   └── maintenance.yml
├── scripts/
│   ├── deploy.sh
│   ├── backup.sh
│   └── health_check.sh
├── tests/
│   ├── test_deployment.yml
│   └── test_connectivity.yml
├── docs/
│   ├── README.md
│   ├── ARCHITECTURE.md
│   ├── DEPLOYMENT.md
│   └── RUNBOOK.md
└── Makefile
````

**inventories/production/hosts.yml:**

yaml

```yaml
all:
  children:
    loadbalancers:
      hosts:
        lb1.prod.example.com:
          ansible_host: 10.0.1.10
        lb2.prod.example.com:
          ansible_host: 10.0.1.11
    
    webservers:
      hosts:
        web1.prod.example.com:
          ansible_host: 10.0.2.10
        web2.prod.example.com:
          ansible_host: 10.0.2.11
        web3.prod.example.com:
          ansible_host: 10.0.2.12
    
    appservers:
      hosts:
        app1.prod.example.com:
          ansible_host: 10.0.3.10
          app_weight: 100
        app2.prod.example.com:
          ansible_host: 10.0.3.11
          app_weight: 100
        app3.prod.example.com:
          ansible_host: 10.0.3.12
          app_weight: 50
    
    databases:
      hosts:
        db1.prod.example.com:
          ansible_host: 10.0.4.10
          pg_role: primary
        db2.prod.example.com:
          ansible_host: 10.0.4.11
          pg_role: standby
    
    cache:
      hosts:
        redis1.prod.example.com:
          ansible_host: 10.0.5.10
          redis_role: master
        redis2.prod.example.com:
          ansible_host: 10.0.5.11
          redis_role: slave
        redis3.prod.example.com:
          ansible_host: 10.0.5.12
          redis_role: slave
    
    monitoring:
      hosts:
        monitor.prod.example.com:
          ansible_host: 10.0.6.10
    
    backup:
      hosts:
        backup.prod.example.com:
          ansible_host: 10.0.7.10

  vars:
    ansible_user: ansible
    ansible_python_interpreter: /usr/bin/python3
    environment: production
```

Continue

8 янв.

**playbooks/site.yml (main playbook):**

yaml

```yaml
---
# Main site playbook - deploys entire infrastructure

- name: Common configuration
  import_playbook: common.yml
  tags: common

- name: Security hardening
  import_playbook: security.yml
  tags: security

- name: Setup monitoring
  import_playbook: monitoring.yml
  tags: monitoring

- name: Setup backup system
  import_playbook: backup.yml
  tags: backup

- name: Deploy database tier
  import_playbook: database.yml
  tags: database

- name: Deploy cache tier
  import_playbook: cache.yml
  tags: cache

- name: Deploy application tier
  import_playbook: application.yml
  tags: application

- name: Deploy web tier
  import_playbook: web.yml
  tags: web

- name: Setup load balancers
  import_playbook: loadbalancer.yml
  tags: loadbalancer

- name: Final health check
  import_playbook: healthcheck.yml
  tags: healthcheck,always
```

**playbooks/deploy.yml (application deployment):**

yaml

```yaml
---
- name: Pre-deployment checks
  hosts: all
  gather_facts: yes
  tags: always
  
  tasks:
    - name: Validate deployment prerequisites
      assert:
        that:
          - deployment_version is defined
          - deployment_version is version('1.0.0', '>=')
        fail_msg: "deployment_version must be specified and >= 1.0.0"
      run_once: true
    
    - name: Check available disk space
      assert:
        that:
          - ansible_mounts | selectattr('mount', 'equalto', '/') | map(attribute='size_available') | first > 5000000000  # 5GB
        fail_msg: "Insufficient disk space"

- name: Create deployment backup
  hosts: appservers
  become: yes
  serial: 1
  tags: backup
  
  tasks:
    - name: Backup current deployment
      archive:
        path: /opt/application
        dest: "/backup/pre-deploy-{{ deployment_version }}-{{ ansible_date_time.epoch }}.tar.gz"
      delegate_to: "{{ groups['backup'][0] }}"

- name: Rolling deployment
  hosts: appservers
  become: yes
  serial: 1  # One server at a time
  tags: deploy
  
  pre_tasks:
    - name: Remove from load balancer
      haproxy:
        state: disabled
        host: "{{ inventory_hostname }}"
        socket: /var/run/haproxy.sock
      delegate_to: "{{ groups['loadbalancers'][0] }}"
  
  roles:
    - role: application
      vars:
        app_version: "{{ deployment_version }}"
  
  post_tasks:
    - name: Wait for application to be ready
      wait_for:
        port: 8080
        state: started
        timeout: 60
    
    - name: Health check
      uri:
        url: "http://localhost:8080/health"
        status_code: 200
      register: health
      until: health.status == 200
      retries: 10
      delay: 5
    
    - name: Add back to load balancer
      haproxy:
        state: enabled
        host: "{{ inventory_hostname }}"
        socket: /var/run/haproxy.sock
      delegate_to: "{{ groups['loadbalancers'][0] }}"
    
    - name: Wait for traffic to stabilize
      pause:
        seconds: 30

- name: Post-deployment verification
  hosts: loadbalancers
  tags: verify
  
  tasks:
    - name: Verify all backends are healthy
      command: echo "show stat" | socat stdio /var/run/haproxy.sock
      register: haproxy_stat
      changed_when: false
    
    - name: Check for DOWN backends
      assert:
        that:
          - "'DOWN' not in haproxy_stat.stdout"
        fail_msg: "Some backends are DOWN!"
    
    - name: Final end-to-end test
      uri:
        url: "http://{{ inventory_hostname }}/api/version"
        return_content: yes
      register: version_check
      failed_when: deployment_version not in version_check.content

- name: Update documentation
  hosts: localhost
  tags: docs
  
  tasks:
    - name: Record deployment
      lineinfile:
        path: "docs/DEPLOYMENTS.md"
        line: "| {{ ansible_date_time.date }} | {{ deployment_version }} | {{ lookup('env', 'USER') }} | {{ environment }} | SUCCESS |"
        create: yes
```

Продолжить с остальными playbooks и скриптами?

Продолжай

9 янв.

**playbooks/common.yml:**

yaml

```yaml
---
- name: Common server configuration
  hosts: all
  become: yes
  
  roles:
    - role: common
      tags: common
  
  tasks:
    - name: Set timezone
      timezone:
        name: "{{ timezone | default('UTC') }}"
      tags: timezone
    
    - name: Configure NTP
      apt:
        name: chrony
        state: present
      tags: ntp
    
    - name: Start chrony
      service:
        name: chrony
        state: started
        enabled: yes
      tags: ntp
    
    - name: Configure DNS
      template:
        src: templates/resolv.conf.j2
        dest: /etc/resolv.conf
        mode: '0644'
      tags: dns
    
    - name: Update package cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
      tags: packages
    
    - name: Install common packages
      apt:
        name: "{{ common_packages }}"
        state: present
      tags: packages
    
    - name: Configure limits
      pam_limits:
        domain: "*"
        limit_type: "{{ item.type }}"
        limit_item: "{{ item.item }}"
        value: "{{ item.value }}"
      loop:
        - { type: 'soft', item: 'nofile', value: '65536' }
        - { type: 'hard', item: 'nofile', value: '65536' }
        - { type: 'soft', item: 'nproc', value: '32768' }
        - { type: 'hard', item: 'nproc', value: '32768' }
      tags: limits
    
    - name: Configure sysctl
      sysctl:
        name: "{{ item.key }}"
        value: "{{ item.value }}"
        state: present
        reload: yes
      loop:
        - { key: 'net.ipv4.tcp_tw_reuse', value: '1' }
        - { key: 'net.ipv4.ip_local_port_range', value: '10240 65535' }
        - { key: 'net.core.somaxconn', value: '4096' }
        - { key: 'net.ipv4.tcp_max_syn_backlog', value: '8192' }
        - { key: 'vm.swappiness', value: '10' }
      tags: sysctl
```

**playbooks/database.yml:**

yaml

```yaml
---
- name: Setup PostgreSQL Primary
  hosts: databases
  become: yes
  serial: 1
  
  roles:
    - role: postgresql
      when: pg_role == 'primary'
  
  tasks:
    - name: Initialize primary database
      block:
        - name: Create application database
          postgresql_db:
            name: "{{ app_db_name }}"
            state: present
          become_user: postgres
        
        - name: Create application user
          postgresql_user:
            name: "{{ app_db_user }}"
            password: "{{ app_db_password }}"
            db: "{{ app_db_name }}"
            priv: ALL
            state: present
          become_user: postgres
          no_log: true
        
        - name: Configure replication user
          postgresql_user:
            name: replicator
            password: "{{ replication_password }}"
            role_attr_flags: REPLICATION
            state: present
          become_user: postgres
          no_log: true
        
        - name: Configure pg_hba for replication
          postgresql_pg_hba:
            dest: /etc/postgresql/14/main/pg_hba.conf
            contype: host
            users: replicator
            source: "{{ item }}"
            databases: replication
            method: md5
          loop: "{{ groups['databases'] | map('extract', hostvars, 'ansible_host') | list }}"
          notify: restart postgresql
      
      when: pg_role == 'primary'

- name: Setup PostgreSQL Standby
  hosts: databases
  become: yes
  
  tasks:
    - name: Setup standby server
      block:
        - name: Stop PostgreSQL
          service:
            name: postgresql
            state: stopped
        
        - name: Remove existing data directory
          file:
            path: /var/lib/postgresql/14/main
            state: absent
        
        - name: Create base backup from primary
          command: >
            pg_basebackup -h {{ hostvars[groups['databases'][0]]['ansible_host'] }}
            -D /var/lib/postgresql/14/main
            -U replicator -v -P -W
          become_user: postgres
          environment:
            PGPASSWORD: "{{ replication_password }}"
        
        - name: Create standby.signal
          file:
            path: /var/lib/postgresql/14/main/standby.signal
            state: touch
            owner: postgres
            group: postgres
        
        - name: Configure recovery settings
          lineinfile:
            path: /var/lib/postgresql/14/main/postgresql.auto.conf
            line: "primary_conninfo = 'host={{ hostvars[groups['databases'][0]]['ansible_host'] }} port=5432 user=replicator password={{ replication_password }}'"
            create: yes
          become_user: postgres
        
        - name: Start PostgreSQL standby
          service:
            name: postgresql
            state: started
      
      when: pg_role == 'standby'

- name: Verify replication
  hosts: databases
  become: yes
  
  tasks:
    - name: Check replication status on primary
      postgresql_query:
        db: postgres
        query: "SELECT * FROM pg_stat_replication;"
      become_user: postgres
      register: repl_status
      when: pg_role == 'primary'
    
    - name: Display replication status
      debug:
        var: repl_status.query_result
      when: pg_role == 'primary'
```

**playbooks/cache.yml:**

yaml

```yaml
---
- name: Setup Redis Cluster
  hosts: cache
  become: yes
  
  roles:
    - role: redis
  
  tasks:
    - name: Configure Redis master
      template:
        src: templates/redis-master.conf.j2
        dest: /etc/redis/redis.conf
        mode: '0644'
      when: redis_role == 'master'
      notify: restart redis
    
    - name: Configure Redis slave
      template:
        src: templates/redis-slave.conf.j2
        dest: /etc/redis/redis.conf
        mode: '0644'
      when: redis_role == 'slave'
      notify: restart redis
    
    - name: Ensure Redis is running
      service:
        name: redis-server
        state: started
        enabled: yes

- name: Setup Redis Sentinel
  hosts: cache
  become: yes
  
  tasks:
    - name: Install Redis Sentinel
      apt:
        name: redis-sentinel
        state: present
    
    - name: Configure Sentinel
      template:
        src: templates/sentinel.conf.j2
        dest: /etc/redis/sentinel.conf
        mode: '0644'
      notify: restart sentinel
    
    - name: Start Sentinel
      service:
        name: redis-sentinel
        state: started
        enabled: yes

- name: Verify Redis cluster
  hosts: cache[0]
  become: yes
  
  tasks:
    - name: Check Redis replication
      command: redis-cli -a {{ redis_password }} INFO replication
      register: redis_info
      changed_when: false
      no_log: true
    
    - name: Display replication info
      debug:
        var: redis_info.stdout_lines
```

**playbooks/loadbalancer.yml:**

yaml

```yaml
---
- name: Setup HAProxy Load Balancers
  hosts: loadbalancers
  become: yes
  
  tasks:
    - name: Install HAProxy
      apt:
        name: haproxy
        state: present
    
    - name: Configure HAProxy
      template:
        src: templates/haproxy.cfg.j2
        dest: /etc/haproxy/haproxy.cfg
        mode: '0644'
        validate: 'haproxy -f %s -c'
      notify: restart haproxy
    
    - name: Enable HAProxy stats socket
      file:
        path: /var/run/haproxy
        state: directory
        mode: '0755'
    
    - name: Start HAProxy
      service:
        name: haproxy
        state: started
        enabled: yes
    
    - name: Verify HAProxy is listening
      wait_for:
        port: 80
        timeout: 30

- name: Setup Keepalived for HA
  hosts: loadbalancers
  become: yes
  
  tasks:
    - name: Install Keepalived
      apt:
        name: keepalived
        state: present
    
    - name: Configure Keepalived
      template:
        src: templates/keepalived.conf.j2
        dest: /etc/keepalived/keepalived.conf
        mode: '0644'
      notify: restart keepalived
    
    - name: Start Keepalived
      service:
        name: keepalived
        state: started
        enabled: yes
```

**templates/haproxy.cfg.j2:**

jinja

```jinja
# {{ ansible_managed }}

global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /var/run/haproxy.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

frontend http_front
    bind *:80
    stats uri /haproxy?stats
    default_backend http_back

frontend https_front
    bind *:443 ssl crt /etc/ssl/{{ app_name }}/cert.pem
    default_backend http_back

backend http_back
    balance roundrobin
    option httpchk GET /health
    http-check expect status 200
    
{% for host in groups['webservers'] %}
    server {{ host }} {{ hostvars[host]['ansible_host'] }}:80 check
{% endfor %}

backend app_back
    balance leastconn
    option httpchk GET /api/health
    
{% for host in groups['appservers'] %}
    server {{ host }} {{ hostvars[host]['ansible_host'] }}:8080 check weight {{ hostvars[host]['app_weight'] | default(100) }}
{% endfor %}

listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 30s
    stats auth admin:{{ haproxy_stats_password }}
```

**playbooks/backup.yml:**

yaml

```yaml
---
- name: Setup Backup System
  hosts: backup
  become: yes
  
  tasks:
    - name: Create backup directories
      file:
        path: "{{ item }}"
        state: directory
        mode: '0755'
      loop:
        - /backup/database
        - /backup/application
        - /backup/configs
    
    - name: Install backup tools
      apt:
        name:
          - postgresql-client
          - rsync
          - borgbackup
        state: present
    
    - name: Create database backup script
      template:
        src: templates/backup_database.sh.j2
        dest: /usr/local/bin/backup_database.sh
        mode: '0755'
    
    - name: Create application backup script
      template:
        src: templates/backup_application.sh.j2
        dest: /usr/local/bin/backup_application.sh
        mode: '0755'
    
    - name: Setup backup cron jobs
      cron:
        name: "{{ item.name }}"
        minute: "{{ item.minute }}"
        hour: "{{ item.hour }}"
        job: "{{ item.job }}"
      loop:
        - name: "Database backup"
          minute: "0"
          hour: "2"
          job: "/usr/local/bin/backup_database.sh"
        - name: "Application backup"
          minute: "0"
          hour: "3"
          job: "/usr/local/bin/backup_application.sh"
    
    - name: Setup backup retention script
      copy:
        content: |
          #!/bin/bash
          # Keep last 7 daily backups
          find /backup/database -name "*.sql.gz" -mtime +7 -delete
          find /backup/application -name "*.tar.gz" -mtime +7 -delete
          
          # Keep last 4 weekly backups
          find /backup/database -name "*weekly*.sql.gz" -mtime +28 -delete
          
          # Keep last 3 monthly backups
          find /backup/database -name "*monthly*.sql.gz" -mtime +90 -delete
        dest: /usr/local/bin/cleanup_backups.sh
        mode: '0755'
    
    - name: Setup cleanup cron
      cron:
        name: "Backup cleanup"
        minute: "0"
        hour: "4"
        job: "/usr/local/bin/cleanup_backups.sh"

- name: Configure backup on database servers
  hosts: databases
  become: yes
  
  tasks:
    - name: Create postgres backup user
      postgresql_user:
        name: backup_user
        password: "{{ backup_user_password }}"
        role_attr_flags: REPLICATION
        state: present
      become_user: postgres
      when: pg_role == 'primary'
      no_log: true
```

**templates/backup_database.sh.j2:**

bash

```bash
#!/bin/bash
# {{ ansible_managed }}

set -e

BACKUP_DIR="/backup/database"
DATE=$(date +%Y%m%d_%H%M%S)
DAY_OF_WEEK=$(date +%u)
DAY_OF_MONTH=$(date +%d)

DB_HOST="{{ hostvars[groups['databases'][0]]['ansible_host'] }}"
DB_NAME="{{ app_db_name }}"
DB_USER="backup_user"

export PGPASSWORD="{{ backup_user_password }}"

# Daily backup
BACKUP_FILE="${BACKUP_DIR}/backup_${DATE}.sql.gz"
pg_dump -h ${DB_HOST} -U ${DB_USER} -d ${DB_NAME} | gzip > ${BACKUP_FILE}

# Weekly backup (on Sunday)
if [ "$DAY_OF_WEEK" -eq "7" ]; then
    cp ${BACKUP_FILE} ${BACKUP_DIR}/backup_weekly_${DATE}.sql.gz
fi

# Monthly backup (on 1st)
if [ "$DAY_OF_MONTH" -eq "01" ]; then
    cp ${BACKUP_FILE} ${BACKUP_DIR}/backup_monthly_${DATE}.sql.gz
fi

# Verify backup
if [ -f "${BACKUP_FILE}" ] && [ -s "${BACKUP_FILE}" ]; then
    echo "$(date): Backup successful - ${BACKUP_FILE}" >> /var/log/backup.log
    
    # Optional: upload to S3 or remote storage
    {% if backup_s3_enabled | default(false) %}
    aws s3 cp ${BACKUP_FILE} s3://{{ backup_s3_bucket }}/database/ --storage-class GLACIER
    {% endif %}
else
    echo "$(date): Backup FAILED - ${BACKUP_FILE}" >> /var/log/backup.log
    exit 1
fi
```

**playbooks/rollback.yml:**

yaml

```yaml
---
- name: Rollback Deployment
  hosts: appservers
  become: yes
  serial: 1
  
  vars:
    rollback_version: "{{ rollback_to | default('previous') }}"
  
  pre_tasks:
    - name: Confirm rollback
      pause:
        prompt: "Are you sure you want to rollback to {{ rollback_version }}? (yes/no)"
      when: force_rollback is not defined
    
    - name: Get available backups
      find:
        paths: /backup/application
        patterns: "*.tar.gz"
      register: available_backups
      delegate_to: "{{ groups['backup'][0] }}"
    
    - name: Determine backup to restore
      set_fact:
        backup_file: "{{ available_backups.files | sort(attribute='mtime', reverse=true) | first }}"
      when: rollback_version == 'previous'
  
  tasks:
    - name: Remove from load balancer
      haproxy:
        state: disabled
        host: "{{ inventory_hostname }}"
        socket: /var/run/haproxy.sock
      delegate_to: "{{ groups['loadbalancers'][0] }}"
    
    - name: Stop application
      service:
        name: "{{ app_name }}"
        state: stopped
    
    - name: Backup current state
      archive:
        path: /opt/application
        dest: "/backup/pre-rollback-{{ ansible_date_time.epoch }}.tar.gz"
      delegate_to: "{{ groups['backup'][0] }}"
    
    - name: Remove current application
      file:
        path: /opt/application
        state: absent
    
    - name: Restore from backup
      unarchive:
        src: "{{ backup_file.path }}"
        dest: /opt
        remote_src: yes
      delegate_to: "{{ groups['backup'][0] }}"
    
    - name: Restore database if needed
      command: >
        psql -h {{ hostvars[groups['databases'][0]]['ansible_host'] }}
        -U {{ app_db_user }}
        -d {{ app_db_name }}
        -f /backup/database/rollback.sql
      when: rollback_database | default(false)
    
    - name: Start application
      service:
        name: "{{ app_name }}"
        state: started
    
    - name: Wait for application
      wait_for:
        port: 8080
        timeout: 60
    
    - name: Health check
      uri:
        url: http://localhost:8080/health
        status_code: 200
      register: health
      until: health.status == 200
      retries: 10
      delay: 5
    
    - name: Add back to load balancer
      haproxy:
        state: enabled
        host: "{{ inventory_hostname }}"
        socket: /var/run/haproxy.sock
      delegate_to: "{{ groups['loadbalancers'][0] }}"
    
    - name: Log rollback
      lineinfile:
        path: /var/log/deployments.log
        line: "{{ ansible_date_time.iso8601 }} | ROLLBACK | {{ rollback_version }} | {{ lookup('env', 'USER') }}"
        create: yes
```

**playbooks/healthcheck.yml:**

yaml

```yaml
---
- name: Comprehensive Health Check
  hosts: all
  gather_facts: yes
  
  tasks:
    - name: Check system resources
      assert:
        that:
          - ansible_memfree_mb > 500
          - ansible_mounts | selectattr('mount', 'equalto', '/') | map(attribute='size_available') | first > 5000000000
        fail_msg: "System resources low"
        success_msg: "System resources OK"
    
    - name: Check critical services
      service_facts:
    
    - name: Verify services running
      assert:
        that:
          - ansible_facts.services[item + '.service'].state == 'running'
        fail_msg: "Service {{ item }} not running"
      loop: "{{ required_services | default([]) }}"
      when: item + '.service' in ansible_facts.services

- name: Check Database Health
  hosts: databases
  become: yes
  
  tasks:
    - name: Check PostgreSQL is accepting connections
      postgresql_ping:
        db: "{{ app_db_name }}"
      become_user: postgres
    
    - name: Check replication lag
      postgresql_query:
        db: postgres
        query: "SELECT EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp())) AS lag;"
      become_user: postgres
      register: repl_lag
      when: pg_role == 'standby'
    
    - name: Verify replication lag is acceptable
      assert:
        that:
          - repl_lag.query_result[0].lag | float < 60
        fail_msg: "Replication lag too high: {{ repl_lag.query_result[0].lag }}s"
      when: pg_role == 'standby'

- name: Check Application Health
  hosts: appservers
  
  tasks:
    - name: Check application endpoint
      uri:
        url: http://localhost:8080/health
        return_content: yes
      register: app_health
    
    - name: Verify application health
      assert:
        that:
          - app_health.status == 200
          - "'healthy' in app_health.content"
        fail_msg: "Application health check failed"

- name: Check Load Balancer Health
  hosts: loadbalancers
  
  tasks:
    - name: Get HAProxy stats
      shell: echo "show stat" | socat stdio /var/run/haproxy.sock
      register: haproxy_stats
      changed_when: false
    
    - name: Check for DOWN backends
      assert:
        that:
          - "'DOWN' not in haproxy_stats.stdout"
        fail_msg: "Some backends are DOWN"
        success_msg: "All backends are UP"

- name: Generate Health Report
  hosts: localhost
  gather_facts: no
  
  tasks:
    - name: Create health report
      copy:
        content: |
          # Infrastructure Health Report
          Generated: {{ ansible_date_time.iso8601 }}
          
          ## Summary
          - Total Hosts: {{ groups['all'] | length }}
          - Web Servers: {{ groups['webservers'] | length }}
          - App Servers: {{ groups['appservers'] | length }}
          - Databases: {{ groups['databases'] | length }}
          - Cache Servers: {{ groups['cache'] | length }}
          
          ## Status
          All systems operational ✓
          
        dest: "reports/health_{{ ansible_date_time.epoch }}.md"
```

**scripts/deploy.sh:**

bash

```bash
#!/bin/bash
# Deployment wrapper script

set -e

# Configuration
ENVIRONMENT="${1:-staging}"
VERSION="${2:-$(git rev-parse --short HEAD)}"
ANSIBLE_OPTS=""

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

function log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

function log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

function log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Pre-deployment checks
log_info "Starting deployment to ${ENVIRONMENT}"
log_info "Version: ${VERSION}"

# Validate environment
if [[ ! -f "inventories/${ENVIRONMENT}/hosts.yml" ]]; then
    log_error "Environment ${ENVIRONMENT} not found"
    exit 1
fi

# Check git status
if [[ $(git status --porcelain | wc -l) -gt 0 ]]; then
    log_warn "You have uncommitted changes"
    read -p "Continue? (yes/no): " confirm
    if [[ "$confirm" != "yes" ]]; then
        exit 0
    fi
fi

# Syntax check
log_info "Running syntax check..."
ansible-playbook playbooks/deploy.yml --syntax-check

# Pre-deployment backup
log_info "Creating pre-deployment backup..."
ansible-playbook playbooks/backup.yml \
    -i inventories/${ENVIRONMENT}/hosts.yml \
    --tags database

# Confirm deployment
if [[ "${ENVIRONMENT}" == "production" ]]; then
    echo ""
    log_warn "=== PRODUCTION DEPLOYMENT ==="
    echo "Environment: ${ENVIRONMENT}"
    echo "Version: ${VERSION}"
    echo ""
    read -p "Are you absolutely sure? (type 'yes' to continue): " confirm
    if [[ "$confirm" != "yes" ]]; then
        log_info "Deployment cancelled"
        exit 0
    fi
fi

# Deploy
log_info "Starting deployment..."
ansible-playbook playbooks/deploy.yml \
    -i inventories/${ENVIRONMENT}/hosts.yml \
    -e "deployment_version=${VERSION}" \
    -e "environment=${ENVIRONMENT}" \
    ${ANSIBLE_OPTS}

if [[ $? -eq 0 ]]; then
    log_info "Deployment completed successfully!"
    
    # Run health checks
    log_info "Running post-deployment health checks..."
    ansible-playbook playbooks/healthcheck.yml \
        -i inventories/${ENVIRONMENT}/hosts.yml
    
    # Generate deployment report
    cat << EOF > "reports/deployment_${ENVIRONMENT}_${VERSION}.md"
# Deployment Report

- **Environment**: ${ENVIRONMENT}
- **Version**: ${VERSION}
- **Date**: $(date -Iseconds)
- **User**: ${USER}
- **Status**: SUCCESS

## Changes
$(git log --oneline -5)

## Health Check
All systems operational ✓
EOF
    
    log_info "Deployment report created: reports/deployment_${ENVIRONMENT}_${VERSION}.md"
else
    log_error "Deployment failed!"
    log_error "Check logs and run rollback if needed:"
    log_error "  ansible-playbook playbooks/rollback.yml -i inventories/${ENVIRONMENT}/hosts.yml"
    exit 1
fi
```

**Makefile:**

makefile

```makefile
.PHONY: help validate test deploy-staging deploy-production backup rollback docs

.DEFAULT_GOAL := help

help: ## Show this help message
	@echo 'Usage: make [target]'
	@echo ''
	@echo 'Available targets:'
	@awk 'BEGIN {FS = ":.*?## "} /^[a-zA-Z_-]+:.*?## / {printf "  %-20s %s\n", $$1, $$2}' $(MAKEFILE_LIST)

validate: ## Validate all playbooks
	@echo "Running validation..."
	@ansible-playbook playbooks/*.yml --syntax-check
	@ansible-lint playbooks/ || true
	@yamllint . || true

test: ## Run tests
	@echo "Running tests..."
	@ansible-playbook tests/test_deployment.yml -i inventories/staging/hosts.yml --check

deploy-staging: ## Deploy to staging
	@./scripts/deploy.sh staging

deploy-production: ## Deploy to production
	@./scripts/deploy.sh production

backup: ## Run backup
	@ansible-playbook playbooks/backup.yml -i inventories/production/hosts.yml

rollback: ## Rollback last deployment
	@ansible-playbook playbooks/rollback.yml -i inventories/production/hosts.yml

health: ## Run health check
	@ansible-playbook playbooks/healthcheck.yml -i inventories/production/hosts.yml

docs: ## Generate documentation
	@echo "Generating documentation..."
	@ansible-playbook-grapher playbooks/site.yml --output-file-name docs/infrastructure.png

clean: ## Clean temporary files
	@find . -name "*.retry" -delete
	@find . -name "__pycache__" -type d -exec rm -rf {} +
	@find . -name "*.pyc" -delete

setup: ## Initial setup
	@echo "Installing requirements..."
	@pip install -r requirements.txt
	@ansible-galaxy install -r requirements.yml
	@pre-commit install
```

**docs/RUNBOOK.md:**

markdown

````markdown
# Operations Runbook

## Table of Contents
1. [Deployment](#deployment)
2. [Rollback](#rollback)
3. [Monitoring](#monitoring)
4. [Troubleshooting](#troubleshooting)
5. [Disaster Recovery](#disaster-recovery)

## Deployment

### Normal Deployment
```bash
# Deploy to staging
make deploy-staging

# Deploy to production
make deploy-production
```

### Emergency Hotfix
```bash
# Create hotfix branch
git checkout -b hotfix/critical-fix

# Make changes and commit
git add .
git commit -m "Fix critical issue"

# Deploy immediately
./scripts/deploy.sh production $(git rev-parse --short HEAD)
```

## Rollback

### Quick Rollback
```bash
# Rollback to previous version
ansible-playbook playbooks/rollback.yml \
    -i inventories/production/hosts.yml

# Rollback to specific version
ansible-playbook playbooks/rollback.yml \
    -i inventories/production/hosts.yml \
    -e "rollback_to=v1.2.3"
```

## Monitoring

### Health Checks
```bash
# Full health check
make health

# Check specific tier
ansible-playbook playbooks/healthcheck.yml \
    -i inventories/production/hosts.yml \
    --limit databases
```

### Log Analysis
```bash
# View application logs
ansible appservers -i inventories/production/hosts.yml \
    -a "tail -100 /var/log/application/app.log"

# Check for errors
ansible all -i inventories/production/hosts.yml \
    -m shell -a "journalctl -p err -n 50"
```

## Troubleshooting

### Application Issues

**Symptom**: High response times

1. Check load balancer:
```bash
ansible loadbalancers -a "echo 'show stat' | socat stdio /var/run/haproxy.sock"
```

2. Check application servers:
```bash
ansible appservers -m shell -a "ps aux --sort=-%cpu | head -20"
```

3. Check database:
```bash
ansible databases -m shell -a "pg_stat_activity" -become_user postgres
```

**Symptom**: Service down

1. Check service status:
```bash
ansible appservers -a "systemctl status {{ app_name }}"
```

2. Check logs:
```bash
ansible appservers -a "journalctl -u {{ app_name }} -n 100"
```

3. Restart if needed:
```bash
ansible appservers -a "systemctl restart {{ app_name }}"
```

### Database Issues

**Symptom**: Replication lag

1. Check lag:
```bash
ansible databases -m postgresql_query \
    -a "query='SELECT * FROM pg_stat_replication;'" \
    -become_user postgres
```

2. If lag > 60s, investigate:
```bash
# Check network
ansible databases -m shell -a "ping -c 5 {{ primary_db_host }}"

# Check disk I/O
ansible databases -m shell -a "iostat -x 1 5"
```

## Disaster Recovery

### Full System Recovery

1. Restore database:
```bash
ansible-playbook playbooks/restore.yml \
    -i inventories/production/hosts.yml \
    -e "restore_date=2024-01-15"
```

2. Verify restore:
```bash
ansible databases -m postgresql_query \
    -a "query='SELECT count(*) FROM users;'" \
    -become_user postgres
```

3. Deploy application:
```bash
make deploy-production
```

### Single Server Recovery

1. Rebuild server from backup:
```bash
ansible-playbook playbooks/site.yml \
    -i inventories/production/hosts.yml \
    --limit failed_server
```

2. Restore data:
```bash
ansible-playbook playbooks/restore.yml \
    -i inventories/production/hosts.yml \
    --limit failed_server
```

3. Add back to cluster:
```bash
ansible-playbook playbooks/healthcheck.yml \
    -i inventories/production/hosts.yml \
    --limit failed_server
```

### Database Failover

**Manual failover to standby:**

```bash
# 1. Promote standby to primary
ansible databases -m shell \
    -a "pg_ctl promote -D /var/lib/postgresql/14/main" \
    -become_user postgres \
    --limit standby_server

# 2. Update application config
ansible-playbook playbooks/update_db_config.yml \
    -e "primary_db={{ new_primary_ip }}"

# 3. Restart applications
ansible appservers -a "systemctl restart {{ app_name }}"
```

## Contacts

- **On-Call**: +1-555-0100
- **Slack**: #devops-alerts
- **Email**: devops@example.com
- **Runbook Updates**: Update this file and commit to git
```

**docs/ARCHITECTURE.md:**

```markdown
# Infrastructure Architecture

## Overview

```
                    ┌─────────────┐
                    │   Users     │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Internet   │
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              │                         │
        ┌─────▼─────┐           ┌──────▼──────┐
        │    LB1    │◄─────────►│     LB2     │
        │ (HAProxy) │  Keepalived │  (HAProxy)  │
        └─────┬─────┘           └──────┬──────┘
              │                         │
              └────────────┬────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
    ┌─────▼─────┐   ┌──────▼──────┐  ┌─────▼─────┐
    │   Web1    │   │    Web2     │  │   Web3    │
    │  (Nginx)  │   │   (Nginx)   │  │  (Nginx)  │
    └─────┬─────┘   └──────┬──────┘  └─────┬─────┘
          │                │                │
          └────────────────┼────────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
    ┌─────▼─────┐   ┌──────▼──────┐  ┌─────▼─────┐
    │   App1    │   │    App2     │  │   App3    │
    │  (Python) │   │  (Python)   │  │  (Python) │
    └─────┬─────┘   └──────┬──────┘  └─────┬─────┘
          │                │                │
          └────────────────┼────────────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
        ┌─────▼─────┐           ┌──────▼──────┐
        │    DB1    │◄─────────►│     DB2     │
        │ (Primary) │ Replication│  (Standby)  │
        │PostgreSQL │           │ PostgreSQL  │
        └─────┬─────┘           └──────┬──────┘
              │                         │
              └────────────┬────────────┘
                           │
              ┌────────────▼────────────┐
              │      Redis Cluster      │
              │   Master + 2 Slaves     │
              │     + Sentinel          │
              └─────────────────────────┘
```

## Components

### Load Balancers
- **Technology**: HAProxy + Keepalived
- **Purpose**: Distribute traffic, provide HA
- **Ports**: 80 (HTTP), 443 (HTTPS), 8404 (Stats)

### Web Tier
- **Technology**: Nginx
- **Purpose**: Static content, SSL termination, reverse proxy
- **Scaling**: Horizontal (3+ servers)

### Application Tier
- **Technology**: Python/Flask, Gunicorn
- **Purpose**: Business logic
- **Scaling**: Horizontal (3+ servers)
- **Health Check**: `/api/health`

### Database Tier
- **Technology**: PostgreSQL 14
- **Architecture**: Primary-Standby replication
- **Backup**: Daily full, WAL archiving
- **Recovery Time**: < 30 minutes

### Cache Tier
- **Technology**: Redis 7
- **Architecture**: Master-Slave with Sentinel
- **Purpose**: Session storage, application cache
- **Persistence**: AOF + RDB

### Monitoring
- **Metrics**: Prometheus + Grafana
- **Logs**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **Alerts**: AlertManager → Slack/Email

## Network

### Subnets
- `10.0.1.0/24` - Load Balancers
- `10.0.2.0/24` - Web Tier
- `10.0.3.0/24` - Application Tier
- `10.0.4.0/24` - Database Tier
- `10.0.5.0/24` - Cache Tier
- `10.0.6.0/24` - Monitoring
- `10.0.7.0/24` - Backup

### Firewall Rules
```
LB → Web:    TCP/80, TCP/443
Web → App:   TCP/8080
App → DB:    TCP/5432
App → Cache: TCP/6379
All → Mon:   TCP/9100
```

## Disaster Recovery

### RPO (Recovery Point Objective)
- Database: 5 minutes (via WAL)
- Application: 1 hour (via backups)
- Config: Real-time (via Git)

### RTO (Recovery Time Objective)
- Database failover: < 2 minutes (automated)
- Full recovery: < 30 minutes (with backups)
- Single server: < 15 minutes (rebuild)

## Capacity Planning

### Current Capacity
- **Web**: 3 servers × 100 req/s = 300 req/s
- **App**: 3 servers × 50 req/s = 150 req/s
- **DB**: 1000 connections, 10k TPS

### Scaling Triggers
- CPU > 70% sustained
- Memory > 85%
- Response time > 500ms (p95)
- Error rate > 1%
```

**playbooks/restore.yml:**

```yaml
---
- name: Restore from Backup
  hosts: "{{ target_host | default('all') }}"
  become: yes
  gather_facts: yes
  
  vars:
    restore_date: "{{ restore_date | default('latest') }}"
    backup_server: "{{ groups['backup'][0] }}"
  
  pre_tasks:
    - name: Confirm restore
      pause:
        prompt: |
          WARNING: This will restore data from backup!
          Target: {{ target_host | default('all') }}
          Date: {{ restore_date }}
          
          This will OVERWRITE current data!
          Type 'yes' to continue
      register: confirmation
      when: force_restore is not defined
    
    - name: Validate confirmation
      fail:
        msg: "Restore cancelled"
      when: 
        - force_restore is not defined
        - confirmation.user_input != 'yes'

- name: Restore Database
  hosts: databases
  become: yes
  serial: 1
  
  tasks:
    - name: Stop PostgreSQL
      service:
        name: postgresql
        state: stopped
    
    - name: Find backup file
      find:
        paths: /backup/database
        patterns: "backup_{{ restore_date }}*.sql.gz"
      register: backup_files
      delegate_to: "{{ backup_server }}"
      when: restore_date != 'latest'
    
    - name: Get latest backup
      find:
        paths: /backup/database
        patterns: "backup_*.sql.gz"
      register: all_backups
      delegate_to: "{{ backup_server }}"
      when: restore_date == 'latest'
    
    - name: Set backup file
      set_fact:
        restore_file: "{{ (all_backups.files | sort(attribute='mtime', reverse=true) | first).path if restore_date == 'latest' else backup_files.files[0].path }}"
    
    - name: Copy backup to database server
      copy:
        src: "{{ restore_file }}"
        dest: /tmp/restore.sql.gz
        remote_src: yes
      delegate_to: "{{ backup_server }}"
    
    - name: Drop existing database
      postgresql_db:
        name: "{{ app_db_name }}"
        state: absent
      become_user: postgres
    
    - name: Create new database
      postgresql_db:
        name: "{{ app_db_name }}"
        state: present
      become_user: postgres
    
    - name: Restore database
      shell: |
        gunzip -c /tmp/restore.sql.gz | psql -d {{ app_db_name }}
      become_user: postgres
    
    - name: Verify restore
      postgresql_query:
        db: "{{ app_db_name }}"
        query: "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = 'public';"
      become_user: postgres
      register: table_count
    
    - name: Display restore info
      debug:
        msg: |
          Database restored successfully
          Tables: {{ table_count.query_result[0].count }}
          Backup: {{ restore_file }}
    
    - name: Start PostgreSQL
      service:
        name: postgresql
        state: started
    
    - name: Cleanup
      file:
        path: /tmp/restore.sql.gz
        state: absent

- name: Restore Application
  hosts: appservers
  become: yes
  serial: 1
  
  tasks:
    - name: Find application backup
      find:
        paths: /backup/application
        patterns: "backup_{{ restore_date }}*.tar.gz"
      register: app_backup
      delegate_to: "{{ backup_server }}"
      when: restore_date != 'latest'
    
    - name: Get latest application backup
      find:
        paths: /backup/application
        patterns: "backup_*.tar.gz"
      register: all_app_backups
      delegate_to: "{{ backup_server }}"
      when: restore_date == 'latest'
    
    - name: Set application backup file
      set_fact:
        app_restore_file: "{{ (all_app_backups.files | sort(attribute='mtime', reverse=true) | first).path if restore_date == 'latest' else app_backup.files[0].path }}"
    
    - name: Stop application
      service:
        name: "{{ app_name }}"
        state: stopped
    
    - name: Backup current state
      archive:
        path: /opt/application
        dest: "/tmp/pre-restore-{{ ansible_date_time.epoch }}.tar.gz"
    
    - name: Remove current application
      file:
        path: /opt/application
        state: absent
    
    - name: Restore application
      unarchive:
        src: "{{ app_restore_file }}"
        dest: /opt
        remote_src: yes
      delegate_to: "{{ backup_server }}"
    
    - name: Set permissions
      file:
        path: /opt/application
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        recurse: yes
    
    - name: Start application
      service:
        name: "{{ app_name }}"
        state: started
    
    - name: Wait for application
      wait_for:
        port: 8080
        timeout: 60
    
    - name: Health check
      uri:
        url: http://localhost:8080/health
        status_code: 200
      register: health
      until: health.status == 200
      retries: 10
      delay: 5

- name: Post-Restore Verification
  hosts: all
  gather_facts: no
  
  tasks:
    - name: Run health checks
      include_tasks: healthcheck.yml
    
    - name: Generate restore report
      copy:
        content: |
          # Restore Report
          
          Date: {{ ansible_date_time.iso8601 }}
          User: {{ lookup('env', 'USER') }}
          Restore Point: {{ restore_date }}
          
          ## Restored Components
          - Database: ✓
          - Application: ✓
          - Configuration: ✓
          
          ## Verification
          All health checks passed ✓
          
          ## Next Steps
          1. Verify application functionality
          2. Check recent transactions
          3. Inform stakeholders
          4. Update documentation
          
        dest: "reports/restore_{{ ansible_date_time.epoch }}.md"
      delegate_to: localhost
      run_once: true
```

---

## Модуль 9: Advanced Topics и Modern Practices (30 минут)

### 🎯 Напоминалка

**Ansible Collections:**

Collections - это дистрибутивный формат для Ansible контента, который может включать роли, модули, плагины и многое другое.

```bash
# Установка коллекций
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.docker

# Из requirements.yml
cat > requirements.yml <<EOF
---
collections:
  - name: community.general
    version: ">=5.0.0"
  - name: community.docker
  - name: ansible.posix
  - name: amazon.aws
EOF

ansible-galaxy collection install -r requirements.yml

# Список установленных коллекций
ansible-galaxy collection list
```

**Использование коллекций в playbooks:**

```yaml
---
# Полное имя модуля
- name: Use collection module
  hosts: localhost
  tasks:
    - name: Docker container
      community.docker.docker_container:
        name: nginx
        image: nginx:latest
        state: started

# Импорт коллекции
- name: Use collection with import
  hosts: localhost
  collections:
    - community.docker
  
  tasks:
    - name: Docker container
      docker_container:  # Короткое имя
        name: nginx
        image: nginx:latest
```

**Ansible Navigator (замена ansible-playbook):**

```bash
# Установка
pip install ansible-navigator

# Запуск playbook
ansible-navigator run playbook.yml

# Интерактивный режим
ansible-navigator

# С execution environment
ansible-navigator run playbook.yml \
  --execution-environment-image quay.io/ansible/creator-ee:latest

# Replay mode (просмотр прошлых запусков)
ansible-navigator replay artifacts/playbook-artifact.json
```

**Execution Environments (EE):**

Execution Environments - это контейнерные образы, которые содержат Ansible и все зависимости.

```yaml
# execution-environment.yml
---
version: 3

images:
  base_image:
    name: quay.io/ansible/creator-base:latest

dependencies:
  galaxy: requirements.yml
  python: requirements.txt
  system: bindep.txt

additional_build_steps:
  append_final:
    - RUN echo "Custom build step"
```

```bash
# Создание EE
ansible-builder build -t my-ee:latest

# Использование
ansible-navigator run playbook.yml \
  --execution-environment-image my-ee:latest
```

**Ansible Content Collections - создание своей:**

```bash
# Создать структуру коллекции
ansible-galaxy collection init mycompany.infrastructure

# Структура:
# mycompany/infrastructure/
# ├── docs/
# ├── galaxy.yml
# ├── plugins/
# │   ├── modules/
# │   ├── inventory/
# │   ├── filter/
# │   └── callback/
# ├── roles/
# ├── playbooks/
# └── tests/
```

**galaxy.yml:**

```yaml
namespace: mycompany
name: infrastructure
version: 1.0.0
readme: README.md
authors:
  - DevOps Team <devops@mycompany.com>
description: Infrastructure automation collection
license:
  - MIT
tags:
  - infrastructure
  - networking
  - cloud
dependencies:
  community.general: ">=5.0.0"
  ansible.posix: "*"
repository: https://github.com/mycompany/infrastructure-collection
documentation: https://docs.mycompany.com/ansible
homepage: https://www.mycompany.com
issues: https://github.com/mycompany/infrastructure-collection/issues
```

**Custom Module пример:**

```python
# plugins/modules/custom_health_check.py
#!/usr/bin/python

from ansible.module_utils.basic import AnsibleModule
import requests

DOCUMENTATION = r'''
---
module: custom_health_check
short_description: Custom health check module
description:
  - Performs HTTP health check on specified endpoint
  - Returns detailed health information
options:
  url:
    description: URL to check
    required: true
    type: str
  timeout:
    description: Request timeout in seconds
    required: false
    type: int
    default: 10
  expected_status:
    description: Expected HTTP status code
    required: false
    type: int
    default: 200
author:
  - DevOps Team
'''

EXAMPLES = r'''
- name: Check application health
  mycompany.infrastructure.custom_health_check:
    url: http://localhost:8080/health
    timeout: 5
    expected_status: 200
'''

RETURN = r'''
status_code:
  description: HTTP status code returned
  type: int
  returned: always
response_time:
  description: Response time in seconds
  type: float
  returned: always
healthy:
  description: Whether service is healthy
  type: bool
  returned: always
'''

def run_module():
    module_args = dict(
        url=dict(type='str', required=True),
        timeout=dict(type='int', required=False, default=10),
        expected_status=dict(type='int', required=False, default=200)
    )

    result = dict(
        changed=False,
        status_code=0,
        response_time=0.0,
        healthy=False
    )

    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    if module.check_mode:
        module.exit_json(**result)

    try:
        import time
        start_time = time.time()
        
        response = requests.get(
            module.params['url'],
            timeout=module.params['timeout']
        )
        
        result['response_time'] = time.time() - start_time
        result['status_code'] = response.status_code
        result['healthy'] = (response.status_code == module.params['expected_status'])
        
        if not result['healthy']:
            module.fail_json(
                msg=f"Health check failed. Expected {module.params['expected_status']}, got {response.status_code}",
                **result
            )
        
        module.exit_json(**result)
        
    except Exception as e:
        module.fail_json(msg=str(e), **result)

def main():
    run_module()

if __name__ == '__main__':
    main()
```

**Custom Filter Plugin:**

```python
# plugins/filter/custom_filters.py

def calculate_percentage(value, total):
    """Calculate percentage"""
    if total == 0:
        return 0
    return round((value / total) * 100, 2)

def bytes_to_human(bytes_value):
    """Convert bytes to human readable format"""
    for unit in ['B', 'KB', 'MB', 'GB', 'TB']:
        if bytes_value < 1024.0:
            return f"{bytes_value:.2f}{unit}"
        bytes_value /= 1024.0
    return f"{bytes_value:.2f}PB"

class FilterModule(object):
    def filters(self):
        return {
            'calculate_percentage': calculate_percentage,
            'bytes_to_human': bytes_to_human
        }
```

**Использование:**

```yaml
---
- name: Use custom filters
  hosts: localhost
  
  tasks:
    - name: Calculate percentages
      debug:
        msg: "{{ 50 | calculate_percentage(200) }}% complete"
    
    - name: Convert bytes
      debug:
        msg: "Size: {{ 1073741824 | bytes_to_human }}"
```

**Ansible Test (ansible-test):**

```bash
# Запуск sanity тестов
ansible-test sanity --docker

# Запуск unit тестов
ansible-test units --docker

# Запуск integration тестов
ansible-test integration --docker

# Для конкретной коллекции
cd mycompany/infrastructure
ansible-test sanity --docker default
```

### 💻 Задание

Создай современную Ansible Collection с best practices:

**1. Создай структуру коллекции:**

```bash
mkdir -p ansible-collections
cd ansible-collections
ansible-galaxy collection init devops.platform
cd devops/platform
```

**2. Обнови galaxy.yml:**

```yaml
namespace: devops
name: platform
version: 1.0.0
readme: README.md
authors:
  - DevOps Team
description: Platform engineering collection
license:
  - MIT
tags:
  - platform
  - kubernetes
  - monitoring
dependencies:
  community.general: ">=5.0.0"
  kubernetes.core: ">=2.3.0"
```

**3. Создай custom module:**

```python
# plugins/modules/service_health.py
#!/usr/bin/python

from ansible.module_utils.basic import AnsibleModule
import subprocess
import json

DOCUMENTATION = r'''
---
module: service_health
short_description: Check service health
description:
  - Checks if a service is running and healthy
  - Returns service metrics
options:
  name:
    description: Service name
    required: true
    type: str
  check_port:
    description: Port to check
    required: false
    type: int
'''

def check_service(name):
    try:
        result = subprocess.run(
            ['systemctl', 'is-active', name],
            capture_output=True,
            text=True
        )
        return result.returncode == 0
    except Exception:
        return False

def get_service_info(name):
    try:
        result = subprocess.run(
            ['systemctl', 'show', name, '--no-pager'],
            capture_output=True,
            text=True
        )
        info = {}
        for line in result.stdout.split('\n'):
            if '=' in line:
                key, value = line.split('=', 1)
                info[key] = value
        return info
    except Exception:
        return {}

def run_module():
    module_args = dict(
        name=dict(type='str', required=True),
        check_port=dict(type='int', required=False)
    )

    result = dict(
        changed=False,
        is_running=False,
        service_info={}
    )

    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    service_name = module.params['name']
    
    result['is_running'] = check_service(service_name)
    result['service_info'] = get_service_info(service_name)
    
    if module.params.get('check_port'):
        import socket
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(1)
        port_open = sock.connect_ex(('localhost', module.params['check_port'])) == 0
        sock.close()
        result['port_open'] = port_open
    
    module.exit_json(**result)

def main():
    run_module()

if __name__ == '__main__':
    main()
```

**4. Создай роль в коллекции:**

```bash
ansible-galaxy role init roles/webapp --init-path roles/
```

**roles/webapp/tasks/main.yml:**

```yaml
---
- name: Install application dependencies
  package:
    name: "{{ item }}"
    state: present
  loop: "{{ webapp_dependencies }}"

- name: Create application user
  user:
    name: "{{ webapp_user }}"
    system: yes
    shell: /bin/false

- name: Deploy application
  copy:
    src: "{{ webapp_source }}"
    dest: "{{ webapp_dest }}"
    owner: "{{ webapp_user }}"
    mode: '0755'

- name: Create systemd service
  template:
    src: webapp.service.j2
    dest: "/etc/systemd/system/{{ webapp_name }}.service"
  notify: restart webapp

- name: Enable and start service
  systemd:
    name: "{{ webapp_name }}"
    enabled: yes
    state: started
    daemon_reload: yes
```

**5. Создай playbook в коллекции:**

```yaml
# playbooks/deploy_stack.yml
---
- name: Deploy Application Stack
  hosts: all
  become: yes
  
  collections:
    - devops.platform
  
  roles:
    - role: webapp
      vars:
        webapp_name: myapp
        webapp_user: appuser
        webapp_dependencies:
          - python3
          - python3-pip
```

**6. Создай тесты:**

```yaml
# tests/integration/targets/service_health/tasks/main.yml
---
- name: Test service_health module
  block:
    - name: Check service
      devops.platform.service_health:
        name: nginx
      register: result
    
    - name: Verify result
      assert:
        that:
          - result.is_running is defined
          - result.service_info is defined
```

**7. Собери и установи коллекцию:**

```bash
# Собрать
ansible-galaxy collection build

# Установить локально
ansible-galaxy collection install devops-platform-1.0.0.tar.gz

# Или опубликовать на Galaxy
ansible-galaxy collection publish devops-platform-1.0.0.tar.gz --api-key=YOUR_KEY
```

**8. Используй коллекцию:**

```yaml
---
- name: Use custom collection
  hosts: localhost
  
  collections:
    - devops.platform
  
  tasks:
    - name: Check service with custom module
      service_health:
        name: nginx
        check_port: 80
      register: health
    
    - name: Display health
      debug:
        var: health
    
    - name: Deploy using collection role
      include_role:
        name: devops.platform.webapp
```

### 🚀 Бонус (cutting-edge)

**1. Ansible Automation Platform (AAP) интеграция:**

```yaml
# Использование AAP API
---
- name: Trigger AAP Job
  hosts: localhost
  
  tasks:
    - name: Launch job template
      ansible.controller.job_launch:
        controller_host: aap.example.com
        controller_username: admin
        controller_password: "{{ aap_password }}"
        job_template: "Deploy Application"
        extra_vars:
          environment: production
          version: v1.2.3
      register: job
    
    - name: Wait for job
      ansible.controller.job_wait:
        controller_host: aap.example.com
        controller_username: admin
        controller_password: "{{ aap_password }}"
        job_id: "{{ job.id }}"
      register: job_result
    
    - name: Show result
      debug:
        var: job_result
```

**2. Event-Driven Ansible:**

```yaml
# rulebook.yml
---
- name: Respond to webhooks
  hosts: all
  sources:
    - name: webhook
      ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000
  
  rules:
    - name: Deploy on push
      condition: event.payload.ref == "refs/heads/main"
      action:
        run_playbook:
          name: deploy.yml
          extra_vars:
            version: "{{ event.payload.after }}"
    
    - name: Rollback on alert
      condition: event.severity == "critical"
      action:
        run_playbook:
          name: rollback.yml
```

```bash
# Запуск EDA
ansible-rulebook --rulebook rulebook.yml --inventory inventory.yml
```

**3. Ansible Lightspeed (AI-powered):**

```yaml
# С использованием AI suggestions
---
- name: Configure web server
  hosts: webservers
  become: yes
  
  tasks:
    # AI может предложить следующие таски на основе контекста
    - name: Install nginx
      # AI suggestion: apt module with update_cache
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: yes
```

---

## Модуль 10: Практические паттерны и Real-World сценарии (25 минут)

### 🎯 Напоминалка

**Infrastructure as Code (IaC) Best Practices:**

```yaml
# 1. Используй переменные вместо хардкода
# ❌ Плохо
- name: Install package
  apt:
    name: nginx-1.18.0
    state: present

# ✅ Хорошо
- name: Install package
  apt:
    name: "nginx{{ nginx_version | default('') }}"
    state: present

# 2. Делай таски идемпотентными
# ❌ Плохо
- name: Add user
  shell: useradd myuser

# ✅ Хорошо
- name: Add user
  user:
    name: myuser
    state: present

# 3. Используй check mode
# ✅ Поддерживай check mode
- name: Deploy config
  template:
    src: config.j2
    dest: /etc/app/config.yml
  check_mode: yes  # Безопасно для dry-run

# 4. Обрабатывай секреты правильно
# ❌ Плохо
- name: Set password
  user:
    name: admin
    password: "PlainTextPassword"  # Никогда так не делай!

# ✅ Хорошо
- name: Set password
  user:
    name: admin
    password: "{{ admin_password | password_hash('sha512') }}"
  no_log: true  # Не логировать секреты

# 5. Документируй код
- name: Configure application (именно что делаем)
  # Почему: это критично для производительности
  # Зависимости: требует nginx_module_x
  lineinfile:
    path: /etc/app/config
    line: "setting=value"
```

**Паттерн: Multi-Environment Management:**

```yaml
# group_vars/all/common.yml
---
app_name: myapp
app_user: appuser

# Deploy method по умолчанию
deploy_method: git
backup_enabled: true

# group_vars/staging/vars.yml
---
environment: staging
replicas: 1
db_pool_size: 10
log_level: debug
monitoring_interval: 60

# group_vars/production/vars.yml
---
environment: production
replicas: 3
db_pool_size: 50
log_level: warning
monitoring_interval: 10

# Использование
---
- name: Environment-aware deployment
  hosts: all
  tasks:
    - name: Show environment
      debug:
        msg: "Deploying to {{ environment }} with {{ replicas }} replicas"
    
    - name: Deploy with environment-specific settings
      template:
        src: config.j2
        dest: /etc/app/config.yml
      vars:
        # Автоматически использует переменные из group_vars/<environment>/
```

**Паттерн: Zero-Downtime Deployment:**

```yaml
---
- name: Zero-Downtime Deployment
  hosts: appservers
  serial: 1  # По одному за раз
  max_fail_percentage: 0  # Остановиться при любой ошибке
  
  pre_tasks:
    - name: Remove from load balancer
      community.general.haproxy:
        state: disabled
        host: "{{ inventory_hostname }}"
        socket: /var/run/haproxy.sock
        wait: yes
      delegate_to: "{{ groups['loadbalancers'][0] }}"
    
    - name: Wait for connections to drain
      wait_for:
        port: 8080
        state: drained
        delay: 5
        timeout: 60
  
  tasks:
    - name: Deploy new version
      copy:
        src: app-{{ version }}.jar
        dest: /opt/app/app.jar
      notify: restart app
    
    - name: Flush handlers
      meta: flush_handlers
    
    - name: Wait for app to be ready
      uri:
        url: http://localhost:8080/health
        status_code: 200
      register: result
      until: result.status == 200
      retries: 30
      delay: 2
  
  post_tasks:
    - name: Add back to load balancer
      community.general.haproxy:
        state: enabled
        host: "{{ inventory_hostname }}"
        socket: /var/run/haproxy.sock
      delegate_to: "{{ groups['loadbalancers'][0] }}"
    
    - name: Verify traffic is flowing
      pause:
        seconds: 10
    
    - name: Check error rate
      uri:
        url: http://monitoring/api/errors?host={{ inventory_hostname }}
      register: errors
      failed_when: errors.json.rate > 0.01
```

**Паттерн: Canary Releases:**

```yaml
---
- name: Canary Release
  hosts: appservers
  
  vars:
    canary_percentage: 10
    canary_duration: 300  # 5 минут
    error_threshold: 1.0  # 1% ошибок
  
  tasks:
    - name: Identify canary servers
      set_fact:
        canary_servers: "{{ groups['appservers'][:((groups['appservers']|length * canary_percentage / 100)|int)] }}"
      run_once: true
    
    - name: Deploy to canary
      when: inventory_hostname in canary_servers
      block:
        - name: Update application
          copy:
            src: app-{{ new_version }}.jar
            dest: /opt/app/app.jar
          notify: restart app
        
        - name: Flush handlers
          meta: flush_handlers
    
    - name: Monitor canary
      when: inventory_hostname in canary_servers
      block:
        - name: Wait for stabilization
          pause:
            seconds: 30
        
        - name: Monitor metrics
          uri:
            url: "http://monitoring/api/metrics?host={{ inventory_hostname }}"
            return_content: yes
          register: metrics
          until: metrics.json.status == "healthy"
          retries: "{{ canary_duration // 10 }}"
          delay: 10
        
        - name: Check error rate
          uri:
            url: "http://monitoring/api/errors?host={{ inventory_hostname }}"
          register: error_rate
          failed_when: error_rate.json.rate > error_threshold
    
    - name: Deploy to all servers
      when: inventory_hostname not in canary_servers
      copy:
        src: app-{{ new_version }}.jar
        dest: /opt/app/app.jar
      notify: restart app
```

**Паттерн: Dynamic Inventory Integration:**

```python
#!/usr/bin/env python3
# dynamic_inventory.py - Интеграция с облачными API

import json
import requests
import argparse

class DynamicInventory:
    def __init__(self):
        self.inventory = {
            '_meta': {'hostvars': {}},
            'all': {'children': ['ungrouped']}
        }
        self.read_cli_args()
        
        if self.args.list:
            self.inventory = self.get_inventory()
        elif self.args.host:
            self.inventory = self.get_host_info(self.args.host)
        
        print(json.dumps(self.inventory, indent=2))
    
    def read_cli_args(self):
        parser = argparse.ArgumentParser()
        parser.add_argument('--list', action='store_true')
        parser.add_argument('--host', action='store')
        self.args = parser.parse_args()
    
    def get_inventory(self):
        """Получить инвентарь из AWS/GCP/Azure/etc"""
        # Пример с AWS EC2
        inventory = {
            'webservers': {
                'hosts': [],
                'vars': {
                    'ansible_user': 'ubuntu'
                }
            },
            'databases': {
                'hosts': [],
                'vars': {
                    'ansible_user': 'ubuntu'
                }
            },
            '_meta': {
                'hostvars': {}
            }
        }
        
        # Вызов AWS API (упрощенно)
        # import boto3
        # ec2 = boto3.client('ec2')
        # instances = ec2.describe_instances(Filters=[{'Name': 'tag:Environment', 'Values': ['production']}])
        
        # Пример статических данных
        instances = [
            {
                'id': 'i-123',
                'ip': '10.0.1.10',
                'tags': {'Name': 'web1', 'Role': 'webserver'},
                'type': 't3.medium'
            },
            {
                'id': 'i-456',
                'ip': '10.0.2.10',
                'tags': {'Name': 'db1', 'Role': 'database'},
                'type': 't3.large'
            }
        ]
        
        for instance in instances:
            role = instance['tags'].get('Role', 'ungrouped')
            
            if role not in inventory:
                inventory[role] = {'hosts': [], 'vars': {}}
            
            hostname = instance['tags']['Name']
            inventory[role]['hosts'].append(hostname)
            inventory['_meta']['hostvars'][hostname] = {
                'ansible_host': instance['ip'],
                'instance_id': instance['id'],
                'instance_type': instance['type']
            }
        
        return inventory
    
    def get_host_info(self, hostname):
        """Получить информацию о конкретном хосте"""
        inventory = self.get_inventory()
        return inventory['_meta']['hostvars'].get(hostname, {})

if __name__ == '__main__':
    DynamicInventory()
```

**Использование:**

```bash
# Проверка работы
./dynamic_inventory.py --list
./dynamic_inventory.py --host web1

# Использование с Ansible
ansible-playbook -i dynamic_inventory.py playbook.yml

# В ansible.cfg
[defaults]
inventory = ./dynamic_inventory.py
```

**Паттерн: Self-Healing Infrastructure:**

```yaml
---
- name: Self-Healing Monitoring
  hosts: all
  gather_facts: yes
  
  vars:
    health_checks:
      - name: disk_space
        threshold: 90
        action: cleanup
      - name: memory
        threshold: 85
        action: restart_services
      - name: service_health
        services:
          - nginx
          - postgresql
        action: restart
  
  tasks:
    - name: Check disk space
      set_fact:
        disk_usage: "{{ (100 - (ansible_mounts | selectattr('mount', 'equalto', '/') | map(attribute='size_available') | first / ansible_mounts | selectattr('mount', 'equalto', '/') | map(attribute='size_total') | first * 100)) | round(0) | int }}"
    
    - name: Self-heal disk space
      when: disk_usage | int > 90
      block:
        - name: Clean package cache
          apt:
            autoclean: yes
            autoremove: yes
        
        - name: Clean old logs
          find:
            paths: /var/log
            patterns: "*.log.*"
            age: 7d
          register: old_logs
        
        - name: Remove old logs
          file:
            path: "{{ item.path }}"
            state: absent
          loop: "{{ old_logs.files }}"
        
        - name: Send alert
          uri:
            url: "{{ slack_webhook }}"
            method: POST
            body_format: json
            body:
              text: "Auto-cleaned disk space on {{ inventory_hostname }}"
          delegate_to: localhost
    
    - name: Check memory usage
      set_fact:
        memory_usage: "{{ ((ansible_memtotal_mb - ansible_memfree_mb) / ansible_memtotal_mb * 100) | round(0) | int }}"
    
    - name: Self-heal memory
      when: memory_usage | int > 85
      block:
        - name: Restart memory-heavy services
          service:
            name: "{{ item }}"
            state: restarted
          loop:
            - nginx
            - php-fpm
          ignore_errors: yes
        
        - name: Clear page cache
          shell: sync; echo 3 > /proc/sys/vm/drop_caches
          when: memory_usage | int > 90
    
    - name: Check service health
      service_facts:
    
    - name: Auto-restart failed services
      service:
        name: "{{ item }}"
        state: started
      loop: "{{ health_checks | selectattr('name', 'equalto', 'service_health') | map(attribute='services') | first }}"
      when: 
        - item + '.service' in ansible_facts.services
        - ansible_facts.services[item + '.service'].state != 'running'
      register: restarted_services
    
    - name: Alert on restarts
      when: restarted_services.changed
      uri:
        url: "{{ slack_webhook }}"
        method: POST
        body_format: json
        body:
          text: "Auto-restarted services on {{ inventory_hostname }}: {{ restarted_services.results | selectattr('changed') | map(attribute='item') | list | join(', ') }}"
      delegate_to: localhost
```

**Cron для автоматического запуска:**

```bash
# Запускать каждые 5 минут
*/5 * * * * ansible-playbook /opt/ansible/self-healing.yml -i /opt/ansible/inventory.ini >> /var/log/self-healing.log 2>&1
```

### 💻 Задание: Real-World Scenario

Создай complete solution для следующего сценария:

**Задача:** Компания запускает новый микросервис в production. Требуется:
- Blue-Green deployment
- Automated rollback при ошибках
- Slack notifications
- Backup перед deployment
- Health monitoring
- Performance testing

**Решение:**

```yaml
# playbooks/microservice_deploy.yml
---
- name: Microservice Deployment Pipeline
  hosts: localhost
  gather_facts: no
  
  vars:
    service_name: payment-service
    new_version: "{{ version | default('latest') }}"
    health_check_url: "/actuator/health"
    performance_threshold_ms: 500
    error_rate_threshold: 1.0
    
  tasks:
    - name: Pre-deployment checks
      include_tasks: tasks/pre_deployment.yml
    
    - name: Determine deployment strategy
      set_fact:
        blue_env: "{{ 'blue' if active_env == 'green' else 'green' }}"
        green_env: "{{ 'green' if active_env == 'blue' else 'blue' }}"
      vars:
        active_env: "{{ lookup('file', '/tmp/active_env') | default('blue') }}"

- name: Backup Current State
  hosts: "{{ blue_env }}"
  become: yes
  
  tasks:
    - name: Create backup
      archive:
        path: /opt/{{ service_name }}
        dest: "/backup/{{ service_name }}-{{ ansible_date_time.epoch }}.tar.gz"
      delegate_to: "{{ groups['backup'][0] }}"

- name: Deploy to Inactive Environment
  hosts: "{{ green_env }}"
  become: yes
  serial: 1
  
  vars:
    rollback_on_failure: yes
  
  tasks:
    - name: Stop service
      service:
        name: "{{ service_name }}"
        state: stopped
    
    - name: Deploy new version
      block:
        - name: Pull new image
          docker_image:
            name: "myregistry/{{ service_name }}:{{ new_version }}"
            source: pull
          when: deploy_method == 'docker'
        
        - name: Deploy artifact
          copy:
            src: "artifacts/{{ service_name }}-{{ new_version }}.jar"
            dest: "/opt/{{ service_name }}/app.jar"
          when: deploy_method == 'jar'
        
        - name: Update configuration
          template:
            src: application.yml.j2
            dest: "/opt/{{ service_name }}/config/application.yml"
        
        - name: Start service
          service:
            name: "{{ service_name }}"
            state: started
        
        - name: Wait for startup
          wait_for:
            port: 8080
            timeout: 120
      
      rescue:
        - name: Deployment failed
          set_fact:
            deployment_failed: true
        
        - name: Notify failure
          include_tasks: tasks/notify_slack.yml
          vars:
            message: "❌ Deployment failed on {{ inventory_hostname }}"
            color: danger
        
        - name: Rollback
          when: rollback_on_failure
          include_tasks: tasks/rollback.yml

- name: Health and Performance Testing
  hosts: "{{ green_env }}"
  
  tasks:
    - name: Health check
      uri:
        url: "http://{{ inventory_hostname }}:8080{{ health_check_url }}"
        return_content: yes
      register: health
      until: health.status == 200 and health.json.status == "UP"
      retries: 30
      delay: 2
    
    - name: Performance test
      uri:
        url: "http://{{ inventory_hostname }}:8080/api/test"
        return_content: yes
      register: perf_test
      failed_when: false
    
    - name: Validate performance
      assert:
        that:
          - perf_test.elapsed < (performance_threshold_ms / 1000)
        fail_msg: "Response time too high: {{ perf_test.elapsed }}s"
    
    - name: Smoke tests
      include_tasks: tasks/smoke_tests.yml
    
    - name: Load test
      command: >
        ab -n 1000 -c 10 
        http://{{ inventory_hostname }}:8080/api/endpoint
      register: load_test
      delegate_to: localhost
    
    - name: Parse load test results
      set_fact:
        load_test_results: "{{ load_test.stdout | regex_findall('Requests per second:\\s+(\\d+\\.\\d+)') }}"
    
    - name: Verify load test
      assert:
        that:
          - load_test_results[0] | float > 100
        fail_msg: "Load test failed: {{ load_test_results[0] }} req/s"

- name: Switch Traffic
  hosts: localhost
  
  tasks:
    - name: Monitor for errors
      uri:
        url: "http://monitoring/api/errors?service={{ service_name }}&env={{ green_env }}"
      register: error_check
      until: error_check.json.rate < error_rate_threshold
      retries: 10
      delay: 30
    
    - name: Update load balancer
      community.general.haproxy:
        backend: "{{ green_env }}"
        state: enabled
      delegate_to: "{{ groups['loadbalancers'][0] }}"
    
    - name: Wait for traffic stabilization
      pause:
        seconds: 60
    
    - name: Disable old environment
      community.general.haproxy:
        backend: "{{ blue_env }}"
        state: disabled
      delegate_to: "{{ groups['loadbalancers'][0] }}"
    
    - name: Update active environment marker
      copy:
        content: "{{ green_env }}"
        dest: /tmp/active_env

- name: Post-Deployment Validation
  hosts: "{{ green_env }}"
  
  tasks:
    - name: Final health check
      uri:
        url: "http://{{ inventory_hostname }}:8080{{ health_check_url }}"
      register: final_health
      failed_when: final_health.status != 200
    
    - name: Check logs for errors
      shell: |
        tail -100 /var/log/{{ service_name }}/app.log | grep -i error || true
      register: log_errors
      changed_when: false
    
    - name: Verify metrics
      uri:
        url: "http://monitoring/api/metrics?service={{ service_name }}"
      register: metrics
    
    - name: Success notification
      include_tasks: tasks/notify_slack.yml
      vars:
        message: |
          ✅ Deployment Successful!
          Service: {{ service_name }}
          Version: {{ new_version }}
          Environment: {{ green_env }}
          Performance: {{ perf_test.elapsed }}s
          Load Test: {{ load_test_results[0] }} req/s
        color: good
      delegate_to: localhost
      run_once: true

- name: Cleanup
  hosts: "{{ blue_env }}"
  
  tasks:
    - name: Stop old version
      service:
        name: "{{ service_name }}"
        state: stopped
    
    - name: Archive old version
      archive:
        path: /opt/{{ service_name }}
        dest: "/archive/{{ service_name }}-old-{{ ansible_date_time.epoch }}.tar.gz"
```

**tasks/smoke_tests.yml:**

```yaml
---
- name: API Health Check
  uri:
    url: "http://{{ inventory_hostname }}:8080/api/health"
    status_code: 200

- name: Database Connection Test
  uri:
    url: "http://{{ inventory_hostname }}:8080/api/db/check"
    status_code: 200

- name: Cache Connection Test
  uri:
    url: "http://{{ inventory_hostname }}:8080/api/cache/check"
    status_code: 200

- name: External API Test
  uri:
    url: "http://{{ inventory_hostname }}:8080/api/external/check"
    status_code: 200

- name: Authentication Test
  uri:
    url: "http://{{ inventory_hostname }}:8080/api/auth/test"
    method: POST
    body_format: json
    body:
      username: test
      password: test
    status_code: 200

- name: Business Logic Test
  uri:
    url: "http://{{ inventory_hostname }}:8080/api/calculate"
    method: POST
    body_format: json
    body:
      amount: 100
      currency: USD
    status_code: 200
  register: calc_result

- name: Verify calculation
  assert:
    that:
      - calc_result.json.result is defined
      - calc_result.json.result == 100
```

**tasks/rollback.yml:**

```yaml
---
- name: Stop failed service
  service:
    name: "{{ service_name }}"
    state: stopped

- name: Find latest backup
  find:
    paths: /backup
    patterns: "{{ service_name }}-*.tar.gz"
  register: backups
  delegate_to: "{{ groups['backup'][0] }}"

- name: Get latest backup
  set_fact:
    latest_backup: "{{ backups.files | sort(attribute='mtime', reverse=true) | first }}"

- name: Restore from backup
  unarchive:
    src: "{{ latest_backup.path }}"
    dest: /opt
    remote_src: yes
  delegate_to: "{{ groups['backup'][0] }}"

- name: Start restored service
  service:
    name: "{{ service_name }}"
    state: started

- name: Verify rollback
  uri:
    url: "http://{{ inventory_hostname }}:8080{{ health_check_url }}"
    status_code: 200
  register: rollback_health
  until: rollback_health.status == 200
  retries: 10
  delay: 5

- name: Rollback notification
  include_tasks: notify_slack.yml
  vars:
    message: "⚠️ Rolled back {{ service_name }} on {{ inventory_hostname }}"
    color: warning
```

**tasks/notify_slack.yml:**

```yaml
---
- name: Send Slack notification
  uri:
    url: "{{ slack_webhook_url }}"
    method: POST
    body_format: json
    body:
      attachments:
        - color: "{{ color | default('good') }}"
          title: "Deployment Pipeline"
          text: "{{ message }}"
          fields:
            - title: "Service"
              value: "{{ service_name }}"
              short: true
            - title: "Version"
              value: "{{ new_version }}"
              short: true
            - title: "Environment"
              value: "{{ environment | default('production') }}"
              short: true
            - title: "Timestamp"
              value: "{{ ansible_date_time.iso8601 }}"
              short: true
          footer: "Ansible Automation"
    status_code: 200
  delegate_to: localhost
```

### 🚀 Бонус: GitOps с Ansible

**ArgoCD интеграция:**

```yaml
---
- name: GitOps Deployment
  hosts: localhost
  
  tasks:
    - name: Update Git repository
      git:
        repo: "{{ gitops_repo }}"
        dest: /tmp/gitops
        version: main
        force: yes
    
    - name: Update deployment manifest
      template:
        src: deployment.yml.j2
        dest: /tmp/gitops/apps/{{ service_name }}/deployment.yml
      vars:
        image_tag: "{{ new_version }}"
    
    - name: Commit changes
      shell: |
        cd /tmp/gitops
        git config user.name "Ansible Bot"
        git config user.email "ansible@example.com"
        git add apps/{{ service_name }}/deployment.yml
        git commit -m "Update {{ service_name }} to {{ new_version }}"
        git push origin main
    
    - name: Trigger ArgoCD sync
      uri:
        url: "{{ argocd_url }}/api/v1/applications/{{ service_name }}/sync"
        method: POST
        headers:
          Authorization: "Bearer {{ argocd_token }}"
        body_format: json
        body:
          prune: true
          dryRun: false
    
    - name: Wait for sync
      uri:
        url: "{{ argocd_url }}/api/v1/applications/{{ service_name }}"
        headers:
          Authorization: "Bearer {{ argocd_token }}"
      register: app_status
      until: app_status.json.status.sync.status == "Synced"
      retries: 30
      delay: 10
```

---

## Заключение и Шпаргалка

### 📝 Quick Reference Card

```bash
# === ОСНОВНЫЕ КОМАНДЫ ===

# Ad-hoc команды
ansible all -m ping
ansible all -m setup
ansible webservers -m apt -a "name=nginx state=present" -b

# Playbook выполнение
ansible-playbook playbook.yml
ansible-playbook playbook.yml --check --diff  # Dry-run
ansible-playbook playbook.yml -vvv  # Verbose
ansible-playbook playbook.yml --limit host1  # Один хост
ansible-playbook playbook.yml --tags deploy  # Только теги
ansible-playbook playbook.yml --start-at-task "Task name"  # С определенной таски

# Vault
ansible-vault create secrets.yml
ansible-vault edit secrets.yml
ansible-vault encrypt file.yml
ansible-vault decrypt file.yml
ansible-vault view secrets.yml
ansible-playbook playbook.yml --ask-vault-pass

# Inventory
ansible-inventory --list
ansible-inventory --graph
ansible-inventory --host hostname

# Galaxy
ansible-galaxy init role_name
ansible-galaxy install role_name
ansible-galaxy install -r requirements.yml
ansible-galaxy collection install community.general

# === DEBUGGING ===

# Проверка синтаксиса
ansible-playbook playbook.yml --syntax-check

# Список хостов
ansible-playbook playbook.yml --list-hosts

# Список тасков
ansible-playbook playbook.yml --list-tasks

# Список тегов
ansible-playbook playbook.yml --list-tags

# Step-by-step выполнение
ansible-playbook playbook.yml --step

# Запуск с debugger
ANSIBLE_ENABLE_TASK_DEBUGGER=True ansible-playbook playbook.yml

# === ПОЛЕЗНЫЕ ПЕРЕМЕННЫЕ ===

inventory_hostname          # Имя хоста из инвентори
ansible_hostname           # Фактическое имя хоста
ansible_fqdn              # FQDN
ansible_default_ipv4.address  # IP адрес
ansible_distribution      # Ubuntu, CentOS и т.д.
ansible_memtotal_mb       # RAM в MB
group_names               # Список групп хоста
groups                    # Все группы в инвентори
hostvars                  # Переменные всех хостов

# === JINJA2 ФИЛЬТРЫ ===

{{ var | default('default_value') }}
{{ var | mandatory }}
{{ list | join(',') }}
{{ dict | to_json }}
{{ dict | to_yaml }}
{{ string | upper }}
{{ string | lower }}
{{ number | int }}
{{ path | basename }}
{{ path | dirname }}

# === МОДУЛИ (MUST-KNOW) ===

# Пакеты
apt/yum/dnf/package

# Файлы
copy, template, file, lineinfile, blockinfile

# Сервисы  
service, systemd

# Команды
command, shell, script

# Пользователи
user, group

# Git
git

# HTTP
uri, get_url

# Docker
docker_container, docker_image

# Database
postgresql_db, postgresql_user, mysql_db, mysql_user
```

### 🎯 Best Practices Checklist

```markdown
✅ Структура проекта
  □ Используй роли для переиспользования
  □ Разделяй переменные по группам/хостам
  □ Храни секреты в vault
  □ Используй group_vars и host_vars
  □ Версионируй код в Git

✅ Код
  □ Делай таски идемпотентными
  □ Используй `changed_when` и `failed_when`
  □ Добавляй `check_mode: yes` где возможно
  □ Документируй сложную логику
  □ Используй `tags` для гибкости

✅ Безопасность
  □ Никогда не коммить секреты
  □ Используй ansible-vault
  □ Добавляй `no_log: true` для sensitive tasks
  □ Минимизируй привилегии (become только где нужно)
  □ Регулярно ротируй секреты

✅ Тестирование
  □ Запускай `--syntax-check`
  □ Используй `--check` перед production
  □ Тестируй на staging первым
  □ Используй ansible-lint
  □ Пиши интеграционные тесты

✅ Production
  □ Используй `serial` для rolling updates
  □ Настрой `max_fail_percentage`
  □ Делай backup перед изменениями
  □ Мониторь выполнение
  □ Имей rollback plan

✅ Performance
  □ Используй `pipelining = True`
  □ Кэшируй facts
  □ Отключай `gather_facts` где не нужно
  □ Используй `async` для долгих тасков
  □ Оптимизируй циклы

✅ Maintenance
  □ Документируй runbooks
  □ Версионируй playbooks
  □ Логируй выполнения
  □ Регулярно обновляй зависимости
  □ Code review для изменений
```

### 🚀 Практические сценарии

**Сценарий 1: Быстрый hotfix в production**

```bash
# 1. Создай hotfix ветку
git checkout -b hotfix/critical-bug

# 2. Внеси изменения и закоммить
git add .
git commit -m "Fix critical bug"

# 3. Деплой только затронутых компонентов
ansible-playbook site.yml \
  -i inventories/production \
  --tags application \
  --limit affected_servers \
  -e "version=$(git rev-parse --short HEAD)"

# 4. Верифицируй
ansible-playbook healthcheck.yml -i inventories/production

# 5. Слей в main
git checkout main
git merge hotfix/critical-bug
git push origin main
```

**Сценарий 2: Rollback после неудачного деплоя**

```bash
# 1. Быстрый rollback на предыдущую версию
ansible-playbook rollback.yml \
  -i inventories/production \
  -e "rollback_to=previous"

# 2. Если нужна конкретная версия
ansible-playbook rollback.yml \
  -i inventories/production \
  -e "rollback_to=v1.2.3"

# 3. Верификация
ansible-playbook healthcheck.yml -i inventories/production
```

**Сценарий 3: Добавление нового сервера**

```bash
# 1. Добавь в инвентори
cat >> inventories/production/hosts.yml <<EOF
    web4.prod.example.com:
      ansible_host: 10.0.2.13
EOF

# 2. Bootstrap нового сервера
ansible-playbook bootstrap.yml \
  -i inventories/production \
  --limit web4.prod.example.com

# 3. Деплой полной конфигурации
ansible-playbook site.yml \
  -i inventories/production \
  --limit web4.prod.example.com

# 4. Добавь в load balancer
ansible-playbook loadbalancer.yml \
  -i inventories/production \
  -e "action=add" \
  -e "new_server=web4.prod.example.com"
```

**Сценарий 4: Database maintenance**

```bash
# 1. Backup перед maintenance
ansible-playbook backup.yml \
  -i inventories/production \
  --tags database

# 2. Переключись на standby
ansible-playbook database.yml \
  -i inventories/production \
  --tags failover \
  -e "promote_standby=db2.prod.example.com"

# 3. Maintenance на старом primary
ansible-playbook database.yml \
  -i inventories/production \
  --tags maintenance \
  --limit db1.prod.example.com

# 4. Rebuild replication
ansible-playbook database.yml \
  -i inventories/production \
  --tags replication
```

---

## Финальное задание: Построй Production-Ready инфраструктуру

### Требования

Создай полную production инфраструктуру для e-commerce приложения:

**Компоненты:**
1. **Frontend**: 3x Nginx (static files + reverse proxy)
2. **Backend**: 3x Node.js приложение
3. **Database**: PostgreSQL Primary + Standby
4. **Cache**: Redis Cluster (Master + 2 Replicas)
5. **Search**: Elasticsearch cluster
6. **Message Queue**: RabbitMQ
7. **Load Balancer**: 2x HAProxy с Keepalived
8. **Monitoring**: Prometheus + Grafana
9. **Logging**: ELK Stack

**Функциональные требования:**
- Zero-downtime deployments
- Automated backups (ежедневные, недельные, месячные)
- Health monitoring каждые 30 секунд
- Auto-healing для критических сервисов
- Disaster recovery с RTO < 15 минут
- Blue-green deployment для backend
- Canary releases для новых фич
- Performance testing перед production
- Automated rollback при ошибках > 1%

**Нефункциональные требования:**
- Все секреты в Ansible Vault
- CI/CD pipeline (GitHub Actions)
- Infrastructure as Code (все в Git)
- Документация (Architecture, Runbook, Deployment guide)
- Мониторинг и алертинг (Slack notifications)
- Capacity planning и autoscaling

### Решение (структура)

```
ecommerce-infrastructure/
├── README.md
├── ARCHITECTURE.md
├── DEPLOYMENT.md
├── RUNBOOK.md
├── ansible.cfg
├── Makefile
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy-staging.yml
│       ├── deploy-production.yml
│       └── disaster-recovery.yml
├── inventories/
│   ├── production/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   └── host_vars/
│   └── staging/
│       └── hosts.yml
├── group_vars/
│   ├── all/
│   │   ├── vars.yml
│   │   └── vault.yml (encrypted)
│   ├── frontend.yml
│   ├── backend.yml
│   ├── databases.yml
│   └── monitoring.yml
├── roles/
│   ├── common/
│   ├── security/
│   ├── nginx/
│   ├── nodejs/
│   ├── postgresql/
│   ├── redis/
│   ├── elasticsearch/
│   ├── rabbitmq/
│   ├── haproxy/
│   ├── prometheus/
│   ├── grafana/
│   ├── filebeat/
│   └── backup/
├── playbooks/
│   ├── site.yml
│   ├── deploy.yml
│   ├── rollback.yml
│   ├── backup.yml
│   ├── restore.yml
│   ├── disaster-recovery.yml
│   ├── health-check.yml
│   ├── performance-test.yml
│   └── maintenance.yml
├── scripts/
│   ├── deploy.sh
│   ├── rollback.sh
│   ├── backup.sh
│   └── dr-test.sh
├── tests/
│   ├── integration/
│   └── smoke/
└── docs/
    ├── api/
    ├── infrastructure/
    └── procedures/
```

**Время на выполнение:** 4-6 часов

**Критерии успеха:**
- ✅ Все сервисы запускаются и работают
- ✅ Deployment проходит без downtime
- ✅ Rollback работает корректно
- ✅ Backups создаются автоматически
- ✅ Restore восстанавливает данные
- ✅ Monitoring показывает метрики
- ✅ Alerts приходят в Slack
- ✅ CI/CD pipeline проходит все стадии
- ✅ Документация полная и актуальная

---

## Дополнительные ресурсы

### 📚 Документация

- **Official Ansible Docs**: https://docs.ansible.com/
- **Ansible Galaxy**: https://galaxy.ansible.com/
- **Ansible Collections**: https://docs.ansible.com/ansible/latest/collections/
- **Best Practices**: https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html

### 🎓 Обучение

- **Ansible for DevOps**: https://www.ansiblefordevops.com/
- **Red Hat Training**: https://www.redhat.com/en/services/training/
- **YouTube канал Jeff Geerling**: https://www.youtube.com/c/JeffGeerling
- **Ansible Community**: https://www.ansible.com/community

### 🛠️ Инструменты

- **ansible-lint**: Проверка качества кода
- **ansible-navigator**: Современный интерфейс
- **Molecule**: Тестирование ролей
- **AWX/Ansible Tower**: Web UI и API
- **VS Code Ansible Extension**: IDE поддержка

### 🌐 Сообщество

- **Ansible Reddit**: r/ansible
- **Ansible Forum**: https://forum.ansible.com/
- **Matrix Chat**: #ansible:matrix.org
- **GitHub Discussions**: https://github.com/ansible/ansible/discussions

---

## Поздравляю! 🎉

Ты завершил полный курс Ansible Refresh!

**Что дальше:**

1. **Практика**: Примени знания на реальных проектах
2. **Автоматизируй**: Начни с малого, автоматизируй рутинные задачи
3. **Масштабируй**: Постепенно переходи к более сложным сценариям
4. **Делись**: Публикуй свои роли на Galaxy
5. **Учись**: Следи за новыми фичами Ansible

**Ключевые навыки, которые ты получил:**

✅ Основы Ansible и архитектура
✅ Написание playbooks и ролей
✅ Управление переменными и secrets
✅ Шаблонизация с Jinja2
✅ Продвинутые паттерны (blue-green, canary)
✅ CI/CD интеграция
✅ Мониторинг и troubleshooting
✅ Безопасность и best practices
✅ Production-ready deployments

**Помни:**
- **Идемпотентность** - основа Ansible
- **Тестируй** перед production
- **Документируй** свой код
- **Версионируй** всё в Git
- **Автоматизируй** постепенно

Удачи в DevOps путешествии! 🚀
