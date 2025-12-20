# CCNA Мини-курс на основе Jeremy's Mega Lab
## Часть 2: VLANs & Layer-2 EtherChannel

---

## 📋 Теоретический минимум

### Ключевые концепции:

**1. VLAN** - логическое разделение коммутатора на изолированные сегменты
**2. Trunk** - канал, передающий трафик нескольких VLAN (802.1Q тегирование)
**3. Access Port** - порт для конечных устройств (один VLAN)
**4. EtherChannel** - агрегация каналов (PAgP - Cisco, LACP - стандарт)
**5. VTP** - синхронизация VLAN между коммутаторами

---

## 🎯 Цели раздела

- ✅ Настроить Layer-2 EtherChannel (PAgP и LACP)
- ✅ Настроить trunk порты с правильными параметрами
- ✅ Создать и назначить VLANs
- ✅ Настроить VTP для синхронизации VLAN
- ✅ Настроить access порты для конечных устройств
- ✅ Отключить неиспользуемые порты

---

## 🗺️ Структура VLANs

### Office A:
- **VLAN 10** - PCs (10.1.0.0/24)
- **VLAN 20** - Phones (10.2.0.0/24)
- **VLAN 40** - Wi-Fi (10.6.0.0/24)
- **VLAN 99** - Management (10.0.0.0/28)

### Office B:
- **VLAN 10** - PCs (10.3.0.0/24)
- **VLAN 20** - Phones (10.4.0.0/24)
- **VLAN 30** - Servers (10.5.0.0/24)
- **VLAN 99** - Management (10.0.0.16/28)

---

## 📝 ЧАСТЬ 1: EtherChannel Configuration

### ШАГ 1: PAgP EtherChannel (Office A)

**DSW-A1 ↔ DSW-A2 через G1/0/4-5**

```cisco
! На DSW-A1
DSW-A1(config)# interface range g1/0/4-5
DSW-A1(config-if-range)# channel-group 1 mode desirable
DSW-A1(config-if-range)# exit

! На DSW-A2
DSW-A2(config)# interface range g1/0/4-5
DSW-A2(config-if-range)# channel-group 1 mode desirable
DSW-A2(config-if-range)# exit
```

**Проверка:**
```cisco
DSW-A2# show etherchannel summary
```
Ожидается: `Po1(SU)` и флаг `P` на портах

---

### ШАГ 2: LACP EtherChannel (Office B)

**DSW-B1 ↔ DSW-B2 через G1/0/4-5**

```cisco
! На DSW-B1
DSW-B1(config)# interface range g1/0/4-5
DSW-B1(config-if-range)# channel-group 1 mode active
DSW-B1(config-if-range)# exit

! На DSW-B2
DSW-B2(config)# interface range g1/0/4-5
DSW-B2(config-if-range)# channel-group 1 mode active
DSW-B2(config-if-range)# exit
```

**Проверка:**
```cisco
DSW-B2# show etherchannel summary
```

💡 **PAgP vs LACP:**
- PAgP: режимы `desirable` (активный) / `auto` (пассивный) - Cisco proprietary
- LACP: режимы `active` (активный) / `passive` (пассивный) - IEEE стандарт

---

## 📝 ЧАСТЬ 2: Trunk Configuration

### ШАГ 3: Настройка Trunk портов

**Параметры для всех trunk:**
- Mode: trunk
- DTP: отключен (`switchport nonegotiate`)
- Native VLAN: 1000 (безопасность)
- Allowed VLANs: Office A (10,20,40,99), Office B (10,20,30,99)

---

### Office A: DSW-A1

```cisco
! Trunk к Access switches
DSW-A1(config)# interface range g1/0/1-3
DSW-A1(config-if-range)# switchport mode trunk
DSW-A1(config-if-range)# switchport nonegotiate
DSW-A1(config-if-range)# switchport trunk native vlan 1000
DSW-A1(config-if-range)# switchport trunk allowed vlan 10,20,40,99
DSW-A1(config-if-range)# exit

! Trunk на PortChannel
DSW-A1(config)# interface port-channel 1
DSW-A1(config-if)# switchport mode trunk
DSW-A1(config-if)# switchport nonegotiate
DSW-A1(config-if)# switchport trunk native vlan 1000
DSW-A1(config-if)# switchport trunk allowed vlan 10,20,40,99
DSW-A1(config-if)# exit
```

---

### Office A: DSW-A2

