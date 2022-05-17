---
layout: post
title: Создание универсальных приложений Nuxt.js с использованием GraphQL на Postgres
tags: [Nuxt.js, SSR, GraphQL, Postgres]
comments: false
---

Создание универсальных приложений может быть сложным, но Nuxt.js стремится упростить его. **Nuxt.js** — это производительная
и модульная структура, которая позволяет разрабатывать универсальные приложения на **Vue.js**.

**Пример/шаблон -> [nuxtjs-postgres-graphql](https://github.com/hasura/graphql-engine/tree/master/community/sample-apps/nuxtjs-postgres-graphql)**

**Что такое универсальное приложение?** Универсальное приложение, также известное как изоморфное приложение, — это
приложение, код которого может выполняться как на стороне клиента, так и на стороне сервера.

В этой статье демонстрируется универсальное приложение, использующее Nuxt.js и Hasura GraphQL Engine. Приложение
представляет собой приложение, которое отображает список авторов и их статей.

## SSR с Nuxt.js

**Nuxt.js** - это платформа на Vue.js, которая позволяет создавать приложения Vue с рендерингом на стороне сервера (SSR). В
универсальном приложении, созданном с помощью Nuxt, страницы генерируются на стороне сервера, а затем отправляются
клиенту.

Благодаря возможностям поддержки как рендеринга на стороне сервера (SSR), так и одностраничных приложений (SPA), Nuxt.js
позволяет создавать универсальные приложения.

Как это работает:

- **Страницы**: Nuxt придерживается структуры каталогов проекта. Он преобразует `.vue` файлы внутри каталога `pages` в
маршруты приложений. Например, файл `mypage.vue` соответствует **mywebsite.com/mypage**. В примере вы можете увидеть,
как структурированы страницы. Для получения дополнительной информации о структуре каталогов, которую обеспечивает Nuxt,
читайте [здесь](https://nuxtjs.org/guide/directory-structure).

- **Получение данных с сервера**: сообщество Nuxt создало модуль Apollo - клиент для работы c GraphQL.

## Реализация проекта

В этом разделе вы увидите выборку данных на стороне сервера в действии. Вы будете использовать Apollo клиент для
получения списка авторов блога и их статей.

Прежде чем идти дальше, создайте **проект Nuxt** следующим образом:

```html
npx create-nuxt-app blog-app
```

После запуска команды, **Nuxt** задает пару вопросов, чтобы помочь вам настроить проект. Выберите эти ответы:

- Programming Language - JavaScript
- Package Manager - npm or yarn (whatever you want to use)
- Rendering Mode - Universal
- Deployment Target - any of the two

Для этого проекта вам понадобится несколько пакетов, таких как "apollo", "graphql-tag" и другие. Полный список зависимостей:

```json
{
  "dependencies": {
    "@nuxtjs/apollo": "^4.0.1-rc.5",
    "apollo-cache-inmemory": "^1.6.6",
    "core-js": "^3.21.1",
    "cross-env": "^7.0.3",
    "graphql-tag": "^2.12.6",
    "nuxt": "^2.15.8"
  },
  "devDependencies": {
    "nodemon": "^2.0.15"
  }
}
```

Добавьте их в свой **.package.json** файл и запустите `npm install`.

## Настройка клиента Apollo

Следующий шаг включает в себя настройку клиента **Apollo**, чтобы вы могли выполнять операции GraphQL.

С **Nuxt.js** вы можете расширить основные функции с помощью модулей. Для этого приложения вы будете использовать модуль **Apollo** для **Nuxt**.

Перейдите в **nuxt.config.js** файл и добавьте следующий код:

```js
  modules: [
      "@nuxtjs/apollo"
  ],

  apollo: {
    cookieAttributes: {
      expires: 7,
    },
    includeNodeModules: true,
    authenticationType: "Bearer",
    errorHandler: "~/plugins/apollo-error-handler.js",
    clientConfigs: {
      default: "~/apollo/clientConfig.js",
    },
  },
```

Вначале вы регистрируете модуль **Apollo** в своем проекте. После этого вы настраиваете модуль:

- поле `expires` в `cookieAttributes` указывает, когда срок действия файла cookie истечет и он будет удален;
- поле `authenticationType` настраивает тип авторизации авторизованных запросов;
- поле `errorHandler` настраивает глобальный обработчик ошибок;
- `apollo-module` в Nuxt принимает `clientConfig` свойство, которое используется для настройки API, аутентификации и других параметров.

Создайте новую папку с именем **apollo**, а затем добавьте в нее новый файл с именем **clientConfig.js**. Откройте только что созданный файл и добавьте следующий код:

```js
import { InMemoryCache } from "apollo-cache-inmemory";

export default function (context) {
  return {
    httpLinkOptions: {
      uri: "http://localhost:8080/v1/graphql",
      credentials: "same-origin",
    },
    cache: new InMemoryCache(),
    wsEndpoint: "ws://localhost:8080/v1/graphql",
  };
}
```

В приведенном выше коде вы настраиваете эндпоинты, необходимые для выполнения запросов GraphQL.

Прежде чем двигаться дальше, вам также необходимо определить обработчик ошибок. Создайте файл
**plugins/apollo-error-handler.js** и добавьте следующий код:

```js
export default (error, nuxtContext) => {
  console.log("Global error handler");
  console.error(error);
};
```

## GraphQL запросы

Для этого приложения у вас есть два запроса:

- один, чтобы получить авторов;
- один, чтобы получить свою статью;

Внутри той же папки - **apollo** создайте новую папку **queries**. После этого создайте новый файл с именем **fetchAuthor.gql** в папке **queries**.

```html
📂 blog-app
 └ 📂 apollo
    └ 📂 queries
        └ fetchAuthor.gql
```

Добавьте этот запрос в файл **fetchAuthor.gql**:

```js
query {
  author {
    id
    name
  }
}
```

Точно так же создайте новый файл с именем **fetchArticle.gql** для хранения запроса на получение статей:

```js
query article($id: Int) {
  article(where: { author_id: { _eq: $id } }) {
    id
    title
    content
  }
}
```

Вы будете импортировать вышеуказанные запросы всякий раз, когда вам нужно использовать их на странице. С учетом
сказанного пришло время создать страницы, на которых отображаются авторы и статьи.

## Получение списка авторов

Откройте **pages/index.vue** файл и добавьте следующий код:

```vue
<template>
  <div>
    <h3>Authors</h3>
    <ul>
      <li v-for="item in author" :key="item.id">
        <nuxt-link :to="`/article/${item.id}`">
          {{ item.name }}
        </nuxt-link>
      </li>
    </ul>
  </div>
</template>

<script>
import author from '~/apollo/queries/fetchAuthor'
export default {
  apollo: {
    author: {
      prefetch: true,
      query: author
    }
  },
  head: {
    title: 'Authors of Blog'
  }
}
</script>
```

В приведенном выше коде есть запрос GraphQL - **author**, который извлекает список авторов из базы данных и обновляет свойство автора в Vue.

После этого он перебирает массив авторов и отображает каждого автора на странице. Нажав на автора, вы перейдете к его статье.

**Как это работает?**

Этот запрос GraphQL выполняется на сервере, и данные доступны в `<template>`, что позволяет отображать страницу на
стороне сервера. Тот же запрос выполняется и на стороне клиента, что делает его универсальным/изоморфным.

## Получение статьи

Создайте файл **pages/article/_id** и добавьте следующий код:

```vue
<template>
  <div v-if="article">
    <div v-for="item in article" :key="item.id">
      <h3>{{ item.title }}</h3>
      <p>{{ item.content }}</p>
      <p>
        <nuxt-link to="/">
          Home page
        </nuxt-link>
      </p>
    </div>
  </div>
</template>

<script>
import article from '~/apollo/queries/fetchArticle'
export default {
  apollo: {
    article: {
      query: article,
      prefetch: ({ route }) => ({ id: route.params.id }),
      variables() {
        return { id: this.$route.params.id }
      }
    }
  },
  head() {
    return {
      title: 'Articles by Author'
    }
  }
}
</script>
```

Глядя на имя файла, вы можете заметить подчеркивание (_). Это означает, что страница является динамической — "имя" страницы исходит от серверной части Hasura.

Если вы вернетесь к **index.vue** файлу, вы должны увидеть:

```html
<nuxt-link :to="`/article/${item.id}`">
```

"item.id" берется из базы данных, что означает, что каждая статья имеет свой URL-адрес. Nuxt (и Vue) позволяет создавать динамические страницы для таких случаев.

Кроме того, здесь вы используете реактивные параметры. Обратите внимание на следующий фрагмент кода:

```html
variables() {
    return { id: this.$route.params.id }
}
```

Всякий раз, когда изменяется параметр - идентификатор статьи, он повторно извлекает запрос и, таким образом, обновляет страницу и показывает запрошенную статью.

В остальном он работает так же, как и предыдущий запрос, который извлекает авторов.

**Nuxt.js + Hasura + модуль Apollo = универсальные приложения с GraphQL!**




