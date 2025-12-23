# CCNA Мини-курс - BONUS
## Часть 10: BGP Basics (Beyond CCNA Scope)

---

## ⚠️ ВАЖНОЕ ПРЕДУПРЕЖДЕНИЕ

**BGP НЕ ВХОДИТ В CCNA 200-301!**

Этот материал:
- 🎓 **Уровень:** CCNP Enterprise / Service Provider
- 📚 **Цель:** Дополнительное обучение
- ⚠️ **НЕ требуется** для CCNA экзамена
- 💡 **Полезно:** Для понимания Enterprise/ISP сетей

**Если вы готовитесь к CCNA - сначала сдайте экзамен!**

---

## 📋 Теоретический минимум BGP

### Что такое BGP?

**BGP (Border Gateway Protocol)** - протокол маршрутизации для Internet:
- Path-vector протокол
- Exterior Gateway Protocol (EGP)
- Связывает разные Autonomous Systems (AS)
- Используется ISP и крупными Enterprise

### Ключевые концепции:

**1. Autonomous System (AS)**
- Группа сетей под единым административным управлением
- AS Number (ASN): 16-bit (1-65535) или 32-bit
- Private ASN: 64512-65534
- Public ASN: выдаётся IANA/RIR

**2. BGP Neighbors (Peers)**
- Ручная настройка (не автоматическое обнаружение)
- TCP port 179 для соединения
- Must be reachable (обычно через IGP или static routes)

**3. eBGP vs iBGP**
- **eBGP** (External BGP): между разными AS
- **iBGP** (Internal BGP): внутри одного AS
- eBGP AD: 20
- iBGP AD: 200

**4. BGP Path Selection**
- Использует атрибуты для выбора best path
- НЕ использует метрику (как OSPF cost)
- Алгоритм из 13+ шагов

**5. BGP Attributes**
- **WEIGHT** (Cisco-proprietary, локальный)
- **LOCAL_PREF** (внутри AS)
- **AS_PATH** (количество AS в пути)
- **ORIGIN** (IGP > EGP > Incomplete)
- **MED** (Multi-Exit Discriminator)
- **Next-hop** (IP следующего hop)

---

## 🎯 Цели Bonus раздела

- ✅ Понять концепцию BGP и AS
- ✅ Настроить eBGP между R1 и ISP
- ✅ Понять BGP attributes
- ✅ Настроить BGP route filtering
- ✅ Использовать BGP вместо static routes
- ✅ Настроить BGP route advertisement

---

## 🗺️ BGP Design для Mega Lab

### Изменения в топологии:

**Текущая конфигурация (Static Routes):**
```
R1 --- Static Route ---> ISP A (203.0.113.1)
R1 --- Static Route ---> ISP B (203.0.113.5)
```

**Новая конфигурация (BGP):**
```
R1 (AS 65001) --- eBGP ---> ISP A (AS 65100)
R1 (AS 65001) --- eBGP ---> ISP B (AS 65200)
```

### AS Numbers:
- **R1 (Enterprise)**: AS 65001 (Private ASN)
- **ISP A**: AS 65100 (Private ASN для лабы)
- **ISP B**: AS 65200 (Private ASN для лабы)

### BGP Neighbors:
- R1 ↔ ISP A: 203.0.113.1 (neighbor)
- R1 ↔ ISP B: 203.0.113.5 (neighbor)

### Advertised Networks:
- R1 → ISPs: Internal networks (10.0.0.0/8)
- ISPs → R1: Default route (0.0.0.0/0)

---

## 📝 ЧАСТЬ 1: Подготовка - Удаление Static Routes

### ШАГ 1: Удаление существующих Static Routes на R1

⚠️ **Важно:** Сначала удалим static routes, которые мы настроили в Части 5.

```cisco
R1(config)# no ip route 0.0.0.0 0.0.0.0 203.0.113.1
R1(config)# no ip route 0.0.0.0 0.0.0.0 203.0.113.5 2
```

**Проверка:**
```cisco
R1# show ip route static
! Не должно быть static default routes
```

