# GitLab CE Refresh: Ежегодный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые возможности GitLab CE за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная задача, которую нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Предварительные требования:**
- Доступ к GitLab CE (self-hosted или GitLab.com)
- Базовые знания Git
- SSH ключ настроен
- Docker установлен (для GitLab Runner)

---

## Модуль 1: Основы GitLab и интерфейс (15 минут)

### 🎯 Напоминалка

**Структура GitLab:**
```
GitLab Instance
├── Groups (организации)
│   ├── Subgroups (подгруппы)
│   └── Projects (проекты)
├── Users (пользователи)
└── Admin Area (администрирование)
```

**Основные концепции:**
```yaml
Project: Репозиторий + CI/CD + Issue tracking + Wiki
Group: Набор проектов + общие настройки
Namespace: Уникальное имя для group/user
Visibility:
  - Private: только участники
  - Internal: авторизованные пользователи
  - Public: все
```

**Роли и права:**
```
Guest       # Просмотр issues, комментарии
Reporter    # Pull, создание issues, просмотр CI/CD
Developer   # Push, merge requests, управление issues
Maintainer  # Управление проектом, защита веток
Owner       # Полный доступ, удаление проекта
```

**Структура проекта:**
```
my-project/
├── Repository      # Git репозиторий
├── Issues          # Задачи и баги
├── Merge Requests  # Pull requests
├── CI/CD           # Пайплайны и jobs
├── Operations      # Environments, Kubernetes
├── Packages        # Registry (Docker, NPM, Maven)
├── Wiki            # Документация
├── Snippets        # Фрагменты кода
└── Settings        # Настройки проекта
```

**Основные файлы конфигурации:**
```bash
.gitlab-ci.yml          # CI/CD конфигурация
.gitignore              # Игнорируемые файлы
README.md               # Описание проекта
LICENSE                 # Лицензия
CHANGELOG.md            # История изменений
.gitlab/                # GitLab-специфичные файлы
  ├── issue_templates/  # Шаблоны issues
  └── merge_request_templates/  # Шаблоны MR
```

**GitLab CLI (glab):**
```bash
# Установка
brew install glab  # macOS
# или
curl -s https://raw.githubusercontent.com/profclems/glab/trunk/scripts/install.sh | bash

# Аутентификация
glab auth login

# Основные команды
glab repo clone <project>
glab issue list
glab issue create
glab mr create
glab mr list
glab mr merge
glab ci view
```

**GitLab API:**
```bash
# Personal Access Token в Settings > Access Tokens
export GITLAB_TOKEN="your-token"
export GITLAB_URL="https://gitlab.com"

# Примеры запросов
# Список проектов
curl --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_URL/api/v4/projects"

# Создание issue
curl --request POST \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --data "title=Bug fix&description=Fix bug" \
  "$GITLAB_URL/api/v4/projects/:id/issues"
```

### 💻 Задание

Подготовь тестовое окружение:
1. Создай новый проект `devops-refresh`
2. Настрой SSH ключ (если еще не настроен):
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   cat ~/.ssh/id_ed25519.pub
   # Добавь в GitLab: Settings > SSH Keys
   ```
3. Клонируй проект:
   ```bash
   git clone git@gitlab.com:username/devops-refresh.git
   cd devops-refresh
   ```
4. Создай базовую структуру:
   ```bash
   echo "# DevOps Refresh Project" > README.md
   mkdir -p src tests docs
   touch .gitignore
   ```
5. Создай первый коммит:
   ```bash
   git add .
   git commit -m "Initial commit: project structure"
   git push origin main
   ```
6. Создай новую ветку и merge request:
   ```bash
   git checkout -b feature/add-documentation
   echo "## Documentation" >> README.md
   git add README.md
   git commit -m "docs: add documentation section"
   git push origin feature/add-documentation
   
   # Создай MR через веб-интерфейс или glab
   glab mr create --title "Add documentation" --description "Initial documentation"
   ```

### 🚀 Бонус (новое)

**1. Настрой Project Templates:**
```bash
# Создай шаблоны для issues и MR
mkdir -p .gitlab/issue_templates .gitlab/merge_request_templates

cat > .gitlab/issue_templates/Bug.md <<EOF
## Bug Description
Describe the bug

## Steps to Reproduce
1. Step 1
2. Step 2

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- OS: 
- Version:
EOF

cat > .gitlab/merge_request_templates/Default.md <<EOF
## What does this MR do?
Describe changes

## Related Issues
Closes #

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] CI/CD passes
EOF
```

**2. Используй GitLab CLI для автоматизации:**
```bash
# Создание issue через CLI
glab issue create \
  --title "Setup CI/CD pipeline" \
  --description "Configure basic pipeline" \
  --label "enhancement" \
  --assignee "@me"

# Быстрое создание MR с автозаполнением
glab mr create --fill --yes
```

**3. Настрой Git aliases для GitLab:**
```bash
cat >> ~/.gitconfig <<EOF
[alias]
    # GitLab shortcuts
    mr = !glab mr create --fill
    mrl = !glab mr list
    mrv = !glab mr view
    ci = !glab ci view
    
    # Better git log for GitLab
    lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
EOF
```

---

## Модуль 2: Git Workflow и Merge Requests (20 минут)

### 🎯 Напоминалка

**GitLab Flow vs Git Flow:**
```
GitLab Flow (рекомендуется):
main → production
  ↓
feature/fix branches → MR → main

Environment branches (опционально):
main → staging → production
```

**Работа с ветками:**
```bash
# Создание ветки
git checkout -b feature/new-feature

# Синхронизация с main
git fetch origin
git rebase origin/main

# Пуш в remote
git push origin feature/new-feature

# Удаление локальной ветки
git branch -d feature/new-feature

# Удаление remote ветки
git push origin --delete feature/new-feature
```

**Merge Request (MR) процесс:**
```
1. Создание feature ветки
2. Коммиты и push
3. Создание MR
4. Code Review
5. Approval (если настроен)
6. CI/CD проверки
7. Merge в target ветку
8. Удаление source ветки
```

**Merge стратегии:**
```yaml
Merge commit:     # Создает merge commit
  - Сохраняет историю
  - Видны все коммиты

Squash:          # Объединяет все коммиты в один
  - Чистая история
  - Теряется детализация

Fast-forward:    # Линейная история
  - Только если нет расхождений
  - Самая чистая история
```

**MR описание (markdown):**
```markdown
## What does this MR do?
Brief description of changes

## Related Issues
Closes #123, #456

## Screenshots (if applicable)
![Screenshot](url)

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing done

## Checklist
- [ ] Documentation updated
- [ ] CHANGELOG updated
- [ ] No breaking changes
```

**Code Review чек-лист:**
```
✅ Код читаемый и понятный
✅ Нет дублирования кода
✅ Нейминг переменных/функций понятен
✅ Есть тесты
✅ Документация обновлена
✅ Нет хардкода credentials
✅ Обработаны ошибки
✅ CI/CD пайплайн проходит
```

**Защита веток:**
```yaml
Protected Branches:
  main/master:
    - Push: No one / Maintainers
    - Merge: Maintainers
    - Force push: Disabled
    - Require MR: Enabled
    - Require approvals: 1-2
    - Require CI/CD: Enabled
```

**MR комментарии:**
```markdown
# Review комментарии
💭 Suggestion: можно улучшить
⚠️ Issue: нужно исправить
✅ Approved: выглядит хорошо
❓ Question: нужно пояснение

# GitLab suggestions (в комментариях)
```suggestion
// Предлагаемый код
const result = newFunction();
```


### 💻 Задание

Реализуй полный MR workflow:

1. **Создай новую feature ветку:**
   ```bash
   git checkout -b feature/add-api-endpoint
   ```

2. **Создай простой API endpoint:**
   ```bash
   mkdir -p src/api
   cat > src/api/server.js <<EOF
   const express = require('express');
   const app = express();
   const port = 3000;

   app.get('/health', (req, res) => {
     res.json({ status: 'healthy', timestamp: new Date() });
   });

   app.get('/api/users', (req, res) => {
     res.json([
       { id: 1, name: 'John Doe' },
       { id: 2, name: 'Jane Smith' }
     ]);
   });

   app.listen(port, () => {
     console.log(\`Server running on port \${port}\`);
   });
   EOF
   ```

3. **Создай тесты:**
   ```bash
   mkdir -p tests
   cat > tests/api.test.js <<EOF
   const request = require('supertest');
   const app = require('../src/api/server');

   describe('API Endpoints', () => {
     test('GET /health returns healthy status', async () => {
       const response = await request(app).get('/health');
       expect(response.status).toBe(200);
       expect(response.body.status).toBe('healthy');
     });

     test('GET /api/users returns user list', async () => {
       const response = await request(app).get('/api/users');
       expect(response.status).toBe(200);
       expect(Array.isArray(response.body)).toBe(true);
     });
   });
   EOF
   ```

4. **Создай package.json:**
   ```bash
   cat > package.json <<EOF
   {
     "name": "devops-refresh",
     "version": "1.0.0",
     "scripts": {
       "start": "node src/api/server.js",
       "test": "jest"
     },
     "dependencies": {
       "express": "^4.18.0"
     },
     "devDependencies": {
       "jest": "^29.0.0",
       "supertest": "^6.3.0"
     }
   }
   EOF
   ```

5. **Коммит и пуш:**
   ```bash
   git add .
   git commit -m "feat: add API endpoints with tests"
   git push origin feature/add-api-endpoint
   ```

6. **Создай MR через веб-интерфейс или CLI:**
   ```bash
   glab mr create \
     --title "feat: Add API endpoints" \
     --description "Implements health check and users API endpoints" \
     --label "enhancement" \
     --assignee "@me" \
     --fill
   ```

7. **Добавь комментарий-suggestion:**
   - Открой MR
   - Найди строку кода
   - Добавь suggestion через "Insert suggestion"

8. **Merge MR после прохождения проверок**

### 🚀 Бонус (новое)

**1. Настрой Draft MR для незавершенной работы:**
```bash
# Создание draft MR
glab mr create --draft --title "WIP: New feature"

# Или через префикс
git push origin feature/wip-feature
glab mr create --title "Draft: Work in progress"

# Когда готов - убери Draft статус через UI
```

**2. Используй MR Templates:**
```bash
# Создай специфичные шаблоны
cat > .gitlab/merge_request_templates/Feature.md <<EOF
## Feature Description
What feature does this MR implement?

## Motivation
Why is this feature needed?

## Implementation Details
Technical details of implementation

## Testing Plan
- [ ] Unit tests
- [ ] Integration tests
- [ ] Load tests

## Documentation
- [ ] API documentation updated
- [ ] README updated
- [ ] CHANGELOG entry added

## Screenshots/Demo
Add screenshots or demo links

## Related Issues
Closes #

/label ~feature
/assign @reviewer
EOF

# Использование при создании MR
glab mr create --template Feature.md
```

**3. Автоматизируй review через Quick Actions:**
```markdown
# В описании или комментарии MR
/approve
/assign @developer
/label ~bug ~high-priority
/milestone %v1.0
/estimate 2h
/spend 1h
/due 2024-12-31
/wip  # Пометить как Draft
/ready  # Убрать Draft статус
```

