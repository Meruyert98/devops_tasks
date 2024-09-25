# Nginx

NGINX — это высокопроизводительный HTTP-сервер, реверс-прокси сервер и сервер для балансировки нагрузки. Он также может быть использован как почтовый сервер и универсальный TCP/UDP прокси. Веб-сервер NGINX стал популярен благодаря своей способности обрабатывать высокие нагрузки и поддерживать большое количество одновременных подключений. Ниже приведены основные концепции NGINX с примерами конфигураций и команд.

1. Конфигурационный файл (nginx.conf)
   Основной файл конфигурации NGINX обычно находится в /etc/nginx/nginx.conf (или /usr/local/nginx/conf/nginx.conf). В нем задаются глобальные настройки сервера, а также настройки для отдельных серверов и локаций.

- Пример базовой конфигурации:

  ```nginx
  user  www-data;
  worker_processes  4;

  events {
      worker_connections  1024;
  }

  http {
      include       /etc/nginx/mime.types;
      default_type  application/octet-stream;

      sendfile        on;
      keepalive_timeout  65;

      server {
          listen       80;
          server_name  example.com;

          location / {
              root   /var/www/html;
              index  index.html index.htm;
          }

          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   /usr/share/nginx/html;
          }
      }
  }
  ```

2. Worker Processes и Events
   worker_processes определяет количество рабочих процессов, которые будут обрабатывать запросы. Обычно рекомендуется устанавливать его равным количеству ядер процессора.

- Пример:
  ```nginx
  worker_processes auto;
  ```
- Секция events задает параметры управления событиями, такими как количество одновременных соединений (worker_connections), которые каждый процесс может обрабатывать. Пример:
  ```nginx
  events {
      worker_connections 1024;
  }
  ```

3. HTTP блок
   Секция http управляет веб-сервером и содержит настройки, касающиеся обработки HTTP-запросов.

- Основные параметры:
  - sendfile: Включает ускоренную передачу файлов.
  - keepalive_timeout: Устанавливает таймаут для поддержания активного соединения.
- Пример:
  ```nginx
  http {
      sendfile on;
      keepalive_timeout 65;
  }
  ```

4. Серверы (Server Block)
   Server Block используется для настройки разных виртуальных хостов. Внутри одного блока можно указать настройки для конкретного домена, например, порты, по которым сервер будет доступен, корневую директорию и т.д.

- Пример настройки сервера:

  ```nginx
  server {
      listen 80;
      server_name example.com www.example.com;

      root /var/www/example;
      index index.html index.htm;

      location / {
          try_files $uri $uri/ =404;
      }
  }
  ```

- Пояснение:
  - listen: Указывает на какой порт будет "слушать" NGINX (в данном случае 80).
  - server_name: Устанавливает доменное имя (можно указать несколько).
  - root: Указывает путь до каталога, в котором находятся файлы сайта.
  - location: Определяет, как обрабатывать запросы на конкретные пути.

5. Location Block
   Location Block позволяет задать обработку для определённых URL или URI. Это могут быть как простые шаблоны, так и регулярные выражения.

- Пример настройки location:
  - Этот блок будет перенаправлять запросы к /images/ в папку /data/images/.
  ```nginx
  location /images/ {
      root /data;
  }
  ```

6. Обработка ошибок
   NGINX позволяет настроить пользовательские страницы для обработки ошибок (например, 404, 500 и т.д.).

- Пример обработки ошибки:
  ```nginx
  error_page 404 /404.html;
  location = /404.html {
      root /usr/share/nginx/html;
  }
  ```

7. Реверс-прокси (Reverse Proxy)
   NGINX может действовать как реверс-прокси, перенаправляя запросы на другие серверы (например, бэкэнд-сервера).

- Пример настройки реверс-прокси (Этот сервер будет перенаправлять все запросы на внутренний сервер, работающий на порту 8080.):

  ```nginx
  server {
      listen 80;
      server_name example.com;

      location / {
          proxy_pass http://127.0.0.1:8080;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
  }
  ```

8. Балансировка нагрузки (Load Balancing)
   NGINX может распределять запросы между несколькими серверами для балансировки нагрузки.

- Пример конфигурации для балансировки нагрузки:

  ```nginx
  upstream backend {
      server backend1.example.com;
      server backend2.example.com;
  }

  server {
      listen 80;

      location / {
          proxy_pass http://backend;
      }
  }
  ```

- В данном примере запросы будут распределяться между двумя серверами backend1 и backend2.

9. SSL/TLS
   NGINX поддерживает настройку HTTPS с использованием сертификатов SSL/TLS.

- Пример настройки HTTPS:

  ```nginx
  server {
      listen 443 ssl;
      server_name example.com;

      ssl_certificate /etc/nginx/ssl/example.com.crt;
      ssl_certificate_key /etc/nginx/ssl/example.com.key;

      location / {
          root /var/www/html;
      }
  }
  ```

- Этот блок включает HTTPS на порту 443 и использует указанные сертификаты для шифрования соединений.

10. Кэширование
    NGINX поддерживает кэширование контента, что может значительно повысить производительность сервера.

- Пример настройки кэширования:

  ```nginx
  location / {
      proxy_cache my_cache;
      proxy_cache_valid 200 1h;
      proxy_pass http://backend;
  }

  proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=my_cache:10m;
  ```

- В этом примере сервер будет кэшировать ответы на запросы в течение одного часа.

11. Геолокация и доступ по IP
    NGINX позволяет ограничить доступ к ресурсу по IP-адресам.

- Пример ограничения доступа:
  ```nginx
  location /admin/ {
      allow 192.168.1.0/24;
      deny all;
  }
  ```
- Этот блок разрешает доступ к /admin/ только для IP-адресов из подсети 192.168.1.0/24.

12. Перенаправления (Redirects)
    NGINX поддерживает перенаправления запросов с одного URL на другой.

- Пример перенаправления:
  ```nginx
  server {
      listen 80;
      server_name www.example.com;
      return 301 https://example.com$request_uri;
  }
  ```
- Этот блок перенаправит все запросы с www.example.com на https://example.com.

13. Логирование (Logging)
    NGINX ведет логи запросов и ошибок, что помогает в мониторинге и диагностике.

- Пример настройки логов:

  ```nginx
  http {
      log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

      access_log /var/log/nginx/access.log main;
      error_log /var/log/nginx/error.log warn;
  }
  ```

- Этот блок определяет формат логов и пути к файлам логов.

14. Комплексные примеры

- Пример 1: Конфигурация для реверс-прокси с балансировкой нагрузки и SSL

```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
}

server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

- Пример 2: Конфигурация с кэшированием статического контента

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        root /var/www/html;
        index index.html;
    }

    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 30d;
        add_header Cache-Control "public";
    }
}
```
