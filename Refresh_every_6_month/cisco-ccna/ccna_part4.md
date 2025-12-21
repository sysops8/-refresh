# CCNA Мини-курс на основе Jeremy's Mega Lab
## Часть 4: Rapid Spanning Tree Protocol (RSTP)

---

## 📋 Теоретический минимум

### Ключевые концепции:

**1. Spanning Tree Protocol (STP)**
- Предотвращение петель в сети Layer 2
- Блокировка избыточных путей
- IEEE 802.1D (классический STP) → 802.1W (Rapid STP)

**2. PVST+ vs Rapid PVST+**
- **PVST+** - Per-VLAN Spanning Tree Plus (Cisco, медленный)
- **Rapid PVST+** - Быстрая сходимость (секунды вместо ~30-50 сек)
- Отдельный экземпляр STP для каждого VLAN

**3. Root Bridge (Корневой мост)**
- Референсная точка для всей топологии STP
- Выбирается по наименьшему Bridge ID
- Bridge ID = Priority (default 32768) + VLAN ID + MAC address

**4. STP Priority**
- Диапазон: 0-61440
- Шаг: 4096
- Lowest priority = Root Bridge

**5. Port States (RSTP)**
- **Discarding** - блокирует трафик
- **Learning** - изучает MAC адреса
- **Forwarding** - передает трафик

**6. PortFast**
- Порт сразу переходит в Forwarding
- Только для портов с конечными устройствами
- Ускоряет подключение устройств

**7. BPDU Guard**
- Защита от петель
- Отключает порт при получении BPDU
- Защита от подключения неавторизованных коммутаторов

---

## 🎯 Цели раздела

- ✅ Включить Rapid PVST+ на всех коммутаторах
- ✅ Выровнять Root Bridge с HSRP Active роутерами
- ✅ Настроить правильные STP приоритеты
- ✅ Включить PortFast на access портах
- ✅ Включить BPDU Guard для защиты

---

## 🗺️ STP Design для лабы

### Office A - Root Bridge Alignment:
- **VLAN 10, 99**: DSW-A1 = Root (Priority 0), DSW-A2 = Secondary (Priority 4096)
- **VLAN 20, 40**: DSW-A2 = Root (Priority 0), DSW-A1 = Secondary (Priority 4096)

### Office B - Root Bridge Alignment:
- **VLAN 10, 99**: DSW-B1 = Root (Priority 0), DSW-B2 = Secondary (Priority 4096)
- **VLAN 20, 30**: DSW-B2 = Root (Priority 0), DSW-B1 = Secondary (Priority 4096)

💡 **Почему выравнивание с HSRP?**
- HSRP Active = Root Bridge
- Оптимизация маршрутизации трафика
- Предотвращение субоптимальных путей

---

## 📝 ЧАСТЬ 1: Enable Rapid PVST+

### ШАГ 1: Включение Rapid PVST+ на всех коммутаторах

**По умолчанию в Packet Tracer:**
- Коммутаторы работают в режиме **PVST+** (classic STP)
- Нужно переключить на **Rapid PVST+**

---

### Distribution Switches

```cisco
! DSW-A1
DSW-A1(config)# spanning-tree mode rapid-pvst
DSW-A1(config)# exit

! DSW-A2
DSW-A2(config)# spanning-tree mode rapid-pvst
DSW-A2(config)# exit

! DSW-B1
DSW-B1(config)# spanning-tree mode rapid-pvst
DSW-B1(config)# exit

! DSW-B2
DSW-B2(config)# spanning-tree mode rapid-pvst
DSW-B2(config)# exit
```

---

### Access Switches

```cisco
! ASW-A1
ASW-A1(config)# spanning-tree mode rapid-pvst

! ASW-A2
ASW-A2(config)# spanning-tree mode rapid-pvst

! ASW-A3
ASW-A3(config)# spanning-tree mode rapid-pvst

! ASW-B1
ASW-B1(config)# spanning-tree mode rapid-pvst

! ASW-B2
ASW-B2(config)# spanning-tree mode rapid-pvst

! ASW-B3
ASW-B3(config)# spanning-tree mode rapid-pvst
```

**Проверка:**
```cisco
DSW-A1# show spanning-tree
```

Должно показать: `Spanning tree enabled protocol rstp`

---

## 📝 ЧАСТЬ 2: STP Priority Configuration

### ШАГ 1a: Office A - Root Bridge Priority

**Концепция:**
- DSW-A1 = Root для VLAN 10, 99 (Priority 0)
- DSW-A2 = Root для VLAN 20, 40 (Priority 0)
- Secondary Root = Priority 4096

---

### DSW-A1 Configuration

