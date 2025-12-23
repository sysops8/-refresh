# CCNA Мини-курс на основе Jeremy's Mega Lab
## Часть 8: IPv6 Configuration & Routing

---

## 📋 Теоретический минимум

### Ключевые концепции IPv6:

**1. IPv6 Address Format**
- 128-bit адрес (vs 32-bit IPv4)
- Формат: 8 групп по 16 bit (hexadecimal)
- Пример: 2001:0db8:0000:0001:0000:0000:0000:0001
- Сокращение: 2001:db8:0:1::1

**2. IPv6 Address Types**
- **Global Unicast** - 2000::/3 (публичные адреса)
- **Link-Local** - fe80::/10 (автоматически на каждом интерфейсе)
- **Unique Local** - fc00::/7 (частные адреса, как 10.0.0.0/8)
- **Multicast** - ff00::/8

**3. EUI-64 (Extended Unique Identifier)**
- Автоматическая генерация Interface ID из MAC адреса
- 48-bit MAC → 64-bit Interface ID
- Процесс: вставка FFFE + flip 7th bit

**4. IPv6 Routing**
- `ipv6 unicast-routing` - включение IPv6 маршрутизации
- Без этой команды устройство = IPv6 host
- Необходимо на всех роутерах/L3 коммутаторах

**5. IPv6 Static Routes**
- Формат: `ipv6 route ::/0 [next-hop]`
- Recursive: только next-hop IP
- Directly Connected: только exit interface
- Fully Specified: next-hop + exit interface

**6. IPv6 на интерфейсах**
- `ipv6 address [address]/[prefix]` - ручная настройка
- `ipv6 address [prefix] eui-64` - автоматическая генерация
- `ipv6 enable` - только link-local (без global unicast)

---

## 🎯 Цели раздела

- ✅ Включить IPv6 routing на R1, CSW1, CSW2
- ✅ Настроить IPv6 адреса на R1 (Internet + LAN)
- ✅ Настроить IPv6 адреса на CSW1, CSW2
- ✅ Использовать EUI-64 для автогенерации
- ✅ Использовать ipv6 enable на Layer-3 EtherChannel
- ✅ Настроить IPv6 static routes на R1
- ✅ Проверить IPv6 connectivity

---

## 🗺️ IPv6 Addressing Plan

### R1 Interfaces:
- **G0/0/0** (Internet ISP A): 2001:db8:a::2/64
- **G0/1/0** (Internet ISP B): 2001:db8:b::2/64
- **G0/0** (→ CSW1): 2001:db8:a1::/64 eui-64
- **G0/1** (→ CSW2): 2001:db8:a2::/64 eui-64

### Core Switches:
- **CSW1 G1/0/1** (→ R1): 2001:db8:a1::/64 eui-64
- **CSW2 G1/0/1** (→ R1): 2001:db8:a2::/64 eui-64
- **CSW1 ↔ CSW2 Po1**: ipv6 enable (link-local only)

💡 **Важно:** Это базовая IPv6 конфигурация, не полная dual-stack сеть.

---

## 📝 ЧАСТЬ 1: Enable IPv6 Routing

### ШАГ 1: Включение IPv6 Unicast Routing

**На R1, CSW1, CSW2:**

```cisco
! R1
R1(config)# ipv6 unicast-routing

! CSW1
CSW1(config)# ipv6 unicast-routing

! CSW2
CSW2(config)# ipv6 unicast-routing
```

**Объяснение:**
- Без этой команды устройство работает только как IPv6 host
- После включения устройство может маршрутизировать IPv6 пакеты
- Автоматически генерируются link-local адреса (fe80::/10)

**Проверка:**
```cisco
R1# show ipv6 interface brief
```

После включения routing вы увидите link-local адреса на всех интерфейсах.

---

## 📝 ЧАСТЬ 2: R1 IPv6 Configuration

### ШАГ 1: R1 Internet Interfaces (Ручная настройка)

**G0/0/0 → ISP A:**
```cisco
R1(config)# interface g0/0/0
R1(config-if)# ipv6 address 2001:db8:a::2/64
R1(config-if)# exit
```

**G0/1/0 → ISP B:**
```cisco
R1(config)# interface g0/1/0
R1(config-if)# ipv6 address 2001:db8:b::2/64
R1(config-if)# exit
```

**Проверка:**
```cisco
R1# show ipv6 interface brief
```

Ожидаемый вывод:
```
GigabitEthernet0/0/0  [up/up]
    FE80::xxxx:xxxx:xxxx:xxxx
    2001:DB8:A::2
GigabitEthernet0/1/0  [up/up]
    FE80::xxxx:xxxx:xxxx:xxxx
    2001:DB8:B::2
```