```cisco
DSW-A2(config)# interface range g1/0/1-3
DSW-A2(config-if-range)# switchport mode trunk
DSW-A2(config-if-range)# switchport nonegotiate
DSW-A2(config-if-range)# switchport trunk native vlan 1000
DSW-A2(config-if-range)# switchport trunk allowed vlan 10,20,40,99
DSW-A2(config-if-range)# exit

DSW-A2(config)# interface port-channel 1
DSW-A2(config-if)# switchport mode trunk
DSW-A2(config-if)# switchport nonegotiate
DSW-A2(config-if)# switchport trunk native vlan 1000
DSW-A2(config-if)# switchport trunk allowed vlan 10,20,40,99
DSW-A2(config-if)# exit
```

---

### Office A: Access Switches (ASW-A1, A2, A3)

```cisco
! Одинаковые команды для всех трёх
ASW-A1(config)# interface range g0/1-2
ASW-A1(config-if-range)# switchport mode trunk
ASW-A1(config-if-range)# switchport nonegotiate
ASW-A1(config-if-range)# switchport trunk native vlan 1000
ASW-A1(config-if-range)# switchport trunk allowed vlan 10,20,40,99
ASW-A1(config-if-range)# exit
```

Повторите на **ASW-A2** и **ASW-A3**

---

### Office B: DSW-B1

```cisco
DSW-B1(config)# interface range g1/0/1-3
DSW-B1(config-if-range)# switchport mode trunk
DSW-B1(config-if-range)# switchport nonegotiate
DSW-B1(config-if-range)# switchport trunk native vlan 1000
DSW-B1(config-if-range)# switchport trunk allowed vlan 10,20,30,99
DSW-B1(config-if-range)# exit

DSW-B1(config)# interface port-channel 1
DSW-B1(config-if)# switchport mode trunk
DSW-B1(config-if)# switchport nonegotiate
DSW-B1(config-if)# switchport trunk native vlan 1000
DSW-B1(config-if)# switchport trunk allowed vlan 10,20,30,99
DSW-B1(config-if)# exit
```

---

### Office B: DSW-B2

```cisco
DSW-B2(config)# interface range g1/0/1-3
DSW-B2(config-if-range)# switchport mode trunk
DSW-B2(config-if-range)# switchport nonegotiate
DSW-B2(config-if-range)# switchport trunk native vlan 1000
DSW-B2(config-if-range)# switchport trunk allowed vlan 10,20,30,99
DSW-B2(config-if-range)# exit

DSW-B2(config)# interface port-channel 1
DSW-B2(config-if)# switchport mode trunk
DSW-B2(config-if)# switchport nonegotiate
DSW-B2(config-if)# switchport trunk native vlan 1000
DSW-B2(config-if)# switchport trunk allowed vlan 10,20,30,99
DSW-B2(config-if)# exit
```

---

### Office B: Access Switches (ASW-B1, B2, B3)

```cisco
! Одинаковые команды для всех трёх
ASW-B1(config)# interface range g0/1-2
ASW-B1(config-if-range)# switchport mode trunk
ASW-B1(config-if-range)# switchport nonegotiate
ASW-B1(config-if-range)# switchport trunk native vlan 1000
ASW-B1(config-if-range)# switchport trunk allowed vlan 10,20,30,99
ASW-B1(config-if-range)# exit
```

Повторите на **ASW-B2** и **ASW-B3**

---

## 📝 ЧАСТЬ 3: VTP Configuration

### ШАГ 4: Настройка VTP

**Концепция:**
- VTP Domain: `JeremysITLab`
- VTP Version: 2
- DSW = Server, ASW = Client

---

### Office A: VTP Setup

```cisco
! DSW-A1 (Server)
DSW-A1(config)# vtp domain JeremysITLab
DSW-A1(config)# vtp version 2
DSW-A1(config)# exit

! ASW-A1 (Client)
ASW-A1(config)# vtp mode client
ASW-A1(config)# exit

! ASW-A2 (Client)
ASW-A2(config)# vtp mode client
ASW-A2(config)# exit

! ASW-A3 (Client)
ASW-A3(config)# vtp mode client
ASW-A3(config)# exit
```

**Проверка:**
```cisco
ASW-A3# show vtp status
```

---

### Office B: VTP Setup

```cisco
! DSW-B1 (Server)
DSW-B1(config)# vtp domain JeremysITLab
DSW-B1(config)# vtp version 2
DSW-B1(config)# exit

! ASW-B1 (Client)
ASW-B1(config)# vtp mode client

! ASW-B2 (Client)
ASW-B2(config)# vtp mode client

! ASW-B3 (Client)
ASW-B3(config)# vtp mode client
```

---

## 📝 ЧАСТЬ 4: VLAN Creation

### ШАГ 5: Создание VLANs Office A

