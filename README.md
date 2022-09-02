[![Django-app workflow](https://github.com/mspitsyn/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)](https://github.com/mspitsyn/yamdb_final/actions/workflows/yamdb_workflow.yml)
# CI/CD для проекта API YAMDB
В проекте описываются следующиезадачи:
- автоматический запуск тестов,
- обновление образов на Docker Hub,
- автоматический деплой на боевой сервер при пуше в главную ветку main.
---

## Workflow
* tests - Проверка кода на соответствие стандарту PEP8 (с помощью пакета flake8) и запуск pytest 
* build_and_push_to_docker_hub - Сборка и доставка докер-образов на Docker Hub
* deploy - Автоматический деплой проекта на боевой сервер
* send_message - Отправка уведомления в Telegram о том, что процесс деплоя успешно завершился.

### Подготовка для запуска workflow
Создайте и активируйте виртуальное окружение, обновите pip:
```
python3 -m venv venv
. venv/bin/activate
python3 -m pip install --upgrade pip
```
Запустите автотесты:
```
pytest
```
Отредактируйте файл `nginx/default.conf` и в строке `server_name` впишите IP виртуальной машины (сервера).  
Скопируйте подготовленные файлы `docker-compose.yaml` и `nginx/default.conf` из вашего проекта на сервер:
Создайте папку `nginx` на сервере:
```
mkdir nginx
```
Зайдите в директорию где находятся файлы на локальной машине и отправьте файлы на сервер.

```
scp docker-compose.yaml <username>@<host>:/home/<username>/
scp default.conf <username>@<host>:/home/<username>/nginx/
```
В репозитории проекта на Гитхабе добавьте данные в `Settings - Secrets - Actions secrets`:
```
DOCKER_USERNAME - имя пользователя в DockerHub
DOCKER_PASSWORD - пароль пользователя в DockerHub
HOST - ip-адрес сервера
USER - пользователь сервера
SSH_KEY - приватный ssh-ключ (публичный должен быть на сервере)
PASSPHRASE - кодовая фраза для ssh-ключа
DB_ENGINE - django.db.backends.postgresql
DB_HOST - db
DB_PORT - 5432
SECRET_KEY - секретный ключ приложения django
DB_NAME - postgres (по умолчанию)
POSTGRES_USER - postgres (по умолчанию)
POSTGRES_PASSWORD - postgres (по умолчанию)
TELEGRAM_TO - id своего телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
TELEGRAM_TOKEN - токен бота (получить токен можно у @BotFather, /token, имя бота)
```

## Как запустить проект на сервере:

Установите Docker и Docker-compose на сервере:
```
sudo apt install docker.io
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
Проверьте корректность установки Docker-compose:
```
sudo docker-compose --version
```

### После успешного деплоя:
Соберите статические файлы (статику):
```
sudo docker-compose exec web python manage.py collectstatic --no-input
```
Примените миграции:
```
sudo docker-compose exec web python manage.py makemigrations
sudo docker-compose exec web python manage.py migrate --noinput
```
Создайте суперпользователя:
```
sudo docker-compose exec web python manage.py createsuperuser
```
или
```
sudo docker-compose exec web python manage.py loaddata fixtures.json
```


### Проверьте работоспособность приложения:
зайдите на http://<ip-адрес>/admin/ и убедитесь, что страница отображается полностью: статика подгрузилась.

### Как пользоваться ресурсом
После запуска правила пользования доступны по адресу http://<ip-адрес>/redoc/.

---

### Автор
[Спицын Максим](https://github.com/mspitsyn) 
