# CCNA Мини-курс на основе Jeremy's Mega Lab
## Часть 3: IP Addressing, Layer-3 EtherChannel & HSRP

---

## 📋 Теоретический минимум

### Ключевые концепции:

**1. Layer-3 Interface** - коммутатор работает на уровне маршрутизации
- Routed Port: `no switchport` на физическом порту
- SVI (Switch Virtual Interface): `interface vlan [id]`

**2. Layer-3 EtherChannel** - агрегация на L3 уровне
- IP адрес настраивается на Port-channel интерфейсе
- Физические порты: `no switchport` + `channel-group`

**3. HSRP (Hot Standby Router Protocol)** - Cisco proprietary
- Виртуальный IP (VIP) для резервирования шлюза
- Active и Standby роутеры
- Priority: 0-255 (default 100)
- Preemption: возврат роли Active

**4. IP Routing на коммутаторах**
- `ip routing` - включение маршрутизации
- Multilayer switches могут быть маршрутизаторами

---

## 🎯 Цели раздела

- ✅ Настроить IP адреса на R1
- ✅ Включить IP routing на коммутаторах
- ✅ Настроить Layer-3 EtherChannel (CSW1 ↔ CSW2)
- ✅ Настроить IP адреса на Core switches
- ✅ Настроить IP адреса на Distribution switches
- ✅ Настроить IP адрес на SRV1
- ✅ Настроить Management IP на Access switches
- ✅ Настроить HSRP для резервирования

---

## 🗺️ IP Addressing Plan

### R1 Interfaces:
- G0/0/0: DHCP (Internet ISP A)
- G0/1/0: DHCP (Internet ISP B)
- G0/0: 10.0.0.33/30 (→ CSW1)
- G0/1: 10.0.0.37/30 (→ CSW2)
- Lo0: 10.0.0.76/32

### Core Switches:
- CSW1 Lo0: 10.0.0.77/32
- CSW2 Lo0: 10.0.0.78/32
- Po1: 10.0.0.40/30 (CSW1: .41, CSW2: .42)

### Distribution Switches Office A:
- DSW-A1 Lo0: 10.0.0.79/32
- DSW-A2 Lo0: 10.0.0.80/32

### Distribution Switches Office B:
- DSW-B1 Lo0: 10.0.0.81/32
- DSW-B2 Lo0: 10.0.0.82/32

---

## 📝 ЧАСТЬ 1: R1 Configuration

### ШАГ 1: Настройка R1 Interfaces

```cisco
R1(config)# interface range g0/0/0, g0/1/0
R1(config-if-range)# ip address dhcp
R1(config-if-range)# no shutdown
R1(config-if-range)# exit

R1(config)# interface g0/0
R1(config-if)# ip address 10.0.0.33 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface g0/1
R1(config-if)# ip address 10.0.0.37 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface loopback 0
R1(config-if)# ip address 10.0.0.76 255.255.255.255
R1(config-if)# exit
R1(config)# do write
```

**Проверка:**
```cisco
R1# show ip interface brief
```

Ожидается:
- G0/0/0: получил IP через DHCP
- G0/1/0: получил IP через DHCP
- G0/0: 10.0.0.33
- G0/1: 10.0.0.37
- Lo0: 10.0.0.76

---

## 📝 ЧАСТЬ 2: Enable IP Routing

### ШАГ 2: Включение IP Routing на коммутаторах

```cisco
! CSW1
CSW1(config)# ip routing

! CSW2
CSW2(config)# ip routing

! DSW-A1
DSW-A1(config)# ip routing

! DSW-A2
DSW-A2(config)# ip routing

! DSW-B1
DSW-B1(config)# ip routing

! DSW-B2
DSW-B2(config)# ip routing
```

💡 **Важно:** Без `ip routing` коммутатор не будет маршрутизировать пакеты между VLAN!

---

## 📝 ЧАСТЬ 3: Layer-3 EtherChannel

### ШАГ 3: CSW1 ↔ CSW2 Layer-3 EtherChannel

