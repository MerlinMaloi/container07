# Лабораторная работа №7

## Цель работы

Выполнив данную работу студент сможет управлять взаимодействием нескольких контейнеров.

## Задание

Создать php приложение  на базе трех контейнеров: `nginx`, `php-fpm`, `mariadb`, используя `docker-compose`


## Выполнение 

### 1. Создание репозиория и директории

1. Создал репозиторий `containers07` и скопировал его себе на компьютер.

2. В директории `containers07` создаk директорию `mounts/site`. В данную директорию скопировал сайт на php, созданный в рамках предмета по php.

3. Создал файл `.gitignore` в корне проекта и записал в него строки:

```
# Ignore files and directories
mounts/site/*
```

4. Создал в директории `containers07` файл `nginx/default.conf` со следующим содержимым:

```
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

5.  Создал в директории `containers07` файл `docker-compose.yml` со следующим содержимым:

```
version: '3.9'

services:
  frontend:
    image: nginx:1.19
    volumes:
      - ./mounts/site:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "80:80"
    networks:
      - internal
  backend:
    image: php:7.4-fpm
    volumes:
      - ./mounts/site:/var/www/html
    networks:
      - internal
    env_file:
      - mysql.env
  database:
    image: mysql:8.0
    env_file:
      - mysql.env
    networks:
      - internal
    volumes:
      - db_data:/var/lib/mysql

networks:
  internal: {}

volumes:
  db_data: {}

```


6.  Создал в директории `containers07` файл `mysql.env` : 

```
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=app
MYSQL_USER=user
MYSQL_PASSWORD=secret
```

### 2. Запуск и тестирование

Команда в терминале из корня проекта:

`docker-compose up -d`


### 3. Ответы на вопросы:

1. В каком порядке запускаются контейнеры?

- `backend`

- `frontend`

- `database`

2. Где хранятся данные базы данных?

В томе `db_data: {}` так как мы указали в файле `docker-compose.yml`:

```
volumes:
  db_data: {}
```

3. Как называются контейнеры проекта?

- container07-backend-1 
- container07-frontend-1
- container07-database-1

4. Вам необходимо добавить еще один файл `app.env`с переменной окружения APP_VERSION для сервисов backend и frontend. Как это сделать?

Создать файл `app.env` в нужной директории в нем объявить переменную окружения APP_VERSION, дальше в файле `docker-compose.yml` в контейнерах `backend` и `frontend` указать переменную окружения:

```
env_file:
    - app.env
```

### 4. Выводы 

В ходе лабараторной был создан файл `docker-compose.yml`, который упростил процесс запуска контейнеров для поднятия приложения на php , обьединяя все настройки, которые в предыдущей лабараторной было необходимо писать в терминале + поднятие базы данных. 