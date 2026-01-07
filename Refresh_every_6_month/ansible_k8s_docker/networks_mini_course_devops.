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
sudo ufw allow 80/tcp # Разрешить HTTP TCP 
sudo ufw allow 443/tcp # Разрешить HTTPS 
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
# Внешний порт 8080 → внутренний 192.168.1.10:80
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.10:80

# Также нужно разрешить forwarding
sudo iptables -A FORWARD -p tcp -d 192.168.1.10 --dport 80 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT

# Port forwarding с изменением порта
sudo iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 192.168.1.10:22

# Port forwarding для диапазона портов
sudo iptables -t nat -A PREROUTING -p tcp --dport 5000:5100 -j DNAT --to-destination 192.168.1.10

# 1:1 NAT (весь трафик для IP)
sudo iptables -t nat -A PREROUTING -d 203.0.113.50 -j DNAT --to-destination 192.168.1.10
sudo iptables -t nat -A POSTROUTING -s 192.168.1.10 -j SNAT --to-source 203.0.113.50

# Просмотр NAT таблицы
sudo iptables -t nat -L -v -n

# Просмотр connection tracking
sudo conntrack -L
sudo conntrack -L | grep 192.168.1.10
```

**Настройка маршрутизатора между сетями:**

bash

```bash
# Сценарий: Linux box с двумя интерфейсами
# eth0: 203.0.113.50/24 (WAN, интернет)
# eth1: 192.168.1.1/24 (LAN, внутренняя сеть)

# 1. Включить IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

# 2. Настроить NAT для выхода в интернет
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# 3. Разрешить forwarding
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# 4. На клиентах в LAN настроить default gateway = 192.168.1.1
```

**Диагностика маршрутизации:**

bash

```bash
# Проверка маршрута до хоста
ip route get 8.8.8.8

# Traceroute
traceroute google.com
traceroute -n google.com           # Без DNS resolution
traceroute -I google.com           # ICMP вместо UDP
traceroute -T -p 80 google.com     # TCP на порт 80

# MTR (комбинация ping и traceroute)
mtr google.com
mtr --report --report-cycles 10 google.com
mtr --tcp -P 443 google.com        # TCP mode

# Проверка connectivity через конкретный gateway
ping -I eth0 8.8.8.8
ping -I 192.168.1.100 8.8.8.8

# Анализ path MTU
tracepath google.com
```

**Продвинутая маршрутизация:**

bash

```bash
# ECMP (Equal-Cost Multi-Path) - балансировка между маршрутами
sudo ip route add default \
    nexthop via 192.168.1.1 weight 1 \
    nexthop via 192.168.2.1 weight 1

# Маршрут с метрикой (приоритетом)
sudo ip route add 10.0.0.0/8 via 192.168.1.254 metric 100
sudo ip route add 10.0.0.0/8 via 192.168.2.254 metric 200

# Source-based routing
sudo ip rule add from 192.168.1.100 table isp1
sudo ip rule add from 192.168.1.101 table isp2

# Mark-based routing (совместно с iptables)
sudo iptables -t mangle -A PREROUTING -p tcp --dport 80 -j MARK --set-mark 1
sudo ip rule add fwmark 1 table isp1
```

### 💻 Задание

Настрой базовую маршрутизацию и NAT:

1. **Проверь текущую конфигурацию маршрутизации:**

bash

```bash
   # Таблица маршрутизации
   ip route show
   
   # Default gateway
   ip route show default
   
   # Метрики маршрутов
   ip route show | grep metric
   
   # IP forwarding
   sysctl net.ipv4.ip_forward
   sysctl net.ipv6.conf.all.forwarding
```

2. **Протестируй разные маршруты:**

bash

```bash
   # Узнай маршрут до конкретного хоста
   ip route get 8.8.8.8
   ip route get google.com
   ip route get 192.168.1.1
   
   # Traceroute до разных хостов
   traceroute -n 8.8.8.8
   traceroute google.com
   mtr --report --report-cycles 5 google.com
```

3. **Создай тестовые статические маршруты:**

bash

```bash
   # Добавь маршрут к несуществующей сети (для теста)
   sudo ip route add 10.99.99.0/24 via 192.168.1.1
   
   # Проверь
   ip route show | grep 10.99.99
   
   # Попробуй "достучаться" (не должно работать)
   ping -c 2 -W 1 10.99.99.1
   
   # Удали маршрут
   sudo ip route del 10.99.99.0/24
```

4. **Проверь NAT (если есть):**

bash

```bash
   # Посмотри NAT правила
   sudo iptables -t nat -L -v -n
   
   # Connection tracking
   sudo apt install conntrack -y
   sudo conntrack -L | head -20
   
   # Статистика
   sudo conntrack -S
