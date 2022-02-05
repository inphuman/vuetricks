---
layout: post
title: Настройка Docker Compose для Express.js и MongoDB
tags: [Node.js, Express.js, MongoDB]
comments: false
---

Docker предоставляет легковесные контейнеры для запуска сервисов в изоляции от нашей инфраструктуры, чтобы мы могли
быстро разворачивать и запускать программное обеспечение. В этом руководстве показано, как запустить на Docker
Express.js и MongoDB с помощью Docker Compose.

## Структура Express.js и MongoDB в Docker

Предположим, что у нас есть Nodejs-приложение, работающее с базой данных MongoDB. Проблема заключается в контейнеризации
системы, для которой требуется более одного контейнера Docker:
- Node.js Express для API
- MongoDB для базы данных

Docker Compose поможет нам настроить систему проще и эффективнее, чем при использовании только Docker. Мы будем
следовать следующим шагам:

- Создаем приложение Nodejs, работающее с базой данных MongoDB.
- Создаем Dockerfile для Nodejs приложения.
- Записываем конфигурацию Docker Compose в YAML-файл.
- Устанавливаем переменные окружения для Docker Compose.
- Запустите систему.

Структура директорий:

![Структура директорий Docker для Express.js]({{ site.baseurl }}/assets/img/posts/2022-02-05-docker-compose-nodejs-mongodb.png)

Во-первых, давайте добавим модуль dotenv в package.json.

{% highlight js %}
{
  ...
  "dependencies": {
    "dotenv": "^10.0.0",
    ...
  }
}
{% endhighlight %}

Далее мы импортируем dotenv в server.js и используем process.env для установки порта.

{% highlight js %}
require("dotenv").config();
..
// set port, listen for requests
const PORT = process.env.NODE_DOCKER_PORT || 8080;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}.`);
});
{% endhighlight %}

Затем мы изменяем конфигурацию и инициализацию базы данных.

`app/config/db.config.js`

{% highlight js %}
const {
  DB_USER,
  DB_PASSWORD,
  DB_HOST,
  DB_PORT,
  DB_NAME,
} = process.env;
module.exports = {
  url: `mongodb://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}?authSource=admin`
};
{% endhighlight %}

Нам также нужно сделать файл-образец .env, в котором будут указаны все необходимые аргументы.

`.env.sample`

{% highlight js %}
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=123456
DB_NAME=bezkoder_db
DB_PORT=27017
NODE_DOCKER_PORT=8080
{% endhighlight %}

## Создание Dockerfile для приложения Nodejs

Dockerfile определяет список команд, которые Docker использует для настройки среды приложения Node.js. Поэтому мы помещаем файл в папку приложения.

Поскольку мы будем использовать Docker Compose, мы не будем определять все команды конфигурации в этом Dockerfile.

`Dockerfile`

{% highlight js %}
FROM node:14
WORKDIR /bezkoder-app
COPY package.json .
RUN npm install
COPY . .
CMD npm start
{% endhighlight %}

Объяснение некоторых моментов:

- FROM: установка образа версии Node.js.
- WORKDIR: путь к рабочему каталогу.
- COPY: скопировать файл package.json в контейнер, затем второй копией скопировать все файлы внутри директории проекта.
- RUN: выполнить командную строку внутри контейнера: npm install для установки зависимостей в package.json.
- CMD: запустить скрипт npm start после сборки образа.

## Создание конфигурации Docker Compose

В корне каталога проекта мы создадим файл docker-compose.yml:

{% highlight js %}
version: '3.8'
services: 
    mongodb:
    app:
volumes:
{% endhighlight %}

- version: Будет использоваться версия формата файла Docker Compose.
- services: отдельные сервисы в изолированных контейнерах. Наше приложение имеет два сервиса: app (Nodejs) и mongodb (база данных MongoDB).
- volumes: именованные тома, которые сохраняют наши данные после перезапуска.

Давайте реализуем детали.

`docker-compose.yml`

{% highlight js %}
version: "3.8"
services:
  mongodb:
    image: mongo:5.0.2
    restart: unless-stopped
    env_file: ./.env
    environment:
      - MONGO_INITDB_ROOT_USERNAME=$MONGODB_USER
      - MONGO_INITDB_ROOT_PASSWORD=$MONGODB_PASSWORD
    ports:
      - $MONGODB_LOCAL_PORT:$MONGODB_DOCKER_PORT
    volumes:
      - db:/data/db
  app:
    depends_on:
      - mongodb
    build: ./bezkoder-app
    restart: unless-stopped
    env_file: ./.env
    ports:
      - $NODE_LOCAL_PORT:$NODE_DOCKER_PORT
    environment:
      - DB_HOST=mongodb
      - DB_USER=$MONGODB_USER
      - DB_PASSWORD=$MONGODB_PASSWORD
      - DB_NAME=$MONGODB_DATABASE
      - DB_PORT=$MONGODB_DOCKER_PORT
    stdin_open: true
    tty: true
volumes:
  db:
{% endhighlight %}

**mongodb**:

- image: официальный образ Docker
- restart: настройка политики перезапуска
- env_file: указать путь к .env, который мы создадим позже
- environment: обеспечить настройку с помощью переменных окружения
- ports: указать порты, которые будут использоваться
- volumes: отобразить папки томов

**app**:

- depends_on: порядок зависимостей, mongodb запускается раньше app
- build: параметры конфигурации, применяемые во время сборки, которые мы определили в Dockerfile с относительным путем
- environment: переменные окружения, которые использует приложение Node
- stdin_open и tty: держать открытым терминал после сборки контейнера