**CSW1 Configuration:**
```cisco
CSW1(config)# interface range g1/0/2-3
CSW1(config-if-range)# no switchport
CSW1(config-if-range)# channel-group 1 mode desirable
CSW1(config-if-range)# exit

CSW1(config)# interface port-channel 1
CSW1(config-if)# ip address 10.0.0.41 255.255.255.252
CSW1(config-if)# exit
```

**CSW2 Configuration:**
```cisco
CSW2(config)# interface range g1/0/2-3
CSW2(config-if-range)# no switchport
CSW2(config-if-range)# channel-group 1 mode desirable
CSW2(config-if-range)# exit

CSW2(config)# interface port-channel 1
CSW2(config-if)# ip address 10.0.0.42 255.255.255.252
CSW2(config-if)# exit
```

**Проверка:**
```cisco
CSW2# show etherchannel summary
```
Ожидается: флаги `RU` (Layer3, In use)

**Тест связности:**
```cisco
CSW2# ping 10.0.0.41
```

💡 **Ключевое отличие L2 vs L3 EtherChannel:**
- L2: флаг `S` (Layer2), IP на VLAN
- L3: флаг `R` (Layer3), IP на Port-channel

---

## 📝 ЧАСТЬ 4: Core Switches IP Configuration

### ШАГ 4 и 5: CSW1 и CSW2

**CSW1:**
```cisco
CSW1(config)# interface g1/0/1
CSW1(config-if)# no switchport
CSW1(config-if)# ip address 10.0.0.34 255.255.255.252
CSW1(config-if)# no shutdown
CSW1(config-if)# exit

CSW1(config)# interface g1/1/1
CSW1(config-if)# no switchport
CSW1(config-if)# ip address 10.0.0.45 255.255.255.252
CSW1(config-if)# no shutdown
CSW1(config-if)# exit

CSW1(config)# interface g1/1/2
CSW1(config-if)# no switchport
CSW1(config-if)# ip address 10.0.0.49 255.255.255.252
CSW1(config-if)# no shutdown
CSW1(config-if)# exit

CSW1(config)# interface g1/1/3
CSW1(config-if)# no switchport
CSW1(config-if)# ip address 10.0.0.53 255.255.255.252
CSW1(config-if)# no shutdown
CSW1(config-if)# exit

CSW1(config)# interface g1/1/4
CSW1(config-if)# no switchport
CSW1(config-if)# ip address 10.0.0.57 255.255.255.252
CSW1(config-if)# no shutdown
CSW1(config-if)# exit

CSW1(config)# interface loopback 0
CSW1(config-if)# ip address 10.0.0.77 255.255.255.255
CSW1(config-if)# exit

CSW1(config)# interface range g1/0/4-24
CSW1(config-if-range)# shutdown
CSW1(config-if-range)# exit
CSW1(config)# do write
```

---

**CSW2:**
```cisco
CSW2(config)# interface g1/0/1
CSW2(config-if)# no switchport
CSW2(config-if)# ip address 10.0.0.38 255.255.255.252
CSW2(config-if)# no shutdown
CSW2(config-if)# exit

CSW2(config)# interface g1/1/1
CSW2(config-if)# no switchport
CSW2(config-if)# ip address 10.0.0.61 255.255.255.252
CSW2(config-if)# no shutdown
CSW2(config-if)# exit

CSW2(config)# interface g1/1/2
CSW2(config-if)# no switchport
CSW2(config-if)# ip address 10.0.0.65 255.255.255.252
CSW2(config-if)# no shutdown
CSW2(config-if)# exit

CSW2(config)# interface g1/1/3
CSW2(config-if)# no switchport
CSW2(config-if)# ip address 10.0.0.69 255.255.255.252
CSW2(config-if)# no shutdown
CSW2(config-if)# exit

CSW2(config)# interface g1/1/4
CSW2(config-if)# no switchport
CSW2(config-if)# ip address 10.0.0.73 255.255.255.252
CSW2(config-if)# no shutdown
CSW2(config-if)# exit

CSW2(config)# interface loopback 0
CSW2(config-if)# ip address 10.0.0.78 255.255.255.255
CSW2(config-if)# exit

CSW2(config)# interface range g1/0/4-24
CSW2(config-if-range)# shutdown
CSW2(config-if-range)# exit
CSW2(config)# do write
```