**4. Настрой автоматическое удаление веток:**
```yaml
# Settings > Merge requests
# ✅ Enable "Delete source branch" option by default
```

---

## Модуль 3: GitLab CI/CD - Основы (30 минут)

### 🎯 Напоминалка

**Структура .gitlab-ci.yml:**
```yaml
# Глобальные настройки
image: node:16
variables:
  NODE_ENV: production

# Стадии (порядок выполнения)
stages:
  - build
  - test
  - deploy

# Кеш (ускорение)
cache:
  paths:
    - node_modules/

# До всех jobs
before_script:
  - npm install

# Job определение
job_name:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
  only:
    - main
```

**Основные ключевые слова:**
```yaml
script:           # Команды для выполнения (обязательно)
stage:            # Стадия выполнения
image:            # Docker образ
services:         # Дополнительные сервисы (DB, Redis)
before_script:    # До script
after_script:     # После script
variables:        # Переменные окружения
artifacts:        # Файлы для передачи между jobs
cache:            # Кеширование зависимостей
dependencies:     # Какие artifacts использовать
only/except:      # Условия выполнения
rules:            # Продвинутые условия (новее only/except)
when:             # Когда выполнять (on_success, on_failure, always, manual)
allow_failure:    # Разрешить провал
retry:            # Количество повторов
timeout:          # Таймаут
tags:             # Теги runner'ов
needs:            # DAG - зависимости между jobs
parallel:         # Параллельное выполнение
```

**Предопределенные переменные:**
```bash
# Git
$CI_COMMIT_SHA              # Короткий SHA коммита
$CI_COMMIT_REF_NAME         # Имя ветки/тега
$CI_COMMIT_MESSAGE          # Сообщение коммита
$CI_COMMIT_TAG              # Тег (если есть)

# Project
$CI_PROJECT_ID              # ID проекта
$CI_PROJECT_NAME            # Имя проекта
$CI_PROJECT_PATH            # Путь проекта
$CI_PROJECT_URL             # URL проекта

# Pipeline
$CI_PIPELINE_ID             # ID пайплайна
$CI_PIPELINE_IID            # Внутренний ID
$CI_PIPELINE_SOURCE         # Источник (push, merge_request, schedule)

# Job
$CI_JOB_ID                  # ID job
$CI_JOB_NAME                # Имя job
$CI_JOB_STAGE               # Стадия
$CI_JOB_TOKEN               # Токен для GitLab API

# Runner
$CI_RUNNER_DESCRIPTION      # Описание runner
$CI_RUNNER_TAGS             # Теги runner

# Environment
$CI_ENVIRONMENT_NAME        # Имя окружения
$CI_ENVIRONMENT_URL         # URL окружения

# Registry
$CI_REGISTRY                # URL registry
$CI_REGISTRY_IMAGE          # Полный путь к image
$CI_REGISTRY_USER           # Username для registry
$CI_REGISTRY_PASSWORD       # Password для registry
```

**Rules (условия выполнения):**
```yaml
rules:
  # Только для main ветки
  - if: '$CI_COMMIT_REF_NAME == "main"'
  
  # Для merge requests
  - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  
  # Для тегов
  - if: '$CI_COMMIT_TAG'
  
  # Для scheduled pipelines
  - if: '$CI_PIPELINE_SOURCE == "schedule"'
  
  # С изменениями в определенных файлах
  - changes:
      - src/**/*
      - tests/**/*
  
  # Комбинированные условия
  - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push"'
    when: always
  - when: never
```

**Artifacts:**
```yaml
artifacts:
  name: "$CI_JOB_NAME-$CI_COMMIT_REF_NAME"
  paths:
    - dist/
    - reports/
  expose_as: 'Build artifacts'
  expire_in: 1 week
  when: on_success  # on_failure, always
  reports:
    junit: reports/junit.xml
    coverage_report:
      coverage_format: cobertura
      path: coverage/cobertura.xml
```

**Cache vs Artifacts:**
```
Cache:
✓ Для зависимостей (node_modules, vendor)
✓ Ускорение между пайплайнами
✓ Не гарантирован (может быть очищен)
✓ По умолчанию shared между branches

Artifacts:
✓ Для результатов сборки (dist, binaries)
✓ Передача между jobs
✓ Гарантированы
✓ Доступны для скачивания
```

**Extends и Templates:**
```yaml
# Базовый template
.deploy_template:
  script:
    - echo "Deploying..."
  only:
    - main

# Использование
deploy_staging:
  extends: .deploy_template
  variables:
    ENVIRONMENT: staging

deploy_production:
  extends: .deploy_template
  variables:
    ENVIRONMENT: production
  when: manual
```

### 💻 Задание

Создай базовый CI/CD пайплайн:

1. **Создай .gitlab-ci.yml:**
   ```yaml
   # Глобальные настройки
   image: node:16-alpine
   
   variables:
     npm_config_cache: "$CI_PROJECT_DIR/.npm"
     CYPRESS_CACHE_FOLDER: "$CI_PROJECT_DIR/.cypress"
   
   # Кеш для ускорения
   cache:
     key: ${CI_COMMIT_REF_SLUG}
     paths:
       - .npm
       - node_modules/
   
   # Стадии
   stages:
     - install
     - lint
     - test
     - build
     - deploy
   
   # Установка зависимостей
   install_dependencies:
     stage: install
     script:
       - npm ci
     artifacts:
       paths:
         - node_modules/
       expire_in: 1 hour
   
   # Линтинг
   lint:
     stage: lint
     script:
       - npm run lint
     needs: ["install_dependencies"]
   
   # Тесты
   test:
     stage: test
     script:
       - npm run test
       - npm run coverage
     coverage: '/Statements\s*:\s*(\d+\.\d+)%/'
     artifacts:
       reports:
         junit: reports/junit.xml
         coverage_report:
           coverage_format: cobertura
           path: coverage/cobertura.xml
       paths:
         - coverage/
     needs: ["install_dependencies"]
   
   # Сборка
   build:
     stage: build
     script:
       - npm run build
     artifacts:
       paths:
         - dist/
       expire_in: 1 week
     needs: ["lint", "test"]
     only:
       - main
       - merge_requests
   
   # Деплой в staging
   deploy_staging:
     stage: deploy
     script:
       - echo "Deploying to staging..."
       - echo "Application URL: https://staging.example.com"
     environment:
       name: staging
       url: https://staging.example.com
     needs: ["build"]
     only:
       - main
   
   # Деплой в production
   deploy_production:
     stage: deploy
     script:
       - echo "Deploying to production..."
       - echo "Application URL: https://example.com"
     environment:
       name: production
       url: https://example.com
     needs: ["build"]
     when: manual
     only:
       - main
   ```

2. **Добавь скрипты в package.json:**
   ```json
   {
     "scripts": {
       "lint": "eslint src/",
       "test": "jest --coverage",
       "coverage": "jest --coverage --coverageReporters=cobertura",
       "build": "webpack --mode production"
     }
   }
   ```

3. **Закоммить и запушить:**
   ```bash
   git add .gitlab-ci.yml package.json
   git commit -m "ci: add basic CI/CD pipeline"
   git push origin main
   ```

4. **Проверь пайплайн:**
   - Открой CI/CD > Pipelines
   - Просмотри logs каждого job
   - Проверь artifacts
   - Запусти manual deploy

### 🚀 Бонус (новое)

**1. Используй include для модульности:**
```yaml
# .gitlab-ci.yml
include:
  - local: '.gitlab/ci/templates.yml'
  - local: '.gitlab/ci/build.yml'
  - local: '.gitlab/ci/test.yml'
  - local: '.gitlab/ci/deploy.yml'

# Или из другого проекта
include:
  - project: 'group/ci-templates'
    ref: main
    file: '/templates/node.yml'

# Или из URL
include:
  - remote: 'https://example.com/ci-template.yml'
```

**2. Настрой DAG (needs) для параллелизма:**
```yaml
stages:
  - build
  - test
  - deploy

build_frontend:
  stage: build
  script: npm run build:frontend

build_backend:
  stage: build
  script: npm run build:backend

test_frontend:
  stage: test
  needs: ["build_frontend"]  # Не ждет build_backend
  script: npm run test:frontend

test_backend:
  stage: test
  needs: ["build_backend"]  # Не ждет build_frontend
  script: npm run test:backend

deploy:
  stage: deploy
  needs: ["test_frontend", "test_backend"]
  script: ./deploy.sh
```

**3. Динамические child pipelines:**
```yaml
generate_config:
  stage: build
  script:
    - ./generate-pipeline-config.sh > generated-config.yml
  artifacts:
    paths:
      - generated-config.yml

trigger_child:
  stage: deploy
  trigger:
    include:
      - artifact: generated-config.yml
        job: generate_config
    strategy: depend
```

**4. Используй Multi-project pipelines:**
```yaml
trigger_downstream:
  stage: deploy
  trigger:
    project: group/downstream-project
    branch: main
    strategy: depend
  only:
    - main
```

---

## Модуль 4: GitLab Runner и Docker (30 минут)

### 🎯 Напоминалка

**GitLab Runner типы:**
```
Shared Runners:    # Общие для всего GitLab instance
Group Runners:     # Для группы проектов
Specific Runners:  # Для конкретного проекта
```

**Executor типы:**
```
shell:      # Запуск команд напрямую в shell
docker:     # В Docker контейнерах (рекомендуется)
kubernetes: # В Kubernetes pods
docker-machine: # Автоскейлинг Docker runners
virtualbox: # В VM VirtualBox
parallels:  # В VM Parallels
ssh:        # Через SSH на удаленной машине
```

**Установка GitLab Runner:**
```bash
# Linux
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
sudo apt-get install gitlab-runner

# macOS
brew install gitlab-runner

# Docker
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

**Регистрация Runner:**
```bash
# Интерактивная регистрация
sudo gitlab-runner register

# Или с параметрами
sudo gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --registration-token "PROJECT_TOKEN" \
  --executor "docker" \
  --docker-image alpine:latest \
  --description "docker-runner" \
  --tag-list "docker,linux" \
  --run-untagged="true" \
  --locked="false"
```

**Управление Runner:**
```bash
# Запуск
sudo gitlab-runner start

# Остановка
sudo gitlab-runner stop

# Статус
sudo gitlab-runner status

# Список runners
sudo gitlab-runner list

# Отмена регистрации
sudo gitlab-runner unregister --url https://gitlab.com/ --token TOKEN

# Проверка конфига
sudo gitlab-runner verify
```

**Конфигурация config.toml:**
```toml
concurrent = 4  # Количество одновременных jobs
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "docker-runner"
  url = "https://gitlab.com/"
  token = "TOKEN"
  executor = "docker"
  
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = false
    disable_cache = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
    shm_size = 0
    
  [runners.cache]
    Type = "s3"
    Shared = true
    [runners.cache.s3]
      ServerAddress = "s3.amazonaws.com"
      BucketName = "gitlab-cache"
      BucketLocation = "us-east-1"
```

**Docker-in-Docker (DinD):**
```yaml
build_image:
  image: docker:24
  services:
    - docker:24-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
    DOCKER_HOST: tcp://docker:2376
    DOCKER_TLS_VERIFY: 1
    DOCKER_CERT_PATH: "$DOCKER_TLS_CERTDIR/client"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

