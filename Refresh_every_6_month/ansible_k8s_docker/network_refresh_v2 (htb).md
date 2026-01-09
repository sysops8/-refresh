# Сети для DevOps/SysAdmin: Практический мини-курс

**Цель:** Освежить ключевые концепции сетей за 2-3 часа практики и узнать 1-2 продвинутые техники для работы с инфраструктурой.

**Формат:** Каждый раздел состоит из:

1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная задача, которую нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Предварительные требования:**

- Доступ к Linux машине (физической или виртуальной)
- Базовые знания командной строки
- Права root/sudo

---

## Модуль 1: Основы TCP/IP и модель OSI (20 минут)

### 🎯 Напоминалка

**Модель OSI (7 уровней):**

```
7. Application    # HTTP, DNS, SSH, FTP
8. Presentation   # SSL/TLS, шифрование
9. Session        # Управление сессиями
10. Transport      # TCP, UDP
11. Network        # IP, ICMP, маршрутизация
12. Data Link      # Ethernet, MAC адреса, коммутация
13. Physical       # Кабели, сигналы
```

**TCP/IP модель (4 уровня):**

```
4. Application    # HTTP, DNS, SSH (OSI 5-7)
5. Transport      # TCP, UDP
6. Internet       # IP, ICMP, ARP
7. Network Access # Ethernet, WiFi (OSI 1-2)
```

**IP адресация:**

bash

```bash
# IPv4 структура
192.168.1.100/24
│   │   │ │   └─ Хост
│   │   │ └───── Подсеть (256 адресов)
│   │   └─────── Частная сеть класса C
│   └─────────── 
└─────────────── 

# Частные диапазоны (RFC 1918)
10.0.0.0/8        # 10.0.0.0 - 10.255.255.255
172.16.0.0/12     # 172.16.0.0 - 172.31.255.255
192.168.0.0/16    # 192.168.0.0 - 192.168.255.255

# Специальные адреса
127.0.0.1         # Loopback (localhost)
0.0.0.0           # Все интерфейсы / неопределенный адрес
255.255.255.255   # Broadcast

# CIDR нотация
/24 = 255.255.255.0     # 256 адресов (254 хоста)
/16 = 255.255.0.0       # 65536 адресов
/8  = 255.0.0.0         # 16777216 адресов
```

**Подсчет адресов в подсети:**

bash

```bash
# Формула: 2^(32-prefix) = всего адресов
/24: 2^8  = 256 адресов  (254 хоста, -2 для сети и broadcast)
/25: 2^7  = 128 адресов  (126 хостов)
/26: 2^6  = 64 адреса    (62 хоста)
/27: 2^5  = 32 адреса    (30 хостов)
/28: 2^4  = 16 адресов   (14 хостов)
/29: 2^3  = 8 адресов    (6 хостов)
/30: 2^2  = 4 адреса     (2 хоста, для point-to-point)
/32: 2^0  = 1 адрес      (единичный хост)
```

**Основные протоколы:**

bash

```bash
# TCP - надежная доставка с установкой соединения
# Характеристики:
- Three-way handshake (SYN, SYN-ACK, ACK)
- Гарантированная доставка
- Упорядоченность пакетов
- Контроль перегрузки
- Примеры: HTTP, SSH, FTP

# UDP - быстрая доставка без гарантий
# Характеристики:
- Без установки соединения
- Нет гарантии доставки
- Нет упорядочивания
- Меньше overhead
- Примеры: DNS, DHCP, VoIP, видео-стриминг

# ICMP - служебные сообщения
- Echo Request/Reply (ping)
- Destination Unreachable
- Time Exceeded (traceroute)
- Redirect
```

**Порты:**

bash

```bash
# Well-known ports (0-1023)
20/21   # FTP (data/control)
22      # SSH
23      # Telnet
25      # SMTP
53      # DNS
80      # HTTP
110     # POP3
143     # IMAP
443     # HTTPS
3306    # MySQL
5432    # PostgreSQL
6379    # Redis
27017   # MongoDB

# Registered ports (1024-49151)
3000    # Node.js dev
5000    # Flask dev
8080    # HTTP alternate
8443    # HTTPS alternate

# Dynamic/Private ports (49152-65535)
# Используются для временных соединений
```

**Основные команды:**

bash

```bash
# Просмотр IP конфигурации
ip addr show              # Современная команда
ifconfig                  # Устаревшая, но все еще используется
ip -4 addr                # Только IPv4
ip -6 addr                # Только IPv6

# Просмотр маршрутов
ip route show             # Таблица маршрутизации
route -n                  # Устаревшая команда
netstat -rn               # Альтернатива

# Проверка connectivity
ping 8.8.8.8              # ICMP echo
ping -c 4 google.com      # 4 пакета
traceroute google.com     # Путь до хоста
mtr google.com            # Интерактивный traceroute

# DNS
dig google.com            # Подробный DNS запрос
nslookup google.com       # Простой DNS запрос
host google.com           # Краткий DNS запрос

# Сетевые соединения
netstat -tunlp            # Все listening порты
ss -tunlp                 # Современная альтернатива netstat
lsof -i :80               # Что слушает порт 80
```

### 💻 Задание

Базовая диагностика сети:

1. Узнай свой IP адрес и сетевой интерфейс:

bash

```bash
   ip addr show
   # или
   hostname -I
```

2. Проверь таблицу маршрутизации:

bash

```bash
   ip route show
   # Найди default gateway
```

3. Проверь connectivity до внешних ресурсов:

bash

```bash
   ping -c 4 8.8.8.8          # Google DNS
   ping -c 4 1.1.1.1          # Cloudflare DNS
   ping -c 4 google.com       # Проверка DNS resolution
```

4. Проверь путь до удаленного хоста:

bash

```bash
   traceroute google.com
   # или
   mtr --report-cycles 10 google.com
```

5. Проверь DNS резолвинг:

bash

```bash
   dig google.com
   dig @8.8.8.8 google.com    # Через конкретный DNS
   dig google.com +short      # Краткий вывод
```

6. Посмотри открытые порты на своей системе:

bash

```bash
   ss -tunlp
   # или
   netstat -tunlp
```

7. Рассчитай количество хостов в подсети:

bash

```bash
   # Для 192.168.1.0/24
   # Первый адрес: 192.168.1.0 (сеть)
   # Последний адрес: 192.168.1.255 (broadcast)
   # Доступные хосты: 192.168.1.1 - 192.168.1.254 (254 хоста)
   
   # Используй калькулятор:
   ipcalc 192.168.1.0/24
```

### 🚀 Бонус (новое)

**1. Используй современные инструменты для диагностики:**

bash

```bash
# Установи полезные утилиты
sudo apt install -y iproute2 dnsutils tcpdump nmap mtr iputils-ping

# ip вместо ifconfig
ip addr                    # Вместо ifconfig
ip link                    # Состояние интерфейсов
ip -s link                 # Статистика интерфейсов
ip route                   # Вместо route

# ss вместо netstat
ss -tunap                  # Все TCP и UDP соединения
ss -tlnp                   # Только listening TCP
ss -4 state established    # Только established IPv4
ss -o state established    # С таймерами

# Посмотри статистику сетевых интерфейсов
ip -s -s link show eth0
```

**2. Проверь MTU и Path MTU Discovery:**

bash

```bash
# Узнай текущий MTU
ip link show eth0 | grep mtu

# Проверь Path MTU до удаленного хоста
ping -M do -s 1472 -c 1 google.com
# -M do: Don't fragment
# -s 1472: 1500 (MTU) - 28 (IP+ICMP headers)

# Если пакет не проходит, уменьши размер
ping -M do -s 1400 -c 1 google.com
```

**3. Используй современные DNS утилиты:**

bash

```bash
# dog - современная альтернатива dig
# Установка
curl -OL https://github.com/ogham/dog/releases/latest/download/dog-v0.1.0-x86_64-unknown-linux-gnu.zip
unzip dog-*.zip
sudo mv bin/dog /usr/local/bin/

# Использование
dog google.com
dog google.com MX          # Mail records
dog google.com TXT         # TXT records
dog google.com @8.8.8.8    # Через конкретный DNS
```

---

## Модуль 2: Сетевые интерфейсы и настройка (25 минут)

### 🎯 Напоминалка

**Типы сетевых интерфейсов:**

bash

```bash
# Физические интерфейсы
eth0, eth1        # Ethernet
wlan0, wlan1      # WiFi
ens33, enp0s3     # Predictable names (systemd)

# Виртуальные интерфейсы
lo                # Loopback (127.0.0.1)
tun0, tap0        # VPN туннели
br0               # Bridge
veth0             # Virtual Ethernet (для контейнеров)
dummy0            # Dummy interface

# Именование (Predictable Network Interface Names)
en - Ethernet
wl - WLAN (WiFi)
ww - WWAN (Mobile broadband)

o<index>          # On-board device
s<slot>           # Hot-plug slot
p<port>s<slot>    # PCI geographical location
```

**NetworkManager vs systemd-networkd:**

bash

```bash
# NetworkManager (Desktop, динамические сети)
nmcli             # Command-line утилита
nmtui             # Text UI
nm-connection-editor  # GUI

# systemd-networkd (Server, статические настройки)
networkctl        # Управление
/etc/systemd/network/*.network  # Конфигурация
```

**Настройка с помощью NetworkManager:**

bash

```bash
# Просмотр соединений
nmcli connection show
nmcli device status

# Создание статического IP
nmcli connection add \
    type ethernet \
    con-name static-eth0 \
    ifname eth0 \
    ipv4.addresses 192.168.1.100/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.dns "8.8.8.8 1.1.1.1" \
    ipv4.method manual

# DHCP конфигурация
nmcli connection add \
    type ethernet \
    con-name dhcp-eth0 \
    ifname eth0 \
    ipv4.method auto

# Активация/деактивация
nmcli connection up static-eth0
nmcli connection down static-eth0

# Изменение существующего
nmcli connection modify static-eth0 ipv4.addresses 192.168.1.101/24
nmcli connection modify static-eth0 ipv4.dns "1.1.1.1 8.8.8.8"

# Перезагрузка соединения
nmcli connection reload
nmcli connection up static-eth0
```

