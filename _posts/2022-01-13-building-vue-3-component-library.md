---
layout: post
title: Создание библиотеки компонентов Vue 3
tags: [Vue.js, библиотека компонентов]
comments: false
---

Библиотеки компонентов - это отличный способ собрать общие решения пользовательского интерфейса в пакет, который можно
повторно использовать в собственных проектах или поделиться с сообществом разработчиков Vue с открытым исходным кодом.

![Создание библиотеки компонентов Vue 3]({{ site.baseurl }}/assets/img/posts/2022-01-13-building-vue-3-component-library.png)

В этом руководстве вы узнаете основные шаги по созданию библиотеки компонентов Vue 3, включая:

- Настройка библиотеки компонентов в Vue 3
- Создание плагина для установки библиотеки компонентов в проект
- Публикация библиотеки на npm
- Создание документации с помощью VuePress
- Публикация документации на GitHub

**Давайте начнем!**

## Базовая настройка

Для создания нашей библиотеки компонентов мы не будем использовать обычный процесс создания проектов Vue.js, т.е. Vue
CLI. Вместо этого мы создадим папку проекта с нуля, где инициализируем npm и Git.

{% highlight html %}
$ mkdir vue3-component-library
$ cd vue3-component-library
$ npm init -y
$ git init
$ touch .gitignore
$ echo 'node_modules' >> .gitignore
{% endhighlight %}

К концу этого поста мы получим структуру папок, которая будет выглядеть следующим образом:

{% highlight html %}
.gitignore
package.json
rollup.config.js
dist/
  library.mjs
  library.js
docs/
  .vuepress
  components
node_modules/
src/
  InputText.vue
  InputTextarea.vue
{% endhighlight %}

## Создание компонентов в Vue

Библиотеке компонентов, конечно же, понадобятся некоторые компоненты. Давайте создадим папку src и добавим следующие два
простых компонента Vue 3:

{% highlight html %}
src/InputText.vue
{% endhighlight %}

{% highlight html %}
<template>
    <input type="text" />
</template>
<script>
export default {
  name: 'InputText'
}
</script>
{% endhighlight %}

{% highlight html %}
src/InputTextarea.vue
{% endhighlight %}

{% highlight html %}
<template>
    <textarea />
</template>
<script>
export default {
  name: 'InputTextarea'
}
</script>
{% endhighlight %}

## Создание плагина

Далее мы создадим файл, в котором зарегистрируем все компоненты, которые мы хотим использовать в нашей библиотеке. Мы
назовем его components.js и в нем мы просто импортируем компоненты, а затем экспортируем их в один объект.

{% highlight html %}
src/components.js
{% endhighlight %}

{% highlight js %}
import InputText from './InputText.vue'
import InputTextarea from './InputTextarea.vue'

export default { InputTextarea, InputText }
{% endhighlight %}

Сейчас мы создадим плагин Vue 3, который будет глобально регистрировать компоненты из вашей библиотеки в другом проекте.

В верхней части файла мы импортируем зарегистрированные компоненты. Затем в методе установки плагина мы выполним
итерацию объекта компонентов и глобально зарегистрируем каждый компонент на экземпляре Vue.

{% highlight html %}
src/index.js
{% endhighlight %}

{% highlight js %}
import components from'./components'

const plugin = {
    install (Vue) {
        for (const prop in components) {
            if (components.hasOwnProperty(prop)) {
                const component = components[prop]
                Vue.component(component.name, component)
            }
        }
    }
}

export default plugin
{% endhighlight %}

## Создание плагина

Теперь нам нужно создать сборку нашей библиотеки, которая будет доступна в модуле npm. Для этого мы будем использовать
Rollup bundler в сочетании с плагином Vue Rollup и Rollup Plugin Peer Deps External. Используя их, мы сможем легко
создать эффективную сборку для нескольких окружений:

{% highlight html %}
$ npm i -D rollup rollup-plugin-vue rollup-plugin-peer-deps-external
{% endhighlight %}

Давайте теперь настроим Rollup, создав файл `rollup.config.js` в корне проекта. Чтобы обеспечить достаточную гибкость,
вы, вероятно, захотите сделать разные сборки для этих сценариев:

- Использование в качестве ES-модуля (для проектов на базе Vite)
- Использование в качестве модуля CommonJS (для проектов на основе webpack)
- Сборка для браузера
- Сборка для рендеринга на стороне сервера

Ниже показано, как настроить сборки модулей CommonJS и ES. Подробности для других типов сборок можно найти в документации Vue Rollup.