---

## 📝 ЧАСТЬ 5: Distribution Switches

### ШАГ 6-9: Distribution Switches IP Configuration

**DSW-A1:**
```cisco
DSW-A1(config)# interface g1/1/1
DSW-A1(config-if)# no switchport
DSW-A1(config-if)# ip address 10.0.0.46 255.255.255.252
DSW-A1(config-if)# no shutdown
DSW-A1(config-if)# exit

DSW-A1(config)# interface g1/1/2
DSW-A1(config-if)# no switchport
DSW-A1(config-if)# ip address 10.0.0.62 255.255.255.252
DSW-A1(config-if)# no shutdown
DSW-A1(config-if)# exit

DSW-A1(config)# interface loopback 0
DSW-A1(config-if)# ip address 10.0.0.79 255.255.255.255
DSW-A1(config-if)# exit
```

---

**DSW-A2:**
```cisco
DSW-A2(config)# interface g1/1/1
DSW-A2(config-if)# no switchport
DSW-A2(config-if)# ip address 10.0.0.50 255.255.255.252
DSW-A2(config-if)# no shutdown
DSW-A2(config-if)# exit

DSW-A2(config)# interface g1/1/2
DSW-A2(config-if)# no switchport
DSW-A2(config-if)# ip address 10.0.0.66 255.255.255.252
DSW-A2(config-if)# no shutdown
DSW-A2(config-if)# exit

DSW-A2(config)# interface loopback 0
DSW-A2(config-if)# ip address 10.0.0.80 255.255.255.255
DSW-A2(config-if)# exit
```

---

**DSW-B1:**
```cisco
DSW-B1(config)# interface g1/1/1
DSW-B1(config-if)# no switchport
DSW-B1(config-if)# ip address 10.0.0.54 255.255.255.252
DSW-B1(config-if)# no shutdown
DSW-B1(config-if)# exit

DSW-B1(config)# interface g1/1/2
DSW-B1(config-if)# no switchport
DSW-B1(config-if)# ip address 10.0.0.70 255.255.255.252
DSW-B1(config-if)# no shutdown
DSW-B1(config-if)# exit

DSW-B1(config)# interface loopback 0
DSW-B1(config-if)# ip address 10.0.0.81 255.255.255.255
DSW-B1(config-if)# exit
```

---

**DSW-B2:**
```cisco
DSW-B2(config)# interface g1/1/1
DSW-B2(config-if)# no switchport
DSW-B2(config-if)# ip address 10.0.0.58 255.255.255.252
DSW-B2(config-if)# no shutdown
DSW-B2(config-if)# exit

DSW-B2(config)# interface g1/1/2
DSW-B2(config-if)# no switchport
DSW-B2(config-if)# ip address 10.0.0.74 255.255.255.252
DSW-B2(config-if)# no shutdown
DSW-B2(config-if)# exit

DSW-B2(config)# interface loopback 0
DSW-B2(config-if)# ip address 10.0.0.82 255.255.255.255
DSW-B2(config-if)# exit
```

---

## 📝 ЧАСТЬ 6: SRV1 Configuration

### ШАГ 10: Настройка SRV1 через GUI

**В Packet Tracer:**
1. Откройте SRV1
2. Перейдите на вкладку **Config**
3. В разделе **GLOBAL** → **Settings**:
   - Default Gateway: `10.5.0.1`
4. В разделе **INTERFACE** → **FastEthernet0**:
   - IPv4 Address: `10.5.0.4`
   - Subnet Mask: `255.255.255.0`

💡 **Важно:** SRV1 настраивается через GUI, не через CLI!

---

## 📝 ЧАСТЬ 7: Access Switches Management

