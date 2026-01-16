# Soft Skills for DevOps/SysAdmin: Ежегодный/Полугодовой курс

**Цель:** Освежить в памяти ключевые навыки коммуникации, командной работы и профессионального роста за 2-3 часа практики и узнать 1-2 новые техники.

**Формат:** Каждый модуль состоит из:

1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная ситуация для отработки
3. **Бонусного задания (для роста)**: Продвинутая техника или сценарий

**Зачем это DevOps/SysAdmin?**

- 70% проблем в IT связаны с коммуникацией, а не с технологиями
- Карьерный рост часто упирается в soft skills, а не в технические знания
- Эффективная коммуникация = меньше инцидентов, быстрее решение проблем
- Умение работать с людьми = возможность влиять на архитектурные решения

---

## Модуль 1: Эффективная коммуникация в кризисных ситуациях (20 минут)

### 🎯 Напоминалка

**Золотое правило инцидентов:**

```
1. ФАКТЫ (что произошло)
2. IMPACT (кто/что затронуто)
3. ДЕЙСТВИЯ (что делается сейчас)
4. ETA (когда ожидается решение)
5. ESCALATION (если нужна помощь)
```

**Шаблон сообщения об инциденте:**

```
🔴 [SEVERITY] Краткое описание проблемы

IMPACT:
- Кто затронут: [Production users / Internal team / Specific service]
- Масштаб: [% пользователей, количество запросов, etc.]

STATUS:
- Обнаружено: [время]
- Текущее действие: [что делается]
- ETA: [примерное время решения или "investigating"]

TEAM:
- Incident Commander: @name
- Engineers: @name1, @name2

Updates: будут каждые [15/30 мин]
```

**Правило 3С для стрессовых ситуаций:**

1. **Calm** (Спокойствие) - дыши, не паникуй
2. **Clear** (Ясность) - короткие предложения, конкретика
3. **Collaborative** (Сотрудничество) - "нам нужна помощь", а не "ты виноват"

**Фразы, которые НЕ нужно говорить во время инцидента:**

- ❌ "Это не моя ответственность"
- ❌ "Кто это сломал?"
- ❌ "Я же говорил, что это произойдет"
- ❌ "Это невозможно исправить"
- ❌ "Паника! Всё упало!"

**Фразы, которые НУЖНО говорить:**

- ✅ "Я вижу проблему в [область], начинаю проверку"
- ✅ "Мне нужна помощь с [конкретная задача]"
- ✅ "Текущий статус: [факты]"
- ✅ "Предлагаю попробовать [действие], риски: [какие]"
- ✅ "Зафиксировал в incident log, продолжаю работу"

**Техника "Incident Commander":**

```
Роли во время инцидента:
├── Incident Commander (координатор)
│   └── Принимает решения, управляет коммуникацией
├── Engineers (исполнители)
│   └── Решают техническую проблему
├── Communications Lead
│   └── Общается со stakeholders
└── Scribe (документатор)
    └── Фиксирует timeline и действия
```

**Post-Mortem культура:**

```
Цель Post-Mortem:
✅ Понять ЧТО произошло
✅ Понять ПОЧЕМУ произошло
✅ Решить КАК предотвратить

НЕ цель:
❌ Найти виноватого
❌ Наказать
❌ Оправдаться
```

**Правило "Blame-free" культуры:**

- "Human error" - это не root cause, это симптом
- Вопрос не "кто виноват?", а "что в процессе позволило это?"
- Фокус на системе, а не на человеке

### 💻 Задание

**Сценарий:** Production база данных показывает высокую latency. Пользователи жалуются на медленную работу сайта.

**Твоя задача:**

1. Напиши первое сообщение в incident канал Slack (используй шаблон выше)
2. Через 15 минут напиши update (предположим, ты нашел проблему - большой query от аналитики)
3. Напиши финальное сообщение о решении
4. Создай краткий outline для Post-Mortem документа

**Пример решения:**

```
1. Первое сообщение:
🔴 [P1] Production DB High Latency

IMPACT:
- Кто затронут: All production users
- Масштаб: ~40% requests slower than 3s (SLA breach)

STATUS:
- Обнаружено: 14:23 UTC (3 min ago)
- Текущее действие: Checking slow query log, monitoring dashboards
- ETA: investigating

TEAM:
- Incident Commander: @alex
- Engineers: @alex, calling @maria (DBA)

Updates: каждые 15 мин

2. Update через 15 минут:
🟡 [UPDATE] Production DB High Latency

FOUND:
- Root cause: Unoptimized analytics query running without index
- Query identified: SELECT * FROM events WHERE user_id IN (...)
- Started: ~14:20 UTC from analytics service

ACTIONS:
- Killed long-running query
- Added query to slow query killlist
- Monitoring recovery

IMPACT:
- Latency improving: 3s → 1.2s (target: <500ms)
- ETA to full resolution: 5-10 min

Next update: 14:55 UTC or when resolved

3. Финальное сообщение:
🟢 [RESOLVED] Production DB High Latency

RESOLUTION:
- Time to resolve: 23 minutes
- Action taken: Killed unoptimized query, added to killlist
- Current status: Latency back to normal (~300ms p95)

POST-MORTEM:
- Scheduled: Tomorrow 10:00 AM
- Doc: [link to doc]
- All incident participants please review

Thanks to @maria for quick DBA support!

4. Post-Mortem Outline:
---
# Post-Mortem: Production DB High Latency (2025-01-16)

## Summary
- Duration: 23 minutes (14:23 - 14:46 UTC)
- Impact: 40% of users experienced degraded performance
- Severity: P1

## Timeline
- 14:20 - Unoptimized query started from analytics service
- 14:23 - Alerts fired, incident declared
- 14:30 - Root cause identified
- 14:35 - Query killed
- 14:46 - Service fully recovered

## Root Cause
Unoptimized analytics query without proper indexing ran on production DB

## What Went Well
- Quick detection (3 min)
- Clear communication in incident channel
- Fast resolution with DBA help

## What Went Wrong
- Analytics queries running directly on production DB
- No query review process before deployment
- Missing indexes for common analytics patterns

## Action Items
- [ ] Move analytics queries to read replica (@maria, Jan 20)
- [ ] Implement query review process (@alex, Jan 25)
- [ ] Add missing indexes (@maria, Jan 18)
- [ ] Set up query timeout limits (@alex, Jan 22)
- [ ] Document analytics query best practices (@team, Jan 30)

## Lessons Learned
- Need separation between production and analytics workloads
- Query review is critical for DB performance
- Incident response process worked well, keep using it
---
```

### 🚀 Бонус (новое)

**Техника "Situation-Complication-Resolution" для эскалации:**

Когда нужно эскалировать проблему руководству, используй SCR:

```
Ситуация: Что происходит (факты)
Осложнение: Почему это проблема (impact)
Решение: Что ты предлагаешь (action)

Пример:
"У нас production DB инцидент (Situation). 
Это влияет на 40% пользователей, и мы рискуем нарушить SLA (Complication). 
Я предлагаю переключиться на failover, но это требует вашего одобрения, 
так как вызовет 5-минутный downtime (Resolution)."
```

**Практика "Blameless Language":**

Вместо:

- ❌ "Разработчик X написал плохой код"
- ❌ "Команда Y не протестировала"

Говори:

- ✅ "Код не прошел edge case проверку"
- ✅ "Тест не покрыл этот сценарий"

**Техника "5 Whys" для Post-Mortem:**

```
Проблема: Сайт упал
Why? → Сервер перегружен
Why? → Слишком много запросов
Why? → Вирусная новость привела трафик
Why? → Нет автоскейлинга
Why? → Не было требования на автоскейлинг

Root cause: Отсутствие процесса capacity planning
```

---

## Модуль 2: Коммуникация с Non-Technical Stakeholders (25 минут)

### 🎯 Напоминалка

**Правило перевода технического на человеческий:**

```
Техническое → Бизнес-язык

❌ "У нас latency 500ms на p95"
✅ "Сайт отвечает в 2 раза медленнее нормы, пользователи это чувствуют"

❌ "Kubernetes pod crashed из-за OOM"
✅ "Сервис перезапустился из-за нехватки памяти, это вызвало 2-минутный перерыв"

❌ "Нужно настроить HPA для deployment"
✅ "Хочу настроить автоматическое масштабирование, чтобы справляться с пиками нагрузки"

❌ "Migration требует downtime"
✅ "Для обновления нужно остановить систему на 30 минут в 3 ночи в выходные"
```

**Структура объяснения для менеджеров:**

```
1. ЗАЧЕМ (бизнес-ценность)
   "Это ускорит deployment с 2 часов до 15 минут"

2. ЧТО (простыми словами)
   "Мы настроим автоматическую систему доставки кода"

3. РИСКИ (что может пойти не так)
   "Первые 2 недели возможны мелкие сбои, пока настраиваем"

4. ПЛАН (конкретные шаги)
   "3 этапа за 4 недели, начинаем с тестовой среды"

5. НУЖНА ПОМОЩЬ (что требуется от них)
   "Нужно 20% времени команды на это в течение месяца"
```

**Техника "Elevator Pitch" для технических решений:**

```
30 секунд для объяснения:

"Сейчас когда мы выкатываем код, это занимает 2 часа и требует ручной работы. 
Я предлагаю автоматизировать это с помощью CI/CD. 
Результат: deployment за 15 минут, меньше ошибок, команда может фокусироваться на разработке.
Нужно 4 недели на настройку, окупится за 2 месяца."
```

**Levels of Explanation (от простого к сложному):**

