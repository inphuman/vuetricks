---
layout: post
title: Создание библиотеки компонентов Vue 3
tags: [Vue.js, библиотека компонентов]
comments : True
---

Библиотеки компонентов - это отличный способ собрать общие решения пользовательского интерфейса в пакет, который можно
повторно использовать в собственных проектах или поделиться с сообществом разработчиков Vue с открытым исходным кодом.

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


## Заключение

В этой статье мы рассмотрели, как создать библиотеку компонентов Vue 3 и опубликовать ее на npm, а также опубликовать
документацию на GitHub Pages.

Используя этот материал, вы сможете создавать библиотеки, которые обеспечат согласованность в ваших собственных проектах
или предоставят замечательные компоненты с открытым исходным кодом сообществу Vue.