**Netplan (Ubuntu/Debian новые версии):**

yaml

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd  # или NetworkManager
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
      routes:
        - to: 10.0.0.0/8
          via: 192.168.1.254

# Применить изменения
sudo netplan generate
sudo netplan apply

# Отладка
sudo netplan --debug apply
```

**Legacy конфигурация (Debian/Ubuntu старые версии):**

bash

```bash
# /etc/network/interfaces
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 1.1.1.1

# Перезапуск
sudo systemctl restart networking
# или
sudo ifdown eth0 && sudo ifup eth0
```

**RHEL/CentOS конфигурация:**

bash

```bash
# /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
BOOTPROTO=static
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=1.1.1.1

# Перезапуск
sudo systemctl restart NetworkManager
# или
sudo nmcli connection reload
sudo nmcli connection up eth0
```

**Временные изменения (не сохраняются после перезагрузки):**

bash

```bash
# Добавить IP адрес
sudo ip addr add 192.168.1.100/24 dev eth0

# Удалить IP адрес
sudo ip addr del 192.168.1.100/24 dev eth0

# Включить/выключить интерфейс
sudo ip link set eth0 up
sudo ip link set eth0 down

# Добавить маршрут
sudo ip route add 10.0.0.0/8 via 192.168.1.254
sudo ip route add default via 192.168.1.1

# Удалить маршрут
sudo ip route del 10.0.0.0/8
sudo ip route del default

# Изменить MTU
sudo ip link set eth0 mtu 9000
```

**Мониторинг интерфейсов:**

bash

```bash
# Статистика
ip -s link show eth0
ip -s -s link show eth0    # Подробнее

# Ошибки
ethtool -S eth0

# Настройки интерфейса
ethtool eth0

# Скорость и дуплекс
ethtool eth0 | grep Speed
ethtool eth0 | grep Duplex

# Изменить скорость (требуется ethtool)
sudo ethtool -s eth0 speed 1000 duplex full autoneg on
```

**Bonding/Teaming (агрегация каналов):**

bash

```bash
# Bonding modes
mode=0  # Round-robin (балансировка)
mode=1  # Active-backup (отказоустойчивость)
mode=2  # XOR (балансировка по MAC)
mode=3  # Broadcast
mode=4  # 802.3ad (LACP)
mode=5  # Adaptive transmit load balancing
mode=6  # Adaptive load balancing

# Пример с NetworkManager
nmcli connection add type bond \
    con-name bond0 \
    ifname bond0 \
    mode active-backup

nmcli connection add type ethernet \
    slave-type bond \
    con-name bond0-port1 \
    ifname eth0 \
    master bond0

nmcli connection add type ethernet \
    slave-type bond \
    con-name bond0-port2 \
    ifname eth1 \
    master bond0
```

### 💻 Задание

Настрой сетевые интерфейсы:

1. **Посмотри текущую конфигурацию:**

bash

```bash
   # Все интерфейсы
   ip addr show
   
   # Статус
   nmcli device status
   
   # Активные соединения
   nmcli connection show --active
```

2. **Создай тестовый dummy интерфейс:**

bash

```bash
   # Загрузи модуль
   sudo modprobe dummy
   
   # Создай интерфейс
   sudo ip link add dummy0 type dummy
   
   # Назначь IP
   sudo ip addr add 10.0.0.1/24 dev dummy0
   
   # Включи интерфейс
   sudo ip link set dummy0 up
   
   # Проверь
   ip addr show dummy0
   ping -c 2 10.0.0.1
```

3. **Настрой дополнительный IP на существующем интерфейсе (IP aliasing):**

bash

```bash
   # Узнай имя основного интерфейса
   ip route | grep default
   
   # Добавь дополнительный IP
   sudo ip addr add 192.168.100.100/24 dev eth0
   
   # Проверь
   ip addr show eth0
   
   # Удали после теста
   sudo ip addr del 192.168.100.100/24 dev eth0
```

4. **Настрой статический маршрут:**

bash

```bash
   # Добавь маршрут к тестовой сети
   sudo ip route add 172.16.0.0/16 via 192.168.1.1
   
   # Проверь
   ip route show
   
   # Удали после теста
   sudo ip route del 172.16.0.0/16
```

5. **Проверь настройки DNS:**

bash

```bash
   # Посмотри текущие DNS серверы
   cat /etc/resolv.conf
   
   # Или через systemd-resolved
   resolvectl status
   
   # Протестируй разрешение имен
   dig google.com
   nslookup google.com
```

6. **Создай постоянную конфигурацию с NetworkManager:**

bash

```bash
   # Создай новое соединение
   sudo nmcli connection add \
       type ethernet \
       con-name test-static \
       ifname eth0 \
       ipv4.addresses 192.168.1.200/24 \
       ipv4.gateway 192.168.1.1 \
       ipv4.dns "8.8.8.8 1.1.1.1" \
       ipv4.method manual \
       autoconnect no
   
   # Посмотри конфигурацию
   nmcli connection show test-static
   
   # НЕ активируй (чтобы не потерять связь)
   # После теста удали
   sudo nmcli connection delete test-static
```

### 🚀 Бонус (новое)

**1. Используй VLAN (802.1Q):**

bash

```bash
# Загрузи модуль 8021q
sudo modprobe 8021q

# Создай VLAN интерфейс (VLAN ID 100)
sudo ip link add link eth0 name eth0.100 type vlan id 100

# Назначь IP
sudo ip addr add 192.168.100.1/24 dev eth0.100

# Включи интерфейс
sudo ip link set eth0.100 up

# Проверь
ip addr show eth0.100

# С помощью NetworkManager (постоянная конфигурация)
nmcli connection add \
    type vlan \
    con-name vlan100 \
    dev eth0 \
    id 100 \
    ipv4.addresses 192.168.100.1/24 \
    ipv4.method manual
```

**2. Создай bridge для виртуальных машин:**

bash

```bash
# Создай bridge
sudo ip link add name br0 type bridge

# Добавь физический интерфейс в bridge
sudo ip link set eth0 master br0

# Настрой IP на bridge
sudo ip addr add 192.168.1.100/24 dev br0

# Включи интерфейсы
sudo ip link set br0 up
sudo ip link set eth0 up

# Проверь
ip addr show br0
bridge link show

# С помощью NetworkManager (постоянная конфигурация)
nmcli connection add \
    type bridge \
    con-name br0 \
    ifname br0 \
    ipv4.addresses 192.168.1.100/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.method manual

nmcli connection add \
    type ethernet \
    slave-type bridge \
    con-name br0-port1 \
    ifname eth0 \
    master br0
```

**3. Настрой Network Namespaces:**

bash

```bash
# Создай namespace
sudo ip netns add testns

# Посмотри список namespaces
ip netns list

# Выполни команду в namespace
sudo ip netns exec testns ip addr

# Создай veth pair
sudo ip link add veth0 type veth peer name veth1

# Перемести один конец в namespace
sudo ip link set veth1 netns testns

# Настрой интерфейсы
sudo ip addr add 10.0.0.1/24 dev veth0
sudo ip link set veth0 up

sudo ip netns exec testns ip addr add 10.0.0.2/24 dev veth1
sudo ip netns exec testns ip link set veth1 up
sudo ip netns exec testns ip link set lo up

# Проверь connectivity
ping -c 2 10.0.0.2
sudo ip netns exec testns ping -c 2 10.0.0.1

# Удали после теста
sudo ip netns del testns
sudo ip link del veth0
```

---

## Модуль 3: Firewall и iptables/nftables (30 минут)

### 🎯 Напоминалка

**Firewall концепции:**

bash

```bash
# Stateless firewall
- Фильтрует пакеты по отдельности
- Не отслеживает соединения
- Быстрее, но менее безопасно

# Stateful firewall
- Отслеживает состояние соединений
- Понимает контекст (NEW, ESTABLISHED, RELATED)
- Более безопасно

# Цепочки (chains) и таблицы (tables)
PREROUTING  → FORWARD → POSTROUTING
              ↓
            INPUT → (local process) → OUTPUT
```

**iptables основы:**

bash

```bash
# Таблицы
filter    # По умолчанию, фильтрация пакетов
nat       # Network Address Translation
mangle    # Изменение пакетов
raw       # Настройка исключений connection tracking

# Цепочки в filter таблице
INPUT     # Входящие пакеты для локальной системы
OUTPUT    # Исходящие пакеты от локальной системы
FORWARD   # Пакеты, проходящие через систему (маршрутизация)

# Цепочки в nat таблице
PREROUTING   # Изменение пакетов при входе
POSTROUTING  # Изменение пакетов при выходе
OUTPUT       # NAT для локально сгенерированных пакетов

# Действия (targets)
ACCEPT    # Разрешить пакет
DROP      # Отбросить пакет молча
REJECT    # Отбросить с уведомлением
LOG       # Залогировать пакет
MASQUERADE # Динамический SNAT (для NAT)
DNAT      # Destination NAT (port forwarding)
SNAT      # Source NAT
```

**Базовые команды iptables:**

bash

```bash
# Просмотр правил
sudo iptables -L                    # Список правил
sudo iptables -L -v                 # С подробностями
sudo iptables -L -n                 # Без DNS resolution
sudo iptables -L -v -n --line-numbers  # С номерами строк
sudo iptables -t nat -L             # NAT таблица

# Очистка правил
sudo iptables -F                    # Очистить все правила
sudo iptables -F INPUT              # Очистить цепочку INPUT
sudo iptables -X                    # Удалить пользовательские цепочки
sudo iptables -t nat -F             # Очистить NAT таблицу

# Политики по умолчанию
sudo iptables -P INPUT DROP         # По умолчанию блокировать входящие
sudo iptables -P OUTPUT ACCEPT      # Разрешить исходящие
sudo iptables -P FORWARD DROP       # Блокировать пересылку

# Добавление правил
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT     # Разрешить SSH
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT     # Разрешить HTTP
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT    # Разрешить HTTPS
sudo iptables -A INPUT -j DROP                         # Блокировать все остальное