---

### ШАГ 1 (продолжение): R1 LAN Interfaces (EUI-64)

**G0/0 → CSW1:**
```cisco
R1(config)# interface g0/0
R1(config-if)# ipv6 address 2001:db8:a1::/64 eui-64
R1(config-if)# exit
```

**G0/1 → CSW2:**
```cisco
R1(config)# interface g0/1
R1(config-if)# ipv6 address 2001:db8:a2::/64 eui-64
R1(config-if)# exit
R1(config)# do write
```

**Объяснение EUI-64:**
- Указываем только prefix: 2001:db8:a1::/64
- Interface ID генерируется автоматически из MAC адреса
- Формат: 2001:db8:a1:0:xxxx:xxff:fexx:xxxx

**Проверка:**
```cisco
R1# show ipv6 interface g0/0
R1# show ipv6 interface brief
```

Вы увидите автоматически сгенерированный адрес с EUI-64 interface ID.

---

## 📝 ЧАСТЬ 3: Core Switches IPv6 Configuration

### ШАГ 1: CSW1 Configuration

**G1/0/1 → R1 (EUI-64):**
```cisco
CSW1(config)# interface g1/0/1
CSW1(config-if)# ipv6 address 2001:db8:a1::/64 eui-64
CSW1(config-if)# exit
```

**Po1 → CSW2 (ipv6 enable):**
```cisco
CSW1(config)# interface port-channel 1
CSW1(config-if)# ipv6 enable
CSW1(config-if)# exit
CSW1(config)# do write
```

**Объяснение ipv6 enable:**
- Включает IPv6 на интерфейсе БЕЗ global unicast адреса
- Генерирует только link-local адрес (fe80::/10)
- Используется когда не нужен полноценный IPv6 адрес

**Проверка:**
```cisco
CSW1# show ipv6 interface brief
CSW1# show ipv6 interface port-channel 1
```

---

### ШАГ 1 (продолжение): CSW2 Configuration

**G1/0/1 → R1 (EUI-64):**
```cisco
CSW2(config)# interface g1/0/1
CSW2(config-if)# ipv6 address 2001:db8:a2::/64 eui-64
CSW2(config-if)# exit
```

**Po1 → CSW1 (ipv6 enable):**
```cisco
CSW2(config)# interface port-channel 1
CSW2(config-if)# ipv6 enable
CSW2(config-if)# exit
CSW2(config)# do write
```

**Проверка:**
```cisco
CSW2# show ipv6 interface brief
```

**Тест connectivity между CSW1 и CSW2:**
```cisco
CSW2# ping ipv6 fe80::xxxx:xxxx:xxxx:xxxx%Po1
```

💡 **Важно:** Для ping link-local адреса нужно указать exit interface (%).

---

## 📝 ЧАСТЬ 4: IPv6 Static Routes

### ШАГ 2: Default Routes на R1

**Концепция:**
- Два подключения к Internet (ISP A и ISP B)
- Primary route: через ISP A (2001:db8:a::1)
- Backup route: через ISP B (2001:db8:b::1) - Floating

**Primary Route (Recursive):**
```cisco
R1(config)# ipv6 route ::/0 2001:db8:a::1
```

**Backup Route (Fully Specified + Floating):**
```cisco
R1(config)# ipv6 route ::/0 g0/1/0 2001:db8:b::1 2
R1(config)# exit
R1# write memory
```

**Объяснение:**
- `::/0` = default route (эквивалент 0.0.0.0/0 в IPv4)
- Первый маршрут: только next-hop (recursive)
- Второй маршрут: exit interface + next-hop + AD=2 (fully specified floating)

**Проверка:**
```cisco
R1# show ipv6 route
R1# show ipv6 route static
```

**Ожидаемый вывод:**
```
S   ::/0 [1/0]
     via 2001:DB8:A::1
```

Floating route не появится в таблице (AD=2 выше, чем AD=1).

---

## ✅ Проверка конфигурации

### 1. IPv6 Routing Status

```cisco
R1# show ipv6 interface brief
CSW1# show ipv6 interface brief
CSW2# show ipv6 interface brief
```