```

5. **Эмулируй простую маршрутизацию с network namespaces:**

bash

```bash
   # Создай два namespace (две "виртуальные машины")
   sudo ip netns add router
   sudo ip netns add client
   
   # Создай veth пары
   sudo ip link add veth-r type veth peer name veth-c
   sudo ip link add veth-r-wan type veth peer name veth-wan
   
   # Перемести интерфейсы в namespaces
   sudo ip link set veth-r netns router
   sudo ip link set veth-c netns client
   sudo ip link set veth-r-wan netns router
   
   # Настрой router namespace
   sudo ip netns exec router ip addr add 10.0.1.1/24 dev veth-r
   sudo ip netns exec router ip addr add 10.0.0.1/24 dev veth-r-wan
   sudo ip netns exec router ip link set veth-r up
   sudo ip netns exec router ip link set veth-r-wan up
   sudo ip netns exec router ip link set lo up
   
   # Включи forwarding в router
   sudo ip netns exec router sysctl -w net.ipv4.ip_forward=1
   
   # Настрой client namespace
   sudo ip netns exec client ip addr add 10.0.1.10/24 dev veth-c
   sudo ip netns exec client ip link set veth-c up
   sudo ip netns exec client ip link set lo up
   sudo ip netns exec client ip route add default via 10.0.1.1
   
   # Настрой "WAN" интерфейс в host namespace
   sudo ip addr add 10.0.0.254/24 dev veth-wan
   sudo ip link set veth-wan up
   
   # Проверь connectivity
   sudo ip netns exec client ping -c 2 10.0.1.1        # client → router
   sudo ip netns exec client ping -c 2 10.0.0.254      # client → wan
   
   # Настрой NAT в router
   sudo ip netns exec router iptables -t nat -A POSTROUTING -o veth-r-wan -j MASQUERADE
   sudo ip netns exec router iptables -A FORWARD -i veth-r -o veth-r-wan -j ACCEPT
   sudo ip netns exec router iptables -A FORWARD -i veth-r-wan -o veth-r -m state --state RELATED,ESTABLISHED -j ACCEPT
   
   # Проверь NAT
   sudo ip netns exec client ping -c 2 10.0.0.254
   
   # Посмотри connection tracking
   sudo ip netns exec router conntrack -L
   
   # Очистка
   sudo ip netns del router
   sudo ip netns del client
   sudo ip link del veth-wan
```

### 🚀 Бонус (новое)

**1. Настрой port knocking для безопасного доступа:**

bash

```bash
# Установи knockd
sudo apt install knockd

# Конфигурация /etc/knockd.conf
sudo tee /etc/knockd.conf << 'EOF'
[options]
    logfile = /var/log/knockd.log

[openSSH]
    sequence    = 7000,8000,9000
    seq_timeout = 5
    command     = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    tcpflags    = syn

[closeSSH]
    sequence    = 9000,8000,7000
    seq_timeout = 5
    command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    tcpflags    = syn
EOF

# Настрой iptables (заблокируй SSH по умолчанию)
sudo iptables -A INPUT -p tcp --dport 22 -j DROP

# Запусти knockd
sudo systemctl start knockd
sudo systemctl enable knockd

# Тест (с другой машины)
# knock your-server-ip 7000 8000 9000
# ssh your-server-ip
# knock your-server-ip 9000 8000 7000  # Закрыть после работы
```

**2. Используй policy routing для разделения трафика:**

bash

```bash
# Сценарий: два ISP, разный трафик через разных провайдеров

# Создай таблицы
echo "100 isp1" | sudo tee -a /etc/iproute2/rt_tables
echo "101 isp2" | sudo tee -a /etc/iproute2/rt_tables

# Настрой маршруты в таблицах
sudo ip route add default via 192.168.1.1 dev eth0 table isp1
sudo ip route add default via 192.168.2.1 dev eth1 table isp2

# Добавь локальные сети в обе таблицы
sudo ip route add 192.168.1.0/24 dev eth0 table isp1
sudo ip route add 192.168.2.0/24 dev eth1 table isp2

# Правила: весь HTTP/HTTPS через ISP1, остальное через ISP2
sudo iptables -t mangle -A PREROUTING -p tcp -m multiport --dports 80,443 -j MARK --set-mark 1
sudo ip rule add fwmark 1 table isp1
sudo ip rule add from all table isp2 priority 200

# Или по source IP
sudo ip rule add from 192.168.1.0/24 table isp1
sudo ip rule add from 192.168.2.0/24 table isp2

# Проверка
ip rule show
ip route show table isp1
ip route show table isp2
```

**3. Настрой transparent proxy с маршрутизацией:**

bash

```bash
# Сценарий: перенаправить весь HTTP трафик на Squid proxy

# Установи Squid
sudo apt install squid

# Настрой Squid для transparent mode
sudo tee -a /etc/squid/squid.conf << 'EOF'
http_port 3129 intercept
acl localnet src 192.168.1.0/24
http_access allow localnet
EOF

sudo systemctl restart squid

# Включи IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Перенаправь HTTP трафик на Squid
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80 -j REDIRECT --to-port 3129

# Проверь (с клиента в LAN)
curl -I http://example.com

# Посмотри логи Squid
sudo tail -f /var/log/squid/access.log
```

---

## Модуль 6: Анализ сетевого трафика (35 минут)

### 🎯 Напоминалка

**tcpdump - основной инструмент захвата пакетов:**

bash

```bash
# Базовый синтаксис
tcpdump [options] [filter expression]

