# Домашнее задание: Мониторинг с Zabbix
**Студент:** Кошелев Дмитрий Владимирович 
**Дата выполнения:** 25.07.2026
**Номер занятия:** 8-03-hw

---

## Задание 1
### Создание шаблона с мониторингом CPU и RAM

#### Процесс выполнения:

1. **Вход в веб-интерфейс Zabbix Server**
   - Открыл браузер и перешел по адресу: `http://192.168.126.132:8080/zabbix`
   - Ввел логин и пароль администратора

2. **Создание нового шаблона**
   - Перешел в раздел **Данные → Шаблоны** (Configuration → Templates)
   - Нажал кнопку **Создать шаблон** (Create template)
   - Заполнил поля:
     - **Имя шаблона:** `Custom CPU RAM Template`
     - **Видимое имя:** `Custom CPU RAM Template`
     - **Группы:** `Templates/Operating systems`
   - Нажал **Добавить** (Add)

3. **Создание Item для загрузки CPU в процентах**
   - В созданном шаблоне перешел на вкладку **Элементы данных** (Items)
   - Нажал **Создать элемент данных** (Create item)
   - Заполнил поля:

| Поле | Значение |
|------|----------|
| **Имя** | `Загрузка CPU в процентах` |
| **Тип** | `Zabbix агент` |
| **Ключ** | `system.cpu.util` |
| **Тип информации** | `Числовое (с плавающей запятой)` |
| **Единицы измерения** | `%` |
| **Интервал обновления** | `30s` |
| **Период хранения** | `31d` |
| **Динамика изменений** | `365d` |
| **Приложения** | `CPU` |

   - Нажал **Добавить**

4. **Создание Item для загрузки RAM в процентах**
   - Нажал **Создать элемент данных** (Create item)
   - Заполнил поля:

| Поле | Значение |
|------|----------|
| **Имя** | `Использование RAM в процентах` |
| **Тип** | `Zabbix агент` |
| **Ключ** | `vm.memory.size[pused]` |
| **Тип информации** | `Числовое (с плавающей запятой)` |
| **Единицы измерения** | `%` |
| **Интервал обновления** | `30s` |
| **Период хранения** | `31d` |
| **Динамика изменений** | `365d` |
| **Приложения** | `Memory` |

   - Нажал **Добавить**

5. **Создание триггеров для шаблона**
   - В шаблоне перешел на вкладку **Триггеры** (Triggers)
   - Нажал **Создать триггер** (Create trigger)

**Триггер для CPU:**

| Поле | Значение |
|------|----------|
| **Имя** | `Высокая загрузка CPU > 80%` |
| **Выражение** | `last(/Custom CPU RAM Template/system.cpu.util)>80` |
| **Степень серьезности** | `Средняя` |
| **Включено** | ✅ |

**Триггер для RAM:**

| Поле | Значение |
|------|----------|
| **Имя** | `Высокое использование RAM > 90%` |
| **Выражение** | `last(/Custom CPU RAM Template/vm.memory.size[pused])>90` |
| **Степень серьезности** | `Средняя` |
| **Включено** | ✅ |

#### Результат:
в папке images
Скриншот: Задание 1

---

## Задание 2-3
### Добавление двух хостов и привязка шаблонов

#### Процесс выполнения:

1. **Настройка Zabbix Agent на первой ВМ (Zabbix Server)**

   Отредактировал конфигурационный файл агента:
   ```bash
   sudo nano /etc/zabbix/zabbix_agentd.conf
   

### Добавление двух хостов и привязка шаблонов

#### Процесс выполнения:

1. **Настройка Zabbix Agent на первой ВМ (Zabbix Server)**

   Отредактировал конфигурационный файл агента:
   ```bash
   sudo nano /etc/zabbix/zabbix_agentd.conf
   
   Server=127.0.0.1
ServerActive=127.0.0.1
Hostname=Zabbix server

Перезапустил агент:

sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
sudo systemctl status zabbix-agent
Настройка Zabbix Agent на второй ВМ (AstraLinux)

Установил агент:

sudo apt update
sudo apt install zabbix-agent -y
Отредактировал конфигурацию:

sudo nano /etc/zabbix/zabbix_agentd.conf
Установил параметры:

ini
Server=192.168.126.128
ServerActive=192.168.126.128
Hostname=KoshelevDV-2
Перезапустил агент:

sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
sudo systemctl status zabbix-agent
Проверка IP-адреса второй ВМ:

zabbix@zabbix:~$ ip a
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:4a:0a:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.126.134/24 brd 192.168.126.255 scope global dynamic noprefixroute eth0
       valid_lft 1399sec preferred_lft 1399sec
IP-адрес второй ВМ: 192.168.126.134

Проверка работы агентов

На Zabbix Server выполнил проверку:

# Проверка первого агента (Zabbix Server)
zabbix_get -s 127.0.0.1 -k agent.ping
# Результат: 1

# Проверка второго агента
zabbix_get -s 192.168.126.134 -k agent.ping
# Результат: 1

# Проверка получения данных CPU с первого агента
zabbix_get -s 127.0.0.1 -k system.cpu.util
# Результат: число (например, 15.5)

# Проверка получения данных CPU со второго агента
zabbix_get -s 192.168.126.134 -k system.cpu.util
# Результат: число (например, 12.3)

