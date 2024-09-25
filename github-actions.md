# Github Actions

GitHub Actions — это инструмент для автоматизации процессов разработки, который позволяет создавать, тестировать и развертывать проекты прямо из вашего репозитория на GitHub. Ниже приведены основные концепты GitHub Actions с примерами.

1. Workflows (Рабочие процессы)
   Рабочий процесс — это набор автоматизированных шагов, определённых в YAML-файле, который описывает, как должна выполняться ваша CI/CD-автоматизация.

- Пример:

  ```yaml
  name: CI

  on:
      push:
          branches:
          - main

  jobs:
      build:
          runs-on: ubuntu-latest
          steps:
          - name: Checkout code
              uses: actions/checkout@v2

          - name: Set up Node.js
              uses: actions/setup-node@v2
              with:
              node-version: '14'

          - name: Install dependencies
              run: npm install

          - name: Run tests
              run: npm test
  ```

2. Jobs (Задачи)
   Задача — это набор шагов, которые выполняются на одном и том же runner (исполнитель).

- Пример:

  ```yaml
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - name: Checkout code
          uses: actions/checkout@v2
  ```

3. Steps (Шаги)
   Шаги — это отдельные команды, которые выполняются в рамках задачи. Шаги могут использоваться для выполнения скриптов или вызова действий.

- Пример:

  ```yaml
  steps:
    - name: Run a script
      run: echo "Hello, World!"
  ```

4. Actions (Действия)
   Действия — это отдельные модули, которые могут выполнять конкретные функции, например, развертывание приложения, отправка уведомлений и т. д. Вы можете использовать действия из GitHub Marketplace или создавать свои собственные.

- Пример использования действия:

  ```yaml
  steps:
      - name: Checkout code
          uses: actions/checkout@v2

      - name: Set up Python
          uses: actions/setup-python@v2
          with:
          python-version: '3.8'

  ```

5. Triggers (Триггеры)
   Триггеры — это события, которые запускают выполнение вашего рабочего процесса. Вы можете настроить триггеры на основе событий, таких как push, pull request и т. д.

- Пример:
  ```yaml
  on:
    push:
      branches:
        - main
    pull_request:
      branches:
        - main
  ```

6. Environment Variables (Переменные окружения)
   Вы можете использовать переменные окружения для передачи конфиденциальной информации, такой как токены доступа или ключи API.

- Пример:
  ```yaml
  env:
    NODE_ENV: production
  ```

7. Secrets (Секреты)
   Секреты — это конфиденциальные данные, которые могут использоваться в рабочих процессах, но не отображаются в журнале. Их можно настроить в настройках вашего репозитория.

- Пример использования секретов:

  ```yaml
  steps:
      - name: Deploy to Server
          run: ./deploy.sh
          env:
          API_TOKEN: ${{ secrets.API_TOKEN }}

  ```

8. Matrix Builds (Матрица сборок)
   Матрица сборок позволяет вам запускать несколько параллельных задач с различными конфигурациями, такими как разные версии программного обеспечения или платформы.

- Пример:
  ```yaml
  jobs:
  build:
      runs-on: ubuntu-latest
      strategy:
      matrix:
          node-version: [12, 14, 16]
      steps:
      - name: Set up Node.js
          uses: actions/setup-node@v2
          with:
          node-version: ${{ matrix.node-version }}
      - run: npm install
      - run: npm test
  ```

9. Caching (Кэширование)
   Кэширование позволяет ускорить выполнение сборок, сохраняя зависимости и артефакты между запусками.

- Пример:

  ```yaml
  steps:
      - name: Cache Node.js modules
          uses: actions/cache@v2
          with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
              ${{ runner.os }}-node-
  ```