# Основные опции
-i <interface>    # Интерфейс для захвата
-n               # Не преобразовывать адреса в имена
-nn              # Не преобразовывать адреса и порты
-v, -vv, -vvv    # Уровень детализации
-c <count>       # Захватить N пакетов
-w <file>        # Записать в файл
-r <file>        # Читать из файла
-A               # Печатать содержимое пакетов в ASCII
-X               # Печатать в hex и ASCII
-s <snaplen>     # Размер захвата (0 = весь пакет)
```

**Фильтры tcpdump:**

bash

```bash
# По хосту
tcpdump host 192.168.1.100
tcpdump src 192.168.1.100
tcpdump dst 192.168.1.100

# По сети
tcpdump net 192.168.1.0/24
tcpdump net 192.168.1.0 mask 255.255.255.0

# По протоколу
tcpdump tcp
tcpdump udp
tcpdump icmp
tcpdump arp

# По порту
tcpdump port 80
tcpdump src port 80
tcpdump dst port 80
tcpdump portrange 8000-9000

# Комбинированные фильтры
tcpdump 'tcp and port 80'
tcpdump 'host 192.168.1.100 and port 22'
tcpdump 'tcp and (port 80 or port 443)'
tcpdump 'net 192.168.1.0/24 and not host 192.168.1.1'

# TCP флаги
tcpdump 'tcp[tcpflags] & (tcp-syn) != 0'      # SYN пакеты
tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-ack) == tcp-syn'  # Только SYN
tcpdump 'tcp[tcpflags] & (tcp-rst) != 0'      # RST пакеты

# Размер пакета
tcpdump 'ip[2:2] > 576'                       # Пакеты больше 576 байт
tcpdump 'less 64'                             # Пакеты меньше 64 байт

# ICMP типы
tcpdump 'icmp[icmptype] == icmp-echo'         # Ping request
tcpdump 'icmp[icmptype] == icmp-echoreply'    # Ping reply
```

**Примеры использования tcpdump:**

bash

```bash
# Захват всего трафика на интерфейсе
sudo tcpdump -i eth0

# Захват HTTP трафика
sudo tcpdump -i eth0 port 80 -A

# Захват DNS запросов
sudo tcpdump -i eth0 port 53 -n

# Захват SSH трафика с конкретным хостом
sudo tcpdump -i eth0 'host 192.168.1.100 and port 22' -nn

# Захват и сохранение в файл
sudo tcpdump -i eth0 -w capture.pcap

# Захват первых 100 пакетов
sudo tcpdump -i eth0 -c 100 -w capture.pcap

# Чтение из файла
tcpdump -r capture.pcap

# Захват с временными метками
sudo tcpdump -i eth0 -tttt

# Захват только заголовков (без данных)
sudo tcpdump -i eth0 -s 0 port 80

# Захват трафика, исключая SSH (чтобы не видеть свои команды)
sudo tcpdump -i eth0 'not port 22'

# Мониторинг SYN flood атаки
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0'
```

**Wireshark - GUI инструмент анализа:**

bash

```bash
# Установка
sudo apt install wireshark

# Разрешить захват без root
sudo dpkg-reconfigure wireshark-common  # Выбрать Yes
sudo usermod -aG wireshark $USER
# Перелогиниться

# Использование из CLI
tshark -i eth0                           # CLI версия Wireshark
tshark -i eth0 -c 10                     # 10 пакетов
tshark -i eth0 -w capture.pcap           # Сохранить в файл
tshark -r capture.pcap                   # Читать файл
tshark -r capture.pcap -Y "http"         # С фильтром

# Display фильтры tshark
tshark -r capture.pcap -Y "ip.addr == 192.168.1.100"
tshark -r capture.pcap -Y "tcp.port == 80"
tshark -r capture.pcap -Y "http.request"
tshark -r capture.pcap -Y "dns"

# Экспорт объектов
tshark -r capture.pcap --export-objects http,./exported
```

**ngrep - grep для сетевого трафика:**

bash

```bash
# Установка
sudo apt install ngrep

# Базовое использование
sudo ngrep -d eth0                       # Весь трафик
sudo ngrep -d eth0 'GET|POST'            # HTTP методы
sudo ngrep -d eth0 'User-Agent' port 80  # User-Agent в HTTP
sudo ngrep -d eth0 '' port 53            # DNS запросы
sudo ngrep -d eth0 'password'            # Поиск слова password

# С хостом
sudo ngrep -d eth0 'GET' host 192.168.1.100

# В hex формате
sudo ngrep -d eth0 -x '' port 80

# Сохранение в файл
sudo ngrep -d eth0 -O capture.pcap
```

**Анализ производительности сети:**

bash

```bash
# iftop - top для сети
sudo apt install iftop
sudo iftop -i eth0                       # По интерфейсу
sudo iftop -i eth0 -n                    # Без DNS resolution
sudo iftop -i eth0 -P                    # Показать порты

# iptraf-ng - интерактивная статистика
sudo apt install iptraf-ng
sudo iptraf-ng                           # Интерактивное меню
sudo iptraf-ng -i eth0                   # Мониторинг интерфейса

# nethogs - по процессам
sudo apt install nethogs
sudo nethogs eth0                        # Трафик по процессам

# vnstat - долгосрочная статистика
sudo apt install vnstat
vnstat -i eth0                           # Статистика
vnstat -i eth0 -l                        # Live view
vnstat -i eth0 -h                        # Почасовая статистика
vnstat -i eth0 -d                        # Дневная статистика

# bmon - bandwidth monitor
sudo apt install bmon
bmon -p eth0                             # Мониторинг интерфейса