```
Level 1 - CEO/Executives:
"Это сэкономит $50k в год и ускорит релизы в 4 раза"

Level 2 - Product Managers:
"Пользователи получат новые фичи в 4 раза быстрее, меньше багов в production"

Level 3 - Engineering Managers:
"Автоматизируем deployment pipeline, добавим automated testing, улучшим monitoring"

Level 4 - Engineers:
"Jenkins pipeline с GitOps, ArgoCD для K8s, Prometheus для метрик..."
```

**Правило "So What?":** После каждого технического утверждения спрашивай себя "И что?":

```
"Настрою Kubernetes" 
→ So what? 
→ "Приложения будут автоматически масштабироваться"
→ So what?
→ "Справимся с пиковыми нагрузками без падений"
→ So what?
→ "Пользователи не испытают downtime во время распродаж"
```

**Email структура для технических предложений:**

```
Subject: [ACTION NEEDED] Proposal: Automated Deployment Pipeline

Hi [Name],

TL;DR: 
Хочу автоматизировать deployment, это сократит время релиза с 2ч до 15мин 
и снизит количество инцидентов. Нужно одобрение на 4 недели работы команды.

PROBLEM:
- Сейчас каждый deployment занимает 2 часа ручной работы
- За последний квартал было 5 инцидентов из-за human error при deployment
- Команда тратит 30% времени на рутинные deployment задачи

PROPOSAL:
Настроить автоматический CI/CD pipeline:
- Automated testing перед каждым релизом
- One-click deployment в production
- Automatic rollback при проблемах

BENEFITS:
- ⏱️ Deployment: 2 часа → 15 минут (8x faster)
- 🐛 Fewer bugs: automated testing catches issues before production
- 👥 Team productivity: 30% времени освободится для feature development
- 💰 ROI: окупится за 2 месяца в сэкономленном времени команды

TIMELINE:
- Week 1-2: Setup pipeline для staging
- Week 3: Testing and refinement
- Week 4: Rollout to production
- Total: 4 weeks, 20% of team time

RISKS & MITIGATION:
- Risk: Learning curve для команды
  Mitigation: Обучающие сессии, документация
- Risk: Первичные настройки могут вызвать задержки
  Mitigation: Начинаем со staging, production в конце

NEED FROM YOU:
- Approval для 20% team allocation на 4 недели
- Бюджет на tooling: ~$500/month (CI/CD tools)

Happy to discuss details or schedule a demo!

Best,
[Your name]
```

**Визуализация для презентаций:**

```
Используй диаграммы вместо текста:

Before:
[Developer] → [Manual testing] → [Manual deployment] → [Production]
             (2 hours, error-prone)

After:
[Developer] → [Automated CI/CD] → [Production]
             (15 min, reliable)
```

### 💻 Задание

**Сценарий:** Ты хочешь внедрить мониторинг (Prometheus + Grafana) для инфраструктуры. Нужно получить одобрение от:

1. CTO (технически подкован, но думает о ROI)
2. Product Manager (не технический, волнуется о приоритетах)
3. CFO (финансы, бюджет)

**Твоя задача:** Напиши 3 разных сообщения (или мини-презентации) для каждого из них, используя подходящий язык и фокус.

**Пример решения:**

```
1. Для CTO:
---
Subject: Proposal: Production Monitoring Infrastructure

Hi [CTO Name],

Хочу обсудить внедрение comprehensive monitoring stack (Prometheus + Grafana).

CURRENT STATE:
- Reactive incident response (узнаем о проблемах от пользователей)
- No historical metrics для capacity planning
- Troubleshooting занимает в среднем 45 минут из-за отсутствия данных

PROPOSAL:
Развернуть Prometheus для метрик + Grafana для визуализации:
- Real-time alerting (узнаем о проблемах ДО пользователей)
- Historical data для capacity planning и trend analysis
- Pre-built dashboards для всех critical services

TECHNICAL BENEFITS:
- MTTD (Mean Time To Detect): текущие ~15min → <1min
- MTTR (Mean Time To Resolve): ~45min → ~15min
- Proactive capacity planning vs reactive firefighting
- Foundation для SRE practices (SLO/SLI tracking)

ROI:
- Стоимость: ~$200/month (cloud hosting) + 2 недели инженерного времени
- Экономия: Каждый prevented incident = ~$5k (lost revenue + engineering time)
- Break-even: После первого prevented major incident

TIMELINE:
- Week 1: Setup и базовые метрики
- Week 2: Dashboards и alerts
- Ongoing: Refinement based on learnings

Можем начать с pilot на staging для proof of concept?

[Your name]
---

2. Для Product Manager:
---
Subject: Улучшение Reliability для лучшего User Experience

Hi [PM Name],

Хочу предложить проект, который напрямую улучшит experience наших пользователей.

PROBLEM USERS FACE:
- В прошлом квартале было 3 инцидента, которые затронули пользователей
- В среднем мы узнаем о проблемах через 15 минут после пользователей
- Troubleshooting занимает до часа, потому что мы "летаем вслепую"

WHAT I PROPOSE:
Настроить систему мониторинга, которая:
- Предупредит нас о проблемах ДО того, как пострадают пользователи
- Покажет, что именно сломалось (быстрее найдем и починим)
- Даст данные для предотвращения будущих проблем

USER IMPACT:
- Меньше downtime (мы узнаем и реагируем быстрее)
- Лучшая производительность (мы видим bottlenecks и оптимизируем)
- Более стабильный сервис (предотвращаем проблемы заранее)

TIMELINE & IMPACT ON ROADMAP:
- 2 недели инженерного времени
- Не блокирует текущие feature releases
- Можно делать параллельно с Q1 roadmap

LONG-TERM VALUE:
- Fewer support tickets (меньше "сайт не работает" complaints)
- Better user retention (stable service = happy users)
- Faster feature development (меньше времени на fixing issues)

Это инвестиция в quality, которая окупится в виде happier users и faster development.

Давай обсудим?

[Your name]
---

3. Для CFO:
---
Subject: Cost-Benefit Analysis: Production Monitoring

Hi [CFO Name],

Прошу рассмотреть инвестицию в monitoring infrastructure с четким ROI.

INVESTMENT:
- One-time: 2 недели инженерного времени (~$8,000 при $100/hour)
- Recurring: $200/month для cloud hosting ($2,400/year)
- Total Year 1: ~$10,400

RETURNS:
Current Cost of Incidents:
- Q4 2024: 3 major incidents
- Avg duration: 1 hour
- Lost revenue per incident: ~$3,000 (based on transaction volume)
- Engineering cost per incident: ~$2,000 (5 engineers × 4 hours)
- Total per incident: ~$5,000

Expected Improvement:
- Earlier detection → 50% faster resolution
- Proactive prevention → 40% fewer incidents
- Year 1 savings: ~$12,000 (prevented incidents + faster resolution)

ROI:
- Break-even: Month 2
- Year 1 ROI: 15% ($12k savings - $10.4k cost)
- Year 2+: 500% ROI (only $2.4k/year recurring cost)

RISK MITIGATION:
- Minimal: Standard industry tooling, low implementation risk
- Can start with 1-month pilot to validate assumptions
- Scalable: Works for current size, grows with company

COMPETITIVE ADVANTAGE:
- Industry standard: 95% of similar-sized companies use monitoring
- Reliability = Customer trust = Revenue retention

Recommend approval. Happy to provide more details or schedule a brief presentation.

Best regards,
[Your name]
---
```

### 🚀 Бонус (новое)

**Техника "BLOT" для презентаций:**

```
B - Bottom Line On Top
L - Logic (why)
O - Options (alternatives)
T - Timeline & Next Steps

Пример:
"Рекомендую внедрить мониторинг за $10k (BLOT).
Потому что мы теряем $15k в год на инциденты (Logic).
Альтернативы: ничего не делать (риск), или дорогой SaaS за $50k/год (Options).
План: 2 недели на setup, старт в следующем спринте (Timeline)."
```

**"Executive Summary" шаблон:**

```
# Executive Summary: [Project Name]

## Recommendation
[One sentence: what you're proposing]

## Business Impact
- Revenue: [+$X increase or -$Y cost savings]
- Risk: [what problem this solves]
- Timeline: [how fast returns come]

## Investment Required
- Money: $X
- Time: Y weeks
- Resources: Z people

## Why Now
[Urgency/opportunity]

## Key Risks
1. [Risk] - [Mitigation]
2. [Risk] - [Mitigation]

## Next Steps
1. [Action] - [Owner] - [Date]
```

**Визуализация метрик для non-technical:**

```
Вместо: "Latency p95 decreased from 500ms to 200ms"
Покажи: 
📊 График с зеленой зоной "Good" и красной "Bad"
💡 "Website is now 2.5x faster for users"
```

---

## Модуль 3: Документация и Knowledge Sharing (20 минут)

### 🎯 Напоминалка

**Правило хорошей документации:**

```
RTFM должна быть удовольствием, а не пыткой

Good docs = 
  Working Examples 
  + Clear Structure 
  + Up-to-date Info 
  + Easy to Find
```

**Структура технической документации:**

```
1. TL;DR / Quick Start
   └── "Как начать за 5 минут"

2. Overview
   └── "Что это и зачем"

3. Architecture / How it Works
   └── "Как это устроено внутри"

4. Setup Guide
   └── "Пошаговая установка"

5. Common Tasks
   └── "Типичные сценарии использования"

6. Troubleshooting
   └── "Что делать, если не работает"

7. FAQ
   └── "Частые вопросы"

8. Advanced Topics
   └── "Для опытных пользователей"

9. Reference
   └── "API, конфиги, все детали"
```

**Runbook шаблон для операционных задач:**

````markdown
# [Task Name] Runbook

## Purpose
Краткое описание, зачем эта задача

## Prerequisites
- Требуемые доступы
- Необходимые инструменты
- Знания/навыки

