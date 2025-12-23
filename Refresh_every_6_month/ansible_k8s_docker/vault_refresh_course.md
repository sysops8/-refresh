# HashiCorp Vault Refresh: Ежегодный/Полугодовой курс для DevOps

**Цель:** Освежить в памяти ключевые концепции HashiCorp Vault за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная задача, которую нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Предварительные требования:**
- Доступ к Linux/macOS машине
- Docker установлен (для dev режима)
- Базовые знания CLI
- curl или аналогичный HTTP клиент

---

## Модуль 1: Архитектура и базовая настройка (20 минут)

### 🎯 Напоминалка

**Архитектура Vault:**
```
┌─────────────────────────────────────┐
│           Vault Server              │
│                                     │
│  ┌──────────────────────────────┐  │
│  │      Storage Backend         │  │
│  │  (Consul, Raft, File, etc.)  │  │
│  └──────────────────────────────┘  │
│                                     │
│  ┌──────────────────────────────┐  │
│  │      Secrets Engines         │  │
│  │  (KV, Database, PKI, etc.)   │  │
│  └──────────────────────────────┘  │
│                                     │
│  ┌──────────────────────────────┐  │
│  │      Auth Methods            │  │
│  │  (Token, AppRole, LDAP, etc.)│  │
│  └──────────────────────────────┘  │
│                                     │
│  ┌──────────────────────────────┐  │
│  │      Audit Devices           │  │
│  │  (File, Syslog, Socket)      │  │
│  └──────────────────────────────┘  │
└─────────────────────────────────────┘
```

**Ключевые концепции:**
- **Seal/Unseal**: Vault запускается в sealed состоянии, требует unseal keys
- **Root Token**: Первоначальный токен с полными правами
- **Policies**: ACL для управления доступом
- **Paths**: Все в Vault организовано по путям (как файловая система)

**Установка и запуск:**
```bash
# Установка (macOS)
brew tap hashicorp/tap
brew install hashicorp/tap/vault

# Установка (Linux)
wget https://releases.hashicorp.com/vault/1.15.4/vault_1.15.4_linux_amd64.zip
unzip vault_1.15.4_linux_amd64.zip
sudo mv vault /usr/local/bin/

# Проверка
vault version

# Dev сервер (НЕ для production!)
vault server -dev

# В новом терминале
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='<root-token-from-output>'

# Проверка статуса
vault status
```

**Production сервер (конфигурация):**
```hcl
# config.hcl
storage "raft" {
  path    = "/opt/vault/data"
  node_id = "vault-1"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = 1  # Только для dev! В prod используй TLS
}

api_addr = "http://127.0.0.1:8200"
cluster_addr = "https://127.0.0.1:8201"
ui = true

# Telemetry (optional)
telemetry {
  prometheus_retention_time = "30s"
  disable_hostname = true
}
```

**Запуск production сервера:**
```bash
# Создать директории
sudo mkdir -p /opt/vault/data
sudo mkdir -p /etc/vault

# Скопировать конфиг
sudo cp config.hcl /etc/vault/

# Запустить
vault server -config=/etc/vault/config.hcl

# Инициализация (в новом терминале)
vault operator init -key-shares=5 -key-threshold=3

# Сохрани unseal keys и root token!
# Unseal (нужно выполнить 3 раза с разными ключами)
vault operator unseal <unseal-key-1>
vault operator unseal <unseal-key-2>
vault operator unseal <unseal-key-3>

# Логин
vault login <root-token>
```

**Базовые команды:**
```bash
# Статус
vault status

# Seal/Unseal
vault operator seal
vault operator unseal <key>

# Логин
vault login <token>

# Список авторизованных методов
vault auth list

# Список secrets engines
vault secrets list

# Помощь
vault path-help <path>
```

**Docker для быстрого старта:**
```bash
# Dev режим
docker run --rm --cap-add=IPC_LOCK \
  -e 'VAULT_DEV_ROOT_TOKEN_ID=myroot' \
  -e 'VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200' \
  -p 8200:8200 \
  hashicorp/vault:latest

export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='myroot'
```

### 💻 Задание

Подготовь тестовое окружение:

1. Запусти Vault в dev режиме:
   ```bash
   vault server -dev -dev-root-token-id="root"
   ```

2. В новом терминале настрой переменные окружения:
   ```bash
   export VAULT_ADDR='http://127.0.0.1:8200'
   export VAULT_TOKEN='root'
   ```

3. Проверь статус:
   ```bash
   vault status
   ```

4. Посмотри список secrets engines и auth methods:
   ```bash
   vault secrets list
   vault auth list
   ```

5. Открой UI в браузере: http://127.0.0.1:8200

6. Создай простой секрет:
   ```bash
   vault kv put secret/hello foo=world
   vault kv get secret/hello
   ```

7. Прочитай через API:
   ```bash
   curl -H "X-Vault-Token: root" \
     http://127.0.0.1:8200/v1/secret/data/hello | jq
   ```

### 🚀 Бонус (новое)

Настрой **production-like** конфигурацию с Raft storage:

1. Создай конфиг файл `vault-config.hcl`:
   ```hcl
   storage "raft" {
     path    = "./vault-data"
     node_id = "node1"
   }

   listener "tcp" {
     address     = "127.0.0.1:8200"
     tls_disable = 1
   }

   api_addr = "http://127.0.0.1:8200"
   cluster_addr = "http://127.0.0.1:8201"
   ui = true
   ```

2. Запусти Vault:
   ```bash
   mkdir -p vault-data
   vault server -config=vault-config.hcl
   ```

3. В новом терминале инициализируй:
   ```bash
   export VAULT_ADDR='http://127.0.0.1:8200'
   vault operator init -key-shares=1 -key-threshold=1 > vault-keys.txt
   ```

4. Unseal и логин:
   ```bash
   vault operator unseal $(grep 'Unseal Key 1' vault-keys.txt | awk '{print $4}')
   vault login $(grep 'Initial Root Token' vault-keys.txt | awk '{print $4}')
   ```

5. Проверь Raft status:
   ```bash
   vault operator raft list-peers
   ```

---

## Модуль 2: KV Secrets Engine (25 минут)

### 🎯 Напоминалка

**KV Secrets Engine - два типа:**

**KV Version 1 (неверсионируемый):**
```bash
# Enable
vault secrets enable -path=kv kv

# Write
vault kv put kv/myapp password=secret123

# Read
vault kv get kv/myapp

# Delete
vault kv delete kv/myapp

# List
vault kv list kv/
```

**KV Version 2 (версионируемый, рекомендуется):**
```bash
# Enable (по умолчанию в dev mode на secret/)
vault secrets enable -path=secret kv-v2

# Write (создает версию 1)
vault kv put secret/myapp password=secret123

# Update (создает версию 2)
vault kv put secret/myapp password=newsecret456

# Read (последняя версия)
vault kv get secret/myapp

# Read конкретную версию
vault kv get -version=1 secret/myapp

# Metadata и версии
vault kv metadata get secret/myapp

# История версий
vault kv get -format=json secret/myapp | jq .data.metadata

# Rollback
vault kv rollback -version=1 secret/myapp

# Soft delete (можно восстановить)
vault kv delete secret/myapp

# Undelete
vault kv undelete -versions=2 secret/myapp

# Hard delete (нельзя восстановить)
vault kv destroy -versions=1,2 secret/myapp

# Удалить все версии
vault kv metadata delete secret/myapp
```

**Работа с JSON:**
```bash
# Запись JSON
vault kv put secret/config @config.json

# Или inline
vault kv put secret/config \
  db_host=localhost \
  db_port=5432 \
  db_user=admin

# Чтение в JSON
vault kv get -format=json secret/config | jq .data.data

# Patch (обновление части данных)
vault kv patch secret/config db_port=5433
```

**CAS (Check-And-Set) для безопасных обновлений:**
```bash
# Получить текущую версию
vault kv get -format=json secret/myapp | jq .data.metadata.version

# Обновить только если версия совпадает
vault kv put -cas=2 secret/myapp password=new_password
```

**Настройки secrets engine:**
```bash
# Максимальное количество версий
vault kv metadata put -max-versions=5 secret/myapp

# Auto-delete старых версий
vault kv metadata put -delete-version-after=30d secret/myapp

# Запрет удаления
vault kv metadata put -custom-metadata=protected=true secret/prod-db
```

**API примеры:**
```bash
# Write
curl -H "X-Vault-Token: $VAULT_TOKEN" \
  -X POST \
  -d '{"data":{"password":"secret123"}}' \
  $VAULT_ADDR/v1/secret/data/myapp

# Read
curl -H "X-Vault-Token: $VAULT_TOKEN" \
  $VAULT_ADDR/v1/secret/data/myapp | jq

# Read конкретную версию
curl -H "X-Vault-Token: $VAULT_TOKEN" \
  $VAULT_ADDR/v1/secret/data/myapp?version=1 | jq
```

### 💻 Задание

Создай систему управления секретами для multi-environment приложения:

1. **Включи KV v2 engine:**
   ```bash
   vault secrets enable -path=apps kv-v2
   ```