**Kaniko (альтернатива DinD):**
```yaml
build_kaniko:
  stage: build
  image:
    name: gcr.io/kaniko-project/executor:debug
    entrypoint: [""]
  script:
    - mkdir -p /kaniko/.docker
    - echo "{\"auths\":{\"${CI_REGISTRY}\":{\"auth\":\"$(printf "%s:%s" "${CI_REGISTRY_USER}" "${CI_REGISTRY_PASSWORD}" | base64 | tr -d '\n')\"}}}" > /kaniko/.docker/config.json
    - /kaniko/executor
      --context "${CI_PROJECT_DIR}"
      --dockerfile "${CI_PROJECT_DIR}/Dockerfile"
      --destination "${CI_REGISTRY_IMAGE}:${CI_COMMIT_TAG}"
```

**GitLab Container Registry:**
```bash
# Логин
docker login registry.gitlab.com -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD

# Тегирование
docker tag myapp:latest registry.gitlab.com/username/project:latest

# Push
docker push registry.gitlab.com/username/project:latest

# Pull
docker pull registry.gitlab.com/username/project:latest
```

**Оптимизация Docker образов:**
```dockerfile
# Multi-stage build
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 💻 Задание

Настрой Docker-based CI/CD:

1. **Установи GitLab Runner локально:**
   ```bash
   # Через Docker (рекомендуется)
   docker run -d --name gitlab-runner --restart always \
     -v /srv/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:latest
   ```

2. **Зарегистрируй Runner:**
   - Получи регистрационный токен: Settings > CI/CD > Runners
   ```bash
   docker exec -it gitlab-runner gitlab-runner register \
     --non-interactive \
     --url "https://gitlab.com/" \
     --registration-token "YOUR_TOKEN" \
     --executor "docker" \
     --docker-image "alpine:latest" \
     --description "my-docker-runner" \
     --tag-list "docker,local" \
     --run-untagged="true"
   ```

3. **Создай Dockerfile:**
   ```dockerfile
   # Dockerfile
   FROM node:16-alpine AS builder
   
   WORKDIR /app
   
   # Копируем зависимости
   COPY package*.json ./
   RUN npm ci --only=production
   
   # Копируем исходники
   COPY . .
   
   # Собираем приложение
   RUN npm run build
   
   # Production образ
   FROM nginx:alpine
   
   # Копируем собранное приложение
   COPY --from=builder /app/dist /usr/share/nginx/html
   
   # Копируем nginx конфиг
   COPY nginx.conf /etc/nginx/nginx.conf
   
   EXPOSE 80
   
   HEALTHCHECK --interval=30s --timeout=3s \
     CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1
   
   CMD ["nginx", "-g", "daemon off;"]
   ```

4. **Создай nginx.conf:**
   ```nginx
   events {
     worker_connections 1024;
   }
   
   http {
     include /etc/nginx/mime.types;
     default_type application/octet-stream;
     
     server {
       listen 80;
       server_name _;
       
       root /usr/share/nginx/html;
       index index.html;
       
       location / {
         try_files $uri $uri/ /index.html;
       }
       
       gzip on;
       gzip_types text/plain text/css application/json application/javascript;
     }
   }
   ```

5. **Обнови .gitlab-ci.yml для Docker:**
   ```yaml
   stages:
     - build
     - test
     - release
     - deploy
   
   variables:
     DOCKER_DRIVER: overlay2
     DOCKER_TLS_CERTDIR: "/certs"
   
   # Сборка приложения
   build:
     stage: build
     image: node:16-alpine
     script:
       - npm ci
       - npm run build
     artifacts:
       paths:
         - dist/
       expire_in: 1 hour
     tags:
       - docker
   
   # Тесты
   test:
     stage: test
     image: node:16-alpine
     script:
       - npm ci
       - npm run test
     tags:
       - docker
   
   # Сборка Docker образа
   docker_build:
     stage: release
     image: docker:24
     services:
       - docker:24-dind
     before_script:
       - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
     script:
       - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
       - docker build -t $CI_REGISTRY_IMAGE:latest .
       - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
       - docker push $CI_REGISTRY_IMAGE:latest
     tags:
       - docker
     only:
       - main
   
   # Деплой
   deploy:
     stage: deploy
     image: alpine:latest
     before_script:
       - apk add --no-cache openssh-client
     script:
       - echo "Deploying $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA"
       - echo "docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA"
       - echo "docker stop myapp || true"
       - echo "docker rm myapp || true"
       - echo "docker run -d --name myapp -p 80:80 $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA"
     environment:
       name: production
       url: https://myapp.example.com
     when: manual
     only:
       - main
     tags:
       - docker
   ```

6. **Закоммить и проверить:**
   ```bash
   git add Dockerfile nginx.conf .gitlab-ci.yml
   git commit -m "ci: add Docker build and registry push"
   git push origin main
   ```

### 🚀 Бонус (новое)

**1. Используй BuildKit для быстрой сборки:**
```yaml
docker_build_fast:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  variables:
    DOCKER_BUILDKIT: 1
  script:
    - docker build 
      --build-arg BUILDKIT_INLINE_CACHE=1
      --cache-from $CI_REGISTRY_IMAGE:latest
      -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA 
      -t $CI_REGISTRY_IMAGE:latest 
      .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
```

**2. Сканирование образов на уязвимости:**
```yaml
container_scanning:
  stage: test
  image: docker:24
  services:
    - docker:24-dind
  variables:
    DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  script:
    - docker pull $DOCKER_IMAGE
    - |
      docker run --rm \
        -v /var/run/docker.sock:/var/run/docker.sock \
        aquasec/trivy:latest image \
        --exit-code 1 \
        --severity HIGH,CRITICAL \
        $DOCKER_IMAGE
  allow_failure: true
  only:
    - main
```

**3. Multi-platform builds:**
```yaml
docker_multiplatform:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker run --privileged --rm tonistiigi/binfmt --install all
    - docker buildx create --use
  script:
    - docker buildx build 
      --platform linux/amd64,linux/arm64 
      --tag $CI_REGISTRY_IMAGE:latest 
      --push 
      .
```

---

## Модуль 5: GitLab Package Registry и Artifacts (20 минут)

### 🎯 Напоминалка

**Типы Package Registry:**
```
Container Registry  # Docker images
NPM Registry       # Node.js packages
Maven Repository   # Java packages
PyPI Repository    # Python packages
NuGet Gallery      # .NET packages
Composer Repository # PHP packages
Conan Repository   # C/C++ packages
Generic Packages   # Любые файлы
```

**Container Registry:**
```bash
# URL формат
registry.gitlab.com/<group>/<project>:<tag>
registry.gitlab.com/username/myproject:latest
registry.gitlab.com/username/myproject:v1.0.0

# Авторизация
docker login registry.gitlab.com -u <username> -p <access_token>

# В CI/CD
docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
```

**NPM Registry:**
```bash
# .npmrc в корне проекта
@scope:registry=https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/npm/
//gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/npm/:_authToken=${CI_JOB_TOKEN}

# Публикация
npm publish

# Установка
npm install @scope/package-name
```

**Generic Package Registry:**
```bash
# Upload
curl --header "PRIVATE-TOKEN: <token>" \
  --upload-file myfile.zip \
  "https://gitlab.com/api/v4/projects/<project_id>/packages/generic/my-package/1.0.0/myfile.zip"

# Download
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/<project_id>/packages/generic/my-package/1.0.0/myfile.zip" \
  -o myfile.zip
```

**Job Artifacts:**
```yaml
# Сохранение artifacts
job:
  script:
    - make build
  artifacts:
    name: "build-$CI_COMMIT_REF_NAME"
    paths:
      - dist/
      - build/
    exclude:
      - dist/**/*.map
    expire_in: 1 week
    when: on_success

# Загрузка artifacts из предыдущих jobs
deploy:
  dependencies:
    - build
  script:
    - ls dist/
```

**Artifacts типы:**
```yaml
artifacts:
  reports:
    junit: report.xml           # JUnit test reports
    cobertura: coverage.xml     # Code coverage
    terraform: plan.json        # Terraform plans
    dotenv: build.env          # Environment variables
    metrics: metrics.txt        # Custom metrics
    sast: sast-report.json     # Security scanning
    dependency_scanning: deps.json
```

**Release и Tags:**
```yaml
release_job:
  stage: release
  image: registry.gitlab.com/gitlab-org/release-cli:latest
  rules:
    - if: $CI_COMMIT_TAG
  script:
    - echo "Running release job"
  release:
    tag_name: '$CI_COMMIT_TAG'
    description: './CHANGELOG.md'
    assets:
      links:
        - name: 'Binary'
          url: 'https://example.com/download/v${CI_COMMIT_TAG}'
```

### 💻 Задание

Настрой Package Registry и Releases:

1. **Создай NPM пакет:**
   ```bash
   # Обнови package.json
   cat > package.json <<EOF
   {
     "name": "@${CI_PROJECT_NAMESPACE}/${CI_PROJECT_NAME}",
     "version": "1.0.0",
     "description": "DevOps refresh package",
     "main": "src/index.js",
     "scripts": {
       "build": "webpack --mode production",
       "test": "jest"
     },
     "publishConfig": {
       "@${CI_PROJECT_NAMESPACE}:registry": "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/npm/"
     }
   }
   EOF
   ```

2. **Добавь в CI/CD публикацию пакета:**
   ```yaml
   publish_npm:
     stage: release
     image: node:16
     script:
       - echo "@${CI_PROJECT_NAMESPACE}:registry=${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/npm/" > .npmrc
       - echo "${CI_API_V4_URL#https?}/projects/${CI_PROJECT_ID}/packages/npm/:_authToken=${CI_JOB_TOKEN}" >> .npmrc
       - npm publish
     only:
       - tags
   ```

3. **Создай Generic Package:**
   ```yaml
   upload_artifacts:
     stage: release
     image: curlimages/curl:latest
     script:
       - 'curl --header "JOB-TOKEN: $CI_JOB_TOKEN" 
         --upload-file dist/app.tar.gz 
         "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/generic/myapp/${CI_COMMIT_TAG}/app.tar.gz"'
     only:
       - tags
   ```

4. **Создай GitLab Release:**
   ```yaml
   release_job:
     stage: release
     image: registry.gitlab.com/gitlab-org/release-cli:latest
     rules:
       - if: $CI_COMMIT_TAG
     script:
       - echo "Creating release for $CI_COMMIT_TAG"
     release:
       tag_name: '$CI_COMMIT_TAG'
       name: 'Release $CI_COMMIT_TAG'
       description: |
         ## What's Changed
         - Feature 1
         - Feature 2
         - Bug fixes
         
         **Full Changelog**: https://gitlab.com/$CI_PROJECT_PATH/-/compare/$CI_COMMIT_BEFORE_SHA...$CI_COMMIT_SHA
       assets:
         links:
           - name: 'Application Binary'
             url: '${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/generic/myapp/${CI_COMMIT_TAG}/app.tar.gz'
           - name: 'Docker Image'
             url: '${CI_REGISTRY_IMAGE}:${CI_COMMIT_TAG}'
   ```

5. **Создай тег и проверь:**
   ```bash
   git tag -a v1.0.0 -m "Release version 1.0.0"
   git push origin v1.0.0
   
   # Проверь в GitLab:
   # - Deploy > Package Registry
   # - Deploy > Releases
   ```

### 🚀 Бонус (новое)

**1. Автоматическое создание CHANGELOG:**
```yaml
generate_changelog:
  stage: release
  image: node:16
  before_script:
    - npm install -g conventional-changelog-cli
  script:
    - conventional-changelog -p angular -i CHANGELOG.md -s -r 0
  artifacts:
    paths:
      - CHANGELOG.md
  only:
    - tags