### ШАГ 11: Management IP на Access Switches

**Office A (10.0.0.0/28):**

```cisco
! ASW-A1
ASW-A1(config)# ip default-gateway 10.0.0.1
ASW-A1(config)# interface vlan 99
ASW-A1(config-if)# ip address 10.0.0.4 255.255.255.240
ASW-A1(config-if)# exit
ASW-A1(config)# do write

! ASW-A2
ASW-A2(config)# ip default-gateway 10.0.0.1
ASW-A2(config)# interface vlan 99
ASW-A2(config-if)# ip address 10.0.0.5 255.255.255.240
ASW-A2(config-if)# exit
ASW-A2(config)# do write

! ASW-A3
ASW-A3(config)# ip default-gateway 10.0.0.1
ASW-A3(config)# interface vlan 99
ASW-A3(config-if)# ip address 10.0.0.6 255.255.255.240
ASW-A3(config-if)# exit
ASW-A3(config)# do write
```

---

**Office B (10.0.0.16/28):**

```cisco
! ASW-B1
ASW-B1(config)# ip default-gateway 10.0.0.17
ASW-B1(config)# interface vlan 99
ASW-B1(config-if)# ip address 10.0.0.20 255.255.255.240
ASW-B1(config-if)# exit
ASW-B1(config)# do write

! ASW-B2
ASW-B2(config)# ip default-gateway 10.0.0.17
ASW-B2(config)# interface vlan 99
ASW-B2(config-if)# ip address 10.0.0.21 255.255.255.240
ASW-B2(config-if)# exit
ASW-B2(config)# do write

! ASW-B3
ASW-B3(config)# ip default-gateway 10.0.0.17
ASW-B3(config)# interface vlan 99
ASW-B3(config-if)# ip address 10.0.0.22 255.255.255.240
ASW-B3(config-if)# exit
ASW-B3(config)# do write
```

💡 **Важно:** 
- L2 коммутаторы используют `ip default-gateway`
- L3 коммутаторы используют маршрутизацию

---

## 📝 ЧАСТЬ 8: HSRP Configuration

### Концепция HSRP в лабе:

**Office A:**
- DSW-A1 = Active для VLAN 10, 99
- DSW-A2 = Active для VLAN 20, 40

**Office B:**
- DSW-B1 = Active для VLAN 10, 99
- DSW-B2 = Active для VLAN 20, 30

**HSRP Parameters:**
- Version: 2
- Priority: Active = 105, Standby = 100 (default)
- Preemption: только на Active

---

### ШАГ 12: Office A - Management VLAN (Group 1)

**DSW-A1 (Active):**
```cisco
DSW-A1(config)# interface vlan 99
DSW-A1(config-if)# ip address 10.0.0.2 255.255.255.240
DSW-A1(config-if)# standby version 2
DSW-A1(config-if)# standby 1 ip 10.0.0.1
DSW-A1(config-if)# standby 1 priority 105
DSW-A1(config-if)# standby 1 preempt
DSW-A1(config-if)# exit
```

**DSW-A2 (Standby):**
```cisco
DSW-A2(config)# interface vlan 99
DSW-A2(config-if)# ip address 10.0.0.3 255.255.255.240
DSW-A2(config-if)# standby version 2
DSW-A2(config-if)# standby 1 ip 10.0.0.1
DSW-A2(config-if)# exit
```

---

### ШАГ 13: Office A - PCs VLAN (Group 2)

**DSW-A1 (Active):**
```cisco
DSW-A1(config)# interface vlan 10
DSW-A1(config-if)# ip address 10.1.0.2 255.255.255.0
DSW-A1(config-if)# standby version 2
DSW-A1(config-if)# standby 2 ip 10.1.0.1
DSW-A1(config-if)# standby 2 priority 105
DSW-A1(config-if)# standby 2 preempt
DSW-A1(config-if)# exit
```