2. **Создай секреты для разных окружений:**
   ```bash
   # Development
   vault kv put apps/myapp/dev \
     db_host=dev-db.example.com \
     db_port=5432 \
     db_user=dev_user \
     db_password=dev_pass123 \
     api_key=dev_key_xyz

   # Staging
   vault kv put apps/myapp/staging \
     db_host=staging-db.example.com \
     db_port=5432 \
     db_user=staging_user \
     db_password=staging_pass456 \
     api_key=staging_key_abc

   # Production
   vault kv put apps/myapp/prod \
     db_host=prod-db.example.com \
     db_port=5432 \
     db_user=prod_user \
     db_password=super_secret_789 \
     api_key=prod_key_secure
   ```

3. **Создай общий конфиг:**
   ```bash
   vault kv put apps/myapp/common \
     app_name="MyApplication" \
     version="1.2.3" \
     log_level=info
   ```

4. **Прочитай и проверь:**
   ```bash
   vault kv get apps/myapp/dev
   vault kv get -format=json apps/myapp/prod | jq .data.data
   ```

5. **Обнови пароль в dev:**
   ```bash
   vault kv put apps/myapp/dev db_password=new_dev_pass
   ```

6. **Посмотри историю версий:**
   ```bash
   vault kv metadata get apps/myapp/dev
   ```

7. **Создай патч (обновление одного поля):**
   ```bash
   vault kv patch apps/myapp/dev db_port=5433
   ```

8. **Soft delete и restore:**
   ```bash
   # Delete
   vault kv delete apps/myapp/dev
   
   # Verify deleted
   vault kv get apps/myapp/dev
   
   # Undelete
   vault kv undelete -versions=3 apps/myapp/dev
   ```

### 🚀 Бонус (новое)

**1. Настрой кастомные metadata и lifecycle:**

```bash
# Установи max versions и auto-delete
vault kv metadata put \
  -max-versions=10 \
  -delete-version-after=90d \
  -custom-metadata=team=platform \
  -custom-metadata=owner=john \
  apps/myapp/prod

# Проверь настройки
vault kv metadata get apps/myapp/prod
```

**2. Используй CAS для безопасного обновления:**

```bash
# Получи текущую версию
current_version=$(vault kv get -format=json apps/myapp/prod | jq -r .data.metadata.version)

# Обнови только если версия не изменилась
vault kv put -cas=$current_version apps/myapp/prod \
  db_password=ultra_secure_new_password

# Если кто-то обновил до тебя - получишь ошибку
```

**3. Создай скрипт для безопасного чтения секретов:**

```bash
#!/bin/bash
# read-secret.sh

VAULT_PATH=$1
FIELD=$2

if [ -z "$VAULT_PATH" ] || [ -z "$FIELD" ]; then
  echo "Usage: $0 <vault-path> <field>"
  exit 1
fi

# Проверка токена
if [ -z "$VAULT_TOKEN" ]; then
  echo "Error: VAULT_TOKEN not set"
  exit 1
fi

# Чтение секрета
secret=$(vault kv get -format=json "$VAULT_PATH" 2>/dev/null)

if [ $? -ne 0 ]; then
  echo "Error: Failed to read secret from $VAULT_PATH"
  exit 1
fi

# Извлечение поля
value=$(echo "$secret" | jq -r ".data.data.${FIELD}")

if [ "$value" == "null" ]; then
  echo "Error: Field $FIELD not found"
  exit 1
fi

echo "$value"
```

**4. Bulk операции через API:**

```bash
# Создай несколько секретов
for env in dev staging prod; do
  curl -H "X-Vault-Token: $VAULT_TOKEN" \
    -X POST \
    -d "{\"data\":{\"env\":\"$env\",\"timestamp\":\"$(date)\"}}" \
    $VAULT_ADDR/v1/apps/data/bulk/$env
done

# List всех секретов
curl -H "X-Vault-Token: $VAULT_TOKEN" \
  -X LIST \
  $VAULT_ADDR/v1/apps/metadata/bulk | jq
```

---

## Модуль 3: Policies и Access Control (30 минут)

### 🎯 Напоминалка

**Policy структура:**
```hcl
# policy.hcl
path "secret/data/myapp/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "secret/metadata/myapp/*" {
  capabilities = ["list", "read"]
}

# Deny always wins
path "secret/data/myapp/admin" {
  capabilities = ["deny"]
}
```

**Capabilities:**
```
create  - POST/PUT для создания
read    - GET для чтения
update  - POST/PUT для обновления существующих
delete  - DELETE
list    - LIST операции
sudo    - Доступ к protected paths
deny    - Явный запрет (приоритет над всем)
```

**Работа с policies:**
```bash
# Создание
vault policy write myapp-dev policy.hcl

# Чтение
vault policy read myapp-dev

# Список
vault policy list

# Удаление
vault policy delete myapp-dev

# Inline создание
vault policy write test - <<EOF
path "secret/data/test/*" {
  capabilities = ["read"]
}
EOF
```

**Template policies (продвинутое):**
```hcl
# identity-based policy
path "secret/data/{{identity.entity.name}}/*" {
  capabilities = ["create", "read", "update", "delete"]
}

path "secret/data/team/{{identity.entity.metadata.team}}/*" {
  capabilities = ["read", "list"]
}
```

**Wildcard и глобы:**
```hcl
# + matches single path segment
path "secret/data/apps/+/config" {
  capabilities = ["read"]
}
# matches: secret/data/apps/app1/config
# matches: secret/data/apps/app2/config
# NOT matches: secret/data/apps/app1/subdir/config

# * matches anything at that level and below
path "secret/data/apps/*/logs/*" {
  capabilities = ["read"]
}
# matches: secret/data/apps/app1/logs/error.log
# matches: secret/data/apps/app1/logs/deep/nested/log.txt
```

**Required parameters:**
```hcl
path "transit/encrypt/orders" {
  capabilities = ["update"]
  required_parameters = ["plaintext"]
}
```

**Allowed/Denied parameters:**
```hcl
path "auth/userpass/users/*" {
  capabilities = ["create", "update"]
  allowed_parameters = {
    "password" = []
    "policies" = []
  }
  denied_parameters = {
    "token_bound_cidrs" = []
  }
}
```

**Min/Max wrapping TTL:**
```hcl
path "sys/wrapping/wrap" {
  capabilities = ["update"]
  min_wrapping_ttl = "100s"
  max_wrapping_ttl = "1h"
}
```

**Policy примеры для разных ролей:**

```hcl
# admin-policy.hcl
path "*" {
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# dev-policy.hcl
path "secret/data/dev/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "secret/metadata/dev/*" {
  capabilities = ["list"]
}

# read-only-policy.hcl
path "secret/data/prod/*" {
  capabilities = ["read", "list"]
}

# app-policy.hcl (для CI/CD)
path "secret/data/apps/myapp/*" {
  capabilities = ["read"]
}

path "database/creds/myapp-role" {
  capabilities = ["read"]
}

path "auth/token/renew-self" {
  capabilities = ["update"]
}

path "auth/token/lookup-self" {
  capabilities = ["read"]
}
```

### 💻 Задание

Создай систему контроля доступа для команды разработки:

1. **Создай policy для разработчиков:**

```bash
cat > dev-policy.hcl <<EOF
# Read/write access to dev secrets
path "secret/data/dev/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "secret/metadata/dev/*" {
  capabilities = ["read", "list"]
}

# Read-only access to common configs
path "secret/data/common/*" {
  capabilities = ["read", "list"]
}

# No access to production
path "secret/data/prod/*" {
  capabilities = ["deny"]
}
EOF

vault policy write dev-policy dev-policy.hcl
```

2. **Создай policy для production окружения:**

```bash
cat > prod-policy.hcl <<EOF
# Read-only access to production secrets
path "secret/data/prod/*" {
  capabilities = ["read", "list"]
}

path "secret/metadata/prod/*" {
  capabilities = ["read", "list"]
}

# Allow token renewal
path "auth/token/renew-self" {
  capabilities = ["update"]
}
EOF

vault policy write prod-policy prod-policy.hcl
```

3. **Создай policy для CI/CD:**

```bash
cat > cicd-policy.hcl <<EOF
# Read secrets for deployment
path "secret/data/apps/myapp/*" {
  capabilities = ["read"]
}

# Create dynamic database credentials
path "database/creds/myapp-role" {
  capabilities = ["read"]
}

# Token operations
path "auth/token/renew-self" {
  capabilities = ["update"]
}

path "auth/token/lookup-self" {
  capabilities = ["read"]
}
EOF

vault policy write cicd-policy cicd-policy.hcl
```

4. **Создай токены с разными policies:**

```bash
# Dev token
vault token create -policy=dev-policy -ttl=8h

# Prod token
vault token create -policy=prod-policy -ttl=1h

# CI/CD token
vault token create -policy=cicd-policy -ttl=24h -renewable=true
```

5. **Протестируй policies:**

```bash
# Сохрани dev токен
DEV_TOKEN=$(vault token create -policy=dev-policy -format=json | jq -r .auth.client_token)

# Попробуй записать в dev (должно работать)
VAULT_TOKEN=$DEV_TOKEN vault kv put secret/dev/test foo=bar

# Попробуй прочитать prod (должно быть запрещено)
VAULT_TOKEN=$DEV_TOKEN vault kv get secret/prod/db
```