# speedtest-cli - тест скорости
sudo apt install speedtest-cli
speedtest-cli                            # Простой тест
speedtest-cli --simple                   # Краткий вывод
speedtest-cli --json                     # JSON вывод
```

**Мониторинг соединений:**

bash

```bash
# ss - socket statistics
ss -tunap                                # Все TCP/UDP соединения
ss -tn state established                 # Только established TCP
ss -tn state time-wait                   # TIME-WAIT соединения
ss -tlnp                                 # Listening TCP порты
ss -s                                    # Статистика
ss -o state established '( dport = :ssh or sport = :ssh )'  # SSH соединения

# netstat (устаревший, но все еще полезный)
netstat -tunap                           # Все соединения
netstat -i                               # Статистика интерфейсов
netstat -s                               # Статистика протоколов

# lsof - list open files (включая сетевые)
sudo lsof -i                             # Все сетевые соединения
sudo lsof -i :80                         # Кто слушает порт 80
sudo lsof -i tcp                         # Только TCP
sudo lsof -i @192.168.1.100              # Соединения с IP
sudo lsof -u www-data -i                 # Соединения процессов пользователя

# nmap - сканирование портов
nmap 192.168.1.100                       # Сканирование хоста
nmap -p 22,80,443 192.168.1.100          # Конкретные порты
nmap -p- 192.168.1.100                   # Все порты
nmap -sV 192.168.1.100                   # Определение версий
nmap -O 192.168.1.100                    # Определение ОС
nmap 192.168.1.0/24                      # Сканирование сети
```

### 💻 Задание

Анализ сетевого трафика:

1. **Базовый захват с tcpdump:**

bash

```bash
   # Захвати 20 пакетов на любом интерфейсе
   sudo tcpdump -i any -c 20 -nn
   
   # Захвати только ICMP
   sudo tcpdump -i any icmp -nn
   
   # Запусти ping в другом терминале
   ping -c 5 8.8.8.8
   
   # Посмотри ICMP пакеты
   sudo tcpdump -i any icmp -nn -c 10
```

2. **Захват HTTP трафика:**

bash

```bash
   # Захвати HTTP в verbose режиме
   sudo tcpdump -i any port 80 -A -s 0 &
   TCPDUMP_PID=$!
   
   # Сделай HTTP запрос
   curl http://example.com
   
   # Останови tcpdump
   sudo kill $TCPDUMP_PID
```

3. **Анализ DNS запросов:**

bash

```bash
   # Захват DNS
   sudo tcpdump -i any port 53 -nn &
   TCPDUMP_PID=$!
   
   # Сделай несколько DNS запросов
   dig google.com
   dig github.com
   nslookup example.com
   
   # Останови захват
   sudo kill $TCPDUMP_PID
```

4. **Сохранение и анализ capture файла:**

bash

```bash
   # Захвати трафик в файл (30 секунд)
   sudo timeout 30 tcpdump -i any -w /tmp/capture.pcap -nn
   
   # В это время генерируй трафик
   ping -c 10 8.8.8.8 &
   curl http://example.com &
   dig google.com
   
   # Анализируй файл
   tcpdump -r /tmp/capture.pcap
   tcpdump -r /tmp/capture.pcap icmp
   tcpdump -r /tmp/capture.pcap port 80
   tcpdump -r /tmp/capture.pcap -nn -c 20
```

5. **Мониторинг активных соединений:**

bash

```bash
   # Посмотри все TCP соединения
   ss -tn
   
   # Только established
   ss -tn state established
   
   # С именами процессов
   sudo ss -tnp state established
   
   # Listening порты
   sudo ss -tlnp
   
   # Статистика
   ss -s
```

6. **Анализ трафика по приложениям:**

bash

```bash
   # Установи nethogs
   sudo apt install nethogs -y
   
   # Запусти мониторинг (Ctrl+C для выхода)
   sudo nethogs eth0
   
   # В другом терминале генерируй трафик
   curl -O http://releases.ubuntu.com/20.04/ubuntu-20.04.6-desktop-amd64.iso
```

### 🚀 Бонус (новое)

**1. Используй bpf фильтры для продвинутого захвата:**

bash

```bash
# BPF (Berkeley Packet Filter) - мощный язык фильтрации

# Захват только SYN пакетов (начало TCP handshake)
sudo tcpdump -i any 'tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0'

# Захват HTTP POST запросов
sudo tcpdump -i any 'tcp dst port 80 and (tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354)'

# Захват фрагментированных пакетов
sudo tcpdump -i any 'ip[6:2] & 0x1fff != 0'

# Захват пакетов с определенным TTL
sudo tcpdump -i any 'ip[8] = 64'

# Захват broadcast/multicast
sudo tcpdump -i any 'ether[0] & 1 != 0'
```

**2. Анализ производительности с iperf3:**

bash

```bash
# Установка
sudo apt install iperf3

# На сервере
iperf3 -s

# На клиенте (тест TCP)
iperf3 -c server-ip

# UDP тест
iperf3 -c server-ip -u -b 100M

# Параллельные потоки
iperf3 -c server-ip -P 4

# Reverse mode (сервер отправляет)
iperf3 -c server-ip -R

# Тест на определенное время
iperf3 -c server-ip -t 30