```cisco
DSW-A1(config)# spanning-tree vlan 10,99 priority 0
DSW-A1(config)# spanning-tree vlan 20,40 priority 4096
DSW-A1(config)# exit
```

**Проверка:**
```cisco
DSW-A1# show spanning-tree vlan 10
```

Должно показать: `This bridge is the root`

```cisco
DSW-A1# show spanning-tree vlan 20
```

Должно показать приоритет 4096.

---

### DSW-A2 Configuration

```cisco
DSW-A2(config)# spanning-tree vlan 20,40 priority 0
DSW-A2(config)# spanning-tree vlan 10,99 priority 4096
DSW-A2(config)# exit
```

**Проверка:**
```cisco
DSW-A2# show spanning-tree vlan 20
```

Должно показать: `This bridge is the root`

---

### ШАГ 1b: Office B - Root Bridge Priority

### DSW-B1 Configuration

```cisco
DSW-B1(config)# spanning-tree vlan 10,99 priority 0
DSW-B1(config)# spanning-tree vlan 20,30 priority 4096
DSW-B1(config)# exit
```

**Проверка:**
```cisco
DSW-B1# show spanning-tree vlan 10
```

---

### DSW-B2 Configuration

```cisco
DSW-B2(config)# spanning-tree vlan 20,30 priority 0
DSW-B2(config)# spanning-tree vlan 10,99 priority 4096
DSW-B2(config)# exit
```

**Проверка:**
```cisco
DSW-B2# show spanning-tree vlan 30
```

**Сохранение конфигурации:**
```cisco
DSW-A1# write memory
DSW-A2# write memory
DSW-B1# write memory
DSW-B2# write memory
```

---

## 📝 ЧАСТЬ 3: PortFast Configuration

### ШАГ 2: Включение PortFast и BPDU Guard

**Где настраивать:**
- Все access порты, подключенные к конечным устройствам
- F0/1 на всех Access switches
- F0/2 на ASW-A1 (WLC1 trunk)

**Почему PortFast?**
- Порт сразу переходит в Forwarding
- Нет ожидания 30 секунд
- Ускоряет подключение ПК, телефонов, серверов

**Почему BPDU Guard?**
- Защита от петель
- Отключает порт если получен BPDU
- Защита от неавторизованных коммутаторов

---

### ASW-A1: LWAP + WLC

```cisco
ASW-A1(config)# interface f0/1
ASW-A1(config-if)# spanning-tree portfast
ASW-A1(config-if)# spanning-tree bpduguard enable
ASW-A1(config-if)# exit

ASW-A1(config)# interface f0/2
ASW-A1(config-if)# spanning-tree portfast trunk
ASW-A1(config-if)# spanning-tree bpduguard enable
ASW-A1(config-if)# exit
ASW-A1(config)# do write
```

**⚠️ Важно:** 
- F0/2 - trunk порт к WLC1
- Используем `spanning-tree portfast trunk`
- Обычный `portfast` не работает на trunk

---

### ASW-A2, A3, B2: PC + Phone

```cisco
! ASW-A2
ASW-A2(config)# interface f0/1
ASW-A2(config-if)# spanning-tree portfast
ASW-A2(config-if)# spanning-tree bpduguard enable
ASW-A2(config-if)# exit
ASW-A2(config)# do write

! ASW-A3
ASW-A3(config)# interface f0/1
ASW-A3(config-if)# spanning-tree portfast
ASW-A3(config-if)# spanning-tree bpduguard enable
ASW-A3(config-if)# exit
ASW-A3(config)# do write

! ASW-B2
ASW-B2(config)# interface f0/1
ASW-B2(config-if)# spanning-tree portfast
ASW-B2(config-if)# spanning-tree bpduguard enable
ASW-B2(config-if)# exit
ASW-B2(config)# do write
```

---

### ASW-B1: LWAP

```cisco
ASW-B1(config)# interface f0/1
ASW-B1(config-if)# spanning-tree portfast
ASW-B1(config-if)# spanning-tree bpduguard enable
ASW-B1(config-if)# exit
ASW-B1(config)# do write
```

---

### ASW-B3: Server

```cisco
ASW-B3(config)# interface f0/1
ASW-B3(config-if)# spanning-tree portfast
ASW-B3(config-if)# spanning-tree bpduguard enable
ASW-B3(config-if)# exit
ASW-B3(config)# do write
```

---

## ✅ Проверка конфигурации

### 1. Проверка режима Spanning Tree

```cisco
DSW-A1# show spanning-tree
```

**Ожидаемый вывод:**
```
Spanning tree enabled protocol rstp
```

Если показывает `ieee` - это PVST+, нужно изменить на `rapid-pvst`.

---

### 2. Проверка Root Bridge

```cisco
DSW-A1# show spanning-tree vlan 10
```

