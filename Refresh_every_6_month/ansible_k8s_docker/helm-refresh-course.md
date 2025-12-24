# Helm Refresh: Ежегодный/Полугодовой курс для DevOps

**Цель:** Освежить в памяти ключевые концепции Helm за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная задача, которую нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Предварительные требования:**
- Доступ к K8s кластеру (minikube, kind, k3s или облачный)
- kubectl установлен и настроен
- Базовые знания Kubernetes (Pods, Deployments, Services)
- Helm 3.x установлен

**Установка Helm:**
```bash
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Windows (Chocolatey)
choco install kubernetes-helm

# Проверка
helm version
```

---

## Модуль 1: Основы Helm и первый Chart (20 минут)

### 🎯 Напоминалка

**Что такое Helm:**
- Package manager для Kubernetes
- Helm Chart = пакет с K8s манифестами
- Helm Release = установленный Chart в кластере
- Helm Hub/Repository = хранилище Charts

**Основные концепции:**
```
Chart (пакет)
├── Chart.yaml          # Метаданные chart
├── values.yaml         # Конфигурация по умолчанию
├── templates/          # K8s манифесты с шаблонами
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl    # Helper функции
│   └── NOTES.txt       # Инструкции после установки
├── charts/             # Зависимости (sub-charts)
└── .helmignore         # Игнорируемые файлы
```

**Основные команды:**
```bash
# Работа с репозиториями
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
helm search hub wordpress

# Установка Chart
helm install my-release bitnami/nginx
helm install my-release bitnami/nginx -n production --create-namespace
helm install my-release bitnami/nginx --set service.type=LoadBalancer
helm install my-release bitnami/nginx -f custom-values.yaml
helm install my-release ./my-chart  # Локальный chart

# Управление releases
helm list
helm list -A                          # Все namespaces
helm status my-release
helm get values my-release            # Посмотреть values
helm get manifest my-release          # Посмотреть манифесты
helm history my-release

# Обновление
helm upgrade my-release bitnami/nginx
helm upgrade my-release bitnami/nginx --set replicas=3
helm upgrade my-release bitnami/nginx -f new-values.yaml
helm upgrade --install my-release bitnami/nginx  # Установить или обновить

# Откат
helm rollback my-release 1

# Удаление
helm uninstall my-release
helm uninstall my-release --keep-history

# Отладка
helm template my-release ./my-chart   # Рендер без установки
helm template my-release ./my-chart --debug
helm install my-release ./my-chart --dry-run --debug
helm lint ./my-chart                  # Проверка синтаксиса
```

**Структура Chart.yaml:**
```yaml
apiVersion: v2                    # Chart API версия (v2 для Helm 3)
name: my-app                      # Имя chart
description: My awesome app       # Описание
type: application                 # application или library
version: 0.1.0                    # Версия chart
appVersion: "1.0.0"              # Версия приложения
keywords:
  - web
  - application
maintainers:
  - name: Your Name
    email: you@example.com
dependencies:                     # Зависимости
  - name: postgresql
    version: 12.x.x
    repository: https://charts.bitnami.com/bitnami
```

**Values.yaml - конфигурация по умолчанию:**
```yaml
replicaCount: 2

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.21-alpine"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: "nginx"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: Prefix

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

**Template синтаксис (Go templates):**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-chart.fullname" . }}
  labels:
    {{- include "my-chart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-chart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-chart.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: 80
        {{- if .Values.resources }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        {{- end }}
```

**Встроенные объекты:**
```yaml
{{ .Release.Name }}        # Имя release
{{ .Release.Namespace }}   # Namespace
{{ .Release.Service }}     # Helm
{{ .Chart.Name }}          # Имя chart
{{ .Chart.Version }}       # Версия chart
{{ .Chart.AppVersion }}    # Версия приложения
{{ .Values.key }}          # Значение из values.yaml
{{ .Files.Get "file.txt" }} # Содержимое файла
{{ .Capabilities.KubeVersion }} # Версия K8s
```

**Функции:**
```yaml
# Строковые
{{ .Values.name | quote }}              # "value"
{{ .Values.name | upper }}              # VALUE
{{ .Values.name | lower }}              # value
{{ .Values.name | title }}              # Value
{{ .Values.name | trim }}               # удалить пробелы
{{ .Values.name | default "default" }}  # значение по умолчанию

# Условия
{{- if .Values.enabled }}
enabled: true
{{- else }}
enabled: false
{{- end }}

# Циклы
{{- range .Values.items }}
- {{ . }}
{{- end }}

# YAML/JSON
{{- toYaml .Values.resources | nindent 2 }}
{{- toJson .Values.data }}

# Форматирование
{{- nindent 4 "text" }}    # отступ 4 пробела
{{- indent 2 "text" }}     # отступ 2 пробела
```

### 💻 Задание

Создай свой первый Helm Chart:

1. **Создай базовую структуру:**
```bash
helm create my-webapp
cd my-webapp
```

2. **Изучи структуру:**
```bash
tree .
cat Chart.yaml
cat values.yaml
cat templates/deployment.yaml
```

3. **Настрой values.yaml:**
```yaml
replicaCount: 2

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "alpine"

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

4. **Проверь синтаксис:**
```bash
helm lint .
```

5. **Посмотри что сгенерируется:**
```bash
helm template my-webapp . --debug
```

6. **Установи chart:**
```bash
helm install my-webapp . -n test --create-namespace
```

7. **Проверь установку:**
```bash
helm list -n test
kubectl get all -n test
helm get values my-webapp -n test
helm get manifest my-webapp -n test
```

8. **Обнови конфигурацию:**
```bash
# Увеличь replicas до 3
helm upgrade my-webapp . -n test --set replicaCount=3

# Проверь
kubectl get pods -n test
helm history my-webapp -n test
```

9. **Откати изменения:**
```bash
helm rollback my-webapp 1 -n test
```

10. **Удали release:**
```bash
helm uninstall my-webapp -n test
```

### 🚀 Бонус (новое)

**1. Используй helm diff plugin для preview изменений:**
```bash
# Установка
helm plugin install https://github.com/databus23/helm-diff

# Использование
helm diff upgrade my-webapp . -n test --set replicaCount=5
```

**2. Создай custom helper функцию в _helpers.tpl:**
```yaml
{{/*
Create a custom environment label
*/}}
{{- define "my-webapp.env" -}}
{{- if .Values.production }}
production
{{- else }}
development
{{- end }}
{{- end }}
```

Используй в deployment.yaml:
```yaml
metadata:
  labels:
    environment: {{ include "my-webapp.env" . }}