**DSW-A2 (Standby):**
```cisco
DSW-A2(config)# interface vlan 10
DSW-A2(config-if)# ip address 10.1.0.3 255.255.255.0
DSW-A2(config-if)# standby version 2
DSW-A2(config-if)# standby 2 ip 10.1.0.1
DSW-A2(config-if)# exit
```

---

### ШАГ 14: Office A - Phones VLAN (Group 3)

**DSW-A2 (Active):**
```cisco
DSW-A2(config)# interface vlan 20
DSW-A2(config-if)# ip address 10.2.0.3 255.255.255.0
DSW-A2(config-if)# standby version 2
DSW-A2(config-if)# standby 3 ip 10.2.0.1
DSW-A2(config-if)# standby 3 priority 105
DSW-A2(config-if)# standby 3 preempt
DSW-A2(config-if)# exit
```

**DSW-A1 (Standby):**
```cisco
DSW-A1(config)# interface vlan 20
DSW-A1(config-if)# ip address 10.2.0.2 255.255.255.0
DSW-A1(config-if)# standby version 2
DSW-A1(config-if)# standby 3 ip 10.2.0.1
DSW-A1(config-if)# exit
```

---

### ШАГ 15: Office A - Wi-Fi VLAN (Group 4)

**DSW-A2 (Active):**
```cisco
DSW-A2(config)# interface vlan 40
DSW-A2(config-if)# ip address 10.6.0.3 255.255.255.0
DSW-A2(config-if)# standby version 2
DSW-A2(config-if)# standby 4 ip 10.6.0.1
DSW-A2(config-if)# standby 4 priority 105
DSW-A2(config-if)# standby 4 preempt
DSW-A2(config-if)# exit
```

**DSW-A1 (Standby):**
```cisco
DSW-A1(config)# interface vlan 40
DSW-A1(config-if)# ip address 10.6.0.2 255.255.255.0
DSW-A1(config-if)# standby version 2
DSW-A1(config-if)# standby 4 ip 10.6.0.1
DSW-A1(config-if)# exit
```

**Сохранение:**
```cisco
DSW-A1(config)# do write
DSW-A2(config)# do write
```

---

### ШАГ 16-19: Office B HSRP Configuration

**DSW-B1 - VLAN 99 (Group 1, Active):**
```cisco
DSW-B1(config)# interface vlan 99
DSW-B1(config-if)# ip address 10.0.0.18 255.255.255.240
DSW-B1(config-if)# standby version 2
DSW-B1(config-if)# standby 1 ip 10.0.0.17
DSW-B1(config-if)# standby 1 priority 105
DSW-B1(config-if)# standby 1 preempt
DSW-B1(config-if)# exit
```

**DSW-B2 - VLAN 99 (Group 1, Standby):**
```cisco
DSW-B2(config)# interface vlan 99
DSW-B2(config-if)# ip address 10.0.0.19 255.255.255.240
DSW-B2(config-if)# standby version 2
DSW-B2(config-if)# standby 1 ip 10.0.0.17
DSW-B2(config-if)# exit
```

---

**DSW-B1 - VLAN 10 (Group 2, Active):**
```cisco
DSW-B1(config)# interface vlan 10
DSW-B1(config-if)# ip address 10.3.0.2 255.255.255.0
DSW-B1(config-if)# standby version 2
DSW-B1(config-if)# standby 2 ip 10.3.0.1
DSW-B1(config-if)# standby 2 priority 105
DSW-B1(config-if)# standby 2 preempt
DSW-B1(config-if)# exit
```

**DSW-B2 - VLAN 10 (Group 2, Standby):**
```cisco
DSW-B2(config)# interface vlan 10
DSW-B2(config-if)# ip address 10.3.0.3 255.255.255.0
DSW-B2(config-if)# standby version 2
DSW-B2(config-if)# standby 2 ip 10.3.0.1
DSW-B2(config-if)# exit
```

---

**DSW-B2 - VLAN 20 (Group 3, Active):**
```cisco
DSW-B2(config)# interface vlan 20
DSW-B2(config-if)# ip address 10.4.0.3 255.255.255.0
DSW-B2(config-if)# standby version 2
DSW-B2(config-if)# standby 3 ip 10.4.0.1
DSW-B2(config-if)# standby 3 priority 105
DSW-B2(config-if)# standby 3 preempt
DSW-B2(config-if)# exit
```

