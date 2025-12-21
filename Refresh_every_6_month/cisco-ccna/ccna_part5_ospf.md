# CCNA Мини-курс на основе Jeremy's Mega Lab
## Часть 5: OSPF Dynamic Routing & Static Routes

---

## 📋 Теоретический минимум

### Ключевые концепции:

**1. OSPF (Open Shortest Path First)**
- Link-state протокол маршрутизации
- Стандарт IEEE (открытый протокол)
- Использует алгоритм Dijkstra (SPF)
- Метрика: cost (основан на bandwidth)
- AD (Administrative Distance): 110

**2. OSPF Areas**
- Логическое разделение сети OSPF
- **Area 0** - backbone area (обязательна)
- Все остальные area подключаются к Area 0
- В нашей лабе: только Area 0

**3. OSPF Process ID**
- Локальное значение на роутере
- Не обязательно совпадать между роутерами
- В лабе используем: Process ID = 1

**4. OSPF Router ID**
- Уникальный идентификатор роутера в OSPF домене
- Формат: как IP адрес (например, 10.0.0.76)
- Выбор: Manual > Highest Loopback > Highest Physical IP

**5. OSPF Network Statement**
- Включает интерфейсы в OSPF процесс
- Формат: `network [ip] [wildcard-mask] area [area-id]`
- Wildcard mask: инверсия subnet mask

**6. OSPF Interface Configuration**
- Альтернатива network statement
- `ip ospf [process-id] area [area-id]` на интерфейсе
- Более гранулярный контроль

**7. OSPF Passive Interface**
- Интерфейс не отправляет Hello пакеты
- Но сеть всё равно анонсируется
- Используется на LAN интерфейсах без OSPF соседей

**8. OSPF Network Type**
- **Broadcast** - выбирает DR/BDR (по умолчанию на Ethernet)
- **Point-to-Point** - не выбирает DR/BDR (рекомендуется)
- В лабе: Point-to-Point на всех p2p линках

**9. ASBR (Autonomous System Boundary Router)**
- Роутер на границе OSPF домена
- Анонсирует внешние маршруты
- R1 в нашей лабе - ASBR

**10. Default Route**
- 0.0.0.0/0 - маршрут "по умолчанию"
- Используется когда нет более специфичного маршрута
- В лабе: R1 анонсирует default route в OSPF

---

## 🎯 Цели раздела

- ✅ Настроить OSPF на R1 (LAN-facing interfaces)
- ✅ Настроить OSPF на Core switches (CSW1, CSW2)
- ✅ Настроить OSPF на Distribution switches
- ✅ Настроить Router ID вручную
- ✅ Настроить Passive interfaces
- ✅ Настроить Point-to-Point network type
- ✅ Настроить Static Default Routes на R1
- ✅ Настроить R1 как ASBR

---

## 🗺️ OSPF Design

### OSPF Parameters:
- **Process ID**: 1
- **Area**: 0 (backbone)
- **Router IDs**: Match loopback IP addresses

### Network Types:
- All point-to-point links: `ip ospf network point-to-point`
- CSW1 ↔ CSW2 PortChannel: default (broadcast) - команда недоступна в PT

### Passive Interfaces:
- Все loopback интерфейсы
- Distribution switches: SVIs (кроме VLAN 99)

---

## 📝 ЧАСТЬ 1: OSPF on R1

### ШАГ 1: Базовая конфигурация OSPF на R1

**R1 Interfaces для OSPF:**
- G0/0 → CSW1 (10.0.0.33/30)
- G0/1 → CSW2 (10.0.0.37/30)
- Lo0 → Loopback (10.0.0.76/32)

```cisco
R1(config)# router ospf 1
R1(config-router)# router-id 10.0.0.76
R1(config-router)# passive-interface loopback 0
R1(config-router)# exit
```

**Объяснение:**
- `router ospf 1` - создаёт OSPF процесс 1
- `router-id 10.0.0.76` - вручную задаём Router ID
- `passive-interface loopback 0` - loopback не отправляет Hello

---

### Активация OSPF на интерфейсах R1

**Метод: Interface Configuration Mode**

```cisco
R1(config)# interface loopback 0
R1(config-if)# ip ospf 1 area 0
R1(config-if)# exit

R1(config)# interface range g0/0-1
R1(config-if-range)# ip ospf 1 area 0
R1(config-if-range)# ip ospf network point-to-point
R1(config-if-range)# exit
R1(config)# do write
```