**Также удалим OSPF default route advertisement:**
```cisco
R1(config)# router ospf 1
R1(config-router)# no default-information originate
R1(config-router)# exit
```

💡 После этого внутренние устройства потеряют доступ к Internet - временно, пока не настроим BGP.

---

## 📝 ЧАСТЬ 2: Базовая конфигурация BGP

### ШАГ 2: Конфигурация BGP на R1

**Создание BGP процесса:**
```cisco
R1(config)# router bgp 65001
R1(config-router)# bgp router-id 10.0.0.76
R1(config-router)# bgp log-neighbor-changes
```

**Объяснение:**
- `router bgp 65001` - создаёт BGP процесс с ASN 65001
- `bgp router-id` - ручная настройка Router ID (best practice)
- `bgp log-neighbor-changes` - логирование изменений соседей

---

### ШАГ 2 (продолжение): Настройка eBGP Neighbors

**Neighbor к ISP A:**
```cisco
R1(config-router)# neighbor 203.0.113.1 remote-as 65100
R1(config-router)# neighbor 203.0.113.1 description ISP-A
```

**Neighbor к ISP B:**
```cisco
R1(config-router)# neighbor 203.0.113.5 remote-as 65200
R1(config-router)# neighbor 203.0.113.5 description ISP-B
```

**Объяснение:**
- `neighbor [ip] remote-as [asn]` - определяет BGP соседа
- `remote-as` отличается от локального → это eBGP
- `description` - для документации

---

### ШАГ 2 (продолжение): Network Advertisement

**Анонсирование сетей в BGP:**

```cisco
R1(config-router)# network 10.0.0.0 mask 255.0.0.0
R1(config-router)# exit
```

**Объяснение:**
- `network` - анонсирует сеть в BGP
- Сеть ДОЛЖНА существовать в routing table
- Мы анонсируем 10.0.0.0/8 (все наши внутренние сети)

💡 **Важно:** BGP `network` команда отличается от OSPF - нужен mask!

---

## 📝 ЧАСТЬ 3: Конфигурация ISP Routers

⚠️ **Примечание:** В реальной лабе Jeremy's Mega Lab ISP роутеры уже настроены. Но для полного понимания покажу конфигурацию.

### Конфигурация ISP A (для справки)

```cisco
! ISP A Router
router bgp 65100
 bgp router-id 203.0.113.1
 neighbor 203.0.113.2 remote-as 65001
 neighbor 203.0.113.2 description Customer-R1
 network 0.0.0.0
```

### Конфигурация ISP B (для справки)

```cisco
! ISP B Router
router bgp 65200
 bgp router-id 203.0.113.5
 neighbor 203.0.113.6 remote-as 65001
 neighbor 203.0.113.6 description Customer-R1
 network 0.0.0.0
```

💡 ISP анонсирует default route (0.0.0.0/0) для выхода в Internet.

---

## 📝 ЧАСТЬ 4: Проверка BGP Neighbors

### ШАГ 3: Проверка BGP соседства

**Проверка BGP neighbors:**
```cisco
R1# show ip bgp summary
```

**Ожидаемый вывод:**
```
BGP router identifier 10.0.0.76, local AS number 65001

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
203.0.113.1     4 65100       5       5        5    0    0 00:01:23        1
203.0.113.5     4 65200       5       5        5    0    0 00:01:20        1
```

**Ключевые столбцы:**
- **Neighbor**: IP адрес BGP peer
- **AS**: Remote AS number
- **State/PfxRcd**: Количество полученных префиксов (или состояние)

✅ **Если видите цифру (1) - соседство установлено!**

❌ **Если видите Idle, Active, Connect - проблемы с соседством**

---

### Детальная проверка neighbor

```cisco
R1# show ip bgp neighbors 203.0.113.1
```

Смотрите на:
- **BGP state**: Established (хорошо) vs Idle/Active (плохо)
- **Prefixes received**: сколько маршрутов получено
- **Connection attempts**: количество попыток подключения

---

## 📝 ЧАСТЬ 5: BGP Routing Table