6. **Проверь возможности токена:**

```bash
vault token lookup $DEV_TOKEN
vault token capabilities $DEV_TOKEN secret/data/dev/test
vault token capabilities $DEV_TOKEN secret/data/prod/db
```

### 🚀 Бонус (новое)

**1. Создай policy с template variables:**

```bash
cat > user-policy.hcl <<EOF
# Each user can only access their own secrets
path "secret/data/users/{{identity.entity.name}}/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

# Team-based access
path "secret/data/teams/{{identity.entity.metadata.team}}/*" {
  capabilities = ["read", "list"]
}
EOF

vault policy write user-policy user-policy.hcl
```

**2. Fine-grained control с parameter constraints:**

```bash
cat > constrained-policy.hcl <<EOF
path "auth/userpass/users/*" {
  capabilities = ["create", "update"]
  
  # Only allow setting these parameters
  allowed_parameters = {
    "password" = []
    "token_ttl" = []
  }
  
  # Prevent setting these parameters
  denied_parameters = {
    "token_policies" = []
    "token_bound_cidrs" = []
  }
}

path "secret/data/restricted/*" {
  capabilities = ["update"]
  
  # Require specific fields
  required_parameters = ["environment", "owner"]
  
  # Control TTL
  min_wrapping_ttl = "1h"
  max_wrapping_ttl = "24h"
}
EOF

vault policy write constrained-policy constrained-policy.hcl
```

**3. Sentinel policy (Enterprise feature - для знакомства):**

```hcl
# sentinel-policy.sentinel
import "time"
import "strings"

# Enforce time-based access
main = rule {
  time.now.weekday in ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"] and
  time.now.hour >= 9 and
  time.now.hour < 18
}

# Enforce naming conventions
secret_path = request.path
main = rule {
  strings.has_prefix(secret_path, "secret/data/") and
  strings.has_suffix(secret_path, "/config")
}
```

**4. Тестирование policies:**

```bash
#!/bin/bash
# test-policy.sh

POLICY_NAME=$1
TEST_PATH=$2
OPERATION=$3

# Создай тестовый токен
TOKEN=$(vault token create -policy=$POLICY_NAME -format=json | jq -r .auth.client_token)

# Тест capabilities
vault token capabilities $TOKEN $TEST_PATH

# Тест операции
case $OPERATION in
  "read")
    VAULT_TOKEN=$TOKEN vault read $TEST_PATH
    ;;
  "write")
    VAULT_TOKEN=$TOKEN vault write $TEST_PATH test=value
    ;;
  "delete")
    VAULT_TOKEN=$TOKEN vault delete $TEST_PATH
    ;;
esac

# Cleanup
vault token revoke $TOKEN
```

---

## Модуль 4: Authentication Methods (30 минут)

### 🎯 Напоминалка

**Типы Auth Methods:**

**Token (встроенный):**
```bash
# Создание
vault token create -policy=myapp

# С TTL и renewable
vault token create -policy=myapp -ttl=1h -renewable=true

# Periodic token (продлевается автоматически)
vault token create -policy=myapp -period=24h

# Lookup
vault token lookup <token>

# Renew
vault token renew <token>

# Revoke
vault token revoke <token>
```

**AppRole (для приложений):**
```bash
# Enable
vault auth enable approle

# Создать role
vault write auth/approle/role/myapp \
  token_ttl=1h \
  token_max_ttl=4h \
  token_policies=myapp-policy

# Получить RoleID
vault read auth/approle/role/myapp/role-id

# Создать SecretID
vault write -f auth/approle/role/myapp/secret-id

# Логин
vault write auth/approle/login \
  role_id=<role-id> \
  secret_id=<secret-id>
```

**UserPass (для людей):**
```bash
# Enable
vault auth enable userpass

# Создать пользователя
vault write auth/userpass/users/john \
  password=supersecret \
  policies=dev-policy

# Логин
vault login -method=userpass username=john password=supersecret

# Смена пароля
vault write auth/userpass/users/john/password password=newpassword
```

**Kubernetes:**
```bash
# Enable
vault auth enable kubernetes

# Configure
vault write auth/kubernetes/config \
  kubernetes_host=https://kubernetes.default.svc:443

# Создать роль
vault write auth/kubernetes/role/myapp \
  bound_service_account_names=myapp-sa \
  bound_service_account_namespaces=default \
  policies=myapp-policy \
  ttl=1h
```

**AWS:**
```bash
# Enable
vault auth enable aws

# Configure
vault write auth/aws/config/client \
  access_key=<key> \
  secret_key=<secret>

# Создать роль
vault write auth/aws/role/dev-role \
  auth_type=iam \
  bound_iam_principal_arn=arn:aws:iam::123456789012:role/MyRole \
  policies=dev-policy \
  ttl=1h
```

**LDAP:**
```bash
# Enable
vault auth enable ldap

# Configure
vault write auth/ldap/config \
  url="ldap://ldap.example.com" \
  userdn="ou=Users,dc=example,dc=com" \
  groupdn="ou=Groups,dc=example,dc=com" \
  binddn="cn=vault,ou=Users,dc=example,dc=com" \
  bindpass="password"

# Map группы к policies
vault write auth/ldap/groups/developers policies=dev-policy
```

**GitHub:**
```bash
# Enable
vault auth enable github

# Configure
vault write auth/github/config organization=myorg

# Map teams к policies
vault write auth/github/map/teams/developers value=dev-policy
```

**OIDC/OAuth:**
```bash
# Enable
vault auth enable oidc

# Configure
vault write auth/oidc/config \
  oidc_discovery_url="https://accounts.google.com" \
  oidc_client_id="<client-id>" \
  oidc_client_secret="<client-secret>" \
  default_role="reader"

# Создать роль
vault write auth/oidc/role/reader \
  bound_audiences="<client-id>" \
  allowed_redirect_uris="http://localhost:8200/ui/vault/auth/oidc/oidc/callback" \
  user_claim="sub" \
  policies=reader
```

**Команды для управления:**
```bash
# Список auth methods
vault auth list

# Описание
vault auth help <method>

# Включить
vault auth enable <method>

# Отключить
vault auth disable <path>

# Настройка TTL
vault auth tune -default-lease-ttl=1h auth/approle
```

### 💻 Задание

Настрой несколько методов аутентификации:

1. **AppRole для CI/CD системы:**

```bash
# Enable AppRole
vault auth enable approle

# Создай роль
vault write auth/approle/role/cicd \
  token_ttl=1h \
  token_max_ttl=4h \
  token_policies=cicd-policy \
  bind_secret_id=true \
  secret_id_ttl=10m

# Получи RoleID (можно хранить в конфиге)
vault read auth/approle/role/cicd/role-id

# Создай SecretID (короткоживущий, для одноразового использования)
vault write -f auth/approle/role/cicd/secret-id

# Тест логина
ROLE_ID=$(vault read -field=role_id auth/approle/role/cicd/role-id)
SECRET_ID=$(vault write -field=secret_id -f auth/approle/role/cicd/secret-id)

vault write auth/approle/login \
  role_id=$ROLE_ID \
  secret_id=$SECRET_ID
```

2. **UserPass для разработчиков:**

```bash
# Enable
vault auth enable userpass

# Создай пользователей
vault write auth/userpass/users/alice \
  password=alice123 \
  policies=dev-policy

vault write auth/userpass/users/bob \
  password=bob456 \
  policies=prod-policy

# Тест логина
vault login -method=userpass username=alice password=alice123

# Смена пароля
vault write auth/userpass/users/alice/password password=newalice123
```

3. **Token с разными параметрами:**

```bash
# Короткоживущий токен
vault token create -policy=dev-policy -ttl=15m

# Renewable токен
vault token create -policy=dev-policy -ttl=1h -renewable=true

# Periodic токен (для сервисов)
vault token create -policy=myapp-policy -period=24h

# Orphan token (не привязан к родителю)
vault token create -policy=dev-policy -orphan

# Token с использованиями
vault token create -policy=dev-policy -use-limit=5
```

4. **Создай wrapper script для AppRole логина:**

```bash
cat > approle-login.sh <<'EOF'
#!/bin/bash

VAULT_ADDR=${VAULT_ADDR:-http://127.0.0.1:8200}
ROLE_NAME=${1:-cicd}

# Read RoleID from env or file
if [ -z "$ROLE_ID" ]; then
  ROLE_ID=$(vault read -field=role_id auth/approle/role/$ROLE_NAME/role-id)
fi

# Generate new SecretID
SECRET_ID=$(vault write -field=secret_id -f auth/approle/role/$ROLE_NAME/secret-id)

# Login and get token
RESPONSE=$(vault write -format=json auth/approle/login \
  role_id=$ROLE_ID \
  secret_id=$SECRET_ID)

TOKEN=$(echo $RESPONSE | jq -r .auth.client_token)

# Export token
export VAULT_TOKEN=$TOKEN
echo "Logged in successfully. Token exported to VAULT_TOKEN"
echo "Token TTL: $(echo $RESPONSE | jq -r .auth.lease_duration) seconds"
EOF

chmod +x approle-login.sh
```