**На DSW-A1 (VTP распространит на остальные):**

```cisco
DSW-A1(config)# vlan 10
DSW-A1(config-vlan)# name PCs
DSW-A1(config-vlan)# exit

DSW-A1(config)# vlan 20
DSW-A1(config-vlan)# name Phones
DSW-A1(config-vlan)# exit

DSW-A1(config)# vlan 40
DSW-A1(config-vlan)# name Wi-Fi
DSW-A1(config-vlan)# exit

DSW-A1(config)# vlan 99
DSW-A1(config-vlan)# name Management
DSW-A1(config-vlan)# exit
```

**Проверка:**
```cisco
DSW-A1# show vlan brief
ASW-A3# show vlan brief
```

---

### ШАГ 6: Создание VLANs Office B

**На DSW-B1:**

```cisco
DSW-B1(config)# vlan 10
DSW-B1(config-vlan)# name PCs
DSW-B1(config-vlan)# exit

DSW-B1(config)# vlan 20
DSW-B1(config-vlan)# name Phones
DSW-B1(config-vlan)# exit

DSW-B1(config)# vlan 30
DSW-B1(config-vlan)# name Servers
DSW-B1(config-vlan)# exit

DSW-B1(config)# vlan 99
DSW-B1(config-vlan)# name Management
DSW-B1(config-vlan)# exit
```

**Проверка:**
```cisco
ASW-B3# show vlan brief
```

---

## 📝 ЧАСТЬ 5: Access Port Configuration

### ШАГ 7: Настройка Access портов

**Типы подключений:**
- LWAP (ASW-A1, B1): VLAN 99
- PC + Phone (ASW-A2, A3, B2): VLAN 10 + Voice VLAN 20
- Server (ASW-B3): VLAN 30

---

### ASW-A1: LWAP подключение

```cisco
ASW-A1(config)# interface f0/1
ASW-A1(config-if)# switchport mode access
ASW-A1(config-if)# switchport nonegotiate
ASW-A1(config-if)# switchport access vlan 99
ASW-A1(config-if)# exit
```

---

### ASW-B1: LWAP подключение

```cisco
ASW-B1(config)# interface f0/1
ASW-B1(config-if)# switchport mode access
ASW-B1(config-if)# switchport nonegotiate
ASW-B1(config-if)# switchport access vlan 99
ASW-B1(config-if)# exit
```

---

### ASW-A2, A3, B2: PC + IP Phone

```cisco
! ASW-A2
ASW-A2(config)# interface f0/1
ASW-A2(config-if)# switchport mode access
ASW-A2(config-if)# switchport nonegotiate
ASW-A2(config-if)# switchport access vlan 10
ASW-A2(config-if)# switchport voice vlan 20
ASW-A2(config-if)# exit

! ASW-A3
ASW-A3(config)# interface f0/1
ASW-A3(config-if)# switchport mode access
ASW-A3(config-if)# switchport nonegotiate
ASW-A3(config-if)# switchport access vlan 10
ASW-A3(config-if)# switchport voice vlan 20
ASW-A3(config-if)# exit

! ASW-B2
ASW-B2(config)# interface f0/1
ASW-B2(config-if)# switchport mode access
ASW-B2(config-if)# switchport nonegotiate
ASW-B2(config-if)# switchport access vlan 10
ASW-B2(config-if)# switchport voice vlan 20
ASW-B2(config-if)# exit
```

💡 **Voice VLAN:**
- Data (PC) → VLAN 10 (untagged)
- Voice (Phone) → VLAN 20 (tagged)

---

### ASW-B3: Server подключение

```cisco
ASW-B3(config)# interface f0/1
ASW-B3(config-if)# switchport mode access
ASW-B3(config-if)# switchport nonegotiate
ASW-B3(config-if)# switchport access vlan 30
ASW-B3(config-if)# exit
```

---

### ШАГ 8: WLC1 подключение (ASW-A1)

**Специальная настройка для WLC:**

```cisco
ASW-A1(config)# interface f0/2
ASW-A1(config-if)# switchport mode trunk
ASW-A1(config-if)# switchport trunk allowed vlan 40,99
ASW-A1(config-if)# switchport trunk native vlan 99
ASW-A1(config-if)# switchport nonegotiate
ASW-A1(config-if)# exit
```

**Объяснение:**
- VLAN 99 (Management) - native/untagged
- VLAN 40 (Wi-Fi) - tagged для клиентов

---

## 📝 ЧАСТЬ 6: Shutdown Unused Ports

### ШАГ 9: Отключение неиспользуемых портов

### Distribution Switches