### ШАГ 4: Проверка BGP Table

**Посмотреть BGP таблицу:**
```cisco
R1# show ip bgp
```

**Ожидаемый вывод:**
```
   Network          Next Hop            Metric LocPrf Weight Path
*> 0.0.0.0          203.0.113.1              0             0 65100 i
*                   203.0.113.5              0             0 65200 i
*> 10.0.0.0         0.0.0.0                  0         32768 i
```

**Объяснение символов:**
- `*` - valid route
- `>` - best route (выбранный путь)
- `i` - origin (IGP)

**Объяснение маршрутов:**
- **0.0.0.0/0**: Default route от обоих ISP
- **Best path**: через ISP A (203.0.113.1)
- **10.0.0.0/8**: Локальная сеть (анонсируем мы)

---

### Проверка IP routing table

```cisco
R1# show ip route bgp
```

**Ожидаемый вывод:**
```
B*    0.0.0.0/0 [20/0] via 203.0.113.1, 00:05:23
```

**Объяснение:**
- `B` - BGP route
- `*` - candidate default route
- `[20/0]` - AD 20, metric 0
- Best path выбран (через ISP A)

✅ Default route теперь получаем через BGP!

---

## 📝 ЧАСТЬ 6: BGP Route Advertisement в OSPF

### ШАГ 5: Редистрибуция BGP в OSPF

**Проблема:** Внутренние устройства не видят BGP маршруты.

**Решение:** Redistribute BGP routes в OSPF.

```cisco
R1(config)# router ospf 1
R1(config-router)# redistribute bgp 65001 subnets
R1(config-router)# exit
```

**Объяснение:**
- `redistribute bgp` - берёт BGP routes и вставляет в OSPF
- `subnets` - включая subnets (обязательно!)
- OSPF анонсирует эти маршруты внутрь сети

**Альтернатива (проще):**
```cisco
R1(config)# router ospf 1
R1(config-router)# default-information originate
R1(config-router)# exit
```

Это анонсирует только default route, не все BGP routes.

---

### Проверка на внутренних устройствах

**На DSW-A1:**
```cisco
DSW-A1# show ip route
```

Должны видеть:
```
O*E2  0.0.0.0/0 [110/1] via 10.0.0.45, GigabitEthernet1/1/1
                [110/1] via 10.0.0.61, GigabitEthernet1/1/2
```

✅ Внутренние устройства снова имеют default route!

---

## 📝 ЧАСТЬ 7: BGP Path Selection

### Понимание BGP Best Path Selection

BGP использует **алгоритм из 13+ шагов** для выбора best path:

**Основные шаги (упрощённо):**
1. **Weight** (выше = лучше) - Cisco only
2. **Local Preference** (выше = лучше) - внутри AS
3. **Locally originated** (prefer local)
4. **AS-PATH** (короче = лучше)
5. **Origin** (IGP > EGP > incomplete)
6. **MED** (ниже = лучше)
7. **eBGP > iBGP**
8. **IGP metric** к next-hop
9. **Oldest route** (стабильность)
10. **Lowest Router ID**

---

### Пример: Влияние на Path Selection

**Текущая ситуация:**
- ISP A и ISP B анонсируют 0.0.0.0/0
- BGP выбирает ISP A (почему?)

**Проверка:**
```cisco
R1# show ip bgp 0.0.0.0
```

**Вывод покажет:**
```
  Path: 65100
  Path: 65200
```

Оба пути одинаковые по AS-PATH length (1 AS).

**BGP выбрал ISP A потому что:**
- Oldest route (ISP A neighbor установился первым)
- Или Lower Router ID

---

### Влияние на выбор с помощью Weight

**Предпочесть ISP B:**
```cisco
R1(config)# router bgp 65001
R1(config-router)# neighbor 203.0.113.5 weight 200
R1(config-router)# end
```

**Проверка:**
```cisco
R1# show ip bgp
```

Теперь best path через ISP B (больший Weight).

**Вернуть обратно:**
```cisco
R1(config)# router bgp 65001
R1(config-router)# no neighbor 203.0.113.5 weight 200
```