```

**3. Добавь NOTES.txt для инструкций после установки:**
```txt
1. Get the application URL by running these commands:
{{- if .Values.ingress.enabled }}
  http{{ if .Values.ingress.tls }}s{{ end }}://{{ .Values.ingress.host }}
{{- else if contains "NodePort" .Values.service.type }}
  export NODE_PORT=$(kubectl get --namespace {{ .Release.Namespace }} -o jsonpath="{.spec.ports[0].nodePort}" services {{ include "my-webapp.fullname" . }})
  export NODE_IP=$(kubectl get nodes --namespace {{ .Release.Namespace }} -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
{{- else if contains "ClusterIP" .Values.service.type }}
  export POD_NAME=$(kubectl get pods --namespace {{ .Release.Namespace }} -l "app.kubernetes.io/name={{ include "my-webapp.name" . }}" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace {{ .Release.Namespace }} port-forward $POD_NAME 8080:80
  echo "Visit http://127.0.0.1:8080"
{{- end }}
```

---

## Модуль 2: Values и переопределение конфигурации (25 минут)

### 🎯 Напоминалка

**Приоритет values (от низшего к высшему):**
```
1. values.yaml в chart (по умолчанию)
2. values.yaml из parent chart
3. values.yaml переданный через -f
4. Значения из --set
```

**Способы передачи values:**
```bash
# Через файл
helm install my-release ./chart -f custom-values.yaml
helm install my-release ./chart -f prod-values.yaml -f secrets-values.yaml

# Через --set
helm install my-release ./chart --set replicaCount=3
helm install my-release ./chart --set image.tag=v2.0.0
helm install my-release ./chart --set-string nodeSelector."kubernetes\.io/role"=master

# Через --set-file (содержимое файла как значение)
helm install my-release ./chart --set-file config=config.yaml

# Через --set-json
helm install my-release ./chart --set-json 'annotations={"key1":"value1","key2":"value2"}'
```

**Работа с вложенными values:**
```yaml
# values.yaml
database:
  host: localhost
  port: 5432
  credentials:
    username: admin
    password: secret

# В templates
{{ .Values.database.host }}
{{ .Values.database.credentials.username }}

# Через --set
--set database.host=prod-db.example.com
--set database.credentials.password=newsecret
```

**Массивы в values:**
```yaml
# values.yaml
hosts:
  - host: example1.com
    paths: ["/api", "/web"]
  - host: example2.com
    paths: ["/admin"]

# В templates
{{- range .Values.hosts }}
- host: {{ .host }}
  paths:
  {{- range .paths }}
  - {{ . }}
  {{- end }}
{{- end }}

# Через --set (сложнее)
--set hosts[0].host=example1.com
--set hosts[0].paths[0]=/api
```

**Условная конфигурация:**
```yaml
# values.yaml
environment: production

features:
  monitoring: true
  backup: true
  debug: false

# В templates
{{- if eq .Values.environment "production" }}
replicas: 3
{{- else }}
replicas: 1
{{- end }}

{{- if .Values.features.monitoring }}
# Добавить sidecar для мониторинга
{{- end }}
```

**Валидация values:**
```yaml
# templates/_validation.tpl
{{- if not .Values.image.repository }}
  {{- fail "image.repository is required" }}
{{- end }}

{{- if and .Values.ingress.enabled (not .Values.ingress.host) }}
  {{- fail "ingress.host is required when ingress is enabled" }}
{{- end }}
```

**Global values:**
```yaml
# values.yaml
global:
  imageRegistry: docker.io
  storageClass: fast-ssd

# В любом chart можно использовать
{{ .Values.global.imageRegistry }}
```

**Переопределение для разных окружений:**
```yaml
# values.yaml (base)
replicaCount: 1
environment: dev

# values-prod.yaml
replicaCount: 3
environment: production
resources:
  limits:
    cpu: 500m
    memory: 512Mi

# values-staging.yaml
replicaCount: 2
environment: staging

# Использование
helm install my-app ./chart -f values-prod.yaml
helm install my-app ./chart -f values-staging.yaml
```

### 💻 Задание

Настрой multi-environment конфигурацию:

1. **Создай базовый values.yaml:**
```yaml
# values.yaml
environment: development

replicaCount: 1

image:
  repository: nginx
  tag: "alpine"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 50m
    memory: 64Mi

ingress:
  enabled: false
  className: nginx
  host: ""
  tls:
    enabled: false

monitoring:
  enabled: false
  
database:
  enabled: false
  host: ""
  port: 5432
```

2. **Создай values-dev.yaml:**
```yaml
# values-dev.yaml
environment: development

replicaCount: 1

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 50m
    memory: 64Mi

ingress:
  enabled: true
  host: dev.myapp.local

monitoring:
  enabled: false

database:
  enabled: true
  host: postgres-dev
  port: 5432
```

3. **Создай values-staging.yaml:**
```yaml
# values-staging.yaml
environment: staging

replicaCount: 2

image:
  tag: "1.21-alpine"

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi

ingress:
  enabled: true
  host: staging.myapp.com
  tls:
    enabled: true

monitoring:
  enabled: true

database:
  enabled: true
  host: postgres-staging.db.local
  port: 5432
```

4. **Создай values-prod.yaml:**
```yaml
# values-prod.yaml
environment: production

replicaCount: 3

image:
  tag: "1.21-alpine"
  pullPolicy: Always

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

ingress:
  enabled: true
  host: myapp.com
  tls:
    enabled: true

monitoring:
  enabled: true

database:
  enabled: true
  host: postgres-prod.db.local
  port: 5432

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPU: 70
```

5. **Обнови deployment.yaml для использования окружения:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-webapp.fullname" . }}
  labels:
    {{- include "my-webapp.labels" . | nindent 4 }}
    environment: {{ .Values.environment }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-webapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-webapp.selectorLabels" . | nindent 8 }}
        environment: {{ .Values.environment }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        env:
        - name: ENVIRONMENT
          value: {{ .Values.environment }}
        {{- if .Values.database.enabled }}
        - name: DB_HOST
          value: {{ .Values.database.host }}
        - name: DB_PORT
          value: {{ .Values.database.port | quote }}
        {{- end }}
        ports:
        - name: http
          containerPort: 80
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
```

6. **Тестируй разные окружения:**
```bash
# Dev
helm template my-webapp . -f values-dev.yaml > dev-manifests.yaml
cat dev-manifests.yaml | grep -A 5 "replicas:"

# Staging
helm template my-webapp . -f values-staging.yaml > staging-manifests.yaml
cat staging-manifests.yaml | grep -A 5 "replicas:"

# Production
helm template my-webapp . -f values-prod.yaml > prod-manifests.yaml
cat prod-manifests.yaml | grep -A 5 "replicas:"
```

7. **Установи в разные namespaces:**
```bash
# Dev
helm install my-webapp-dev . -f values-dev.yaml -n dev --create-namespace

# Staging
helm install my-webapp-staging . -f values-staging.yaml -n staging --create-namespace

# Production
helm install my-webapp-prod . -f values-prod.yaml -n production --create-namespace
```

8. **Проверь различия:**
```bash
kubectl get deploy -n dev -o yaml | grep -A 3 "replicas:"
kubectl get deploy -n staging -o yaml | grep -A 3 "replicas:"
kubectl get deploy -n production -o yaml | grep -A 3 "replicas:"
```

### 🚀 Бонус (новое)

**1. Используй JSON Schema для валидации values:**

Создай `values.schema.json`:
```json
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["replicaCount", "image"],
  "properties": {
    "replicaCount": {
      "type": "integer",
      "minimum": 1,
      "maximum": 100
    },
    "image": {
      "type": "object",
      "required": ["repository", "tag"],
      "properties": {
        "repository": {
          "type": "string"
        },
        "tag": {
          "type": "string"
        }
      }
    },
    "environment": {
      "type": "string",
      "enum": ["development", "staging", "production"]
    }
  }
}
```

Helm автоматически валидирует при установке.

**2. Используй --set с вычислениями:**
```bash
# Динамическое вычисление replicas
REPLICAS=$(($(kubectl get nodes --no-headers | wc -l) * 2))
helm upgrade my-webapp . --set replicaCount=$REPLICAS

# Использование переменных окружения
helm install my-webapp . \
  --set image.tag=${GIT_COMMIT:0:7} \
  --set environment=${ENV:-development}
```

**3. Создай values патчи для быстрых изменений:**
```bash
# patch-high-traffic.yaml
replicaCount: 10
resources:
  limits:
    cpu: 1000m
    memory: 1Gi

# Применить патч
helm upgrade my-webapp . \
  -f values-prod.yaml \
  -f patch-high-traffic.yaml
```

---

## Модуль 3: Зависимости и Sub-charts (30 минут)

### 🎯 Напоминалка

**Зависимости в Chart.yaml:**
```yaml
# Chart.yaml
apiVersion: v2
name: my-app
version: 1.0.0
dependencies:
  - name: postgresql
    version: "12.1.9"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled      # Условная установка
    tags:
      - database
  
  - name: redis
    version: "17.3.14"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
    tags:
      - cache
  
  - name: common
    version: "2.2.0"
    repository: "https://charts.bitnami.com/bitnami"
    tags:
      - bitnami-common
```

**Управление зависимостями:**
```bash
# Скачать зависимости
helm dependency update ./my-chart
helm dependency build ./my-chart

# Посмотреть зависимости
helm dependency list ./my-chart

# Структура после загрузки
my-chart/
├── charts/
│   ├── postgresql-12.1.9.tgz
│   └── redis-17.3.14.tgz
├── Chart.lock
└── Chart.yaml
```

**Chart.lock - фиксация версий:**
```yaml
# Chart.lock (генерируется автоматически)
dependencies:
- name: postgresql
  repository: https://charts.bitnami.com/bitnami
  version: 12.1.9
- name: redis
  repository: https://charts.bitnami.com/bitnami
  version: 17.3.14
digest: sha256:abc123...
generated: "2025-01-15T10:30:00Z"
```

**Переопределение values sub-charts:**
```yaml
# values.yaml родительского chart
# Конфигурация sub-chart через его имя
postgresql:
  enabled: true
  auth:
    username: myapp
    password: secret123
    database: myappdb
  primary:
    persistence:
      enabled: true
      size: 10Gi

redis:
  enabled: true
  auth:
    enabled: false
  master:
    persistence:
      enabled: false

# Глобальные values для всех sub-charts
global:
  storageClass: "fast-ssd"
  imageRegistry: "my-registry.com"
```

**Условная установка зависимостей:**
```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    condition: postgresql.enabled
    tags:
      - database
  - name: mysql
    condition: mysql.enabled
    tags:
      - database

# values.yaml
tags:
  database: true  # Установить все charts с тегом database

postgresql:
  enabled: true   # Установить postgresql

mysql:
  enabled: false  # Не устанавливать mysql
```

**Алиасы для зависимостей:**
```yaml
# Chart.yaml - использовать один chart несколько раз
dependencies:
  - name: postgresql
    version: "12.1.9"
    repository: "https://charts.bitnami.com/bitnami"
    alias: postgresql-primary
  
  - name: postgresql
    version: "12.1.9"
    repository: "https://charts.bitnami.com/bitnami"
    alias: postgresql-replica

# values.yaml
postgresql-primary:
  enabled: true
  replication:
    enabled: false

postgresql-replica:
  enabled: true
  replication:
    enabled: true
    readReplicas: 2
```

**Import values из sub-chart:**
```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    version: "12.1.9"
    repository: "https://charts.bitnami.com/bitnami"
    import-values:
      - child: auth.username
        parent: db.user
      - child: auth.password
        parent: db.password

# Теперь в родительском chart
# .Values.db.user и .Values.db.password
# берутся из postgresql.auth.username и postgresql.auth.password
```

**Локальные sub-charts:**
```
my-chart/
├── charts/
│   └── my-subchart/          # Локальный sub-chart
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
├── Chart.yaml
└── values.yaml
```

**Доступ к values родительского chart из sub-chart:**
```yaml
# В sub-chart можно использовать
{{ .Values.global.key }}      # Глобальные values
# Но НЕ напрямую {{ .Values.parentKey }}
```

### 💻 Задание

Создай полноценное приложение с зависимостями:

1. **Создай новый chart для web-приложения:**
```bash
helm create fullstack-app
cd fullstack-app
```

2. **Обнови Chart.yaml с зависимостями:**
```yaml
apiVersion: v2
name: fullstack-app
description: Full-stack application with database and cache
type: application
version: 1.0.0
appVersion: "1.0.0"

dependencies:
  - name: postgresql
    version: "12.1.9"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
    tags:
      - database
  
  - name: redis
    version: "17.3.14"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
    tags:
      - cache

  - name: mongodb
    version: "13.6.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: mongodb.enabled
    tags:
      - database
```

3. **Настрой values.yaml:**
```yaml
# Конфигурация главного приложения
replicaCount: 2

image:
  repository: nginx
  tag: "alpine"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

# Включить/выключить зависимости
tags:
  database: true
  cache: true

# Конфигурация PostgreSQL
postgresql:
  enabled: true
  auth:
    username: webapp
    password: webapppass123
    database: webappdb
  primary:
    persistence:
      enabled: true
      size: 5Gi
    resources:
      limits:
        cpu: 500m
        memory: 512Mi
      requests:
        cpu: 250m
        memory: 256Mi

# Конфигурация Redis
redis:
  enabled: true
  auth:
    enabled: true
    password: redispass123
  master:
    persistence:
      enabled: false
    resources:
      limits:
        cpu: 200m
        memory: 256Mi
      requests:
        cpu: 100m
        memory: 128Mi

# Конфигурация MongoDB (выключена по умолчанию)
mongodb:
  enabled: false
  auth:
    enabled: true
    rootPassword: mongopass123
    username: mongouser
    password: mongopass
    database: mongodb

# Глобальные настройки
global:
  storageClass: ""
  imageRegistry: ""
```

4. **Обнови deployment.yaml для подключения к зависимостям:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "fullstack-app.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "fullstack-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "fullstack-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        env:
        {{- if .Values.postgresql.enabled }}
        - name: DATABASE_URL
          value: "postgresql://{{ .Values.postgresql.auth.username }}:{{ .Values.postgresql.auth.password }}@{{ include "fullstack-app.fullname" . }}-postgresql:5432/{{ .Values.postgresql.auth.database }}"
        {{- end }}
        {{- if .Values.redis.enabled }}
        - name: REDIS_URL
          value: "redis://:{{ .Values.redis.auth.password }}@{{ include "fullstack-app.fullname" . }}-redis-master:6379"
        {{- end }}
        {{- if .Values.mongodb.enabled }}
        - name: MONGODB_URL
          value: "mongodb://{{ .Values.mongodb.auth.username }}:{{ .Values.mongodb.auth.password }}@{{ include "fullstack-app.fullname" . }}-mongodb:27017/{{ .Values.mongodb.auth.database }}"
        {{- end }}
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
```

5. **Скачай зависимости:**
```bash
helm dependency update .
helm dependency list .

# Проверь что появилось
ls -la charts/
cat Chart.lock
```

6. **Протестируй с разными конфигурациями:**
```bash
# Все зависимости включены
helm template fullstack-app . > all-deps.yaml
cat all-deps.yaml | grep -E "kind:|name:" | head -20

# Только PostgreSQL
helm template fullstack-app . \
  --set redis.enabled=false \
  --set mongodb.enabled=false > only-postgres.yaml

# Только Redis
helm template fullstack-app . \
  --set postgresql.enabled=false \
  --set mongodb.enabled=false > only-redis.yaml

# Без баз данных
helm template fullstack-app . \
  --set tags.database=false \
  --set tags.cache=false > no-deps.yaml
```

7. **Установи приложение:**
```bash
# Создай namespace
kubectl create namespace fullstack

# Установи с зависимостями
helm install fullstack-app . -n fullstack

# Проверь что все установилось
helm list -n fullstack
kubectl get all -n fullstack
kubectl get pvc -n fullstack
kubectl get secrets -n fullstack
```

8. **Проверь подключения:**
```bash
# Получи пароли из secrets
export POSTGRES_PASSWORD=$(kubectl get secret --namespace fullstack fullstack-app-postgresql -o jsonpath="{.data.password}" | base64 -d)
export REDIS_PASSWORD=$(kubectl get secret --namespace fullstack fullstack-app-redis -o jsonpath="{.data.redis-password}" | base64 -d)

echo "PostgreSQL password: $POSTGRES_PASSWORD"
echo "Redis password: $REDIS_PASSWORD"

# Проверь подключение к PostgreSQL
kubectl run postgresql-client --rm --tty -i --restart='Never' \
  --namespace fullstack \
  --image docker.io/bitnami/postgresql:15 \
  --env="PGPASSWORD=$POSTGRES_PASSWORD" \
  --command -- psql --host fullstack-app-postgresql -U webapp -d webappdb -p 5432

# Проверь подключение к Redis
kubectl run redis-client --rm --tty -i --restart='Never' \
  --namespace fullstack \
  --image docker.io/bitnami/redis:7.0 \
  --command -- redis-cli -h fullstack-app-redis-master -a $REDIS_PASSWORD
```

9. **Обнови конфигурацию зависимости:**
```bash
# Увеличь размер PostgreSQL storage
helm upgrade fullstack-app . -n fullstack \
  --set postgresql.primary.persistence.size=10Gi

# Проверь изменения
kubectl get pvc -n fullstack
```

10. **Очистка:**
```bash
helm uninstall fullstack-app -n fullstack
kubectl delete namespace fullstack
```

### 🚀 Бонус (новое)

**1. Создай локальный sub-chart:**

```bash
# Создай структуру
mkdir -p charts/backend
cd charts/backend
helm create .
cd ../..

# Теперь можно использовать локальный sub-chart
# Он не указывается в dependencies Chart.yaml
# Просто лежит в charts/ директории
```

**2. Используй repository в виде OCI registry:**
```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    version: "12.1.9"
    repository: "oci://registry-1.docker.io/bitnamicharts"
```

**3. Создай umbrella chart:**

```yaml
# umbrella-chart/Chart.yaml
apiVersion: v2
name: microservices
description: Umbrella chart for microservices architecture
type: application
version: 1.0.0

dependencies:
  - name: frontend
    version: "1.0.0"
    repository: "file://../frontend-chart"
  
  - name: backend-api
    version: "1.0.0"
    repository: "file://../backend-chart"
  
  - name: worker
    version: "1.0.0"
    repository: "file://../worker-chart"
  
  - name: postgresql
    version: "12.1.9"
    repository: "https://charts.bitnami.com/bitnami"
  
  - name: redis
    version: "17.3.14"
    repository: "https://charts.bitnami.com/bitnami"
```

---

## Модуль 4: Hooks и Lifecycle Management (25 минут)

### 🎯 Напоминалка

**Helm Hooks - выполнение действий в определенные моменты:**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "my-chart.fullname" . }}-migration
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      containers:
      - name: migration
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        command: ['sh', '-c', 'echo Running database migration']
      restartPolicy: Never
  backoffLimit: 3
```

**Типы hooks:**
```yaml
"helm.sh/hook": pre-install      # До установки
"helm.sh/hook": post-install     # После установки
"helm.sh/hook": pre-delete       # До удаления
"helm.sh/hook": post-delete      # После удаления
"helm.sh/hook": pre-upgrade      # До обновления
"helm.sh/hook": post-upgrade     # После обновления
"helm.sh/hook": pre-rollback     # До отката
"helm.sh/hook": post-rollback    # После отката
"helm.sh/hook": test             # Тестовый hook
```

**Hook weight (порядок выполнения):**
```yaml
# Hooks выполняются в порядке от меньшего к большему
"helm.sh/hook-weight": "-5"      # Выполнится первым
"helm.sh/hook-weight": "0"       # По умолчанию
"helm.sh/hook-weight": "5"       # Выполнится последним
```

**Hook delete policies:**
```yaml
"helm.sh/hook-delete-policy": before-hook-creation  # Удалить перед созданием нового
"helm.sh/hook-delete-policy": hook-succeeded        # Удалить после успеха
"helm.sh/hook-delete-policy": hook-failed           # Удалить после неудачи
```

**Комбинирование policies:**
```yaml
"helm.sh/hook-delete-policy": hook-succeeded,hook-failed
```

**Resource Policy - защита от удаления:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: important-data
  annotations:
    "helm.sh/resource-policy": keep  # Не удалять при helm uninstall
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

**Test hooks:**
```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "my-chart.fullname" . }}-test-connection"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args: ['{{ include "my-chart.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

**Запуск тестов:**
```bash
helm test my-release
helm test my-release --logs
```

**Примеры использования hooks:**

1. **Database Migration (pre-upgrade):**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-db-migration"
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "-1"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      containers:
      - name: migration
        image: migrate/migrate
        command:
        - migrate
        - -path=/migrations
        - -database=$(DATABASE_URL)
        - up
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
      restartPolicy: Never
```

2. **Database Backup (pre-upgrade):**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-db-backup"
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "-2"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: backup
        image: postgres:14
        command:
        - pg_dump
        - -h
        - postgres-service
        - -U
        - postgres
        - mydb
        - -f
        - /backup/db-{{ .Release.Revision }}.sql
        volumeMounts:
        - name: backup
          mountPath: /backup
      volumes:
      - name: backup
        persistentVolumeClaim:
          claimName: backup-pvc
      restartPolicy: Never
```

3. **Cleanup Job (pre-delete):**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-cleanup"
  annotations:
    "helm.sh/hook": pre-delete
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: cleanup
        image: bitnami/kubectl
        command:
        - kubectl
        - delete
        - pvc
        - --all
        - -n
        - {{ .Release.Namespace }}
      restartPolicy: Never
```

### 💻 Задание

Создай систему с lifecycle hooks:

1. **Создай chart с hooks:**
```bash
helm create app-with-hooks
cd app-with-hooks
```

2. **Создай pre-install hook для проверки:**
```yaml
# templates/hooks/pre-install-check.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "app-with-hooks.fullname" . }}-pre-install-check"
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  containers:
  - name: pre-install-check
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      echo "Running pre-installation checks..."
      echo "Checking cluster resources..."
      echo "✓ Cluster is ready"
      echo "✓ Namespace is accessible"
      echo "Pre-install checks completed successfully!"
  restartPolicy: Never
```

3. **Создай post-install hook для инициализации:**
```yaml
# templates/hooks/post-install-init.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ include "app-with-hooks.fullname" . }}-post-install"
  annotations:
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      name: "{{ include "app-with-hooks.fullname" . }}-post-install"
    spec:
      containers:
      - name: post-install-job
        image: busybox
        command: ['sh', '-c']
        args:
        - |
          echo "Running post-installation tasks..."
          echo "Initializing application data..."
          sleep 5
          echo "Creating default configuration..."
          echo "✓ Application initialized successfully!"
      restartPolicy: Never
  backoffLimit: 3
```

4. **Создай pre-upgrade hook для backup:**
```yaml
# templates/hooks/pre-upgrade-backup.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ include "app-with-hooks.fullname" . }}-backup-{{ .Release.Revision }}"
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "-10"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: backup
        image: busybox
        command: ['sh', '-c']
        args:
        - |
          echo "Creating backup before upgrade..."
          echo "Backup revision: {{ .Release.Revision }}"
          echo "Backup timestamp: $(date)"
          mkdir -p /backup
          echo "Backup data" > /backup/backup-{{ .Release.Revision }}.txt
          echo "✓ Backup completed successfully!"
          sleep 3
        volumeMounts:
        - name: backup
          mountPath: /backup
      volumes:
      - name: backup
        emptyDir: {}
      restartPolicy: Never
```

5. **Создай post-upgrade hook для миграции:**
```yaml
# templates/hooks/post-upgrade-migration.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ include "app-with-hooks.fullname" . }}-migration-{{ .Release.Revision }}"
  annotations:
    "helm.sh/hook": post-upgrade
    "helm.sh/hook-weight": "5"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      containers:
      - name: migration
        image: busybox
        command: ['sh', '-c']
        args:
        - |
          echo "Running database migration..."
          echo "Migration revision: {{ .Release.Revision }}"
          echo "Applying schema changes..."
          sleep 5
          echo "✓ Migration completed successfully!"
      restartPolicy: Never
  backoffLimit: 3
```

6. **Создай test hook:**
```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "app-with-hooks.fullname" . }}-test-connection"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args:
    - '--spider'
    - '--timeout=5'
    - 'http://{{ include "app-with-hooks.fullname" . }}:{{ .Values.service.port }}'
  restartPolicy: Never
```

7. **Создай pre-delete cleanup hook:**
```yaml
# templates/hooks/pre-delete-cleanup.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ include "app-with-hooks.fullname" . }}-cleanup"
  annotations:
    "helm.sh/hook": pre-delete
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: cleanup
        image: busybox
        command: ['sh', '-c']
        args:
        - |
          echo "Running cleanup before deletion..."
          echo "Removing temporary files..."
          echo "Closing connections..."
          echo "✓ Cleanup completed successfully!"
      restartPolicy: Never
```

8. **Создай PVC с resource-policy keep:**
```yaml
# templates/persistent-data.yaml
{{- if .Values.persistence.enabled }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "app-with-hooks.fullname" . }}-data
  annotations:
    "helm.sh/resource-policy": keep
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: {{ .Values.persistence.size }}
{{- end }}
```

9. **Обнови values.yaml:**
```yaml
persistence:
  enabled: true
  size: 1Gi
```

10. **Тестируй hooks:**
```bash
# Установка (pre-install + post-install hooks)
helm install my-app . -n test --create-namespace
kubectl get jobs -n test
kubectl get pods -n test
kubectl logs -n test -l job-name=$(kubectl get jobs -n test -o name | head -1 | cut -d'/' -f2)

# Проверь что приложение работает
kubectl get all -n test

# Запусти тесты
helm test my-app -n test --logs

# Обновление (pre-upgrade + post-upgrade hooks)
helm upgrade my-app . -n test --set replicaCount=3
kubectl get jobs -n test
kubectl logs -n test -l job-name=$(kubectl get jobs -n test -o jsonpath='{.items[-1].metadata.name}')

# Проверь историю
helm history my-app -n test

# Удаление (pre-delete hook)
helm uninstall my-app -n test

# Проверь что PVC остался (resource-policy: keep)
kubectl get pvc -n test
```

### 🚀 Бонус (новое)

**1. Создай условный hook:**
```yaml
{{- if .Values.hooks.migration.enabled }}
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ include "app-with-hooks.fullname" . }}-migration"
  annotations:
    "helm.sh/hook": post-upgrade
    "helm.sh/hook-weight": "{{ .Values.hooks.migration.weight }}"
spec:
  template:
    spec:
      containers:
      - name: migration
        image: "{{ .Values.hooks.migration.image }}"
        command: {{ .Values.hooks.migration.command }}
      restartPolicy: Never
{{- end }}
```

**2. Используй hook для wait условий:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-wait-for-db"
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-1"
spec:
  template:
    spec:
      containers:
      - name: wait
        image: busybox
        command:
        - sh
        - -c
        - |
          until nc -z postgres-service 5432; do
            echo "Waiting for database..."
            sleep 2
          done
          echo "Database is ready!"
      restartPolicy: Never
```

**3. Создай rollback hook для восстановления:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-rollback-restore"
  annotations:
    "helm.sh/hook": post-rollback
    "helm.sh/hook-weight": "0"
spec:
  template:
    spec:
      containers:
      - name: restore
        image: postgres:14
        command:
        - sh
        - -c
        - |
          echo "Restoring from backup after rollback..."
          # Restore logic here
      restartPolicy: Never
```

---

## Модуль 5: Helm Plugins и расширения (20 минут)

### 🎯 Напоминалка

**Helm Plugins - расширение функциональности:**

```bash
# Управление плагинами
helm plugin list
helm plugin install <plugin-url>
helm plugin update <plugin-name>
helm plugin uninstall <plugin-name>
```

**Популярные плагины:**

**1. helm-diff - просмотр изменений:**
```bash
# Установка
helm plugin install https://github.com/databus23/helm-diff

# Использование
helm diff upgrade my-release ./my-chart
helm diff upgrade my-release ./my-chart -f new-values.yaml
helm diff upgrade my-release ./my-chart --set replicaCount=5
```

**2. helm-secrets - работа с зашифрованными secrets:**
```bash
# Установка (требует sops)
helm plugin install https://github.com/jkroepke/helm-secrets

# Шифрование values
helm secrets encrypt secrets.yaml

# Использование
helm secrets install my-release ./chart -f secrets.yaml
helm secrets upgrade my-release ./chart -f secrets.yaml.dec
```

**3. helm-git - установка charts из Git:**
```bash
# Установка
helm plugin install https://github.com/aslafy-z/helm-git

# Использование
helm install my-release git+https://github.com/user/repo@charts/my-chart?ref=main
```

**4. helm-push - публикация в ChartMuseum:**
```bash
# Установка
helm plugin install https://github.com/chartmuseum/helm-push

# Использование
helm push ./my-chart chartmuseum
helm cm-push ./my-chart chartmuseum
```

**5. helm-unittest - unit тесты для charts:**
```bash
# Установка
helm plugin install https://github.com/helm-unittest/helm-unittest

# Создание теста
# tests/deployment_test.yaml
suite: test deployment
templates:
  - deployment.yaml
tests:
  - it: should create deployment
    asserts:
      - isKind:
          of: Deployment
      - equal:
          path: spec.replicas
          value: 3

# Запуск
helm unittest ./my-chart
```

**6. helm-dashboard - UI для Helm:**
```bash
# Установка
helm plugin install https://github.com/komodorio/helm-dashboard

# Запуск
helm dashboard
```

**7. helm-docs - генерация документации:**
```bash
# Установка
helm plugin install https://github.com/norwoodj/helm-docs

# Генерация README.md
helm-docs
```

**Создание собственного плагина:**

```bash
# Структура плагина
my-plugin/
├── plugin.yaml
└── my-plugin.sh

# plugin.yaml
name: "my-plugin"
version: "0.1.0"
usage: "My custom Helm plugin"
description: "Does something awesome"
command: "$HELM_PLUGIN_DIR/my-plugin.sh"

# my-plugin.sh
#!/bin/bash
echo "Hello from my plugin!"
echo "Release: $1"

# Установка
helm plugin install ./my-plugin
helm my-plugin my-release
```

### 💻 Задание

Настрой и используй helm plugins:

**1. Установи helm-diff:**
```bash
helm plugin install https://github.com/databus23/helm-diff
helm plugin list
```

**2. Создай тестовый chart:**
```bash
helm create diff-demo
cd diff-demo
```

**3. Установи начальную версию:**
```bash
helm install diff-demo . -n test --create-namespace
```

**4. Используй diff для просмотра изменений:**
```bash
# Изменение replicas
helm diff upgrade diff-demo . --set replicaCount=5 -n test

# Изменение image
helm diff upgrade diff-demo . --set image.tag=1.22 -n test

# Несколько изменений
helm diff upgrade diff-demo . \
  --set replicaCount=5 \
  --set image.tag=1.22 \
  --set service.type=LoadBalancer \
  -n test
```

**5. Примени изменения после проверки:**
```bash
helm upgrade diff-demo . --set replicaCount=5 -n test
```

**6. Установи helm-unittest:**
```bash
helm plugin install https://github.com/helm-unittest/helm-unittest
```

**7. Создай unit тесты:**
```bash
mkdir -p tests

# tests/deployment_test.yaml
cat > tests/deployment_test.yaml << 'EOF'
suite: test deployment
templates:
  - deployment.yaml
tests:
  - it: should create a Deployment
    asserts:
      - isKind:
          of: Deployment
      - equal:
          path: metadata.name
          value: RELEASE-NAME-diff-demo

  - it: should use correct image
    set:
      image.repository: nginx
      image.tag: "alpine"
    asserts:
      - equal:
          path: spec.template.spec.containers[0].image
          value: nginx:alpine

  - it: should have correct replica count
    set:
      replicaCount: 3
    asserts:
      - equal:
          path: spec.replicas
          value: 3

  - it: should set resources when specified
    set:
      resources:
        limits:
          cpu: 200m
          memory: 256Mi
    asserts:
      - equal:
          path: spec.template.spec.containers[0].resources.limits.cpu
          value: 200m
EOF

# tests/service_test.yaml
cat > tests/service_test.yaml << 'EOF'
suite: test service
templates:
  - service.yaml
tests:
  - it: should create a Service
    asserts:
      - isKind:
          of: Service
      - equal:
          path: spec.type
          value: ClusterIP

  - it: should use NodePort when configured
    set:
      service.type: NodePort
      service.nodePort: 30080
    asserts:
      - equal:
          path: spec.type
          value: NodePort
EOF
```

**8. Запусти тесты:**
```bash
helm unittest .
helm unittest . -f 'tests/*.yaml'
helm unittest . -o junit.xml  # JUnit формат
```

**9. Установи helm-docs:**
```bash
# macOS
brew install norwoodj/tap/helm-docs

# Linux
GO111MODULE=on go get github.com/norwoodj/helm-docs/cmd/helm-docs

# Или через docker
docker run --rm -v "$(pwd):/helm-docs" jnorwood/helm-docs:latest
```

**10. Добавь комментарии в values.yaml:**
```yaml
# -- Number of replicas
replicaCount: 2

image:
  # -- Image repository
  repository: nginx
  # -- Image pull policy
  pullPolicy: IfNotPresent
  # -- Image tag (defaults to chart appVersion)
  tag: ""

service:
  # -- Service type
  type: ClusterIP
  # -- Service port
  port: 80
```

**11. Генерируй документацию:**
```bash
helm-docs
cat README.md
```

**12. Очистка:**
```bash
helm uninstall diff-demo -n test
```

### 🚀 Бонус (новое)

**1. Создай свой простой плагин:**
```bash
mkdir -p ~/.local/share/helm/plugins/helm-info
cd ~/.local/share/helm/plugins/helm-info

# plugin.yaml
cat > plugin.yaml << 'EOF'
name: "info"
version: "1.0.0"
usage: "Show detailed release info"
description: "Display comprehensive information about a Helm release"
command: "$HELM_PLUGIN_DIR/info.sh"
EOF

# info.sh
cat > info.sh << 'EOF'
#!/bin/bash
RELEASE=$1
NAMESPACE=${2:-default}

echo "===== Release Info ====="
helm status $RELEASE -n $NAMESPACE

echo ""
echo "===== Values ====="
helm get values $RELEASE -n $NAMESPACE

echo ""
echo "===== History ====="
helm history $RELEASE -n $NAMESPACE

echo ""
echo "===== Resources ====="
kubectl get all -n $NAMESPACE -l app.kubernetes.io/instance=$RELEASE
EOF

chmod +x info.sh

# Использование
helm info my-release test
```

**2. Используй helm-schema-gen для генерации JSON Schema:**
```bash
# Установка
helm plugin install https://github.com/karuppiah7890/helm-schema-gen

# Генерация
helm schema-gen values.yaml > values.schema.json
```

**3. Настрой helm-secrets с SOPS:**
```bash
# Установка sops
brew install sops  # macOS
# или скачай с https://github.com/mozilla/sops/releases

# Установка helm-secrets
helm plugin install https://github.com/jkroepke/helm-secrets

# Создай secrets.yaml
cat > secrets.yaml << EOF
database:
  password: supersecret123
  username: admin
api:
  key: api-key-12345
EOF

# Зашифруй (требует GPG или cloud KMS)
helm secrets encrypt secrets.yaml

# Используй в установке
helm secrets install my-app ./chart -f secrets.yaml
```

---

## Модуль 6: Helm Repositories и публикация (25 минут)

### 🎯 Напоминалка

**Helm Repositories - хранилища charts:**

**Типы репозиториев:**
1. HTTP/HTTPS репозиторий (классический)
2. OCI Registry (Docker-подобный)
3. Git репозиторий
4. ChartMuseum (специализированный)

**Работа с репозиториями:**
```bash
# Добавление
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add my-repo https://my-charts.example.com

# Обновление
helm repo update
helm repo update stable

# Поиск
helm search repo nginx
helm search repo nginx --versions  # Все версии
helm search hub wordpress          # Поиск в Artifact Hub

# Список
helm repo list

# Удаление
helm repo remove stable
```

**Упаковка chart:**
```bash
# Создание .tgz архива
helm package ./my-chart
helm package ./my-chart --version 1.2.3
helm package ./my-chart --app-version 2.0.0
helm package ./my-chart --destination ./packages/

# Результат: my-chart-1.0.0.tgz
```

**Создание index.yaml:**
```bash
# Генерация индекса репозитория
helm repo index .
helm repo index . --url https://my-charts.example.com

# Обновление существующего индекса
helm repo index . --merge existing-index.yaml
```

**Структура index.yaml:**
```yaml
apiVersion: v1
entries:
  my-chart:
  - apiVersion: v2
    appVersion: 1.0.0
    created: "2025-01-15T10:00:00Z"
    description: My awesome chart
    digest: abc123...
    name: my-chart
    urls:
    - https://my-charts.example.com/my-chart-1.0.0.tgz
    version: 1.0.0
  - apiVersion: v2
    appVersion: 0.9.0
    created: "2025-01-10T10:00:00Z"
    description: My awesome chart
    digest: def456...
    name: my-chart
    urls:
    - https://my-charts.example.com/my-chart-0.9.0.tgz
    version: 0.9.0
generated: "2025-01-15T10:00:00Z"
```

**OCI Registry (Helm 3.8+):**
```bash
# Логин в registry
helm registry login registry-1.docker.io -u username

# Сохранение chart в OCI
helm push my-chart-1.0.0.tgz oci://registry-1.docker.io/myuser

# Установка из OCI
helm install my-release oci://registry-1.docker.io/myuser/my-chart --version 1.0.0

# Pull chart
helm pull oci://registry-1.docker.io/myuser/my-chart --version 1.0.0
```

**ChartMuseum - self-hosted репозиторий:**
```bash
# Запуск ChartMuseum в K8s
helm repo add chartmuseum https://chartmuseum.github.io/charts
helm install chartmuseum chartmuseum/chartmuseum \
  --set env.open.DISABLE_API=false \
  --set persistence.enabled=true

# Добавление в Helm
helm repo add my-museum http://chartmuseum.example.com

# Публикация (с helm-push plugin)
helm plugin install https://github.com/chartmuseum/helm-push
helm cm-push ./my-chart my-museum
```

**GitHub Pages как репозиторий:**
```bash
# 1. Создай gh-pages ветку
git checkout -b gh-pages

# 2. Упакуй charts
helm package ./charts/*

# 3. Создай index
helm repo index . --url https://username.github.io/repo-name

# 4. Commit и push
git add .
git commit -m "Publish charts"
git push origin gh-pages

# 5. Использование
helm repo add my-repo https://username.github.io/repo-name
helm repo update
helm install my-release my-repo/my-chart
```

**Artifact Hub - публичный registry:**
```yaml
# artifacthub-repo.yml
repositoryID: <uuid>
owners:
  - name: your-name
    email: your-email@example.com
```

### 💻 Задание

Создай и опубликуй свой Helm repository:

**1. Создай несколько charts:**
```bash
mkdir helm-repo
cd helm-repo
mkdir charts

# Chart 1
helm create charts/webapp
# Chart 2
helm create charts/api-service
# Chart 3
helm create charts/worker
```

**2. Настрой Chart.yaml для каждого:**
```bash
# webapp/Chart.yaml
cat > charts/webapp/Chart.yaml << EOF
apiVersion: v2
name: webapp
description: Web application frontend
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - web
  - frontend
maintainers:
  - name: Your Name
    email: you@example.com
EOF

# api-service/Chart.yaml
cat > charts/api-service/Chart.yaml << EOF
apiVersion: v2
name: api-service
description: REST API service
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - api
  - backend
maintainers:
  - name: Your Name
    email: you@example.com
EOF

# worker/Chart.yaml
cat > charts/worker/Chart.yaml << EOF
apiVersion: v2
name: worker
description: Background worker service
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - worker
  - queue
maintainers:
  - name: Your Name
    email: you@example.com
EOF
```

**3. Упакуй charts:**
```bash
mkdir packages
helm package charts/webapp -d packages/
helm package charts/api-service -d packages/
helm package charts/worker -d packages/

ls -la packages/
```

**4. Создай index для репозитория:**
```bash
cd packages
helm repo index . --url https://my-charts.example.com
cat index.yaml
```

**5. Протестируй локально:**
```bash
# Запусти простой HTTP сервер
python3 -m http.server 8080

# В другом терминале
helm repo add local http://localhost:8080
helm repo update
helm search repo local/
helm show chart local/webapp
helm show values local/webapp
```

**6. Установи из локального репозитория:**
```bash
helm install my-webapp local/webapp -n test --create-namespace
helm list -n test
kubectl get all -n test
```

**7. Обнови версию chart:**
```bash
cd ..

# Обнови версию в Chart.yaml
sed -i 's/version: 1.0.0/version: 1.1.0/' charts/webapp/Chart.yaml

# Упакуй новую версию
helm package charts/webapp -d packages/

# Обнови индекс (merge с существующим)
cd packages
helm repo index . --url https://my-charts.example.com --merge index.yaml

# Проверь что обе версии в индексе
cat index.yaml | grep -A 5 "webapp:"
```

**8. Обнови локальный репозиторий:**
```bash
helm repo update
helm search repo local/webapp --versions
```

**9. Обнови установленный release до новой версии:**
```bash
helm upgrade my-webapp local/webapp --version 1.1.0 -n test
helm history my-webapp -n test
```

**10. Очистка:**
```bash
helm uninstall my-webapp -n test
helm repo remove local
```

### 🚀 Бонус (новое)

**1. Используй OCI Registry (Docker Hub/GitHub Container Registry):**

```bash
# Логин в GitHub Container Registry
export CR_PAT=YOUR_TOKEN
echo $CR_PAT | helm registry login ghcr.io -u USERNAME --password-stdin

# Упакуй и push
helm package charts/webapp
helm push webapp-1.0.0.tgz oci://ghcr.io/username

# Установи из OCI
helm install my-webapp oci://ghcr.io/username/webapp --version 1.0.0

# List tags
helm show chart oci://ghcr.io/username/webapp --version 1.0.0
```

**2. Создай GitHub Actions для автоматической публикации:**

```yaml
# .github/workflows/release-charts.yaml
name: Release Charts

on:
  push:
    branches:
      - main
    paths:
      - 'charts/**'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "$GITHUB_ACTOR"
          git config user.email "$GITHUB_ACTOR@users.noreply.github.com"

      - name: Install Helm
        uses: azure/setup-helm@v3

      - name: Run chart-releaser
        uses: helm/chart-releaser-action@v1.5.0
        env:
          CR_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
```

**3. Настрой Helm repository с аутентификацией:**

```bash
# Создай basic auth credentials
htpasswd -c auth myuser

# Запусти nginx с auth
kubectl create secret generic helm-auth --from-file=auth -n helm-repo
kubectl create configmap nginx-config --from-file=nginx.conf -n helm-repo

# nginx.conf
server {
    listen 80;
    root /usr/share/nginx/html;
    
    auth_basic "Helm Repository";
    auth_basic_user_file /etc/nginx/auth;
    
    location / {
        autoindex on;
    }
}

# Использование
helm repo add private-repo https://charts.example.com \
  --username myuser \
  --password mypassword
```

---

## Модуль 7: Продвинутые техники (30 минут)

### 🎯 Напоминалка

**Named Templates (_helpers.tpl):**
```yaml
# templates/_helpers.tpl
{{/*
Expand the name of the chart
*/}}
{{- define "my-chart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name
*/}}
{{- define "my-chart.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "my-chart.labels" -}}
helm.sh/chart: {{ include "my-chart.chart" . }}
{{ include "my-chart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "my-chart.selectorLabels" -}}
app.kubernetes.io/name: {{ include "my-chart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Custom function - environment specific config
*/}}
{{- define "my-chart.env.config" -}}
{{- if eq .Values.environment "production" }}
LOG_LEVEL: error
CACHE_TTL: "3600"
{{- else }}
LOG_LEVEL: debug
CACHE_TTL: "60"
{{- end }}
{{- end }}
```

**Использование Named Templates:**
```yaml
# В deployment.yaml
metadata:
  name: {{ include "my-chart.fullname" . }}
  labels:
    {{- include "my-chart.labels" . | nindent 4 }}
spec:
  selector:
    matchLabels:
      {{- include "my-chart.selectorLabels" . | nindent 6 }}
```

**Range и Loops:**
```yaml
# values.yaml
environments:
  - name: DATABASE_URL
    value: postgres://db:5432/mydb
  - name: REDIS_URL
    value: redis://cache:6379
  - name: LOG_LEVEL
    value: info

hosts:
  - example.com
  - www.example.com
  - api.example.com

# В template
env:
{{- range .Values.environments }}
- name: {{ .name }}
  value: {{ .value }}
{{- end }}

# С индексом
{{- range $index, $host := .Values.hosts }}
- host: {{ $host }}
  priority: {{ $index }}
{{- end }}

# С key-value
{{- range $key, $val := .Values.config }}
- name: {{ $key }}
  value: {{ $val | quote }}
{{- end }}
```

**With - изменение scope:**
```yaml
# values.yaml
database:
  host: postgres
  port: 5432
  credentials:
    username: admin
    password: secret

# В template
{{- with .Values.database }}
- name: DB_HOST
  value: {{ .host }}
- name: DB_PORT
  value: {{ .port | quote }}
{{- with .credentials }}
- name: DB_USER
  value: {{ .username }}
- name: DB_PASS
  value: {{ .password }}
{{- end }}
{{- end }}
```

**Файлы и ConfigMaps:**
```yaml
# templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "my-chart.fullname" . }}-config
data:
  # Включить содержимое файла
  config.json: |-
{{ .Files.Get "files/config.json" | indent 4 }}
  
  # Или использовать AsConfig
  {{- (.Files.Glob "configs/*").AsConfig | nindent 2 }}
  
  # Или AsSecrets (base64)
  {{- (.Files.Glob "secrets/*").AsSecrets | nindent 2 }}
```

**Capabilities - проверка возможностей кластера:**
```yaml
{{- if .Capabilities.APIVersions.Has "networking.k8s.io/v1/Ingress" }}
apiVersion: networking.k8s.io/v1
{{- else if .Capabilities.APIVersions.Has "networking.k8s.io/v1beta1/Ingress" }}
apiVersion: networking.k8s.io/v1beta1
{{- end }}
kind: Ingress

# Проверка версии Kubernetes
{{- if semverCompare ">=1.19-0" .Capabilities.KubeVersion.GitVersion }}
# Использовать новые фичи
{{- end }}
```

**Lookup - получение существующих ресурсов:**
```yaml
{{- $secret := lookup "v1" "Secret" .Release.Namespace "my-secret" }}
{{- if $secret }}
# Secret существует, используем его
data:
  password: {{ $secret.data.password }}
{{- else }}
# Secret не существует, создаем новый
data:
  password: {{ randAlphaNum 16 | b64enc }}
{{- end }}
```

**Include vs Template:**
```yaml
# template - вывод напрямую
{{- template "my-chart.labels" . }}

# include - можно постобработать
{{- include "my-chart.labels" . | nindent 4 }}
{{- include "my-chart.labels" . | indent 2 }}
```

**Validation и Required:**
```yaml
{{- if not .Values.image.repository }}
{{- fail "image.repository is required!" }}
{{- end }}

# Или с required
image: {{ required "image.repository is required!" .Values.image.repository }}:{{ .Values.image.tag }}
```

**Chart Development Best Practices:**

1. **Именование:**
```yaml
# Всегда используй fullname template
name: {{ include "my-chart.fullname" . }}

# Не хардкодь имена
# ❌ Плохо
name: my-app-deployment

# ✅ Хорошо
name: {{ include "my-chart.fullname" . }}-deployment
```

2. **Labels:**
```yaml
# Используй стандартные labels
labels:
  app.kubernetes.io/name: {{ include "my-chart.name" . }}
  app.kubernetes.io/instance: {{ .Release.Name }}
  app.kubernetes.io/version: {{ .Chart.AppVersion }}
  app.kubernetes.io/managed-by: {{ .Release.Service }}
  helm.sh/chart: {{ include "my-chart.chart" . }}
```

3. **Defaults:**
```yaml
# Всегда предоставляй defaults
image:
  repository: {{ .Values.image.repository | default "nginx" }}
  tag: {{ .Values.image.tag | default .Chart.AppVersion | default "latest" }}
```

4. **Комментарии в values.yaml:**
```yaml
# -- Number of replicas for the deployment
replicaCount: 2

image:
  # -- Docker image repository
  repository: nginx
  # -- Docker image tag (defaults to chart appVersion)
  tag: ""
```

### 💻 Задание

Создай production-ready chart с продвинутыми техниками:

**1. Создай базовую структуру:**
```bash
helm create advanced-app
cd advanced-app
```

**2. Расширь _helpers.tpl:**
```yaml
# templates/_helpers.tpl - добавь к существующему
{{/*
Create chart name and version as used by the chart label
*/}}
{{- define "advanced-app.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Return the appropriate apiVersion for Ingress
*/}}
{{- define "advanced-app.ingress.apiVersion" -}}
{{- if .Capabilities.APIVersions.Has "networking.k8s.io/v1/Ingress" -}}
networking.k8s.io/v1
{{- else if .Capabilities.APIVersions.Has "networking.k8s.io/v1beta1/Ingress" -}}
networking.k8s.io/v1beta1
{{- else -}}
extensions/v1beta1
{{- end -}}
{{- end -}}

{{/*
Environment-specific configuration
*/}}
{{- define "advanced-app.env.vars" -}}
- name: ENVIRONMENT
  value: {{ .Values.environment | default "development" }}
- name: LOG_LEVEL
  value: {{ .Values.logLevel | default "info" }}
{{- if eq .Values.environment "production" }}
- name: DEBUG
  value: "false"
- name: CACHE_ENABLED
  value: "true"
{{- else }}
- name: DEBUG
  value: "true"
- name: CACHE_ENABLED
  value: "false"
{{- end }}
{{- end }}

{{/*
Validate required values
*/}}
{{- define "advanced-app.validate" -}}
{{- if not .Values.image.repository }}
  {{- fail "image.repository is required" }}
{{- end }}
{{- if and .Values.ingress.enabled (not .Values.ingress.hosts) }}
  {{- fail "ingress.hosts is required when ingress is enabled" }}
{{- end }}
{{- end }}
```

**3. Создай расширенный values.yaml:**
```yaml
# values.yaml
# Default values for advanced-app

# -- Application environment (development, staging, production)
environment: development

# -- Log level
logLevel: info

# -- Number of replicas
replicaCount: 2

image:
  # -- Image repository
  repository: nginx
  # -- Image pull policy
  pullPolicy: IfNotPresent
  # -- Image tag (overrides chart appVersion)
  tag: ""

# -- Image pull secrets
imagePullSecrets: []

# -- Override chart name
nameOverride: ""
# -- Override full name
fullnameOverride: ""

serviceAccount:
  # -- Create service account
  create: true
  # -- Annotations for service account
  annotations: {}
  # -- Service account name
  name: ""

# -- Pod annotations
podAnnotations: {}

# -- Pod security context
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 2000

# -- Container security context
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
    - ALL

service:
  # -- Service type
  type: ClusterIP
  # -- Service port
  port: 80
  # -- Target port
  targetPort: 8080
  # -- Node port (if type is NodePort)
  nodePort: null

ingress:
  # -- Enable ingress
  enabled: false
  # -- Ingress class name
  className: "nginx"
  # -- Ingress annotations
  annotations: {}
    # cert-manager.io/cluster-issuer: letsencrypt-prod
    # nginx.ingress.kubernetes.io/rate-limit: "100"
  # -- Ingress hosts
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: Prefix
  # -- Ingress TLS configuration
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

# -- Resource limits and requests
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi

# -- Liveness probe configuration
livenessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

# -- Readiness probe configuration
readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3

# -- Horizontal Pod Autoscaler
autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80

# -- Node selector
nodeSelector: {}

# -- Tolerations
tolerations: []

# -- Affinity rules
affinity: {}

# -- Extra environment variables
extraEnv: []
# - name: CUSTOM_VAR
#   value: "custom-value"

# -- Extra environment variables from ConfigMap/Secret
extraEnvFrom: []
# - configMapRef:
#     name: extra-config
# - secretRef:
#     name: extra-secrets

# -- Extra volumes
extraVolumes: []
# - name: config
#   configMap:
#     name: app-config

# -- Extra volume mounts
extraVolumeMounts: []
# - name: config
#   mountPath: /etc/config

# -- ConfigMap data
configData: {}
# config.json: |
#   {
#     "key": "value"
#   }

# -- Secret data
secretData: {}
# api-key: supersecret
```

**4. Обнови deployment.yaml с продвинутыми фичами:**
```yaml
# templates/deployment.yaml
{{- include "advanced-app.validate" . -}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "advanced-app.fullname" . }}
  labels:
    {{- include "advanced-app.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "advanced-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
        {{- with .Values.podAnnotations }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
      labels:
        {{- include "advanced-app.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "advanced-app.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
      - name: {{ .Chart.Name }}
        securityContext:
          {{- toYaml .Values.securityContext | nindent 12 }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
          protocol: TCP
        env:
        {{- include "advanced-app.env.vars" . | nindent 8 }}
        {{- range .Values.extraEnv }}
        - name: {{ .name }}
          value: {{ .value | quote }}
        {{- end }}
        {{- with .Values.extraEnvFrom }}
        envFrom:
          {{- toYaml . | nindent 8 }}
        {{- end }}
        {{- with .Values.livenessProbe }}
        livenessProbe:
          {{- toYaml . | nindent 10 }}
        {{- end }}
        {{- with .Values.readinessProbe }}
        readinessProbe:
          {{- toYaml . | nindent 10 }}
        {{- end }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        {{- if .Values.configData }}
        - name: config
          mountPath: /etc/config
          readOnly: true
        {{- end }}
        {{- with .Values.extraVolumeMounts }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
      volumes:
      - name: tmp
        emptyDir: {}
      {{- if .Values.configData }}
      - name: config
        configMap:
          name: {{ include "advanced-app.fullname" . }}
      {{- end }}
      {{- with .Values.extraVolumes }}
      {{- toYaml . | nindent 6 }}
      {{- end }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

**5. Создай условный ConfigMap:**
```yaml
# templates/configmap.yaml
{{- if .Values.configData }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "advanced-app.fullname" . }}
  labels:
    {{- include "advanced-app.labels" . | nindent 4 }}
data:
  {{- range $key, $value := .Values.configData }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
{{- end }}
```

**6. Создай условный Secret:**
```yaml
# templates/secret.yaml
{{- if .Values.secretData }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "advanced-app.fullname" . }}
  labels:
    {{- include "advanced-app.labels" . | nindent 4 }}
type: Opaque
data:
  {{- range $key, $value := .Values.secretData }}
  {{ $key }}: {{ $value | b64enc | quote }}
  {{- end }}
{{- end }}
```

**7. Обнови ingress.yaml с версионностью API:**
```yaml
# templates/ingress.yaml
{{- if .Values.ingress.enabled -}}
apiVersion: {{ include "advanced-app.ingress.apiVersion" . }}
kind: Ingress
metadata:
  name: {{ include "advanced-app.fullname" . }}
  labels:
    {{- include "advanced-app.labels" . | nindent 4 }}
  {{- with .Values.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  {{- if .Values.ingress.className }}
  ingressClassName: {{ .Values.ingress.className }}
  {{- end }}
  {{- if .Values.ingress.tls }}
  tls:
    {{- range .Values.ingress.tls }}
    - hosts:
        {{- range .hosts }}
        - {{ . | quote }}
        {{- end }}
      secretName: {{ .secretName }}
    {{- end }}
  {{- end }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ include "advanced-app.fullname" $ }}
                port:
                  number: {{ $.Values.service.port }}
          {{- end }}
    {{- end }}
{{- end }}
```

**8. Добавь HPA:**
```yaml
# templates/hpa.yaml
{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "advanced-app.fullname" . }}
  labels:
    {{- include "advanced-app.labels" . | nindent 4 }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "advanced-app.fullname" . }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  metrics:
  {{- if .Values.autoscaling.targetCPUUtilizationPercentage }}
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
  {{- end }}
  {{- if .Values.autoscaling.targetMemoryUtilizationPercentage }}
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: {{ .Values.autoscaling.targetMemoryUtilizationPercentage }}
  {{- end }}
{{- end }}
```

**9. Создай values для разных окружений:**
```yaml
# values-prod.yaml
environment: production
logLevel: error
replicaCount: 3

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: app.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: app-tls
      hosts:
        - app.example.com
```

**10. Тестируй:**
```bash
# Lint
helm lint .
helm lint . -f values-prod.yaml

# Template
helm template advanced-app . --debug
helm template advanced-app . -f values-prod.yaml

# Dry-run
helm install advanced-app . --dry-run --debug -n test --create-namespace

# Установка
helm install advanced-app . -n test --create-namespace
helm install advanced-app-prod . -f values-prod.yaml -n production --create-namespace

# Проверка
helm list -A
kubectl get all -n test
kubectl get all -n production
```

### 🚀 Бонус (новое)

**1. Используй lookup для сохранения существующих значений:**

```yaml
# templates/secret.yaml
{{- $secret := lookup "v1" "Secret" .Release.Namespace (include "advanced-app.fullname" .) }}
{{- $password := "" }}
{{- if $secret }}
  {{- $password = index $secret.data "password" | b64dec }}
{{- else }}
  {{- $password = randAlphaNum 32 }}
{{- end }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "advanced-app.fullname" . }}
data:
  password: {{ $password | b64enc | quote }}