**Ожидаемый вывод:**
```
VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    10
             Address     [MAC адрес DSW-A1]
             This bridge is the root
```

---

### 3. Проверка Priority

```cisco
DSW-A1# show spanning-tree vlan 10 | include Priority
```

**Ожидаемый вывод:**
```
Root ID    Priority    10
           (priority 0 sys-id-ext 10)
Bridge ID  Priority    10
           (priority 0 sys-id-ext 10)
```

💡 **Примечание:** Priority = Base Priority (0) + VLAN ID (10) = 10

---

### 4. Проверка PortFast

```cisco
ASW-A1# show spanning-tree interface f0/1 detail
```

**Ожидаемый вывод:**
```
Port [номер] (FastEthernet0/1) of VLAN0099 is forwarding
  Port path cost 19, Port priority 128, Port Identifier 128.[номер]
  Designated root has priority 99, address [MAC]
  ...
  The port is in the portfast mode
  Bpdu guard is enabled
```

---

### 5. Проверка BPDU Guard

```cisco
ASW-A1# show spanning-tree interface f0/1 detail | include Bpdu
```

**Ожидаемый вывод:**
```
Bpdu guard is enabled
```

---

### 6. Общая проверка STP

```cisco
DSW-A1# show spanning-tree summary
```

**Ожидаемый вывод:**
```
Switch is in rapid-pvst mode
Root bridge for: VLAN0010, VLAN0099
Extended system ID           : enabled
Portfast Default             : disabled
PortFast BPDU Guard Default  : disabled
...
```

---

## 📊 STP Priority Summary Tables

### Office A - STP Priority

| Switch | VLAN 10 | VLAN 20 | VLAN 40 | VLAN 99 | Role |
|--------|---------|---------|---------|---------|------|
| DSW-A1 | 0 (Root) | 4096 | 4096 | 0 (Root) | Primary Gw |
| DSW-A2 | 4096 | 0 (Root) | 0 (Root) | 4096 | Secondary Gw |

### Office B - STP Priority

| Switch | VLAN 10 | VLAN 20 | VLAN 30 | VLAN 99 | Role |
|--------|---------|---------|---------|---------|------|
| DSW-B1 | 0 (Root) | 4096 | 4096 | 0 (Root) | Primary Gw |
| DSW-B2 | 4096 | 0 (Root) | 0 (Root) | 4096 | Secondary Gw |

---

## 📊 PortFast Configuration Summary

| Switch | Interface | Type | PortFast | BPDU Guard | Device |
|--------|-----------|------|----------|------------|--------|
| ASW-A1 | F0/1 | Access | ✅ | ✅ | LWAP1 |
| ASW-A1 | F0/2 | Trunk | ✅ (trunk) | ✅ | WLC1 |
| ASW-A2 | F0/1 | Access | ✅ | ✅ | PC1+Phone1 |
| ASW-A3 | F0/1 | Access | ✅ | ✅ | PC2+Phone2 |
| ASW-B1 | F0/1 | Access | ✅ | ✅ | LWAP2 |
| ASW-B2 | F0/1 | Access | ✅ | ✅ | PC3+Phone3 |
| ASW-B3 | F0/1 | Access | ✅ | ✅ | SRV1 |

---

## 💡 Практические советы

### STP Best Practices:
1. ✅ Всегда выравнивайте Root Bridge с L3 gateway (HSRP Active)
2. ✅ Используйте Rapid PVST+ вместо PVST+
3. ✅ Настройте Secondary Root (Priority 4096)
4. ✅ PortFast только на access портах
5. ✅ BPDU Guard - обязательно с PortFast
6. ✅ Root Guard на uplink портах (advanced)

### Типичные ошибки:
❌ Забыть `spanning-tree mode rapid-pvst`  
❌ Неправильный priority (не кратен 4096)  
❌ PortFast на uplink портах (опасно!)  
❌ BPDU Guard без PortFast  
❌ `portfast` на trunk порту (используйте `portfast trunk`)  
❌ Забыть сохранить конфигурацию  

---

## 🧪 Тестирование STP

### Тест 1: Проверка Root Bridge

```cisco
! На DSW-A1
DSW-A1# show spanning-tree vlan 10 | include Root
DSW-A1# show spanning-tree vlan 99 | include Root

! Должно показать "This bridge is the root"
```

---

### Тест 2: Failover Test

```cisco
! На Root Bridge отключить порт
DSW-A1(config)# interface g1/0/1
DSW-A1(config-if)# shutdown

! Проверить на Access switch
ASW-A1# show spanning-tree vlan 10

! Включить обратно
DSW-A1(config-if)# no shutdown
```

---

### Тест 3: BPDU Guard Test

