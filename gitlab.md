# GitLab CI/CD

GitLab CI/CD — это встроенный инструмент непрерывной интеграции и непрерывного развертывания (CI/CD), который позволяет автоматизировать процесс сборки, тестирования и развертывания кода. Ниже представлены основные концепты GitLab CI/CD с примерами их использования.

1. .gitlab-ci.yml
   Это основной файл конфигурации для CI/CD в GitLab. Он определяет все этапы и задачи, которые GitLab Runner должен выполнять. Файл должен находиться в корне вашего репозитория.

- Пример .gitlab-ci.yml:

  ```yaml
  stages:
    - build
    - test
    - deploy
    - notify

  build:
    stage: build
    script:
      - echo "Building the application..."
      - make build

  test:
    stage: test
    script:
      - echo "Running tests..."
      - make test

  deploy:
    stage: deploy
    script:
      - echo "Deploying the application..."
      - make deploy

  telegram_notify_success:
  stage: notify
  script:
    - 'curl -s -X POST https://api.telegram.org/bot<YOUR_BOT_TOKEN>/sendMessage -d chat_id=<YOUR_CHAT_ID> -d text="✅ Pipeline Success: $CI_PROJECT_NAME\nCommit: $CI_COMMIT_MESSAGE"'
  only:
    - main # Adjust according to your branch preferences
  when: on_success

  telegram_notify_failure:
    stage: notify
    script:
      - 'curl -s -X POST https://api.telegram.org/bot<YOUR_BOT_TOKEN>/sendMessage -d chat_id=<YOUR_CHAT_ID> -d text="❌ Pipeline Failure: $CI_PROJECT_NAME\nCommit: $CI_COMMIT_MESSAGE"'
    only:
      - main # Adjust according to your branch preferences
    when: on_failure
  ```

2. Этапы (Stages)
   Этапы — это логические группы задач. Обычно они представляют собой основные шаги в процессе CI/CD, такие как сборка, тестирование и развертывание.

- Определение этапов: В примере выше мы определили три этапа: build, test и deploy.

3. Задачи (Jobs)
   Задачи — это конкретные команды, которые выполняются на каждом этапе. Каждая задача выполняется в изолированном окружении.

- Пример задачи: В примере выше задачи build, test и deploy выполняют команды, определенные в блоке script.

4. Артефакты (Artifacts)
   Артефакты — это файлы, которые сохраняются после выполнения задач и могут быть использованы в последующих этапах. Это полезно для передачи результатов сборки или тестирования между этапами.

- Пример использования артефактов:
  yaml
  `yaml
build:
    stage: build
    script:
        - make build
    artifacts:
        paths:
        - build/
`

5. Кэш (Cache)
   Кэш позволяет сохранять зависимости между запусками CI/CD, что ускоряет процесс сборки. Он хранит файлы, которые могут быть использованы в будущих задачах.

- Пример использования кэша:
  ```yaml
  cache:
    paths:
      - node_modules/
  ```

6. Условия (Conditions)
   Условия позволяют запускать задачи или этапы только при определенных обстоятельствах, например, только при выполнении определенных условий или на определенных ветках.

- Пример условия:
  ```yaml
  deploy:
    stage: deploy
    script:
      - echo "Deploying..."
    only:
      - master
  ```

7. Параметры (Variables)
   Переменные позволяют хранить конфиденциальные данные и конфигурации, которые могут быть использованы в задачах. Это может быть полезно для управления окружениями (например, ключи API).

- Пример определения переменной:
  ```yaml
  variables:
    NODE_ENV: "production"
  ```

8. Деплой (Deploy)
   Деплой — это процесс развертывания приложения на сервере или в облаке. GitLab CI/CD позволяет автоматизировать этот процесс.

- Пример команды для развертывания:
  ```yaml
  deploy:
    stage: deploy
    script:
      - echo "Deploying to production server..."
      - scp -r ./build/* user@server:/path/to/deploy
  ```

9. Сложные пайплайны (Complex Pipelines)
   GitLab CI/CD поддерживает сложные пайплайны, которые могут включать параллельное выполнение задач, динамическое создание задач и сложные зависимости.

- Пример параллельного выполнения:
  ```yaml
  test:
    stage: test
    parallel:
      matrix:
        - NODE_VERSION: 10
        - NODE_VERSION: 12
        - NODE_VERSION: 14
    script:
      - echo "Testing with Node.js version $NODE_VERSION"
      - npm install
      - npm test
  ```

10. Триггеры (Triggers)
    Триггеры позволяют запускать пайплайны по определенным событиям, таким как пуш в репозиторий, создание Merge Request и другие.

- Пример использования триггера:
  ```yaml
  workflow:
    rules:
      - if: "$CI_MERGE_REQUEST_ID"
  ```

11. Полный пример .gitlab-ci.yml
    Вот полный пример файла .gitlab-ci.yml, который демонстрирует использование различных концептов:

    ```yaml
    stages: - build - test - deploy

        variables:
            NODE_ENV: "production"

        cache:
            paths:
                - node_modules/

        build:
            stage: build
            script:
                - echo "Building the application..."
                - npm install
                - npm run build
            artifacts:
                paths:
                - build/

        test:
            stage: test
            script:
                - echo "Running tests..."
                - npm test

        deploy:
            stage: deploy
            script:
                - echo "Deploying to production server..."
                - scp -r ./build/* user@server:/path/to/deploy
            only:
                - master
    ```

- Отправка уведомлений на телеграм канал

  ```yaml
  stages:
  - build
  - test
  - deploy

  variables:
  TELEGRAM_BOT_TOKEN: "ВАШ_ТОКЕН_БОТА"
  TELEGRAM_CHAT_ID: "ВАШ_ID_ЧАТА"

  before_script:
  - 'export TELEGRAM_URL="https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage"'

  after_script:
  - >
      if [ "$CI_JOB_STATUS" == "success" ]; then
      curl -s -X POST $TELEGRAM_URL -d chat_id=$TELEGRAM_CHAT_ID -d text="Сборка успешна: $CI_PROJECT_NAME - $CI_COMMIT_REF_NAME"
      elif [ "$CI_JOB_STATUS" == "failed" ]; then
      curl -s -X POST $TELEGRAM_URL -d chat_id=$TELEGRAM_CHAT_ID -d text="Сборка не удалась: $CI_PROJECT_NAME - $CI_COMMIT_REF_NAME"
      fi

  build_job:
  stage: build
  script:
      - echo "Строим проект..."

  test_job:
  stage: test
  script:
      - echo "Запускаем тесты..."

  deploy_job:
  stage: deploy
  script:
      - echo "Разворачиваем проект..."
  ```