5. **Протестируй разные auth methods:**

```bash
# Token lookup для каждого метода
vault token lookup

# Проверь capabilities
vault token capabilities secret/data/dev/test
```

### 🚀 Бонус (новое)

**1. Настрой Kubernetes auth method (если есть доступ к K8s):**

```bash
# Enable
vault auth enable kubernetes

# Configure (из Pod в K8s)
vault write auth/kubernetes/config \
  kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

# Создай роль
vault write auth/kubernetes/role/myapp \
  bound_service_account_names=vault-auth \
  bound_service_account_namespaces=default \
  policies=myapp-policy \
  ttl=24h

# В Pod используй:
# KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
# vault write auth/kubernetes/login role=myapp jwt=$KUBE_TOKEN
```

**2. Entity и Entity Aliases (для unified identity):**

```bash
# Создай entity
vault write identity/entity name="john-doe" \
  policies="base-policy" \
  metadata=team=engineering

# Получи entity ID
ENTITY_ID=$(vault read -field=id identity/entity/name/john-doe)

# Создай alias для userpass
ACCESSOR=$(vault auth list -format=json | jq -r '.["userpass/"].accessor')
vault write identity/entity-alias \
  name="john" \
  canonical_id=$ENTITY_ID \
  mount_accessor=$ACCESSOR

# Теперь john в userpass связан с entity john-doe
```

**3. AppRole с CIDR binding:**

```bash
# Ограничь доступ по IP
vault write auth/approle/role/secure-app \
  token_policies=myapp-policy \
  token_bound_cidrs="10.0.0.0/8,192.168.1.0/24" \
  token_ttl=1h
```

**4. Response Wrapping для секретного SecretID:**

```bash
# Создай wrapped SecretID
WRAPPED_TOKEN=$(vault write -wrap-ttl=1h \
  -format=json \
  -f auth/approle/role/cicd/secret-id | jq -r .wrap_info.token)

# Получи SecretID из wrapped токена
vault unwrap -format=json $WRAPPED_TOKEN | jq -r .data.secret_id
```

---

## Модуль 5: Dynamic Secrets - Database (30 минут)

### 🎯 Напоминалка

**Database Secrets Engine** - генерирует динамические credentials для БД.

**Поддерживаемые БД:**
- PostgreSQL
- MySQL/MariaDB
- MongoDB
- Cassandra
- Oracle
- MSSQL
- Elasticsearch
- InfluxDB
- Redshift

**Workflow:**
1. Vault подключается к БД с admin правами
2. При запросе создает временного пользователя
3. Выдает credentials с определенным TTL
4. После истечения TTL автоматически удаляет пользователя

**PostgreSQL пример:**
```bash
# Enable
vault secrets enable database

# Configure connection
vault write database/config/postgresql \
  plugin_name=postgresql-database-plugin \
  allowed_roles="readonly,readwrite" \
  connection_url="postgresql://{{username}}:{{password}}@localhost:5432/mydb?sslmode=disable" \
  username="vaultadmin" \
  password="vaultpass"

# Создать роль (readonly)
vault write database/roles/readonly \
  db_name=postgresql \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

# Создать роль (readwrite)
vault write database/roles/readwrite \
  db_name=postgresql \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
    GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

# Получить credentials
vault read database/creds/readonly

# Output:
# Key                Value
# ---                -----
# lease_id           database/creds/readonly/abc123
# lease_duration     1h
# lease_renewable    true
# password           A1a-2b3c4d5e6f
# username           v-token-readonly-xyz123
```

**MySQL пример:**
```bash
vault write database/config/mysql \
  plugin_name=mysql-database-plugin \
  connection_url="{{username}}:{{password}}@tcp(localhost:3306)/" \
  allowed_roles="app-role" \
  username="root" \
  password="rootpass"

vault write database/roles/app-role \
  db_name=mysql \
  creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; \
    GRANT SELECT ON mydb.* TO '{{name}}'@'%';" \
  default_ttl="1h" \
  max_ttl="24h"
```

**MongoDB пример:**
```bash
vault write database/config/mongodb \
  plugin_name=mongodb-database-plugin \
  allowed_roles="app-role" \
  connection_url="mongodb://{{username}}:{{password}}@localhost:27017/admin" \
  username="admin" \
  password="admin"

vault write database/roles/app-role \
  db_name=mongodb \
  creation_statements='{ "db": "mydb", "roles": [{ "role": "readWrite" }] }' \
  default_ttl="1h" \
  max_ttl="24h"
```

**Root Rotation (безопасность):**
```bash
# Ротация root credentials после настройки
vault write -f database/rotate-root/postgresql
```

**Lease управление:**
```bash
# Renew lease
vault lease renew database/creds/readonly/abc123

# Revoke lease
vault lease revoke database/creds/readonly/abc123

# Revoke все leases для роли
vault lease revoke -prefix database/creds/readonly
```

**Static roles (для существующих пользователей):**
```bash
vault write database/static-roles/static-account \
  db_name=postgresql \
  username="existing_user" \
  rotation_period="24h"

# Получить пароль (будет ротироваться)
vault read database/static-creds/static-account
```

### 💻 Задание

Настрой динамическую генерацию credentials для PostgreSQL:

1. **Запусти PostgreSQL в Docker:**

```bash
docker run --name postgres \
  -e POSTGRES_PASSWORD=rootpass \
  -e POSTGRES_DB=testdb \
  -p 5432:5432 \
  -d postgres:14
```

2. **Создай admin пользователя для Vault:**

```bash
docker exec -it postgres psql -U postgres -d testdb <<EOF
CREATE ROLE vaultadmin WITH LOGIN PASSWORD 'vaultpass' SUPERUSER;
EOF
```

3. **Настрой Database secrets engine:**

```bash
# Enable
vault secrets enable database

# Configure
vault write database/config/postgres \
  plugin_name=postgresql-database-plugin \
  allowed_roles="readonly,readwrite" \
  connection_url="postgresql://{{username}}:{{password}}@localhost:5432/testdb?sslmode=disable" \
  username="vaultadmin" \
  password="vaultpass"

# Test connection
vault read database/config/postgres
```

4. **Создай роли:**

```bash
# Readonly role
vault write database/roles/readonly \
  db_name=postgres \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
    GRANT CONNECT ON DATABASE testdb TO \"{{name}}\"; \
    GRANT USAGE ON SCHEMA public TO \"{{name}}\"; \
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\"; \
    ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

# Readwrite role
vault write database/roles/readwrite \
  db_name=postgres \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
    GRANT CONNECT ON DATABASE testdb TO \"{{name}}\"; \
    GRANT USAGE ON SCHEMA public TO \"{{name}}\"; \
    GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO \"{{name}}\"; \
    ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO \"{{name}}\";" \
  default_ttl="30m" \
  max_ttl="2h"
```

5. **Получи и протестируй credentials:**

```bash
# Получи readonly credentials
vault read database/creds/readonly

# Сохрани output
# username: v-token-readonly-xyz
# password: A1a-2b3c4d5e6f

# Тест подключения
PGPASSWORD='A1a-2b3c4d5e6f' psql -h localhost -U v-token-readonly-xyz -d testdb -c "SELECT 1;"

# Проверь что пользователь создан
docker exec -it postgres psql -U postgres -d testdb -c "\du"
```

6. **Создай таблицу и проверь permissions:**

```bash
# Создай тестовую таблицу
docker exec -it postgres psql -U postgres -d testdb <<EOF
CREATE TABLE test_data (
  id SERIAL PRIMARY KEY,
  value TEXT
);
INSERT INTO test_data (value) VALUES ('test1'), ('test2');
EOF

# Readonly может читать
PGPASSWORD='<readonly-pass>' psql -h localhost -U <readonly-user> -d testdb -c "SELECT * FROM test_data;"

# Readonly не может писать (должно быть ошибка)
PGPASSWORD='<readonly-pass>' psql -h localhost -U <readonly-user> -d testdb -c "INSERT INTO test_data (value) VALUES ('test3');"
```

7. **Тест lease renewal и revocation:**

```bash
# Получи credentials
CREDS=$(vault read -format=json database/creds/readonly)
LEASE_ID=$(echo $CREDS | jq -r .lease_id)

# Renew
vault lease renew $LEASE_ID

# Lookup lease
vault lease lookup $LEASE_ID

# Revoke
vault lease revoke $LEASE_ID

# Проверь что пользователь удален
docker exec -it postgres psql -U postgres -d testdb -c "\du" | grep v-token
```

### 🚀 Бонус (новое)

**1. Static roles с автоматической ротацией:**

```bash
# Создай статичного пользователя
docker exec -it postgres psql -U postgres -d testdb <<EOF
CREATE ROLE app_user WITH LOGIN PASSWORD 'initial_pass';
GRANT CONNECT ON DATABASE testdb TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_user;
EOF

# Настрой static role
vault write database/static-roles/app-user \
  db_name=postgres \
  username="app_user" \
  rotation_period="1h"

# Получи текущий пароль
vault read database/static-creds/app-user

# Vault будет автоматически ротировать пароль каждый час
```