# Вставка правила на конкретную позицию
sudo iptables -I INPUT 1 -p tcp --dport 8080 -j ACCEPT

# Удаление правила
sudo iptables -D INPUT -p tcp --dport 8080 -j ACCEPT
sudo iptables -D INPUT 1                # По номеру строки

# Сохранение/восстановление правил
sudo iptables-save > /tmp/iptables.rules
sudo iptables-restore < /tmp/iptables.rules

# Для постоянного сохранения (Debian/Ubuntu)
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

**Типичные правила iptables:**

bash

```bash
# Разрешить loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Разрешить established и related соединения
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Разрешить SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Разрешить HTTP/HTTPS
sudo iptables -A INPUT -p tcp -m multiport --dports 80,443 -j ACCEPT

# Разрешить ping (ICMP)
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Блокировать конкретный IP
sudo iptables -A INPUT -s 192.168.1.100 -j DROP

# Разрешить порт только с определенного IP
sudo iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 3306 -j ACCEPT

# Rate limiting (защита от DDoS)
sudo iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT

# Логирование
sudo iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7
```

**NAT с iptables:**

bash

```bash
# Включить IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

# MASQUERADE (для динамического IP на WAN)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# SNAT (для статического IP на WAN)
sudo iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 203.0.113.10

# Port forwarding (DNAT)
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80

# Разрешить forwarding для NAT
sudo iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
```

**ufw (Uncomplicated Firewall) - упрощенный интерфейс:**

bash

```bash
# Установка и включение
sudo apt install ufw
sudo ufw enable

# Базовые команды
sudo ufw status                     # Статус
sudo ufw status verbose             # Подробный статус
sudo ufw disable                    # Отключить

# Правила по умолчанию
sudo ufw default deny incoming      # Блокировать входящие
sudo ufw default allow outgoing     # Разрешить исходящие

# Добавление правил
sudo ufw allow 22                   # Разрешить SSH
sudo ufw allow 80/tcp               # Разрешить HTTP TCP 
sudo ufw allow 443/tcp              # Разрешить HTTPS 
sudo ufw allow from 192.168.1.0/24 # Разрешить с подсети 
sudo ufw allow from 192.168.1.100 to any port 3306 # Разрешить MySQL с IP

# Удаление правил

sudo ufw delete allow 80 sudo ufw status numbered # Посмотреть с номерами sudo ufw delete 2 # Удалить по номеру

# Сброс всех правил

sudo ufw reset

````

**firewalld (RHEL/CentOS):**
```bash
# Установка и запуск
sudo yum install firewalld
sudo systemctl start firewalld
sudo systemctl enable firewalld

# Базовые команды
sudo firewall-cmd --state                    # Статус
sudo firewall-cmd --get-default-zone         # Зона по умолчанию
sudo firewall-cmd --get-active-zones         # Активные зоны
sudo firewall-cmd --list-all                 # Все правила

# Зоны
sudo firewall-cmd --get-zones                # Список зон
sudo firewall-cmd --zone=public --list-all   # Правила зоны

# Добавление сервисов
sudo firewall-cmd --add-service=http         # Временно
sudo firewall-cmd --add-service=http --permanent  # Постоянно
sudo firewall-cmd --reload                   # Применить постоянные правила

# Добавление портов
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --add-port=3000-3100/tcp --permanent

# Rich rules
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" accept' --permanent

# Port forwarding
sudo firewall-cmd --add-forward-port=port=8080:proto=tcp:toport=80 --permanent

# Masquerading (NAT)
sudo firewall-cmd --add-masquerade --permanent
```

**nftables (современная замена iptables):**
```bash
# Установка
sudo apt install nftables

# Просмотр правил
sudo nft list ruleset

# Базовая конфигурация
sudo nft add table inet filter
sudo nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
sudo nft add chain inet filter forward { type filter hook forward priority 0 \; policy drop \; }
sudo nft add chain inet filter output { type filter hook output priority 0 \; policy accept \; }

# Добавление правил
sudo nft add rule inet filter input ct state established,related accept
sudo nft add rule inet filter input iif lo accept
sudo nft add rule inet filter input tcp dport 22 accept
sudo nft add rule inet filter input tcp dport { 80, 443 } accept

# Сохранение правил
sudo nft list ruleset > /etc/nftables.conf
```

### 💻 Задание

Настрой базовый firewall:

1. **Проверь текущие правила firewall:**
```bash
   # iptables
   sudo iptables -L -v -n
   
   # ufw (если используется)
   sudo ufw status verbose
   
   # firewalld (если используется)
   sudo firewall-cmd --list-all
```

2. **Создай тестовые iptables правила (в безопасной среде!):**
```bash
   # ВНИМАНИЕ: Не делай это на production сервере!
   # Можешь потерять доступ!
   
   # Сохрани текущие правила
   sudo iptables-save > /tmp/iptables_backup.rules
   
   # Очисти правила (для теста)
   sudo iptables -F
   sudo iptables -X
   
   # Разрешить loopback
   sudo iptables -A INPUT -i lo -j ACCEPT
   sudo iptables -A OUTPUT -o lo -j ACCEPT
   
   # Разрешить established соединения
   sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
   
   # Разрешить SSH (ВАЖНО!)
   sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
   
   # Разрешить HTTP/HTTPS
   sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
   sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
   
   # Разрешить ping
   sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
   
   # Логировать остальное перед блокировкой
   sudo iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: "
   
   # Блокировать остальное
   sudo iptables -A INPUT -j DROP
   
   # Посмотри правила
   sudo iptables -L -v -n --line-numbers
   
   # Протестируй
   curl -I localhost
   ping -c 2 localhost
   
   # Восстанови оригинальные правила
   sudo iptables-restore < /tmp/iptables_backup.rules
```

3. **Настрой ufw (если доступен):**
```bash
   # Установи
   sudo apt install ufw -y
   
   # НЕ включай сразу! Сначала настрой правила
   
   # Сброс к defaults
   sudo ufw --force reset
   
   # Политики по умолчанию
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   
   # Разрешить SSH (ВАЖНО!)
   sudo ufw allow 22/tcp
   
   # Разрешить HTTP/HTTPS
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   
   # Посмотри правила (до включения)
   sudo ufw show added
   
   # Включи (осторожно!)
   sudo ufw enable
   
   # Проверь статус
   sudo ufw status verbose
   
   # После теста отключи
   sudo ufw disable
```

4. **Протестируй подключение к порту:**
```bash
   # Запусти простой HTTP сервер
   python3 -m http.server 8080 &
   
   # Проверь подключение
   curl http://localhost:8080
   
   # Заблокируй порт
   sudo iptables -I INPUT -p tcp --dport 8080 -j DROP
   
   # Попробуй подключиться (должно зависнуть)
   timeout 5 curl http://localhost:8080
   
   # Разблокируй порт
   sudo iptables -D INPUT -p tcp --dport 8080 -j DROP
   
   # Проверь снова
   curl http://localhost:8080
   
   # Останови сервер
   pkill -f "python3 -m http.server"
```

### 🚀 Бонус (новое)

**1. Настрой connection tracking и rate limiting:**
```bash
# Защита от SYN flood
sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
sudo iptables -A INPUT -p tcp --syn -j DROP

# Защита от port scanning
sudo iptables -N port-scanning
sudo iptables -A port-scanning -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s --limit-burst 2 -j RETURN
sudo iptables -A port-scanning -j DROP

# Блокировка invalid пакетов
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# Защита от ping flood
sudo iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

**2. Используй ipset для блокировки множества IP:**
```bash
# Установка
sudo apt install ipset

# Создай набор IP адресов
sudo ipset create blacklist hash:ip

# Добавь IP в blacklist
sudo ipset add blacklist 192.168.1.100
sudo ipset add blacklist 192.168.1.101
sudo ipset add blacklist 10.0.0.0/8

# Посмотри список
sudo ipset list blacklist

# Используй в iptables
sudo iptables -A INPUT -m set --match-set blacklist src -j DROP

# Сохрани ipset
sudo ipset save > /etc/ipset.conf

# Восстанови
sudo ipset restore < /etc/ipset.conf
```

**3. Настрой nftables (современная альтернатива):**
```bash
# Установка
sudo apt install nftables

# Создай файл конфигурации
sudo tee /etc/nftables.conf << 'EOF'
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        
        # Разрешить loopback
        iif lo accept
        
        # Разрешить established/related
        ct state established,related accept
        
        # Разрешить SSH
        tcp dport 22 accept
        
        # Разрешить HTTP/HTTPS
        tcp dport { 80, 443 } accept
        
        # Разрешить ping
        icmp type echo-request limit rate 1/second accept
        
        # Логировать и дропать остальное
        limit rate 5/minute log prefix "nftables dropped: "
        drop
    }
    
    chain forward {
        type filter hook forward priority 0; policy drop;
    }
    
    chain output {
        type filter hook output priority 0; policy accept;
    }
}
EOF

# Применить правила
sudo nft -f /etc/nftables.conf

# Посмотреть правила
sudo nft list ruleset

# Включить автозапуск
sudo systemctl enable nftables
sudo systemctl start nftables
```

---

## Модуль 4: DNS и службы имен (25 минут)

### 🎯 Напоминалка

**DNS иерархия:**
````

. (root) ├── com │ ├── google.com │ └── example.com ├── org │ └── wikipedia.org └── net └── cloudflare.net

# Типы записей DNS

A # IPv4 адрес AAAA # IPv6 адрес CNAME # Canonical name (алиас) MX # Mail exchange NS # Name server TXT # Text record PTR # Pointer (обратное разрешение) SOA # Start of authority SRV # Service record

````

**DNS разрешение:**
```bash
# Процесс разрешения имени
1. Проверка локального кэша
2. Проверка /etc/hosts
3. Запрос к DNS серверу из /etc/resolv.conf
4. Рекурсивный запрос (если разрешено)
   - Root servers → TLD servers → Authoritative servers

# Файл /etc/hosts
127.0.0.1       localhost
::1             localhost
192.168.1.100   server1.example.com server1

# Файл /etc/resolv.conf (традиционный)
nameserver 8.8.8.8
nameserver 1.1.1.1
search example.com
options timeout:2 attempts:3

# systemd-resolved (/etc/systemd/resolved.conf)
[Resolve]
DNS=8.8.8.8 1.1.1.1
FallbackDNS=8.8.4.4 1.0.0.1
Domains=example.com
DNSSEC=allow-downgrade
```

