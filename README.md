# 21 задача из опыта DevOps-инженера

https://tproger.ru/articles/21-zadacha-iz-opyta-devops-inzhenera

**Задача 0:**
Написать Ansible-плейбук, для конфигурации веб-сервера Nginx, который будет настраивать также и firewall-правила, открыв 22, 80, 443 порты.

```
---
- name: Configure nginx and firewall rules
  hosts: webservers
  become: yes

  tasks:
  - name: Install nginx
    apt:
        name: nginx
        state: present
        update_cache: yes

  - name: Ensure nginx is enabled and started
    systemd:
	name: nginx
	enabled: yes
	state: started

  - name: Configure firewall rules (UFW)
    ufw:
	rule: allow
	port: "{{ item }}"
	proto: tcp
    with_items:
	- 22
	- 80
	- 443

   - name: Enable UFW firewall
     ufw:
	state: enabled
	policy: deny

   - name: Ensure firewall rules are active
     shell: ufw status verbose
     register: ufw_status

   - debug:
	msg: "Firewall status: {{ ufw_status.stdout }}
```

```
ansible-playbook nginx_firewall.yml -i inventory_file
```

**Задача 1:**
Описать конфигурацию Nginx, где он будет выполнять роль балансировщика c использованием upstream.

/etc/nginx/nginx.conf

```
http {
upstream backend {
	server server1:8080;
	server server2:8080;
}

server{
	location / {
		proxy_pass http://backend;
	}
}
}


```

```
sudo nginx -t

sudo systemctl reload nginx
```

**Upstream**
В Nginx директива upstream используется для определения группы серверов, к которым Nginx будет перенаправлять запросы. Она поддерживает несколько видов конфигураций, в зависимости от требуемого поведения балансировки нагрузки.
Вот основные виды конфигураций upstream в Nginx:

1. Обычная балансировка нагрузки
   Это самый простой и распространенный способ. Запросы распределяются между серверами в равных долях.

```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
}
```

2. Пул с активным и резервным сервером
   Вы можете указать один из серверов как резервный. Если основной сервер становится недоступным, запросы будут перенаправляться на резервный сервер.

```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com backup;
}
```

3. Сессионная привязка (sticky sessions)
   Для некоторых приложений может потребоваться, чтобы все запросы от одного клиента направлялись на один и тот же сервер. Nginx не поддерживает сессионную привязку "из коробки", но это можно реализовать с помощью модуля sticky (не включен в стандартную сборку, но доступен в некоторых дистрибутивах).

```nginx
upstream backend {
    sticky;
    server backend1.example.com;
    server backend2.example.com;
}
```

4. Методы балансировки
   Nginx поддерживает различные алгоритмы балансировки нагрузки. По умолчанию используется метод Round Robin. Однако вы можете использовать другие методы:

Least Connections: Запрос будет отправлен на сервер с наименьшим количеством активных соединений.

```nginx
upstream backend {
    least_conn;
    server backend1.example.com;
    server backend2.example.com;
}
```

IP Hash: Запросы от одного и того же IP-адреса будут всегда направляться на один и тот же сервер.

```nginx
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
}
```

5. Состояние серверов
   Вы можете управлять состоянием серверов в пуле. Например, сервер может быть временно отключен:

```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com down;  # Сервер отключен
}
```

6. Серверы с различными весами
   Вы можете задать "вес" для каждого сервера, что позволяет контролировать, как часто будут получать запросы каждый сервер. Сервер с большим весом будет получать больше запросов.

```nginx
upstream backend {
    server backend1.example.com weight=3;  # Получает 3 раза больше запросов
    server backend2.example.com weight=1;
}
```

7. Серверы с тайм-аутами и другими параметрами
   Вы можете настраивать различные параметры для каждого сервера, такие как тайм-ауты, проверки состояния и т.д.

```nginx
upstream backend {
    server backend1.example.com max_fails=3 fail_timeout=30s;
    server backend2.example.com max_fails=5 fail_timeout=60s;
}
```

**Задача 2:**
Собрать docker image, который включает в себя приложение и веб-сервер Nginx, при запуске открыть порт 80 и добавить контейнер в автостарт системы.

1. Структура проекта
   Создайте структуру папок для вашего проекта:

```
myapp/
│
├── Dockerfile
├── nginx.conf
└── index.html
```

2. Создание файлов index.html
   Создайте файл index.html в папке myapp с простым HTML-контентом:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My App</title>
</head>
<body>
    <h1>Hello, World!</h1>
</body>
</html>
```

- Создайте файл nginx.conf в папке myapp для настройки Nginx:

```
server {
    listen 80;

    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }

    error_page  404 /404.html;
    location = /404.html {
        internal;
    }
}
```

- Создайте файл Dockerfile в папке myapp со следующим содержимым:

```
# Используем официальный образ Nginx
FROM nginx:alpine

# Копируем конфигурацию Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Копируем HTML файл
COPY index.html /usr/share/nginx/html/

# Открываем порт 80
EXPOSE 80

# Команда по умолчанию для запуска Nginx
CMD ["nginx", "-g", "daemon off;"]
```

3. Сборка Docker Image
   Перейдите в директорию myapp и выполните команду для сборки Docker image:

```
docker build -t myapp:latest .
```

4. Запуск контейнера
   Теперь запустите контейнер, открыв порт 80 и добавив его в автозапуск:

```
docker run -d --name myapp_container -p 80:80 --restart always myapp:latest
```

**Задача 4:**
Написать bash скрипт, который будет создавать PostgreSQL бэкап. Для автоматического запуска скрипта создаём запись в Cron.

```
#!/bin/bash

# Configs
DB_NAME="postgres"
DB_USER="postgres"
DB_HOST="localhost"
BACKUP_DIR="/etc/backups/postgres"
RETENTION_DAYS=7 # Количество дней для хранения резервных копий