**2. Root credentials rotation:**

```bash
# После настройки, ротируй root пароль для безопасности
vault write -f database/rotate-root/postgres

# Теперь только Vault знает root пароль
```

**3. Connection leasing (для pool optimization):**

```bash
vault write database/config/postgres \
  plugin_name=postgresql-database-plugin \
  connection_url="postgresql://{{username}}:{{password}}@localhost:5432/testdb" \
  username="vaultadmin" \
  password="vaultpass" \
  max_open_connections=5 \
  max_idle_connections=2 \
  max_connection_lifetime="1h"
```

**4. Custom SQL с revocation:**

```bash
vault write database/roles/custom-revoke \
  db_name=postgres \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';" \
  revocation_statements="REASSIGN OWNED BY \"{{name}}\" TO postgres; DROP OWNED BY \"{{name}}\"; DROP ROLE IF EXISTS \"{{name}}\";" \
  default_ttl="1h"
```

**5. Мониторинг и автоматизация:**

```bash
#!/bin/bash
# db-creds-monitor.sh

ROLE_NAME="readonly"
THRESHOLD=300  # 5 minutes

while true; do
  # Получи текущий lease
  LEASE_ID=$(vault list -format=json sys/leases/lookup/database/creds/$ROLE_NAME 2>/dev/null | jq -r '.[0]')
  
  if [ -n "$LEASE_ID" ] && [ "$LEASE_ID" != "null" ]; then
    # Проверь TTL
    TTL=$(vault lease lookup -format=json $LEASE_ID | jq -r .data.ttl)
    
    if [ $TTL -lt $THRESHOLD ]; then
      echo "Lease expiring soon (${TTL}s), renewing..."
      vault lease renew $LEASE_ID
    fi
  fi
  
  sleep 60
done
```

---

## Модуль 6: Transit Secrets Engine - Encryption as a Service (25 минут)

### 🎯 Напоминалка

**Transit Engine** - шифрование как сервис, без хранения данных.

**Возможности:**
- Encrypt/Decrypt данных
- Sign/Verify подписей
- Generate HMAC
- Generate random bytes
- Key rotation с automatic re-wrapping

**Типы ключей:**
```
aes128-gcm96   - AES-128 GCM (default)
aes256-gcm96   - AES-256 GCM
chacha20-poly1305 - ChaCha20-Poly1305
ed25519        - Ed25519 (signing)
ecdsa-p256     - ECDSA P-256 (signing)
ecdsa-p384     - ECDSA P-384 (signing)
ecdsa-p521     - ECDSA P-521 (signing)
rsa-2048       - RSA 2048 (signing/encryption)
rsa-3072       - RSA 3072 (signing/encryption)
rsa-4096       - RSA 4096 (signing/encryption)
```

**Базовое использование:**
```bash
# Enable
vault secrets enable transit

# Создать ключ
vault write -f transit/keys/my-key

# Encrypt
vault write transit/encrypt/my-key \
  plaintext=$(echo "secret data" | base64)

# Output: vault:v1:abc123def456...

# Decrypt
vault write transit/decrypt/my-key \
  ciphertext="vault:v1:abc123def456..."

# Decode
echo "c2VjcmV0IGRhdGE=" | base64 -d
```

**Key rotation:**
```bash
# Rotate key
vault write -f transit/keys/my-key/rotate

# Проверь версии
vault read transit/keys/my-key

# Re-encrypt старые данные
vault write transit/rewrap/my-key \
  ciphertext="vault:v1:old_data"
# Output: vault:v2:new_wrapped_data
```

**Convergent encryption (детерминированное):**
```bash
# Создать ключ с convergent encryption
vault write transit/keys/orders \
  convergent_encryption=true \
  derived=true

# Encrypt с контекстом
vault write transit/encrypt/orders \
  plaintext=$(echo "order-123" | base64) \
  context=$(echo "user-id-456" | base64)
```

**Signing и verification:**
```bash
# Создать signing ключ
vault write transit/keys/signatures \
  type=ed25519

# Sign data
vault write transit/sign/signatures \
  input=$(echo "important data" | base64)

# Output: vault:v1:signature_data

# Verify
vault write transit/verify/signatures \
  input=$(echo "important data" | base64) \
  signature="vault:v1:signature_data"
```

**HMAC:**
```bash
vault write transit/hmac/my-key \
  input=$(echo "data to hmac" | base64)
```

**Random bytes:**
```bash
# Generate 32 random bytes
vault write transit/random/32

# Base64 encoded
vault write transit/random/32 format=base64
```

**Key configuration:**
```bash
# Минимальная версия для decrypt
vault write transit/keys/my-key/config \
  min_decryption_version=2

# Минимальная версия для encrypt
vault write transit/keys/my-key/config \
  min_encryption_version=3

# Запретить plaintext export
vault write transit/keys/my-key/config \
  exportable=false

# Разрешить deletion
vault write transit/keys/my-key/config \
  deletion_allowed=true
```

**Batch operations (для производительности):**
```bash
# Batch encrypt
vault write transit/encrypt/my-key \
  batch_input='[
    {"plaintext":"'"$(echo "data1" | base64)"'"},
    {"plaintext":"'"$(echo "data2" | base64)"'"},
    {"plaintext":"'"$(echo "data3" | base64)"'"}
  ]'

# Batch decrypt
vault write transit/decrypt/my-key \
  batch_input='[
    {"ciphertext":"vault:v1:..."},
    {"ciphertext":"vault:v1:..."}
  ]'
```

### 💻 Задание

Создай систему шифрования для приложения:

1. **Enable Transit и создай ключи:**

```bash
# Enable
vault secrets enable transit

# Encryption key
vault write -f transit/keys/customer-data

# Signing key
vault write transit/keys/api-signatures type=ed25519

# Convergent encryption key (для поиска)
vault write transit/keys/search-index \
  convergent_encryption=true \
  derived=true
```

2. **Encrypt и decrypt данных:**

```bash
# Prepare data
PLAINTEXT=$(echo "John Doe" | base64)
EMAIL=$(echo "john@example.com" | base64)
SSN=$(echo "123-45-6789" | base64)

# Encrypt
NAME_CIPHER=$(vault write -field=ciphertext transit/encrypt/customer-data plaintext=$PLAINTEXT)
EMAIL_CIPHER=$(vault write -field=ciphertext transit/encrypt/customer-data plaintext=$EMAIL)
SSN_CIPHER=$(vault write -field=ciphertext transit/encrypt/customer-data plaintext=$SSN)

echo "Encrypted name: $NAME_CIPHER"
echo "Encrypted email: $EMAIL_CIPHER"
echo "Encrypted SSN: $SSN_CIPHER"

# Decrypt
vault write -field=plaintext transit/decrypt/customer-data ciphertext=$NAME_CIPHER | base64 -d
vault write -field=plaintext transit/decrypt/customer-data ciphertext=$EMAIL_CIPHER | base64 -d
```

3. **Batch операции для производительности:**

```bash
# Создай batch input
cat > batch-encrypt.json <<EOF
{
  "batch_input": [
    {"plaintext": "$(echo "customer1@example.com" | base64)"},
    {"plaintext": "$(echo "customer2@example.com" | base64)"},
    {"plaintext": "$(echo "customer3@example.com" | base64)"}
  ]
}
EOF

# Batch encrypt
vault write -format=json transit/encrypt/customer-data @batch-encrypt.json | jq

# Batch decrypt
cat > batch-decrypt.json <<EOF
{
  "batch_input": [
    {"ciphertext": "$CIPHER1"},
    {"ciphertext": "$CIPHER2"}
  ]
}
EOF

vault write transit/decrypt/customer-data @batch-decrypt.json
```

4. **Sign и verify:**

```bash
# Sign API request
API_DATA="GET /api/users?page=1"
INPUT=$(echo "$API_DATA" | base64)

SIGNATURE=$(vault write -field=signature transit/sign/api-signatures input=$INPUT)
echo "Signature: $SIGNATURE"

# Verify
vault write transit/verify/api-signatures \
  input=$INPUT \
  signature=$SIGNATURE
```

5. **Key rotation и rewrap:**

```bash
# Rotate key
vault write -f transit/keys/customer-data/rotate

# Проверь версии
vault read transit/keys/customer-data

# Rewrap старые данные (обновление до новой версии)
NEW_CIPHER=$(vault write -field=ciphertext transit/rewrap/customer-data ciphertext=$NAME_CIPHER)
echo "Old: $NAME_CIPHER"
echo "New: $NEW_CIPHER"

# Decrypt работает с обеими версиями
vault write -field=plaintext transit/decrypt/customer-data ciphertext=$NAME_CIPHER | base64 -d
vault write -field=plaintext transit/decrypt/customer-data ciphertext=$NEW_CIPHER | base64 -d
```

6. **Convergent encryption для поиска:**