**Ожидаемый вывод для R1:**
```
GigabitEthernet0/0/0       [up/up]
    FE80::xxxx:xxxx:xxxx:xxxx
    2001:DB8:A::2
GigabitEthernet0/1/0       [up/up]
    FE80::xxxx:xxxx:xxxx:xxxx
    2001:DB8:B::2
GigabitEthernet0/0         [up/up]
    FE80::xxxx:xxxx:xxxx:xxxx
    2001:DB8:A1::xxxx:xxFF:FExx:xxxx
GigabitEthernet0/1         [up/up]
    FE80::xxxx:xxxx:xxxx:xxxx
    2001:DB8:A2::xxxx:xxFF:FExx:xxxx
```

---

### 2. IPv6 Neighbor Discovery

```cisco
R1# show ipv6 neighbors
```

**Ожидаемый вывод:**
```
IPv6 Address                  Age Link-layer Addr State Interface
2001:DB8:A::1                   0 xxxx.xxxx.xxxx  REACH Gi0/0/0
2001:DB8:A1::xxxx:xxFF:FExx:xx  1 xxxx.xxxx.xxxx  REACH Gi0/0
FE80::xxxx:xxxx:xxxx:xxxx       0 xxxx.xxxx.xxxx  REACH Gi0/0
```

---

### 3. IPv6 Routing Table

```cisco
R1# show ipv6 route
```

**Ожидаемый вывод:**
```
C   2001:DB8:A::/64 [0/0]
     via GigabitEthernet0/0/0, directly connected
L   2001:DB8:A::2/128 [0/0]
     via GigabitEthernet0/0/0, receive
C   2001:DB8:A1::/64 [0/0]
     via GigabitEthernet0/0, directly connected
L   2001:DB8:A1::xxxx:xxFF:FExx:xxxx/128 [0/0]
     via GigabitEthernet0/0, receive
S   ::/0 [1/0]
     via 2001:DB8:A::1
```

Обратите внимание:
- `C` - Connected (прямо подключенные сети)
- `L` - Local (локальные адреса интерфейсов)
- `S` - Static (статические маршруты)

---

### 4. Ping Tests

**Ping IPv6 Internet Gateway:**
```cisco
R1# ping ipv6 2001:db8:a::1
```

**Ping между R1 и CSW1:**
```cisco
R1# ping ipv6 2001:db8:a1::xxxx:xxFF:FExx:xxxx
```

**Ping link-local адреса:**
```cisco
CSW1# ping ipv6 fe80::xxxx:xxxx:xxxx:xxxx%G1/0/1
```

💡 **Важно:** Для link-local пинга обязательно указывать exit interface (%).

---

## 📊 IPv6 Address Summary

### R1 IPv6 Addresses

| Interface | IPv6 Address | Type | Method |
|-----------|-------------|------|--------|
| G0/0/0 | 2001:db8:a::2/64 | Global Unicast | Manual |
| G0/1/0 | 2001:db8:b::2/64 | Global Unicast | Manual |
| G0/0 | 2001:db8:a1::[EUI-64]/64 | Global Unicast | EUI-64 |
| G0/1 | 2001:db8:a2::[EUI-64]/64 | Global Unicast | EUI-64 |
| All | fe80::[link-local] | Link-Local | Auto |

---

### CSW1 IPv6 Addresses

| Interface | IPv6 Address | Type | Method |
|-----------|-------------|------|--------|
| G1/0/1 | 2001:db8:a1::[EUI-64]/64 | Global Unicast | EUI-64 |
| Po1 | fe80::[link-local] | Link-Local | ipv6 enable |

---

### CSW2 IPv6 Addresses

| Interface | IPv6 Address | Type | Method |
|-----------|-------------|------|--------|
| G1/0/1 | 2001:db8:a2::[EUI-64]/64 | Global Unicast | EUI-64 |
| Po1 | fe80::[link-local] | Link-Local | ipv6 enable |

---

## 📊 IPv6 Static Routes Summary

| Route | Next-Hop | Exit Interface | AD | Type | Device |
|-------|----------|----------------|----|----|--------|
| ::/0 | 2001:db8:a::1 | - | 1 | Recursive | R1 |
| ::/0 | 2001:db8:b::1 | G0/1/0 | 2 | Fully Specified (Floating) | R1 |

---

## 💡 Практические советы

### IPv6 Best Practices:
1. ✅ Всегда включайте `ipv6 unicast-routing` первым
2. ✅ Используйте EUI-64 для автоматизации
3. ✅ Link-local адреса автоматически генерируются
4. ✅ Для ping link-local указывайте exit interface (%)
5. ✅ IPv6 не требует ARP (использует NDP - Neighbor Discovery)