# Получение текущей даты
DATE=$(date +"%Y-%m-%d")

# Имя файла резервной копии
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_backup_$DATE.sql"

# Создание резервной копии
echo "Создание резервной копии базы данных $DB_NAME..."
pg_dump -U $DB_USER -h $DB_HOST $DB_NAME > $BACKUP_FILE

# Проверка успешности создания резервной копии
if [ $? -eq 0 ]; then
  echo "Резервная копия успешно создана: $BACKUP_FILE"
else
  echo "Ошибка при создании резервной копии!"
  exit 1
fi

# Удаление старых резервных копий
echo "Удаление старых резервных копий старше $RETENTION_DAYS дней..."
find $BACKUP_DIR -type f -name "${DB_NAME}_backup_*.sql" -mtime +$RETENTION_DAYS -exec rm {} \;

echo "Задача завершена."
```

Сделайте скрипт исполняемым:

```bash
chmod +x /path/to/your/backup_script.sh
```

Откройте Crontab для редактирования:

```bash
crontab -e
```

Добавьте запись для ежедневного запуска скрипта (например, в 2:00 ночи):

```bash
0 2 * * * /path/to/your/backup_script.sh >> /path/to/your/backup.log 2>&1
```

**Задача 5:**
Сделать так, чтоб весь вывод скрипта из предыдущей задачи записывался в лог-файл /var/log/backup/backup.log.
Формат лога: [Название скрипта][Время выполнения] — полный вывод.

```
#!/bin/bash

# Configs
DB_NAME="postgres"
DB_USER="postgres"
DB_HOST="localhost"
BACKUP_DIR="/etc/backups/postgres"
RETENTION_DAYS=7 # Количество дней для хранения резервных копий
LOG_FILE="/var/log/backup/backup.log"  # Путь к лог-файлу
SCRIPT_NAME="backup_script.sh"  # Название скрипта

# Создаем директорию для логов, если она не существует
mkdir -p /var/log/backup

# Функция для логирования
log() {
    echo "[$SCRIPT_NAME][$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a $LOG_FILE
}

# Начало логирования
log "Начало выполнения скрипта."

# Получение текущей даты
DATE=$(date +"%Y-%m-%d")

# Имя файла резервной копии
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_backup_$DATE.sql"

# Создание резервной копии
log "Создание резервной копии базы данных $DB_NAME..."
pg_dump -U $DB_USER -h $DB_HOST $DB_NAME > $BACKUP_FILE 2>>$LOG_FILE

# Проверка успешности создания резервной копии
if [ $? -eq 0 ]; then
  log "Резервная копия успешно создана: $BACKUP_FILE"
else
  log "Ошибка при создании резервной копии!"
  exit 1
fi

# Удаление старых резервных копий
log "Удаление старых резервных копий старше $RETENTION_DAYS дней..."
find $BACKUP_DIR -type f -name "${DB_NAME}_backup_*.sql" -mtime +$RETENTION_DAYS -exec rm {} \; 2>>$LOG_FILE

log "Задача завершена."
```

**Задача 6:**
Написать скрипт, который будет выполнять проверку состояния диска и, если места меньше чем 85%, то высылать алерт на почту. Для отправки писем прямиком из консоли можно использовать ssmtp клиент.

Установка ssmtp: Убедитесь, что у вас установлен ssmtp.

```
sudo apt-get install ssmtp
```

Настройка ssmtp: Отредактируйте файл конфигурации ssmtp (обычно находится по пути /etc/ssmtp/ssmtp.conf) и добавьте параметры вашей почты:

```
root=your_email@example.com
mailhub=smtp.example.com:587
AuthUser=your_email@example.com
AuthPass=your_password
UseSTARTTLS=YES
```

Bash-скрипт для проверки дискового пространства:

```
#!/bin/bash

# Настройки
THRESHOLD=85 # Порог использования диска в процентах
EMAIL="your_email@example.com"  # Электронная почта для уведомлений

# Проверка использования диска
USAGE=$(df / | grep / | awk '{ print $5 }' | sed 's/%//g')

# Логика отправки уведомления
if [ "$USAGE" -gt "$THRESHOLD" ]; then
    SUBJECT="Alert: Disk Space Usage Critical"
    MESSAGE="Warning! Disk space usage is at ${USAGE}% which exceeds the threshold of ${THRESHOLD}%."

    echo -e "Subject: ${SUBJECT}\n\n${MESSAGE}" | ssmtp $EMAIL
fi
```

Настройка автоматического запуска:
Сделайте скрипт исполняемым:

```
chmod +x /path/to/your/disk_check_script.sh
```

Добавьте запись в Crontab для регулярного выполнения скрипта (например, каждые 30 минут):

```
crontab -e

*/30 * * * * /path/to/your/disk_check_script.sh
```

**Задача 9:**
Настройка мониторинга в связке c Prometheus+Grafana+Node exporter+Alertmanager.

1. Установите Prometheus

- Скачайте и установите Prometheus:

```
wget https://github.com/prometheus/prometheus/releases/download/v2.44.0/prometheus-2.44.0.linux-amd64.tar.gz
tar xvf prometheus-2.44.0.linux-amd64.tar.gz
cd prometheus-2.44.0.linux-amd64
```

- Создайте файл конфигурации prometheus.yml:

```
global:
	scrape_interval: 15s

scrape_configs:
	- job_name: 'node_exporter'
	  static_configs:
		- targets: ['localhost:9100']
```

- Запустите Prometheus:
  ./prometheus --config.file=prometheus.yml

2. Установите Node Exporter

- Скачайте и установите Node Exporter:

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xvf node_exporter-1.6.1.linux-amd64.tar.gz
cd node_exporter-1.6.1.linux-amd64

```

- Запустите Node Exporter:

```
./node_exporter &
```

3. Установите Alertmanager

- Скачайте и установите Alertmanager:

```
wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz
tar xvf alertmanager-0.25.0.linux-amd64.tar.gz
cd alertmanager-0.25.0.linux-amd64
```

- Создайте файл конфигурации alertmanager.yml:

```
global:
	resolve_timeout: 5m

route:
  group_by: ['alertme']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 3h
  receiver: 'email'

receivers:
  - name: 'email'
    email_configs:
      - to: 'your_email@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.example.com:587'
        auth_username: 'your_email@example.com'
        auth_password: 'your_password'
```

- Запустите Alertmanager:

```
./alertmanager --config.file=alertmanager.yml
```

4. Настройка Prometheus для использования Alertmanager

- Обновите файл конфигурации prometheus.yml, добавив Alertmanager:

```
global:
	scrape_interval: 15s

scrape_configs:
	- job_name: 'node_exporter'
	  static_configs:
		- targets: ['localhost:9100']

alerting:
	alertmanagers:
		- static_configs:
			- targets: ['localhost:9093']
```

- Перезапустите Prometheus, чтобы применить изменения:

```
./prometheus --config.file=prometheus.yml
```

5. Настройка Grafana

- Скачайте и установите Grafana:

```
wget https://dl.grafana.com/oss/release/grafana_9.4.7_amd64.deb
sudo dpkg -i grafana_9.4.7_amd64.deb
```

- Запустите Grafana:

```
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

- Откройте Grafana в браузере:
  - Перейдите по адресу http://localhost:3000.
  - Войдите с использованием стандартных учетных данных: логин admin, пароль admin.
- Добавьте источник данных для Prometheus:
  - Перейдите в Configuration > Data Sources > Add data source.
  - Выберите Prometheus и укажите URL: http://localhost:9090.
  - Нажмите Save & Test.

6. Создание дашборда в Grafana

- Создайте новый дашборд:
  - Перейдите в Dashboard > New Dashboard.
  - Добавьте панель и выберите тип графика.
  - Используйте запросы, такие как node_cpu_seconds_total или node_memory_MemAvailable_bytes, чтобы получить данные о состоянии системы.

7. Настройка алертов в Grafana

- Добавьте алерт в панель:
  - Перейдите на панель, которую вы создали.
  - Нажмите на заголовок панели и выберите Edit.
  - Перейдите в раздел Alerts и настройте условие для алерта, например:
    - Условие: WHEN avg() OF query(A, 5m, now) IS ABOVE 85
    - Укажите действие для отправки уведомлений на Email.
  - Сохраните изменения и проверьте работоспособность алертов.

8. Обновите файл конфигурации алертов alertmanager.yml для получения уведомления с почты и телеграма

```
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 3h
  receiver: 'default-receiver'

  routes:
    - match:
        severity: critical
      receiver: 'telegram-receiver'  # For critical alerts
    - match:
        severity: warning
      receiver: 'email-receiver'      # For warning alerts

receivers:
  - name: 'default-receiver'
    email_configs:
      - to: 'default_email@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.example.com:587'
        auth_username: 'your_email@example.com'
        auth_password: 'your_password'

  - name: 'telegram-receiver'
    telegram_configs:
      - chat_id: 'YOUR_CHAT_ID'
        bot_token: 'YOUR_BOT_TOKEN'
        parse_mode: 'Markdown'
        send_resolved: true  # Send notifications when alerts are resolved

  - name: 'email-receiver'
    email_configs:
      - to: 'warning_alerts@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.example.com:587'
        auth_username: 'your_email@example.com'
        auth_password: 'your_password'
```

**Задача 10:**
Описать конфигурацию мониторинг сервера в Ansible. Конфигурация должна включать настройку Prometheus+Grafana+Alertmanager на сервере мониторинга и конфигурацию с открытием портов в firewall правилах на серверах, которые мы хотим мониторить. Для мониторинга инстансов, ставим Node exporter, который выводит метрики на порт 9100 (его мы и открываем на отслеживаемых серверах). В целях безопасности порт 9100 открываем только для мониторинг инстанса.

```
---
- name: Monitor server configuration
  hosts: monitoring_server
  become: yes

  tasks:
    - name: Install required packages
      apt:
       name:
	- prometheus
        - grafana
        - alertmanager
        - ufw
       state: present
      notify:
       - Enable and start services

    - name: Configure Prometheus
      template:
	src: prometheus.yml
        dest: /etc/prometheus/prometheus.yml
      notify:
	- Restart Prometheus

     - name: Allow Prometheus port
       ufw:
         rule: allow
         nameL 'Prometheus'
         port: 9090

     - name: Allow Grafana port
       ufw:
         rule: allow
	 name: 'Grafana'
	 port: 3000

     - name: Enable UFW
       ufw:
	state: enabled

  handlers:
    - name: Enable and start services
      systemd:
	name: "{{ item }}"
        state: started
	enabled: yes
      loop:
	- prometheus
	- grafana-server
	- alertmanager

    - name: Restart Prometheus
      systemd:
	name: prometheus
	state: restarted

- name: Install Node Exporter
  hosts: monitored_servers
  become: yes
  tasks:
    - name: Download Node Exporter
      get_url:
	url: "https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz"
        dest: /tmp/node_exporter.tar.gz

     - name: Extract Node Exporter
       unarchive:
        src: /tmp/node_exporter.tar.gz
        dest: /usr/local/bin/
        remote_src: yes

     - name: Create Node Exporter service
       copy:
	dest: /etc/systemd/system/node_exporter.service
	content: |
	 [Unit]
	 Description=Node Exporter

	 [Service]
	 User=root
	 ExecStart=/usr/local/bin/node-exporter-{{ node_exporter_version }}.linux-amd64/node-exporter

	 [Install]
	 WantedBy=multi-user.target

     - name: Start Node Exporter
       systemd:
	 name: node_exporter
	 state: started
	 enabled: yes

     - name: Open port 9100 for Node Exporter
       ufw:
	rule: allow
	port: 9100
	from_ip: "{{ monitoring_server_ip }}"  # IP адрес сервера мониторинга