**DNS утилиты:**
```bash
# dig - основной инструмент диагностики DNS
dig example.com                    # A запись
dig example.com +short             # Краткий вывод
dig example.com MX                 # Mail exchange
dig example.com NS                 # Name servers
dig example.com TXT                # TXT записи
dig example.com ANY                # Все записи
dig @8.8.8.8 example.com           # Через конкретный DNS
dig -x 8.8.8.8                     # Обратное разрешение (PTR)
dig example.com +trace             # Трассировка DNS запроса

# Секции dig вывода
;; QUESTION SECTION:       # Что спросили
;; ANSWER SECTION:         # Ответ
;; AUTHORITY SECTION:      # Authoritative серверы
;; ADDITIONAL SECTION:     # Дополнительная информация

# nslookup - простой DNS запрос
nslookup example.com
nslookup example.com 8.8.8.8
nslookup -type=mx example.com
nslookup -type=ns example.com

# host - краткий DNS запрос
host example.com
host example.com 8.8.8.8
host -t MX example.com
host -t NS example.com
host -a example.com                # Все записи

# resolvectl (systemd-resolved)
resolvectl status                  # Статус DNS
resolvectl query example.com       # DNS запрос
resolvectl flush-caches            # Очистить кэш
resolvectl statistics              # Статистика
```

**Локальный DNS кэш:**
```bash
# systemd-resolved (встроенный)
systemctl status systemd-resolved
resolvectl status
resolvectl statistics

# dnsmasq (легковесный DNS/DHCP)
sudo apt install dnsmasq
sudo systemctl start dnsmasq

# Конфигурация /etc/dnsmasq.conf
server=8.8.8.8
server=1.1.1.1
cache-size=1000
no-resolv

# nscd (name service cache daemon)
sudo apt install nscd
sudo systemctl start nscd
sudo nscd -i hosts                 # Сбросить кэш
```

**Настройка локального DNS сервера (bind9):**
```bash
# Установка
sudo apt install bind9 bind9utils bind9-doc

# Основные файлы
/etc/bind/named.conf               # Главная конфигурация
/etc/bind/named.conf.options       # Опции сервера
/etc/bind/named.conf.local         # Локальные зоны
/var/cache/bind/                   # Файлы зон

# Конфигурация форвардера (/etc/bind/named.conf.options)
options {
    directory "/var/cache/bind";
    forwarders {
        8.8.8.8;
        1.1.1.1;
    };
    dnssec-validation auto;
    listen-on { 127.0.0.1; 192.168.1.10; };
    allow-query { localhost; 192.168.1.0/24; };
};

# Создание зоны (/etc/bind/named.conf.local)
zone "example.local" {
    type master;
    file "/etc/bind/db.example.local";
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.1";
};

# Проверка конфигурации
sudo named-checkconf
sudo named-checkzone example.local /etc/bind/db.example.local

# Перезапуск
sudo systemctl restart bind9
```

**Тестирование DNS:**
```bash
# Проверка разрешения имен
ping -c 1 example.com
curl -I http://example.com

# Измерение времени DNS запроса
time dig example.com +short

# Проверка всех DNS серверов из /etc/resolv.conf
cat /etc/resolv.conf | grep nameserver | while read _ ns; do
    echo "Testing $ns"
    dig @$ns example.com +short +time=2
done

# DNS через TCP (вместо UDP)
dig +tcp example.com

# DNS с DNSSEC валидацией
dig example.com +dnssec

# Массовый DNS lookup
echo -e "google.com\nexample.com\ngithub.com" | xargs -I {} dig {} +short
```

**mDNS (Multicast DNS) и .local домены:**
```bash
# Avahi (реализация mDNS для Linux)
sudo apt install avahi-daemon avahi-utils

# Просмотр локальных хостов
avahi-browse -a                    # Все сервисы
avahi-browse -t _http._tcp         # HTTP сервисы
avahi-resolve -n hostname.local    # Разрешить имя

# Настройка hostname для mDNS
sudo hostnamectl set-hostname myserver
# Теперь доступен как myserver.local
```

**DNS over HTTPS (DoH) и DNS over TLS (DoT):**
```bash
# Cloudflare DoH
curl -H 'accept: application/dns-json' \
  'https://cloudflare-dns.com/dns-query?name=example.com&type=A'

# Google DoH
curl -H 'accept: application/dns-json' \
  'https://dns.google/resolve?name=example.com&type=A'

# systemd-resolved с DoT
# /etc/systemd/resolved.conf
[Resolve]
DNS=1.1.1.1#cloudflare-dns.com
DNS=8.8.8.8#dns.google
DNSOverTLS=yes
```

### 💻 Задание

Настрой и протестируй DNS:

1. **Проверь текущую DNS конфигурацию:**
```bash
   # Посмотри DNS серверы
   cat /etc/resolv.conf
   
   # Если используется systemd-resolved
   resolvectl status
   
   # Проверь порядок разрешения имен
   cat /etc/nsswitch.conf | grep hosts
```

2. **Протестируй разные DNS серверы:**
```bash
   # Google DNS
   dig @8.8.8.8 google.com +short
   
   # Cloudflare DNS
   dig @1.1.1.1 google.com +short
   
   # Quad9 DNS
   dig @9.9.9.9 google.com +short
   
   # Локальный resolver
   dig @127.0.0.1 google.com +short
   
   # Измерь время ответа
   time dig @8.8.8.8 google.com +short
   time dig @1.1.1.1 google.com +short
```

3. **Исследуй DNS записи:**
```bash
   # A записи (IPv4)
   dig google.com A
   
   # AAAA записи (IPv6)
   dig google.com AAAA
   
   # MX записи (mail servers)
   dig google.com MX
   
   # NS записи (name servers)
   dig google.com NS
   
   # TXT записи (SPF, DKIM, etc)
   dig google.com TXT
   
   # Все записи
   dig google.com ANY
```

4. **Трассировка DNS запроса:**
```bash
   # Полная трассировка от root серверов
   dig google.com +trace
   
   # Краткая версия
   dig google.com +trace +short
```

5. **Настрой локальные DNS записи в /etc/hosts:**
```bash
   # Добавь тестовую запись
   echo "127.0.0.1 test.local" | sudo tee -a /etc/hosts
   
   # Проверь
   ping -c 2 test.local
   dig test.local
   
   # Удали после теста
   sudo sed -i '/test.local/d' /etc/hosts
```

6. **Измени DNS серверы (временно):**
```bash
   # Для systemd-resolved
   sudo mkdir -p /etc/systemd/resolved.conf.d/
   
   sudo tee /etc/systemd/resolved.conf.d/dns_servers.conf << EOF
[Resolve]
DNS=1.1.1.1 8.8.8.8
FallbackDNS=9.9.9.9
EOF
   
   # Перезапусти resolved
   sudo systemctl restart systemd-resolved
   
   # Проверь
   resolvectl status
   
   # Протестируй
   dig google.com
   
   # Удали после теста
   sudo rm /etc/systemd/resolved.conf.d/dns_servers.conf
   sudo systemctl restart systemd-resolved
```

### 🚀 Бонус (новое)

**1. Настрой локальный DNS кэш с dnsmasq:**
```bash
# Установка
sudo apt install dnsmasq

# Базовая конфигурация
sudo tee /etc/dnsmasq.d/local.conf << 'EOF'
# Upstream DNS серверы
server=1.1.1.1
server=8.8.8.8

# Размер кэша
cache-size=10000

# Локальные домены
address=/test.local/127.0.0.1
address=/dev.local/192.168.1.100

# Логирование
log-queries
log-facility=/var/log/dnsmasq.log

# Слушать только на localhost
listen-address=127.0.0.1
bind-interfaces
EOF

# Отключи systemd-resolved (конфликт портов)
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved

# Настрой resolv.conf
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf

# Запусти dnsmasq
sudo systemctl start dnsmasq
sudo systemctl enable dnsmasq

# Проверь
dig @127.0.0.1 google.com
tail -f /var/log/dnsmasq.log

# Тест кэша
time dig google.com  # Первый запрос (медленно)
time dig google.com  # Второй запрос (быстро, из кэша)

# Очистка кэша
sudo systemctl restart dnsmasq
```

**2. Используй split-DNS (разные DNS для разных доменов):**
```bash
# С dnsmasq
sudo tee -a /etc/dnsmasq.d/split-dns.conf << 'EOF'
# Внутренние домены через внутренний DNS
server=/internal.company.com/10.0.0.53
server=/corp.local/10.0.0.53

# Все остальное через публичные DNS
server=1.1.1.1
server=8.8.8.8
EOF

# С systemd-resolved
sudo tee /etc/systemd/resolved.conf.d/split-dns.conf << 'EOF'
[Resolve]
DNS=1.1.1.1 8.8.8.8
Domains=~internal.company.com ~corp.local
EOF

sudo systemctl restart systemd-resolved
```

**3. Настрой DNS over TLS с stubby:**
```bash
# Установка
sudo apt install stubby

# Конфигурация /etc/stubby/stubby.yml
sudo tee /etc/stubby/stubby.yml << 'EOF'
resolution_type: GETDNS_RESOLUTION_STUB
dns_transport_list:
  - GETDNS_TRANSPORT_TLS
tls_authentication: GETDNS_AUTHENTICATION_REQUIRED
tls_query_padding_blocksize: 128
idle_timeout: 10000
listen_addresses:
  - 127.0.0.1@53000
  - 0::1@53000

upstream_recursive_servers:
  # Cloudflare
  - address_data: 1.1.1.1
    tls_auth_name: "cloudflare-dns.com"
  - address_data: 1.0.0.1
    tls_auth_name: "cloudflare-dns.com"
  # Quad9
  - address_data: 9.9.9.9
    tls_auth_name: "dns.quad9.net"
EOF

# Запуск
sudo systemctl start stubby
sudo systemctl enable stubby

# Настрой систему использовать stubby
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf

# Проверка (трафик должен быть зашифрован)
dig google.com
```