**Объяснение:**
- `ip ospf 1 area 0` - включает OSPF на интерфейсе
- `ip ospf network point-to-point` - меняет тип сети

**Проверка:**
```cisco
R1# show ip ospf interface brief
R1# show ip ospf neighbor
```

---

## 📝 ЧАСТЬ 2: OSPF on Core Switches

### ШАГ 1: CSW1 Configuration

**CSW1 Interfaces для OSPF:**
- G1/0/1 → R1
- G1/1/1 → DSW-A1
- G1/1/2 → DSW-A2
- G1/1/3 → DSW-B1
- G1/1/4 → DSW-B2
- Po1 → CSW2
- Lo0 → Loopback

```cisco
CSW1(config)# router ospf 1
CSW1(config-router)# router-id 10.0.0.77
CSW1(config-router)# passive-interface loopback 0
CSW1(config-router)# network 10.0.0.41 0.0.0.0 area 0
CSW1(config-router)# network 10.0.0.34 0.0.0.0 area 0
CSW1(config-router)# network 10.0.0.45 0.0.0.0 area 0
CSW1(config-router)# network 10.0.0.49 0.0.0.0 area 0
CSW1(config-router)# network 10.0.0.53 0.0.0.0 area 0
CSW1(config-router)# network 10.0.0.57 0.0.0.0 area 0
CSW1(config-router)# network 10.0.0.77 0.0.0.0 area 0
CSW1(config-router)# exit
```

**Объяснение:**
- `network [ip] 0.0.0.0 area 0` - /32 wildcard mask (точный IP)
- Включает все интерфейсы с этим IP в OSPF

---

### Настройка Point-to-Point на физических портах CSW1

```cisco
CSW1(config)# interface range g1/0/1, g1/1/1-4
CSW1(config-if-range)# ip ospf network point-to-point
CSW1(config-if-range)# exit
CSW1(config)# do write
```

💡 **Важно:** На PortChannel1 команда `ip ospf network point-to-point` недоступна в Packet Tracer - оставляем default (broadcast).

---

### ШАГ 1 (продолжение): CSW2 Configuration

```cisco
CSW2(config)# router ospf 1
CSW2(config-router)# router-id 10.0.0.78
CSW2(config-router)# passive-interface loopback 0
CSW2(config-router)# network 10.0.0.42 0.0.0.0 area 0
CSW2(config-router)# network 10.0.0.38 0.0.0.0 area 0
CSW2(config-router)# network 10.0.0.61 0.0.0.0 area 0
CSW2(config-router)# network 10.0.0.65 0.0.0.0 area 0
CSW2(config-router)# network 10.0.0.69 0.0.0.0 area 0
CSW2(config-router)# network 10.0.0.73 0.0.0.0 area 0
CSW2(config-router)# network 10.0.0.78 0.0.0.0 area 0
CSW2(config-router)# exit

CSW2(config)# interface range g1/0/1, g1/1/1-4
CSW2(config-if-range)# ip ospf network point-to-point
CSW2(config-if-range)# exit
CSW2(config)# do write
```

**Проверка соседства:**
```cisco
CSW2# show ip ospf neighbor
```

Должны видеть: R1, CSW1, DSW-A1, DSW-A2, DSW-B1, DSW-B2

---

## 📝 ЧАСТЬ 3: OSPF on Distribution Switches

### ШАГ 1: DSW-A1 Configuration

**Интерфейсы DSW-A1:**
- G1/1/1 → CSW1
- G1/1/2 → CSW2
- VLAN 10, 20, 40, 99 (SVIs)
- Lo0 → Loopback

**Passive Interfaces:**
- Loopback 0
- VLAN 10, 20, 40 (не VLAN 99 - соседство с DSW-A2)

```cisco
DSW-A1(config)# router ospf 1
DSW-A1(config-router)# router-id 10.0.0.79
DSW-A1(config-router)# passive-interface loopback 0
DSW-A1(config-router)# passive-interface vlan 10
DSW-A1(config-router)# passive-interface vlan 20
DSW-A1(config-router)# passive-interface vlan 40
DSW-A1(config-router)# network 10.0.0.46 0.0.0.0 area 0
DSW-A1(config-router)# network 10.0.0.62 0.0.0.0 area 0
DSW-A1(config-router)# network 10.0.0.2 0.0.0.0 area 0
DSW-A1(config-router)# network 10.1.0.2 0.0.0.0 area 0
DSW-A1(config-router)# network 10.2.0.2 0.0.0.0 area 0
DSW-A1(config-router)# network 10.6.0.2 0.0.0.0 area 0
DSW-A1(config-router)# network 10.0.0.79 0.0.0.0 area 0
DSW-A1(config-router)# exit

DSW-A1(config)# interface range g1/1/1-2
DSW-A1(config-if-range)# ip ospf network point-to-point
DSW-A1(config-if-range)# exit
DSW-A1(config)# do write
```