```

- Шаблон конфигурации Prometheus

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets:
          - 'MONITORED_SERVER_IP:9100'  # Замените на IP адреса ваших monitored_servers
```

- Сохраните ваш Ansible-плейбук в файл, например monitoring_setup.yml Запустите его с помощью следующей команды:

```
ansible-playbook -i inventory monitoring_setup.yml
```

**Задача 11:**
Настроить Gitlab Ci пайплайн для сборки и деплоя контейнера в окружение. В качестве окружения — для упрощения — это может быть инстанс где-либо, в качестве контейнера может быть что угодно (даже ранее собранный контейнер со статической веб страницей). То есть при коммите, мы запускаем наш пайплайн, собираем docker image, и деплоим его на наш тестовый инстанс, где потенциально его могут опробовать — например, тестировщики.

**Задача 12:**
Поднять Kubernetes cluster из трёх нод. Сделать первичную конфигурацию и задеплоить в него контейнер.

**Задача 13:**
Распарсить JSON данные и сохранить их в CSV. Сгенерировать JSON можно здесь — Клик.
Комментарий к задаче: куда же без скриптовых задач. Для решения можно использовать Bash, Python или что-то другое. Пока важен именно результат, а уж пути, какими мы к нему придём, можно в последствии оптимизировать.

data.json

```json
[
  {
    "id": 1,
    "name": "John Doe",
    "email": "john.doe@example.com"
  },
  {
    "id": 2,
    "name": "Jane Smith",
    "email": "jane.smith@example.com"
  }
]
```

json_to_csv.py

```python
import json
import csv

# Load JSON data from a file (assuming the file is named 'data.json')
with open('data.json') as json_file:
    data = json.load(json_file)

# Specify the CSV file where the data will be saved
csv_file = 'output.csv'

# Get the keys from the first dictionary in the list as the header for CSV
headers = data[0].keys()

# Write the data to the CSV file
with open(csv_file, mode='w', newline='') as file:
    writer = csv.DictWriter(file, fieldnames=headers)
    writer.writeheader()
    writer.writerows(data)

print(f"Data has been successfully written to {csv_file}")
```

Запустите python script

```bash
python json_to_csv.py
```

Bash

```bash
#!/bin/bash

# Define the input JSON file and output CSV file
json_file="data.json"
csv_file="output.csv"

# Extract the headers from the JSON file
headers=$(jq -r 'keys_unsorted | @csv' <(jq -s '.[0]' "$json_file"))

# Write headers to the CSV file
echo "$headers" > "$csv_file"

# Extract the data and append to the CSV file
jq -r '.[] | [.id, .name, .email] | @csv' "$json_file" >> "$csv_file"

echo "Data has been successfully written to $csv_file"

```

Для запуска скрипта

```
chmod +x json_to_csv.sh
./json_to_csv.sh
```

**Задача 14:**
Описать в Terraform создание в Azure виртуальной машины

1. Установите Terraform
2. Установите Azure CLI и зайдите
   `az login`
3. Установите управление доступом на основе ролей Azure (Azure RBAC) который помогает вам управлять тем, кто имеет доступ к ресурсам Azure, что они могут делать с этими ресурсами и к каким областям у них есть доступ.
   ```
   az ad sp create-for-rbac --name "terraform" --role="Contributor" \
   --scopes="/subscriptions/YOUR_SUBSCRIPTION_ID"
   ```
4. Создание файлов конфигурации Terraform

- Создайте директорию для конфигурации Terraform и перейдите в неё:

  ```
  mkdir my-terraform-azure-setup
  cd my-terraform-azure-setup
  ```

- Создайте main.tf. Это основной конфигурационный файл, где вы определяете ресурсы Azure.

  ```
  # main.tf

  provider "azurerm" {
  features {}
  }

  # Создание группы ресурсов
  resource "azurerm_resource_group" "rg" {
  name     = "myResourceGroup"
  location = "East US"
  }

  # Создание виртуальной сети
  resource "azurerm_virtual_network" "vnet" {
  name                = "myVNet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  }

  # Создание подсети
  resource "azurerm_subnet" "subnet" {
  name                 = "mySubnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
  }

  # Создание публичного IP
  resource "azurerm_public_ip" "pubip" {
  name                = "myPublicIP"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  allocation_method   = "Dynamic"
  }

  # Создание сетевого интерфейса
  resource "azurerm_network_interface" "nic" {
  name                = "myNIC"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  ip_configuration {
      name                          = "internal"
      subnet_id                     = azurerm_subnet.subnet.id
      private_ip_address_allocation = "Dynamic"
      public_ip_address_id          = azurerm_public_ip.pubip.id
  }
  }

  # Создание виртуальной машины
  resource "azurerm_linux_virtual_machine" "vm" {
  name                = "myVM"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  size                = "Standard_DS1_v2"

  admin_username      = "azureuser"
  admin_password      = "P@ssw0rd1234!"

  network_interface_ids = [
      azurerm_network_interface.nic.id,
  ]

  os_disk {
      caching              = "ReadWrite"
      storage_account_type = "Standard_LRS"
  }

  source_image_reference {
      publisher = "Canonical"
      offer     = "UbuntuServer"
      sku       = "18.04-LTS"
      version   = "latest"
  }
  }
  ```

- Создайте variables.tf. В этом файле можно определить переменные для гибкости и повторного использования конфигурации.

  ```
  # variables.tf

  variable "resource_group_name" {
  description = "Название группы ресурсов"
  default     = "myResourceGroup"
  }

  variable "location" {
  description = "Регион Azure для развертывания ресурсов"
  default     = "East US"
  }
  ```