**DSW-B1 - VLAN 20 (Group 3, Standby):**
```cisco
DSW-B1(config)# interface vlan 20
DSW-B1(config-if)# ip address 10.4.0.2 255.255.255.0
DSW-B1(config-if)# standby version 2
DSW-B1(config-if)# standby 3 ip 10.4.0.1
DSW-B1(config-if)# exit
```

---

**DSW-B2 - VLAN 30 (Group 4, Active):**
```cisco
DSW-B2(config)# interface vlan 30
DSW-B2(config-if)# ip address 10.5.0.3 255.255.255.0
DSW-B2(config-if)# standby version 2
DSW-B2(config-if)# standby 4 ip 10.5.0.1
DSW-B2(config-if)# standby 4 priority 105
DSW-B2(config-if)# standby 4 preempt
DSW-B2(config-if)# exit
```

**DSW-B1 - VLAN 30 (Group 4, Standby):**
```cisco
DSW-B1(config)# interface vlan 30
DSW-B1(config-if)# ip address 10.5.0.2 255.255.255.0
DSW-B1(config-if)# standby version 2
DSW-B1(config-if)# standby 4 ip 10.5.0.1
DSW-B1(config-if)# exit
```

**Сохранение:**
```cisco
DSW-B1(config)# do write
DSW-B2(config)# do write
```

---

## ✅ Проверка конфигурации

### 1. IP Connectivity
```cisco
R1# ping 10.0.0.34
CSW1# ping 10.0.0.42
DSW-A1# ping 10.0.0.46
```

### 2. HSRP Status
```cisco
DSW-A1# show standby brief
```

Ожидается:
```
Vlan10  2   P   10.1.0.2    Active  local
Vlan99  1   P   10.0.0.2    Active  local
Vlan20  3       10.2.0.2    Standby remote
Vlan40  4       10.6.0.2    Standby remote
```

### 3. HSRP Virtual IP
```cisco
DSW-A1# show standby vlan 10
```

### 4. Ping из Access Switch
```cisco
ASW-A1# ping 10.0.0.1
```

---

## 📊 HSRP Summary Tables

### Office A HSRP Groups

| Group | VLAN | VIP | Active | Standby |
|-------|------|-----|--------|---------|
| 1 | 99 (Mgmt) | .1 | DSW-A1 (.2) | DSW-A2 (.3) |
| 2 | 10 (PCs) | .1 | DSW-A1 (.2) | DSW-A2 (.3) |
| 3 | 20 (Phones) | .1 | DSW-A2 (.3) | DSW-A1 (.2) |
| 4 | 40 (Wi-Fi) | .1 | DSW-A2 (.3) | DSW-A1 (.2) |

### Office B HSRP Groups

| Group | VLAN | VIP | Active | Standby |
|-------|------|-----|--------|---------|
| 1 | 99 (Mgmt) | .17 | DSW-B1 (.18) | DSW-B2 (.19) |
| 2 | 10 (PCs) | .1 | DSW-B1 (.2) | DSW-B2 (.3) |
| 3 | 20 (Phones) | .1 | DSW-B2 (.3) | DSW-B1 (.2) |
| 4 | 30 (Servers) | .1 | DSW-B2 (.3) | DSW-B1 (.2) |

---

## 💡 Практические советы

### HSRP Troubleshooting:
```cisco
show standby brief           # Краткая информация
show standby                 # Полная информация
show standby vlan [id]       # Конкретный VLAN
debug standby                # Отладка (осторожно!)
```

### Типичные ошибки:
❌ Забыть `standby version 2` - по умолчанию v1  
❌ Неправильный priority на Active  
❌ Забыть `preempt` на Active роутере  
❌ Неправильный group number  
❌ Несовпадение VIP между устройствами  

---

## 🎓 Ключевые команды Части 3