# JSON вывод
iperf3 -c server-ip -J
```

**3. Анализ SSL/TLS трафика:**

bash

```bash
# Захват TLS handshake
sudo tcpdump -i any 'tcp port 443' -nn -v

# Анализ сертификата сайта
openssl s_client -connect example.com:443 -showcerts

# Проверка TLS версий
nmap --script ssl-enum-ciphers -p 443 example.com

# Мониторинг TLS соединений
sudo tshark -i any -Y "ssl.handshake.type == 1"
```

**4. Создай dashboard для мониторинга сети:**

bash

```bash
# Установи необходимые инструменты
sudo apt install vnstat nethogs iftop

# Создай скрипт для сбора метрик
cat > /tmp/network_monitor.sh << 'EOF'
#!/bin/bash
while true; do
    clear
    echo "=== Network Monitor ==="
    echo
    echo "=== Interface Statistics ==="
    ip -s -s link show eth0 | grep -A 5 "eth0:"
    echo
    echo "=== Active Connections ==="
    ss -tn state established | wc -l
    echo
    echo "=== Top Bandwidth Users ==="
    sudo timeout 5 nethogs eth0 -t 2>&1 | tail -10
    echo
    echo "=== Recent Traffic ==="
    vnstat -i eth0 --oneline
    sleep 10
done
EOF

chmod +x /tmp/network_monitor.sh
/tmp/network_monitor.sh
```

---

## Модуль 7: Практические сценарии (40 минут)

### 🎯 Реальные задачи DevOps/SysAdmin

**Сценарий 1: Диагностика проблем с подключением**

bash

```bash
# Проблема: Пользователи не могут подключиться к веб-серверу

# Шаг 1: Проверка локальной сети
ip addr show                              # IP адрес назначен?
ip route show                             # Есть default gateway?
ping -c 3 $(ip route | grep default | awk '{print $3}')  # Gateway доступен?

# Шаг 2: Проверка DNS
cat /etc/resolv.conf                      # DNS серверы настроены?
dig google.com +short                     # DNS работает?
dig @8.8.8.8 yourserver.com +short        # Внешний DNS видит сервер?

# Шаг 3: Проверка доступности сервера
ping -c 3 yourserver.com                  # ICMP проходит?
traceroute yourserver.com                 # Где обрывается путь?

# Шаг 4: Проверка портов
telnet yourserver.com 80                  # Порт открыт?
nc -zv yourserver.com 80                  # Альтернатива
nmap -p 80,443 yourserver.com             # Сканирование портов

# Шаг 5: Проверка на самом сервере
sudo ss -tlnp | grep :80                  # Веб-сервер слушает?
sudo systemctl status nginx               # Сервис запущен?
sudo tail -f /var/log/nginx/error.log     # Ошибки в логах?

# Шаг 6: Проверка firewall
sudo iptables -L -n -v                    # Правила firewall
sudo ufw status                           # Если используется ufw

# Шаг 7: Проверка SELinux/AppArmor (если есть)
getenforce                                # SELinux режим
sudo aa-status                            # AppArmor статус

# Шаг 8: Тест с сервера
curl -I http://localhost                  # Локально работает?
curl -I http://$(hostname -I | awk '{print $1}')  # По IP работает?

# Диагностический отчет
cat << EOF
=== Network Diagnostic Report ===
IP Address: $(ip -4 addr show | grep inet | grep -v 127.0.0.1 | awk '{print $2}')
Gateway: $(ip route | grep default | awk '{print $3}')
DNS Servers: $(cat /etc/resolv.conf | grep nameserver | awk '{print $2}')
Listening Ports: $(sudo ss -tlnp | grep LISTEN | wc -l)
Firewall Status: $(sudo ufw status 2>/dev/null || echo "iptables active")
EOF
```

**Сценарий 2: Настройка reverse proxy с nginx**

bash

```bash
# Задача: Настроить nginx как reverse proxy для backend сервисов

# Установка
sudo apt install nginx

# Конфигурация /etc/nginx/sites-available/reverse-proxy
sudo tee /etc/nginx/sites-available/reverse-proxy << 'EOF'
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
EOF

# Включить конфигурацию
sudo ln -s /etc/nginx/sites-available/reverse-proxy /etc/nginx/sites-enabled/

# Проверка конфигурации
sudo nginx -t

# Перезагрузка
sudo systemctl reload nginx

# Тестирование
curl -I http://localhost
curl http://localhost/health

# Мониторинг логов
sudo tail -f /var/log/nginx/access.log
```

**Сценарий 3: Настройка VPN туннеля с WireGuard**

bash

```bash
# Задача: Создать безопасный туннель между двумя серверами

# Установка WireGuard
sudo apt install wireguard

# На Server 1 (10.0.0.1/24)
# Генерация ключей
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod 600 /etc/wireguard/private.key
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key

# Конфигурация /etc/wireguard/wg0.conf
sudo tee /etc/wireguard/wg0.conf << 'EOF'
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <SERVER1_PRIVATE_KEY>

# Server 2 peer
[Peer]
PublicKey = <SERVER2_PUBLIC_KEY>
AllowedIPs = 10.0.0.2/32
EOF