# Проверка получения данных RAM с первого агента
zabbix_get -s 127.0.0.1 -k vm.memory.size[pused]
# Результат: число (например, 45.2)

# Проверка получения данных RAM со второго агента
zabbix_get -s 192.168.126.134 -k vm.memory.size[pused]
# Результат: число (например, 38.7)
Решение проблемы с дублированием ключей

При добавлении второго хоста возникла ошибка:

Не удалось унаследовать элементы данных с ключом "system.cpu.util" с обоих шаблонов 
"Custom CPU RAM Template" и "Linux by Zabbix agent", так как ключ должен быть уникален 
на узле сети "KoshelevDV-2".
Решение:

Удалил дублирующиеся элементы данных из шаблона Custom CPU RAM Template:

Загрузка CPU в процентах (ключ: system.cpu.util)

Использование RAM в процентах (ключ: vm.memory.size[pused])

Оставил в шаблоне только триггеры, которые ссылаются на элементы из шаблона Linux by Zabbix agent

Обновил выражения триггеров:

Триггер CPU:

last(/Linux by Zabbix agent/system.cpu.util)>80
Триггер RAM:

last(/Linux by Zabbix agent/vm.memory.size[pused])>90
Добавление первого хоста в Zabbix

Перешел в раздел Данные → Узлы сети (Configuration → Hosts)

Нажал Создать узел сети (Create host)
Заполнил вкладку "Узел сети":

Поле	Значение
Имя узла сети	Zabbix server
Видимое имя	Zabbix server
Группы	Linux servers
Интерфейсы	Тип: Zabbix agent, IP: 127.0.0.1, Порт: 10050
Перешел на вкладку "Шаблоны":

Добавил шаблоны:
Linux by Zabbix agent
Custom CPU RAM Template
Нажал Добавить
Добавление второго хоста
Повторил те же шаги для второго хоста:

Поле	Значение
Имя узла сети	KoshelevDV-2
Видимое имя	KoshelevDV-2
Группы	Linux servers
Интерфейсы	Тип: Zabbix agent, IP: 192.168.126.134, Порт: 10050
Шаблоны	Linux by Zabbix agent, Custom CPU RAM Template
Нажал Добавить

Проверка статуса хостов

Перешел в Данные → Узлы сети

Проверил статус обоих хостов:

Zabbix server — статус Z (зеленый) и A (зеленый)

KoshelevDV-2 — статус Z (зеленый) и A (зеленый)

Проверка поступления данных

Перешел в Мониторинг → Последние данные (Monitoring → Latest Data)

Выбрал хост Zabbix server и проверил наличие данных:

CPU idle time

CPU system time

CPU user time

Memory utilization

Выбрал хост KoshelevDV-2 и проверил наличие данных:
CPU idle time
CPU system time
CPU user time
Memory utilization
Проверка работы триггеров

Для проверки срабатывания триггеров создал нагрузку на CPU с помощью stress-ng:


# Установка stress-ng (в Astra Linux stress недоступен)
sudo apt update
sudo apt install stress-ng -y

# Создание нагрузки на CPU на 60 секунд
sudo stress-ng --cpu 4 --timeout 60
После создания нагрузки перешел в Мониторинг → Проблемы и убедился, что триггеры сработали.

#### Результат:
в папке images
Скриншот: Задание 2-3

## Задание 4
### Создание кастомного дашборда

#### Процесс выполнения:

1. **Создание нового дашборда**

   - Перешел в раздел **Панели** (Dashboards) в веб-интерфейсе Zabbix
   - Нажал кнопку **Создать панель** (Create dashboard)
   - В поле **Имя** ввел: `Мониторинг системы`
   - Нажал **Добавить**

2. **Переход в режим редактирования**

   - На созданной панели нажал кнопку **Править** (Edit)
   - Дашборд перешел в режим редактирования

3. **Добавление виджетов на панель**

   **Виджет 1,2: Общая загрузка CPU**

   | Параметр | Значение |
   |----------|----------|
   | **Тип виджета** | `График` (Graph) |
   | **Имя** | `Общая загрузка CPU` |
   | **Ширина** | `2` столбца |
   | **Высота** | `300` пикселей |
   | **Источники данных** | `Zabbix server` → `Linux: CPU utilization`, `KoshelevDV-2` → `Linux: CPU utilization` |

   **Виджет 3,4: Использование RAM**

   | Параметр | Значение |
   |----------|----------|
   | **Тип виджета** | `График` (Graph) |
   | **Имя** | `Использование RAM` |
   | **Ширина** | `2` столбца |
   | **Высота** | `300` пикселей |
   | **Источники данных** | `Zabbix server` → `Linux: Memory utilization`, `KoshelevDV-2` → `Linux: Memory utilization` |
   
#### Результат:
в папке images
Скриншот: Задание 4



==========================
#### Результат заданий:
в папке images
Скриншот: Задание 1
Скриншот: Задание 2-3
Скриншот: Задание 4


![Задание 1](https://github.com/rooot-root/Zabbix_NETODOL2/blob/master/task1.png);
![Задание 2_3](https://github.com/rooot-root/Zabbix_NETODOL2/blob/master/task2_3.png);
![Задание 4](https://github.com/rooot-root/Zabbix_NETODOL2/blob/master/task4.png);