**4. Используй dog - современную альтернативу dig:**
```bash
# Установка
wget https://github.com/ogham/dog/releases/download/v0.1.0/dog-v0.1.0-x86_64-unknown-linux-gnu.zip
unzip dog-*.zip
sudo mv bin/dog /usr/local/bin/

# Использование
dog google.com                     # Простой запрос
dog google.com A                   # A записи
dog google.com MX                  # MX записи
dog google.com @1.1.1.1            # Через конкретный DNS
dog google.com --time              # С временем ответа
dog google.com --json              # JSON вывод
```

---

## Модуль 5: NAT и маршрутизация (30 минут)

### 🎯 Напоминалка

**Типы NAT:**
```bash
# SNAT (Source NAT)
- Изменяет source IP адрес исходящих пакетов
- Используется для выхода в интернет с приватных IP
- Пример: 192.168.1.10 → 203.0.113.50

# DNAT (Destination NAT)
- Изменяет destination IP адрес входящих пакетов
- Используется для port forwarding
- Пример: 203.0.113.50:8080 → 192.168.1.10:80

# MASQUERADE
- Динамический SNAT для интерфейсов с изменяющимся IP
- Используется для PPPoE, DHCP подключений
- Автоматически определяет source IP

# PAT (Port Address Translation)
- Вариант NAT с трансляцией портов
- Множество внутренних хостов → один внешний IP
- Каждое соединение получает уникальный порт
```

**IP forwarding:**
```bash
# Включить временно
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -w net.ipv6.conf.all.forwarding=1

# Проверка
sysctl net.ipv4.ip_forward
cat /proc/sys/net/ipv4/ip_forward

# Включить постоянно
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

**Базовая маршрутизация:**
```bash
# Просмотр таблицы маршрутизации
ip route show
ip route show table main
netstat -rn

# Вывод:
# default via 192.168.1.1 dev eth0
# 192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100

# Компоненты маршрута:
# Destination: default (0.0.0.0/0) или конкретная сеть
# via: Gateway (следующий хоп)
# dev: Интерфейс
# proto: Протокол (kernel, boot, static, dhcp)
# scope: link, host, global
# src: Source address для исходящих пакетов

# Добавление статических маршрутов
sudo ip route add 10.0.0.0/8 via 192.168.1.254
sudo ip route add 172.16.0.0/16 via 192.168.1.254 dev eth0
sudo ip route add default via 192.168.1.1

# Удаление маршрутов
sudo ip route del 10.0.0.0/8
sudo ip route del default via 192.168.1.1

# Изменение маршрута
sudo ip route change 10.0.0.0/8 via 192.168.1.253

# Маршрут только для конкретного source IP
sudo ip route add 10.0.0.0/8 via 192.168.1.254 src 192.168.1.100

# Blackhole route (отбросить пакеты)
sudo ip route add blackhole 192.168.100.0/24

# Prohibit route (отбросить с ICMP ошибкой)
sudo ip route add prohibit 192.168.100.0/24
```

**Policy routing (множественные таблицы маршрутизации):**
```bash
# Просмотр таблиц
cat /etc/iproute2/rt_tables

# Создание новой таблицы
echo "100 isp1" | sudo tee -a /etc/iproute2/rt_tables
echo "101 isp2" | sudo tee -a /etc/iproute2/rt_tables

# Добавление маршрутов в таблицу
sudo ip route add default via 192.168.1.1 table isp1
sudo ip route add default via 192.168.2.1 table isp2

# Правила для выбора таблицы
sudo ip rule add from 192.168.1.0/24 table isp1
sudo ip rule add from 192.168.2.0/24 table isp2

# Просмотр правил
ip rule show

# Просмотр конкретной таблицы
ip route show table isp1
```

**NAT с iptables:**
```bash
# Включить IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# MASQUERADE (для DHCP/PPPoE)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# SNAT (для статического IP)
sudo iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 203.0.113.50

# SNAT для конкретной подсети
sudo iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE

# Port forwarding (DNAT)
# Внешний порт 8080 → внутренний 192.168.1.100:80 
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80

# Port forwarding с изменением порта
sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to-destination 192.168.1.100:8443

# Диапазон портов
sudo iptables -t nat -A PREROUTING -p tcp --dport 8000:9000 -j DNAT --to-destination 192.168.1.100

# Разрешить forwarding для NAT
sudo iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT 
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT

````

**1:1 NAT (Bidirectional NAT):**
```bash
# Внешний IP 203.0.113.50 ↔ Внутренний 192.168.1.100
sudo iptables -t nat -A PREROUTING -d 203.0.113.50 -j DNAT --to-destination 192.168.1.100
sudo iptables -t nat -A POSTROUTING -s 192.168.1.100 -j SNAT --to-source 203.0.113.50
```

**Load balancing:**
```bash
# Random load balancing между двумя серверами
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -m statistic --mode random --probability 0.5 -j DNAT --to-destination 192.168.1.10
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.11

# Round-robin load balancing
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.1.10
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.11
```

**Transparent proxy (redirect):**
```bash
# Перенаправить HTTP трафик на локальный прокси
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 3128
sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-ports 3129
```

### 💻 Задание

Настрой базовую маршрутизацию и NAT:

1. **Проверь текущую таблицу маршрутизации:**
```bash
   # Покажи все маршруты
   ip route show
   
   # Default gateway
   ip route show default
   
   # Маршруты для конкретной сети
   ip route get 8.8.8.8
   ip route get 192.168.1.1
   
   # Статистика маршрутизации
   ip -s route
```

2. **Добавь статические маршруты (тестовые):**
```bash
   # Создай dummy интерфейс для теста
   sudo ip link add dummy0 type dummy
   sudo ip addr add 10.0.0.1/24 dev dummy0
   sudo ip link set dummy0 up
   
   # Добавь маршрут к другой подсети через dummy
   sudo ip route add 10.0.1.0/24 via 10.0.0.1 dev dummy0
   
   # Проверь
   ip route show
   ip route get 10.0.1.10
   
   # Попробуй ping (не сработает, т.к. сеть не существует)
   ping -c 1 10.0.1.10
   
   # Удали после теста
   sudo ip route del 10.0.1.0/24
   sudo ip link del dummy0
```

3. **Протестируй traceroute:**
```bash
   # Трассировка до удаленного хоста
   traceroute google.com
   
   # С ICMP вместо UDP
   traceroute -I google.com
   
   # С TCP
   traceroute -T -p 80 google.com
   
   # MTR (интерактивный traceroute)
   mtr google.com
   mtr --report --report-cycles 10 google.com
```

4. **Проверь NAT таблицу:**
```bash
   # Посмотри NAT правила
   sudo iptables -t nat -L -v -n
   
   # Посмотри connection tracking
   sudo cat /proc/net/nf_conntrack | head -20
   
   # Или через conntrack утилиту
   sudo apt install conntrack
   sudo conntrack -L | head -20
```

5. **Симуляция простого NAT (в изолированной среде):**
```bash
   # Создай два network namespaces
   sudo ip netns add client
   sudo ip netns add server
   
   # Создай veth пары
   sudo ip link add veth-client type veth peer name veth-gw-client
   sudo ip link add veth-server type veth peer name veth-gw-server
   
   # Перемести концы в namespaces
   sudo ip link set veth-client netns client
   sudo ip link set veth-server netns server
   
   # Настрой IP адреса
   # Gateway стороны
   sudo ip addr add 192.168.1.1/24 dev veth-gw-client
   sudo ip addr add 10.0.0.1/24 dev veth-gw-server
   sudo ip link set veth-gw-client up
   sudo ip link set veth-gw-server up
   
   # Client namespace
   sudo ip netns exec client ip addr add 192.168.1.100/24 dev veth-client
   sudo ip netns exec client ip link set veth-client up
   sudo ip netns exec client ip link set lo up
   sudo ip netns exec client ip route add default via 192.168.1.1
   
   # Server namespace
   sudo ip netns exec server ip addr add 10.0.0.100/24 dev veth-server
   sudo ip netns exec server ip link set veth-server up
   sudo ip netns exec server ip link set lo up
   sudo ip netns exec server ip route add default via 10.0.0.1
   
   # Включи forwarding
   sudo sysctl -w net.ipv4.ip_forward=1
   
   # Настрой NAT
   sudo iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o veth-gw-server -j MASQUERADE
   
   # Тест: client пингует server
   sudo ip netns exec client ping -c 2 10.0.0.100
   
   # Проверь NAT
   sudo iptables -t nat -L -v -n
   sudo conntrack -L | grep 192.168.1.100
   
   # Cleanup
   sudo ip netns del client
   sudo ip netns del server
   sudo ip link del veth-gw-client
   sudo ip link del veth-gw-server
   sudo iptables -t nat -F
```

### 🚀 Бонус (новое)

**1. Настрой dual-WAN с failover:**
```bash
# Создай две таблицы маршрутизации
echo "100 isp1" | sudo tee -a /etc/iproute2/rt_tables
echo "101 isp2" | sudo tee -a /etc/iproute2/rt_tables

# Настрой маршруты для каждого ISP
sudo ip route add default via 192.168.1.1 dev eth0 table isp1
sudo ip route add default via 192.168.2.1 dev eth1 table isp2

# Правила для источников
sudo ip rule add from 192.168.1.0/24 table isp1 priority 100
sudo ip rule add from 192.168.2.0/24 table isp2 priority 100

# Основной маршрут через ISP1
sudo ip route add default scope global nexthop via 192.168.1.1 dev eth0 weight 2 nexthop via 192.168.2.1 dev eth1 weight 1

# Скрипт для мониторинга и failover
cat << 'EOF' | sudo tee /usr/local/bin/wan-monitor.sh
#!/bin/bash
PRIMARY_GW=192.168.1.1
SECONDARY_GW=192.168.2.1
CHECK_HOST=8.8.8.8

while true; do
    if ping -c 2 -W 2 -I eth0 $CHECK_HOST > /dev/null 2>&1; then
        # Primary работает
        ip route replace default via $PRIMARY_GW dev eth0
    else
        # Failover на secondary
        ip route replace default via $SECONDARY_GW dev eth1
    fi
    sleep 10
done
EOF

sudo chmod +x /usr/local/bin/wan-monitor.sh
```