**Объяснение Passive Interfaces:**
- VLAN 99: Active - соседство с DSW-A2 через этот VLAN
- VLAN 10, 20, 40: Passive - нет других OSPF роутеров

---

### ШАГ 1 (продолжение): DSW-A2 Configuration

```cisco
DSW-A2(config)# router ospf 1
DSW-A2(config-router)# router-id 10.0.0.80
DSW-A2(config-router)# passive-interface loopback 0
DSW-A2(config-router)# passive-interface vlan 10
DSW-A2(config-router)# passive-interface vlan 20
DSW-A2(config-router)# passive-interface vlan 40
DSW-A2(config-router)# network 10.0.0.50 0.0.0.0 area 0
DSW-A2(config-router)# network 10.0.0.66 0.0.0.0 area 0
DSW-A2(config-router)# network 10.0.0.3 0.0.0.0 area 0
DSW-A2(config-router)# network 10.1.0.3 0.0.0.0 area 0
DSW-A2(config-router)# network 10.2.0.3 0.0.0.0 area 0
DSW-A2(config-router)# network 10.6.0.3 0.0.0.0 area 0
DSW-A2(config-router)# network 10.0.0.80 0.0.0.0 area 0
DSW-A2(config-router)# exit

DSW-A2(config)# interface range g1/1/1-2
DSW-A2(config-if-range)# ip ospf network point-to-point
DSW-A2(config-if-range)# exit
DSW-A2(config)# do write
```

**Проверка соседства DSW-A2:**
```cisco
DSW-A2# show ip ospf neighbor
```

Должны видеть:
- 10.0.0.79 (DSW-A1) через VLAN 99
- 10.0.0.77 (CSW1) через G1/1/1
- 10.0.0.78 (CSW2) через G1/1/2

---

### ШАГ 1 (продолжение): DSW-B1 Configuration

```cisco
DSW-B1(config)# router ospf 1
DSW-B1(config-router)# router-id 10.0.0.81
DSW-B1(config-router)# passive-interface loopback 0
DSW-B1(config-router)# passive-interface vlan 10
DSW-B1(config-router)# passive-interface vlan 20
DSW-B1(config-router)# passive-interface vlan 30
DSW-B1(config-router)# network 10.0.0.54 0.0.0.0 area 0
DSW-B1(config-router)# network 10.0.0.70 0.0.0.0 area 0
DSW-B1(config-router)# network 10.0.0.18 0.0.0.0 area 0
DSW-B1(config-router)# network 10.3.0.2 0.0.0.0 area 0
DSW-B1(config-router)# network 10.4.0.2 0.0.0.0 area 0
DSW-B1(config-router)# network 10.5.0.2 0.0.0.0 area 0
DSW-B1(config-router)# network 10.0.0.81 0.0.0.0 area 0
DSW-B1(config-router)# exit

DSW-B1(config)# interface range g1/1/1-2
DSW-B1(config-if-range)# ip ospf network point-to-point
DSW-B1(config-if-range)# exit
DSW-B1(config)# do write
```

---

### ШАГ 1 (продолжение): DSW-B2 Configuration

```cisco
DSW-B2(config)# router ospf 1
DSW-B2(config-router)# router-id 10.0.0.82
DSW-B2(config-router)# passive-interface loopback 0
DSW-B2(config-router)# passive-interface vlan 10
DSW-B2(config-router)# passive-interface vlan 20
DSW-B2(config-router)# passive-interface vlan 30
DSW-B2(config-router)# network 10.0.0.58 0.0.0.0 area 0
DSW-B2(config-router)# network 10.0.0.74 0.0.0.0 area 0
DSW-B2(config-router)# network 10.0.0.19 0.0.0.0 area 0
DSW-B2(config-router)# network 10.3.0.3 0.0.0.0 area 0
DSW-B2(config-router)# network 10.4.0.3 0.0.0.0 area 0
DSW-B2(config-router)# network 10.5.0.3 0.0.0.0 area 0
DSW-B2(config-router)# network 10.0.0.82 0.0.0.0 area 0
DSW-B2(config-router)# exit

DSW-B2(config)# interface range g1/1/1-2
DSW-B2(config-if-range)# ip ospf network point-to-point
DSW-B2(config-if-range)# exit
DSW-B2(config)# do write
```

