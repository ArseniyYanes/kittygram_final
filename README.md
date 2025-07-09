# Kittygram — социальная сеть для обмена фотографиями питомцев

### О проекте
Kittygram — это платформа, где пользователи могут регистрироваться, загружать фотографии своих любимых питомцев с описаниями и просматривать фотографии питомцев других участников.

### Установка
*Примечание: примеры приведены для Linux-систем*

1. Клонируйте репозиторий на ваш компьютер:

    ```
    git clone git@github.com:1emd/kittygram_final.git
    ```

2. Создайте файл `.env` и заполните его своими параметрами. Все необходимые переменные перечислены в файле `.env.example`, расположенном в корне проекта.

### Сборка Docker-образов

1. Замените `YOUR_USERNAME` на ваш DockerHub логин:

    ```
    cd frontend
    docker build -t YOUR_USERNAME/kittygram_frontend .
    cd ../backend
    docker build -t YOUR_USERNAME/kittygram_backend .
    cd ../nginx
    docker build -t YOUR_USERNAME/kittygram_gateway .
    ```

2. Отправьте собранные образы в DockerHub:

    ```
    docker push YOUR_USERNAME/kittygram_frontend
    docker push YOUR_USERNAME/kittygram_backend
    docker push YOUR_USERNAME/kittygram_gateway
    ```

### Развёртывание на сервере

1. Подключитесь к удалённому серверу:

    ```
    ssh -i PATH_TO_SSH_KEY/SSH_KEY_NAME YOUR_USERNAME@SERVER_IP_ADDRESS
    ```

2. Создайте на сервере каталог `kittygram`:

    ```
    mkdir kittygram
    ```

3. Установите Docker Compose:

    ```
    sudo apt update
    sudo apt install curl
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    sudo apt install docker-compose
    ```

4. Перенесите файлы `docker-compose.production.yml` и `.env` в директорию `kittygram/` на сервере:

    ```
    scp -i PATH_TO_SSH_KEY/SSH_KEY_NAME docker-compose.production.yml YOUR_USERNAME@SERVER_IP_ADDRESS:/home/YOUR_USERNAME/kittygram/docker-compose.production.yml
    ```

    Пояснения:
    - `PATH_TO_SSH_KEY` — путь к SSH-ключу
    - `SSH_KEY_NAME` — имя файла с SSH-ключом
    - `YOUR_USERNAME` — имя пользователя на сервере
    - `SERVER_IP_ADDRESS` — IP-адрес сервера

5. Запустите контейнеры с помощью Docker Compose в фоне:

    ```
    sudo docker-compose -f /home/YOUR_USERNAME/kittygram/docker-compose.production.yml up -d
    ```

6. Выполните миграции, соберите статику и скопируйте её в `/backend_static/static/`:

    ```
    sudo docker-compose -f /home/YOUR_USERNAME/kittygram/docker-compose.production.yml exec backend python manage.py migrate
    sudo docker-compose -f /home/YOUR_USERNAME/kittygram/docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker-compose -f /home/YOUR_USERNAME/kittygram/docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

7. Откройте файл конфигурации Nginx для редактирования:

    ```
    sudo nano /etc/nginx/sites-enabled/default
    ```

8. В секции `server` измените блок `location` следующим образом:

    ```
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
    ```

9. Проверьте корректность конфигурации Nginx:

    ```
    sudo nginx -t
    ```

    Если вывод такой:

    ```
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```

    — значит конфигурация верна.

10. Перезапустите Nginx для применения изменений:

    ```
    sudo service nginx reload
    ```

### Настройка CI/CD

1. Файл workflow для GitHub Actions уже создан и расположен в:

    ```
    kittygram/.github/workflows/main.yml
    ```

2. Для корректной работы адаптируйте workflow, добавив секреты в GitHub Actions:

    ```
    DOCKER_USERNAME                # ваш логин в DockerHub
    DOCKER_PASSWORD                # пароль от DockerHub
    HOST                           # IP вашего сервера
    USER                           # имя пользователя на сервере
    SSH_KEY                        # содержимое приватного SSH-ключа (cat ~/.ssh/id_rsa)
    SSH_PASSPHRASE                 # пароль для SSH-ключа

    TELEGRAM_TO                    # ваш Telegram ID (узнать можно у @userinfobot, команда /start)
    TELEGRAM_TOKEN                 # токен Telegram-бота (получить у @BotFather, команда /token)
    ```

### Используемые технологии
- Python 3.11.1  
- Django 3.2.3  
- djangorestframework 3.12.4  
- PostgreSQL 13.10  

### Автор
Руссу Арсений Дмитриевич
Python-разработчик (Backend)
E-mail: y4ne242@gmail.com
