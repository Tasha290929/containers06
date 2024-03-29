# Лабораторная работа № 6 Создание многоконтейнерного приложения

## Цель работы
Ознакомиться с работой многоконтейнерного приложения на базе docker-compose.

## Задание
Создать php приложение на базе двух контейнеров: nginx, php-fpm, mariadb.

## Подготовка
Для выполнения данной работы необходимо иметь установленный на компьютере Docker.

Работа выполняется на базе лабораторной работы №5.


## Выполнение
Создайте репозиторий `containers06` и скопируйте его себе на компьютер.

В директории `containers06` создайте директорию `mounts/site`. В данную директорию перепишите сайт на php, созданный в рамках предмета по php.

Создайте файл `.gitignore` в корне проекта и добавьте в него строки:
```
# Ignore files and directories
mounts/site/*
Создайте в директории containers06 файл nginx/default.conf со следующим содержимым:

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
Создайте в директории containers06 файл docker-compose.yml со следующим содержимым:

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
Создайте файл mysql.env в корне проекта и добавьте в него строки:
```
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=app
MYSQL_USER=user
MYSQL_PASSWORD=secret
Запуск и тестирование
Запустите контейнеры командой:

docker-compose up -d
```
Проверьте работу сайта в браузере, перейдя по адресу `http://localhost`. Если отображается базовая страница nginx, то перегрузите страницу.

![](./mounts/site/img/Screenshot%202024-03-29%20162239.png)

## Ответьте на вопросы:

* В каком порядке запускаются контейнеры?

```
 Container containers06-database-1
 Container containers06-frontend-1 
 Container containers06-backend-1   
```

* Где хранятся данные базы данных?

База данных храниться в томе db_data

* Как называются контейнеры проекта?

`frontend` - контейнер с веб-сервером Nginx.
`backend` - контейнер с PHP-FPM.
`database` - контейнер с базой данных MariaDB.

* Вам необходимо добавить еще один файл `app.env` с переменной окружения `APP_VERSION` для сервисов `backend` и `frontend`. Как это сделать?
Создаем в главной директории файл `app.env` с переменной `APP_VERSION=1.0`. В файл `docker-compose.yml` добавляем в `backend` и `frontend` 
 ```
 env_file:
      - app.env
      ```