**Проверка маршрутов:**
```cisco
DSW-B2# show ip route ospf
```

Должны видеть маршруты через CSW1 и CSW2 ко всем сетям.

---

## 📝 ЧАСТЬ 4: Static Default Routes

### ШАГ 2: Настройка Static Default Routes на R1

**Концепция:**
- Два подключения к Internet: ISP A и ISP B
- Основной маршрут через ISP A (AD = 1)
- Backup маршрут через ISP B (AD = 2) - Floating Static Route

**R1 Internet Interfaces:**
- G0/0/0: 203.0.113.2/30 → ISP A (203.0.113.1)
- G0/1/0: 203.0.113.6/30 → ISP B (203.0.113.5)

```cisco
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.5 2
R1(config)# exit
```

**Объяснение:**
- Первый маршрут: AD = 1 (default) - основной
- Второй маршрут: AD = 2 - backup (floating)

**Проверка:**
```cisco
R1# show ip route static
```

Должен быть активен только маршрут через 203.0.113.1:
```
S*    0.0.0.0/0 [1/0] via 203.0.113.1
```

---

## 📝 ЧАСТЬ 5: OSPF Default Route Advertisement

### ШАГ 2b: Настройка R1 как ASBR

**ASBR (Autonomous System Boundary Router):**
- Роутер на границе OSPF домена
- Анонсирует внешние маршруты в OSPF
- R1 анонсирует default route

```cisco
R1(config)# router ospf 1
R1(config-router)# default-information originate
R1(config-router)# exit
R1(config)# do write
```

**Объяснение:**
- `default-information originate` - анонсирует default route в OSPF
- Все роутеры в OSPF получат маршрут 0.0.0.0/0

**Проверка на DSW-B2:**
```cisco
DSW-B2# show ip route
```

Должны видеть:
```
O*E2  0.0.0.0/0 [110/1] via [next-hop CSW1/CSW2]
```

- `O*E2` - OSPF External Type 2 (default route)

**Тест связности:**
```cisco
DSW-B2# ping 203.0.113.2
```

Должен работать ping до Internet IP R1.

---

## ✅ Проверка конфигурации

### 1. OSPF Neighbors

```cisco
R1# show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.77         0   FULL/  -        00:00:3x    10.0.0.34       GigabitEthernet0/0
10.0.0.78         0   FULL/  -        00:00:3x    10.0.0.38       GigabitEthernet0/1
```

**Ожидаемые соседи:**
- R1: CSW1, CSW2
- CSW1: R1, CSW2, DSW-A1, DSW-A2, DSW-B1, DSW-B2
- CSW2: R1, CSW1, DSW-A1, DSW-A2, DSW-B1, DSW-B2
- DSW-A1: CSW1, CSW2, DSW-A2 (через VLAN 99)
- DSW-A2: CSW1, CSW2, DSW-A1 (через VLAN 99)
- DSW-B1: CSW1, CSW2, DSW-B2 (через VLAN 99)
- DSW-B2: CSW1, CSW2, DSW-B1 (через VLAN 99)

---

### 2. OSPF Interfaces

```cisco
R1# show ip ospf interface brief

Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.0.0.76/32       1     LOOP  0/0
Gi0/0        1     0               10.0.0.33/30       1     P2P   1/1
Gi0/1        1     0               10.0.0.37/30       1     P2P   1/1
```

---

### 3. OSPF Routes

```cisco
DSW-A1# show ip route ospf
```

**Ожидаемые OSPF маршруты:**
```
O     10.0.0.36/30 [110/2] via 10.0.0.1, Vlan99
O     10.0.0.40/30 [110/2] via 10.0.0.1, Vlan99
O     10.0.0.76/32 [110/2] via 10.0.0.45, GigabitEthernet1/1/1
O     10.0.0.77/32 [110/1] via 10.0.0.45, GigabitEthernet1/1/1
O     10.0.0.78/32 [110/1] via 10.0.0.61, GigabitEthernet1/1/2
O*E2  0.0.0.0/0 [110/1] via 10.0.0.45, GigabitEthernet1/1/1
                [110/1] via 10.0.0.61, GigabitEthernet1/1/2
```