```bash
# Encrypt с контекстом (одинаковые данные + контекст = одинаковый ciphertext)
CONTEXT=$(echo "user-123" | base64)
EMAIL1=$(echo "john@example.com" | base64)

CIPHER1=$(vault write -field=ciphertext transit/encrypt/search-index \
  plaintext=$EMAIL1 \
  context=$CONTEXT)

CIPHER2=$(vault write -field=ciphertext transit/encrypt/search-index \
  plaintext=$EMAIL1 \
  context=$CONTEXT)

# Ciphertexts идентичны!
echo "Cipher1: $CIPHER1"
echo "Cipher2: $CIPHER2"
```

### 🚀 Бонус (новое)

**1. Создай encryption wrapper для приложения:**

```bash
#!/bin/bash
# vault-encrypt.sh

VAULT_ADDR=${VAULT_ADDR:-http://127.0.0.1:8200}
KEY_NAME=${1:-customer-data}
OPERATION=${2:-encrypt}

if [ "$OPERATION" == "encrypt" ]; then
  # Read from stdin
  DATA=$(cat)
  PLAINTEXT=$(echo "$DATA" | base64 -w0)
  
  # Encrypt
  CIPHER=$(vault write -field=ciphertext transit/encrypt/$KEY_NAME plaintext=$PLAINTEXT)
  echo "$CIPHER"
  
elif [ "$OPERATION" == "decrypt" ]; then
  # Read ciphertext from stdin
  CIPHER=$(cat)
  
  # Decrypt
  PLAINTEXT=$(vault write -field=plaintext transit/decrypt/$KEY_NAME ciphertext=$CIPHER)
  echo "$PLAINTEXT" | base64 -d
fi

# Usage:
# echo "secret data" | ./vault-encrypt.sh customer-data encrypt
# echo "vault:v1:..." | ./vault-encrypt.sh customer-data decrypt
```

**2. Auto-rewrap script для key rotation:**

```bash
#!/bin/bash
# auto-rewrap.sh

KEY_NAME=$1
CIPHERTEXT_FILE=$2

if [ ! -f "$CIPHERTEXT_FILE" ]; then
  echo "File not found: $CIPHERTEXT_FILE"
  exit 1
fi

# Получи текущую версию ключа
CURRENT_VERSION=$(vault read -format=json transit/keys/$KEY_NAME | jq -r .data.latest_version)

# Rewrap все ciphertexts
while IFS= read -r line; do
  # Проверь версию в ciphertext
  VERSION=$(echo "$line" | cut -d':' -f2 | cut -d':' -f1)
  
  if [ "$VERSION" != "v$CURRENT_VERSION" ]; then
    echo "Rewrapping: $line"
    NEW_CIPHER=$(vault write -field=ciphertext transit/rewrap/$KEY_NAME ciphertext="$line")
    echo "$NEW_CIPHER"
  else
    echo "$line"
  fi
done < "$CIPHERTEXT_FILE"
```

**3. Structured encryption (JSON fields):**

```bash
#!/bin/bash
# encrypt-json-fields.sh

KEY_NAME="customer-data"

# Input JSON
JSON='{"name":"John Doe","email":"john@example.com","age":30}'

# Extract и encrypt sensitive fields
NAME=$(echo "$JSON" | jq -r .name)
EMAIL=$(echo "$JSON" | jq -r .email)

NAME_ENC=$(vault write -field=ciphertext transit/encrypt/$KEY_NAME plaintext=$(echo "$NAME" | base64))
EMAIL_ENC=$(vault write -field=ciphertext transit/encrypt/$KEY_NAME plaintext=$(echo "$EMAIL" | base64))

# Создай новый JSON с зашифрованными полями
echo "$JSON" | jq \
  --arg name "$NAME_ENC" \
  --arg email "$EMAIL_ENC" \
  '.name = $name | .email = $email'
```

**4. Performance testing:**

```bash
#!/bin/bash
# transit-benchmark.sh

KEY_NAME="customer-data"
ITERATIONS=1000

echo "Encrypting $ITERATIONS values..."
time {
  for i in $(seq 1 $ITERATIONS); do
    echo "data-$i" | base64 | \
      xargs -I {} vault write -field=ciphertext transit/encrypt/$KEY_NAME plaintext={} > /dev/null
  done
}

echo "Testing batch encryption..."
# Создай batch input
BATCH_INPUT='{"batch_input":['
for i in $(seq 1 100); do
  BATCH_INPUT+="{\"plaintext\":\"$(echo "data-$i" | base64)\"},"
done
BATCH_INPUT="${BATCH_INPUT%,}]}"

time {
  for i in $(seq 1 10); do
    echo "$BATCH_INPUT" | vault write transit/encrypt/$KEY_NAME - > /dev/null
  done
}
```

---

## Модуль 7: PKI Secrets Engine - Certificate Management (30 минут)

### 🎯 Напоминалка

**PKI Engine** - управление TLS/SSL сертификатами.

**Возможности:**
- Создание Root и Intermediate CA
- Генерация сертификатов
- Revocation lists (CRL)
- OCSP responder
- Automated certificate rotation

**Setup Root CA:**
```bash
# Enable PKI
vault secrets enable pki

# Tune TTL (max 10 years)
vault secrets tune -max-lease-ttl=87600h pki

# Generate root CA
vault write -field=certificate pki/root/generate/internal \
  common_name="My Root CA" \
  ttl=87600h > root_ca.crt

# Configure URLs
vault write pki/config/urls \
  issuing_certificates="http://127.0.0.1:8200/v1/pki/ca" \
  crl_distribution_points="http://127.0.0.1:8200/v1/pki/crl"
```

**Intermediate CA:**
```bash
# Enable intermediate PKI
vault secrets enable -path=pki_int pki

# Tune TTL (5 years)
vault secrets tune -max-lease-ttl=43800h pki_int

# Generate CSR
vault write -format=json pki_int/intermediate/generate/internal \
  common_name="My Intermediate CA" \
  | jq -r '.data.csr' > pki_intermediate.csr

# Sign with root CA
vault write -format=json pki/root/sign-intermediate \
  csr=@pki_intermediate.csr \
  format=pem_bundle \
  ttl=43800h \
  | jq -r '.data.certificate' > intermediate.cert.pem

# Set signed certificate
vault write pki_int/intermediate/set-signed \
  certificate=@intermediate.cert.pem
```

**Create role:**
```bash
vault write pki_int/roles/example-dot-com \
  allowed_domains=example.com \
  allow_subdomains=true \
  max_ttl=72h
```

**Issue certificate:**
```bash
vault write pki_int/issue/example-dot-com \
  common_name=test.example.com \
  ttl=24h
```

**Revoke certificate:**
```bash
vault write pki_int/revoke \
  serial_number=<serial>

# Check CRL
curl http://127.0.0.1:8200/v1/pki_int/crl
```

### 💻 Задание

Создай полную PKI инфраструктуру:

1. **Root CA setup:**
```bash
# Enable root PKI
vault secrets enable pki
vault secrets tune -max-lease-ttl=87600h pki

# Generate root
vault write -field=certificate pki/root/generate/internal \
  common_name="Example Root CA" \
  ttl=87600h > root_ca.crt

# Configure URLs
vault write pki/config/urls \
  issuing_certificates="http://127.0.0.1:8200/v1/pki/ca" \
  crl_distribution_points="http://127.0.0.1:8200/v1/pki/crl"
```

2. **Intermediate CA:**
```bash
vault secrets enable -path=pki_int pki
vault secrets tune -max-lease-ttl=43800h pki_int

vault write -format=json pki_int/intermediate/generate/internal \
  common_name="Example Intermediate CA" \
  | jq -r '.data.csr' > pki_int.csr

vault write -format=json pki/root/sign-intermediate \
  csr=@pki_int.csr \
  format=pem_bundle \
  ttl=43800h \
  | jq -r '.data.certificate' > intermediate.cert.pem

vault write pki_int/intermediate/set-signed \
  certificate=@intermediate.cert.pem

vault write pki_int/config/urls \
  issuing_certificates="http://127.0.0.1:8200/v1/pki_int/ca" \
  crl_distribution_points="http://127.0.0.1:8200/v1/pki_int/crl"
```

3. **Create roles:**
```bash
# Web servers
vault write pki_int/roles/web-server \
  allowed_domains=example.com \
  allow_subdomains=true \
  max_ttl=720h \
  key_type=rsa \
  key_bits=2048

# Internal services
vault write pki_int/roles/internal \
  allowed_domains=internal.example.com \
  allow_subdomains=true \
  allow_bare_domains=true \
  max_ttl=168h
```

4. **Issue certificates:**
```bash
# Web certificate
vault write -format=json pki_int/issue/web-server \
  common_name=www.example.com \
  ttl=720h > www_cert.json

# Extract components
cat www_cert.json | jq -r .data.certificate > www.crt
cat www_cert.json | jq -r .data.private_key > www.key
cat www_cert.json | jq -r .data.ca_chain[] > ca_chain.crt

# Verify
openssl x509 -in www.crt -text -noout
```

5. **Test with nginx:**
```bash
cat > nginx.conf <<EOF
server {
    listen 443 ssl;
    server_name www.example.com;
    
    ssl_certificate /etc/nginx/certs/www.crt;
    ssl_certificate_key /etc/nginx/certs/www.key;
    
    location / {
        return 200 "Secured by Vault PKI!";
    }
}
EOF
```