- Создайте outputs.tf. Используйте этот файл для вывода важной информации, например, публичного IP-адреса виртуальной машины.

  ```
  # outputs.tf

  output "public_ip" {
  value = azurerm_public_ip.pubip.ip_address
  }
  ```

- Инициализация Terraform
  Инициализируйте Terraform для загрузки необходимых плагинов провайдеров:
  `  terraform init`
- Планирование инфраструктуры
  Просмотрите изменения, которые Terraform собирается внести:
  `  terraform plan`
- Применение конфигурации
  Примените конфигурацию для создания ресурсов в Azure:
  `  terraform apply`
- Проверка развертывания
  После завершения apply вы можете убедиться, что ресурсы созданы:

      Проверьте портал Azure, чтобы увидеть группу ресурсов, виртуальную сеть, подсеть и виртуальную машину.

      Используйте выведенный публичный IP для подключения по SSH к виртуальной машине:
      ```
      ssh azureuser@<public_ip>
      ```

- Очистка
  Если вы хотите удалить ресурсы и очистить среду Azure:
  `  terraform destroy`

**Задача 15**
Анализ лог файлов. Есть сервис, где периодически что-то происходит, из-за чего он медленнее обрабатывает запросы пользователей. Нужно посмотреть логи. Важно уметь искать в логах нужное событие. Быстро ориентироваться в бесконечно больших лог -файлах, при этом не нагружая диск при открытии(да здравствует less).

1. Использование less
   Команда less позволяет просматривать большие файлы, загружая их построчно, что минимизирует нагрузку на диск:
   `less /path/to/your/logfile.log` - Прокрутка: Используйте клавиши j и k для прокрутки вниз и вверх. - Поиск: Нажмите / и введите поисковый запрос, затем нажмите Enter. Для перехода к следующему совпадению нажмите n.
2. Использование grep для поиска ключевых слов
   Для поиска определенных строк в лог-файлах используйте grep:
   `bash
grep "ключевое_слово" /path/to/your/logfile.log
` - Игнорирование регистра: Чтобы игнорировать регистр, используйте флаг -i:
   `bash
grep -i "ключевое_слово" /path/to/your/logfile.log
` - Поиск с указанием времени: Если ваши логи содержат временные метки, можно искать записи за определенный период:
   `bash
grep "2024-09-25" /path/to/your/logfile.log
`
3. Комбинирование grep и less
   Вы можете отфильтровать нужные строки с помощью grep и просмотреть их через less:
   `bash
grep "ключевое_слово" /path/to/your/logfile.log | less
`
4. tail для просмотра последних строк
   Если интересуют последние записи в логе, используйте команду tail:
   `bash
tail -n 100 /path/to/your/logfile.log
` - Непрерывный мониторинг: Для непрерывного просмотра лога в реальном времени:
   `bash
    tail -f /path/to/your/logfile.log
    `
5. awk для более сложного анализа
   awk позволяет выполнять более сложные операции с логами, например, выводить только определенные столбцы или строки, соответствующие определенным условиям:
   `bash
awk '/ключевое_слово/ {print $0}' /path/to/your/logfile.log
`
6. sed для поиска и замены
   Если нужно найти и заменить текст в лог-файле:
   `bash
sed 's/старое_слово/новое_слово/g' /path/to/your/logfile.log
`
7. Использование zgrep для архивированных логов
   Если ваши логи архивированы (.gz), используйте zgrep для поиска:
   `bash
zgrep "ключевое_слово" /path/to/your/logfile.log.gz
`
8. find для поиска логов по времени создания
   Чтобы найти все логи, созданные за последние 24 часа:
   `bash
find /var/log/ -mtime -1 -name "*.log"
`

**Задача 16**
Нужно написать Ansible -скрипт, который можно будет применить на нескольких окружениях с разными внутренними IP-адресами, DNS-именами и секретами. Важно уметь правильно разделять и описывать конфигурации разных окружений. То есть использовать одни и те же роли для всех окружений, но разделять их с помощью inventory файлов и файлов с переменными и секретами.

1. Структура каталогов Ansible
   Предположим, у нас есть несколько окружений: production, staging, и dev. Для каждого из этих окружений мы будем использовать общие роли, но конфигурации, переменные и секреты будут отличаться.

```
ansible/
├── ansible.cfg
├── inventories/
│   ├── production/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   │   ├── all.yml
│   │   │   └── secrets.yml
│   │   └── host_vars/
│   │       └── host1.yml
│   ├── staging/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   │   ├── all.yml
│   │   │   └── secrets.yml
│   │   └── host_vars/
│   │       └── host1.yml
│   └── dev/
│       ├── hosts.yml
│       ├── group_vars/
│       │   ├── all.yml
│       │   └── secrets.yml
│       └── host_vars/
│           └── host1.yml
├── playbooks/
│   ├── deploy.yml
│   └── setup.yml
└── roles/
    ├── common/
    ├── webserver/
    └── database/
```

2. Инвентори-файлы
   Создадим инвентори-файл для каждого окружения (hosts.yml):

inventories/production/hosts.yml:

```
all:
  hosts:
    web1:
      ansible_host: 192.168.1.10
      dns_name: web1.prod.example.com
    db1:
      ansible_host: 192.168.1.11
      dns_name: db1.prod.example.com
  vars:
    environment: production
```

inventories/staging/hosts.yml:

```
all:
  hosts:
    web1:
      ansible_host: 192.168.2.10
      dns_name: web1.staging.example.com
    db1:
      ansible_host: 192.168.2.11
      dns_name: db1.staging.example.com
  vars:
    environment: staging
```

inventories/dev/hosts.yml:

```
all:
  hosts:
    web1:
      ansible_host: 192.168.3.10
      dns_name: web1.dev.example.com
    db1:
      ansible_host: 192.168.3.11
      dns_name: db1.dev.example.com
  vars:
    environment: dev
```

3. Файлы переменных
   В файлах переменных (all.yml), хранятся общие для окружения настройки.

