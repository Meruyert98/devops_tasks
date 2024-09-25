# Systemd

**systemd** — это система инициализации и управления службами (сервисами), которая используется в большинстве современных дистрибутивов Linux (например, Ubuntu, CentOS, Debian). systemd позволяет управлять службами, процессами, точками монтирования, устройствами и многими другими аспектами операционной системы. Ниже приведены основные концепции systemd с примерами.

1. Службы (services)
   В systemd служба — это процесс или приложение, которое запускается и управляется системой. Определение служб содержится в так называемых unit файлах.

- Пример команды для запуска службы:
  ```bash
  sudo systemctl start nginx  # Запуск службы Nginx
  ```
- Пример команды для остановки службы:
  ```bash
  sudo systemctl stop nginx  # Остановка службы Nginx
  ```
- Пример команды для перезапуска службы:
  ```bash
  sudo systemctl restart nginx  # Перезапуск службы Nginx
  ```
- Пример команды для проверки статуса службы:
  ```bash
  sudo systemctl status nginx  # Проверка статуса службы Nginx
  ```

2. Unit файлы
   Unit файлы описывают поведение служб и других управляемых ресурсов в системе systemd. Они имеют расширение .service, .socket, .mount, и так далее.

- Пример файла службы (/etc/systemd/system/myapp.service):

  ```bash
  [Unit]
  Description=My Custom Application
  After=network.target

  [Service]
  ExecStart=/usr/bin/myapp
  Restart=always

  [Install]
  WantedBy=multi-user.target
  ```

- Основные секции Unit файлов:

  - [Unit]: Описание службы и её зависимостей.
  - [Service]: Настройки конкретной службы (как её запускать, перезапускать и т.д.).
  - [Install]: Указывает, как и когда служба должна быть активирована (например, при загрузке системы).

3. Автозапуск (Enable/Disable)
   Службы могут быть настроены на автоматический запуск при старте системы.

- Пример включения автозапуска:
  ```bash
  sudo systemctl enable nginx  # Включение автозапуска службы Nginx
  ```
- Пример отключения автозапуска:
  ```bash
  sudo systemctl disable nginx  # Отключение автозапуска службы Nginx
  ```

4. Перезагрузка демона systemd
   После изменения Unit файлов или установки новых служб, нужно перезапустить систему systemd, чтобы она применяла изменения.

- Перезагрузка конфигурации systemd:
  ```bash
  sudo systemctl daemon-reload  # Перезагрузка демона systemd
  ```

5. Журналы (Logging) с journalctl
   systemd использует journalctl для просмотра системных журналов.

- Просмотр журнала конкретной службы:
  ```bash
  sudo journalctl -u nginx.service  # Просмотр логов службы Nginx
  ```
- Просмотр последних 100 строк журнала:
  ```bash
  sudo journalctl -n 100
  ```
- Просмотр журнала с момента последней перезагрузки:
  ```bash
  sudo journalctl -b
  ```
- Просмотр журнала в реальном времени:
  ```bash
  sudo journalctl -f
  ```

6. Состояние системы и службы
   systemd позволяет получить общую информацию о состоянии системы и её компонентов.

- Общее состояние системы:
  ```bash
  systemctl status
  ```
- Просмотр всех активных служб:
  ```bash
  systemctl list-units --type=service --state=active
  ```

7. Сокеты (sockets)
   systemd может управлять сокетами и автоматически запускать службы при получении подключения к определенному сокету.

- Пример файла сокета:

  ```bash
  [Unit]
  Description=My Custom App Socket

  [Socket]
  ListenStream=8080

  [Install]
  WantedBy=sockets.target
  ```

8. Таймеры (timers)
   Таймеры в systemd заменяют традиционные задачи cron. Они позволяют запускать службы по расписанию.

- Пример файла таймера:

  ```bash
  [Unit]
  Description=Run myapp every hour

  [Timer]
  OnCalendar=hourly

  [Install]
  WantedBy=timers.target
  ```

- Запуск таймера:
  ```bash
  sudo systemctl start myapp.timer
  ```
- Просмотр активных таймеров:
  ```bash
  sudo systemctl list-timers
  ```

9. Цели (targets)
   Цели (targets) — это наборы служб, которые systemd пытается запустить при достижении определенного состояния системы, например при загрузке или выключении.

- Пример переключения на цель выключения:
  ```bash
  sudo systemctl isolate poweroff.target  # Немедленное завершение работы системы
  ```
- Просмотр текущей цели:
  ```bash
  systemctl get-default
  ```
- Установка цели по умолчанию:
  ```bash
  sudo systemctl set-default multi-user.target  # Установить многопользовательский режим по умолчанию
  ```

10. Отключение и завершение системы
    systemd управляет процессами завершения работы и перезагрузки.

- Завершение работы системы:
  ```bash
  sudo systemctl poweroff
  ```
- Перезагрузка системы:
  ```bash
  sudo systemctl reboot
  ```
- Перевод системы в спящий режим:
  ```bash
  sudo systemctl suspend
  ```

11. Монтирование файловых систем
    systemd также управляет точками монтирования и автоподключением файловых систем через .mount юниты.

- Пример файла монтирования:

  ```bash
  [Unit]
  Description=Mount My Disk

  [Mount]
  What=/dev/sdb1
  Where=/mnt/mydisk
  Type=ext4

  [Install]
  WantedBy=multi-user.target
  ```

- Запуск точки монтирования:
  ```bash
  sudo systemctl start mnt-mydisk.mount
  ```

12. Обслуживание устройств
    systemd позволяет управлять устройствами через .device юниты. Это полезно для автоматической реакции на появление новых устройств.

- Пример:
  ```bash
  [Unit]
  Description=USB Flash Drive Auto Mount
  Requires=dev-sdb1.device
  ```

13. Параллельный запуск и зависимые службы
    systemd поддерживает параллельный запуск служб и возможность указания зависимостей между службами.

- Пример зависимости (Служба будет запущена после инициализации сети.):
  ```bash
  [Unit]
  Description=My Dependent Service
  After=network.target
  ```

14. Перезагрузка служб по отказу
    systemd поддерживает автоматический перезапуск служб в случае их сбоя.

- Пример перезапуска:
  ```bash
  [Service]
  ExecStart=/usr/bin/myapp
  Restart=on-failure
  ```

15. Кэширование с помощью systemd
    systemd поддерживает кэширование через специальные юниты .cache.

- Пример:

  ```bash
  [Unit]
  Description=My Cache

  [Cache]
  Type=memory
  ```

16. Управление окружением служб
    Службы могут использовать переменные окружения, которые задаются через файл конфигурации.

- Пример:
  ```bash
  [Service]
  Environment="ENV_VAR=/path/to/dir"
  ```

17. Отладка и диагностика
    systemd имеет встроенные инструменты для диагностики проблем служб.

- Пример проверки зависимостей юнита:
  ```bash
  systemctl list-dependencies nginx
  ```
- Отладочный режим:
  ```bash
  sudo systemctl --debug start nginx
  ```