# На Server 2 (10.0.0.2/24)
# Аналогично генерируем ключи
# Конфигурация /etc/wireguard/wg0.conf
sudo tee /etc/wireguard/wg0.conf << 'EOF'
[Interface]
Address = 10.0.0.2/24
PrivateKey = <SERVER2_PRIVATE_KEY>

[Peer]
PublicKey = <SERVER1_PUBLIC_KEY>
Endpoint = server1-ip:51820
AllowedIPs = 10.0.0.1/32
PersistentKeepalive = 25
EOF

# Запуск на обоих серверах
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0

# Проверка
sudo wg show
ping -c 3 10.0.0.2  # С Server 1
ping -c 3 10.0.0.1  # С Server 2

# Роутинг трафика через VPN
sudo ip route add 192.168.2.0/24 via 10.0.0.2 dev wg0  # На Server 1
```

**Сценарий 4: Защита от DDoS с rate limiting**

bash

```bash
# Задача: Защитить веб-сервер от простых DDoS атак

# Метод 1: iptables rate limiting
# Ограничение новых подключений (80 в минуту)
sudo iptables -A INPUT -p tcp --dport 80 -m state --state NEW -m recent --set
sudo iptables -A INPUT -p tcp --dport 80 -m state --state NEW -m recent --update --seconds 60 --hitcount 80 -j DROP

# Защита от SYN flood
sudo iptables -N syn_flood
sudo iptables -A INPUT -p tcp --syn -j syn_flood
sudo iptables -A syn_flood -m limit --limit 1/s --limit-burst 3 -j RETURN
sudo iptables -A syn_flood -j DROP

# Метод 2: nginx rate limiting
sudo tee /etc/nginx/conf.d/rate-limit.conf << 'EOF'
# Определение зоны rate limit
limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=api:10m rate=5r/s;

# Connection limit
limit_conn_zone $binary_remote_addr zone=addr:10m;

server {
    listen 80;
    
    # Общий rate limit
    limit_req zone=general burst=20 nodelay;
    limit_conn addr 10;
    
    location /api/ {
        # Строже для API
        limit_req zone=api burst=5;
    }
}
EOF

# Метод 3: fail2ban
sudo apt install fail2ban

# Конфигурация /etc/fail2ban/jail.local
sudo tee /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5

[nginx-limit-req]
enabled = true
filter = nginx-limit-req
logpath = /var/log/nginx/error.log

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
logpath = /var/log/nginx/error.log
maxretry = 3
EOF

sudo systemctl restart fail2ban

# Проверка
sudo fail2ban-client status
sudo fail2ban-client status nginx-limit-req
```

**Сценарий 5: Мониторинг сетевых проблем**

bash

```bash
# Задача: Автоматический мониторинг и алертинг

# Скрипт мониторинга connectivity
cat > /usr/local/bin/network-monitor.sh << 'EOF'
#!/bin/bash

LOGFILE="/var/log/network-monitor.log"
ALERT_EMAIL="admin@example.com"

check_connectivity() {
    local host=$1
    local service=$2
    
    if ! ping -c 3 -W 5 "$host" > /dev/null 2>&1; then
        echo "$(date) - ALERT: Cannot reach $host ($service)" | tee -a "$LOGFILE"
        # Отправка уведомления
        echo "Cannot reach $host ($service)" | mail -s "Network Alert" "$ALERT_EMAIL"
        return 1
    fi
    return 0
}

check_port() {
    local host=$1
    local port=$2
    local service=$3
    
    if ! nc -z -w5 "$host" "$port" 2>/dev/null; then
        echo "$(date) - ALERT: Port $port on $host ($service) is not responding" | tee -a "$LOGFILE"
        echo "Port $port on $host ($service) is not responding" | mail -s "Network Alert" "$ALERT_EMAIL"
        return 1
    fi
    return 0
}

check_latency() {
    local host=$1
    local threshold=$2
    
    latency=$(ping -c 3 "$host" | tail -1 | awk -F '/' '{print $5}')
    
    if (( $(echo "$latency > $threshold" | bc -l) )); then
        echo "$(date) - WARNING: High latency to $host: ${latency}ms" | tee -a "$LOGFILE"
    fi
}

# Мониторинг критичных сервисов
check_connectivity "8.8.8.8" "Internet"
check_connectivity "192.168.1.1" "Gateway"
check_port "192.168.1.10" 80 "Web Server"
check_port "192.168.1.11" 3306 "Database"
check_latency "8.8.8.8" 100

# Проверка DNS
if ! dig google.com +short > /dev/null 2>&1; then
    echo "$(date) - ALERT: DNS resolution failed" | tee -a "$LOGFILE"
fi

# Проверка использования bandwidth
RX_BEFORE=$(cat /sys/class/net/eth0/statistics/rx_bytes)
sleep 1
RX_AFTER=$(cat /sys/class/net/eth0/statistics/rx_bytes)
RX_RATE=$((($RX_AFTER - $RX_BEFORE) * 8 / 1000000))  # Mbps

if [ $RX_RATE -gt 800 ]; then
    echo "$(date) - WARNING: High bandwidth usage: ${RX_RATE}Mbps" | tee -a "$LOGFILE"
fi
EOF

chmod +x /usr/local/bin/network-monitor.sh

# Добавить в cron (каждые 5 минут)
(crontab -l 2>/dev/null; echo "*/5 * * * * /usr/local/bin/network-monitor.sh") | crontab -