inventories/production/group_vars/all.yml:

```
app_port: 8080
db_port: 5432
```

inventories/staging/group_vars/all.yml:

```
app_port: 8081
db_port: 5433
```

inventories/dev/group_vars/all.yml:

```
app_port: 8082
db_port: 5434
```

4. Файлы секретов
   Секреты могут быть сохранены в отдельных файлах (secrets.yml) и зашифрованы с помощью Ansible Vault.

inventories/production/group_vars/secrets.yml:

```
db_password: "super_secret_password_prod"
```

inventories/staging/group_vars/secrets.yml:

```
db_password: "super_secret_password_staging"
```

inventories/dev/group_vars/secrets.yml:

```
db_password: "super_secret_password_dev"
```

5. Роли
   Роли содержат конфигурации, которые будут применяться на всех окружениях. Например, роль webserver для настройки веб-сервера.

Пример структуры роли webserver:

```
roles/webserver/
├── tasks/
│   ├── main.yml
└── templates/
    └── nginx.conf.j2
```

roles/webserver/tasks/main.yml:

```
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Configure Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf

- name: Start Nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

6. Playbook
   Playbook будет включать в себя вызов необходимых ролей.

playbooks/deploy.yml:

```
- hosts: all
  become: yes
  roles:
    - common
    - webserver
    - database
```

7. Запуск Playbook
   Для выполнения плейбука на конкретном окружении используйте команду:

```
ansible-playbook -i inventories/production/hosts.yml playbooks/deploy.yml
```

**Задача 17**
Сделать анализ облачных провайдеров и выбрать подходящий под определённые требования. Например: клиент расположен в Казахстане, гос регулятор выдвигает требования, что данные пользователей должны хранится только в Казахстане, соответственно, сразу отпадает перечень провайдеров, которые расположены вне требуемой зоны. Важно провести глубокую аналитику всех канадских провайдеров, их перечня услуг, готовых решений, цен и так далее. Также важно смотреть на количество локаций дата центров, реальных отзывов и множество других переменных.

1. Обзор облачных провайдеров, доступных в Казахстане
   - Локальные провайдеры: Это могут быть казахстанские компании, которые предлагают облачные услуги. Они, вероятнее всего, соответствуют требованиям о хранении данных в стране.
   - Международные провайдеры: Некоторые крупные международные провайдеры, такие как AWS, Microsoft Azure, Google Cloud, могут иметь центры обработки данных в Казахстане или поддерживать локальные решения.
2. Критерии выбора провайдеров
   - Наличие дата-центров в Казахстане: Проверьте, есть ли у провайдера дата-центры, расположенные на территории Казахстана.
   - Соответствие требованиям законодательства: Убедитесь, что провайдер соответствует местным законам и регуляциям о защите данных.
   - Перечень услуг и решений: Оцените, какие услуги предлагает каждый провайдер, включая вычислительные ресурсы, хранилища данных, базы данных, инструменты аналитики и безопасность.
   - Цены и модели ценообразования: Сравните цены на услуги различных провайдеров, включая модели оплаты (по факту, предоплата и т. д.).
   - Отзывы и репутация: Исследуйте реальные отзывы клиентов, чтобы понять уровень сервиса, поддержку и надежность провайдера.
   - Техническая поддержка: Оцените, как быстро и качественно провайдер реагирует на запросы и проблемы клиентов.
   - Гибкость и масштабируемость: Рассмотрите, насколько легко можно масштабировать ресурсы и адаптироваться к изменяющимся требованиям бизнеса.
3. Сравнение провайдеров
   Составьте таблицу для сравнения облачных провайдеров по следующим параметрам: - Провайдер - Локации дата-центров - Услуги и решения - Цены - Отзывы клиентов - Техническая поддержка - Масштабируемость

4. Провайдеры для анализа

- Казахстанские провайдеры:
  - Kazteleport: Облачные решения с фокусом на безопасность и хранение данных в Казахстане.
  - Megaline: Услуги хостинга и облачные решения, ориентированные на местный рынок.
- Международные провайдеры:
  - Microsoft Azure: Проверьте наличие дата-центров и соответствие требованиям.
  - Amazon Web Services (AWS): Ознакомьтесь с предложениями в регионе, особенно если есть локальные зоны доступности.
  - Google Cloud: Анализируйте их возможности и поддерживаемые решения для бизнеса.

5. Заключение и рекомендации
   После сбора и анализа всех данных, сформулируйте рекомендации по выбору провайдера, который наилучшим образом соответствует требованиям клиента. Обсудите плюсы и минусы каждого провайдера и сделайте окончательный выбор, основываясь на фактических потребностях бизнеса и его готовности к определённым компромиссам.

**Задача 18**
Сравнение инструментов балансировки Nginx/Haproxy

Nginx и HAProxy — это два популярных инструмента для балансировки нагрузки, каждый из которых имеет свои особенности и преимущества. Вот сравнение этих двух инструментов, включая их функции, производительность и случаи использования.

1.  Nginx
    Основные характеристики:

        - Веб-сервер и балансировщик нагрузки: Nginx изначально разрабатывался как веб-сервер, но быстро завоевал популярность в качестве балансировщика нагрузки благодаря своей высокой производительности.
        - Поддержка HTTP/2 и WebSocket: Nginx поддерживает современные протоколы, что делает его хорошим выбором для веб-приложений.
        - Многофункциональность: Помимо балансировки нагрузки, Nginx может выполнять функции обратного прокси, кэширования, защиты от DDoS и т.д.
        - Настройка и управление: Конфигурация Nginx осуществляется с помощью простых текстовых файлов, что позволяет легко настраивать серверы и маршрутизацию.
        - Методы балансировки нагрузки:
            - Round Robin (по умолчанию): Запросы распределяются равномерно между серверами.
            - Least Connections: Запрос отправляется на сервер с наименьшим количеством активных соединений.
            - IP Hash: Запросы от одного IP-адреса будут направляться на один и тот же сервер.
            - Weighted Round Robin: Различные веса для серверов позволяют управлять объемом запросов, направляемых на каждый сервер.
        - Сценарии использования:
            - Идеально подходит для веб-приложений, которые требуют быстрой обработки HTTP-запросов.
            - Подходит для создания кластера, обработки статического контента и балансировки нагрузки для приложений, использующих HTTP/2.

2.  HAProxy
    Основные характеристики: - Специализированный балансировщик нагрузки: HAProxy был специально разработан для распределения нагрузки и обеспечивает высокую доступность и производительность. - Поддержка TCP и HTTP: HAProxy может балансировать как HTTP, так и TCP-трафик, что делает его универсальным инструментом. - Мониторинг и алерты: HAProxy предоставляет множество возможностей для мониторинга состояния серверов и генерации алертов при возникновении проблем. - Сложные сценарии маршрутизации: HAProxy поддерживает более сложные правила маршрутизации и балансировки нагрузки, чем Nginx. - Методы балансировки нагрузки: - Round Robin: Балансировка нагрузки с равномерным распределением запросов. - Least Connections: Запрос отправляется на сервер с наименьшим количеством соединений. - Source: Запросы от одного источника направляются на один и тот же сервер. - Random: Запросы распределяются случайным образом между серверами. - Сценарии использования: - Подходит для сложных архитектур, где требуется балансировка как HTTP, так и TCP-трафика. - Идеально для высоконагруженных систем, требующих высокой производительности и отказоустойчивости.

**Задача 19**
Напишите пошаговые примеры конфигурации Nginx и HAProxy в качестве балансировщиков нагрузки.

1. Пример конфигурации Nginx: шаг за шагом

- Установка Nginx
  Если Nginx еще не установлен, выполните установку. Например, для Ubuntu используйте:
  ```
  sudo apt update
  sudo apt install nginx
  ```
- Настройка Nginx для балансировки нагрузки
  Откройте файл конфигурации Nginx. Обычно он находится по пути /etc/nginx/nginx.conf или /etc/nginx/sites-available/default

  ```
  sudo nano /etc/nginx/sites-available/default
  ```

  Добавьте следующий код в файл:

  ```
  http {
  upstream backend {
      server backend1.example.com;  # IP-адрес или DNS имя первого бэкенда
      server backend2.example.com;  # IP-адрес или DNS имя второго бэкенда
  }

  server {
      listen 80;  # Порт, на котором будет слушать Nginx
      server_name example.com;  # Доменное имя вашего приложения

      location / {
          proxy_pass http://backend;  # Перенаправление запросов на upstream
          proxy_set_header Host $host;  # Устанавливаем заголовок Host
          proxy_set_header X-Real-IP $remote_addr;  # Передаем реальный IP-адрес клиента
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  # Передаем IP-адрес клиента
          }
      }
  }
  ```

- Проверьте конфигурацию на наличие ошибок:
  ```
  sudo nginx -t
  ```
- Перезапустите Nginx для применения изменений:
  ```
  sudo systemctl restart nginx
  ```

2. Пример конфигурации HAProxy: шаг за шагом

- Установка HAProxy
  Если HAProxy еще не установлен, выполните установку. Например, для Ubuntu используйте:
  `   sudo apt update
  sudo apt install haproxy