```cisco
! DSW-A1, DSW-A2, DSW-B1, DSW-B2
DSW-A1(config)# interface range g1/0/6-24, g1/1/3-4
DSW-A1(config-if-range)# shutdown
DSW-A1(config-if-range)# exit
```

Повторите на **DSW-A2, DSW-B1, DSW-B2**

---

### Access Switches

```cisco
! ASW-A1 (особый случай - WLC на F0/2)
ASW-A1(config)# interface range f0/3-24
ASW-A1(config-if-range)# shutdown
ASW-A1(config-if-range)# exit

! ASW-A2, A3, B1, B2, B3
ASW-A2(config)# interface range f0/2-24
ASW-A2(config-if-range)# shutdown
ASW-A2(config-if-range)# exit
```

Повторите на **ASW-A3, ASW-B1, ASW-B2, ASW-B3**

---

## ✅ Проверка конфигурации

### 1. EtherChannel
```cisco
show etherchannel summary
```
Ожидается: `Po1(SU)` и флаг `P` на портах

### 2. Trunk
```cisco
show interfaces trunk
show interfaces g1/0/1 switchport
```

### 3. VTP
```cisco
show vtp status
```

### 4. VLANs
```cisco
show vlan brief
```

### 5. Access порты
```cisco
show interfaces f0/1 switchport
```

### 6. Отключенные порты
```cisco
show interfaces status | include disabled
```

---

## 📊 Справочные таблицы

### VLANs Summary

| VLAN | Название | Office A | Office B |
|------|----------|----------|----------|
| 10 | PCs | ✅ | ✅ |
| 20 | Phones | ✅ | ✅ |
| 30 | Servers | ❌ | ✅ |
| 40 | Wi-Fi | ✅ | ❌ |
| 99 | Management | ✅ | ✅ |

### EtherChannel Summary

| Связь | Протокол | Режим | PortChannel |
|-------|----------|-------|-------------|
| DSW-A1 ↔ DSW-A2 | PAgP | desirable | Po1 |
| DSW-B1 ↔ DSW-B2 | LACP | active | Po1 |

### Access Ports Summary

| Switch | Port | VLAN | Voice | Устройство |
|--------|------|------|-------|------------|
| ASW-A1 | F0/1 | 99 | - | LWAP1 |
| ASW-A1 | F0/2 | trunk | - | WLC1 |
| ASW-A2 | F0/1 | 10 | 20 | PC+Phone |
| ASW-A3 | F0/1 | 10 | 20 | PC+Phone |
| ASW-B1 | F0/1 | 99 | - | LWAP2 |
| ASW-B2 | F0/1 | 10 | 20 | PC+Phone |
| ASW-B3 | F0/1 | 30 | - | SRV1 |

---

## 💡 Практические советы

### Ускорение работы:
1. Подготовьте команды в Notepad
2. Используйте Copy-Paste (правый клик → Paste)
3. Сохраняйте после каждого устройства: `do write`

### Типичные ошибки:
❌ Забыть `switchport nonegotiate`
❌ Неправильный список VLAN (40 vs 30)
❌ Забыть trunk на PortChannel
❌ Не установить VTP mode client
❌ Забыть `do write`

---

## 🎓 Ключевые команды

```cisco
# EtherChannel
channel-group [number] mode {desirable|active}
show etherchannel summary

# Trunk
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan [vlan-id]
switchport trunk allowed vlan [vlan-list]
show interfaces trunk

# VLAN
vlan [vlan-id]
name [name]
show vlan brief

# VTP
vtp domain [name]
vtp version 2
vtp mode {server|client|transparent}
show vtp status

# Access Port
switchport mode access
switchport access vlan [vlan-id]
switchport voice vlan [vlan-id]

# Shutdown
shutdown
show interfaces status
```

---

## 🎯 Чек-лист завершения Части 2

- [ ] EtherChannel PAgP настроен (DSW-A1 ↔ A2)
- [ ] EtherChannel LACP настроен (DSW-B1 ↔ B2)
- [ ] Все trunk порты настроены
- [ ] VTP настроен в обоих офисах
- [ ] VLANs созданы и синхронизированы
- [ ] Access порты настроены
- [ ] WLC1 trunk настроен
- [ ] Неиспользуемые порты отключены
- [ ] Все конфигурации сохранены

---

## 🚀 Готовы к Части 3?

В следующей части:
- **IPv4 адресация** на всех устройствах
- **Layer-3 EtherChannel** между Core switches
- **HSRP** для резервирования шлюзов
- **Management IP** на Access switches

**До встречи в Части 3! 🎓**