---

## 📝 ЧАСТЬ 8: BGP Route Filtering

### ШАГ 6: Prefix-List для фильтрации

**Задача:** Разрешить только default route от ISP, блокировать остальное.

**Создание Prefix-List:**
```cisco
R1(config)# ip prefix-list ALLOW-DEFAULT permit 0.0.0.0/0
R1(config)# ip prefix-list ALLOW-DEFAULT deny 0.0.0.0/0 le 32
```

**Применение к neighbor:**
```cisco
R1(config)# router bgp 65001
R1(config-router)# neighbor 203.0.113.1 prefix-list ALLOW-DEFAULT in
R1(config-router)# neighbor 203.0.113.5 prefix-list ALLOW-DEFAULT in
R1(config-router)# exit
```

**Clear BGP для применения:**
```cisco
R1# clear ip bgp * soft in
```

**Объяснение:**
- `prefix-list` - более гибкий чем ACL для BGP
- `permit 0.0.0.0/0` - разрешить точно default route
- `deny 0.0.0.0/0 le 32` - запретить всё остальное
- `in` - входящие updates от neighbor

---

### Исходящая фильтрация (Outbound)

**Задача:** Анонсировать только агрегированный 10.0.0.0/8 prefix.

```cisco
R1(config)# ip prefix-list ADVERTISE-AGGREGATE permit 10.0.0.0/8
R1(config)# ip prefix-list ADVERTISE-AGGREGATE deny 0.0.0.0/0 le 32

R1(config)# router bgp 65001
R1(config-router)# neighbor 203.0.113.1 prefix-list ADVERTISE-AGGREGATE out
R1(config-router)# neighbor 203.0.113.5 prefix-list ADVERTISE-AGGREGATE out
R1(config-router)# exit
```

**Clear BGP:**
```cisco
R1# clear ip bgp * soft out
```

Теперь ISP получают только 10.0.0.0/8, не детальные subnets.

---

## 📝 ЧАСТЬ 9: BGP Attributes Manipulation

### Использование AS-PATH Prepending

**Задача:** Сделать путь через ISP B менее предпочтительным.

**AS-PATH Prepending:**
```cisco
R1(config)# route-map PREPEND-AS permit 10
R1(config-route-map)# set as-path prepend 65001 65001 65001
R1(config-route-map)# exit

R1(config)# router bgp 65001
R1(config-router)# neighbor 203.0.113.5 route-map PREPEND-AS out
R1(config-router)# exit
```

**Объяснение:**
- `set as-path prepend` - добавляет наш ASN несколько раз
- Делает AS-PATH длиннее
- Другие AS выберут более короткий путь (через ISP A)

**Используется для:** Traffic engineering (контроль входящего трафика)

---

### Использование Local Preference (iBGP)

💡 Если бы у нас было iBGP (несколько роутеров в AS 65001):

```cisco
R1(config)# route-map PREFER-ISP-A permit 10
R1(config-route-map)# set local-preference 200
R1(config-route-map)# exit

R1(config)# router bgp 65001
R1(config-router)# neighbor 203.0.113.1 route-map PREFER-ISP-A in
```

**Объяснение:**
- Local Preference - внутри AS
- Больше = лучше (default: 100)
- Все роутеры в AS предпочтут этот путь

---

## 📝 ЧАСТЬ 10: BGP Failover Testing

### ШАГ 7: Тестирование отказоустойчивости

**Тест 1: Отключение primary link**

```cisco
R1(config)# interface g0/0/0
R1(config-if)# shutdown
R1(config-if)# exit
```

**Проверка:**
```cisco
R1# show ip bgp summary
```

ISP A neighbor должен быть в состоянии Idle.

```cisco
R1# show ip route
```

Default route теперь через ISP B (203.0.113.5).

**Тест connectivity с PC1:**
```
PC> ping 8.8.8.8
```

Ping должен работать через backup link!

---

**Восстановление:**
```cisco
R1(config)# interface g0/0/0
R1(config-if)# no shutdown
R1(config-if)# exit
```

