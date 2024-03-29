---
layout: post
title: Зачем использовать TypeScript в Vue.js?
tags: [TypeScript, Vue.Js]
comments: false
---

Vue.js - это очень известный гибкий фреймворк, применение которого варьируется от постепенного улучшения статических
страниц до создания полноценных одностраничных приложений. Аналогичным образом, Vue также обеспечивает гибкость
разработки на обычном JavaScript или на его более строгом безопасном для типов аналоге, TypeScript.

В связи с этим возникает вопрос, стоит ли использовать **TypeScript** в приложении на **Vue.js**? В этой статье мы рассмотрим
некоторые причины, по которым вы можете сделать такой выбор.

## Отлавливайте ошибки в процессе написания кода

Поскольку **TypeScript** является сильно типизированным языком, он способен обнаружить гораздо более широкий спектр проблем
прямо в вашей IDE. Альтернативой этому является обнаружение таких проблем только после запуска кода в браузере. Это
может означать обнаружение проблемы во время ручного тестирования или только после того, как конечный пользователь
сообщит о ней.

Например, допустим, вы хотите отформатировать 9-значный номер телефона так, чтобы в нем были прочерки для отображения.

```vue
<script setup>
    import { ref, computed } from "vue";
    const phone = ref("55566677777");
    const [match, one, two, three] = phone.value.match(/^(\d{3})(\d{3})(\d{4})$/);
    
    // 555-666-7777
    const formatted = computed(() => `${one}-${two}-${three}`);
</script>

<template>Phone: {{ formatted }}</template>
```

Я намеренно не указывал lang="ts", чтобы показать, что произойдет d обычным JavaScript. Итак, вышеописанное сработает,
как и ожидалось, но, возможно, вы торопитесь, или номер телефона находится в другом файле, или является результатом
функции. Короче говоря, что если номер телефона окажется не строкой, а действительным числом?

```vue
const phone = ref(55566677777);
```

При работе в моей IDE нет никаких признаков проблемы.

Этот снимок экрана сделан непосредственно из VS Code, и я не могу сказать, что что-то не так. Нет никаких красных
загогулин, и я даже использую ESLint. У нас есть желтая загогулистая линия под match, но это только потому, что мы не
используем переменную, это не повод для ошибки.

![Этот снимок экрана сделан непосредственно из VS Code]({{ site.baseurl }}/assets/img/posts/phone-number-number-type-js.jpeg)

Однако если я перехожу в браузер, **все сразу же ломается**. Все приложение перестает работать, и я **получаю ошибку в
консоли**.

Проблема в том, что у чисел нет метода сопоставления, как у строк. Это достаточно просто решить. Вернувшись в
IDE, мы можем снова заключить число в кавычки, но при этом случайно добавим к нему еще одно число.

В этот раз **match** равен **null**, потому что строка не соответствует выражению **regex**, а мы не можем передать **null** в массив.

Даже при небольшом объеме кода, с которым мы работаем, этот процесс уже стал немного раздражающим, но в больших
приложениях он может быть не просто раздражающим. Это может привести к повторяющемуся, трудоемкому тестированию,
требующему многократных щелчков и ожидания в браузере, а также к пропуску некоторых крайних случаев.

Что если бы мне не нужно было ждать выполнения кода, чтобы отловить эту ошибку? Вот тут-то и приходит на помощь **TypeScript**.
Добавление `lang="ts"` в раздел `script` компонента сразу же выявляет ошибку в IDE, а наведение курсора на ошибку
напоминает, что результатом **match** может быть массив, но может быть и **null**.

![Этот снимок экрана сделан непосредственно из VS Code]({{ site.baseurl
}}/assets/img/posts/match-can-be-null-ts-hint.jpeg)

Имея эту информацию под рукой, мы теперь можем легко справиться с этим крайним случаем, используя по умолчанию пустой
массив, если соответствие возвращает **null**, а затем отображая строку **"Invalid Phone"** для нашего отформатированного
телефонного номера, если соответствие не определено.

```vue
<script setup lang="ts">
    import { ref, computed } from "vue";
    const phone = ref("555666777777");

    // default to empty array if result of .match is null
    // (making match, one, two, and three undefined)
    const [match, one, two, three] = phone.value.match(/^(\d{3})(\d{3})(\d{4})$/) || [];

    // 555-666-7777
    const formatted = computed(() =>
        // check if match is undefined
        match ? `${one}-${two}-${three}` : "Invalid Phone"
);
</script>

<template>Phone: {{ formatted }}</template>
```