`
- Настройка HAProxy для балансировки нагрузки

  - Откройте файл конфигурации HAProxy. Обычно он находится по пути /etc/haproxy/haproxy.cfg.

  ```
  sudo nano /etc/haproxy/haproxy.cfg
  ```

  - Добавьте следующий код в файл:

  ```nginx
  frontend http_front
      bind *:80  # Порт, на котором будет слушать HAProxy
      acl url_static path_beg /static  # Правило для статических файлов
      use_backend static_servers if url_static  # Использование другого бэкенда для статических файлов
      default_backend web_servers  # Основной бэкенд для остальных запросов

  backend web_servers
      balance roundrobin  # Балансировка нагрузки по кругу
      server web1 backend1.example.com:80 check  # Первый бэкенд
      server web2 backend2.example.com:80 check  # Второй бэкенд

  backend static_servers
      balance static-rr  # Балансировка нагрузки для статических файлов
      server static1 static1.example.com:80 check  # Сервер для статических файлов

  ```

  - Проверьте конфигурацию на наличие ошибок:

  ```
  sudo haproxy -c -f /etc/haproxy/haproxy.cfg
  ```

  - Перезапустите HAProxy для применения изменений:

  ```
  sudo systemctl restart haproxy
  ```

**Задача 20**
Настройка кластера Kubernetes включает несколько этапов, от установки необходимых инструментов до развертывания приложений. Вот пошаговая инструкция по настройке Kubernetes с использованием Minikube, который идеален для локальной разработки и тестирования.

1. Установка необходимых инструментов

- Установите Minikube: Minikube позволяет запускать локальный кластер Kubernetes. Для Linux:

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

- Установите kubectl: kubectl — это CLI инструмент для управления Kubernetes.

```
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

2. Запуск Minikube

- Запустите Minikube:
  `minikube start`
  Это создаст и запустит локальный кластер Kubernetes.
- Проверьте статус Minikube:
  `minikube status`

3. Основные команды kubectl

- Проверьте текущий контекст:
  `kubectl config current-context`
- Проверьте состояние кластера:
  `kubectl cluster-info`
- Проверьте ноды в кластере:
  `kubectl get nodes`

4. Развертывание приложения

- Создайте файл конфигурации для приложения: Создайте файл deployment.yaml с конфигурацией развертывания вашего приложения. Например, для Nginx:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

- Примените конфигурацию:
  `kubectl apply -f deployment.yaml`
- Проверьте статус развертывания:
  `kubectl get deployments`

