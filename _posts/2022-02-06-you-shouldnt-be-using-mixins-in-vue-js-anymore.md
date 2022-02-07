---
layout: post
title: Вам не нужно больше использовать миксины в Vue.js
tags: [VueJS]
comments: false
---

Миксины были основным способом обмена многократно используемой логикой между компонентами в Vue 2. Но теперь, благодаря
наличию API Composition, мы можем добиться повторного использования кода гораздо более чистым способом.

## Что такое миксины?

Миксины - это способ повторного использования функциональности между компонентами путем объединения свойств миксина с
компонентом. Это позволяет нам определить определенную логику один раз и использовать ее в нескольких компонентах.

## Почему уже не нужно использовать миксины?

Миксины, как правило, не являются идеальным способом совместного использования логики, поскольку они имеют ряд недостатков:

**Пересекающиеся названия**

Конфликты имен часто возникают в миксинах. Во время процесса объединения конфликтующие свойства между миксинами и
компонентами переопределяются на основе некоторой стратегии объединения. Это может привести к неожиданному поведению в
нашем приложении.

**Жесткое соединение**

Нередко можно столкнуться с неявными зависимостями между миксином и компонентом. Это делает мучительно трудным
рефакторинг компонента или миксина без разрушения существующего кода. Миксины также могут быть тесно связаны с другими
миксинами с течением времени по мере появления новых требований.

**Трудно понять и отладить**

Новому разработчику очень сложно разобраться в кодовой базе с большим количеством миксинов. Не очевидно, откуда берется
то или иное свойство, особенно если есть глобально зарегистрированные миксины. Изолировать ошибку также становится
кошмаром по мере роста приложения.

## Замена миксинов на компоненты

Давайте начнем с примера, иллюстрирующего некоторые недостатки миксинов и то, как мы будем решать эти проблемы с помощью API Composition.

Допустим, у нас есть простой таймер обратного отсчета, который должен подождать несколько секунд перед показом основного
содержимого компонента. Мы вынесли эту логику в миксин под названием countDownMixin . Вот как выглядит код:

`countDownMixin.js`

```js
const countDownMixin = {
  data() {
    return {
      countDown: 10,
      interval: null,
    };
  },
  computed: {
    isDone: function () {
      return this.countDown <= 0;
    },
  },
  methods: {
    decrement() {
      if (this.countDown > 0) this.countDown -= 1;
    },
  },
  mounted: function () {
    this.interval = setInterval(this.decrement, 1000);
  },
  unmounted: function () {
    clearInterval(this.interval);
  },
};

export default countDownMixin;
```

Зарегистрировав этот миксин, мы сможем "подмешать" логику обратного отсчета в компонент.

`HelloWorldMixin.vue`

```vue
<template>
  <div>
  <h1 v-if="!isDone">Countdown: {{ countDown }} seconds</h1>
  <h1 v-else>Hello World</h1>
  </div>
</template>

<script>
import countDownMixin from "./countDownMixin";
export default {
  name: "HelloWorld",
  mixins: [countDownMixin]
};
</script>
```

На первый взгляд, это изящное решение, но что если я хочу подождать не 10, а 20 секунд?

Одним из решений может быть переопределение начального состояния countDown из потребляющего компонента. Поступая таким
образом, мы явно создаем конфликтующие имена, зная, что свойство данных, определенное в компоненте, будет иметь высший
приоритет.

`HelloWoldMixin2.vue`

```vue
<template>
  <div>
  <h1 v-if="!isDone">Countdown: {{ countDown }} seconds</h1>
  <h1 v-else>Hello World</h1>
  </div>
</template>

<script>
import countDownMixin from "./countDownMixin";
export default {
  name: "HelloWorld",
  mixins: [countDownMixin],
  data(){
  return {countDown:20}
  }
};
</script>
```

Это решение работает, но наличие такой возможности делает миксин уязвимым к непреднамеренной замене и других опций.
Теперь, если компонент определит метод `decrement`, не зная, что `countDownMixin` также имеет метод с таким же именем,
компонент не будет работать так, как ожидалось.

`HelloWorldMixin3.vue`

```vue
<template>
  <div>
  <h1 v-if="!isDone">Countdown: {{ countDown }} seconds</h1>
  <h1 v-else>Hello World</h1>
  </div>
</template>

<script>
import countDownMixin from "./countDownMixin";
export default {
  name: "HelloWorld",
  mixins: [countDownMixin],
  data(){
  return {countDown:20}
  },
  methods:{
    decrement() {
      console.log("decrement from self");
   }
  },
};
</script>
```

Хотя метод декремента определен только для внутреннего использования, невозможно избежать его переопределения.

## Лучшее решение с помощью API Composition

`useCountDown.js`

```js
import { ref, computed, onMounted, onUnmounted, readonly } from "vue";

export default function useCountDown(countDownDuration = 10) {
  const countDown = ref(countDownDuration);
  const interval = ref(null);
  const decrement = () => {
    if (countDown.value > 0) countDown.value -= 1;
  };
  const isDone = computed(() => {
    return countDown.value <= 0;
  });
  onMounted(() => {
    interval.value = setInterval(decrement, 1000);
  });
  onUnmounted(() => {
    clearInterval(interval.value);
  });

  return { countDown:readonly(countDown), isDone };
}
```

Используя API Composition, мы избегаем всех проблем, с которыми сталкивались при работе с миксинами. Композитная функция
useCountDown принимает в качестве параметра длительность обратного отсчета. Не нужно ничего переопределять, а
зависимости аккуратно организованы как параметры композитной функции.

Он также раскрывает свойства, предназначенные только для использования потребляющим компонентом, а внутренние состояния
защищены, поэтому компонент не может случайно изменить поведение компонуемого.

Наложение имен также не является проблемой, поскольку от нас требуется явное предоставление имен для всех возвращаемых переменных композита.

```vue
<template>
  <div>
  <h1 v-if="!isDone">Countdown: {{ countDown }} seconds</h1>
  <h1 v-else>Hello World</h1>
  </div>
</template>

<script>
import useCountDown from "./useCountDown";
export default {
  name: "HelloWorld",
  setup(){
  const {countDown, isDone} = useCountDown();
  return {countDown, isDone};
  }
};
</script>
```

## Заключительные мысли

API Composition делает повторное использование кода очень простым и понятным. Я надеюсь, что эта статья показалась вам
достаточно убедительной для того, чтобы перейти от миксинов и принять силу композитов. Если вам нужна дополнительная
информация об API Composition, пожалуйста, ознакомьтесь с [официальной документацией](https://v3.vuejs.org/guide/composition-api-introduction.html).