release_with_changelog:
  stage: release
  image: registry.gitlab.com/gitlab-org/release-cli:latest
  needs: ["generate_changelog"]
  script:
    - echo "Release with generated changelog"
  release:
    tag_name: '$CI_COMMIT_TAG'
    description: './CHANGELOG.md'
  only:
    - tags
```

**2. Semantic versioning автоматизация:**
```yaml
semantic_release:
  stage: release
  image: node:16
  before_script:
    - npm install -g semantic-release @semantic-release/gitlab
  script:
    - semantic-release
  only:
    - main
```

---

## Модуль 6: Environments и Deployments (25 минут)

### 🎯 Напоминалка

**Environments концепция:**
```
Environments = Окружения развертывания
- Отслеживание деплоев
- История развертываний
- Rollback возможности
- Просмотр текущей версии
```

**Environment определение:**
```yaml
deploy_staging:
  stage: deploy
  script:
    - ./deploy.sh staging
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop_staging
  only:
    - main

stop_staging:
  stage: deploy
  script:
    - ./cleanup.sh staging
  environment:
    name: staging
    action: stop
  when: manual
```

**Dynamic Environments:**
```yaml
review_app:
  stage: deploy
  script:
    - ./deploy-review.sh
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: https://$CI_COMMIT_REF_SLUG.review.example.com
    on_stop: stop_review
    auto_stop_in: 1 day
  only:
    - merge_requests

stop_review:
  stage: deploy
  script:
    - ./cleanup-review.sh
  environment:
    name: review/$CI_COMMIT_REF_NAME
    action: stop
  when: manual
  only:
    - merge_requests
```

**Deployment стратегии:**
```yaml
# Rolling Deployment
deploy:
  script:
    - for server in $SERVERS; do
        ssh $server "docker pull $IMAGE && docker restart app";
      done

# Blue-Green Deployment
deploy_green:
  script:
    - ./deploy-to-green.sh
    - ./run-tests-on-green.sh
    - ./switch-traffic-to-green.sh

# Canary Deployment
deploy_canary:
  script:
    - ./deploy-canary.sh  # 10% traffic
    - sleep 300  # Monitor
    - ./deploy-full.sh  # 100% traffic
```

**Protected Environments:**
```
Settings > CI/CD > Protected Environments
- Только Maintainers могут деплоить
- Approval required
- Deployment branch restrictions
```

**Kubernetes Integration:**
```yaml
deploy_k8s:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context $KUBE_CONTEXT
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - kubectl rollout status deployment/myapp
  environment:
    name: production
    url: https://example.com
    kubernetes:
      namespace: production
```

### 💻 Задание

Настрой полноценные Environments:

1. **Создай deployment скрипты:**
   ```bash
   # deploy.sh
   #!/bin/bash
   ENV=$1
   
   echo "Deploying to $ENV environment..."
   
   case $ENV in
     staging)
       SERVER="staging.example.com"
       ;;
     production)
       SERVER="example.com"
       ;;
     *)
       echo "Unknown environment"
       exit 1
       ;;
   esac
   
   echo "Deploying to $SERVER"
   echo "Image: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA"
   
   # Симуляция деплоя
   sleep 5
   echo "Deployment successful!"
   ```

2. **Создай multi-environment pipeline:**
   ```yaml
   stages:
     - build
     - test
     - review
     - staging
     - production
   
   build:
     stage: build
     script:
       - echo "Building application..."
       - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
     only:
       - main
       - merge_requests
   
   test:
     stage: test
     script:
       - echo "Running tests..."
       - npm test
   
   # Review App для MR
   review:
     stage: review
     script:
       - echo "Deploying review app for $CI_COMMIT_REF_NAME"
       - echo "URL: https://$CI_COMMIT_REF_SLUG.review.example.com"
     environment:
       name: review/$CI_COMMIT_REF_NAME
       url: https://$CI_COMMIT_REF_SLUG.review.example.com
       on_stop: stop_review
       auto_stop_in: 1 day
     only:
       - merge_requests
   
   stop_review:
     stage: review
     script:
       - echo "Stopping review app"
     environment:
       name: review/$CI_COMMIT_REF_NAME
       action: stop
     when: manual
     only:
       - merge_requests
   
   # Staging автоматически
   deploy_staging:
     stage: staging
     script:
       - chmod +x deploy.sh
       - ./deploy.sh staging
     environment:
       name: staging
       url: https://staging.example.com
       deployment_tier: staging
     only:
       - main
   
   # Production с approval
   deploy_production:
     stage: production
     script:
       - chmod +x deploy.sh
       - ./deploy.sh production
     environment:
       name: production
       url: https://example.com
       deployment_tier: production
     when: manual
     only:
       - main
   
   # Rollback
   rollback_production:
     stage: production
     script:
       - echo "Rolling back to previous version"
       - echo "Previous: $CI_COMMIT_BEFORE_SHA"
     environment:
       name: production
       action: rollback
     when: manual
     only:
       - main
   ```

3. **Добавь переменные окружения:**
   - Settings > CI/CD > Variables
   ```
   STAGING_SERVER = staging.example.com
   PRODUCTION_SERVER = example.com
   DEPLOY_TOKEN = secret_token
   ```

4. **Настрой Protected Environment:**
   - Settings > CI/CD > Protected Environments
   - Add protected environment: `production`
   - Allowed to deploy: Maintainers
   - Approval required: Yes
   - Approvers: 1-2 maintainers

5. **Проверь деплоймент:**
   ```bash
   git add deploy.sh .gitlab-ci.yml
   git commit -m "feat: add multi-environment deployment"
   git push origin main
   
   # Проверь:
   # - Deployments > Environments
   # - Просмотри историю
   # - Попробуй rollback
   ```

### 🚀 Бонус (новое)

**1. Feature Flags integration:**
```yaml
variables:
  FEATURE_FLAGS_URL: "https://unleash.example.com"

deploy_with_flags:
  stage: deploy
  script:
    - |
      curl -X POST $FEATURE_FLAGS_URL/api/admin/features/new-feature/toggle/on \
        -H "Authorization: $UNLEASH_TOKEN"
    - ./deploy.sh
  environment:
    name: production
```

**2. Deployment notifications:**
```yaml
.notify_template: &notify
  image: curlimages/curl:latest
  script:
    - |
      curl -X POST $SLACK_WEBHOOK \
        -H 'Content-Type: application/json' \
        -d '{
          "text": "🚀 Deployment to '"$CI_ENVIRONMENT_NAME"'",
          "blocks": [{
            "type": "section",
            "text": {
              "type": "mrkdwn",
              "text": "*Project:* '"$CI_PROJECT_NAME"'\n*Environment:* '"$CI_ENVIRONMENT_NAME"'\n*Commit:* '"$CI_COMMIT_SHORT_SHA"'\n*Author:* '"$GITLAB_USER_NAME"'"
            }
          }]
        }'

notify_deploy:
  <<: *notify
  stage: .post
  environment:
    name: production
  when: on_success
  only:
    - main
```

**3. Smoke tests после деплоя:**
```yaml
deploy_production:
  stage: deploy
  script:
    - ./deploy.sh production
  environment:
    name: production
    url: https://example.com

smoke_tests:
  stage: .post
  image: curlimages/curl:latest
  script:
    - |
      echo "Running smoke tests..."
      curl -f https://example.com/health || exit 1
      curl -f https://example.com/api/status || exit 1
      echo "Smoke tests passed!"
  environment:
    name: production
    action: verify
  when: on_success
  only:
    - main
```

---

## Модуль 7: GitLab Security и SAST/DAST (25 минут)

### 🎯 Напоминалка

**GitLab Security сканирования:**
```
SAST: Static Application Security Testing
  - Анализ кода на уязвимости
  - Запуск без приложения
  
DAST: Dynamic Application Security Testing
  - Тестирование работающего приложения
  - Поиск runtime уязвимостей

Dependency Scanning:
  - Проверка зависимостей
  - Known vulnerabilities

Container Scanning:
  - Сканирование Docker образов
  - OS packages vulnerabilities

Secret Detection:
  - Поиск credentials в коде
  - API keys, tokens, passwords
```

**SAST базовая настройка:**
```yaml
include:
  - template: Security/SAST.gitlab-ci.yml

variables:
  SAST_EXCLUDED_PATHS: "spec, test, tests, tmp"
  SAST_EXCLUDED_ANALYZERS: "eslint"
```

**DAST базовая настройка:**
```yaml
include:
  - template: Security/DAST.gitlab-ci.yml

variables:
  DAST_WEBSITE: https://staging.example.com
  DAST_FULL_SCAN_ENABLED: "true"
```

**Dependency Scanning:**
```yaml
include:
  - template: Security/Dependency-Scanning.gitlab-ci.yml

variables:
  DS_EXCLUDED_PATHS: "spec, test, tests"
```

**Container Scanning:**
```yaml
include:
  - template: Security/Container-Scanning.gitlab-ci.yml

variables:
  CS_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  CS_SEVERITY_THRESHOLD: "high"
```

**Secret Detection:**
```yaml
include:
  - template: Security/Secret-Detection.gitlab-ci.yml

variables:
  SECRET_DETECTION_EXCLUDED_PATHS: "tests/"
```

**Custom Security Scanning:**
```yaml
trivy_scan:
  stage: test
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  script:
    - trivy filesystem --exit-code 1 --severity HIGH,CRITICAL .
  artifacts:
    reports:
      container_scanning: trivy-report.json