### Типичные ошибки:
❌ Забыть `ipv6 unicast-routing` - устройство не маршрутизирует  
❌ Пытаться ping link-local без указания интерфейса  
❌ Путать EUI-64 с ручной настройкой  
❌ Забыть `/64` в конце prefix при использовании EUI-64  
❌ Неправильный формат IPv6 адреса (пропущенные ::)  

---

## 🧪 Тестирование IPv6

### Тест 1: Basic Connectivity

```cisco
! На R1
R1# ping ipv6 2001:db8:a::1
R1# ping ipv6 2001:db8:a1::xxxx:xxFF:FExx:xxxx
```

---

### Тест 2: Neighbor Discovery

```cisco
R1# show ipv6 neighbors
CSW1# show ipv6 neighbors
```

Должны видеть соседей по IPv6.

---

### Тест 3: Link-Local Ping

```cisco
CSW1# show ipv6 interface brief
! Скопируйте link-local адрес CSW2 на Po1

CSW1# ping ipv6 fe80::[CSW2-link-local]%Po1
```

---

### Тест 4: Routing Table

```cisco
R1# show ipv6 route
R1# show ipv6 route static
```

Проверьте наличие:
- Connected routes (C)
- Local routes (L)
- Static default route (S ::/0)

---

## 🎓 Ключевые команды Части 8

```cisco
# Enable IPv6 Routing
ipv6 unicast-routing

# IPv6 Address Configuration
interface [type] [number]
 ipv6 address [address]/[prefix-length]           # Manual
 ipv6 address [prefix]/[prefix-length] eui-64     # EUI-64
 ipv6 enable                                       # Link-local only

# IPv6 Static Routes
ipv6 route [destination]/[prefix] [next-hop]                    # Recursive
ipv6 route [destination]/[prefix] [exit-if]                     # Directly Connected
ipv6 route [destination]/[prefix] [exit-if] [next-hop]         # Fully Specified
ipv6 route [destination]/[prefix] [next-hop] [AD]              # Floating

# Verification
show ipv6 interface brief
show ipv6 interface [type] [number]
show ipv6 route
show ipv6 route static
show ipv6 neighbors
ping ipv6 [address]
ping ipv6 [link-local]%[interface]
```

---

## 📖 Теория IPv6 - Дополнительно

### EUI-64 Process

**Шаг 1:** Возьмите MAC адрес (48 bits)
```
MAC: 00:1A:2B:3C:4D:5E
```

**Шаг 2:** Разделите на две части
```
00:1A:2B | 3C:4D:5E
```

**Шаг 3:** Вставьте FF:FE посередине
```
00:1A:2B:FF:FE:3C:4D:5E
```

**Шаг 4:** Flip 7th bit первого октета
```
00 = 0000 0000
Flip 7th bit (universal/local bit):
02 = 0000 0010

Result: 02:1A:2B:FF:FE:3C:4D:5E
```

**Финальный IPv6 адрес:**
```
Prefix: 2001:db8:a1::/64
EUI-64: 021A:2BFF:FE3C:4D5E

Full address: 2001:db8:a1:0:021A:2BFF:FE3C:4D5E
Compressed: 2001:db8:a1::21a:2bff:fe3c:4d5e
```

---

### IPv6 Address Types Summary

| Type | Prefix | Scope | Description |
|------|--------|-------|-------------|
| Global Unicast | 2000::/3 | Global | Публичные адреса (Internet) |
| Unique Local | fc00::/7 | Private | Частные адреса (как 10.0.0.0/8) |
| Link-Local | fe80::/10 | Link | Только на локальном сегменте |
| Loopback | ::1/128 | Host | Эквивалент 127.0.0.1 |
| Unspecified | ::/128 | - | Эквивалент 0.0.0.0 |
| Multicast | ff00::/8 | Various | Групповые адреса |

---

### IPv6 vs IPv4 Comparison

| Feature | IPv4 | IPv6 |
|---------|------|------|
| **Address Size** | 32 bits | 128 bits |
| **Address Format** | Decimal (192.168.1.1) | Hexadecimal (2001:db8::1) |
| **Address Space** | ~4 billion | 340 undecillion |
| **Broadcast** | Yes | No (uses multicast) |
| **ARP** | Yes | No (uses NDP) |
| **Header Size** | Variable (20-60 bytes) | Fixed (40 bytes) |
| **Fragmentation** | Router + Host | Host only |
| **Configuration** | Manual/DHCP | Manual/SLAAC/DHCPv6 |

---

## 🔍 Troubleshooting IPv6

### Проблема 1: Нет IPv6 адресов на интерфейсах