```cisco
# IP Addressing
interface [type] [number]
 ip address [ip] [mask]
 no shutdown

# Layer-3 Interface
no switchport

# IP Routing
ip routing

# HSRP
standby version 2
standby [group] ip [vip]
standby [group] priority [value]
standby [group] preempt

# Verification
show ip interface brief
show interfaces [type] [number]
show standby brief
show standby vlan [id]
ping [ip]
```

---

## 🎯 Чек-лист завершения Части 3

- [ ] R1 интерфейсы настроены (DHCP + статические)
- [ ] IP routing включен на всех L3 коммутаторах
- [ ] Layer-3 EtherChannel работает (CSW1 ↔ CSW2)
- [ ] IP адреса на Core switches настроены
- [ ] IP адреса на Distribution switches настроены
- [ ] SRV1 IP настроен через GUI
- [ ] Management IP на Access switches настроены
- [ ] HSRP Group 1, 2 в Office A работает
- [ ] HSRP Group 3, 4 в Office A работает
- [ ] HSRP Group 1, 2 в Office B работает
- [ ] HSRP Group 3, 4 в Office B работает
- [ ] Все конфигурации сохранены (`do write`)
- [ ] Ping тесты успешны

---

## 🧪 Тестирование HSRP

### Тест 1: Проверка Active/Standby
```cisco
DSW-A1# show standby brief
DSW-A2# show standby brief
```

### Тест 2: Failover тест
```cisco
! На Active роутере отключить интерфейс
DSW-A1(config)# interface vlan 10
DSW-A1(config-if)# shutdown

! На Standby должен смениться статус
DSW-A2# show standby brief
! Теперь DSW-A2 должен быть Active

! Включить обратно
DSW-A1(config-if)# no shutdown
! Благодаря preempt, DSW-A1 вернёт роль Active
```

### Тест 3: Ping VIP
```cisco
ASW-A1# ping 10.0.0.1
ASW-A2# ping 10.1.0.1
```

---

## 📊 IP Addressing Reference

### Loopback Addresses
| Device | Loopback IP |
|--------|-------------|
| R1 | 10.0.0.76/32 |
| CSW1 | 10.0.0.77/32 |
| CSW2 | 10.0.0.78/32 |
| DSW-A1 | 10.0.0.79/32 |
| DSW-A2 | 10.0.0.80/32 |
| DSW-B1 | 10.0.0.81/32 |
| DSW-B2 | 10.0.0.82/32 |

### Point-to-Point Links (/30)
| Link | Subnet | Device 1 | Device 2 |
|------|--------|----------|----------|
| R1 ↔ CSW1 | 10.0.0.32/30 | .33 | .34 |
| R1 ↔ CSW2 | 10.0.0.36/30 | .37 | .38 |
| CSW1 ↔ CSW2 | 10.0.0.40/30 | .41 | .42 |
| CSW1 ↔ DSW-A1 | 10.0.0.44/30 | .45 | .46 |
| CSW1 ↔ DSW-A2 | 10.0.0.48/30 | .49 | .50 |
| CSW1 ↔ DSW-B1 | 10.0.0.52/30 | .53 | .54 |
| CSW1 ↔ DSW-B2 | 10.0.0.56/30 | .57 | .58 |
| CSW2 ↔ DSW-A1 | 10.0.0.60/30 | .61 | .62 |
| CSW2 ↔ DSW-A2 | 10.0.0.64/30 | .65 | .66 |
| CSW2 ↔ DSW-B1 | 10.0.0.68/30 | .69 | .70 |
| CSW2 ↔ DSW-B2 | 10.0.0.72/30 | .73 | .74 |

---

## 🚀 Готовы к Части 4?

В следующей части мы настроим:
- **Rapid PVST+** - быстрый Spanning Tree
- **Root Bridge** - определение корневого моста
- **STP Priority** - настройка приоритетов
- **PortFast** - быстрое включение портов
- **BPDU Guard** - защита от петель

**До встречи в Части 4! 🎓**