```

**Security Dashboard:**
```
Security & Compliance > Vulnerability Report
- Все найденные уязвимости
- Severity levels
- Status (Detected, Confirmed, Dismissed, Resolved)
- Assign to users
```

**Merge Request Security Widget:**
```
Автоматически показывает:
- Новые уязвимости
- Resolved уязвимости
- Security approvals
```

### 💻 Задание

Настрой комплексный security scanning:

1. **Создай базовый security pipeline:**
   ```yaml
   include:
     - template: Security/SAST.gitlab-ci.yml
     - template: Security/Dependency-Scanning.gitlab-ci.yml
     - template: Security/Secret-Detection.gitlab-ci.yml
   
   stages:
     - build
     - test
     - security
     - deploy
   
   variables:
     SAST_EXCLUDED_PATHS: "spec, test, tests, tmp, vendor"
     DS_EXCLUDED_PATHS: "spec, test, tests"
     SECRET_DETECTION_EXCLUDED_PATHS: "tests/, docs/"
   
   # Сборка образа
   build_image:
     stage: build
     image: docker:24
     services:
       - docker:24-dind
     script:
       - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
       - docker save $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA > image.tar
     artifacts:
       paths:
         - image.tar
       expire_in: 1 day
     only:
       - main
       - merge_requests
   
   # Container scanning
   container_scanning:
     stage: security
     image:
       name: aquasec/trivy:latest
       entrypoint: [""]
     dependencies:
       - build_image
     before_script:
       - docker load < image.tar
     script:
       - trivy image 
         --format json 
         --output trivy-report.json 
         --exit-code 0
         --severity HIGH,CRITICAL
         $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
     artifacts:
       reports:
         container_scanning: trivy-report.json
     allow_failure: true
     only:
       - main
       - merge_requests
   ```

2. **Добавь custom security checks:**
   ```yaml
   # Проверка на hardcoded secrets
   check_secrets:
     stage: security
     image: python:3.9
     before_script:
       - pip install detect-secrets
     script:
       - detect-secrets scan --all-files --force-use-all-plugins > .secrets.baseline
       - |
         if [ $(jq '.results | length' .secrets.baseline) -gt 0 ]; then
           echo "⚠️  Potential secrets found!"
           jq '.results' .secrets.baseline
           exit 1
         fi
     artifacts:
       reports:
         sast: .secrets.baseline
     allow_failure: true
   
   # License compliance
   license_scanning:
     stage: security
     image: node:16
     script:
       - npm install -g license-checker
       - license-checker --json > licenses.json
       - |
         # Проверка на запрещенные лицензии (GPL, AGPL и т.д.)
         FORBIDDEN_LICENSES="GPL AGPL"
         for license in $FORBIDDEN_LICENSES; do
           if grep -q "$license" licenses.json; then
             echo "❌ Forbidden license found: $license"
             exit 1
           fi
         done
     artifacts:
       paths:
         - licenses.json
     allow_failure: true
   ```

3. **Создай security policy:**
   ```yaml
   # .gitlab/security-policies/scan-execution-policy.yml
   type: scan_execution_policy
   name: Required Security Scans
   description: Run security scans on all merge requests
   enabled: true
   rules:
     - type: pipeline
       branches:
         - main
         - develop
   actions:
     - scan: sast
     - scan: secret_detection
     - scan: dependency_scanning
   ```

4. **Настрой MR approval rules:**
   - Settings > Merge requests > Merge request approvals
   - Add approval rule: "Security"
   - Approvals required: 1
   - Users: Security team

5. **Тест security pipeline:**
   ```bash
   # Создай тестовый файл с "уязвимостью"
   echo "const apiKey = 'sk-1234567890abcdef';" > src/config.js
   
   git add .
   git commit -m "test: add security scanning"
   git push origin main
   
   # Проверь Security Dashboard
   ```

### 🚀 Бонус (новое)

**1. GitLab Advisory Database:**
```yaml
# Автоматическое создание Security Advisories
create_advisory:
  stage: security
  script:
    - |
      curl --request POST \
        --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
        --header "Content-Type: application/json" \
        --data '{
          "title": "CVE-2024-1234",
          "description": "Security vulnerability description",
          "severity": "high",
          "solution": "Update to version 1.2.3"
        }' \
        "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/vulnerabilities"
  when: manual
```

**2. Integration с внешними security tools:**
```yaml
# SonarQube integration
sonarqube_scan:
  stage: security
  image: sonarsource/sonar-scanner-cli:latest
  variables:
    SONAR_HOST_URL: "https://sonarqube.example.com"
  script:
    - sonar-scanner
      -Dsonar.projectKey=$CI_PROJECT_NAME
      -Dsonar.sources=src
      -Dsonar.host.url=$SONAR_HOST_URL
      -Dsonar.login=$SONAR_TOKEN
  allow_failure: true

# Snyk integration
snyk_scan:
  stage: security
  image: snyk/snyk:node
  script:
    - snyk auth $SNYK_TOKEN
    - snyk test --severity-threshold=high
    - snyk monitor
  allow_failure: true
```

**3. Automated vulnerability patching:**
```yaml
auto_patch:
  stage: security
  image: node:16
  script:
    - npm audit fix
    - |
      if [[ $(git status --porcelain) ]]; then
        git config user.email "ci@example.com"
        git config user.name "GitLab CI"
        git checkout -b security/auto-patch-$CI_COMMIT_SHORT_SHA
        git add package*.json
        git commit -m "security: auto-patch vulnerabilities"
        git push origin security/auto-patch-$CI_COMMIT_SHORT_SHA
        
        # Создать MR
        glab mr create \
          --title "🔒 Security: Auto-patch vulnerabilities" \
          --description "Automated security patches from npm audit" \
          --label "security,automated"
      fi
  only:
    - schedules
```

---

## Модуль 8: GitLab Variables и Secrets Management (20 минут)

### 🎯 Напоминалка

**Типы переменных:**
```
CI/CD Variables:
  - Project level: для конкретного проекта
  - Group level: для всех проектов группы
  - Instance level: для всего GitLab (Admin)

Variable types:
  - Variable: обычная переменная
  - File: сохраняется как временный файл
```

**Защита переменных:**
```yaml
Settings > CI/CD > Variables

Options:
  ✅ Protect variable (только protected branches/tags)
  ✅ Mask variable (скрыть в логах)
  ✅ Expand variable reference (интерполяция)
  