### 🚀 Бонус (новое)

**1. Automated certificate renewal script:**
```bash
#!/bin/bash
# renew-cert.sh

ROLE="web-server"
COMMON_NAME="www.example.com"
CERT_DIR="/etc/nginx/certs"
RENEWAL_THRESHOLD=86400  # 1 day

# Check current certificate
if [ -f "$CERT_DIR/www.crt" ]; then
  EXPIRY=$(openssl x509 -in "$CERT_DIR/www.crt" -noout -enddate | cut -d= -f2)
  EXPIRY_EPOCH=$(date -d "$EXPIRY" +%s)
  NOW_EPOCH=$(date +%s)
  TIME_LEFT=$((EXPIRY_EPOCH - NOW_EPOCH))
  
  if [ $TIME_LEFT -gt $RENEWAL_THRESHOLD ]; then
    echo "Certificate valid for $((TIME_LEFT / 86400)) days, skipping renewal"
    exit 0
  fi
fi

# Issue new certificate
vault write -format=json pki_int/issue/$ROLE \
  common_name=$COMMON_NAME \
  ttl=720h > /tmp/new_cert.json

# Extract and save
jq -r .data.certificate /tmp/new_cert.json > "$CERT_DIR/www.crt"
jq -r .data.private_key /tmp/new_cert.json > "$CERT_DIR/www.key"
jq -r .data.ca_chain[] /tmp/new_cert.json > "$CERT_DIR/ca_chain.crt"

# Reload nginx
nginx -s reload

rm /tmp/new_cert.json
echo "Certificate renewed successfully"
```

**2. Certificate with SANs:**
```bash
vault write pki_int/issue/web-server \
  common_name=www.example.com \
  alt_names="example.com,api.example.com,admin.example.com" \
  ip_sans="10.0.1.100" \
  ttl=720h
```

---

## Модуль 8: Monitoring, Audit и Production Best Practices (30 минут)

### 🎯 Напоминалка

**Audit Devices:**
```bash
# File audit
vault audit enable file file_path=/var/log/vault_audit.log

# Syslog
vault audit enable syslog

# Socket
vault audit enable socket address=127.0.0.1:9090 socket_type=tcp

# List
vault audit list

# Disable
vault audit disable file/
```

**Audit log format:**
```json
{
  "time": "2024-01-15T10:30:00Z",
  "type": "request",
  "auth": {
    "client_token": "hmac-sha256:...",
    "accessor": "hmac-sha256:...",
    "display_name": "token",
    "policies": ["default", "dev-policy"]
  },
  "request": {
    "id": "abc-123",
    "operation": "update",
    "path": "secret/data/myapp",
    "data": {
      "password": "hmac-sha256:..."
    }
  }
}
```

**Metrics и Telemetry:**
```hcl
telemetry {
  prometheus_retention_time = "30s"
  disable_hostname = true
}
```

**Health checks:**
```bash
# Sys health (200 if initialized, unsealed, and active)
curl http://127.0.0.1:8200/v1/sys/health

# Seal status
vault status
curl http://127.0.0.1:8200/v1/sys/seal-status

# Leader status (HA)
curl http://127.0.0.1:8200/v1/sys/leader
```

**Prometheus metrics:**
```bash
# Enable
vault write sys/metrics/config enabled=true

# Scrape
curl http://127.0.0.1:8200/v1/sys/metrics?format=prometheus
```

**Production Best Practices:**

**1. High Availability:**
```hcl
# config.hcl with Raft
storage "raft" {
  path    = "/opt/vault/data"
  node_id = "vault-1"
  
  retry_join {
    leader_api_addr = "http://vault-2:8200"
  }
  retry_join {
    leader_api_addr = "http://vault-3:8200"
  }
}

listener "tcp" {
  address = "0.0.0.0:8200"
  tls_cert_file = "/opt/vault/tls/vault.crt"
  tls_key_file = "/opt/vault/tls/vault.key"
}

api_addr = "https://vault-1.example.com:8200"
cluster_addr = "https://vault-1.example.com:8201"
```

**2. Seal/Unseal:**
```bash
# Auto-unseal с AWS KMS
seal "awskms" {
  region     = "us-east-1"
  kms_key_id = "alias/vault-unseal"
}

# Auto-unseal с Azure Key Vault
seal "azurekeyvault" {
  tenant_id      = "..."
  client_id      = "..."
  client_secret  = "..."
  vault_name     = "vault-keys"
  key_name       = "vault-unseal"
}
```

**3. Backup:**
```bash
# Raft snapshot (для Raft storage)
vault operator raft snapshot save backup.snap

# Restore
vault operator raft snapshot restore backup.snap

# Automated backup script
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
vault operator raft snapshot save /backups/vault_$DATE.snap
# Rotate old backups
find /backups -name "vault_*.snap" -mtime +7 -delete
```

**4. Token lifecycle:**
```bash
# Batch tokens (performance)
vault token create -type=batch -policy=app-policy

# Orphan tokens (для сервисов)
vault token create -orphan -policy=service-policy -period=24h

# Token TTL management
vault token create -policy=app-policy \
  -ttl=1h \
  -renewable=true \
  -explicit-max-ttl=24h
```

### 💻 Задание

Настрой production-ready мониторинг:

1. **Enable audit logging:**
```bash
# File audit
sudo mkdir -p /var/log/vault
sudo vault audit enable file file_path=/var/log/vault/audit.log

# Test
vault kv put secret/test foo=bar
tail -f /var/log/vault/audit.log | jq
```

2. **Setup Prometheus monitoring:**
```bash
# Enable telemetry
vault write sys/metrics/config \
  enabled=true \
  enable_hostname_label=true

# Prometheus config
cat > prometheus.yml <<EOF
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'vault'
    metrics_path: '/v1/sys/metrics'
    params:
      format: ['prometheus']
    bearer_token: 'root'  # Use proper token in production
    static_configs:
      - targets: ['localhost:8200']
EOF

# Run Prometheus
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

3. **Health check script:**
```bash
#!/bin/bash
# vault-health.sh

