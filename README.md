# Лабораторная работа №6: Взаимодействие нескольких контейнеров

## Выполнил

* Mihailov Piotr I2302
* Дата выполнения: 05.04.25

## Цель работы

Выполнив данную работу, мы сможем управлять взаимодействием нескольких контейнеров, создавая PHP-приложение на базе `nginx` и `php-fpm`, используя Docker.

## Задание

Создать PHP-приложение на базе двух контейнеров:

* Контейнер с `nginx`
* Контейнер с `php-fpm`

## Подготовка

Для выполнения лабораторной работы необходимо:

* Установленный Docker
* Опыт выполнения лабораторной работы №3
* Наличие ранее разработанного сайта на PHP

## Выполнение

### Шаг 1: Клонирование и структура проекта

1. Создаю репозиторий `containers06`.
2. Клонирую репозиторий на локальный компьютер.

![clone](/images/gitclone.png)
3. Создаю следующую структуру директорий:

```struct
containers06/
├── mounts/
│   └── site/             # PHP-сайт, разработанный ранее
├── nginx/
│   └── default.conf      # Конфигурация nginx
├── .gitignore
└── README.md
```

### Шаг 2: Настройка `.gitignore`

Создаю файл `.gitignore` в корне проекта со следующим содержимым:

```gitignore
# Ignore files and directories
mounts/site/*
```

![ignore](/images/gitignore.png)

### Шаг 3: Конфигурация Nginx

Создаю файл `nginx/default.conf` со следующим содержимым:

```nginx
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

![conf](/images/conf.png)

### Шаг 4: Запуск и тестирование

1. Создаю сеть `internal`:

```bash
docker network create internal
```

`docker network create internal` — создаёт изолированную Docker-сеть internal для безопасного взаимодействия контейнеров без доступа в интернет.

![create](/images/dockernetwork.png)
2. Запуск контейнера `backend` (php-fpm):

Для выполнения я использую следующие команды:

```sh
docker run -d `
  --name backend `
  --network internal `
  -v ${PWD}\mounts\site:/var/www/html `
  php:7.4-fpm
```

Эта команда запускает контейнер Docker с PHP-FPM 7.4 в фоновом режиме (-d).

* name backend — называет контейнер backend
* network internal — подключает контейнер к сети internal
* -v ${PWD}\mounts\site:/var/www/html — монтирует папку ./mounts/site (относительно текущего пути) в /var/www/html внутри контейнера
* php:7.4-fpm — использует официальный образ PHP 7.4 с FPM

![image](/images/1.png)
3. Запуск контейнера `frontend` (nginx):

Для выполнения я использую следующие команды:

```sh
docker run -d `
  --name frontend `
  --network internal `
  -v ${PWD}\mounts\site:/var/www/html `
  -v ${PWD}\nginx\default.conf:/etc/nginx/conf.d/default.conf `
  -p 80:80 `
  nginx:1.23-alpine
```

Эти команды:

* Поднимает Nginx (Alpine-версия) в контейнере frontend
* Сажает его в сеть internal (чтобы видел PHP-FPM контейнер)
* Кладёт ваш код сайта в /var/www/html (общая папка с PHP-контейнером)
* Подсовывает свой конфиг Nginx (должен быть прописан proxy_pass на backend:9000)
* Открывает 80 порт наружу

![2](/images/2.png)
4. Открытие в браузере по адресу: [http://localhost](http://localhost)

После выполнения всех вышеперечисленных действий, переходим на `localhost` и видим, что все работает.

![image](/images/sitek.png)

## Ответы на вопросы

**1. Каким образом в данном примере контейнеры могут взаимодействовать друг с другом?**

Контейнеры подключены к одной и той же пользовательской сети (`internal`), что позволяет им обращаться друг к другу по именам контейнеров.

**2. Как видят контейнеры друг друга в рамках сети internal?**

В сети internal контейнеры видят друг друга по имени (--name).

Docker автоматически настраивает внутренний DNS, поэтому контейнер frontend может обратиться к backend просто по имени (например, backend:9000 для PHP-FPM), а backend — к frontend аналогично.

Это работает без дополнительных настроек, если оба контейнера подключены к одной сети.

**3. Почему необходимо было переопределять конфигурацию nginx?**

Переопределение конфига Nginx необходимо, чтобы:

* Настроить передачу PHP-запросов в контейнер backend
* Согласовать пути к файлам между Nginx и PHP-FPM

Стандартный конфиг не содержит этих настроек, из-за чего Nginx не сможет корректно обрабатывать PHP-файлы, а будет отдавать их как текст.

Это обязательный шаг при раздельном запуске Nginx и PHP-FPM в разных контейнерах.

## Выводы

Выполнив данную лабораторную работу был изученен способ взаимодействия нескольких контейнеров через общую сеть Docker.

Также была освоена найстройка взаимодействия nginx и php-fpm в разных контейнерах, а также закреплены знания монтирования директорий и настройки конфигурационных файлов.

## Библиография

* [Repository by M.Croitor](https://github.com/mcroitor/app_containerization_ru/commits?author=mcroitor)
* [Docker Docs – Networking Overview](https://docs.docker.com/network/)
* [Docker Hub php:7.4-fpm](https://hub.docker.com/_/php)
* [Руководство про nginx](https://hub.docker.com/_/nginx)