Environment scope:
  - * (все)
  - production
  - staging
  - review/*
```

**Использование переменных:**
```yaml
# В script
script:
  - echo $MY_VARIABLE
  - echo $CI_COMMIT_SHA

# Как файл
variables:
  CONFIG_FILE:
    value: |
      {
        "key": "value"
      }
    description: "Configuration file"

script:
  - cat $CONFIG_FILE
```

**HashiCorp Vault Integration:**
```yaml
variables:
  VAULT_SERVER_URL: "https://vault.example.com"
  VAULT_AUTH_ROLE: "gitlab-ci"

secrets:
  DATABASE_PASSWORD:
    vault:
      engine:
        name: kv-v2
        path: gitlab
      path: production/db
      field: password

script:
  - echo "Using password from Vault: $DATABASE_PASSWORD"
```

**AWS Secrets Manager:**
```yaml
fetch_secrets:
  image: amazon/aws-cli:latest
  before_script:
    - export AWS_DEFAULT_REGION=us-east-1
  script:
    - |
      export DB_PASSWORD=$(aws secretsmanager get-secret-value \
        --secret-id production/db/password \
        --query SecretString \
        --output text)
    - echo "Using secret from AWS"
```

**Best Practices:**
```yaml
✅ DO:
  - Используй protected + masked для production secrets
  - Используй Group variables для общих секретов
  - Используй Environment scope для разных окружений
  - Храни secrets в Vault/AWS Secrets Manager
  - Регулярно ротируй credentials

❌ DON'T:
  - Не коммить secrets в код
  - Не логируй sensitive данные
  - Не используй слабые пароли
  - Не шарь токены между проектами без необходимости
```

### 💻 Задание

Настрой безопасное управление секретами:

1. **Создай переменные на разных уровнях:**
   ```bash
   # Project variables (Settings > CI/CD > Variables)
   DATABASE_URL_DEV = postgresql://localhost:5432/dev
   DATABASE_URL_PROD = postgresql://prod.example.com:5432/prod (Protected + Masked)
   
   API_KEY = secret_key_12345 (Masked)
   
   # Group variables (Group > Settings > CI/CD)
   DOCKER_REGISTRY = registry.example.com
   NOTIFICATION_WEBHOOK = https://hooks.slack.com/... (Masked)
   ```

2. **Используй Environment scope:**
   ```yaml
   # Settings > CI/CD > Variables
   
   DEPLOY_TOKEN:
     - Value: staging_token_xyz
     - Environment scope: staging
     - Protect: Yes
     - Mask: Yes
   
   DEPLOY_TOKEN:
     - Value: production_token_abc
     - Environment scope: production
     - Protect: Yes
     - Mask: Yes
   ```

3. **Создай pipeline с разными секретами:**
   ```yaml
   variables:
     # Публичные переменные
     APP_NAME: "devops-refresh"
     NODE_ENV: "production"
   
   stages:
     - build
     - test
     - deploy
   
   build:
     stage: build
     script:
       - echo "Building $APP_NAME"
       - echo "Registry: $DOCKER_REGISTRY"
       - docker build -t $DOCKER_REGISTRY/$APP_NAME:$CI_COMMIT_SHA .
   
   test:
     stage: test
     variables:
       DATABASE_URL: $DATABASE_URL_DEV
     script:
       - echo "Running tests with dev database"
       - npm test
   
   deploy_staging:
     stage: deploy
     environment:
       name: staging
     script:
       - echo "Deploying to staging"
       - echo "Using token: ${DEPLOY_TOKEN:0:4}..." # Показываем только первые 4 символа
       - echo "Database: $DATABASE_URL_DEV"
     only:
       - main
   
   deploy_production:
     stage: deploy
     environment:
       name: production
     script:
       - echo "Deploying to production"
       - echo "Using token: ${DEPLOY_TOKEN:0:4}..."
       - echo "Database: $DATABASE_URL_PROD"
     when: manual
     only:
       - main
   ```

4. **Создай script для ротации секретов:**
   ```bash
   # rotate-secrets.sh
   #!/bin/bash
   
   PROJECT_ID="your-project-id"
   GITLAB_TOKEN="your-token"
   
   # Генерация нового токена
   NEW_TOKEN=$(openssl rand -hex 32)
   
   # Обновление в GitLab
   curl --request PUT \
     --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
     --form "key=DEPLOY_TOKEN" \
     --form "value=$NEW_TOKEN" \
     --form "protected=true" \
     --form "masked=true" \
     "https://gitlab.com/api/v4/projects/$PROJECT_ID/variables/DEPLOY_TOKEN"
   
   echo "Secret rotated successfully"
   ```

5. **Добавь scheduled rotation:**
   ```yaml
   rotate_secrets:
     stage: maintenance
     script:
       - chmod +x rotate-secrets.sh
       - ./rotate-secrets.sh
     only:
       - schedules
   ```

### 🚀 Бонус (новое)

**1. External Secrets Operator integration:**
```yaml
# Использование переменных из внешних хранилищ
fetch_from_vault:
  image: vault:latest
  before_script:
    - export VAULT_ADDR=$VAULT_SERVER_URL
    - export VAULT_TOKEN=$VAULT_CI_TOKEN
  script:
    - |
      # Получение секрета из Vault
      DB_PASSWORD=$(vault kv get -field=password secret/production/database)
      
      # Использование в деплойменте
      kubectl create secret generic db-credentials \
        --from-literal=password=$DB_PASSWORD \
        --dry-run=client -o yaml | kubectl apply -f -
```

**2. Dynamic credentials generation:**
```yaml
generate_temp_credentials:
  stage: deploy
  script:
    - |
      # Генерация временных AWS credentials
      TEMP_CREDS=$(aws sts assume-role \
        --role-arn $AWS_ROLE_ARN \
        --role-session-name gitlab-ci-$CI_PIPELINE_ID)
      
      export AWS_ACCESS_KEY_ID=$(echo $TEMP_CREDS | jq -r '.Credentials.AccessKeyId')
      export AWS_SECRET_ACCESS_KEY=$(echo $TEMP_CREDS | jq -r '.Credentials.SecretAccessKey')
      export AWS_SESSION_TOKEN=$(echo $TEMP_CREDS | jq -r '.Credentials.SessionToken')
      
      # Использование временных credentials
      aws s3 cp dist/ s3://my-bucket/ --recursive
  environment:
    name: production
```

**3. Secrets scanning в pipeline:**
```yaml
detect_leaked_secrets:
  stage: security
  image: trufflesecurity/trufflehog:latest
  script:
    - trufflehog filesystem . --json > secrets-scan.json
    - |
      if [ -s secrets-scan.json ]; then
        echo "⚠️  Potential secrets detected!"
        cat secrets-scan.json
        exit 1
      fi
  artifacts:
    reports:
      secret_detection: secrets-scan.json
  allow_failure: false
```

---

## Модуль 9: GitLab Monitoring и Performance (25 минут)

### 🎯 Напоминалка

**GitLab встроенный мониторинг:**
```
Operations > Metrics
- Auto DevOps metrics
- Prometheus integration
- Custom dashboards
- Alerts

Operations > Incidents
- Incident management
- On-call schedules
- Escalation policies
```

**Prometheus Integration:**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'gitlab-ci'
    static_configs:
      - targets: ['localhost:9090']
        labels:
          env: 'production'
```

**Custom Metrics:**
```yaml
test_metrics:
  stage: test
  script:
    - npm run test
    - npm run test:performance > metrics.txt
  artifacts:
    reports:
      metrics: metrics.txt

# metrics.txt format
# metric_name metric_value
response_time_ms 245
requests_per_second 1500
error_rate_percent 0.5
```

**Performance Testing:**
```yaml
# k6 load testing
load_test:
  stage: test
  image: grafana/k6:latest
  script:
    - k6 run --vus 100 --duration 30s tests/load-test.js
  artifacts:
    reports:
      load_performance: load-test-report.json
```

**Browser Performance Testing:**
```yaml
performance:
  stage: test
  image: sitespeedio/sitespeed.io:latest
  script:
    - sitespeed.io https://example.com --outputFolder sitespeed-results
  artifacts:
    paths:
      - sitespeed-results/
    reports:
      browser_performance: sitespeed-results/data/browsertime.json
```

**Pipeline Efficiency Metrics:**
```yaml
# Анализ времени выполнения
analyze_pipeline:
  stage: .post
  script:
    - |
      echo "Pipeline Duration: $CI_PIPELINE_DURATION seconds"
      echo "Jobs:"
      curl --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
        "$CI_API_V4_URL/projects/$CI_PROJECT_ID/pipelines/$CI_PIPELINE_ID/jobs" \
        | jq '.[] | "\(.name): \(.duration)s"'
  when: always
```

**Дашборды Grafana:**
```yaml
# Интеграция с Grafana
deploy:
  script:
    - ./deploy.sh
  after_script:
    - |
      # Отправка метрик в Grafana
      curl -X POST $GRAFANA_URL/api/annotations \
        -H "Authorization: Bearer $GRAFANA_API_KEY" \
        -H "Content-Type: application/json" \
        -d '{
          "text": "Deployment to '"$CI_ENVIRONMENT_NAME"'",
          "tags": ["deployment", "'"$CI_ENVIRONMENT_NAME"'"],
          "time": '$(date +%s)'000
        }'
```

### 💻 Задание

Настрой полный мониторинг pipeline:

1. **Создай performance test:**
   ```javascript
   // tests/load-test.js
   import http from 'k6/http';
   import { check, sleep } from 'k6';
   
   export const options = {
     vus: 10,
     duration: '30s',
     thresholds: {
       http_req_duration: ['p(95)<500'], // 95% requests < 500ms
       http_req_failed: ['rate<0.01'],   // Error rate < 1%
     },
   };
   
   export default function () {
     const res = http.get('https://staging.example.com/health');
     
     check(res, {
       'status is 200': (r) => r.status === 200,
       'response time < 500ms': (r) => r.timings.duration < 500,
     });
     
     sleep(1);
   }
   ```

2. **Добавь monitoring в CI/CD:**
   ```yaml
   stages:
     - build
     - test
     - performance
     - deploy
     - monitor
   
   # Unit tests с coverage
   test:
     stage: test
     image: node:16
     script:
       - npm test -- --coverage
     coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
     artifacts:
       reports:
         coverage_report:
           coverage_format: cobertura
           path: coverage/cobertura-coverage.xml
   
   # Load testing
   load_test:
     stage: performance
     image: grafana/k6:latest
     script:
       - k6 run --out json=results.json tests/load-test.js
     artifacts:
       paths:
         - results.json
       reports:
         load_performance: results.json
     allow_failure: true
   
   # Browser performance
   lighthouse:
     stage: performance
     image: markhobson/node-chrome:latest
     script:
       - npm install -g lighthouse
       - lighthouse https://staging.example.com 
         --output json 
         --output-path lighthouse-report.json
         --chrome-flags="--headless --no-sandbox"
     artifacts:
       paths:
         - lighthouse-report.json
       reports:
         browser_performance: lighthouse-report.json
     allow_failure: true
   
   # Deployment
   deploy:
     stage: deploy
     script:
       - ./deploy.sh
     environment:
       name: production
       url: https://example.com
   
   # Post-deployment monitoring
   health_check:
     stage: monitor
     image: curlimages/curl:latest
     script:
       - |
         echo "Running health checks..."
         
         # Health endpoint
         response=$(curl -s -o /dev/null -w "%{http_code}" https://example.com/health)
         if [ $response -ne 200 ]; then
           echo "❌ Health check failed: $response"
           exit 1
         fi
         
         # Response time
         time=$(curl -s -w "%{time_total}" -o /dev/null https://example.com)
         echo "⏱️  Response time: ${time}s"
         
         if (( $(echo "$time > 1.0" | bc -l) )); then
           echo "⚠️  Response time too high!"
           exit 1
         fi
         
         echo "✅ All checks passed"
     retry:
       max: 2
       when: script_failure
   
   # Metrics collection
   collect_metrics:
     stage: monitor
     image: alpine:latest
     before_script:
       - apk add --no-cache curl jq bc
     script:
       - |
         echo "Collecting pipeline metrics..."
         
         # Pipeline duration
         echo "pipeline_duration_seconds $CI_PIPELINE_DURATION" > metrics.txt
         
         # Job durations
         curl --silent --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
           "$CI_API_V4_URL/projects/$CI_PROJECT_ID/pipelines/$CI_PIPELINE_ID/jobs" \
           | jq -r '.[] | "job_duration_seconds{job=\"\(.name)\"} \(.duration)"' \
           >> metrics.txt
         
         # Send to monitoring system (example)
         # cat metrics.txt | curl --data-binary @- http://pushgateway:9091/metrics/job/gitlab-ci
         
         cat metrics.txt
     artifacts:
       paths:
         - metrics.txt
     when: always
   ```

3. **Создай Grafana dashboard config:**
   ```json
   {
     "dashboard": {
       "title": "GitLab CI/CD Metrics",
       "panels": [
         {
           "title": "Pipeline Duration",
           "targets": [
             {
               "expr": "pipeline_duration_seconds"
             }
           ]
         },
         {
           "title": "Job Success Rate",
           "targets": [
             {
               "expr": "rate(gitlab_ci_pipeline_success_total[5m])"
             }
           ]
         },
         {
           "title": "Deployment Frequency",
           "targets": [
             {
               "expr": "count_over_time(deployment_event[1d])"
             }
           ]
         }
       ]
     }
   }
   ```

4. **Добавь alerting:**
   ```yaml
   # Alert на долгий pipeline
   check_pipeline_duration:
     stage: .post
     script:
       - |
         if [ $CI_PIPELINE_DURATION -gt 600 ]; then
           curl -X POST $SLACK_WEBHOOK \
             -H 'Content-Type: application/json' \
             -d '{
               "text": "⚠️ Pipeline took too long: '"$CI_PIPELINE_DURATION"'s",
               "attachments": [{
                 "color": "warning",
                 "fields": [{
                   "title": "Pipeline",
                   "value": "'"$CI_PIPELINE_URL"'"
                 }]
               }]
             }'
         fi
     when: always
   ```

5. **Проверь метрики:**
   ```bash
   git add tests/ .gitlab-ci.yml
   git commit -m "feat: add comprehensive monitoring"
   git push origin main
   
   # Проверь:
   # - CI/CD > Pipelines > Pipeline #X > Tests
   # - Operations > Metrics
   ```

### 🚀 Бонус (новое)

**1. DORA metrics tracking:**
   ```yaml
   # Deployment Frequency, Lead Time, MTTR, Change Failure Rate
   track_dora_metrics:
     stage: .post
     script:
       - |
         # Deployment Frequency
         DEPLOY_COUNT=$(curl --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
           "$CI_API_V4_URL/projects/$CI_PROJECT_ID/deployments?environment=production&per_page=100" \
           | jq '. | length')
         
         # Lead Time (время от коммита до деплоя)
         LEAD_TIME=$(($(date +%s) - $(git log -1 --format=%ct)))
         
         # Отправка в метрики
         echo "dora_deployment_frequency $DEPLOY_COUNT"
         echo "dora_lead_time_seconds $LEAD_TIME"
     when: on_success
     only:
       - main
   ```

**2. Custom Prometheus exporter:**
   ```python
   # metrics_exporter.py
   from prometheus_client import start_http_server, Gauge
   import time
   import requests
   
   # Метрики
   pipeline_duration = Gauge('gitlab_pipeline_duration_seconds', 
                             'Pipeline duration', 
                             ['project', 'branch'])
   
   deployment_count = Gauge('gitlab_deployments_total', 
                            'Total deployments', 
                            ['environment'])
   
   def collect_metrics():
       # Получение данных из GitLab API
       response = requests.get(
           f"{GITLAB_URL}/api/v4/projects/{PROJECT_ID}/pipelines",
           headers={"PRIVATE-TOKEN": GITLAB_TOKEN}
       )
       
       for pipeline in response.json():
           pipeline_duration.labels(
               project=PROJECT_NAME,
               branch=pipeline['ref']
           ).set(pipeline['duration'])
   
   if __name__ == '__main__':
       start_http_server(8000)
       while True:
           collect_metrics()
           time.sleep(60)
   ```

**3. Synthetic monitoring:**
   ```yaml
   synthetic_monitoring:
     stage: monitor
     image: node:16
     script:
       - npm install -g @datadog/datadog-ci
       - |
         datadog-ci synthetics run-tests \
           --apiKey $DATADOG_API_KEY \
           --appKey $DATADOG_APP_KEY \
           --public-id abc-def-ghi
     only:
       - schedules
   ```

---

## Модуль 10: Advanced CI/CD Patterns (30 минут)

### 🎯 Напоминалка

**Monorepo strategies:**
```yaml
# Определение изменений
workflow:
  rules:
    - changes:
        - backend/**/*
      variables:
        BUILD_BACKEND: "true"
    - changes:
        - frontend/**/*
      variables:
        BUILD_FRONTEND: "true"

build_backend:
  rules:
    - if: '$BUILD_BACKEND == "true"'
  script:
    - cd backend && npm run build

build_frontend:
  rules:
    - if: '$BUILD_FRONTEND == "true"'
  script:
    - cd frontend && npm run build