**2. Используй tc (traffic control) для QoS:**
```bash
# Установка
sudo apt install iproute2

# Простой rate limiting на интерфейсе
sudo tc qdisc add dev eth0 root tbf rate 1mbit burst 32kbit latency 400ms

# HTB (Hierarchical Token Bucket) для классов трафика
# Root qdisc
sudo tc qdisc add dev eth0 root handle 1: htb default 30

# Классы
sudo tc class add dev eth0 parent 1: classid 1:1 htb rate 10mbit ceil 10mbit
sudo tc class add dev eth0 parent 1:1 classid 1:10 htb rate 5mbit ceil 10mbit prio 1  # High priority
sudo tc class add dev eth0 parent 1:1 classid 1:20 htb rate 3mbit ceil 8mbit prio 2   # Medium
sudo tc class add dev eth0 parent 1:1 classid 1:30 htb rate 2mbit ceil 6mbit prio 3   # Low (default)

# Фильтры для классификации трафика
sudo tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 match ip dport 22 0xffff flowid 1:10    # SSH high prio
sudo tc filter add dev eth0 protocol ip parent 1:0 prio 2 u32 match ip dport 80 0xffff flowid 1:20    # HTTP medium
sudo tc filter add dev eth0 protocol ip parent 1:0 prio 3 u32 match ip dport 443 0xffff flowid 1:20   # HTTPS medium

# Просмотр
sudo tc -s qdisc show dev eth0
sudo tc -s class show dev eth0
sudo tc -s filter show dev eth0

# Удаление
sudo tc qdisc del dev eth0 root
```

**3. Настрой ECMP (Equal-Cost Multi-Path) routing:**
```bash
# Добавь несколько маршрутов с одинаковой метрикой
sudo ip route add default \
    nexthop via 192.168.1.1 dev eth0 weight 1 \
    nexthop via 192.168.2.1 dev eth1 weight 1

# Проверь load balancing
for i in {1..10}; do
    ip route get 8.8.8.8
done

# Или с разными весами
sudo ip route add default \
    nexthop via 192.168.1.1 dev eth0 weight 2 \
    nexthop via 192.168.2.1 dev eth1 weight 1
```

---

## Модуль 6: Troubleshooting сети (30 минут)

### 🎯 Напоминалка

**Методология troubleshooting:**
````

1. Определи проблему (что не работает?)
2. Собери информацию
3. Изолируй причину (по уровням OSI)
4. Реши проблему
5. Проверь решение
6. Задокументируй

OSI Bottom-up подход: Layer 1 (Physical) → Кабель подключен? Link up? Layer 2 (Data Link) → MAC адреса? ARP работает? Layer 3 (Network) → IP адреса? Routing? Layer 4 (Transport) → Порты открыты? Firewall? Layer 7 (Application)→ Сервис работает?

````

**Основные инструменты:**
```bash
# Connectivity
ping          # ICMP echo
traceroute    # Путь до хоста
mtr           # Continuous traceroute
nc (netcat)   # TCP/UDP тестирование
telnet        # TCP подключение
curl          # HTTP requests
wget          # HTTP downloads

# Интерфейсы и адреса
ip            # Современная утилита
ifconfig      # Legacy
ethtool       # Информация об интерфейсе

# Routing
ip route      # Таблица маршрутизации
traceroute    # Traceroute
mtr           # Better traceroute

# DNS
dig           # DNS queries
nslookup      # Simple DNS lookup
host          # Quick DNS lookup

# Порты и соединения
ss            # Socket statistics
netstat       # Network statistics
lsof          # List open files/sockets
nmap          # Port scanner
nc            # TCP/UDP testing

# Packet capture
tcpdump       # Packet sniffer
tshark        # Wireshark CLI
wireshark     # GUI packet analyzer

# Performance
iperf3        # Bandwidth testing
speedtest-cli # Internet speed test
```

**ping диагностика:**
```bash
# Базовый ping
ping 8.8.8.8
ping google.com

# Ограничение количества пакетов
ping -c 4 8.8.8.8

# Изменение interval
ping -i 0.2 8.8.8.8    # 0.2 секунды (требует root)

# Изменение размера пакета
ping -s 1000 8.8.8.8   # 1000 байт payload

# Flood ping (тест производительности, требует root)
ping -f 8.8.8.8

# Ping с timestamp
ping -D 8.8.8.8

# Ping через конкретный интерфейс
ping -I eth0 8.8.8.8

# Интерпретация результатов
# время < 10ms     → Отлично (локальная сеть)
# время 10-50ms    → Хорошо
# время 50-100ms   → Приемлемо
# время > 100ms    → Медленно
# packet loss > 1% → Проблемы в сети
```

**traceroute диагностика:**
```bash
# UDP traceroute (по умолчанию)
traceroute google.com

# ICMP traceroute
traceroute -I google.com

# TCP traceroute
traceroute -T -p 80 google.com

# С номерами AS
traceroute -A google.com

# Увеличить max hops
traceroute -m 50 google.com

# mtr - better traceroute
mtr google.com
mtr --report --report-cycles 10 google.com
mtr --tcp google.com
```

**netcat (nc) - Swiss Army knife:**
```bash
# TCP подключение к порту
nc google.com 80
# GET / HTTP/1.0

# Проверка открыт ли порт
nc -zv google.com 80
nc -zv 192.168.1.100 22

# Сканирование портов
nc -zv 192.168.1.100 20-30

# UDP test
nc -u 8.8.8.8 53

# Слушать порт (простой сервер)
nc -l 8080

# Отправить файл
# На сервере:
nc -l 8080 > received_file
# На клиенте:
nc server_ip 8080 < file_to_send

# Chat между двумя хостами
# Хост A:
nc -l 8080
# Хост B:
nc host_a_ip 8080

# Banner grabbing
echo "HEAD / HTTP/1.0\r\n\r\n" | nc google.com 80
```

**ss (socket statistics):**
```bash
# Все соединения
ss -a

# Listening sockets
ss -l
ss -tunlp          # TCP/UDP listening + process

# Established соединения
ss -o state established

# Фильтр по порту
ss -tunp 'sport = :22'
ss -tunp 'dport = :80'

# Статистика по протоколам
ss -s

# Показать процессы
ss -p

# IPv4/IPv6
ss -4              # Только IPv4
ss -6              # Только IPv6

# Memory usage по соединениям
ss -m

# Таймеры
ss -o
```

**tcpdump - packet capture:**
```bash
# Захват всех пакетов
sudo tcpdump

# На конкретном интерфейсе
sudo tcpdump -i eth0

# Сохранить в файл
sudo tcpdump -i eth0 -w capture.pcap

# Читать из файла
sudo tcpdump -r capture.pcap

# Фильтр по хосту
sudo tcpdump host 192.168.1.100
sudo tcpdump src 192.168.1.100
sudo tcpdump dst 192.168.1.100

# Фильтр по порту
sudo tcpdump port 80
sudo tcpdump src port 22
sudo tcpdump dst port 443

# Фильтр по протоколу
sudo tcpdump tcp
sudo tcpdump udp
sudo tcpdump icmp

# Комбинированные фильтры
sudo tcpdump 'tcp port 80 and host 192.168.1.100'
sudo tcpdump 'tcp port 80 or tcp port 443'
sudo tcpdump 'not arp and not icmp'

# Показывать ASCII
sudo tcpdump -A

# Показывать hex и ASCII
sudo tcpdump -X

# Ограничение по количеству пакетов
sudo tcpdump -c 100

# Verbose output
sudo tcpdump -v
sudo tcpdump -vv
sudo tcpdump -vvv

# Не resolve hostname
sudo tcpdump -n

# TCP флаги
sudo tcpdump 'tcp[tcpflags] & (tcp-syn) != 0'  # SYN пакеты
sudo tcpdump 'tcp[tcpflags] & (tcp-rst) != 0'  # RST пакеты

# HTTP traffic
sudo tcpdump -A 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# DNS queries
sudo tcpdump -i any -s 0 port 53
```

**nmap - port scanner:**
```bash
# Scan single host
nmap 192.168.1.100

# Scan range
nmap 192.168.1.1-100
nmap 192.168.1.0/24

# Scan specific ports
nmap -p 22,80,443 192.168.1.100
nmap -p 1-1000 192.168.1.100
nmap -p- 192.168.1.100      # All ports

# Service version detection
nmap -sV 192.168.1.100

# OS detection
nmap -O 192.168.1.100

# Aggressive scan
nmap -A 192.168.1.100

# Scan без ping
nmap -Pn 192.168.1.100

# TCP SYN scan (stealth)
sudo nmap -sS 192.168.1.100

# UDP scan
sudo nmap -sU 192.168.1.100

# Fast scan (top 100 ports)
nmap -F 192.168.1.100

# Скрипты NSE
nmap --script http-title 192.168.1.100
nmap --script vuln 192.168.1.100
```

**iperf3 - bandwidth testing:**
```bash
# На сервере
iperf3 -s

# На клиенте
iperf3 -c server_ip

# Reverse test
iperf3 -c server_ip -R

# UDP test
iperf3 -c server_ip -u -b 100M

# Parallel streams
iperf3 -c server_ip -P 4

# Тест в течение 30 секунд
iperf3 -c server_ip -t 30

# JSON output
iperf3 -c server_ip -J
```