## Steps
### 1. [Step Name]
```bash
# Команда
command --with-flags
````

**Expected output:**

```
Success message
```

**If error:** See troubleshooting section

### 2. [Next Step]

...

## Verification

Как проверить, что всё сработало:

- [ ] Checklist item 1
- [ ] Checklist item 2

## Rollback

Что делать, если нужно откатить:

```bash
rollback-command
```

## Troubleshooting

### Error: "Permission denied"

**Solution:** Check access rights...

## Related Resources

- Link to monitoring dashboard
- Link to related docs
- Slack channel for questions

## Maintenance

- Last updated: 2025-01-16
- Owner: @yourname
- Review frequency: Monthly

````

**README.md best practices:**
```markdown
# Project Name

Brief one-liner description

[![Build Status](badge)](link)
[![License](badge)](link)

## Quick Start

```bash
# Install
npm install project-name

# Run
npm start

# Visit
http://localhost:3000
````

## Features

- ✅ Feature 1
- ✅ Feature 2
- 🚧 Feature 3 (coming soon)

## Installation

[Detailed steps]

## Usage

[Examples with code]

## Configuration

[Available options]

## Troubleshooting

[Common issues]

## Contributing

[How to contribute]

## License

[License info]

## Support

- Slack: #project-channel
- Issues: [GitHub issues](https://claude.ai/chat/link)
- Docs: [Full documentation](https://claude.ai/chat/link)

````

**Decision Records (ADR - Architecture Decision Records):**
```markdown
# ADR-001: Use PostgreSQL for Primary Database

## Status
Accepted

## Context
We need to choose a database for our application.
Requirements:
- ACID transactions
- Complex queries support
- Strong community
- Good performance for OLTP

## Decision
We will use PostgreSQL 14+

## Consequences

### Positive
- Battle-tested, mature technology
- Excellent documentation
- Strong data integrity
- Rich feature set (JSON, full-text search, etc.)

### Negative
- More complex than NoSQL for simple use cases
- Requires more operational overhead
- Scaling writes requires sharding

### Neutral
- Team needs to learn PostgreSQL best practices
- Will use RDS/Cloud SQL for managed hosting

## Alternatives Considered
- MySQL: Less feature-rich
- MongoDB: No ACID, eventual consistency
- DynamoDB: Vendor lock-in, limited query flexibility

## References
- [PostgreSQL docs](link)
- [Benchmark results](link)
- [Team discussion](link to Slack thread)

## Review Date
2026-01-16 (yearly review)
````

**Changelog best practices:**

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]
### Added
- Feature X for better UX

### Changed
- Updated dependency Y to v2.0

### Deprecated
- Old API endpoint /v1/users

### Removed
- Support for Node.js 12

### Fixed
- Bug where Z crashed on edge case

### Security
- Patched XSS vulnerability in search

## [1.2.0] - 2025-01-15
### Added
- New dashboard for analytics
- Export functionality for reports

### Fixed
- Memory leak in background worker

## [1.1.0] - 2024-12-01
...
```

**Code Comments лучшие практики:**

```python
# ❌ BAD: Obvious comments
def calculate_total(items):
    # Initialize total to 0
    total = 0
    # Loop through items
    for item in items:
        # Add item price to total
        total += item.price
    return total

# ✅ GOOD: Why, not what
def calculate_total(items):
    """Calculate total price with tax.
    
    Note: We add tax here instead of in the frontend
    because tax rates vary by user location and should
    be calculated server-side for security.
    """
    subtotal = sum(item.price for item in items)
    # Tax rate of 10% is mandated by regulations XYZ-123
    # See: https://company.wiki/tax-policy
    tax = subtotal * 0.10
    return subtotal + tax

# ✅ GOOD: Document workarounds
def fetch_user_data(user_id):
    """Fetch user data from API.
    
    HACK: We retry 3 times because the API is flaky.
    TODO: Remove this once API team fixes the stability issue.
    Tracking: JIRA-1234
    """
    for attempt in range(3):
        try:
            return api.get_user(user_id)
        except APIError:
            if attempt == 2:
                raise
            time.sleep(1)
```

**Принцип "Documentation as Code":**

```
✅ Docs живут в Git рядом с кодом
✅ Docs проверяются в code review
✅ Docs автоматически публикуются при merge
✅ Broken links проверяются в CI/CD
✅ Docs версионируются вместе с кодом
```

### 💻 Задание

**Сценарий:** Твоя команда часто сталкивается с одной и той же проблемой: "Как правильно deploy в production?" Каждый раз новые люди задают одни и те же вопросы.

**Твоя задача:**

1. Создай Runbook для deployment в production
2. Напиши Quick Start guide (для нового члена команды)
3. Создай ADR (Architecture Decision Record) для выбора deployment стратегии

**Требования к Runbook:**

- Четкие шаги (numbered)
- Команды с примерами
- Что делать при ошибках
- Как проверить успешность
- Как откатить, если что-то пошло не так

**Пример решения:**

```markdown
````markdown
# Production Deployment Runbook

## Purpose
Deploy new version of application to production environment safely.