Обратите внимание, что мы добавляем плагин vue, который компилирует наши шаблоны компонентов, и плагин peerDepsExternal,
который автоматически экстернализирует одноранговые зависимости (т.е. Vue 3), чтобы они не были включены в вашу сборку.

{% highlight html %}
rollup.config.js
{% endhighlight %}


{% highlight js %}
import vue from 'rollup-plugin-vue'
import peerDepsExternal from 'rollup-plugin-peer-deps-external'

export default [
  {
    input: 'src/index.js',
    output: [
      {
        format: 'esm',
        file: 'dist/library.mjs'
      },
      {
        format: 'cjs',
        file: 'dist/library.js'
      }
    ],
    plugins: [
      vue(), peerDepsExternal()
    ]
  }
]
{% endhighlight %}

Чтобы запустить сборку, мы создадим сценарий сборки в файле `package.json`.

{% highlight js %}
{
    "scripts": {
      "build": "rollup -c"
    }
}
{% endhighlight %}

Когда мы запустим `npm run build`, вы увидите два новых созданных файла, `dist/library.mjs` и `dist/library.js`, которые
являются вашими сборками ES-модуля и CommonJS, соответственно.

## Публикация плагина

Чтобы распространить нашу библиотеку компонентов, мы опубликуем ее в реестре npm. Первый шаг - убедиться, что ваш файл
`package.json` содержит конфигурацию, необходимую для публикации. Вам понадобятся следующие основные значения:

- `name`: оно должно быть уникальным для всех npm, поэтому, возможно, добавьте к нему префикс `@yourname/`
- `version`: начинайте с 0.0.1 (или где вам больше нравится) и увеличивайте каждый раз, когда обновляете библиотеку. Узнайте больше о семантическом версионировании [здесь](https://docs.npmjs.com/about-semantic-versioning) 
- `main`: это "входной файл" для доступа к вашему пакету. Он должен указывать на файл сборки CommonJS
- `module`: То же, что и main, но должен указывать на файл сборки модуля ES
- `files`: это белый список файлов, которые npm включит в опубликованный пакет. Поскольку наши файлы сборки являются самодостаточными, нам нужно включить только `dist`

{% highlight js %}
{
  "name": "@yourname/yourlibrary",
  "version": "0.0.1",
  "main": "dist/library.js",
  "module": "dist/library.mjs",
  "files": [
    "dist/*"
  ]
}
{% endhighlight %}

_Вы также можете добавить исходные файлы ваших компонентов в белый список файлов в `package.json`, если хотите, чтобы
ваши компоненты импортировались по отдельности без плагина._

Теперь, когда наш пакет настроен правильно, давайте опубликуем его. Запустите `npm login` в терминале, чтобы убедиться,
что вы вошли в npm. Затем запустите `npm publish --access=public`, чтобы опубликовать пакет.

{% highlight html %}
$ npm login
$ npm publish --access=public
{% endhighlight %}

После этого мы можем проверить реестр npm, чтобы узнать, можем ли мы найти наш опубликованный пакет. Запустите npm view
`@yourname/yourlibrary`. Если публикация прошла успешно, вы увидите информацию о пакете в консоли.

## Использование опубликованной библиотеки Vue 3

Теперь, когда наша библиотека компонентов публично опубликована на npm, мы можем использовать ее в проекте, как и любой
другой модуль npm.

Вот как можно включить вашу библиотеку в проект Vue 3 с помощью ES-модулей. После установки плагина вы можете ссылаться
на свои компоненты в любом шаблоне Vue этого проекта.

{% highlight js %}
import { createApp } from 'vue'
import App from './App.vue'

import plugin from '@yourname/yourlibrary'

createApp(App)
  .use(plugin)
  .mount('#app')
{% endhighlight %}

## Настройка сайта документации

Ваша библиотека теперь пригодна для использования, но мы еще не закончили! Если вы собираетесь опубликовать библиотеку
компонентов, вам необходимо предоставить документацию, чтобы разработчики знали, как ее использовать.

К счастью, экосистема Vue имеет свою собственную структуру документации, которая идеально подходит для этой работы:
[VuePress](https://v2.vuepress.vuejs.org/). VuePress позволяет создать простой, но хорошо выглядящий статический сайт документации с контентом в формате
markdown.

Поскольку в нашем проекте используется Vue 3, нам понадобится VuePress v2. Эта версия все еще находится в бета-версии,
поэтому мы можем установить ее с помощью пакета версии `vupress@next`.

{% highlight html %}
$ npm i -D vuepress@next
{% endhighlight %}

Чтобы настроить наши документы, мы сначала создадим каталог `docs` и добавим в него файл `README.md`, который VuePress будет
использовать в качестве содержимого главной страницы.

Как и на большинстве сайтов с документацией, которые вы видели, главная страница является хорошим местом для краткого
введения в библиотеку, инструкций по быстрому запуску и т.д.

{% highlight html %}
# My Component library

Here's a brief introduction.

### Installation

$ npm install @yourname/yourlibrary
{% endhighlight %}

## Запуск сервера разработки VuePress

Теперь мы добавим в `package.json еще два скрипта, один для запуска dev-сервера VuePress, а другой для создания продакшн сборки.

{% highlight js %}
"scripts": {
  "build": "rollup -c",
  "docs:dev": "vuepress dev docs",
  "docs:build": "vuepress build docs"
},
{% endhighlight %}

Запустим сервер dev командой `npm run docs:dev`. При первом запуске VuePress создаст подпапку с именем `docs/.vuepress`,
куда мы вскоре добавим дополнительные настройки.

## Документирование компонентов в Vue 3

Чтобы задокументировать два компонента в библиотеке, давайте создадим два файла markdown в другой подпапке,
`docs/components`. В этих файлах вы будете объяснять API компонентов, приводить примеры использования и все остальное, что
поможет пользователю.

{% highlight html %}
# input-text

`InputText` is a cool component. Here's how to use it...

<template>
  <input-text />
</template>
{% endhighlight %}

_Помимо обычного текста и кода в формате разметки, VuePress позволяет проводить интерактивные демонстрации компонентов,
прикрепляя ваш компонент к экземпляру Vue на странице документации. Подробнее об этом можно узнать [здесь](https://dev.to/siegerts/creating-a-vue-js-component-library-part-iv-documentation-with-vuepress-56h5)._

При запуске VuePress эти файлы разметки будут опубликованы как страницы. Чтобы сделать эти страницы доступными, мы
добавим их на боковую панель docs, добавив в конфигурационный файл VuePress следующую информацию о тематике:

{% highlight js %}
module.exports = {
  themeConfig: {
    sidebar: [
      {
        title: 'Components',
        collapsable: false,
        children: [
          '/components/input-text.md',
          '/components/input-textarea.md'
        ]
      }
    ]
  }
}
{% endhighlight %}

Проверьте браузер. Теперь ваш сайт VuePress будет выглядеть примерно так:

![Создание библиотеки компонентов Vue 3]({{ site.baseurl }}/assets/img/posts/2022-01-13-building-vue-3-component-library-2.png)

## Публикация документации на GitHub Pages

Для публикации наших документов мы можем использовать [GitHub Pages](https://pages.github.com/), который предоставляет бесплатный хостинг.

Если вы не хотите создавать собственный домен, URL вашего сайта Pages будет выглядеть следующим образом:

{% highlight html %}
https://<yourname>.github.io/<yourlibrary>/
{% endhighlight %}

Важно отметить, что он будет находиться во вложенной папке, а не в корневом каталоге. По этой причине вам необходимо
указать опцию базовой конфигурации в конфигурации VuePress, чтобы относительные пути работали правильно.

`docs/.vuepress/config.js`

{% highlight js %}
module.exports = {
  ...
  base: '/yourlibrary/'
}
{% endhighlight %}

После этого мы создадим сценарий развертывания. Этот сценарий создаст документацию, зафиксирует сборку в ветке `gh-pages`,
а затем отправит commit на GitHub, где она будет опубликована.

Если вы этого еще не сделали, убедитесь, что ваша библиотека компонентов опубликована на GitHub (yourname/yourlibrary).
Запустите скрипт и ваша статическая сборка будет размещена в репозитории.

`deploy.sh`

{% highlight html %}
#!/usr/bin/env sh

set -e

npm run docs:build
cd docs/.vuepress/dist

git init
git add -A
git commit -m 'deploy'

git push -f git@github.com:yourname/yourlibrary.git master:gh-pages

cd -
{% endhighlight %}

Чтобы указать GitHub на публикацию сайта, вы можете включить функцию Pages в настройках репозитория. Убедитесь, что вы
выбрали ветку `gh-pages` и исходные файлы из корневого каталога.

![Создание библиотеки компонентов Vue 3]({{ site.baseurl }}/assets/img/posts/2022-01-13-building-vue-3-component-library-3.png)

## Заключение

В этой статье мы рассмотрели, как создать библиотеку компонентов Vue 3 и опубликовать ее на npm, а также опубликовать
документацию на GitHub Pages.

Используя этот материал, вы сможете создавать библиотеки, которые обеспечат согласованность в ваших собственных проектах
или предоставят замечательные компоненты с открытым исходным кодом сообществу Vue.