**Проблемы и решения:**
```bash
# Проблема: "Network is unreachable"
# Причина: Нет маршрута до сети
# Решение:
ip route show              # Проверить маршруты
ping gateway_ip            # Проверить gateway
ip link show               # Проверить интерфейс

# Проблема: "No route to host"
# Причина: Хост недостижим (может firewall)
# Решение:
ping host_ip
traceroute host_ip
sudo iptables -L -n        # Проверить firewall
nc -zv host_ip port        # Проверить порт

# Проблема: "Connection refused"
# Причина: Порт закрыт или сервис не слушает
# Решение:
ss -tunlp | grep port     # Проверить listening порты
sudo netstat -tunlp | grep port
systemctl status service   # Проверить сервис

# Проблема: "Connection timeout"
# Причина: Firewall блокирует или сеть медленная
# Решение:
sudo tcpdump -i any port XX  # Смотреть пакеты
sudo iptables -L -n -v      # Проверить firewall
traceroute host_ip          # Проверить путь

# Проблема: "DNS resolution failed"
# Причина: Проблемы с DNS
# Решение:
cat /etc/resolv.conf       # Проверить DNS серверы
dig @8.8.8.8 domain.com    # Тест с другим DNS
ping 8.8.8.8               # Проверить connectivity к DNS

# Проблема: Медленное соединение
# Причина: Packet loss, высокая latency, bandwidth проблемы
# Решение:
mtr -r -c 100 host         # Check packet loss
iperf3 -c host             # Bandwidth test
ping -s 1400 host          # MTU issues
```

### 💻 Задание

Практика troubleshooting:

1. **Диагностика сетевого стека (bottom-up):**
```bash
   # Layer 1/2: Physical & Data Link
   ip link show                     # Интерфейсы UP?
   ethtool eth0                     # Link detected?
   ip -s link show eth0             # Errors/drops?
   
   # Layer 3: Network
   ip addr show                     # IP адрес назначен?
   ip route show                    # Default gateway есть?
   ping gateway_ip                  # Gateway доступен?
   ping 8.8.8.8                     # Internet доступен?
   
   # Layer 4: Transport
   ss -tunlp                        # Сервисы listening?
   sudo iptables -L -n              # Firewall блокирует?
   
   # Layer 7: Application
   systemctl status service_name    # Сервис запущен?
   journalctl -u service_name       # Errors в логах?
```

2. **Packet capture с tcpdump:**
```bash
   # Запусти HTTP сервер
   python3 -m http.server 8080 &
   
   # Захвати трафик на порт 8080
   sudo tcpdump -i lo port 8080 -w /tmp/capture.pcap &
   TCPDUMP_PID=$!
   
   # Сделай несколько запросов
   curl http://localhost:8080/
   curl http://localhost:8080/non-existent
   
   # Останови tcpdump
   sudo kill $TCPDUMP_PID
   
   # Анализируй capture
   sudo tcpdump -r /tmp/capture.pcap -A
   sudo tcpdump -r /tmp/capture.pcap 'tcp[tcpflags] & (tcp-syn) != 0'
   
   # Cleanup
   pkill -f "python3 -m http.server"
```

3. **Port scanning с nmap:**
```bash
   # Scan localhost
   nmap localhost
   
   # Scan с service detection
   nmap -sV localhost
   
   # Scan конкретные порты
   nmap -p 22,80,443,3306 localhost
   
   # Проверь что сервис действительно listening
   ss -tunlp | grep -E ':(22|80|443|3306)'
```

4. **Bandwidth testing с iperf3:**
```bash
   # Установи iperf3
   sudo apt install iperf3 -y
   
   # Запусти сервер
   iperf3 -s &
   IPERF_PID=$!
   
   # Тест в loopback
   iperf3 -c localhost -t 10
   
   # Тест с parallel streams
   iperf3 -c localhost -P 4 -t 10
   
   # Останови сервер
   kill $IPERF_PID
```

5. **Connectivity matrix test:**
```bash
   # Создай скрипт для массовой проверки
   cat << 'EOF' > /tmp/connectivity_test.sh
   #!/bin/bash
   HOSTS="8.8.8.8 1.1.1.1 google.com github.com"
   PORTS="80 443"
   
   for host in $HOSTS; do
       echo "Testing $host..."
       ping -c 2 -W 2 $host && echo "  ✓ ICMP OK" || echo "  ✗ ICMP FAILED"
       
       for port in $PORTS; do
           nc -zv -w 2 $host $port 2>&1 | grep -q succeeded && \
               echo "  ✓ Port $port OK" || \
               echo "  ✗ Port $port FAILED"
       done
       echo
   done
   EOF
   
   chmod +x /tmp/connectivity_test.sh
   /tmp/connectivity_test.sh
```

6. **DNS troubleshooting:**
```bash
   # Проверь DNS серверы
   cat /etc/resolv.conf
   
   # Тест через разные DNS
   dig @8.8.8.8 google.com +short
   dig @1.1.1.1 google.com +short
   dig @9.9.9.9 google.com +short
   
   # Сравни время ответа
   for dns in 8.8.8.8 1.1.1.1 9.9.9.9; do
       echo -n "DNS $dns: "
       time dig @$dns google.com +short > /dev/null
   done
   
   # Trace DNS запроса
   dig google.com +trace | head -30
```

### 🚀 Бонус (новое)

**1. Создай комплексный network monitoring скрипт:**
```bash
cat << 'EOF' > /tmp/network_monitor.sh
#!/bin/bash

# Network Monitoring Script

echo "=== Network Diagnostics Report ==="
echo "Generated: $(date)"
echo

echo "--- Interfaces ---"
ip -brief link show
echo

echo "--- IP Addresses ---"
ip -brief addr show
echo

echo "--- Routes ---"
ip route show
echo

echo "--- DNS Servers ---"
cat /etc/resolv.conf | grep nameserver
echo

echo "--- Active Connections ---"
ss -tunp | head -20
echo

echo "--- Listening Ports ---"
ss -tunlp
echo

echo "--- Firewall Rules ---"
sudo iptables -L -n -v | head -30
echo

echo "--- Connectivity Tests ---"
for host in 8.8.8.8 google.com; do
    if ping -c 2 -W 2 $host > /dev/null 2>&1; then
        echo "✓ $host reachable"
    else
        echo "✗ $host unreachable"
    fi
done
echo

echo "--- DNS Resolution Test ---"
for domain in google.com github.com; do
    if dig $domain +short > /dev/null 2>&1; then
        echo "✓ $domain resolves"
    else
        echo "✗ $domain fails"
    fi
done
echo

echo "--- Interface Statistics ---"
ip -s link show | grep -A 5 "eth0\|ens\|enp"
echo

echo "=== End of Report ==="
EOF

chmod +x /tmp/network_monitor.sh
sudo /tmp/network_monitor.sh
```

**2. Используй Wireshark/tshark для продвинутого анализа:**
```bash
# Установка
sudo apt install tshark

# Capture с автоматической остановкой
sudo tshark -i any -a duration:30 -w /tmp/capture.pcap

# HTTP анализ
sudo tshark -r /tmp/capture.pcap -Y "http.request"
sudo tshark -r /tmp/capture.pcap -Y "http.response.code == 404"

# DNS анализ
sudo tshark -r /tmp/capture.pcap -Y "dns"
sudo tshark -r /tmp/capture.pcap -Y "dns.qry.name contains google"

# TCP SYN packets
sudo tshark -r /tmp/capture.pcap -Y "tcp.flags.syn==1 && tcp.flags.ack==0"

# Follow TCP stream
sudo tshark -r /tmp/capture.pcap -z follow,tcp,ascii,0

# Статистика
sudo tshark -r /tmp/capture.pcap -z io,phs
sudo tshark -r /tmp/capture.pcap -z conv,tcp
```

**3. Performance tuning и диагностика:**
```bash
# Проверь параметры TCP
sysctl -a | grep tcp

# Важные параметры для производительности
sudo sysctl net.core.rmem_max=134217728
sudo sysctl net.core.wmem_max=134217728
sudo sysctl net.ipv4.tcp_rmem="4096 87380 134217728"
sudo sysctl net.ipv4.tcp_wmem="4096 65536 134217728" 
sudo sysctl net.ipv4.tcp_window_scaling=1 
sudo sysctl net.ipv4.tcp_timestamps=1 
sudo sysctl net.ipv4.tcp_sack=1 
sudo sysctl net.core.netdev_max_backlog=5000

# MTU testing

ip link show | grep mtu ping -M do -s 1472 -c 1 google.com # 1500 - 28

# Ring buffer size

ethtool -g eth0 sudo ethtool -G eth0 rx 4096 tx 4096

# Offloading

ethtool -k eth0 sudo ethtool -K eth0 tso on gso on

```

---

## Финальный проект (60 минут)

### Задача: Настроить multi-tier network infrastructure

Создай изолированную сетевую инфраструктуру с несколькими сегментами:

**Архитектура:**
```
Internet
│
├─ Gateway/Router (NAT + Firewall)
│
├─ DMZ (192.168.100.0/24)
│   ├─ Web Server (192.168.100.10)
│   └─ Mail Server (192.168.100.11)
│
├─ Internal Network (192.168.1.0/24)
│   ├─ App Servers (192.168.1.10-20)
│   └─ Workstations (192.168.1.100-200)
│
└─ Management Network (192.168.99.0/24)
└─ Monitoring/Jump Box (192.168.99.10)
````

**Требования:**

1. **Network Namespaces (симуляция разных сетей):**
   - Создай 4 namespaces: gateway, dmz, internal, mgmt
   - Настрой veth пары между ними
   - Назначь IP адреса согласно схеме

2. **Routing:**
   - Gateway должен маршрутизировать между всеми сетями
   - Включи IP forwarding
   - Настрой статические маршруты где необходимо

3. **NAT:**
   - MASQUERADE для выхода в интернет из всех сетей
   - Port forwarding: внешний 80 → DMZ web server 80
   - Port forwarding: внешний 443 → DMZ web server 443

4. **Firewall (iptables/nftables):**
   - Default policy: DROP для INPUT и FORWARD
   - Разрешить established/related
   - DMZ → Internet: разрешить HTTP/HTTPS outbound
   - Internal → DMZ: разрешить HTTP/HTTPS
   - Internal → Internet: разрешить через NAT
   - Management → все сети: разрешить SSH
   - Заблокировать прямой доступ Internet → Internal

5. **DNS:**
   - Настрой локальный DNS сервер (dnsmasq)
   - Локальные записи для всех серверов
   - Forward внешние запросы на 8.8.8.8

6. **Services:**
   - Web server в DMZ (nginx/python http.server)
   - Простой API server в Internal
   - Monitoring (простой ping monitor)

7. **Monitoring:**
   - Скрипт для проверки connectivity между сегментами
   - Packet capture между сегментами
   - Bandwidth test между зонами

8. **Documentation:**
   - README с описанием архитектуры
   - Скрипты для setup и teardown
   - Диаграмма сети

**Starter скрипт:**
```bash
#!/bin/bash
# network-lab-setup.sh