**Проверка:**
```cisco
R1# show ip bgp summary
```

Оба neighbors снова Established.

---

### Тест 2: BGP Graceful Restart

**Перезапуск BGP процесса:**
```cisco
R1# clear ip bgp *
```

⏳ BGP neighbors переустановятся (~30 секунд).

**Проверка:**
```cisco
R1# show ip bgp summary
```

Оба neighbors должны вернуться в Established.

---

## ✅ Проверка конфигурации BGP

### 1. BGP Process

```cisco
R1# show ip protocols
```

Должны видеть BGP process с ASN 65001.

---

### 2. BGP Neighbors

```cisco
R1# show ip bgp summary

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
203.0.113.1     4 65100      XX      XX       XX    0    0 hh:mm:ss        1
203.0.113.5     4 65200      XX      XX       XX    0    0 hh:mm:ss        1
```

✅ Оба neighbors в состоянии с цифрой (получены prefixes).

---

### 3. BGP Table

```cisco
R1# show ip bgp
```

Должны видеть:
- Default route (0.0.0.0/0) от обоих ISP
- Best path выбран (символ `>`)
- Локальная сеть 10.0.0.0/8

---

### 4. IP Routing Table

```cisco
R1# show ip route bgp
```

Default route должен быть установлен через BGP.

---

### 5. BGP Advertised Routes

```cisco
R1# show ip bgp neighbors 203.0.113.1 advertised-routes
```

Должны видеть 10.0.0.0/8 анонсируемый к ISP A.

---

### 6. BGP Received Routes

```cisco
R1# show ip bgp neighbors 203.0.113.1 received-routes
```

Должны видеть default route от ISP A.

💡 **Примечание:** Эта команда требует `neighbor soft-reconfiguration inbound`.

---

## 📊 BGP Configuration Summary

### R1 BGP Configuration

```cisco
router bgp 65001
 bgp router-id 10.0.0.76
 bgp log-neighbor-changes
 neighbor 203.0.113.1 remote-as 65100
 neighbor 203.0.113.1 description ISP-A
 neighbor 203.0.113.5 remote-as 65200
 neighbor 203.0.113.5 description ISP-B
 network 10.0.0.0 mask 255.0.0.0
```

---

### BGP Neighbors Summary

| Neighbor | Remote AS | Description | Type | Status |
|----------|-----------|-------------|------|--------|
| 203.0.113.1 | 65100 | ISP-A | eBGP | Established |
| 203.0.113.5 | 65200 | ISP-B | eBGP | Established |

---

### BGP Routes Summary

| Network | Next-Hop | AS-PATH | Best | Source |
|---------|----------|---------|------|--------|
| 0.0.0.0/0 | 203.0.113.1 | 65100 | ✅ | ISP A |
| 0.0.0.0/0 | 203.0.113.5 | 65200 | ❌ | ISP B |
| 10.0.0.0/8 | 0.0.0.0 | - | ✅ | Local |

---

## 🎓 Ключевые команды BGP

```cisco
# BGP Process
router bgp [asn]
 bgp router-id [ip]
 bgp log-neighbor-changes
 neighbor [ip] remote-as [asn]
 neighbor [ip] description [text]
 neighbor [ip] weight [value]
 network [network] mask [mask]

# Route Manipulation
neighbor [ip] route-map [name] in|out
neighbor [ip] prefix-list [name] in|out
set as-path prepend [asn] [asn]...
set local-preference [value]

# Verification
show ip bgp
show ip bgp summary
show ip bgp neighbors
show ip bgp neighbors [ip] advertised-routes
show ip bgp neighbors [ip] received-routes
show ip protocols
clear ip bgp *
clear ip bgp * soft in|out

# Filtering
ip prefix-list [name] permit|deny [prefix]
route-map [name] permit|deny [seq]
```

---

## 💡 BGP Best Practices

### Design:
1. ✅ Всегда используйте Router ID вручную
2. ✅ Документируйте neighbors (description)
3. ✅ Используйте prefix-lists для фильтрации
4. ✅ Логируйте изменения neighbors
5. ✅ Планируйте AS numbering (private vs public)