```

**2. Создай library chart для переиспользования:**

```bash
# Создай library chart
helm create common-lib
cd common-lib

# Chart.yaml
cat > Chart.yaml << EOF
apiVersion: v2
name: common-lib
description: Common templates library
type: library
version: 1.0.0
EOF

# templates/_deployment.tpl
cat > templates/_deployment.tpl << 'EOF'
{{- define "common.deployment" -}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "common.fullname" . }}
  labels:
    {{- include "common.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "common.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "common.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - name: http
          containerPort: 80
{{- end }}
EOF

# Используй в другом chart через зависимость
```

**3. Добавь schema validation с примерами:**

```json
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["image"],
  "properties": {
    "environment": {
      "type": "string",
      "enum": ["development", "staging", "production"],
      "description": "Application environment"
    },
    "replicaCount": {
      "type": "integer",
      "minimum": 1,
      "maximum": 100,
      "examples": [2, 3, 5]
    },
    "image": {
      "type": "object",
      "required": ["repository"],
      "properties": {
        "repository": {
          "type": "string",
          "pattern": "^[a-z0-9-./]+$"
        },
        "tag": {
          "type": "string"
        }
      }
    },
    "resources": {
      "type": "object",
      "properties": {
        "limits": {
          "type": "object",
          "properties": {
            "cpu": {"type": "string", "pattern": "^[0-9]+m?$"},
            "memory": {"type": "string", "pattern": "^[0-9]+(Mi|Gi)$"}
          }
        }
      }
    }
  }
}
```

---

## Финальный проект (60 минут)

### Задача: Создать полноценный production-ready Helm chart для микросервисного приложения

Создай umbrella chart для комплексного приложения со всеми best practices.

**Архитектура:**
- Frontend (React/Vue)
- Backend API (REST)
- Worker (Background jobs)
- PostgreSQL Database
- Redis Cache
- NGINX Ingress
- Monitoring (Prometheus/Grafana)

**Требования:**

**1. Структура проекта:**
```
microservices-platform/
├── Chart.yaml                 # Umbrella chart
├── values.yaml               # Общие настройки
├── values-dev.yaml          # Dev окружение
├── values-staging.yaml      # Staging окружение
├── values-prod.yaml         # Production окружение
├── charts/                   # Sub-charts
│   ├── frontend/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   ├── backend/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   └── worker/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
├── templates/
│   ├── _helpers.tpl
│   ├── namespace.yaml
│   └── tests/
└── README.md
```

**2. Frontend Chart требования:**
- Deployment с 2+ репликами
- HPA (min=2, max=10, cpu=70%)
- Service (ClusterIP)
- ConfigMap для nginx.conf
- Liveness/Readiness probes
- Security context (non-root)
- Resource limits
- Anti-affinity для распределения по нодам

**3. Backend Chart требования:**
- Deployment с 3+ репликами
- HPA (min=3, max=15, cpu=75%)
- Service (ClusterIP)
- ConfigMap для конфигурации приложения
- Secret для API keys и DB credentials
- Init container для database migration
- Liveness/Readiness probes на /health и /ready
- ServiceAccount с минимальными правами RBAC
- NetworkPolicy (allow from frontend, allow to db/redis)
- PodDisruptionBudget (minAvailable: 2)

**4. Worker Chart требования:**
- Deployment с 2+ репликами
- HPA на основе custom metrics (queue length)
- ConfigMap для worker конфигурации
- Secret для credentials
- Liveness probe
- Priority class для фоновых задач
- Resource requests/limits

**5. Dependencies (в Chart.yaml umbrella):**
```yaml
dependencies:
  - name: postgresql
    version: "12.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
  - name: redis
    version: "17.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
  - name: prometheus
    version: "25.x.x"
    repository: "https://prometheus-community.github.io/helm-charts"
    condition: monitoring.prometheus.enabled
    alias: prometheus
```

**6. Ingress настройки:**
- Единый Ingress для всех сервисов
- TLS/SSL сертификаты (cert-manager integration)
- Маршрутизация:
  - `/` → Frontend
  - `/api/*` → Backend
  - `/health` → Health endpoint
- Rate limiting
- CORS headers
- Custom error pages
- Basic Auth для staging окружения

**7. Hooks:**
- **pre-install**: Проверка prerequisites (namespace, secrets)
- **post-install**: Создание default admin user
- **pre-upgrade**: Database backup
- **post-upgrade**: Database migration
- **pre-delete**: Cleanup job
- **test**: Smoke tests для всех сервисов

**8. Monitoring и observability:**
- ServiceMonitor для Prometheus
- Grafana dashboard ConfigMap
- Loki для логов (опционально)
- Jaeger для tracing (опционально)
- Custom metrics endpoints

**9. Security:**
- NetworkPolicies для всех компонентов
- PodSecurityContext (non-root, read-only FS)
- SecurityContext для контейнеров
- RBAC с минимальными правами
- Secret management (поддержка external-secrets)
- Image pull secrets

**10. Multi-environment support:**
```yaml
# values.yaml (base)
global:
  environment: development
  domain: example.local
  storageClass: standard

# values-prod.yaml
global:
  environment: production
  domain: example.com
  storageClass: fast-ssd

frontend:
  replicaCount: 3
  resources:
    limits:
      cpu: 500m
      memory: 512Mi

backend:
  replicaCount: 5
  autoscaling:
    enabled: true
    minReplicas: 5
    maxReplicas: 20

postgresql:
  primary:
    persistence:
      size: 100Gi
  readReplicas:
    replicaCount: 2
```

**11. CI/CD Integration:**
- GitHub Actions workflow для lint/test/package
- Automated versioning
- Changelog generation
- Automated publishing to chart repository

**12. Documentation:**
- Comprehensive README.md
- Architecture diagram
- Installation guide
- Configuration guide
- Troubleshooting section
- Upgrade guide
- Values reference (auto-generated with helm-docs)

**Пример начальной структуры umbrella Chart.yaml:**
```yaml
apiVersion: v2
name: microservices-platform
description: Complete microservices platform with monitoring
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - microservices
  - platform
  - production-ready
maintainers:
  - name: Your Name
    email: you@example.com
dependencies:
  # Приложения (локальные sub-charts)
  - name: frontend
    version: "1.0.0"
    repository: "file://charts/frontend"
    condition: frontend.enabled
  - name: backend
    version: "1.0.0"
    repository: "file://charts/backend"
    condition: backend.enabled
  - name: worker
    version: "1.0.0"
    repository: "file://charts/worker"
    condition: worker.enabled
  
  # Инфраструктура (внешние charts)
  - name: postgresql
    version: "12.1.9"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
    tags:
      - database
  - name: redis
    version: "17.3.14"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
    tags:
      - cache
  - name: kube-prometheus-stack
    version: "51.0.0"
    repository: "https://prometheus-community.github.io/helm-charts"
    condition: monitoring.enabled
    alias: monitoring
```

**Пример values.yaml для umbrella:**
```yaml
# Global настройки
global:
  environment: development
  domain: app.local
  imageRegistry: docker.io
  imagePullSecrets: []
  storageClass: ""

# Feature flags
tags:
  database: true
  cache: true
  monitoring: true

# Frontend
frontend:
  enabled: true
  replicaCount: 2
  image:
    repository: my-org/frontend
    tag: "1.0.0"
  service:
    type: ClusterIP
    port: 80
  ingress:
    enabled: true
    path: /
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 10

# Backend
backend:
  enabled: true
  replicaCount: 3
  image:
    repository: my-org/backend
    tag: "1.0.0"
  service:
    type: ClusterIP
    port: 8080
  ingress:
    enabled: true
    path: /api
  database:
    host: "{{ .Release.Name }}-postgresql"
    port: 5432
    name: app
  redis:
    host: "{{ .Release.Name }}-redis-master"
    port: 6379
  autoscaling:
    enabled: true
    minReplicas: 3
    maxReplicas: 15

# Worker
worker:
  enabled: true
  replicaCount: 2
  image:
    repository: my-org/worker
    tag: "1.0.0"
  autoscaling:
    enabled: false

# PostgreSQL
postgresql:
  enabled: true
  auth:
    username: app
    password: changeme
    database: app
  primary:
    persistence:
      enabled: true
      size: 10Gi
    resources:
      limits:
        memory: 512Mi
      requests:
        memory: 256Mi

# Redis
redis:
  enabled: true
  auth:
    enabled: true
    password: changeme
  master:
    persistence:
      enabled: false
    resources:
      limits:
        memory: 256Mi
      requests:
        memory: 128Mi

# Monitoring
monitoring:
  enabled: false
  prometheus:
    enabled: true
  grafana:
    enabled: true
    adminPassword: admin

# Ingress (общий)
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
  hosts:
    - host: "{{ .Values.global.domain }}"
  tls:
    - secretName: app-tls
      hosts:
        - "{{ .Values.global.domain }}"
```

**Команды для работы:**
```bash
# Создание структуры
helm create microservices-platform
cd microservices-platform

# Создание sub-charts
mkdir -p charts
cd charts
helm create frontend
helm create backend
helm create worker
cd ..

# Настройка зависимостей
helm dependency update

# Тестирование
helm lint .
helm template microservices-platform . --debug
helm template microservices-platform . -f values-prod.yaml

# Unit тесты
helm unittest .

# Dry-run
helm install microservices-platform . \
  --dry-run --debug \
  -n production --create-namespace

# Установка dev
helm install ms-dev . \
  -f values-dev.yaml \
  -n development --create-namespace

# Установка staging
helm install ms-staging . \
  -f values-staging.yaml \
  -n staging --create-namespace

# Установка production
helm install ms-prod . \
  -f values-prod.yaml \
  -n production --create-namespace

# Мониторинг установки
watch -n 2 'helm list -A && echo && kubectl get pods -A | grep ms-'

# Upgrade
helm upgrade ms-prod . -f values-prod.yaml -n production

# Rollback
helm rollback ms-prod 1 -n production

# Тесты
helm test ms-prod -n production --logs

# Cleanup
helm uninstall ms-dev -n development
helm uninstall ms-staging -n staging
helm uninstall ms-prod -n production
```

**Дополнительные улучшения:**
- GitOps с ArgoCD/Flux
- Canary deployments с Flagger
- Service Mesh (Istio/Linkerd) integration
- Backup/Restore процедуры
- Disaster Recovery план
- Load testing (k6/Locust)
- Security scanning (Trivy/Snyk)
- Cost optimization
- Performance optimization

---

## Справочная секция: Быстрые шпаргалки

### Helm команды - полный список

```bash
# Установка и управление
helm install <release> <chart>              # Установить
helm upgrade <release> <chart>              # Обновить
helm upgrade --install <release> <chart>    # Установить или обновить
helm uninstall <release>                    # Удалить
helm rollback <release> <revision>          # Откатить

# Информация
helm list                                   # Список releases
helm list -A                               # Все namespaces
helm list --all-namespaces                 # Все namespaces
helm status <release>                      # Статус
helm history <release>                     # История
helm get values <release>                  # Values
helm get manifest <release>                # Манифесты
helm get all <release>                     # Вся информация
helm get hooks <release>                   # Hooks
helm get notes <release>                   # Notes

# Работа с charts
helm create <name>                         # Создать chart
helm package <chart-dir>                   # Упаковать
helm lint <chart-dir>                      # Проверить
helm template <release> <chart>            # Рендер без установки
helm show chart <chart>                    # Показать Chart.yaml
helm show values <chart>                   # Показать values.yaml
helm show readme <chart>                   # Показать README
helm show all <chart>                      # Показать всё
helm pull <chart>                          # Скачать chart
helm pull <chart> --untar                  # Скачать и распаковать
helm dependency update <chart-dir>         # Обновить зависимости
helm dependency build <chart-dir>          # Собрать зависимости
helm dependency list <chart-dir>           # Список зависимостей

# Репозитории
helm repo add <name> <url>                 # Добавить
helm repo list                             # Список
helm repo update                           # Обновить
helm repo remove <name>                    # Удалить
helm search repo <keyword>                 # Поиск в репозитории
helm search hub <keyword>                  # Поиск в Artifact Hub

# Тестирование
helm test <release>                        # Запустить тесты
helm test <release> --logs                 # С выводом логов

# Отладка
helm install <release> <chart> --dry-run --debug
helm template <release> <chart> --debug
helm lint <chart-dir>
helm get manifest <release> | kubectl diff -f -

# Флаги
--namespace / -n <namespace>               # Указать namespace
--create-namespace                         # Создать namespace
--values / -f <file>                       # Values файл
--set key=value                           # Установить значение
--set-string key=value                    # Установить строку
--set-file key=path                       # Установить из файла
--version <version>                       # Версия chart
--timeout <duration>                      # Таймаут (default 5m)
--wait                                    # Ждать готовности
--wait-for-jobs                          # Ждать Jobs
--atomic                                  # Откатить при ошибке
--cleanup-on-fail                        # Очистить при ошибке
--force                                   # Принудительно
--debug                                   # Debug режим
--dry-run                                # Не применять изменения

# Плагины
helm plugin list                          # Список плагинов
helm plugin install <url>                 # Установить плагин
helm plugin update <name>                 # Обновить плагин
helm plugin uninstall <name>              # Удалить плагин

# Environment variables
$HELM_CACHE_HOME                          # Кэш директория
$HELM_CONFIG_HOME                         # Конфиг директория
$HELM_DATA_HOME                          # Data директория
$HELM_DEBUG                              # Debug режим
$HELM_NAMESPACE                          # Default namespace
$HELM_KUBECONTEXT                        # Kubecontext
```

### Template функции - шпаргалка

```yaml
# Строки
{{ quote .Values.str }}                   # "value"
{{ squote .Values.str }}                  # 'value'
{{ upper .Values.str }}                   # UPPERCASE
{{ lower .Values.str }}                   # lowercase
{{ title .Values.str }}                   # Title Case
{{ untitle .Values.str }}                 # remove title case
{{ repeat 3 "hello" }}                    # hellohellohello
{{ trim .Values.str }}                    # удалить пробелы
{{ trimAll "/" .Values.str }}             # удалить символы
{{ trimPrefix "pre-" .Values.str }}       # удалить префикс
{{ trimSuffix "-post" .Values.str }}      # удалить суффикс
{{ contains "sub" .Values.str }}          # проверка подстроки
{{ hasPrefix "pre" .Values.str }}         # проверка префикса
{{ hasSuffix "post" .Values.str }}        # проверка суффикса
{{ replace "old" "new" .Values.str }}     # замена
{{ substr 0 5 .Values.str }}              # подстрока
{{ trunc 63 .Values.str }}                # обрезать до длины
{{ shuffle .Values.str }}                 # перемешать
{{ regexMatch "^[a-z]+$" .Values.str }}   # regex проверка
{{ regexReplaceAll "a" "b" .Values.str }} # regex замена

# Числа
{{ add 1 2 }}                             # 3
{{ sub 5 2 }}                             # 3
{{ mul 2 3 }}                             # 6
{{ div 10 2 }}                            # 5
{{ mod 10 3 }}                            # 1
{{ max 1 2 3 }}                           # 3
{{ min 1 2 3 }}                           # 1
{{ ceil 1.5 }}                            # 2
{{ floor 1.5 }}                           # 1
{{ round 1.5 2 }}                         # 1.50

# Дата и время
{{ now }}                                 # текущее время
{{ date "2006-01-02" now }}              # форматирование даты
{{ dateInZone "2006-01-02" now "UTC" }}  # дата в зоне
{{ durationRound "2h" }}                  # округлить duration

# Default и fallback
{{ default "default" .Values.key }}       # значение по умолчанию
{{ empty .Values.key }}                   # проверка на пустоту
{{ coalesce .Val1 .Val2 "default" }}     # первое не пустое
{{ ternary "yes" "no" .Values.bool }}    # тернарный оператор

# Type conversion
{{ toString .Values.num }}                # к строке
{{ toJson .Values.obj }}                  # к JSON
{{ toPrettyJson .Values.obj }}           # к JSON с форматированием
{{ toYaml .Values.obj }}                  # к YAML
{{ toToml .Values.obj }}                  # к TOML
{{ fromJson .Values.json }}               # из JSON
{{ fromYaml .Values.yaml }}               # из YAML

# Encoding
{{ b64enc .Values.str }}                  # base64 encode
{{ b64dec .Values.b64 }}                  # base64 decode
{{ urlquery .Values.str }}                # URL encode

# Lists
{{ list 1 2 3 }}                          # создать список
{{ append .Values.list "item" }}          # добавить элемент
{{ prepend .Values.list "item" }}         # добавить в начало
{{ first .Values.list }}                  # первый элемент
{{ rest .Values.list }}                   # все кроме первого
{{ last .Values.list }}                   # последний элемент
{{ initial .Values.list }}                # все кроме последнего
{{ reverse .Values.list }}                # реверс
{{ uniq .Values.list }}                   # уникальные
{{ sortAlpha .Values.list }}              # сортировка
{{ join "," .Values.list }}               # объединить в строку
{{ has "item" .Values.list }}             # проверка наличия
{{ without .Values.list "item" }}         # исключить элемент
{{ compact .Values.list }}                # удалить пустые

# Dictionaries
{{ dict "key1" "val1" "key2" "val2" }}   # создать dict
{{ get .Values.dict "key" }}              # получить значение
{{ set .Values.dict "key" "val" }}        # установить значение
{{ unset .Values.dict "key" }}            # удалить ключ
{{ hasKey .Values.dict "key" }}           # проверка ключа
{{ pluck "key" .Values.dicts }}           # выбрать значения
{{ merge .dict1 .dict2 }}                 # объединить
{{ mergeOverwrite .dict1 .dict2 }}       # объединить с перезаписью
{{ keys .Values.dict }}                   # список ключей
{{ pick .Values.dict "k1" "k2" }}        # выбрать ключи
{{ omit .Values.dict "k1" "k2" }}        # исключить ключи
{{ values .Values.dict }}                 # список значений

# Crypto
{{ sha256sum .Values.str }}               # SHA256 хеш
{{ sha1sum .Values.str }}                 # SHA1 хеш
{{ adler32sum .Values.str }}              # Adler32 хеш
{{ htpasswd "user" "pass" }}              # htpasswd

# Random
{{ randAlpha 10 }}                        # случайные буквы
{{ randAlphaNum 10 }}                     # буквы и цифры
{{ randNumeric 10 }}                      # случайные цифры
{{ randAscii 10 }}                        # ASCII символы
{{ uuidv4 }}                              # UUID v4

# Semantic версии
{{ semver "1.2.3" }}                      # parse версии
{{ semverCompare ">=1.2.0" .Version }}   # сравнение версий

# Пути
{{ base "path/to/file.txt" }}             # file.txt
{{ dir "path/to/file.txt" }}              # path/to
{{ ext "file.txt" }}                      # .txt
{{ clean "path//to/./file" }}             # path/to/file

# Kubernetes
{{ include "chart.fullname" . }}          # вызов named template
{{ required "msg" .Values.required }}     # обязательное значение
{{ fail "error message" }}                # ошибка с сообщением
{{ lookup "v1" "Secret" "ns" "name" }}   # получить ресурс

# Flow control
{{- if .Values.enabled }}                 # if
{{- else if .Values.other }}              # else if
{{- else }}                               # else
{{- end }}                                # конец блока

{{- range .Values.items }}                # цикл
  {{ . }}                                 # текущий элемент
{{- end }}

{{- with .Values.section }}               # изменить scope
  {{ .key }}                              # .Values.section.key
{{- end }}

# Whitespace control
{{- "no leading space" }}                 # убрать пробелы слева
{{ "no trailing space" -}}                # убрать пробелы справа
{{- "no spaces" -}}                       # убрать пробелы с обеих сторон

# Indentation
{{ indent 2 "text" }}                     # 2 пробела слева
{{ nindent 2 "text" }}                    # newline + 2 пробела
```

### Best Practices Checklist

**Chart Development:**
- ✅ Используй semantic versioning
- ✅ Всегда указывай appVersion в Chart.yaml
- ✅ Пиши подробное description
- ✅ Указывай keywords для поиска
- ✅ Добавь maintainers информацию
- ✅ Используй _helpers.tpl для переиспользования
- ✅ Всегда используй include вместо template
- ✅ Добавляй стандартные labels (app.kubernetes.io/*)
- ✅ Используй fullname template для имен ресурсов
- ✅ Не хардкодь namespace в templates
- ✅ Добавь NOTES.txt с инструкциями
- ✅ Создай values.schema.json для валидации
- ✅ Пиши комментарии в values.yaml
- ✅ Группируй related values вместе
- ✅ Предоставляй reasonable defaults
- ✅ Делай всё настраиваемым через values
- ✅ Используй conditions для опциональных ресурсов
- ✅ Добавь tests/ для smoke tests
- ✅ Используй hooks где необходимо
- ✅ Добавь .helmignore
- ✅ Генерируй README.md с helm-docs

**Security:**
- ✅ Не храни secrets в values.yaml
- ✅ Используй securityContext
- ✅ Запускай контейнеры от non-root user
- ✅ Используй readOnlyRootFilesystem
- ✅ Drop ALL capabilities
- ✅ Не используй privileged режим
- ✅ Добавь NetworkPolicies
- ✅ Используй RBAC с минимальными правами
- ✅ Валидируй inputs
- ✅ Используй image digests в production

**Production Readiness:**
- ✅ Установи resource requests/limits
- ✅ Добавь liveness/readiness probes
- ✅ Настрой HPA
- ✅ Используй PodDisruptionBudget
- ✅ Настрой anti-affinity
- ✅ Добавь monitoring (ServiceMonitor)
- ✅ Настрой logging
- ✅ Добавь backup процедуры
- ✅ Настрой alerting
- ✅ Документируй upgrade процедуру
- ✅ Тестируй rollback
- ✅ Используй canary deployments для критичных сервисов

**Testing:**
- ✅ Запускай helm lint перед commit
- ✅ Используй helm unittest для unit тестов
- ✅ Тестируй с helm template
- ✅ Используй --dry-run перед установкой
- ✅ Тестируй на разных K8s версиях
- ✅ Тестируй upgrade сценарии
- ✅ Тестируй rollback
- ✅ Проверяй что hooks работают
- ✅ Запускай helm test после установки

---

## План повторения

### При первом прохождении (2-3 часа):
- Пройди модули 1-3 обязательно (основы)
- Модули 4-5 по желанию
- Финальный проект упрощенный (только umbrella chart без всех фич)

### При повторном прохождении (через 6-12 месяцев):
- Бегло просмотри теорию модулей 1-3
- Сфокусируйся на бонусных заданиях
- Пройди модули 4-7 обязательно
- Финальный проект полностью с всеми best practices

### Для закрепления:

- Автоматизируй деплой с помощью CI/CD (GitHub Actions, GitLab CI, ArgoCD)
- Реализуй GitOps подход: храни Helm charts и values в Git, применяй через ArgoCD
- Настрой автоматическое тестирование charts при каждом изменении
- Внедри проверку безопасности (kube-score, checkov, trivy) в pipeline
- Создай свою библиотеку common charts для повторного использования
- Организуй внутренний Helm repository (ChartMuseum, Harbor, Nexus)
- Настрой автоматический выпуск версий при тегировании в Git

Дополнительные ресурсы
Официальная документация:

    Helm Docs
    Chart Template Guide
    Best Practices
    Helm Hub (Artifact Hub)
    
Полезные инструменты:

    - kubeval – валидация K8s манифестов
    - helm-docs – генерация документации
    - helm-unittest – unit-тесты для charts
    - helm-secrets – работа с зашифрованными values
    - helm-diff – просмотр изменений перед применением
    - chart-testing – тестирование charts в CI

Сообщество и блоги:

    - Bitnami Helm Charts – примеры production-ready charts
    - Artifact Hub – поиск готовых charts
    - Helm на Medium – статьи и туториалы

Практические задания для самостоятельной работы:

    - Мигрируй существующие K8s манифесты в Helm chart
    - Настрой деплой одного chart в несколько кластеров
    - Реализуй canary-деплоймент с помощью Helm hooks и Flagger
    - Создай chart, который динамически генерирует конфигурацию на основе внешних источников (Consul, Vault)
    - Настрой мониторинг Helm releases с помощью Prometheus и Alertmanager

Итог

За 2–3 часа этого курса ты:

    ✅ Освежил в памяти ключевые концепции Helm
    ✅ Создал и настроил несколько charts разной сложности
    ✅ Узнал про зависимости, hooks, plugins и репозитории
    ✅ Познакомился с продвинутыми техниками (named templates, validation, multi-environment)
    ✅ Построил production-ready umbrella chart для микросервисного приложения
    ✅ Получил шпаргалки и чеклисты для ежедневной работы
    
Что дальше?
    - Примени полученные знания в своем проекте
    - Автоматизируй рутину с помощью Helm plugins
    - Внедри best practices из чеклиста
    - Исследуй интеграцию с GitOps-инструментами (ArgoCD, Flux)
    - Участвуй в разработке open-source charts

Формула успеха:
Практика → Автоматизация → Документация → Совершенствование

Удачи в использовании Helm! 🚀

Курс составлен для ежегодного/полугодового повторения.
Сохрани шпаргалки, возвращайся к сложным модулям и не забывай практиковаться.