Более того, если мы случайно снова используем число вместо строки, **TypeScript** немедленно предупредит нас об
этом.

## Рефакторинг с уверенностью того, что вы делаете

Еще одним преимуществом работы с **TypeScript** в проектах Vue.js является то, что вы можете рефакторить с большей
уверенностью того, что вы делаете.

Допустим, у вас есть компонент **PostForm** для приложения блога, который на обычном **JavaScript** выглядел примерно так.

```vue
// PostForm.vue
<script setup>
    import { ref } from "vue";
    defineEmits(["create"]);
    const title = ref("");
    const body = ref("");
</script>

<template>
    <form @submit.prevent="$emit('create', { body, title })">
        <input v-model="title" />
        <textarea v-model="body"></textarea>
        <button>Create Post</button>
    </form>
</template>
```

Он отслеживает **title** и **body** как реактивные ссылки. Он также эмитит событие **create** при отправке формы с объектом,
который включает **title** и **body** в качестве свойств.

Теперь предположим, что вы хотите отрефакторить этот **emit** в собственную функцию-обработчик, чтобы вы могли делать с
ним больше (например, проводить валидацию, сбрасывать данные формы после отправки или что-то еще).

```vue
// PostForm.vue
<script setup>
    import { ref } from "vue";
    defineEmits(["create"]);
    const title = ref("");
    const body = ref("");
    const handleSubmit = () => {
      $emit("create", { body, title });
    };
</script>
<template>
    <form @submit.prevent="handleSubmit">
      <input v-model="title" />
      <textarea v-model="body"></textarea>
      <button>Create Post</button>
    </form>
</template>
```

При включенном ESLint в моем проекте я, по крайней мере, получаю уведомление о том, что **$emit** не определен.

Чтобы исправить это, мы должны захватить функцию **emit**, возвращаемую макросом **defineEmits**, а затем заменить ею функцию
**emit** внутри функции обработчика.

```js
const emit = defineEmits(["create"]);
//...
const handleSubmit = () => {
  emit("create", { body, title });
};
```

С этим, насколько известно моей IDE, все в порядке. Однако на самом деле при рефакторинге скрывается еще одна проблема.

```vue
<script setup>
    import { ref } from "vue";
    const emit = defineEmits(["create"]);
    const title = ref("");
    const body = ref("");
    const handleSubmit = () => {
      emit("create", { body, title });
    };
    </script>
    <template>
        <form @submit.prevent="handleSubmit">
          <input v-model="title" />
          <textarea v-model="body"></textarea>
          <button>Create Post</button>
        </form>
    </template>
```

Не волнуйтесь, если не сделали этого. Это сложно! Проблема в том, что раньше мы передавали строки для **body** и **title**.
Теперь же мы емитим реактивные ссылки.

Поэтому, чтобы сохранить интерфейс нашего компонента таким же, как до рефакторинга, мы должны изменить **emit**, чтобы включить
**body.value** и **title.value**.

```js
emit("create", { body: body.value, title: title.value });
```

Вполне понятно, как вы можете пропустить подобное в рефакторинге вашего кода.
Результатом, к сожалению, может стать несколько минут или даже больше потраченного впустую времени на попытки понять,
что пошло не так после тестирования в браузере.

Теперь давайте рассмотрим, как тот же самый рефакторинг мог бы выглядеть в **TypeScript**.

Прежде всего, если бы вы использовали **TypeScript**, то у вас была бы привычка типизировать свои пользовательские события.
Таким образом, вы бы уже определили тип полезной нагрузки событий примерно так.

```vue
defineEmits<{
  (e: "create", payload: { body: string; title: string }): void;
}>();
```

В **TypeScript** это просто определяет тип свойств **body** и **title** в объекте полезной нагрузки как строки.

Затем, после того как вы обработали **$emit**, как и раньше, вы получите следующий код.

```vue
<script setup lang="ts">
    import { ref } from "vue";
    const emit = defineEmits<{
      (e: "create", payload: { body: string; title: string }): void;
    }>();
    const title = ref("");
    const body = ref("");
    const handleSubmit = () => {
      emit("create", { body, title });
    };
</script>

<template>
    <form @submit="handleSubmit">
      <input v-model="title" />
      <textarea v-model="body"></textarea>
      <button>Create Post</button>
    </form>
</template>
```