VAULT_ADDR=${VAULT_ADDR:-http://127.0.0.1:8200}

# Check health
HEALTH=$(curl -s $VAULT_ADDR/v1/sys/health)
INITIALIZED=$(echo $HEALTH | jq -r .initialized)
SEALED=$(echo $HEALTH | jq -r .sealed)
STANDBY=$(echo $HEALTH | jq -r .standby)

if [ "$INITIALIZED" != "true" ]; then
  echo "ERROR: Vault not initialized"
  exit 1
fi

if [ "$SEALED" = "true" ]; then
  echo "ERROR: Vault is sealed"
  exit 1
fi

if [ "$STANDBY" = "true" ]; then
  echo "WARNING: Vault is in standby mode"
fi

echo "Vault is healthy"
exit 0
```

4. **Backup automation:**
```bash
#!/bin/bash
# vault-backup.sh

BACKUP_DIR="/backups/vault"
RETENTION_DAYS=7

mkdir -p $BACKUP_DIR

# Create snapshot
DATE=$(date +%Y%m%d_%H%M%S)
vault operator raft snapshot save $BACKUP_DIR/vault_$DATE.snap

if [ $? -eq 0 ]; then
  echo "Backup created: vault_$DATE.snap"
  
  # Cleanup old backups
  find $BACKUP_DIR -name "vault_*.snap" -mtime +$RETENTION_DAYS -delete
  
  # Upload to S3 (optional)
  # aws s3 cp $BACKUP_DIR/vault_$DATE.snap s3://vault-backups/
else
  echo "Backup failed!"
  exit 1
fi
```

5. **Audit log analysis:**
```bash
# Failed authentications
jq 'select(.error != null and .type == "request")' /var/log/vault/audit.log

# Top users by requests
jq -r 'select(.type == "request") | .auth.display_name' /var/log/vault/audit.log | \
  sort | uniq -c | sort -rn | head -10

# Secret access patterns
jq -r 'select(.type == "request" and .request.path | startswith("secret/")) | 
  .request.path' /var/log/vault/audit.log | \
  sort | uniq -c | sort -rn
```

### 🚀 Бонус (новое)

**1. Grafana dashboard для Vault:**
```bash
# Run Grafana
docker run -d \
  --name grafana \
  -p 3000:3000 \
  grafana/grafana

# Import Vault dashboard
# Dashboard ID: 12904 (from grafana.com)
```

**2. Alerting с AlertManager:**
```yaml
# alertmanager.yml
route:
  receiver: 'vault-alerts'
  group_by: ['alertname']
  
receivers:
  - name: 'vault-alerts'
    webhook_configs:
      - url: 'http://slack-webhook'

# Prometheus alerts
groups:
  - name: vault
    rules:
      - alert: VaultSealed
        expr: vault_core_unsealed == 0
        for: 5m
        annotations:
          summary: "Vault is sealed"
          
      - alert: VaultDown
        expr: up{job="vault"} == 0
        for: 5m
        annotations:
          summary: "Vault is down"
```

**3. Log forwarding с Fluentd:**
```conf
# fluentd.conf
<source>
  @type tail
  path /var/log/vault/audit.log
  pos_file /var/log/vault/audit.log.pos
  tag vault.audit
  format json
</source>

<match vault.audit>
  @type elasticsearch
  host elasticsearch
  port 9200
  index_name vault-audit
  type_name audit
</match>
```

---

## Финальный проект (60 минут)

### Задача: Развернуть полноценную Vault инфраструктуру

Создай production-ready Vault setup для микросервисного приложения.

**Требования:**

1. **Infrastructure:**
   - 3-node Raft cluster
   - TLS enabled
   - Auto-unseal (simulate с Transit)
   - Audit logging

2. **Secrets Management:**
   - KV v2 для app configs
   - Database dynamic credentials (PostgreSQL)
   - Transit encryption для PII
   - PKI для internal certificates

3. **Authentication:**
   - AppRole для services
   - UserPass для developers
   - Kubernetes auth (simulate)

4. **Policies:**
   - Admin policy
   - Dev policy
   - Service policies (read-only secrets)
   - CI/CD policy

5. **Monitoring:**
   - Prometheus metrics
   - Audit logging
   - Health checks
   - Backup automation

6. **Documentation:**
   - Architecture diagram
   - Deployment guide
   - Runbook для common operations

**Пример структуры:**
```
vault-production/
├── config/
│   ├── vault-node1.hcl
│   ├── vault-node2.hcl
│   └── vault-node3.hcl
├── policies/
│   ├── admin.hcl
│   ├── dev.hcl
│   ├── cicd.hcl
│   └── service.hcl
├── scripts/
│   ├── init-cluster.sh
│   ├── setup-auth.sh
│   ├── setup-secrets.sh
│   ├── backup.sh
│   └── health-check.sh
├── monitoring/
│   ├── prometheus.yml
│   └── grafana-dashboard.json
└── README.md
```

---

## Справочная секция: Быстрые шпаргалки

### Vault CLI shortcuts

```bash
# Aliases
alias v='vault'
alias vl='vault login'
alias vr='vault read'
alias vw='vault write'
alias vd='vault delete'
alias vlist='vault list'

# Common commands
vault status                           # Статус сервера
vault login                            # Интерактивный логин
vault kv get secret/path               # Читать секрет
vault kv put secret/path key=value     # Записать секрет
vault token lookup                     # Инфо о текущем токене
vault policy list                      # Список policies
vault auth list                        # Список auth methods
vault secrets list                     # Список secrets engines

# JSON output
vault kv get -format=json secret/path | jq
vault token lookup -format=json | jq .data.policies
```

### API примеры

```bash
# Health check
curl $VAULT_ADDR/v1/sys/health

# Read secret
curl -H "X-Vault-Token: $VAULT_TOKEN" \
  $VAULT_ADDR/v1/secret/data/myapp | jq

# Write secret
curl -H "X-Vault-Token: $VAULT_TOKEN" \
  -X POST \
  -d '{"data":{"password":"secret123"}}' \
  $VAULT_ADDR/v1/secret/data/myapp

# List
curl -H "X-Vault-Token: $VAULT_TOKEN" \
  -X LIST \
  $VAULT_ADDR/v1/secret/metadata | jq

# Token lookup
curl -H "X-Vault-Token: $VAULT_TOKEN" \
  $VAULT_ADDR/v1/auth/token/lookup-self | jq
```

### Troubleshooting Guide

**Vault не запускается:**
```bash
# Проверь logs
journalctl -u vault -f

# Проверь конфиг
vault server -config=/etc/vault/config.hcl -log-level=debug

# Проверь permissions
ls -la /opt/vault/data
```

**Vault sealed:**
```bash
# Check status
vault status

# Unseal
vault operator unseal <key1>
vault operator unseal <key2>
vault operator unseal <key3>

# Auto-unseal troubleshooting
vault read sys/seal-status
```

**Permission denied:**
```bash
# Проверь токен
vault token lookup

# Проверь capabilities
vault token capabilities secret/data/myapp

# Проверь policy
vault policy read myapp-policy
```

**Performance issues:**
```bash
# Check metrics
curl $VAULT_ADDR/v1/sys/metrics?format=prometheus

# Connection pool
vault read sys/storage/raft/configuration

# Batch operations
# Используй batch API вместо множественных запросов
```

### Production Checklist

**Перед деплоем:**
- ✅ TLS настроен для всех endpoints
- ✅ Auto-unseal настроен (KMS/Transit)
- ✅ HA cluster с 3+ nodes
- ✅ Audit logging включен
- ✅ Backup автоматизирован
- ✅ Monitoring и alerting настроены
- ✅ Resource limits установлены
- ✅ Network policies настроены
- ✅ Root token удален/сохранен безопасно
- ✅ Unseal keys распределены (Shamir)
- ✅ Policies протестированы
- ✅ Disaster recovery plan задокументирован
- ✅ Runbooks созданы

**Security hardening:**
- ✅ Минимальные privileges для policies
- ✅ Short TTL для tokens
- ✅ CIDR binding где возможно
- ✅ Namespace isolation (Enterprise)
- ✅ Sentinel policies (Enterprise)
- ✅ MFA enabled (Enterprise)
- ✅ Rate limiting настроен
- ✅ Audit log rotation настроена

### Useful Scripts

**Token renewal daemon:**
```bash
#!/bin/bash
# token-renewer.sh

while true; do
  # Renew token
  vault token renew -increment=3600
  
  # Sleep for 50 minutes (10min buffer)
  sleep 3000
done
```

**Secret rotation:**
```bash
#!/bin/bash
# rotate-database-creds.sh

APP_NAME=$1

# Get new credentials
NEW_CREDS=$(vault read -format=json database/creds/readonly)
NEW_USER=$(echo $NEW_CREDS | jq -r .data.username)
NEW_PASS=$(echo $NEW_CREDS | jq -r .data.password)

# Update application config
vault kv put secret/$APP_NAME/db \
  username=$NEW_USER \
  password=$NEW_PASS

# Restart application
kubectl rollout restart deployment/$APP_NAME
```

---

## План повторения

### При первом прохождении (2-3 часа):
- Пройди модули 1-5 обязательно
- Модули 6-7 по желанию
- Финальный проект упрощенный

### При повторном прохождении (через 6-12 месяцев):
- Бегло просмотри теорию
- Сфокусируйся на бонусных заданиях
- Пройди модули 8 обязательно
- Финальный проект полностью
- Добавь свои кастомизации

### Для закрепления:
- Интегрируй Vault в свои проекты
- Настрой Vault в Kubernetes
- Попробуй Enterprise features (trial)
- Изучи HashiCorp Boundary + Vault
- Получи сертификацию Vault Associate
- Изучи Vault Agent и Templates

### Дополнительные ресурсы:
- **Vault Documentation** - официальная документация
- **Learn HashiCorp Vault** - интерактивные туториалы
- **Vault GitHub** - примеры и issues
- **HashiCorp Forum** - community support
- **Awesome Vault** - curated resources
- **Vault Slack** - сообщество

---

## Чек-лист навыков

После прохождения курса ты должен уметь:

### Базовые навыки:
- ✅ Устанавливать и запускать Vault
- ✅ Seal/Unseal операции
- ✅ Работать с KV secrets
- ✅ Создавать и применять policies
- ✅ Настраивать auth methods
- ✅ Управлять tokens

### Продвинутые навыки:
- ✅ Настраивать Database secrets engine
- ✅ Использовать Transit для encryption
- ✅ Управлять PKI инфраструктурой
- ✅ Настраивать audit logging
- ✅ Работать с API
- ✅ Автоматизировать operations

### Expert навыки:
- ✅ Настраивать HA cluster
- ✅ Внедрять auto-unseal
- ✅ Настраивать monitoring
- ✅ Backup и disaster recovery
- ✅ Performance tuning
- ✅ Security hardening

### Архитектурные навыки:
- ✅ Проектировать secrets management
- ✅ Интегрировать с K8s/Cloud
- ✅ Планировать disaster recovery
- ✅ Оптимизировать для production
- ✅ Troubleshooting сложных проблем

---

## Заключение

Поздравляю! Ты прошел курс по освежению знаний HashiCorp Vault.

**Следующие шаги:**
1. Практикуйся регулярно - используй Vault в своих проектах
2. Автоматизируй всё - secrets rotation, backup, monitoring
3. Изучи смежные технологии: Consul, Nomad, Boundary
4. Получи сертификацию HashiCorp Vault Associate
5. Делись знаниями - помогай новичкам в community

**Помни:**
- Vault - это инструмент безопасности, используй его правильно
- Начинай с простого, усложняй постепенно
- Документация - твой лучший друг
- Community очень дружелюбное и готово помочь
- Безопасность - это процесс, а не результат

Проходи этот курс каждые 6-12 месяцев, чтобы оставаться в форме. Каждый раз ты будешь узнавать что-то новое и замечать, как выросли твои навыки!

Happy Vault learning! 🔐🚀

---

**Версия курса:** 1.0  
**Последнее обновление:** Декабрь 2024  
**Версия Vault:** 1.15+

**Обратная связь приветствуется!** Если нашел ошибки или есть предложения - создавай issues или pull requests.