```

**Matrix builds:**
```yaml
test:
  parallel:
    matrix:
      - NODE_VERSION: ['14', '16', '18']
        OS: ['ubuntu', 'alpine']
  image: node:${NODE_VERSION}-${OS}
  script:
    - npm test
```

**Conditional pipelines:**
```yaml
workflow:
  rules:
    # Только для MR
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      variables:
        PIPELINE_TYPE: "MR"
    # Только для main
    - if: '$CI_COMMIT_BRANCH == "main"'
      variables:
        PIPELINE_TYPE: "MAIN"
    # Только для тегов
    - if: '$CI_COMMIT_TAG'
      variables:
        PIPELINE_TYPE: "RELEASE"
```

**Parent-Child pipelines:**
```yaml
# Parent pipeline
trigger_child:
  stage: build
  trigger:
    include: .gitlab-ci-child.yml
    strategy: depend

# .gitlab-ci-child.yml
child_job:
  script:
    - echo "Child pipeline job"
```

**Pipeline templates:**
```yaml
# templates/deploy.yml
.deploy_template:
  script:
    - echo "Deploying to $ENVIRONMENT"
  environment:
    name: $ENVIRONMENT
    url: https://$ENVIRONMENT.example.com

# Использование
include:
  - local: 'templates/deploy.yml'

deploy_staging:
  extends: .deploy_template
  variables:
    ENVIRONMENT: staging

deploy_production:
  extends: .deploy_template
  variables:
    ENVIRONMENT: production
```

**Caching strategies:**
```yaml
# Global cache
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
  policy: pull-push  # pull, push, pull-push

# Per-job cache
test:
  cache:
    key: test-cache
    paths:
      - .jest-cache/
    policy: pull

# Cache с fallback
cache:
  key:
    files:
      - package-lock.json
  fallback_keys:
    - ${CI_COMMIT_REF_SLUG}
    - default
```

**GitOps with GitLab:**
```yaml
# ArgoCD integration
deploy_with_argocd:
  stage: deploy
  image: argoproj/argocd:latest
  script:
    - argocd login $ARGOCD_SERVER --username admin --password $ARGOCD_PASSWORD
    - argocd app sync myapp
    - argocd app wait myapp --health
```

### 💻 Задание

Создай production-ready monorepo pipeline:

1. **Структура monorepo:**
   ```bash
   mkdir -p backend frontend shared
   
   # Backend
   cat > backend/package.json <<EOF
   {
     "name": "backend",
     "version": "1.0.0",
     "scripts": {
       "test": "echo 'Backend tests'",
       "build": "echo 'Building backend'",
       "deploy": "echo 'Deploying backend'"
     }
   }
   EOF
   
   # Frontend
   cat > frontend/package.json <<EOF
   {
     "name": "frontend",
     "version": "1.0.0",
     "scripts": {
       "test": "echo 'Frontend tests'",
       "build": "echo 'Building frontend'",
       "deploy": "echo 'Deploying frontend'"
     }
   }
   EOF
   
   # Shared
   cat > shared/package.json <<EOF
   {
     "name": "shared",
     "version": "1.0.0",
     "scripts": {
       "test": "echo 'Shared tests'",
       "build": "echo 'Building shared'"
     }
   }
   EOF
   ```

2. **Создай продвинутый .gitlab-ci.yml:**
   ```yaml
   # Workflow rules
   workflow:
     rules:
       - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
       - if: '$CI_COMMIT_BRANCH == "main"'
       - if: '$CI_COMMIT_TAG'
   
   # Variables
   variables:
     CACHE_VERSION: "v1"
     FF_USE_FASTZIP: "true"
     ARTIFACT_COMPRESSION_LEVEL: "fast"
     CACHE_COMPRESSION_LEVEL: "fast"
   
   # Stages
   stages:
     - detect-changes
     - install
     - lint
     - test
     - build
     - deploy
   
   # Detect что изменилось
   detect-changes:
     stage: detect-changes
     image: alpine:latest
     script:
       - |
         # Проверяем изменения
         if [ "$CI_PIPELINE_SOURCE" == "merge_request_event" ]; then
           git diff --name-only $CI_MERGE_REQUEST_DIFF_BASE_SHA $CI_COMMIT_SHA > changes.txt
         else
           git diff --name-only HEAD~1 HEAD > changes.txt
         fi
         
         # Определяем что нужно собирать
         if grep -q "^backend/" changes.txt; then
           echo "BUILD_BACKEND=true" >> build.env
         fi
         if grep -q "^frontend/" changes.txt; then
           echo "BUILD_FRONTEND=true" >> build.env
         fi
         if grep -q "^shared/" changes.txt; then
           echo "BUILD_SHARED=true" >> build.env
           echo "BUILD_BACKEND=true" >> build.env
           echo "BUILD_FRONTEND=true" >> build.env
         fi
         
         cat build.env
     artifacts:
       reports:
         dotenv: build.env
   
   # Templates
   .install_template:
     cache:
       key:
         files:
           - $COMPONENT/package-lock.json
         prefix: ${CACHE_VERSION}-${COMPONENT}
       paths:
         - $COMPONENT/node_modules/
     before_script:
       - cd $COMPONENT
       - npm ci
   
   .test_template:
     extends: .install_template
     script:
       - npm test
   
   .build_template:
     extends: .install_template
     script:
       - npm run build
     artifacts:
       paths:
         - $COMPONENT/dist/
       expire_in: 1 day
   
   # Shared
   install:shared:
     stage: install
     extends: .install_template
     variables:
       COMPONENT: shared
     rules:
       - if: '$BUILD_SHARED == "true"'
   
   test:shared:
     stage: test
     extends: .test_template
     variables:
       COMPONENT: shared
     needs: ["install:shared"]
     rules:
       - if: '$BUILD_SHARED == "true"'
   
   build:shared:
     stage: build
     extends: .build_template
     variables:
       COMPONENT: shared
     needs: ["test:shared"]
     rules:
       - if: '$BUILD_SHARED == "true"'
   
   # Backend
   install:backend:
     stage: install
     extends: .install_template
     variables:
       COMPONENT: backend
     needs: ["build:shared"]
     rules:
       - if: '$BUILD_BACKEND == "true"'
   
   test:backend:
     stage: test
     extends: .test_template
     variables:
       COMPONENT: backend
     needs: ["install:backend"]
     rules:
       - if: '$BUILD_BACKEND == "true"'
   
   build:backend:
     stage: build
     extends: .build_template
     variables:
       COMPONENT: backend
     needs: ["test:backend"]
     rules:
       - if: '$BUILD_BACKEND == "true"'
   
   deploy:backend:
     stage: deploy
     script:
       - cd backend && npm run deploy
     needs: ["build:backend"]
     environment:
       name: production/backend
       url: https://api.example.com
     rules:
       - if: '$BUILD_BACKEND == "true" && $CI_COMMIT_BRANCH == "main"'
         when: manual
   
   # Frontend
   install:frontend:
     stage: install
     extends: .install_template
     variables:
       COMPONENT: frontend
     needs: ["build:shared"]
     rules:
       - if: '$BUILD_FRONTEND == "true"'
   
   test:frontend:
     stage: test
     extends: .test_template
     variables:
       COMPONENT: frontend
     parallel:
       matrix:
         - BROWSER: [chrome, firefox]
     script:
       - npm test -- --browser=$BROWSER
     needs: ["install:frontend"]
     rules:
       - if: '$BUILD_FRONTEND == "true"'
   
   build:frontend:
     stage: build
     extends: .build_template
     variables:
       COMPONENT: frontend
     needs: ["test:frontend"]
     rules:
       - if: '$BUILD_FRONTEND == "true"'
   
   deploy:frontend:
     stage: deploy
     script:
       - cd frontend && npm run deploy
     needs: ["build:frontend"]
     environment:
       name: production/frontend
       url: https://example.com
     rules:
       - if: '$BUILD_FRONTEND == "true" && $CI_COMMIT_BRANCH == "main"'
         when: manual
   ```

3. **Добавь динамический child pipeline:**
   ```yaml
   # В основной .gitlab-ci.yml добавь
   generate_dynamic_pipeline:
     stage: detect-changes
     script:
       - |
         cat > dynamic-pipeline.yml <<EOF
         stages:
           - dynamic-build
         
         EOF
         
         # Добавляем jobs только для измененных компонентов
         if grep -q "^backend/" changes.txt; then
           cat >> dynamic-pipeline.yml <<EOF
         build-backend-dynamic:
           stage: dynamic-build
           script:
             - echo "Dynamic backend build"
         EOF
         fi
     artifacts:
       paths:
         - dynamic-pipeline.yml
   
   trigger_dynamic:
     stage: build
     needs: ["generate_dynamic_pipeline"]
     trigger:
       include:
         - artifact: dynamic-pipeline.yml
           job: generate_dynamic_pipeline
       strategy: depend
   ```

4. **Тестируй:**
   ```bash
   # Измени только backend
   echo "// change" >> backend/index.js
   git add backend/
   git commit -m "feat: update backend"
   git push origin main
   
   # Проверь что собирается только backend
   ```

### 🚀 Бонус (новое)

**1. Микросервисная архитектура с service mesh:**
```yaml
# Деплой в Istio service mesh
deploy_with_istio:
  stage: deploy
  image: istio/istioctl:latest
  script:
    - |
      # Virtual Service для канарейки
      cat <<EOF | kubectl apply -f -
      apiVersion: networking.istio.io/v1beta1
      kind: VirtualService
      metadata:
        name: myapp
      spec:
        hosts:
        - myapp
        http:
        - match:
          - headers:
              canary:
                exact: "true"
          route:
          - destination:
              host: myapp
              subset: v2
            weight: 100
        - route:
          - destination:
              host: myapp
              subset: v1
            weight: 90
          - destination:
              host: myapp
              subset: v2
            weight: 10
      EOF
```

**2. Feature flags с LaunchDarkly:**
```yaml
deploy_with_feature_flags:
  stage: deploy
  script:
    - |
      # Включение feature flag после деплоя
      curl -X PATCH https://app.launchdarkly.com/api/v2/flags/$PROJECT/new-feature \
        -H "Authorization: $LD_API_KEY" \
        -H "Content-Type: application/json" \
        -d '{
          "comment": "Enabling in production",
          "environmentKey": "production",
          "instructions": [{
            "kind": "turnFlagOn"
          }]
        }'
```

**3. Auto-rollback при ошибках:**
```yaml
deploy_production:
  stage: deploy
  script:
    - ./deploy.sh
    - sleep 60  # Даем время на прогрев
  after_script:
    - |
      # Проверяем метрики
      ERROR_RATE=$(curl -s $PROMETHEUS_URL/api/v1/query?query=rate\(http_errors_total[5m]\) | jq -r '.data.result[0].value[1]')
      
      if (( $(echo "$ERROR_RATE > 0.05" | bc -l) )); then
        echo "⚠️ High error rate detected: $ERROR_RATE"
        echo "Rolling back..."
        ./rollback.sh
        exit 1
      fi
  environment:
    name: production
