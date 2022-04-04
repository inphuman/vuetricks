---
layout: post
title: Компонент поиска в JSON для Vue на основе Fuse.Js
tags: [поиск в данных, Fuse.Js]
comments: false
---

**Ville Säävuori** предлагает новый компонент поиска в JSON для Vue 3, основанный на Fuse.js. Этот поиск был разработан для
статических генераторов типа Hugo, но он может работать с любым сайтом, способным создавать JSON-базу. Он обладает
следующими возможностями.

- Легко настраивается с помощью любого программного обеспечения;
- 100% контроль над разметкой и стилями с помощью - компонентов и слотов Vue;
- Легкий вес и минимум зависимостей (Fuse.js и Vue 3), ~8 Кб в упакованном виде;

Для получения более подробной информации об этом пакете вы можете посетить его полную документацию и исходный код на [Github](https://github.com/Uninen/vue-json-search).

## Простой пример со статическим сайтом

Следующие инструкции предполагают, что в вашем проекте есть `package.json`.

1. Установите `vue@next` и `vue-json-search`
2. Создайте простой скрипт search.js для вашего сайта:

```vue
import { createApp, h } from 'vue'
import { JsonSearch } from 'vue-json-search'

createApp({
  render: () => h(JsonSearch, { showTags: true }), // Props argument dict is optional
}).mount('#searchapp')
```

Выше показан минимально функциональный способ использования этого компонента. Это всего лишь JavaScript, используйте его
так, как вам удобно. (Преимущество примера в том, что для него не нужны шаблоны Vue, что позволяет уменьшить размер
пакета).

3. Добавьте компонент поиска в ваш HTML-шаблон:

```vue
<div>
    <h2>Search</h2>
    <div id="searchapp"></div>
</div>
```

4. Подключите `/index.json` (см. ожидаемый формат JSON и параметры конфигурации ниже)

## Использование в качестве компонента Vue

Вы можете использовать его как любой другой компонент Vue.

1. Импортируйте компонент в свой проект.

```vue
import { JsonSearch } from 'vue-json-search'
```

2. А затем используйте его в своем шаблоне как любой другой **компонент Vue**:

```vue
<JsonSearch :max-results="20" />
```

## Настройка разметки

Вы можете настроить 100% разметки с помощью слотов Vue.

Сначала импортируйте необходимые компоненты:

```vue
import { JsonSearch, ResultList, ResultListItem, ResultTitle, SearchInput, SearchResults } from 'vue-json-search'
```

Затем делайте с ними все, что захотите. Вот простой пример:

```vue
<JsonSearch :show-tags="true" v-slot="{ results }">
  <SearchInput />
  <SearchResults>
    <ResultTitle />
    <div v-for="res in results" :key="res.refIndex">
      <ResultListItem v-slot="{ result }" :result="res.item">
        <p>Title: {{ result.title }}</p>
        <p>Tags: {{ result.tags }}</p>
      </ResultListItem>
    </div>
  </SearchResults>
</JsonSearch>
```

Документация по компонентам не очень хороша, но исходный текст легко читается и понимается, если вы знакомы с Vue.