5. Создание сервиса

- Создайте файл конфигурации для сервиса: Создайте файл service.yaml:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
  selector:
    app: nginx
```

- Примените конфигурацию сервиса:
  `kubectl apply -f service.yaml`
- Проверьте статус сервиса:
  `kubectl get services`

6. Доступ к приложению

- Получите URL для доступа к приложению:
  `minikube service nginx-service --url`
- Откройте браузер и введите полученный URL.

7. Удаление кластера

- Остановите Minikube:
  `minikube stop`
- Удалите Minikube:
  `minikube delete`

**Задача 21**
Для развертывания PostgreSQL в Kubernetes с использованием ConfigMaps, Secrets и Persistent Volumes мы можем следовать следующим шагам. Этот подход позволит сохранить настройки и данные базы данных безопасно и эффективно.

1. Создание Persistent Volume (PV) и Persistent Volume Claim (PVC)
   Persistent Volume (PV) позволяет Kubernetes сохранить данные после перезагрузки пода, а Persistent Volume Claim (PVC) — это запрос на использование хранилища.

- Создайте файл postgres-pv.yaml:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/postgres
```

- Создайте файл postgres-pvc.yaml:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

- Примените эти файлы:

```
kubectl apply -f postgres-pv.yaml
kubectl apply -f postgres-pvc.yaml
```

2. Создание Secrets
   Secrets используются для хранения конфиденциальной информации, такой как пароли и токены. В нашем случае мы создадим Secret для пароля PostgreSQL.

- Создайте файл postgres-secret.yaml:

```
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
type: Opaque
data:
  postgres-password: <base64_encoded_password>
```

- Чтобы закодировать пароль в base64, вы можете использовать команду:

```
echo -n "your_password" | base64
```

- Примените Secret:

```
kubectl apply -f postgres-secret.yaml
```

3. Создание ConfigMap
   ConfigMap позволяет управлять конфигурациями приложения. Мы создадим ConfigMap для хранения параметров конфигурации PostgreSQL.

- Создайте файл postgres-configmap.yaml:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
data:
  POSTGRES_DB: mydatabase
  POSTGRES_USER: myuser
```

- Примените ConfigMap:

```
kubectl apply -f postgres-configmap.yaml
```

4. Создание Deployment для PostgreSQL
   Теперь мы можем создать Deployment для PostgreSQL, используя ранее созданные Persistent Volume, Secret и ConfigMap.

- Создайте файл postgres-deployment.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:13
          env:
            - name: POSTGRES_DB
              valueFrom:
                configMapKeyRef:
                  name: postgres-config
                  key: POSTGRES_DB
            - name: POSTGRES_USER
              valueFrom:
                configMapKeyRef:
                  name: postgres-config
                  key: POSTGRES_USER
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: postgres-password
          ports:
            - containerPort: 5432
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgres-storage
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pvc
```

- Примените Deployment:

```
kubectl apply -f postgres-deployment.yaml
```

5. Проверка развертывания

- Проверьте статус пода:

```
kubectl get pods
```

- Получите логи PostgreSQL:

```
kubectl logs <pod-name>
```

6. Создание Service для PostgreSQL
   Чтобы другие приложения могли подключаться к PostgreSQL, создадим Service.

- Создайте файл postgres-service.yaml:

```
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  type: ClusterIP
  ports:
    - port: 5432
      targetPort: 5432
  selector:
    app: postgres
```

- Примените Service:

```
kubectl apply -f postgres-service.yaml
```

**Задание 22**
Для настройки стека мониторинга и логирования на Kubernetes с использованием Prometheus, Grafana и ELK (Elasticsearch, Logstash, Kibana), выполните следующие шаги. Этот процесс включает в себя установку всех компонентов, их конфигурацию и интеграцию.

1. Установка Helm
   Helm — это пакетный менеджер для Kubernetes, который упрощает установку и управление приложениями в кластере.

- Установите Helm, если он еще не установлен:

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

2. Установка Prometheus и Grafana

- Добавьте репозиторий Helm для Prometheus:

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

- Установите Prometheus:

```
helm install prometheus prometheus-community/prometheus
```

- Установите Grafana:

```
helm install grafana grafana/grafana
```

- Получите пароль для доступа к Grafana:

```
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

- Получите URL для доступа к Grafana:

```
kubectl get svc grafana
```

3. Установка ELK Stack

- Добавьте репозиторий Helm для ELK:

```
helm repo add elastic https://helm.elastic.co
helm repo update
```

- Установите Elasticsearch:

```
helm install elasticsearch elastic/elasticsearch
```

- Установите Kibana:

```
helm install kibana elastic/kibana
```

- Установите Logstash:

```
helm install logstash elastic/logstash
```

4. Настройка интеграции

- Настройка Logstash для отправки логов в Elasticsearch

  - Создайте файл конфигурации для Logstash logstash-configmap.yaml:

  ```
  apiVersion: v1
  kind: ConfigMap
  metadata:
  name: logstash-config
  data:
  logstash.conf: |
      input {
      beats {
          port => 5044
      }
      }
      filter {
      # Примените необходимые фильтры
      }
      output {
      elasticsearch {
          hosts => ["http://elasticsearch:9200"]
          index => "logs-%{+YYYY.MM.dd}"
      }
      }
  ```

  - Примените ConfigMap:

  ```
  kubectl apply -f logstash-configmap.yaml
  ```

  - Измените Deployment Logstash, чтобы использовать этот ConfigMap.

- Настройка Filebeat или Fluentd для отправки логов в Logstash

  - Установите Filebeat:

  ```
  helm repo add elastic https://helm.elastic.co
  helm install filebeat elastic/filebeat
  ```

  - Настройте Filebeat для отправки логов в Logstash.

- Доступ к Kibana и Grafana
  - Получите URL для доступа к Kibana:
  ```
  kubectl get svc kibana
  ```
