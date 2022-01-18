---
layout: post
title: Как настроить HTTPS локально для Express.js (Node.js)?
tags: [Node.js, Express.js]
comments: false
---

Если вы запускаете свой бэкенд Node.js локально, то по умолчанию он работает по протоколу HTTP. В некоторых случаях
требуется, чтобы ваш бэкенд работал по протоколу https для интеграции таких сервисов, как Azure B2C или подобных. Эта
статья о том, как настроить Express.js на обслуживание бэкенда Node.js по https при локальной разработке. Давайте
посмотрим, как этого можно добиться в Node.js.

Чтобы использовать https локально, мы должны сделать следующее:

1. Сгенерировать локальный центр сертификации и SSL-сертификат
2. Установить сертификаты для работы бэкенда Node.js по HTTPS
3. протестировать!

## Пользовательский SSL-сертификат

Вы должны создать локальный центр сертификации и SSL сертификат, установить `SSL_CERT_FILE` и `SSL_KEY_FILE` в
сгенерированные файлы.

## Генерация SSL-сертификата

В качестве первого шага необходимо сгенерировать локальный центр сертификации и SSL-сертификат для локальной разработки.

1. Для установки `mkcert` вам понадобится менеджер пакетов:

   - MacOS: Используйте Homebrew или Macports.
   - Linux: Используйте certutil. Только для Arch Linux, mkcert доступен в репозитории Arch Linux.
   - Windows: Используйте chocolatey.

2. Установите [mkcert](https://github.com/FiloSottile/mkcert).
3. Создайте локальный доверенный CA с помощью `mkcert -install`.
4. Сгенерируйте SSL-сертификат с помощью `mkcert localhost`.

## Установка пользовательского SSL-сертификата

Чтобы приложение Express.js работало локально с SSL, мы должны обновить объект options - свойства `key` и `cert`.
Следовательно, после генерации локального центра сертификации и ssl-сертификата мы должны установить в свойствах `key` и
`cert` путь к файлам сертификата и ключа.

Рассмотрим простой пример Express сервера. Переменные `CERT-PATH` и `KEY-PATH` - это пути к сгенерированным файлам.

Создайте или добавьте папку проекта.

{% highlight html %}
mkdir node-ssl-test
{% endhighlight %}

Инициализируйте проект с помощью npm `init -y`, чтобы иметь возможность установить пакеты node.

{% highlight html %}
cd node-ssl-test
npm init -y
{% endhighlight %}

Установка Express.

{% highlight html %}
npm install express
{% endhighlight %}

Создайте `index.js`.

{% highlight html %}
touch index.js
{% endhighlight %}

Скопируйте пример кода.

{% highlight js %}
const https = require('https');
const fs = require('fs');
const express = require('express');

const app = express();

const options = {
  key: fs.readFileSync(CERT_PATH),
  cert: fs.readFileSync(KEY_PATH),
};

app.use((req, res, next) => {
  res.send('<h1>HTTPS is working!</h1>');
});

const port = 3000;

https.createServer(options, app).listen(port, () => {
  console.log('Server listening on port ' + port);
});
{% endhighlight %}

Теперь запусттите `index.js` коммандой `node index.js` и откройте страницу в браузере `https://localhost:3000`, HTTPS должен теперь заработать!. Вы также можете проверить сертификат в инструментах разработчика браузера (Chrome -> Security Tab или по клику на иконке замка).