set -e

echo "=== Network Lab Setup ==="

# Функция очистки
cleanup() {
    echo "Cleaning up..."
    ip netns del gateway 2>/dev/null || true
    ip netns del dmz 2>/dev/null || true
    ip netns del internal 2>/dev/null || true
    ip netns del mgmt 2>/dev/null || true
    
    # Удали все veth интерфейсы
    ip link show | grep veth | cut -d: -f2 | xargs -I {} ip link del {} 2>/dev/null || true
    
    # Очисти iptables
    iptables -t nat -F
    iptables -F
}

# Cleanup при ошибке
trap cleanup EXIT

# 1. Создай namespaces
echo "Creating namespaces..."
ip netns add gateway
ip netns add dmz
ip netns add internal
ip netns add mgmt

# 2. Создай veth пары
echo "Creating veth pairs..."
# Gateway <-> DMZ
ip link add veth-gw-dmz type veth peer name veth-dmz-gw
ip link set veth-dmz-gw netns dmz
ip link set veth-gw-dmz netns gateway

# Gateway <-> Internal
ip link add veth-gw-int type veth peer name veth-int-gw
ip link set veth-int-gw netns internal
ip link set veth-gw-int netns gateway

# Gateway <-> Management
ip link add veth-gw-mgmt type veth peer name veth-mgmt-gw
ip link set veth-mgmt-gw netns mgmt
ip link set veth-gw-mgmt netns gateway

# 3. Настрой IP адреса
echo "Configuring IP addresses..."

# Gateway
ip netns exec gateway ip addr add 192.168.100.1/24 dev veth-gw-dmz
ip netns exec gateway ip addr add 192.168.1.1/24 dev veth-gw-int
ip netns exec gateway ip addr add 192.168.99.1/24 dev veth-gw-mgmt
ip netns exec gateway ip link set veth-gw-dmz up
ip netns exec gateway ip link set veth-gw-int up
ip netns exec gateway ip link set veth-gw-mgmt up
ip netns exec gateway ip link set lo up

# DMZ
ip netns exec dmz ip addr add 192.168.100.10/24 dev veth-dmz-gw
ip netns exec dmz ip link set veth-dmz-gw up
ip netns exec dmz ip link set lo up
ip netns exec dmz ip route add default via 192.168.100.1

# Internal
ip netns exec internal ip addr add 192.168.1.10/24 dev veth-int-gw
ip netns exec internal ip link set veth-int-gw up
ip netns exec internal ip link set lo up
ip netns exec internal ip route add default via 192.168.1.1

# Management
ip netns exec mgmt ip addr add 192.168.99.10/24 dev veth-mgmt-gw
ip netns exec mgmt ip link set veth-mgmt-gw up
ip netns exec mgmt ip link set lo up
ip netns exec mgmt ip route add default via 192.168.99.1

# 4. Включи IP forwarding в gateway
echo "Enabling IP forwarding..."
ip netns exec gateway sysctl -w net.ipv4.ip_forward=1

# 5. Настрой NAT
echo "Configuring NAT..."
# MASQUERADE для выхода в интернет (через default namespace)
ip netns exec gateway iptables -t nat -A POSTROUTING -o veth-gw-ext -j MASQUERADE

# 6. Настрой firewall
echo "Configuring firewall..."
ip netns exec gateway iptables -P INPUT DROP
ip netns exec gateway iptables -P FORWARD DROP
ip netns exec gateway iptables -P OUTPUT ACCEPT

# Разрешить loopback
ip netns exec gateway iptables -A INPUT -i lo -j ACCEPT

# Разрешить established/related
ip netns exec gateway iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
ip netns exec gateway iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Разрешить Internal → DMZ (HTTP/HTTPS)
ip netns exec gateway iptables -A FORWARD -s 192.168.1.0/24 -d 192.168.100.0/24 -p tcp -m multiport --dports 80,443 -j ACCEPT

# Разрешить Management → везде (SSH)
ip netns exec gateway iptables -A FORWARD -s 192.168.99.0/24 -p tcp --dport 22 -j ACCEPT

# Разрешить DMZ → Internet
ip netns exec gateway iptables -A FORWARD -s 192.168.100.0/24 -j ACCEPT

# Разрешить Internal → Internet
ip netns exec gateway iptables -A FORWARD -s 192.168.1.0/24 -j ACCEPT

echo "=== Lab Setup Complete ==="
echo
echo "Test connectivity:"
echo "  ip netns exec internal ping -c 2 192.168.100.10  # Internal → DMZ"
echo "  ip netns exec dmz ping -c 2 8.8.8.8              # DMZ → Internet"
echo "  ip netns exec mgmt ping -c 2 192.168.1.10        # Mgmt → Internal"
echo
echo "Start services:"
echo "  ip netns exec dmz python3 -m http.server 80 &"
echo "  ip netns exec internal python3 -m http.server 8080 &"
echo
echo "Cleanup:"
echo "  ./network-lab-teardown.sh"
```

**Дополнительные улучшения (опционально):**
- VLAN tagging
- QoS с tc
- IDS/IPS с Suricata
- VPN туннель между сегментами
- Load balancing с HAProxy
- Centralized logging
- Grafana dashboard для метрик
- Automated testing скрипты

---

## Справочная секция: Быстрые шпаргалки

### IP Subnetting Quick Reference
````

CIDR    Subnet Mask       Hosts    Networks (Class C)
/30     255.255.255.252   2        64
/29     255.255.255.248   6        32
/28     255.255.255.240   14       16
/27     255.255.255.224   30       8
/26     255.255.255.192   62       4
/25     255.255.255.128   126      2
/24     255.255.255.0     254      1
/23     255.255.254.0     510      2 Class C
/22     255.255.252.0     1022     4 Class C
/21     255.255.248.0     2046     8 Class C
/20     255.255.240.0     4094     16 Class C
/19     255.255.224.0     8190     32 Class C
/18     255.255.192.0     16382    64 Class C
/17     255.255.128.0     32766    128 Class C
/16     255.255.0.0       65534    256 Class C (Class B)

````

### Common Ports Reference
```
ПРОТОКОЛ    PORT    ОПИСАНИЕ
SSH         22      Secure Shell
Telnet      23      Unencrypted text communications
SMTP        25      Simple Mail Transfer Protocol
DNS         53      Domain Name System
HTTP        80      Hypertext Transfer Protocol
POP3        110     Post Office Protocol 3
IMAP        143     Internet Message Access Protocol
HTTPS       443     HTTP Secure
SMB         445     Server Message Block (Windows shares)
SMTPS       587     SMTP Secure
LDAP        389     Lightweight Directory Access Protocol
LDAPS       636     LDAP Secure
MySQL       3306    MySQL Database
PostgreSQL  5432    PostgreSQL Database
MongoDB     27017   MongoDB Database
Redis       6379    Redis Database
Docker      2375    Docker REST API
Kubernetes  6443    Kubernetes API
Prometheus  9090    Prometheus Metrics
Grafana     3000    Grafana Dashboard

````

### iptables Quick Rules
```bash
# Сброс всех правил
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

# Базовая защита
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -j DROP

# Простой NAT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Port forwarding
iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to 192.168.1.10:80

# Блокировка IP
iptables -A INPUT -s 1.2.3.4 -j DROP

# Rate limiting
iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT
```

### Quick Network Commands
```bash
# Быстрая диагностика
ip a                          # IP адреса
ip r                          # Маршруты
ss -tunlp                     # Listening ports
ping -c 4 8.8.8.8            # Connectivity
dig google.com +short         # DNS
curl -I http://site.com      # HTTP test

# Быстрый troubleshooting
sudo tcpdump -i any -c 100    # Packet capture
sudo iptables -L -n -v        # Firewall rules
sudo netstat -tunlp           # Connections
mtr -r -c 10 host            # Path analysis
nc -zv host port             # Port check
```

---

## Чек-лист навыков

После прохождения курса ты должен уметь:

### Базовые навыки:
- ✅ Понимать модель OSI/TCP-IP
- ✅ Рассчитывать подсети (subnetting)
- ✅ Настраивать сетевые интерфейсы
- ✅ Работать с routing таблицами
- ✅ Использовать ping, traceroute, dig
- ✅ Проверять connectivity и диагностировать проблемы

### Продвинутые навыки:
- ✅ Настраивать iptables/nftables firewall
- ✅ Настраивать NAT и port forwarding
- ✅ Работать с DNS (bind9, dnsmasq)
- ✅ Использовать tcpdump для анализа трафика
- ✅ Настраивать VLAN и bridge интерфейсы
- ✅ Работать с network namespaces

### Expert навыки:
- ✅ Policy-based routing
- ✅ QoS и traffic shaping с tc
- ✅ Troubleshooting сложных сетевых проблем
- ✅ Performance tuning сетевого стека
- ✅ Настройка multi-WAN с failover
- ✅ Packet analysis с Wireshark/tshark

---

## Заключение

Поздравляю! Ты прошел курс по освежению знаний сетей для DevOps/SysAdmin.

**Следующие шаги:**
1. Практикуйся регулярно - создай home lab
2. Изучай смежные технологии: Service Mesh, SDN, eBPF
3. Получи сертификацию (Network+, CCNA)
4. Изучи cloud networking (AWS VPC, Azure VNet, GCP VPC)
5. Делись знаниями - помогай новичкам

**Помни:**
- Сети - основа всей IT инфраструктуры
- Начинай с простого, усложняй постепенно
- Документация и packet captures - твои лучшие друзья
- Practice makes perfect!

Проходи этот курс каждые 6-12 месяцев, чтобы оставаться в форме!

Happy Networking! 🌐🚀