---

### 4. Default Route

```cisco
R1# show ip route static

S*    0.0.0.0/0 [1/0] via 203.0.113.1
```

```cisco
DSW-B2# show ip route | include 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.0.0.73, GigabitEthernet1/1/2
                [110/1] via 10.0.0.57, GigabitEthernet1/1/1
```

---

### 5. Ping Tests

```cisco
! Ping между офисами
DSW-A1# ping 10.0.0.18

! Ping loopback
DSW-A1# ping 10.0.0.82

! Ping Internet IP
DSW-A1# ping 203.0.113.2

! Ping ISP (после NAT в следующей части)
DSW-A1# ping 203.0.113.1
```

---

## 📊 OSPF Router ID Summary

| Device | Router ID | Loopback IP |
|--------|-----------|-------------|
| R1 | 10.0.0.76 | 10.0.0.76/32 |
| CSW1 | 10.0.0.77 | 10.0.0.77/32 |
| CSW2 | 10.0.0.78 | 10.0.0.78/32 |
| DSW-A1 | 10.0.0.79 | 10.0.0.79/32 |
| DSW-A2 | 10.0.0.80 | 10.0.0.80/32 |
| DSW-B1 | 10.0.0.81 | 10.0.0.81/32 |
| DSW-B2 | 10.0.0.82 | 10.0.0.82/32 |

---

## 📊 OSPF Network Type Summary

| Link | Network Type | DR/BDR |
|------|--------------|--------|
| R1 ↔ CSW1/CSW2 | Point-to-Point | No |
| CSW1 ↔ DSW switches | Point-to-Point | No |
| CSW2 ↔ DSW switches | Point-to-Point | No |
| CSW1 ↔ CSW2 (Po1) | Broadcast | Yes (но не критично) |
| DSW ↔ DSW (VLAN 99) | Broadcast | Yes |

---

## 📊 Passive Interfaces Summary

| Device | Passive Interfaces |
|--------|--------------------|
| R1 | Lo0 |
| CSW1 | Lo0 |
| CSW2 | Lo0 |
| DSW-A1 | Lo0, VLAN 10, 20, 40 |
| DSW-A2 | Lo0, VLAN 10, 20, 40 |
| DSW-B1 | Lo0, VLAN 10, 20, 30 |
| DSW-B2 | Lo0, VLAN 10, 20, 30 |

**VLAN 99 NOT passive** - нужно для соседства между DSW

---

## 💡 Практические советы

### OSPF Best Practices:
1. ✅ Всегда задавайте Router ID вручную
2. ✅ Используйте Point-to-Point на p2p линках
3. ✅ Делайте passive interface на LAN без OSPF соседей
4. ✅ Используйте loopback для Router ID
5. ✅ Проверяйте соседства перед продолжением

### Типичные ошибки:
❌ Забыть `router-id` - роутер выберет сам (может измениться)  
❌ Неправильный wildcard mask в network statement  
❌ Забыть `passive-interface` на loopback  
❌ Забыть `ip ospf network point-to-point`  
❌ Passive interface на VLAN 99 (нужно соседство!)  
❌ Забыть `default-information originate`  

---

## 🧪 Тестирование OSPF

### Тест 1: Проверка соседств

```cisco
! На каждом устройстве
show ip ospf neighbor
```

Подсчитайте количество соседей:
- R1: 2 (CSW1, CSW2)
- CSW1: 6 (R1, CSW2, 4×DSW)
- DSW-A1: 3 (CSW1, CSW2, DSW-A2)

---

### Тест 2: Failover Test

```cisco
! На R1 отключить G0/0
R1(config)# interface g0/0
R1(config-if)# shutdown

! Проверить маршрут
R1# show ip route static
! Должен активироваться floating route через ISP B

! Включить обратно
R1(config-if)# no shutdown
```

---

### Тест 3: OSPF Route Propagation

```cisco
! На DSW-B2 проверить маршруты Office A
DSW-B2# show ip route | include 10.1.0.0
DSW-B2# show ip route | include 10.2.0.0

! Должны быть OSPF маршруты
```

---

## 🎓 Ключевые команды Части 5

