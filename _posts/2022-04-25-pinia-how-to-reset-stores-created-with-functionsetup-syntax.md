---
layout: post
title: Pinia - как сбросить хранилища, созданные с помощью функционального синтаксиса
tags: [Pinia, VueJS]
comments: false
---

## Что такое Pinia

Pinia - это решение для управления состояниями для Vue 3.
Если вы думаете: "О, еще один инструмент?", то правда в том, что Pinia можно рассматривать как следующую версию Vuex 4.

Pinia имеет много общего с Vuex, но она проще, безопасна для типов и расширяема!

## Объектные и функциональные хранилища

**Объектный синтаксис:**

```js
import { defineStore } from 'pinia'

export const useStore = defineStore('main', {
  state: () => ({
    counter: 0,
  }),
  getters: {
    doubleCount: (state) => state.counter * 2,
  },
  actions: {
    increment() {
      this.counter++
    },
  }
})
```

**Функциональный синтаксис:**

```js
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'

export const useStore = defineStore('main', () => {
  const counter = ref(0)
  const doubleCounter = computed(() => counter.value * 2)

  function increment() {
    counter.value++
  }

  return {
   counter,
   doubleCounter,
   increment
  }
})
```

Объектный синтаксис очень похож на синтаксис Vuex. Функциональный синтаксис похож на метод **setup()** компонента, где мы
вручную определяем реактивность.

## Как сбросить хранилище с помощью объектного синтаксиса

При использовании объектного синтаксиса можно просто вызвать встроенный метод **$reset**:

```js
<script setup>
import { useStore } from './useStore'

const store = useStore()

store.$reset // 👈

</script>
```

## Как сбросить хранилище с помощью функционального синтаксиса

Для того чтобы заставить метод **$reset** работать в функциональном синтаксисе, мы создадим **плагин Pinia**: ✨

**reset-store.js**

```js
import cloneDeep from 'lodash.clonedeep'

export default function resetStore({ store }) {
  const initialState = cloneDeep(store.$state)
  store.$reset = () => store.$patch(cloneDeep(initialState))
}
```

Здесь мы глубоко копируем начальное состояние хранилища с помощью [lodash.clonedeep](https://www.npmjs.com/package/lodash.clonedeep) и добавляем функцию **$reset**, которая
устанавливает состояние в его начальное значение. Важно снова глубоко скопировать состояние, чтобы удалить ссылки на
саму копию.

С помощью `store.$patch` мы применяем несколько изменений одновременно.

Затем в файле **main.js** или там, где вы инициализируете **Pinia**:

```js
// ...
import { resetStore } from './reset-store'

//...
const pinia = createPinia()
pinia.use(resetStore)

app.use(pinia)

//...
```

## Заключение

Потрясающе! Теперь независимо от того, какой синтаксис мы используем, метод **$reset** работает!