Следует отметить, что порт хоста (LOCAL_PORT) и порт контейнера (DOCKER_PORT) отличаются. Сетевая связь между сервисами использует порт контейнера, а внешняя - порт хоста.

В конфигурации сервиса мы использовали переменные окружения, определенные в файле .env. Теперь мы начнем его писать.

{% highlight js %}
MONGODB_USER=root
MONGODB_PASSWORD=123456
MONGODB_DATABASE=bezkoder_db
MONGODB_LOCAL_PORT=7017
MONGODB_DOCKER_PORT=27017
NODE_LOCAL_PORT=6868
NODE_DOCKER_PORT=8080
{% endhighlight %}

## Запуск Nodejs MongoDB с помощью Docker Compose

Мы можем легко запустить все с помощью всего одной команды:

`docker-compose up`

Docker подтянет образы MongoDB и Node.js (если на нашей машине их еще не было).

Сервисы могут быть запущены в фоновом режиме с помощью команды:

`docker-compose up -d`

{% highlight html %}
$ docker-compose up -d
Creating network "node-mongodb_default" with the default driver
Creating volume "node-mongodb_db" with default driver
Pulling mongodb (mongo:5.0.2)...
5.0.2: Pulling from library/mongo
16ec32c2132b: Pull complete
6335cf672677: Pull complete
cbc70ccc8ebe: Pull complete
0d1a3c6bd417: Pull complete
960f3b9b27d3: Pull complete
aff995a136b4: Pull complete
4249be7550a8: Pull complete
cc105ff5aa3c: Pull complete
82819807d07a: Pull complete
81447d2c233f: Pull complete
Digest: sha256:54d24682d00278f64bf21ff62b7ee62b59dae50f65139831a884b345922b0f8a
Status: Downloaded newer image for mongo:5.0.2
Building app
Sending build context to Docker daemon  19.46kB
Step 1/6 : FROM node:14
14: Pulling from library/node
eb18d230e067: Pull complete
83600c1b4583: Pull complete
4ae15c65bfa0: Pull complete
c19c058edda5: Pull complete
05cdaa0fd103: Pull complete
8461dcff50c4: Pull complete
84be4117f79d: Pull complete
4ccb803887fd: Pull complete
ab52680a5469: Pull complete
Digest: sha256:c1fa7759eeff3f33ba08ff600ffaca4558954722a4345653ed1a0d87dffed9aa
Status: Downloaded newer image for node:14
---> 256d6360f157
Step 2/6 : WORKDIR /bezkoder-app
---> Running in 71b4b2e9dd6c
Removing intermediate container 71b4b2e9dd6c
---> 194372d3695c
Step 3/6 : COPY package.json .
---> 093f866b404a
Step 4/6 : RUN npm install
---> Running in 025f0f0365a9
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN node-express-mongodb@1.0.0 No repository field.
added 81 packages from 128 contributors and audited 81 packages in 6.902s
2 packages are looking for funding
run `npm fund` for details
found 0 vulnerabilities
Removing intermediate container 025f0f0365a9
---> 2f04aeaa93b1
Step 5/6 : COPY . .
---> 50e31a045a02
Step 6/6 : CMD npm start
---> Running in 7353ac17fa02
Removing intermediate container 7353ac17fa02
---> bd9d66054ea2
Successfully built bd9d66054ea2
Successfully tagged node-mongodb_app:latest
WARNING: Image for service app was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating node-mongodb_mongodb_1 ... done
Creating node-mongodb_app_1     ... done
{% endhighlight %}


Теперь вы можете проверить текущие рабочие контейнеры:

{% highlight html %}
$ docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS         PORTS                                         NAMES
42b9271dd73f   node-mongodb_app   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:6868->8080/tcp, :::6868->8080/tcp     node-mongodb_app_1
e17bf545c0ba   mongo:5.0.2        "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:7017->27017/tcp, :::7017->27017/tcp   node-mongodb_mongodb_1
{% endhighlight %}

И образы Docker:

{% highlight html %}
$ docker images
REPOSITORY            TAG            IMAGE ID       CREATED         SIZE
node-mongodb_app      latest         bd9d66054ea2   5 minutes ago   960MB
node                  14             256d6360f157   6 minutes ago   944MB
mongo                 5.0.2          269b735e72cb   6 minutes ago   682MB
{% endhighlight %}

Далее для теста можно отправить HTTP-запрос для проверки Express Rest API и подключение к базе данных MongoDB.

## Отсановка работы приложения

Остановить все запущенные контейнеры также просто с помощью одной команды:

`docker-compose down`

{% highlight html %}
$ docker-compose down
Stopping node-mongodb_app_1     ... done
Stopping node-mongodb_mongodb_1 ... done
Removing node-mongodb_app_1     ... done
Removing node-mongodb_mongodb_1 ... done
Removing network node-mongodb_default
{% endhighlight %}

Если вам нужно остановить и удалить все контейнеры, сети и все образы, используемые какой-либо службой в файле docker-compose.yml, используйте команду:

`docker-compose down --rmi all`

{% highlight html %}
$ docker-compose down --rmi all
Stopping node-mongodb_app_1     ... done
Stopping node-mongodb_mongodb_1 ... done
Removing node-mongodb_app_1     ... done
Removing node-mongodb_mongodb_1 ... done
Removing network node-mongodb_default
Removing image mongo:5.0.2
Removing image node-mongodb_app
{% endhighlight %}

## Заключение

Сегодня мы успешно создали файл Docker Compose для приложения Nodejs и MongoDB. Теперь мы можем развернуть Nodejs Express и MongoDB с помощью Docker очень простым способом c помощью docker-compose.yml.