```cisco
# OSPF Configuration
router ospf [process-id]
 router-id [ip-address]
 network [ip] [wildcard-mask] area [area-id]
 passive-interface [type] [number]
 default-information originate

# OSPF Interface Configuration
interface [type] [number]
 ip ospf [process-id] area [area-id]
 ip ospf network point-to-point

# Static Routes
ip route [network] [mask] [next-hop] [ad]

# Verification
show ip ospf
show ip ospf neighbor
show ip ospf interface [type] [number]
show ip ospf interface brief
show ip route ospf
show ip route static
show ip protocols
```

---

## 📖 Wildcard Mask Calculation

### Subnet Mask → Wildcard Mask

**Правило:** Инвертировать биты subnet mask

**Примеры:**

| Subnet Mask | Wildcard Mask | Использование |
|-------------|---------------|---------------|
| 255.255.255.255 | 0.0.0.0 | Точный IP адрес (/32) |
| 255.255.255.252 | 0.0.0.3 | /30 сеть (p2p links) |
| 255.255.255.0 | 0.0.255.255 | /24 сеть |
| 255.255.0.0 | 0.0.255.255 | /16 сеть |
| 255.0.0.0 | 0.255.255.255 | /8 сеть |

**Формула:**
```
Wildcard Mask = 255.255.255.255 - Subnet Mask

Пример для /30 (255.255.255.252):
255.255.255.255
-255.255.255.252
----------------
  0.  0.  0.  3
```

---

## 🔍 Troubleshooting OSPF

### Проблема 1: Нет соседства

**Симптомы:**
```cisco
R1# show ip ospf neighbor
% OSPF: No neighbors found
```

**Причины и решения:**

1. **OSPF не включен на интерфейсе**
```cisco
! Проверка
show ip ospf interface brief

! Решение
interface g0/0
 ip ospf 1 area 0
```

2. **Разные Area**
```cisco
! Проверка
show ip ospf interface g0/0

! Решение - должны быть в одной area
router ospf 1
 network 10.0.0.33 0.0.0.0 area 0
```

3. **Passive interface**
```cisco
! Проверка
show ip ospf interface g0/0

! Решение
router ospf 1
 no passive-interface g0/0
```

---

### Проблема 2: Нет маршрутов в таблице

**Симптомы:**
```cisco
DSW-A1# show ip route ospf
! Пусто или мало маршрутов
```

**Решение:**
```cisco
! Проверить соседства
show ip ospf neighbor

! Проверить OSPF database
show ip ospf database

! Проверить что OSPF включен на нужных интерфейсах
show ip ospf interface brief
```

---

### Проблема 3: Default route не распространяется

**Симптомы:**
```cisco
DSW-B2# show ip route | include 0.0.0.0
! Нет маршрута 0.0.0.0/0
```

**Решение на R1:**
```cisco
! Проверка
show ip route static
show running-config | section router ospf

! Решение
router ospf 1
 default-information originate
```

---

### Проблема 4: Floating route не активируется

**Симптомы:**
Primary маршрут down, но backup не активируется

**Решение:**
```cisco
! Проверка
show ip route static

! Убедиться что AD floating route выше
ip route 0.0.0.0 0.0.0.0 203.0.113.5 2

! Обновить OSPF анонс
router ospf 1
 no default-information originate
 default-information originate
```

---

## 🎯 Чек-лист завершения Части 5

- [ ] OSPF включен на R1 (G0/0, G0/1, Lo0)
- [ ] OSPF включен на CSW1 (все L3 интерфейсы)
- [ ] OSPF включен на CSW2 (все L3 интерфейсы)
- [ ] OSPF включен на DSW-A1 (uplinks + SVIs)
- [ ] OSPF включен на DSW-A2 (uplinks + SVIs)
- [ ] OSPF включен на DSW-B1 (uplinks + SVIs)
- [ ] OSPF включен на DSW-B2 (uplinks + SVIs)
- [ ] Router ID настроен вручную на всех устройствах
- [ ] Passive interfaces настроены (loopbacks + SVIs)
- [ ] Point-to-Point network type на p2p линках
- [ ] OSPF соседства установлены (проверить на каждом)
- [ ] Static default routes настроены на R1
- [ ] Floating static route работает (AD=2)
- [ ] R1 анонсирует default route (default-information originate)
- [ ] Default route виден на всех устройствах (O*E2)
- [ ] Ping работает между офисами
- [ ] Ping работает до R1 Internet IP
- [ ] Все конфигурации сохранены