# Или с systemd timer
cat > /etc/systemd/system/network-monitor.service << 'EOF'
[Unit]
Description=Network Monitor
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/network-monitor.sh
EOF

cat > /etc/systemd/system/network-monitor.timer << 'EOF'
[Unit]
Description=Run Network Monitor every 5 minutes

[Timer]
OnBootSec=5min
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable network-monitor.timer
sudo systemctl start network-monitor.timer
```

**Сценарий 6: Troubleshooting медленной сети**

bash

```bash
# Задача: Диагностика и устранение проблем с производительностью

# Шаг 1: Проверка физического уровня
ethtool eth0                              # Скорость линка, дуплекс
ethtool -S eth0                           # Ошибки и дропы
ip -s link show eth0                      # Статистика интерфейса

# Шаг 2: Проверка TCP параметров
sysctl net.ipv4.tcp_rmem                  # Буферы на чтение
sysctl net.ipv4.tcp_wmem                  # Буферы на запись
sysctl net.core.rmem_max                  # Максимальный размер буфера
sysctl net.core.wmem_max

# Оптимизация (если нужно)
cat >> /etc/sysctl.conf << 'EOF'
# Увеличение TCP буферов
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864

# TCP congestion control
net.ipv4.tcp_congestion_control = bbr

# Increase netdev backlog
net.core.netdev_max_backlog = 5000
EOF

sudo sysctl -p

# Шаг 3: Проверка активных соединений
ss -tin | grep -i ESTAB | wc -l          # Количество established соединений
ss -s                                     # Общая статистика

# Шаг 4: Поиск проблемных соединений
# Соединения в TIME-WAIT
ss -tan | grep TIME-WAIT | wc -l

# Top connections по IP
ss -tn | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head

# Шаг 5: Тест пропускной способности
# С iperf3 (нужен сервер на другом конце)
iperf3 -c remote-server -t 30

# Локальный тест
iperf3 -s &                               # Запустить сервер
iperf3 -c 127.0.0.1 -t 10                 # Клиент
pkill iperf3

# Шаг 6: Проверка MTU и фрагментации
ping -M do -s 1472 8.8.8.8                # Проверка Path MTU
tracepath 8.8.8.8                         # Определение MTU на пути

# Шаг 7: Анализ трафика
# Поиск ретрансмиссий
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-rst != 0' -c 20

# Шаг 8: CPU и контекстные переключения
mpstat 1 10                               # CPU usage
vmstat 1 10                               # Context switches

# Шаг 9: Проверка DNS latency
time dig google.com +short
time dig @8.8.8.8 google.com +short

# Комплексный отчет
cat << EOF
=== Network Performance Report ===
Interface: $(ip route | grep default | awk '{print $5}')
Link Speed: $(ethtool eth0 | grep Speed)
Active Connections: $(ss -tn | grep ESTAB | wc -l)
TIME-WAIT: $(ss -tn | grep TIME-WAIT | wc -l)
Errors: $(ip -s link show eth0 | grep errors | awk '{print $3}')
MTU: $(ip link show eth0 | grep mtu | awk '{print $5}')
EOF
```

**Сценарий 7: Настройка Load Balancing с HAProxy**

bash

```bash
# Задача: Балансировка нагрузки между несколькими backend серверами

# Установка
sudo apt install haproxy

# Конфигурация /etc/haproxy/haproxy.cfg
sudo tee /etc/haproxy/haproxy.cfg << 'EOF'
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000

# Stats page
listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 30s
    stats admin if TRUE

# Frontend
frontend http_front
    bind *:80
    default_backend http_back

# Backend with health checks
backend http_back
    balance roundrobin
    option httpchk GET /health
    http-check expect status 200
    
    server web1 192.168.1.10:8080 check inter 2000 rise 2 fall 3
    server web2 192.168.1.11:8080 check inter 2000 rise 2 fall 3
    server web3 192.168.1.12:8080 check inter 2000 rise 2 fall 3

# SSL termination (if needed)
frontend https_front
    bind *:443 ssl crt /etc/ssl/certs/example.pem
    default_backend http_back
EOF

# Проверка конфигурации
sudo haproxy -c -f /etc/haproxy/haproxy.cfg

# Запуск
sudo systemctl restart haproxy
sudo systemctl enable haproxy

# Проверка
curl http://localhost
curl http://localhost:8404/stats

# Тестирование балансировки
for i in {1..10}; do
    curl -s http://localhost | grep "Server:"
done

# Мониторинг
sudo tail -f /var/log/haproxy.log
```

### 💻 Финальное задание

Комплексная задача: Настрой полную сетевую инфраструктуру

bash

```bash
# Сценарий:
# - Linux gateway с NAT для внутренней сети
# - Локальный DNS кэш
# - Reverse proxy для веб-сервисов
# - Firewall с rate limiting
# - Мониторинг

# 1. Настройка сетевых интерфейсов
# eth0: WAN (интернет) - получает IP по DHCP
# eth1: LAN (внутренняя сеть) - 192.168.100.1/24

sudo ip addr add 192.168.100.1/24 dev eth1
sudo ip link set eth1 up

# 2. Включение IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

# 3. Настройка NAT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# 4. Firewall правила
# Базовая защита
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT

# Разрешить SSH (с rate limiting)
sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Разрешить HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Разрешить DNS для локальной сети
sudo iptables -A INPUT -i eth1 -p udp --dport 53 -j ACCEPT

# Блокировать остальное
sudo iptables -A INPUT -j DROP

# Сохранить правила
sudo iptables-save | sudo tee /etc/iptables/rules.v4

# 5. Настройка DNS кэша (dnsmasq)
sudo apt install dnsmasq
sudo tee /etc/dnsmasq.d/local.conf << 'EOF'
server=1.1.1.1
server=8.8.8.8
cache-size=10000
listen-address=127.0.0.1,192.168.100.1
bind-interfaces
EOF

sudo systemctl restart dnsmasq

# 6. Настройка DHCP для LAN (опционально)
sudo tee -a /etc/dnsmasq.d/local.conf << 'EOF'
dhcp-range=192.168.100.50,192.168.100.150,12h
dhcp-option=3,192.168.100.1
dhcp-option=6,192.168.100.1
EOF

sudo systemctl restart dnsmasq

# 7. Настройка мониторинга
cat > /usr/local/bin/gateway-monitor.sh << 'EOF'
#!/bin/bash
LOGFILE="/var/log/gateway-monitor.log"

# Проверка connectivity
ping -c 1 8.8.8.8 > /dev/null 2>&1 || echo "$(date) - Internet down" >> "$LOGFILE"

# Проверка NAT
CONNTRACK=$(sudo conntrack -L 2>/dev/null | wc -l)
echo "$(date) - Active connections: $CONNTRACK" >> "$LOGFILE"

# Проверка firewall
FW_DROPPED=$(sudo iptables -L INPUT -v -n | grep DROP | awk '{sum+=$1} END {print sum}')
echo "$(date) - Dropped packets: $FW_DROPPED" >> "$LOGFILE"

# Bandwidth monitoring
RX_BYTES=$(cat /sys/class/net/eth0/statistics/rx_bytes)
TX_BYTES=$(cat /sys/class/net/eth0/statistics/tx_bytes)
echo "$(date) - RX: $RX_BYTES TX: $TX_BYTES" >> "$LOGFILE"
EOF

chmod +x /usr/local/bin/gateway-monitor.sh
(crontab -l 2>/dev/null; echo "*/5 * * * * /usr/local/bin/gateway-monitor.sh") | crontab -

# 8. Тестирование
# На клиенте в LAN настрой:
# IP: 192.168.100.50/24
# Gateway: 192.168.100.1
# DNS: 192.168.100.1

# Проверки:
ping -c 3 192.168.100.1              # Gateway доступен
ping -c 3 8.8.8.8                    # Интернет через NAT
dig google.com                       # DNS через кэш
curl http://example.com              # HTTP работает

# На gateway проверь:
sudo conntrack -L | grep ESTABLISHED  # NAT соединения
sudo tail -f /var/log/dnsmasq.log    # DNS запросы
sudo iptables -L -v -n               # Firewall статистика
```

### 🎓  Чему вы научились

После завершения этого модуля вы умеете:

1. ✅ Диагностировать сложные сетевые проблемы
2. ✅ Настраивать reverse proxy и load balancing
3. ✅ Защищать инфраструктуру от DDoS атак
4. ✅ Создавать VPN туннели
5. ✅ Мониторить сетевую инфраструктуру
6. ✅ Оптимизировать производительность сети
7. ✅ Настраивать комплексные сетевые решения

### 📚 Дополнительные ресурсы

**Документация:**

- Linux Network Administrators Guide: [https://www.tldp.org/LDP/nag2/](https://www.tldp.org/LDP/nag2/)
- Red Hat Networking Guide: [https://access.redhat.com/documentation/](https://access.redhat.com/documentation/)
- Ubuntu Server Guide: [https://ubuntu.com/server/docs](https://ubuntu.com/server/docs)

**Инструменты:**

- Wireshark Documentation: [https://www.wireshark.org/docs/](https://www.wireshark.org/docs/)
- HAProxy Documentation: [http://www.haproxy.org/](http://www.haproxy.org/)
- iptables Tutorial: [https://www.frozentux.net/iptables-tutorial/](https://www.frozentux.net/iptables-tutorial/)

**Практика:**

- Network Labs: [https://labs.networkreliability.engineering/](https://labs.networkreliability.engineering/)
- Cisco Packet Tracer (для понимания основ)
- GNS3 (для эмуляции сетей)

---

## Заключение

Поздравляю! Вы завершили практический мини-курс по сетям для DevOps/SysAdmin.

**Основные навыки, которые вы освоили:**

- 🔍 Диагностика сетевых проблем
- 🔧 Настройка и оптимизация сетей
- 🛡️ Защита инфраструктуры
- 📊 Мониторинг и анализ трафика
- 🚀 Автоматизация сетевых задач

**Следующие шаги:**

1. Практикуйтесь на тестовых окружениях
2. Изучите специфичные для вашего стека технологии (Kubernetes networking, Cloud networking)
3. Автоматизируйте рутинные задачи с помощью скриптов
4. Следите за новыми технологиями (eBPF, Cilium, WireGuard)

**Помните:** Сети - это основа любой инфраструктуры. Понимание сетевых принципов делает вас более эффективным DevOps/SysAdmin специалистом!