```

---

## Финальный проект (60 минут)

### Задача: Развернуть полноценное CI/CD для микросервисного приложения

Создай production-ready GitLab CI/CD для трехуровневого приложения с полным набором практик DevOps.

**Архитектура:**
- Frontend (React)
- Backend API (Node.js/Python)
- Database (PostgreSQL)
- Redis Cache
- Nginx Ingress
- Monitoring (Prometheus/Grafana)

**Требования:**

1. **Repository Structure:**
   ```
   my-app/
   ├── .gitlab-ci.yml
   ├── frontend/
   │   ├── Dockerfile
   │   ├── package.json
   │   └── src/
   ├── backend/
   │   ├── Dockerfile
   │   ├── package.json
   │   └── src/
   ├── database/
   │   ├── migrations/
   │   └── seeds/
   ├── k8s/
   │   ├── base/
   │   └── overlays/
   ├── terraform/
   │   └── main.tf
   └── docs/
       └── README.md
   ```

2. **CI/CD Pipeline должен включать:**
   - ✅ Автоматическое определение изменений (monorepo)
   - ✅ Параллельные jobs для frontend/backend
   - ✅ Unit, Integration, E2E тесты
   - ✅ SAST, Dependency Scanning, Container Scanning
   - ✅ Docker build с multi-stage
   - ✅ Push в GitLab Container Registry
   - ✅ Semantic versioning
   - ✅ Review Apps для MR
   - ✅ Staging деплой (автоматически)
   - ✅ Production деплой (manual, с approval)
   - ✅ Smoke tests после деплоя
   - ✅ Performance testing
   - ✅ Rollback capability

3. **Security:**
   - ✅ Secrets в GitLab Variables/Vault
   - ✅ RBAC для environments
   - ✅ Security scanning
   - ✅ Image signing
   - ✅ Network policies

4. **Monitoring:**
   - ✅ Metrics collection
   - ✅ Logging aggregation
   - ✅ Alerting
   - ✅ DORA metrics

5. **Documentation:**
   - ✅ Architecture diagram
   - ✅ Setup guide
   - ✅ Deployment guide
   - ✅ Troubleshooting guide

**Базовый шаблон .gitlab-ci.yml:**

```yaml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
  - local: '.gitlab/ci/frontend.yml'
  - local: '.gitlab/ci/backend.yml'
  - local: '.gitlab/ci/deploy.yml'

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  CACHE_VERSION: "v1"

workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'
    - if: '$CI_COMMIT_TAG =~ /^v[0-9]+\.[0-9]+\.[0-9]+$/'

stages:
  - detect
  - install
  - lint
  - test
  - security
  - build
  - review
  - staging
  - production
  - performance
  - cleanup

# Остальная конфигурация...
```

**Начальная точка:**
1. Создай структуру репозитория
2. Настрой базовый pipeline
3. Добавь Docker builds
4. Настрой environments
5. Добавь security scanning
6. Настрой monitoring
7. Документируй всё

---

## Справочная секция: Быстрые шпаргалки

### GitLab CLI (glab) команды

```bash
# Repository
glab repo clone owner/repo
glab repo view
glab repo create

# Issues
glab issue list
glab issue create --title "Bug" --description "Description" --label bug
glab issue view 123
glab issue close 123

# Merge Requests
glab mr create --title "Feature" --description "Description"
glab mr list
glab mr view 456
glab mr merge 456
glab mr approve 456
glab mr checkout 456

# CI/CD
glab ci view
glab ci trace
glab ci list
glab ci retry

# Variables
glab variable set KEY value
glab variable list

# API
glab api projects/:id/pipelines
```

### GitLab API примеры

```bash
# Authentication
export GITLAB_TOKEN="your-token"
export GITLAB_URL="https://gitlab.com"

# Projects
curl --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_URL/api/v4/projects"

# Pipelines
curl --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_URL/api/v4/projects/:id/pipelines"

# Trigger pipeline
curl --request POST \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_URL/api/v4/projects/:id/pipeline"

# Environments
curl --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_URL/api/v4/projects/:id/environments"

# Deployments
curl --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_URL/api/v4/projects/:id/deployments"
```

### Troubleshooting Guide

**Pipeline не запускается:**
```bash
# Проверка workflow rules
# Проверь .gitlab-ci.yml syntax
gitlab-ci-lint .gitlab-ci.yml

# Проверь runner'ы
# Settings > CI/CD > Runners

# Логи runner
sudo gitlab-runner verify
sudo journalctl -u gitlab-runner -f
```

**Job падает:**
```bash
# Проверка логов
glab ci trace

# Локальный запуск с gitlab-runner
gitlab-runner exec docker job_name

# Debug mode
variables:
  CI_DEBUG_TRACE: "true"
```

**Docker build проблемы:**
```bash
# Проверка DinD
docker info

# Очистка
docker system prune -a

# BuildKit
export DOCKER_BUILDKIT=1
```

**Кеш не работает:**
```bash
# Очистка кеша
# Settings > CI/CD > Clear runner caches

# Проверка cache key
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
```

**Artifacts не сохраняются:**
```bash
# Проверка размера
# Settings > CI/CD > Maximum artifacts size

# Проверка expire_in
artifacts:
  expire_in: 1 week
```

### Best Practices

**1. Pipeline оптимизация:**
```yaml
✅ DO:
  - Используй DAG (needs) для параллелизма
  - Кешируй зависимости
  - Используй artifacts только для необходимого
  - Оптимизируй Docker layers
  - Используй only/except или rules для условий

❌ DON'T:
  - Не запускай всё последовательно
  - Не качай зависимости в каждом job
  - Не сохраняй огромные artifacts
  - Не делай FROM scratch каждый раз
```

**2. Security:**
```yaml
✅ DO:
  - Используй protected variables
  - Используй Vault для secrets
  - Сканируй образы на уязвимости
  - Используй RBAC
  - Регулярно ротируй credentials

❌ DON'T:
  - Не коммить secrets
  - Не использовать plain text passwords
  - Не давать всем доступ к production
  - Не логировать sensitive данные
```

**3. Git workflow:**
```yaml
✅ DO:
  - Используй feature branches
  - Делай small commits
  - Пиши понятные commit messages
  - Используй MR templates
  - Code review обязателен

❌ DON'T:
  - Не коммить в main напрямую
  - Не делать massive commits
  - Не мержить без review
  - Не игнорировать CI/CD failures
```

### Полезные интеграции

**Slack notifications:**
```yaml
.notify_slack:
  image: curlimages/curl:latest
  script:
    - |
      curl -X POST $SLACK_WEBHOOK \
        -H 'Content-Type: application/json' \
        -d '{
          "text": "'"$MESSAGE"'",
          "username": "GitLab CI",
          "icon_emoji": ":gitlab:"
        }'

notify_success:
  extends: .notify_slack
  variables:
    MESSAGE: "✅ Pipeline succeeded: $CI_PIPELINE_URL"
  when: on_success

notify_failure:
  extends: .notify_slack
  variables:
    MESSAGE: "❌ Pipeline failed: $CI_PIPELINE_URL"
  when: on_failure
```

**Jira integration:**
```yaml
update_jira:
  stage: .post
  script:
    - |
      ISSUE_KEY=$(echo $CI_COMMIT_MESSAGE | grep -oP 'PROJ-\d+')
      if [ ! -z "$ISSUE_KEY" ]; then
        curl -X POST \
          -H "Authorization: Basic $JIRA_TOKEN" \
          -H "Content-Type: application/json" \
          -d '{
            "body": "Build completed: '"$CI_PIPELINE_URL"'"
          }' \
          "$JIRA_URL/rest/api/2/issue/$ISSUE_KEY/comment"
      fi
```

**GitHub mirror:**
```yaml
mirror_to_github:
  stage: .post
  script:
    - git push --mirror https://$GITHUB_TOKEN@github.com/user/repo.git
  only:
    - main
```

---

## План повторения

### При первом прохождении (2-3 часа):
- Пройди модули 1-4 обязательно
- Модули 5-7 по желанию
- Финальный проект упрощенный

### При повторном прохождении (через 6-12 месяцев):
- Бегло просмотри теорию
- Сфокусируйся на бонусных заданиях
- Пройди модули 8-10 обязательно
- Финальный проект полностью
- Добавь свои кастомизации

### Для закрепления:
- Автоматизируй деплой своих проектов через GitLab CI/CD
- Настрой GitOps с ArgoCD/Flux
- Попробуй разные Cloud провайдеры
- Изучи GitLab Runner на Kubernetes
- Получи сертификацию GitLab Certified Associate

### Дополнительные ресурсы:
- **GitLab Documentation** - официальная документация
- **GitLab CI/CD Examples** - примеры конфигураций
- **GitLab YouTube Channel** - видео туториалы
- **GitLab Forum** - сообщество
- **Awesome GitLab** - коллекция ресурсов
- **GitLab CI/CD Pipeline Configuration Reference** - полный справочник

---

## Чек-лист навыков

После прохождения курса ты должен уметь:

### Базовые навыки:
- ✅ Работать с GitLab интерфейсом уверенно
- ✅ Создавать и управлять проектами
- ✅ Использовать Merge Requests эффективно
- ✅ Настраивать базовые CI/CD пайплайны
- ✅ Работать с GitLab Runner
- ✅ Использовать Container Registry

### Продвинутые навыки:
- ✅ Настраивать сложные multi-stage пайплайны
- ✅ Работать с Environments и Deployments
- ✅ Настраивать Security scanning
- ✅ Управлять Secrets безопасно
- ✅ Использовать Dynamic environments
- ✅ Настраивать мониторинг и метрики

### Expert навыки:
- ✅ Проектировать CI/CD для monorepo
- ✅ Настраивать GitOps workflows
- ✅ Оптимизировать pipeline performance
- ✅ Интегрировать с внешними системами
- ✅ Настраивать auto-scaling runners
- ✅ Создавать reusable templates

### Архитектурные навыки:
- ✅ Проектировать CI/CD стратегию для организации
- ✅ Планировать disaster recovery
- ✅ Оптимизировать costs
- ✅ Обеспечивать compliance и security
- ✅ Настраивать observability
- ✅ Масштабировать GitLab infrastructure

---

## Заключение

Поздравляю! Ты прошел курс по освежению знаний GitLab CE.

**Следующие шаги:**
1. Практикуйся регулярно - используй GitLab для всех своих проектов
2. Автоматизируй всё что можно через CI/CD
3. Изучай смежные технологии: Kubernetes, Terraform, ArgoCD
4. Получи сертификацию GitLab Certified Associate
5. Делись знаниями - помогай новичкам, пиши статьи

**Помни:**
- GitLab - это не только CI/CD, это полноценная DevOps платформа
- Начинай с простого, усложняй постепенно
- Документация - твой лучший друг
- Community очень дружелюбное и готово помочь

Проходи этот курс каждые 6-12 месяцев, чтобы оставаться в форме. Каждый раз ты будешь узнавать что-то новое и замечать, как выросли твои навыки!

**Что нового в последних версиях GitLab:**

**GitLab 16.x:**
- AI-powered code suggestions
- Improved Security Dashboard
- Better Kubernetes integration
- Enhanced Package Registry
- GitOps improvements

**GitLab 17.x (upcoming):**
- Advanced AI features
- Improved pipeline efficiency
- Better observability
- Enhanced security scanning
- Multi-cloud support improvements

Следи за release notes на about.gitlab.com

Happy GitLab learning! 🦊🚀