### Security:
1. ✅ MD5 authentication между peers
2. ✅ Maximum-prefix limits
3. ✅ Strict prefix filtering (in и out)
4. ✅ Не принимайте full routing table (default only)
5. ✅ TTL security (eBGP multihop protection)

### Operations:
1. ✅ Мониторьте BGP sessions (state changes)
2. ✅ Используйте soft reconfiguration (soft reset)
3. ✅ Документируйте route-maps и policy
4. ✅ Тестируйте failover регулярно
5. ✅ Следите за routing table size

---

## 🔍 Troubleshooting BGP

### Проблема 1: Neighbor не устанавливается

**Симптомы:**
```cisco
R1# show ip bgp summary
Neighbor in Idle or Active state
```

**Причины и решения:**

1. **Нет IP connectivity**
```cisco
R1# ping 203.0.113.1
! Должен работать
```

2. **Неправильный remote-as**
```cisco
show ip bgp neighbors 203.0.113.1
! Проверьте configured vs expected AS
```

3. **TCP port 179 blocked**
```cisco
show ip access-lists
! Проверьте ACL на интерфейсах
```

4. **Неправильный neighbor IP**
```cisco
router bgp 65001
 no neighbor 203.0.113.1
 neighbor 203.0.113.1 remote-as 65100
```

---

### Проблема 2: Routes не появляются

**Симптомы:** Neighbor Established, но нет routes

**Причины:**

1. **Network не в routing table**
```cisco
R1# show ip route 10.0.0.0
! Сеть должна существовать
```

2. **Prefix-list блокирует**
```cisco
R1# show ip bgp neighbors 203.0.113.1 | include filter
```

3. **Route-map блокирует**
```cisco
show route-map
```

---

### Проблема 3: Неправильный best path

**Симптомы:** BGP выбирает не тот путь

**Решение:** Проверьте path selection algorithm

```cisco
R1# show ip bgp [network]
```

Смотрите:
- Weight (выше = лучше)
- Local Preference (выше = лучше)
- AS-PATH (короче = лучше)
- Origin
- MED
- eBGP vs iBGP

Используйте route-map для изменения attributes.

---

## 📚 BGP для реального мира

### Когда использовать BGP?

**✅ Используйте BGP:**
- Multihoming к 2+ ISP
- Вы являетесь ISP или Service Provider
- Крупная Enterprise сеть (multiple sites)
- Нужен детальный контроль routing policy
- Интеграция с Cloud providers

**❌ НЕ используйте BGP:**
- Один ISP (используйте default route + NAT)
- Малый/средний бизнес без multihoming
- Нет опыта администрирования BGP
- Нет необходимости в сложной routing policy

---

### BGP в Cloud (AWS, Azure, GCP)

**AWS:**
- Virtual Private Gateway (VGW) с BGP
- AWS ASN: 64512-65534 или custom
- BGP для Direct Connect, VPN

**Azure:**
- VPN Gateway с BGP
- Azure ASN: 65515 (default)
- ExpressRoute с BGP

**GCP:**
- Cloud Router с BGP
- Google ASN: 16550
- Cloud Interconnect с BGP

💡 Знание BGP критически важно для Cloud networking!

---

## 🎯 Чек-лист завершения Bonus Part 10

### BGP Configuration:
- [ ] Static routes удалены на R1
- [ ] BGP process создан (AS 65001)
- [ ] Router ID настроен (10.0.0.76)
- [ ] Neighbor к ISP A настроен (203.0.113.1, AS 65100)
- [ ] Neighbor к ISP B настроен (203.0.113.5, AS 65200)
- [ ] Network 10.0.0.0/8 анонсируется

### BGP Verification:
- [ ] Show ip bgp summary - оба neighbors Established
- [ ] Show ip bgp - default route получен
- [ ] Show ip route bgp - default route в routing table
- [ ] Ping 8.8.8.8 работает с внутренних устройств

### Advance