```cisco
! Подключить коммутатор к access порту
! Порт должен автоматически отключиться

ASW-A1# show interfaces status | include err-disabled

! Для восстановления порта:
ASW-A1(config)# interface f0/1
ASW-A1(config-if)# shutdown
ASW-A1(config-if)# no shutdown
```

---

### Тест 4: PortFast Speed Test

1. Отключите ПК от порта
2. Подключите обратно
3. ПК должен получить IP адрес сразу (без 30 сек ожидания)

---

## 🎓 Ключевые команды Части 4

```cisco
# Enable Rapid PVST+
spanning-tree mode rapid-pvst

# STP Priority
spanning-tree vlan [vlan-list] priority [0-61440]

# PortFast (Access port)
interface [type] [number]
 spanning-tree portfast
 spanning-tree bpduguard enable

# PortFast (Trunk port)
interface [type] [number]
 spanning-tree portfast trunk
 spanning-tree bpduguard enable

# Verification Commands
show spanning-tree
show spanning-tree vlan [vlan-id]
show spanning-tree interface [type] [number]
show spanning-tree interface [type] [number] detail
show spanning-tree summary
show interfaces status | include err-disabled
```

---

## 📖 Теория STP Priority Calculation

### Bridge Priority Calculation:
```
Bridge Priority = Base Priority + VLAN ID

Пример для VLAN 10:
Base Priority: 0
VLAN ID: 10
Bridge Priority: 10

Пример для VLAN 99:
Base Priority: 4096
VLAN ID: 99
Bridge Priority: 4195
```

### Priority Values (кратны 4096):
- 0
- 4096
- 8192
- 12288
- 16384
- ...
- 61440

---

## 🔍 Troubleshooting STP

### Проблема: Root Bridge не тот коммутатор

**Решение:**
```cisco
! Проверить текущий Root
show spanning-tree vlan [id]

! Установить правильный priority
spanning-tree vlan [id] priority 0

! Проверить снова
show spanning-tree vlan [id]
```

---

### Проблема: Порт в err-disabled из-за BPDU Guard

**Решение:**
```cisco
! Проверить статус
show interfaces status | include err

! Восстановить порт
interface [type] [number]
 shutdown
 no shutdown

! Или автоматическое восстановление
errdisable recovery cause bpduguard
errdisable recovery interval 30
```

---

### Проблема: Медленная сходимость STP

**Причина:** Используется PVST+ вместо Rapid PVST+

**Решение:**
```cisco
spanning-tree mode rapid-pvst
```

---

## 🎯 Чек-лист завершения Части 4

- [ ] Rapid PVST+ включен на всех коммутаторах
- [ ] DSW-A1: Root для VLAN 10, 99 (Priority 0)
- [ ] DSW-A1: Secondary для VLAN 20, 40 (Priority 4096)
- [ ] DSW-A2: Root для VLAN 20, 40 (Priority 0)
- [ ] DSW-A2: Secondary для VLAN 10, 99 (Priority 4096)
- [ ] DSW-B1: Root для VLAN 10, 99 (Priority 0)
- [ ] DSW-B1: Secondary для VLAN 20, 30 (Priority 4096)
- [ ] DSW-B2: Root для VLAN 20, 30 (Priority 0)
- [ ] DSW-B2: Secondary для VLAN 10, 99 (Priority 4096)
- [ ] PortFast включен на всех access портах
- [ ] BPDU Guard включен на всех access портах
- [ ] PortFast trunk включен на ASW-A1 F0/2 (WLC)
- [ ] Все конфигурации сохранены
- [ ] Root Bridge проверен для всех VLANs
- [ ] PortFast проверен на access портах

---

## 📚 Дополнительная информация

### STP Port States (RSTP):

| State | Description | Forwards Data | Learns MAC |
|-------|-------------|---------------|------------|
| Discarding | Блокирует трафик | ❌ | ❌ |
| Learning | Изучает MAC, не передает | ❌ | ✅ |
| Forwarding | Полная работа | ✅ | ✅ |

### STP Port Roles (RSTP):

| Role | Description |
|------|-------------|
| Root Port | Лучший путь к Root Bridge |
| Designated Port | Назначенный порт для сегмента |
| Alternate Port | Backup для Root Port |
| Backup Port | Backup для Designated Port |

---

## 🚀 Готовы к Части 5?

В следующей части мы настроим:
- **OSPF** - динамическая маршрутизация
- **Router ID** - идентификация OSPF роутеров
- **Network statements** - включение OSPF на интерфейсах
- **Passive interfaces** - оптимизация OSPF
- **Static routes** - маршруты по умолчанию
- **Default route advertisement** - ASBR конфигурация

**До встречи в Части 5! 🎓**