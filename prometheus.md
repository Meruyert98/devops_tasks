# Prometheus

**Prometheus** — это система мониторинга и сбора метрик с открытым исходным кодом, предназначенная для отслеживания состояния и производительности инфраструктуры. Она позволяет собирать метрики из различных систем, хранить их, выполнять запросы и визуализировать данные. Вот основные концепты Prometheus с примерами:

**1. Метрики (Metrics)**
Метрики — это данные, которые собираются Prometheus. Каждая метрика имеет название, значение, время сбора и метки (labels).

- Основные типы метрик:

  - Counter: Метрика, которая только увеличивается. Пример: количество запросов к API.
  - Gauge: Метрика, которая может увеличиваться и уменьшаться. Пример: текущее количество подключений.
  - Histogram: Счётчик, который записывает распределение данных (например, время обработки запросов).
  - Summary: Похож на гистограмму, но с фокусом на агрегированные значения, такие как процентиль.

- Пример:
  Метрика, которая отслеживает количество HTTP-запросов:
  ```bash
  http_requests_total{method="POST", endpoint="/api"} 500
  ```
  - http_requests_total — название метрики.
  - method="POST" и endpoint="/api" — метки.
  - 500 — значение метрики (общее количество запросов).

**2. Экспортеры (Exporters)**
Экспортеры — это сервисы, которые собирают метрики с систем (например, операционных систем, баз данных) и предоставляют их в формате, понятном для Prometheus.

- Пример:
  Экспортер Node Exporter для сбора метрик операционной системы (например, загрузка процессора, использование памяти).

      ```bash
      node_cpu_seconds_total{cpu="0",mode="idle"} 10000
      ```
      - Эта метрика показывает, сколько времени процессор провел в режиме бездействия.

**3. Сбор данных (Scraping)**
Prometheus работает на основе модели pull — он периодически запрашивает метрики с систем через HTTP. Все системы, с которых собираются данные, должны предоставлять метрики в формате, совместимом с Prometheus (обычно это /metrics endpoint).

- Пример:
  Сервер Nginx, настроенный на экспортер, предоставляет метрики на /metrics endpoint. Prometheus с определённой частотой делает HTTP-запросы:
  ```yaml
  scrape_configs:
  - job_name: 'nginx'
    static_configs: - targets: ['localhost:9113']
  ```

**4. Запросы (PromQL)**
Prometheus использует собственный язык запросов PromQL для анализа и агрегирования метрик. С его помощью можно выполнять сложные запросы, фильтровать и вычислять данные.

- Примеры запросов:
  - Сумма всех запросов:
  ```sql
  sum(http_requests_total)
  ```
  - Средняя загрузка процессора за последние 5 минут:
  ```sql
  avg_over_time(node_load1[5m])
  ```
  - Процентиль времени ответа (95-й процентиль):
  ```sql
  histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
  ```

**5. Алерты (Alerts)**
Prometheus может настраивать алерты на основе определённых условий. Когда условие срабатывает (например, высокий уровень нагрузки на CPU), Prometheus может отправлять уведомления через Alertmanager.

- Пример:
  Алерт, который срабатывает, если загрузка процессора превышает 80% более 5 минут:
  ```yaml
  groups:
      - name: cpu_alerts
          rules:
          - alert: HighCPUUsage
              expr: avg_over_time(node_cpu_seconds_total[5m]) > 0.8
              for: 5m
              labels:
              severity: critical
              annotations:
              summary: "Высокая загрузка процессора на {{ $labels.instance }}"
              description: "Процессор на {{ $labels.instance }} загружен больше чем на 80% в течение последних 5 минут."
  ```

**6. Хранение данных (Storage)**
Prometheus использует встроенное хранилище для временных рядов метрик. Оно оптимизировано для быстрого чтения и записи данных. Метрики хранятся в виде временных рядов, каждый из которых состоит из значений и временных меток.

- Пример:
  Метрика, собранная каждые 15 секунд:
  ```bash
  timestamp: 2024-09-26T10:00:00Z value: 500
  timestamp: 2024-09-26T10:00:15Z value: 510
  timestamp: 2024-09-26T10:00:30Z value: 505
  ```

**7. Визуализация (Grafana)**
Prometheus интегрируется с системами визуализации, например, с Grafana. Это позволяет создавать дашборды и графики на основе метрик, собранных Prometheus.

- Пример визуализации:
  - График использования CPU: показывающий текущее и среднее использование процессора за разные временные промежутки.
  - График ошибок HTTP 500: отображающий количество ошибок на веб-сервере за последний час.

**8. Pushgateway**
Иногда сервисы или задачи могут быть кратковременными и не всегда доступны для прямого запроса Prometheus. В таких случаях используется Pushgateway, который позволяет отправлять метрики в Prometheus.

- Пример:
  Кратковременная задача может отправить данные о своём статусе через Pushgateway.
  ```bash
    curl -X POST http://localhost:9091/metrics/job/batch_job \
    -d "batch_job_last_success_timestamp_seconds 1664184000"
  ```