---

## 📚 OSPF Теория - Дополнительно

### OSPF Packet Types

| Type | Name | Description |
|------|------|-------------|
| 1 | Hello | Обнаружение соседей |
| 2 | DBD | Database Description |
| 3 | LSR | Link State Request |
| 4 | LSU | Link State Update |
| 5 | LSAck | Link State Acknowledgment |

### OSPF Neighbor States

| State | Description |
|-------|-------------|
| Down | Начальное состояние |
| Init | Получен Hello |
| 2-Way | Двустороннее общение |
| ExStart | Начало обмена DBD |
| Exchange | Обмен DBD |
| Loading | Запрос LSA |
| Full | Полное соседство |

### OSPF LSA Types

| Type | Name | Description |
|------|------|-------------|
| 1 | Router LSA | Информация о роутере |
| 2 | Network LSA | Информация от DR |
| 3 | Summary LSA | Inter-area routes (ABR) |
| 4 | ASBR Summary | Путь до ASBR |
| 5 | External LSA | External routes (ASBR) |

---

## 📊 OSPF Cost Calculation

**Formula:**
```
Cost = Reference Bandwidth / Interface Bandwidth

Default Reference Bandwidth = 100 Mbps (100,000 Kbps)
```

**Examples:**

| Interface | Bandwidth | Cost Calculation | Cost |
|-----------|-----------|------------------|------|
| FastEthernet | 100 Mbps | 100,000 / 100,000 | 1 |
| GigabitEthernet | 1 Gbps | 100,000 / 1,000,000 | 1 |
| Serial (T1) | 1.544 Mbps | 100,000 / 1,544 | 64 |
| Serial (64k) | 64 Kbps | 100,000 / 64 | 1562 |

💡 **Важно:** В современных сетях GigE и FastE имеют одинаковый cost (1). Для правильного расчёта используйте `auto-cost reference-bandwidth 10000` для 10G сетей.

---

## 🎓 OSPF Administrative Distance

| Route Source | AD |
|--------------|-----|
| Directly Connected | 0 |
| Static Route | 1 |
| EIGRP Summary | 5 |
| External BGP | 20 |
| Internal EIGRP | 90 |
| OSPF | 110 |
| IS-IS | 115 |
| RIP | 120 |
| External EIGRP | 170 |
| Internal BGP | 200 |

**В нашей лабе:**
- Static route: AD = 1 (primary)
- Static floating route: AD = 2 (backup)
- OSPF: AD = 110

---

## 💡 Advanced Tips

### Оптимизация OSPF Timers

```cisco
! По умолчанию: Hello 10s, Dead 40s
interface g0/0
 ip ospf hello-interval 5
 ip ospf dead-interval 20
```

### OSPF Authentication (не требуется в лабе)

```cisco
interface g0/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 MyPassword
```

### OSPF Summarization (не требуется в лабе)

```cisco
router ospf 1
 area 1 range 10.1.0.0 255.255.0.0
```

---

## 📝 Quick Reference Commands

```cisco
# Базовая конфигурация
router ospf 1
 router-id 10.0.0.76
 network 10.0.0.33 0.0.0.0 area 0
 passive-interface lo0
 default-information originate

# Интерфейс
interface g0/0
 ip ospf 1 area 0
 ip ospf network point-to-point

# Проверка - Краткая
show ip ospf neighbor
show ip ospf interface brief
show ip route ospf

# Проверка - Детальная
show ip ospf
show ip ospf interface g0/0
show ip ospf database
show ip protocols

# Troubleshooting
debug ip ospf hello
debug ip ospf adj
clear ip ospf process
```

---

## 🚀 Готовы к Части 6?

В следующей части мы настроим все Network Services:

### Network Services (Часть 6):
1. 🌐 **DHCP** - автоматическая выдача IP адресов
2. 📡 **DHCP Relay** - пересылка DHCP через роутеры
3. 🔍 **DNS** - разрешение имён
4. ⏰ **NTP** - синхронизация времени
5. 📊 **SNMP** - мониторинг сети
6. 📝 **Syslog** - централизованное логирование
7. 📁 **FTP** - передача файлов
8. 🔐 **SSH** - безопасный удалённый доступ
9. 🔄 **NAT** - трансляция адресов (Static + PAT)

Это будет самая насыщенная часть курса!

**До встречи в Части 6! 🎓**