**Симптомы:**
```cisco
R1# show ipv6 interface brief
! Нет IPv6 адресов
```

**Причина:** IPv6 routing не включен

**Решение:**
```cisco
R1(config)# ipv6 unicast-routing
```

---

### Проблема 2: EUI-64 не работает

**Симптомы:** Адрес не генерируется

**Причины и решения:**

1. **Забыли /64**
```cisco
! Неправильно
ipv6 address 2001:db8:a1:: eui-64

! Правильно
ipv6 address 2001:db8:a1::/64 eui-64
```

2. **Интерфейс в down состоянии**
```cisco
interface g0/0
 no shutdown
```

---

### Проблема 3: Ping link-local не работает

**Симптомы:**
```cisco
R1# ping ipv6 fe80::1
% Invalid source address
```

**Причина:** Не указан exit interface

**Решение:**
```cisco
R1# ping ipv6 fe80::1%g0/0
```

---

### Проблема 4: Default route не работает

**Симптомы:** Нет connectivity к Internet

**Проверка:**
```cisco
R1# show ipv6 route
! Нет маршрута ::/0
```

**Решение:**
```cisco
ipv6 route ::/0 2001:db8:a::1
```

---

## 🎯 Чек-лист завершения Части 8

### IPv6 Routing:
- [ ] IPv6 unicast-routing включен на R1
- [ ] IPv6 unicast-routing включен на CSW1
- [ ] IPv6 unicast-routing включен на CSW2
- [ ] Link-local адреса автоматически сгенерированы

### R1 Configuration:
- [ ] G0/0/0: 2001:db8:a::2/64 (manual)
- [ ] G0/1/0: 2001:db8:b::2/64 (manual)
- [ ] G0/0: 2001:db8:a1::/64 (EUI-64)
- [ ] G0/1: 2001:db8:a2::/64 (EUI-64)

### CSW1 Configuration:
- [ ] G1/0/1: 2001:db8:a1::/64 (EUI-64)
- [ ] Po1: ipv6 enable (link-local only)

### CSW2 Configuration:
- [ ] G1/0/1: 2001:db8:a2::/64 (EUI-64)
- [ ] Po1: ipv6 enable (link-local only)

### Static Routes:
- [ ] Primary default route на R1 (::/0 → 2001:db8:a::1)
- [ ] Floating default route на R1 (AD=2)
- [ ] Primary route активен в routing table

### Verification:
- [ ] Show ipv6 interface brief работает
- [ ] Show ipv6 route показывает маршруты
- [ ] Show ipv6 neighbors показывает соседей
- [ ] Ping 2001:db8:a::1 работает (ISP A)
- [ ] Ping между R1 и CSW1 работает
- [ ] Ping между R1 и CSW2 работает
- [ ] Ping link-local с % работает

### General:
- [ ] Все конфигурации сохранены (write memory)
- [ ] IPv6 не ломает IPv4 connectivity
- [ ] Dual-stack частично работает (IPv4 + IPv6)

---

## 📚 IPv6 для CCNA Exam

### Важные концепции для экзамена:

1. **IPv6 Address Shortening Rules:**
   - Leading zeros можно опустить: 0001 → 1
   - Consecutive zeros заменяются на ::: (только ОДИН раз)
   - Пример: 2001:0db8:0000:0000:0000:0000:0000:0001
   - → 2001:db8::1

2. **EUI-64:**
   - MAC разделяется на две части
   - FF:FE вставляется посередине
   - 7th bit первого октета инвертируется

3. **Link-Local:**
   - Всегда начинается с fe80::/10
   - Автоматически генерируется при включении IPv6
   - Используется для NDP, routing protocols

4. **IPv6 Routing:**
   - Требует `ipv6 unicast-routing`
   - NDP вместо ARP
   - Нет broadcast (только multicast)

5. **Static Routes:**
   - ::/0 = default route
   - Recursive, Directly Connected, Fully Specified
   - AD работает так же как в IPv4

---

## 🚀 Готовы к Части 9?

В последней части мы настроим **Wireless (Wi-Fi)**:

### Wireless (Часть 9):
1. 📡 **WLC Configuration** - настройка контроллера
2. 🌐 **Dynamic Interface** - интерфейс для Wi-Fi VLAN
3. 📶 **WLAN Creation** - создание беспроводной сети
4. 🔐 **WPA2-PSK** - безопасность Wi-Fi
5. ✅ **LWAP Association** - подключение точек доступа
6. 💻 **Wireless Client Test** - проверка работы

Это финальная часть курса!

**До встречи в Части 9! 🎓**