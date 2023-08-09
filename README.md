#  Проект Kittygram

Kittygram — социальная сеть для обмена фотографиями любимых питомцев. Это полностью рабочий проект, который состоит из бэкенд-приложения на Django и фронтенд-приложения на React.

# Способ установки

Клонируем репозиторий:

    ```bash
    git clone git@github.com:meveladron/kittygram_final.git
    ```

Переходим в директорию kittygram:

    ```bash
    cd kittygram
    ```

В данной директории создаём файл .env. Все необходимые данные для заполнения указаны в файле .env_example

Создаём docker образы:

    ```bash
    cd frontend
    docker build -t <_имя_пользователя_dockerhub_>/kittygram_frontend .
    cd ../backend
    docker build -t <_имя_пользователя_dockerhub_>/kittygram_backend .
    cd ../nginx
    docker build -t <_имя_пользователя_dockerhub_>/kittygram_gateway . 
    ```

Выгружаем образы на DockerHub:

    ```bash
    docker push <_имя_пользователя_dockerhub_>/kittygram_frontend
    docker push <_имя_пользователя_dockerhub_>/kittygram_backend
    docker push <_имя_пользователя_dockerhub_>/kittygram_gateway
    ```

Подключаемся к серверу через ssh:

    ```bash
    ssh -i <_путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера_>
    ```

Создаём на сервере директорию kittygram:

    ```bash
    mkdir kittygram
    ```

Устанавливаем Docker Compose:

    ```bash
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt-get install docker-compose-plugin
    ```

На сервере создаём файлы docker-compose.production.yml и .env и копируем всё содержимое из локальных аналогичных файлов:

    ```bash
    sudo nano docker-compose.production.yml
    sudo nano .env
    ```

Запусrftv Docker Compose в режиме демона:

    ```bash
    sudo docker compose -f docker-compose.production.yml up -d
    ```

Проводим миграции, соберите статику бекенда:

    ```bash
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

На сервере открываем конфиг Nginx:

    ```bash
    sudo nano /etc/nginx/sites-enabled/default
    ```

Меняем настройки location на представленные ниже:

    ```bash
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
    ```

Проверяем работоспособность конфига и перезапускаем Nginx:

    ```bash
    sudo nginx -t
    sudo service nginx reload
    ```

# Настройка CI/CD

В проекте используются приведённые ниже секреты. Для внедрения на своем сервере добавьте секреты в GitHub Actions:

    ```bash
    DOCKER_USERNAME - логин в DockerHub
    DOCKER_PASSWORD - пароль в DockerHub
    HOST - IP сервера
    USER - имя пользователя
    SSH_KEY - ключ ssh
    SSH_PASSPHRASE - пароль для ssh-ключа
    TELEGRAM_TO - id телеграм-аккаунта
    TELEGRAM_TOKEN - токен бота telegram

#  Как работать с репозиторием финального задания

## Что нужно сделать

Настроить запуск проекта Kittygram в контейнерах и CI/CD с помощью GitHub Actions

## Как проверить работу с помощью автотестов

В корне репозитория создайте файл tests.yml со следующим содержимым:
```yaml
repo_owner: ваш_логин_на_гитхабе
kittygram_domain: полная ссылка (https://доменное_имя) на ваш проект Kittygram
taski_domain: полная ссылка (https://доменное_имя) на ваш проект Taski
dockerhub_username: ваш_логин_на_докерхабе
```

Скопируйте содержимое файла `.github/workflows/main.yml` в файл `kittygram_workflow.yml` в корневой директории проекта.

Для локального запуска тестов создайте виртуальное окружение, установите в него зависимости из backend/requirements.txt и запустите в корневой директории проекта `pytest`.

## Чек-лист для проверки перед отправкой задания

- Проект Taski доступен по доменному имени, указанному в `tests.yml`.
- Проект Kittygram доступен по доменному имени, указанному в `tests.yml`.
- Пуш в ветку main запускает тестирование и деплой Kittygram, а после успешного деплоя вам приходит сообщение в телеграм.
- В корне проекта есть файл `kittygram_workflow.yml`.