## Prerequisites
- [ ] You have `kubectl` access to production cluster
- [ ] You have reviewed and approved the changes in PR
- [ ] All tests are passing in staging
- [ ] Deployment window approved (see #deployments channel)
- [ ] Stakeholders notified (post in #deployments 30 min before)
- [ ] Backup/snapshot created (automatic via CI/CD)

## Steps

### 1. Pre-deployment Health Check
```bash
# Check current production status
kubectl get pods -n production
kubectl get deployments -n production

# Verify all pods are healthy
kubectl get pods -n production | grep -v Running
```
**Expected output:** All pods should show `Running` status

**If error:** Do not proceed. Investigate unhealthy pods first.

### 2. Create Deployment Announcement
Post in Slack #deployments:
```
🚀 Production Deployment Starting

Version: v1.2.3
ETA: 10 minutes
Changes: [link to changelog]
Deploying: @yourname

Will update when complete.
```

### 3. Tag and Build
```bash
# Pull latest main
git checkout main
git pull origin main

# Create release tag
git tag -a v1.2.3 -m "Release v1.2.3"
git push origin v1.2.3

# Verify CI/CD pipeline started
# Check: https://ci.company.com/pipelines
```
**Expected:** Pipeline builds Docker image and pushes to registry

**If error:** Check CI/CD logs. Common issues:
- Build fails: Fix code and create new tag
- Docker push fails: Check registry credentials

### 4. Deploy to Production
```bash
# Update deployment manifest
kubectl set image deployment/app \
  app=registry.company.com/app:v1.2.3 \
  -n production

# Watch rollout status
kubectl rollout status deployment/app -n production
```
**Expected output:**
```
deployment "app" successfully rolled out
```

**If error:** See Troubleshooting section

### 5. Verify Deployment
```bash
# Check pods are running new version
kubectl get pods -n production -o wide

# Check pod logs for errors
kubectl logs -n production -l app=app --tail=50

# Verify application health endpoint
curl https://api.company.com/health
```

**Expected:**
- All pods running with new image
- No error logs
- Health endpoint returns `{"status": "healthy"}`

### 6. Smoke Tests
- [ ] Visit https://app.company.com and verify homepage loads
- [ ] Test critical user flow: Login → Dashboard → Create item
- [ ] Check monitoring dashboard: https://grafana.company.com/production
- [ ] Verify error rate < 0.1% (in Grafana)
- [ ] Verify latency p95 < 500ms (in Grafana)

### 7. Post-deployment Announcement
Post in Slack #deployments:
```
✅ Production Deployment Complete

Version: v1.2.3
Duration: 8 minutes
Status: Healthy
Metrics: Error rate 0.02%, Latency p95 320ms

Monitoring for next 30 minutes.
```

### 8. Monitor for 30 Minutes
Keep Grafana dashboard open and watch for:
- Error rate spikes
- Latency increases
- 5xx responses
- Memory/CPU abnormalities

## Verification
Final checklist:
- [ ] All pods running with new version
- [ ] Health checks passing
- [ ] Error rate within normal range
- [ ] Latency within SLA
- [ ] No alerts firing
- [ ] Smoke tests passed

## Rollback
If deployment fails or causes issues:
```bash
# Quick rollback to previous version
kubectl rollout undo deployment/app -n production

# Verify rollback
kubectl rollout status deployment/app -n production

# Check pods are healthy
kubectl get pods -n production
```

Post in Slack #deployments:
```
🔴 ROLLBACK: Production Deployment v1.2.3

Reason: [brief explanation]
Status: Rolled back to v1.2.2
Current state: Stable

Post-mortem: [link when ready]
```

## Troubleshooting

### Error: "ImagePullBackOff"
**Причина:** Не удается скачать Docker image из registry

**Solution:**
```bash
# Check if image exists in registry
docker pull registry.company.com/app:v1.2.3

# Verify registry credentials
kubectl get secret registry-credentials -n production -o yaml
```

### Error: "CrashLoopBackOff"
**Причина:** Pods continuously crashing after start

**Solution:**
```bash
# Check pod logs
kubectl logs -n production [pod-name] --previous

# Check pod events
kubectl describe pod [pod-name] -n production

# Common causes:
# - Configuration error
# - Missing environment variables
# - Database connection issues
```

### Error: Health Check Failing
**Причина:** Application started but health endpoint not responding

**Solution:**
```bash
# Check if pods are running
kubectl get pods -n production

# Check application logs
kubectl logs -n production -l app=app --tail=100

# Test health endpoint from inside cluster
kubectl run test-pod --rm -it --image=curlimages/curl -- \
  curl http://app-service:8080/health
```

### High Error Rate After Deployment
**Solution:**
1. Check Grafana for specific error types
2. Review application logs for exceptions
3. If critical (>5% error rate): ROLLBACK immediately
4. If minor: Monitor for 5 more minutes, then decide

## Related Resources
- Monitoring Dashboard: https://grafana.company.com/d/prod-overview
- CI/CD Pipeline: https://ci.company.com/app
- Deployment Calendar: https://calendar.company.com/deployments
- Slack channel: #deployments
- Runbook repository: https://github.com/company/runbooks

## Maintenance
- Last updated: 2025-01-16
- Owner: @devops-team
- Review frequency: Monthly
- Feedback: Create PR or post in #devops-feedback

---

# Quick Start Guide: Your First Production Deployment

## Добро пожаловать в команду! 👋

Этот гайд поможет тебе сделать твой первый production deployment безопасно и уверенно.

## Перед первым deployment

### 1. Получи необходимые доступы (1-2 дня)
Напиши в #access-requests:
```
Hi! Мне нужны доступы для production deployments:
- kubectl access to production cluster
- GitHub write access to main repository
- Slack #deployments channel
- Grafana view access

CC: @devops-lead
```

### 2. Настрой свое окружение (30 минут)
```bash
# Установи kubectl
brew install kubectl

# Скачай kubeconfig
# (попроси у @devops-lead)

# Проверь доступ
kubectl get pods -n production

# Установи нужные алиасы
echo "alias k='kubectl'" >> ~/.zshrc
echo "alias kprod='kubectl -n production'" >> ~/.zshrc
```

### 3. Изучи документацию (1 час)
- [ ] Прочитай полный Production Deployment Runbook (выше)
- [ ] Посмотри последние 3 deployment в #deployments
- [ ] Изучи Grafana dashboard: https://grafana.company.com/prod
- [ ] Найди наш incident response plan: wiki.company.com/incidents

### 4. Сделай staging deployment (практика)
Перед production попробуй на staging:
```bash
# Все те же команды, но для staging
kubectl get pods -n staging
kubectl set image deployment/app app=registry.company.com/app:v1.2.3 -n staging
```

## Твой первый production deployment

### Пошаговая инструкция (следуй строго!)

**День до deployment:**
1. Убедись что все тесты проходят в staging
2. Забронируй deployment window в календаре
3. Напиши в #deployments что планируешь deployment завтра

**За 30 минут до deployment:**
1. Напиши в #deployments announcement (см. шаблон в Runbook)
2. Открой Grafana dashboard в отдельной вкладке
3. Открой этот Runbook рядом

**Во время deployment (10-15 минут):**
1. Следуй шагам 1-8 из Production Deployment Runbook
2. Не паникуй если что-то идет не так
3. Если непонятно - спроси в #deployments, кто-то поможет

**После deployment:**
1. Мониторь 30 минут
2. Напиши в #deployments что deployment завершен
3. Запиши что-то новое что узнал

### Частые вопросы новичков

**Q: Что если я что-то сломаю?**
A: У нас есть rollback за 2 минуты. Серьезные проблемы почти невозможны, если следуешь runbook.

**Q: Когда можно делать deployment?**
A: Лучшее время: вторник-четверг, 10:00-15:00 UTC. Избегай пятницу и понедельник утром.

**Q: Нужно ли кого-то предупреждать?**
A: Да, всегда пиши в #deployments за 30 минут. Для больших изменений - предупреди за день.

**Q: Что делать если паникую во время инцидента?**
A: Напиши в #deployments "Need help with deployment", кто-то придет на помощь. Мы все были на твоем месте.

**Q: Как понять что deployment успешный?**
A: Смотри на Grafana: error rate < 0.1%, latency p95 < 500ms, никаких alerts. И smoke tests прошли.

### Контакты для помощи

- Emergency (инцидент): @devops-oncall в Slack
- Вопросы по deployment: #deployments
- Вопросы по инфраструктуре: #infrastructure
- Твой buddy: [имя твоего ментора]

### Следующие шаги

После первого deployment:
1. ✅ Сделай еще 2-3 под присмотром buddy
2. 📝 Предложи улучшения в runbook (что было непонятно?)
3. 🎓 Изучи advanced темы: blue-green deployments, canary releases
4. 👥 Стань buddy для следующего новичка

**Удачи! Ты справишься! 🚀**

---

# ADR-003: Kubernetes Rolling Updates для Production Deployments

## Status
Accepted (2025-01-16)

## Context

Нам нужна стратегия deployment в production, которая:
- Минимизирует downtime (цель: zero-downtime)
- Позволяет быстро откатиться при проблемах
- Проста в использовании для всей команды
- Надежна и предсказуема

### Текущая ситуация
- 3-4 deployments в неделю
- Команда из 8 инженеров, не все опытны в DevOps
- Kubernetes кластер в production (AWS EKS)
- Монолитное приложение постепенно разбивается на микросервисы

### Требования
- Zero-downtime deployments (99.9% uptime SLA)
- Rollback за < 5 минут при проблемах
- Automated health checks
- Простой процесс для инженеров
- Возможность постепенного rollout для снижения риска

## Decision

Мы выбираем **Kubernetes Rolling Updates** как основную стратегию deployment.

### Конфигурация:
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

### Процесс:
1. CI/CD pipeline собирает и тестирует код
2. Tag создается вручную для production release
3. `kubectl set image` запускает rolling update
4. Kubernetes постепенно заменяет pods (один за раз)
5. Health checks подтверждают работоспособность перед переключением трафика
6. Rollback через `kubectl rollout undo` если проблемы

## Consequences

### Positive ✅
- **Zero-downtime**: Всегда есть working pods во время deployment
- **Automatic**: Kubernetes управляет процессом, минимум ручной работы
- **Safe**: Health checks предотвращают deployment broken code
- **Fast rollback**: `kubectl rollout undo` откатывает за 1-2 минуты
- **Simple**: Команда легко понимает и использует
- **Standard**: Industry best practice, много документации

### Negative ❌
- **Slow для больших deployments**: Постепенная замена pods занимает 5-10 минут
- **Mixed versions**: Во время deployment работают старая и новая версия одновременно (нужна backward compatibility)
- **Database migrations**: Требуют осторожности, т.к. старый код может работать с новой схемой
- **Resource overhead**: Нужен `maxSurge=1`, т.е. временно на 1 pod больше

### Neutral 🔄
- **Learning curve**: Команде нужно научиться работать с kubectl и понимать rolling updates
- **Monitoring required**: Нужен хороший мониторинг чтобы быстро заметить проблемы
- **Config management**: Нужно правильно настроить health checks и resource limits

## Alternatives Considered

### 1. Blue-Green Deployment
**Плюсы:**
- Мгновенное переключение между версиями
- Легкий instant rollback
- Полное тестирование перед переключением

**Минусы:**
- Требует 2x ресурсов (двойные инфраструктурные затраты)
- Сложнее для database migrations
- Переключение load balancer может быть сложным

**Почему отклонили:** Слишком дорого для нашего масштаба, требует больше инфраструктуры.

### 2. Canary Deployment
**Плюсы:**
- Постепенный rollout к небольшому % пользователей
- Раннее обнаружение проблем на малой аудитории
- Снижение blast radius

**Минусы:**
- Требует sophisticated traffic management (Istio/Linkerd)
- Сложнее настроить и поддерживать
- Нужна продвинутая observability для анализа

**Почему отклонили:** Слишком сложно для нашей команды сейчас. Можем вернуться к этому когда вырастем.

### 3. Recreate Strategy
**Плюсы:**
- Самая простая стратегия
- Нет mixed versions

**Минусы:**
- Downtime во время deployment (неприемлемо)
- Нарушает наш SLA

**Почему отклонили:** Не соответствует требованию zero-downtime.

## Implementation Plan

### Phase 1: Setup (Week 1)
- [ ] Настроить readiness/liveness probes для всех services
- [ ] Обновить deployment manifests с rolling update strategy
- [ ] Создать runbook для deployment процесса
- [ ] Настроить Grafana dashboards для мониторинга deployments

### Phase 2: Training (Week 2)
- [ ] Провести training session для команды
- [ ] Сделать practice deployments на staging
- [ ] Каждый инженер делает минимум 1 supervised deployment

### Phase 3: Rollout (Week 3-4)
- [ ] Первый production deployment с новым процессом (с supervision)
- [ ] Собрать feedback от команды
- [ ] Улучшить документацию на основе feedback
- [ ] Сделать процесс стандартным для всех services

## Risks & Mitigations

### Risk 1: Database Migration Issues
**Проблема:** Старая версия кода не совместима с новой database schema

**Mitigation:**
- Всегда делаем backward-compatible migrations
- Разделяем migrations на 2 deployments: сначала additive changes, потом removals
- Процесс: Deploy code with new+old schema support → Run migration → Deploy code that drops old schema support

### Risk 2: Mixed Version Issues
**Проблема:** Bugs из-за одновременной работы двух версий

**Mitigation:**
- Требуем backward compatibility в code review
- Comprehensive integration tests на staging
- Мониторинг error rates во время deployment
- Fast rollback если error rate > 1%

### Risk 3: Health Check False Positives
**Проблема:** Health check показывает OK, но приложение broken

**Mitigation:**
- Улучшить health checks (проверять database connectivity, external APIs)
- Smoke tests после deployment
- 30-минутный monitoring period после каждого deployment

### Risk 4: Team Learning Curve
**Проблема:** Инженеры делают ошибки из-за незнания процесса

**Mitigation:**
- Подробная документация (runbook, quick start guide)
- Buddy system: первые 3 deployments с опытным коллегой
- Slack #deployments для быстрой помощи
- Monthly review процесса и обновление документации

## Success Metrics

Будем считать решение успешным если через 3 месяца:
- ✅ Zero unplanned downtime из-за deployments
- ✅ Average deployment time < 15 минут
- ✅ 100% команды comfortable с процессом
- ✅ Rollback используется < 5% от deployments
- ✅ No incidents caused by deployment process

## References

- [Kubernetes Rolling Update Documentation](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
- [Google SRE Book - Deployment Strategies](https://sre.google/sre-book/release-engineering/)
- Our Production Deployment Runbook (link to internal docs)
- Team discussion: [Slack thread](link)
- Load testing results: [Confluence page](link)

## Review & Evolution

- **Next review date:** 2025-07-16 (6 months)
- **Review triggers:**
  - Если > 3 incidents связанных с deployment за квартал
  - Если команда выросла > 15 инженеров
  - Если переходим на микросервисную архитектуру
  - Если появляются новые требования к deployment процессу

- **Future considerations:**
  - Когда вырастем: рассмотреть Canary deployments для critical services
  - Когда перейдем на microservices: изучить service mesh (Istio/Linkerd)
  - Автоматизация: Helm charts, GitOps с ArgoCD

## Approvals

- DevOps Lead: @alice ✅
- Engineering Manager: @bob ✅
- CTO: @carol ✅

---

## Changelog
- 2025-01-16: Initial version, accepted by team


### 🚀 Бонус (новое)

**Техника "Documentation Debt Tracking":**

Создай файл `docs/TODO.md` в репозитории:

markdown

````markdown
# Documentation TODO

## High Priority
- [ ] Add troubleshooting guide for Redis connection issues (@alex, Jan 20)
- [ ] Update API authentication docs (changed in v2.0) (@maria, Jan 18)
- [ ] Create runbook for database failover (@bob, Jan 25)

## Medium Priority  
- [ ] Improve getting started guide based on new hire feedback
- [ ] Add diagrams to architecture doc
- [ ] Document monitoring alert thresholds

## Low Priority
- [ ] Add more code examples to SDK docs
- [ ] Translate docs to Spanish
- [ ] Create video tutorials

## Completed
- [x] Production deployment runbook (2025-01-16)
- [x] ADR for deployment strategy (2025-01-16)
```

**"README-Driven Development":**
```
Принцип: Пиши README ПЕРЕД написанием кода

1. Опиши что ты хочешь построить
2. Покажи как это будет использоваться (примеры API)
3. Получи feedback от команды
4. Теперь пиши код чтобы соответствовать README

Результат: Лучший API design, меньше переделок
````

**Шаблон для Release Notes:**

markdown

````markdown
# Release v1.2.0 - January 16, 2025

## 🎉 Highlights

The biggest improvements in this release:
- **50% faster** page load times
- **New feature:** Dark mode for all pages
- **Fixed:** Critical bug in payment processing

## ✨ New Features

### Dark Mode Support
You can now enable dark mode in Settings → Appearance.
Perfect for late-night coding sessions!

[Screenshot or demo GIF]

### Bulk Actions
Select multiple items and perform actions on all at once.
Saves time when managing large datasets.

## 🐛 Bug Fixes

- Fixed payment processing error for cards ending in 0000
- Resolved memory leak in background sync
- Corrected timezone display for users in Asia/Pacific

## ⚡ Performance Improvements

- Reduced API response time by 200ms (p95)
- Optimized database queries (3x faster for large datasets)
- Lazy loading for images (40% faster initial page load)

## 🔧 Changes

- Updated to Node.js 20 LTS
- Changed default pagination from 10 to 25 items
- Deprecated `/v1/users` endpoint (use `/v2/users` instead)

## 🚨 Breaking Changes

**None** - This is a backwards-compatible release

## 📚 Documentation

- Updated [API docs](link) with new endpoints
- Added [migration guide](link) for v2 API
- New [video tutorial](link) for dark mode setup

## 🙏 Contributors

Thanks to everyone who contributed:
@alice, @bob, @carol, and 12 others

[Full changelog](link to GitHub)
```

---

## Модуль 4: Командная работа и Conflict Resolution (25 минут)

### 🎯 Напоминалка

**Типы конфликтов в IT командах:**
```
1. Технические разногласия
   "Нам нужен Postgres!" vs "MongoDB лучше!"

2. Приоритеты
   "Сначала фичи!" vs "Сначала технический долг!"

3. Процессы
   "Слишком много митингов!" vs "Нужно больше координации!"

4. Личностные
   "Этот человек меня не слушает!"
```

**Правило ЛУРК для разрешения конфликтов:**
```
Л - Listening (Слушай активно)
У - Understanding (Понимай позицию другого)
Р - Respect (Уважай мнение, даже если не согласен)
К - Компромисс (Ищи win-win решения)
```

**Фреймворк для технических дискуссий:**
```
1. Define the Problem
   "Что мы пытаемся решить?"

2. List Requirements
   "Какие у нас критерии успеха?"

3. Propose Solutions
   "Какие есть варианты?"

4. Evaluate Trade-offs
   "Плюсы и минусы каждого варианта"

5. Make Decision
   "Выбираем [X], потому что [Y]"

6. Document (ADR!)
   "Записываем решение и причины"
```

**Техника "Disagree and Commit":**
```
Шаги:
1. Высказываешь свое мнение полностью
2. Слушаешь других
3. Если команда выбрала другое решение - commit to it
4. Не саботируй, не говори "я же говорил"
5. Если решение не сработает - учитесь все вместе

Фраза: "I disagree but I commit"
```

**Признаки токсичного поведения (как НЕ надо):**
```
❌ Постоянная критика без предложения решений
❌ "Это не моя проблема" attitude
❌ Ignoring teammates' input
❌ Blame culture ("кто виноват?")
❌ Passive-aggressive комментарии
❌ Monopolizing discussions
❌ Dismissing ideas without consideration
```

**Признаки здорового collaboration:**
```
✅ Active listening
✅ Constructive feedback
✅ Sharing knowledge openly
✅ Celebrating team wins
✅ Taking ownership collectively
✅ Asking "how can I help?"
✅ Respectful disagreement
```

**Code Review best practices (для командной работы):**
```
Для reviewer:
✅ "Consider using X instead of Y because [reason]"
✅ "This might cause [issue] when [scenario]"
✅ "Nice solution! Could we also handle [edge case]?"
✅ Ask questions: "Can you explain why you chose this approach?"

❌ "This is wrong"
❌ "Why didn't you do X?"
❌ Nitpicking style without explaining why
❌ Demanding changes without discussion

Для автора:
✅ "Good point! I'll refactor this"
✅ "I chose this because [reason]. Open to alternatives!"
✅ "Could you elaborate on the concern?"
❌ "No, my way is better"
❌ Defensive reactions
❌ Ignoring feedback without response
```

**Правило "Strong Opinions, Weakly Held":**
```
- Имей свое мнение и аргументируй его
- Но будь готов изменить мнение при новых данных
- Не цепляйся за решение из гордости
- "Change my mind" - это хорошо, не слабость
```

**1-on-1 Meeting structure:**
```
Для инженеров с тимлидом (30 мин):

1. Personal check-in (5 мин)
   "Как дела? Что тебя беспокоит?"

2. Current work discussion (10 мин)
   "Какие блокеры? Нужна помощь?"

3. Career growth (10 мин)
   "Что хочешь изучить? Какие цели?"

4. Feedback (5 мин)
   "Что я могу улучшить как лид?"
   "Вот мой feedback для тебя..."

Tips:
- Пусть direct report ведет agenda
- Это их время, не твое  
- Не только о work, но и о росте и wellbeing
- Делай notes и follow-up на action items
```

**Dealing with difficult teammates:**
```
Situation 1: Senior инженер dismissive к младшим
→ Private conversation: "I noticed X. It impacts team morale. Can we discuss?"
→ Focus on behavior, not person
→ Ask them to mentor instead of criticize

Situation 2: Team member постоянно пропускает дедлайны
→ 1-on-1: "I see you're struggling with deadlines. What's blocking you?"
→ Maybe они overloaded или не понимают приоритеты
→ Help them, не обвиняй

Situation 3: Constant negativity
→ "I understand your concerns. What would you propose to improve?"
→ Переводи с complaints на solutions
→ Если не помогает - escalate to manager
````

### 💻 Задание

**Сценарий:** Твоя команда не может решить: использовать microservices или улучшить существующий монолит. Это блокирует development уже 2 недели.

**Участники:**

- **Ты** (SysAdmin/DevOps)
- **Alice** (Senior Dev): хочет microservices, "монолит устарел"
- **Bob** (Junior Dev): боится сложности microservices
- **Carol** (Product Manager): нужны фичи ASAP

**Твоя задача:**

1. Напиши план для technical decision meeting (структура встречи)
2. Подготовь список вопросов для обсуждения
3. Создай comparison matrix (монолит vs microservices)
4. Напиши email после встречи с итоговым решением

**Пример решения:**

markdown

```markdown
## Technical Decision Meeting: Architecture Strategy

**Date:** January 17, 2025, 14:00-15:30
**Attendees:** Alice (Senior Dev), Bob (Junior Dev), Carol (PM), You (DevOps)
**Goal:** Decide on architecture direction and unblock development

### Meeting Agenda (90 min)

#### 1. Frame the Problem (10 min)
- Current state: Monolith with 50K LOC, 3 years old
- Pain points: What's NOT working now?
- Success criteria: What would "solved" look like?

#### 2. Gather Requirements (15 min)
Everyone shares their constraints:
- Alice: Technical perspective
- Bob: Team capability perspective  
- Carol: Business needs perspective
- You: Operations perspective

Key questions:
- What's our deployment frequency need?
- What's our team size/skill level?
- What's our timeline?
- What's our budget?

#### 3. Present Options (20 min)

**Option A: Improve Monolith**
- Refactor into modules
- Improve CI/CD
- Add better monitoring

**Option B: Microservices (Big Bang)**
- Split immediately
- 6-month migration
- New infrastructure

**Option C: Hybrid (Strangler Fig)**
- Extract одн service at a time
- Monolith + микросервисы параллельно
- Gradual migration

Each option: Presenter (5 min) + Q&A (2 min)

#### 4. Evaluate Trade-offs (25 min)
Go through comparison matrix together (see below)
Discuss each criterion, vote on importance (1-5)

#### 5. Make Decision (15 min)
- Summarize findings
- Check if we have consensus
- If not: Identify remaining concerns
- Vote if needed (but prefer consensus)
- Decision maker (You as DevOps lead) makes final call if no consensus

#### 6. Next Steps (5 min)
- Document decision (ADR)
- Create action items with owners
- Set follow-up meeting if needed

### Discussion Questions

**Business Questions (for Carol):**
- What's the cost of current velocity? Are we losing business?
- How critical is time-to-market for next 6 months?
- What's the risk tolerance for this quarter?
- Can we afford 6-month productivity dip during migration?

**Technical Questions (for Alice & Bob):**
- What are specific pain points with monolith? (Be concrete)
- Have we tried modularizing inside monolith first?
- What's our team's experience with microservices?
- Who will maintain 10+ services vs 1 monolith?
- Do we have problems that microservices actually solve?

**Operations Questions (for You):**
- Can our current infrastructure handle microservices?
- What's the operational overhead? (monitoring, logging, deployments)
- Do we have service mesh / API gateway experience?
- What's the learning curve for the team?
- How will
```

Continue

15:01

this affect our on-call burden?

**Team Capability Questions:**

- Current team: 1 Senior, 3 Junior - realistic?
- Who will own each microservice?
- Training needed? Time? Cost?

### Comparison Matrix

|Criterion|Weight|Monolith Refactor|Microservices (Big Bang)|Hybrid (Strangler Fig)|
|---|---|---|---|---|
|**Time to Value**|5|⭐⭐⭐⭐⭐ (2 weeks)|⭐ (6+ months)|⭐⭐⭐ (1 month start)|
|**Team Skill Match**|5|⭐⭐⭐⭐⭐ (we know this)|⭐⭐ (learning curve)|⭐⭐⭐ (gradual learning)|
|**Operational Complexity**|4|⭐⭐⭐⭐ (1 app to manage)|⭐⭐ (10+ services)|⭐⭐⭐ (grows over time)|
|**Scalability**|3|⭐⭐ (scale all or nothing)|⭐⭐⭐⭐⭐ (independent scale)|⭐⭐⭐⭐ (scale what's needed)|
|**Development Velocity (long-term)**|4|⭐⭐⭐ (better than now)|⭐⭐⭐⭐⭐ (independent teams)|⭐⭐⭐⭐ (improves over time)|
|**Risk**|5|⭐⭐⭐⭐ (low risk)|⭐⭐ (high risk)|⭐⭐⭐⭐ (managed risk)|
|**Cost**|3|⭐⭐⭐⭐⭐ ($10k infra)|⭐⭐ ($50k infra + 6mo dev)|⭐⭐⭐ ($20k + 3mo)|
|**Deployment Independence**|3|⭐ (all or nothing)|⭐⭐⭐⭐⭐ (fully independent)|⭐⭐⭐⭐ (gradual independence)|
|**Disaster Recovery**|4|⭐⭐⭐ (one point of failure)|⭐⭐⭐⭐ (isolated failures)|⭐⭐⭐ (mixed)|

**Scoring:**

- Multiply stars by weight for each option
- Sum total scores
- Highest score = best fit (но не единственный фактор!)

### Ground Rules for Discussion

1. **Respectful disagreement**
    - "I see it differently because..." not "You're wrong"
    - Focus on ideas, not people
2. **Data over opinions**
    - "Based on [metric/experience]..."
    - Avoid "I feel like..." without backing
3. **Everyone speaks**
    - Junior voices matter as much as Senior
    - Timeboxed turns if someone dominates
4. **Solutions over problems**
    - Identify concerns, but also propose mitigations
    - "This is risky AND here's how we can reduce risk"
5. **Commit to decision**
    - Once decided, everyone commits
    - No "I told you so" later
    - Re-evaluate in 3 months if needed

---

## Post-Meeting Email

Subject: [DECISION] Architecture Strategy: Hybrid Approach (Strangler Fig Pattern)

Team,

Спасибо всем за продуктивную дискуссию сегодня! Вот summary нашей встречи и итоговое решение.

### Decision Made

**We will proceed with a Hybrid Approach (Strangler Fig Pattern)**

Start with monolith improvements while gradually extracting microservices for specific high-value use cases.

### Rationale

После оценки всех опций через comparison matrix и обсуждения concerns:

**Why NOT pure monolith:**

- Alice's point valid: некоторые части (например, payment processing) действительно нужно изолировать
- We'll hit scaling issues in 6-12 months with current growth

**Why NOT big-bang microservices:**

- Bob's concern valid: team не готова поддерживать 10+ services сразу
- Carol's timeline: нельзя позволить 6-month productivity drop
- Operational complexity too high too fast для нашего текущего размера команды

**Why Hybrid/Strangler Fig:**

- Balances risk and reward
- Delivers value incrementally
- Team learns gradually
- Can course-correct based on learnings
- First wins in 4-6 weeks, not 6 months

### Implementation Plan

**Phase 1: Foundation (Weeks 1-2)**

- [ ]  Refactor monolith into better internal modules (@alice, @bob)
- [ ]  Improve CI/CD for faster deployments (@you)
- [ ]  Set up monitoring baseline (@you)
- [ ]  Document current architecture (@bob)

**Phase 2: First Extraction (Weeks 3-6)**

- [ ]  Extract Payment Service (high-value, well-defined boundary)
- [ ]  Set up API gateway
- [ ]  Establish microservice patterns/templates
- [ ]  Team training on microservice best practices

**Phase 3: Evaluate & Iterate (Week 7+)**

- [ ]  Review Phase 2 results
- [ ]  Decide on next service to extract (if successful)
- [ ]  OR pivot back to monolith if not working

### Success Metrics (Review in 6 weeks)

- ✅ Payment service extracted and stable
- ✅ No regression in deployment frequency
- ✅ Team confidence level improved
- ✅ Operational overhead manageable (< 20% increase)
- ✅ Faster payment feature iterations (measure velocity)

### Key Agreements

1. **Disagree and Commit:** Alice, I know you preferred full microservices - спасибо что поддерживаешь это решение. Bob, I know you were nervous about microservices - we'll go slow and support you.
2. **Regular Check-ins:**
    - Weekly sync during Phase 2
    - 6-week retrospective to evaluate
    - Can pivot if not working
3. **Documentation:**
    - I'll create ADR-004 documenting this decision
    - Alice will document microservice patterns as we build
    - Bob will maintain architecture diagrams

### Next Steps

1. **Immediate (This Week):**
    - I'll create ADR and share for review
    - Alice: Start designing Payment Service extraction
    - Bob: Document current monolith modules
    - Carol: Prioritize Payment features for new service
2. **Kick-off Meeting:**
    - Monday, Jan 20, 10:00 AM
    - Detailed planning for Phase 1

### Questions?

Если есть concerns или questions - пишите в #architecture или DM me.

Спасибо за constructive discussion и willingness to find best solution together!

Let's build this! 🚀

Best, [Your name]

---

## Приложение: Что делать если решение не работает

Договорились заранее:

**If after 6 weeks:**

- Payment service extraction failed or too costly
- Team overwhelmed with operational complexity
- Business velocity decreased significantly

**Then:**

- Встречаемся снова
- Честно анализируем что пошло не так
- Pivot: либо продолжить с learning, либо вернуться к монолиту
- No blame - это был informed риск, который мы together решили взять

**Decision review date:** March 1, 2025

```

### 🚀 Бонус (новое)

**Техника "Pre-Mortem" для командных решений:**

Перед началом большого проекта, представь что он провалился. Спроси команду: "It's 6 months from now, and this project failed spectacularly. What went wrong?"
```

Примеры ответов:

- "Мы недооценили operational complexity"
- "Team burnout из-за oncall burden"
- "Database migration сломала production"
- "Мы не обучили команду вовремя"

Результат: Идентифицируй риски до того как они материализуются

```

**Conflict Resolution Script:**
```

Когда конфликт с коллегой, используй эту структуру:

1. "Я заметил [конкретное поведение]" "I noticed you dismissed my suggestion in the meeting"
2. "Это заставило меня почувствовать [эмоция]"  
    "It made me feel like my input isn't valued"
3. "Я предполагаю что [твоя интерпретация]" "I'm guessing you were focused on timeline concerns"
4. "Но возможно [альтернативная причина]" "But maybe I'm misunderstanding the situation"
5. "Можем обсудить?" "Can we talk about this?"

Избегай: ❌ "You always ignore me" ❌ "You're being disrespectful" ❌ "You don't care about my opinion"

```

**Retrospective Format (для команды):**
```

Sprint/Project Retrospective (45 min):

1. Set the Stage (5 min)
    - Reminder: Safe space, no blame
    - Focus on improvement, not criticism
2. Gather Data (10 min)
    - What went well? (Green sticky notes)
    - What didn't go well? (Red sticky notes)
    - What questions/puzzles do we have? (Yellow sticky notes)
3. Generate Insights (15 min)
    - Group similar items
    - Vote on top 3 items to discuss
    - Discuss root causes (5 Whys)
4. Decide Actions (10 min)
    - Pick 1-3 actionable improvements (not 10!)
    - Assign owners
    - Set timeline
5. Close (5 min)
    - Appreciation round: Each person thanks someone
    - Commit to actions

Template for action items: "We will [action] by [date] to improve [outcome], owned by [person]"



---

## Модуль 5: Time Management и Prioritization (20 минут)

### 🎯 Напоминалка

**Eisenhower Matrix для DevOps:**
```
                URGENT (СРОЧНО)
          YES          |          NO
    +==================+===================+
    | 🔥 КВАДРАНТ I    | 📅 КВАДРАНТ II   |
    | DO NOW           | SCHEDULE          |
    | (СДЕЛАТЬ СЕЙЧАС) | (ЗАПЛАНИРОВАТЬ)  |
I   |------------------+-------------------|
M   | • P0-инциденты   | • Проекты         |
P   | • Критический    | • Tech Debt &     |
O   |   сбой продакшена|   рефакторинг     |
R   | • Критичные      | • Обучение / Изу- |
T   |   security alerts|   чение нового    |
A   | • Блокирующие    | • Автоматизация   |
N   |   баги в релизе  |   рутины (цель)   |
T   | • Атаки / Утечки | • Улучшение       |
    |   данных         |   мониторинга     |
    |                  | • Документация    |
    |                  | • Пентесты, аудит |
    +==================+===================+
    | ⚡ КВАДРАНТ III   | 🗑️ КВАДРАНТ IV   |
    | DELEGATE         | ELIMINATE         |
    | (ДЕЛЕГИРОВАТЬ)   | (УСТРАНИТЬ)       |
N   |------------------+-------------------|
O   | • Рутинные задачи| • "Занятость ради|
    |   (резерв. копии)|   занятости"      |
T   | • Мониторинг     | • Избыточные      |
    |   (первичный     |   совещания       |
    |   анализ)        | • Низкие задачи   |
U   | • Стандартные    |   с малым ROI     |
R   |   тикеты         | • Устаревшие      |
G   | • Шаблонные      |   процессы/       |
E   |   запросы        |   отчеты          |
N   | • Обучение       | • Ручные повторя- |
T   |   junior-ов      |   ющиеся действия |
    +------------------+-------------------+
```
**Правило "Maker's Schedule" для инженеров:**
```

Проблема: Meetings разбивают день на маленькие куски

Решение:

- Block 4-hour chunks для deep work
- Schedule meetings back-to-back (не разбросаны по дню)
- "No meeting" days (например, каждый вторник)
- Communicate boundaries: "Я доступен для meetings только утром"

Пример дня: 09:00-10:00: Meetings 10:00-14:00: Deep work block (NO INTERRUPTIONS) 14:00-14:30: Lunch 14:30-18:00: Deep work block 18:00-18:30: Email/Slack catchup
```



**Техника Pomodoro (адаптированная для DevOps):**
```

Standard: 25 min work + 5 min break

DevOps version:

- 50 min deep work
- 10 min break (check Slack, alerts, email)
- После 4 cycles: 30 min longer break

Perfect для:

- Writing runbooks
- Debugging complex issues
- Learning new technology
- Code reviews

Не для:

- Incidents (нельзя делать break!)
- Collaborative work
- Meetings

```
**Say "No" gracefully:**

```

❌ "No, I'm too busy" ❌ "No, that's not my job"

✅ "I'd love to help, but I'm currently focused on [X]. Can this wait until [date]?"

✅ "This sounds important. Given my current commitments [list], which should I deprioritize to take this on?"

✅ "I can't do the full project, but I can [smaller thing]. Would that help?"

✅ "Let me check with my manager about priorities and get back to you."

```


**On-call time management:**

```
Проблема: On-call прерывает planned work

Strategies:

1. Light Tasks During On-call Week
    - Documentation
    - Code reviews
    - Small bug fixes
    - NOT: Critical projects, complex refactoring
2. Handoff Protocol
    - 15-min handoff meeting between on-call shifts
    - Document ongoing issues
    - Share context для smooth transition
3. Post-Incident Recovery
    - Block 2 hours after major incident для recovery
    - Don't jump into next task immediately
    - Write post-mortem while context fresh
4. On-call Preparation
    - Day before: Review runbooks
    - Check monitoring dashboards
    - Ensure laptop charged, internet backup ready

```



**Ticket Triage система:**
```

Daily ticket review (15 min):

P0 (Do Now):

- Production down
- Security breach
- Data loss

P1 (Today):

- Service degraded
- Customer escalations
- Urgent fixes

P2 (This Week):

- Standard bugs
- Feature requests
- Improvements

P3 (Backlog):

- Nice-to-haves
- Tech debt
- Long-term ideas

Close/Redirect:

- Duplicates
- Out of scope
- Not actionable
```


**Правило "2-Minute Rule":**
```

If a task takes < 2 minutes: ✅ DO IT NOW (не откладывай)

Examples:

- Quick Slack response
- Approve simple PR
- Restart failing service
- Update Jira ticket status

If task takes > 2 minutes: → Add to proper queue/schedule → Don't context-switch

Saves mental overhead of tracking tiny tasks

```

**Batching similar tasks:**
```

Instead of:

- Check email every 10 minutes
- Review PRs as they come
- Respond to Slack constantly

Try:

- Email: 3 times per day (9am, 1pm, 5pm)
- PRs: 2 batches (morning, afternoon)
- Slack: Check every hour, respond in batch

Reduces context switching Improves focus Faster overall completion

```

### 💻 Задание

**Сценарий:** Твой типичный понедельник выглядит так:

Inbox:

- 47 unread emails
- 25 Slack messages
- 12 Jira tickets assigned to you

Calendar:

- 09:00-09:30: Standup
- 10:00-11:00: Planning meeting
- 14:00-14:30: 1-on-1 with manager
- 15:00-16:00: Architecture review

Your commitments:

- Production deployment (due today!)
- Security patch (critical, due this week)
- Documentation update (requested by PM)
- Help новый сотрудник с onboarding

Unexpected:

- 08:30: Slack message: "Production API slow!"
- 11:15: Email from CEO: "Can you check why report failed?"
- 13:00: Teammate asks: "Can you review my PR?" (2000 lines of code)


**Твоя задача:**
1. Prioritize tasks using Eisenhower Matrix
2. Create realistic schedule для этого дня
3. Identify tasks to delegate/defer/decline
4. Write responses для 3 requests you can't do today

**Пример решения:**

## Task Prioritization (Eisenhower Matrix)

### 🔥 DO NOW (Urgent + Important)
1. **08:30 - Production API slow**
   - Impact: Customer-facing
   - Action: Investigate immediately
   - Est time: 30-60 min

2. **Production deployment (due today)**
   - Committed deadline
   - Must happen during deploy window
   - Est time: 60 min (with testing)

3. **Security patch (critical)**
   - Can't wait until end of week
   - Security risk
   - Est time: 90 min

### 📅 SCHEDULE (Important, Not Urgent)
1. **Documentation update**
   - Important but not today-critical
   - → Schedule for Wednesday

2. **Architecture review meeting (15:00)**
   - Keep it (already scheduled)

3. **1-on-1 with manager (14:00)**
   - Keep it (important for context)

### ⚡ DELEGATE (Urgent, Less Important)
1. **CEO email: "Report failed"**
   - Probably not DevOps issue
   - → Forward to data team
   - Provide quick triage first

2. **Help new employee onboarding**
   - Important but not you-specific
   - → Ask teammate or assign buddy

### 🗑️ DEFER/DECLINE (Not Urgent, Not Important Today)
1. **2000-line PR review**
   - Too big for today
   - → Ask for smaller PRs or schedule tomorrow

2. **Standup (09:00)**
   - Optional: Send async update instead?
   - Frees up 30 min if critical work

3. **Planning meeting (10:00)**
   - Consider: Really needed today?
   - Could send delegate if firefighting

## Realistic Daily Schedule

### 08:00-08:30: Morning Setup

- Quick email scan (5 min) - identify anything P0
- Slack scan (5 min) - urgent alerts only
- Review calendar for conflicts
- Prepare for standup update

ACTION:
- Saw "Production API slow" alert
- This becomes priority #1


### 08:30-09:30: INCIDENT - Production API Slow ⚠️

```
- Investigate issue
- Found: Database connection pool exhausted
- Quick fix: Restart service
- Monitor recovery
- Document in incident log

RESULT:
- Issue resolved in 30 min
- Normal standup might be delayed
```

### 09:00-09:15: Standup (ASYNC)

```
Send Slack update instead of attending:

"Morning team! Quick update:

Yesterday: ✅ Completed monitoring dashboard updates
Today: 🔥 Production API issue (resolved, monitoring)
       🚀 Production deployment scheduled 16:00
       🔒 Security patch work
Blockers: None

Skipping standup due to production incident recovery.
Available on Slack if needed."

SAVED: 15 minutes
```

### 09:15-09:30: Incident Follow-up

```
- Post incident update in #incidents
- Create Jira for proper fix (connection pool sizing)
- Quick Post-Mortem notes
```

### 09:30-11:00: Security Patch (Deep Work Block)

```
NO INTERRUPTIONS (Close Slack, email)
- Apply security patch
- Test in staging
- Document changes
- Create deployment plan

BUFFER: 30 min in case of issues
```

### 11:00-11:15: Break & Email Triage

```
- Process emails (quick responses only)
- Check Slack for urgent items
- CEO email about report → Forward to data team with context
```

### 11:15-12:00: CEO Report Issue (Quick Triage)

```
NOT your job, but CEO asked, so:
- 15 min: Quick investigation
- Identify it's data pipeline issue
- Email response (see below)
- Forward to data team lead
- Done

Time saved by NOT deep-diving: 2+ hours
```

### 12:00-13:00: LUNCH (Actual Break!)

```
- Step away from computer
- No work talk
- Recharge for afternoon
```

### 13:00-13:15: PR Review Request Response

```
Teammate: "Can you review my PR?" (2000 lines)

Response (see below)
- Can't do today
- Offer alternative
- Set expectations

Time saved: 1-2 hours
```

### 13:15-14:00: Deployment Preparation

```
- Review deployment runbook
- Check monitoring dashboards
- Prepare rollback plan
- Coordinate with team
```

### 14:00-14:30: 1-on-1 with Manager

```
KEEP THIS - важный context sharing
- Update on incident
- Discuss prioritization challenges
- Get alignment on what to defer
- Ask for help delegating new hire onboarding
```

### 14:30-15:00: Production Deployment (Part 1)

```
- Pre-deployment checks
- Create announcement
- Start deployment process
```

### 15:00-16:00: Architecture Review Meeting

```
EVALUATE: Do I need to be here?
- If deployment in progress → Skip or send notes
- If someone else can represent DevOps → Delegate

DECISION: Stay (because deployment automated, just monitoring)
- Participate in first 30 min
- Monitor deployment in parallel
```

### 16:00-16:30: Production Deployment (Part 2)

```
- Verify deployment success
- Run smoke tests
- Post completion message
- Monitor for issues
```

### 16:30-17:00: Post-Deployment + Day Wrap

```
- Document deployment notes
- Update incident notes
- Plan tomorrow's priorities
- Process remaining Slack/Email (final batch)
```

### 17:00-17:30: Async Work / Buffer

```
- Catch up on deferred items if time
- OR: Leave early if everything done (self-care!)
```

## Delegation & Communication

### 1. CEO Report Email - DELEGATE

```
Subject: Re: Report Failure - Forwarding to Data Team

Hi [CEO Name],

Я быстро проверил issue с отчетом. Это выглядит как проблема в data pipeline, а не в infrastructure.

QUICK FINDINGS:
- Infrastructure/services: Все работает нормально
- Database: Healthy
- Likely cause: ETL job failed на data ingestion step

NEXT STEPS:
- Forwarded to Data Team Lead (@data-lead)
- They're best positioned to debug data pipeline issues
- Copied you on that email
- They'll update you directly

If it turns out to be infrastructure issue, I'm available to help.

Best,
[Your name]
```

**Почему это хороший response:**

- Показываешь что среагировал быстро
- Даешь preliminary findings
- Направляешь к правильному owner
- Остаешься available если нужно
- Не берешь на себя не свою работу

### 2. PR Review - DEFER with Alternative

```
Hi [Teammate],

Спасибо за PR! I see it's 2000 lines - это большой review.

TIMELINE:
К сожалению, сегодня у меня critical deployment + security patch,
и не смогу сделать качественный review такому большому PR.

ALTERNATIVES:
1. ✅ I can review it tomorrow morning (first thing)
2. ✅ If urgent: попроси @alice (она свободнее сегодня)
3. ✅ If можешь: разбей на smaller PRs (300-400 lines each)
   → Это и review быстрее, и merge safer

Smaller PRs = faster reviews для всех!

What works better для тебя?

[Your name]
```

**Почему это хороший response:**

- Объясняешь причину (не просто "no")
- Предлагаешь alternatives
- Учишь best practice (smaller PRs)
- Остаешься helpful

### 3. New Employee Onboarding - DELEGATE

```
Slack to Team Lead:

Hey [Lead],

New teammate [Name] needs onboarding help, but я сегодня overloaded 
(production incident + critical deployment).

REQUEST:
Можем ли назначить [Name] buddy из команды? 

SUGGESTION:
- @bob был бы отличный buddy (недавно onboarded, помнит процесс)
- Или @carol (хорошо объясняет, patient)

Я могу сделать 30-min tech overview завтра или в среду.

Сейчас просто не смогу дать качественный onboarding из-за fires.

Thoughts?
```

**Почему это хороший response:**

- Признаешь request важный
- Предлагаешь specific alternatives
- Commit к тому что можешь сделать
- Escalate к тому кто может delegating

## Lessons Learned

### ✅ What Worked

1. **Eisenhower Matrix помог prioritize**
    - Фокус на P0 items first
    - Defer non-urgent tasks без guilt
2. **Async standup сэкономил 15 минут**
    - В critical days, это OK
    - Team понимает
3. **Delegation освободил 3+ hours**
    - CEO email: 15 min вместо 2 hours
    - PR review: defer вместо rush job
    - Onboarding: делегировать подходящему человеку
4. **Deep work blocks**
    - Security patch сделан качественно
    - Не interrupted каждые 10 минут

### 🔄 What to Improve

1. **Better communication раньше в день**
    - Мог предупредить PM что documentation отложена
    - Раньше попросить help с onboarding
2. **Block calendar для deep work**
    - Marking "Busy" для focus time
    - Prevents random meetings
3. **More ruthless prioritization**
    - Возможно стоило skip Architecture review если deployment критичен

### 📊 Metrics

- **Planned tasks completed:** 4/6 (67%)
- **Critical tasks completed:** 3/3 (100%) ← Это важнее!
- **Hours saved via delegation:** ~3 hours
- **Incidents handled:** 1 (resolved quickly)
- **Burnout risk:** Medium (busy day, but managed)

### 🎯 Tomorrow's Priorities

1. Security patch deployment to production
2. PR review (promised to teammate)
3. Documentation update
4. Incident Post-Mortem
5. Connection pool sizing fix (follow-up from incident)



### 🚀 Бонус (новое)

**Техника "Time Blocking" в Calendar:**
```
Блокируй календарь для защиты своего времени:

Monday schedule: 
09:00-09:30: 📧 Email & Planning 
09:30-13:00: 🔒 DEEP WORK (No meetings, blocked) 
13:00-14:00: 🍽️ Lunch 
14:00-15:00: 👥 Meetings allowed 
15:00-17:00: 🔒 DEEP WORK 
17:00-17:30: 📧 Email wrap-up

Color code:

- 🔴 Deep work (Do not disturb)
- 🟡 Meetings (available)
- 🟢 Flexible (can move if needed)
- ⚫ Personal (non-negotiable)

```

**"Energy Management" не только "Time Management":**
```

Track свою энергию в течение дня:

High Energy Tasks (утро для меня):

- Complex debugging
- Architecture decisions
- Important writing (docs, proposals)
- Learning new tech

Low Energy Tasks (после обеда):

- Email responses
- Code reviews (routine)
- Meetings (less decision-making)
- Administrative work

Schedule соответственно!

```

**Weekly Review процесс (30 минут каждую пятницу):**
```markdown
# Weekly Review Template

## Week of: [Date]

### 🎯 Wins This Week
- What went well?
- What am I proud of?
- What did I learn?

### 😓 Challenges
- What was difficult?
- What took longer than expected?
- What frustrated me?

### 📊 Time Analysis
- Hours in meetings: X
- Hours in deep work: Y
- Hours firefighting: Z
- On-call incidents: N

### 🔄 Process Improvements
- What slowed me down?
- What can I automate?
- What should I delegate?
- What should I stop doing?

### 📅 Next Week Planning
Top 3 priorities:
1. [Priority 1]
2. [Priority 2]
3. [Priority 3]

Time blocks needed:
- [ ] Monday: X hours for [task]
- [ ] Wednesday: Y hours for [task]

### 🏃 Action Items
- [ ] Schedule time for [X]
- [ ] Delegate [Y] to [person]
- [ ] Automate [Z] process
````

---

## Модуль 6: Непрерывное обучение и профессиональный рост (20 минут)

### 🎯 Напоминалка

**Правило "T-Shaped" skillset:**
````

```
    Deep expertise (↓)
          |
Broad knowledge (→→→→→)
```

Пример для DevOps: Deep (специализация):

- Kubernetes orchestration
- или CI/CD pipelines
- или Database optimization

Broad (understanding):

- Cloud platforms (AWS, Azure, GCP)
- Monitoring & observability
- Security practices
- Networking basics
- Development workflows

```

**20% Time для learning (Google rule):**
```

Выдели 1 день в неделю (или 4 hours) для:

- Learning новых технологий
- Experiments
- Side projects
- Reading documentation
- Contributing to open source
- Writing blog posts

Примеры:

- "Пятничные эксперименты"
- "Tool Tuesday" - каждый вторник пробуем новый tool
- "Doc Wednesday" - читаем documentation hour

```

**Learning Path Structure:**
```

1. Foundation (20%)
    - Understand the "why"
    - Core concepts
    - Watch overview videos
2. Hands-on Practice (60%)
    - Build real projects
    - Break things, fix things
    - Follow tutorials, then modify them
3. Deep Dive (15%)
    - Read official docs
    - Understand internals
    - Learn advanced features
4. Teaching Others (5%)
    - Write blog post
    - Present to team
    - Mentor colleague

Teaching = Best way to solidify knowledge

```

**Staying Current стратегия:**
```

Weekly (1-2 hours):

- [ ]  Read Hacker News top posts
- [ ]  Check /r/devops, /r/sysadmin
- [ ]  Skim newsletters (DevOps Weekly, SRE Weekly)

Monthly (2-3 hours):

- [ ]  Try one new tool/technology
- [ ]  Read 1 technical blog post deep
- [ ]  Watch 1 conference talk

Quarterly (1 day):

- [ ]  Attend conference or meetup
- [ ]  Complete online course
- [ ]  Contribute to open source project

Yearly (1 week):

- [ ]  Major skill acquisition (new language, framework)
- [ ]  Certification if valuable
- [ ]  Review and update personal learning goals

````

**Personal Learning Backlog:**
```markdown
# My Learning Backlog

## In Progress (actively learning)
- [ ] Kubernetes CKA certification (Est: 2 months)
- [ ] Terraform deep dive (Est: 3 weeks)

## Next Up (priority queue)
1. [ ] Service mesh (Istio/Linkerd) basics
2. [ ] Advanced observability (eBPF, OpenTelemetry)
3. [ ] Go programming language

## Someday/Maybe (interesting but not urgent)
- Rust programming
- Machine Learning for ops
- Game
````