Однако на этот раз ваша IDE даст вам понять, что что-то пахнет чем-то подозрительным. VS Code очень прямо называет
свойства **body** и **title** полезной нагрузки события красными волнистыми линиями. Наведение курсора на каждое из них точно
показывает проблему: реактивная ссылка - это не то же самое, что строка.

![Однако на этот раз ваша IDE даст вам понять]({{ site.baseurl
}}/assets/img/posts/typescript-hint-vue-custom-event.jpeg)

Решение, конечно, такое же, как и раньше, но на этот раз мы смогли исправить ошибку сразу же, поскольку обратная связь
была мгновенной и в контексте IDE. После исправления красные волнистые линии исчезли.

## Расширенная функциональность IDE для взаимодействия между компонентами

Помимо улучшения видимости ошибок в вашей **IDE** при рефакторинге, **TypeScript** также поможет вам при взаимодействии между
компонентами. Это возможно благодаря более сфокусированным и точным опциям автозавершения и обнаружения ошибок для
реквизитов и событий.

Возьмем тот же пример, что и выше. Напомним, что в **JavaScript** мы бы написали определение **emits** следующим образом.

```js
// PostForm.vue
const emit = defineEmits(["create"]);
```

Если бы вы сейчас использовали компонент `PostForm` в другом компоненте и прослушали событие `create`, вы бы не получили
никаких опций автозаполнения в полезной нагрузке события.

```vue
<PostForm @create="$event"/>
```

Однако, если вы переключите его обратно на синтаксис объявления типов для определения эмитов, как это сделано здесь:

```vue
const emit = defineEmits<{
  (e: "create", payload: { body: string; title: string }): void;
}>();
```

После этого вы получите полный и предельно сфокусированный автозаполняемый список доступных свойств события вместе с их
типами.

![После этого вы получите полный и предельно сфокусированный автозаполняемый список доступных свойств события вместе с их
типами.]({{ site.baseurl
}}/assets/img/posts/typescript-vue-custom-event-autocomplete-options.jpeg)

Только представьте, насколько это упрощает взаимодействие между компонентами! Нет необходимости искать код или
документацию потребляемого компонента, чтобы узнать полезную нагрузку конкретного события.

То же самое преимущество относится и к реквизитам. Если бы мы хотели позволить нашей форме `PostForm` принимать
существующий пост для редактирования, это выглядело бы следующим образом.

```js
//PostForm.vue
const props = defineProps<{
    // the question mark makes it an optional prop 
    // (since our form could be used for new or existing posts)
  post?: { body: string; title: string };
}>();

//...

const title = ref(props.post?.title || "");
const body = ref(props.post?.body || "");
```

На этом этапе мы могли бы даже нормализовать форму нашего объекта `post` в многократно используемый интерфейс.

```js
// PostForm.vue
interface Post {
  body: string;
  title: string;
}
const props = defineProps<{
  post?: Post;
}>();
const emit = defineEmits<{
  (e: "create", payload: Post): void;
}>();
```

Наконец, мы должны посмотреть, как выглядит фактическая передача свойства в компонент.

```vue
// App.vue
<script setup lang="ts">
    import { ref } from "vue";
    import PostForm from "@/components/PostForm.vue";
    
    const existingPost = ref({
      body: "",
      tite: "",
    });
</script>

<template>
    <PostForm :post="existingPost" />
</template>
```

Если вы были внимательны, вы могли бы заметить, что в коде есть опечатка. Однако если бы вы
работали непосредственно в коде VS, вам не нужно было бы обращать на это внимание, потому что TypeScript явно укажет вам
на ошибку.

![После этого вы получите полный и предельно сфокусированный автозаполняемый список доступных свойств события вместе с их
типами.]({{ site.baseurl
}}/assets/img/posts/typescript-hint-vuejs-props.jpeg)

Это полезно не только для выявления опечаток, но и, особенно для больших объектов, для обеспечения того, что все
необходимые свойства действительно существуют и имеют правильный тип. Другими словами, вы получаете документацию прямо в
вашей IDE о том, как именно должен выглядеть реквизит.


