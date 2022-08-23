---
layout: post
title: Как валидировать свойства (props) Vue как профессионал
tags: [Vue, свойства, валидация]
comments: false
---

Vue требует явного объявления любых данных, передаваемых компоненту в качестве свойств (props). Кроме того, он предоставляет
мощный встроенный механизм для проверки этих данных. Это действует как контракт между компонентом и потребителем и
гарантирует, что компонент используется по назначению.

Давайте изучим этот мощный инструмент, который поможет нам уменьшить количество ошибок и повысить уверенность во
время разработки и отладки.

## Примитивные типы

Валидация примитивных типов осуществляется через установку опции type в конструкторе примитивного типа.

```js
export default {
  props: {
    // Basic type check
    //  (`null` and `undefined` values will allow any type)
    propA: Number,
    // Multiple possible types
    propB: [String, Number],
    // Required string
    propC: {
      type: String,
      required: true
    },
    // Number with a default value
    propD: {
      type: Number,
      default: 100
    },
  }
}
```

## Сложные типы

Сложные типы также могут быть проверены таким же образом.

```js
export default {
  props: {
    // Object with a default value
    propE: {
      type: Object,
      // Object or array defaults must be returned from
      // a factory function. The function receives the raw
      // props received by the component as the argument.
      default(rawProps) {
        return { message: 'hello' }
      }
    },
    // Array with a default value
    propF: {
      type: Array,
      default() {
        return []
      }
    },
    // Function with a default value
    propG: {
      type: Function,
      // Unlike object or array default, 
      // this is not a factory function 
      // - this is a function to serve as a default value
      default() {
        return 'Default function'
      }
    }
  }
}
```

`Type` может быть одним из следующих значений:

- Number
- String
- Boolean
- Array
- Object
- Date
- Function
- Symbol

Кроме того, `Type` может быть пользовательским классом или функцией конструктора, и утверждение будет выполнено с помощью
проверки instanceof. 

```js
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
  }
}
```

Вы можете использовать его в качестве `props`, например, так:

```js
export default {
  props: {
    author: Person
  }
}
```

## Дополнительная валидация

**Функция валидатор**

`props` поддерживают использование функции валидатора. Эта функция принимает значение и
должна вернуть булево значение, чтобы определить, является ли это свойство действительным или нет.

```js
// Custom validator function
prop: {
  validator(value) {
    // The value must match one of these strings
    return ['success', 'warning', 'danger'].includes(value)
  }
},
```

## Использование enums

Иногда нужно сузить круг значений до определенного набора, что можно сделать следующим образом:

```js
export const Position = Object.freeze({
  TOP: "top",
  RIGHT: "right",
  BOTTOM: "bottom",
  LEFT: "left"
});
```

Это можно импортировать и использовать как в валидаторе, так и в качестве значения по умолчанию.

```vue
<template>
  <span :class="`arrow-position--${position}`">
    {{ position }}
  </span>
</template>

<script>
import { Position } from "./types";
export default {
  props: {
    position: {
      validator(value) {
        return Object.values(Position).includes(value);
      },
      default: Position.BOTTOM,
    },
  },
};
</script>
```

Родительский компонент также может импортировать и использовать это перечисление, которое устраняет использование
магических строк из нашего приложения.

```vue
<template>
  <DropDownComponent :position="Position.BOTTOM" />
</template>

<script>
import DropDownComponent from "./components/DropDownComponent.vue";
import { Position } from "./components/types";
export default {
  components: {
    DropDownComponent,
  },
  data() {
    return {
      Position,
    };
  },
};
</script>
```

## Приведение булевых значений

Boolean `props` имеют уникальное поведение. Наличие или отсутствие атрибута может определять значение реквизита.

```vue
<!-- equivalent of passing :disabled="true" -->
<MyComponent disabled />

<!-- equivalent of passing :disabled="false" -->
<MyComponent />
```

## TypeScript

Объединение встроенной проверки свойств в Vue и TypeScript может дать нам еще больше контроля,
поскольку TypeScript изначально поддерживает интерфейсы и перечисления.

```vue
<script lang="ts">
import Vue, { PropType } from 'vue'
interface Book {
  title: string
  author: string
  year: number
}
const Component = Vue.extend({
  props: {
    book: {
      type: Object as PropType<Book>,
      required: true,
      validator (book: Book) {
        return !!book.title;
      }
    }
  }
})
</script>
```

## Real enums

Мы уже рассмотрели, как использовать фейковые перечисления в Javascript. В TypeScript это не требуется, поскольку перечисления
поддерживаются изначально.

```js
<script lang="ts">
import Vue, { PropType } from 'vue'
enum Position {
  TOP = 'top',
  RIGHT = 'right',
  BOTTOM = 'bottom',
  LEFT = 'left',
}
export default {
  props: {
    position: {
      type: String as PropType<Position>,
      default: Position.BOTTOM,
    },
  },
};
</script>
```

## Vue 3

Все вышесказанное справедливо при использовании Vue 3 с API Options или Composition. Разница заключается в
использовании **<script setup>**. Свойства должны быть объявлены с помощью defineProps() следующим образом:

```vue
<script setup>
const props = defineProps(['foo'])
console.log(props.foo)
</script>


<script setup>
// Long synstax is also supported
defineProps({
  title: String,
  likes: Number
})
</script>
```

При использовании TypeScript с <script setup>:

```vue
<script setup lang="ts">
defineProps<{
  title?: string
  likes?: number
}>()
</script>
```

Или с использованием интерфейсов:

```vue
<script setup lang="ts">
interface Props {
  foo: string
  bar?: number
}
const props = defineProps<Props>()
</script>
```

Наконец, объявление значений по умолчанию в свойствах при использовании типов:

```vue

<script setup lang="ts">
interface Props {
  foo: string
  bar?: number
}
// reactive destructure for defineProps()
// default value is compiled to equivalent runtime option
const { foo, bar = 100 } = defineProps<Props>()
</script>
```

## Заключение

Проверка типов - это отличная первая линия защиты от ошибок по мере роста размера вашего приложения. Встроенная в Vue
проверка типов является мощной. В сочетании с TypeScript она может дать вам высокую уверенность в правильном
использовании интерфейса компонента, уменьшить количество ошибок и улучшить общее качество кода и опыт разработки.

