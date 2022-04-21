---
layout: post
title: Как добавить авторизацию Supabase в приложение Vue
tags: [Supabase, VueJS, авторизация]
comments: false
---

##### В этом посте мы рассмотрим настройку аутентификации с помощью Supabase и Vue 3.

## Предварительные условия

Вы должны быть знакомы с JavaScript и иметь некоторый опыт работы с Vue 3. Опыт работы с Supabase будет полезен, но не
обязателен.

Если вам нужен краткий обзор Supabase, вы можете ознакомиться с [официальной документацией](https://supabase.com/).

Вам также понадобится **Node.js** и **NPM**, установленные на вашей машине.


## Начало работы

Давайте начнем с создания фронтенда, прежде чем приступим к созданию базы данных с помощью Supabase.

Первое, что нам нужно сделать, это настроить наш проект. В терминале и в папке, где будет находиться проект, выполните
следующую команду:

```vue
npm init vite@latest vue-supabase-auth --template vue
```

Это инициализирует новый проект Vite с Vue 3 в папке под названием `vue-supabase-auth`.

Откройте его в выбранном вами редакторе кода и откройте файл **App.vue** внутри папки **src**. Когда я инициализировал проект,
Vite поместил тег script выше тега template. Я предпочитаю перемещать тег шаблона на самый верх, но это не обязательно.
Это инициализирует новый проект Vite с Vue 3 в папке под названием `vue-supabase-auth`.

## Добавление аутентификации в приложение

Следующим шагом будет добавление аутентификации в наше приложение. **Supabase** дает нам возможность аутентифицировать
пользователя различными способами.

Мы рассмотрим, как настроить базовую аутентификацию по электронной почте/паролю, а также аутентификацию по "волшебной
ссылке". Волшебная ссылка" - это просто ссылка, отправленная на электронную почту пользователя, при нажатии на которую
он попадает в ваше приложение и регистрируется в нем.

## Создание учетной записи в Supabase

Если вы еще этого не сделали, вам нужно зарегистрировать учетную запись на Supabase. Он попросит вас зарегистрироваться
на GitHub, поэтому, если у вас нет аккаунта на GitHub, вам также следует зарегистрировать его.

После того как вы вошли в систему, нажмите на зеленую кнопку с надписью "New Project" и выберите организацию по
умолчанию, которая была создана при входе в систему.

Появится окно, в котором вы предоставите некоторую информацию о проекте. Когда проект будет настроен, зайдите в
Authentication -> Settings и отключите опцию "Enable email confirmations". Это
просто немного упростит работу в данном руководстве.

## Настройка Supabase в проекте Vue

Сначала нужно `run npm install @supabase/supabase-js`, чтобы получить пакет JavaScript для интеграции с Supabase.

Затем нам нужно создать файл **supabase.js** в папке **src** проекта. Он должен содержать следующее:

```js
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY

export const supabase = createClient(supabaseUrl, supabaseAnonKey)
```

Как видно из приведенного выше кода, нам необходимо установить некоторые переменные окружения, содержащие ключи нашей
**Supabase**. Создайте файл **.env.local** в корне проекта и добавьте VITE_SUPABASE_URL и VITE_SUPABASE_ANON_KEY. Вы можете
найти свои **url** и **anon_key** в панели вашего проекта **Supabase**.

Ваш файл **.env.local** будет выглядеть следующим образом:

```js
VITE_SUPABASE_URL=YOUR_SUPABASE_URL
VITE_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
```

Нам также нужно создать центральное хранилище для данных, которые необходимы во всем приложении, например, для информации
о пользователях. Создайте файл **store.js** в папке **src** с этим кодом:

```js
import { reactive } from "vue";

export const store = {
  state: reactive({
    user: {},
  }),
};
```

## Создание компонентов SignIn и SignUp

Аутентификация **Supabase** разделяет процессы **signIn** и **signUp**, поэтому нам нужно будет обрабатывать их по-разному. Я решил
создать два отдельных компонента, просто чтобы немного прояснить ситуацию в голове.

Создайте файл **SignUp.vue** в папке components и добавьте следующий код:

```vue
<template>
  <div>
    <h2>Sign up for an account</h2>
    <form @submit.prevent="handleSignup">
      <div>
        <label for="email">Email</label>
        <input id="email" type="email" v-model="email" />
      </div>
      <div>
        <label for="password">Password</label>
        <input id="password" type="password" v-model="password" />
      </div>
      <div>
        <button type="submit">Sign up</button>
      </div>
    </form>
  </div>
</template>

<script>
import { ref } from "vue";
import { supabase } from "../supabase";

export default {
  setup() {
    const email = ref("");
    const password = ref("");

    const handleSignup = async () => {
      try {
        // Use the Supabase provided method to handle the signup
        const { error } = await supabase.auth.signUp({
          email: email.value,
          password: password.value,
        });
        if (error) throw error;
      } catch (error) {
        alert(error.error_description || error.message);
      }
    };

    return {
      email,
      password,
      handleSignup,
    };
  },
};
</script>
```

Теперь создайте файл **SignIn.vue** и добавьте в него приведенный ниже код. Единственные отличия - это имена методов,
которые вызываются, и немного другой текст в разметке.

```vue
<template>
  <div>
    <h2>Sign in to your account</h2>
    <form @submit.prevent="handleSignin">
      <div>
        <label for="email">Email</label>
        <input id="email" type="email" v-model="email" />
      </div>
      <div>
        <label for="password">Password</label>
        <input id="password" type="password" v-model="password" />
      </div>
      <div>
        <button type="submit">Sign in</button>
      </div>
    </form>
  </div>
</template>

<script>
import { ref } from "vue";
import { supabase } from "../supabase";

export default {
  setup() {
    const email = ref("");
    const password = ref("");

    const handleSignin = async () => {
      try {
        // Use the Supabase provided method to handle the signin
        const { error } = await supabase.auth.signIn({
          email: email.value,
          password: password.value,
        });
        if (error) throw error;
      } catch (error) {
        alert(error.error_description || error.message);
      }
    };

    return {
      email,
      password,
      handleSignin,
    };
  },
};
</script>
```

Теперь нам нужно создать компонент-обертку для этих двух компонентов. Создайте файл с именем **Auth.vue** с приведенным ниже
кодом:

```vue
<template>
  <div>
    <sign-up v-if="isSignUp" />
    <sign-in v-else />
    <button @click="isSignUp = !isSignUp">
      {{
        isSignUp
          ? "Already have an account? Sign In"
          : "Don't have an account yet? Sign Up"
      }}
    </button>
  </div>
</template>

<script>
import { ref } from "vue";
import SignUp from "./SignUp.vue";
import SignIn from "./SignIn.vue";

export default {
  components: { SignUp, SignIn },
  setup() {
    const isSignUp = ref(true);

    return {
      isSignUp,
    };
  },
};
</script>
```

Это просто позволит пользователю переключаться между представлениями **SignIn** и **SignUp**. Теперь снова откройте **App.vue** и
обновите код следующим образом:

```vue
<template>
  <!-- Check if user is available in the store, if not show auth compoenent -->
  <Auth v-if="!store.state.user" />
  <!-- If user is available, show the Hello World component -->
  <HelloWorld v-else msg="Hello Vue 3 + Vite" />
</template>

<script>
import Auth from "./components/Auth.vue";
import HelloWorld from "./components/HelloWorld.vue";

import { store } from "./store";
import { supabase } from "./supabase";

export default {
  components: {
    HelloWorld,
    Auth,
  },
  setup() {
    // we initially verify if a user is logged in with Supabase
    store.state.user = supabase.auth.user();
    // we then set up a listener to update the store when the user changes either by logging in or out
    supabase.auth.onAuthStateChange((event, session) => {
      if (event == "SIGNED_OUT") {
        store.state.user = null;
      } else {
        store.state.user = session.user;
      }
    });

    return {
      store,
    };
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

Это покажет компонент **Auth**, если пользователь не вошел в систему, в противном случае будет показан **HelloWorld.vue**, как
мы установили изначально.

Зарегистрируйтесь, используя свою электронную почту и созданный вами пароль, после чего вы снова увидите компонент
**HelloWorld**.

## Как выйти из системы

Выход из системы относительно прост. Внутри компонента **HelloWorld** добавьте в нижнюю часть тега шаблона следующее:

```vue
<button @click="signOut">Sign Out</button>
```

Затем обновите тег сценария в **HelloWorld** на следующий:

```vue
<script setup>
import { ref } from "vue";
import { supabase } from "../supabase";

defineProps({
  msg: String,
});

const count = ref(0);
async function signOut() {
  const { error } = await supabase.auth.signOut();
}
</script>
```

Вы можете видеть, что теперь мы импортируем файл **supabase**, созданный ранее, а затем создаем метод **signOut**, который
вызывается при нажатии на кнопку.

## Аутентификация с помощью Magic Link

**Supabase** также предлагает возможность отправить пользователям волшебную ссылку на их электронную почту, нажав на
которую, они перейдут в приложение и зарегистрируются. Отправленная ссылка перенаправит их на ваш сайт, поэтому нам
нужно убедиться, что в настройках **Supabase** указан правильный URL перенаправления.

Перейдите на страницу **Auth -> Settings** в панели **Supabase** для вашего проекта и убедитесь, что **URL localhost**
находится в поле **Site URL**.

## Создание компонента MagicLink

Создайте новый файл в папке компонентов под названием **MagicLink.vue** и добавьте следующий код:

```vue
<template>
  <div>
    <h2>Sign in With Magic Link</h2>
    <form @submit.prevent="handleMagicLink">
      <div>
        <label for="email">Email</label>
        <input id="email" type="email" v-model="email" />
      </div>
      <div>
        <button type="submit">Sign in</button>
      </div>
    </form>
  </div>
</template>

<script>
import { ref } from "vue";
import { supabase } from "../supabase";

export default {
  setup() {
    const email = ref("");

    const handleMagicLink = async () => {
      try {
        // We call the signIn method from Supabase to send the magic link. We only pass it the email though.
        const { error } = await supabase.auth.signIn({
          email: email.value,
        });
        if (error) throw error;
      } catch (error) {
        alert(error.error_description || error.message);
      }
    };

    return {
      email,
      handleMagicLink,
    };
  },
};
</script>
```

Этот компонент очень похож на компонент **SignIn**. Он использует тот же метод, но для получения волшебной ссылки мы
передаем только электронную почту.

Теперь нам нужно обновить **Auth.vue**, чтобы он также использовал компонент **MagicLink**. Обновите **Auth.vue** следующим образом:

```vue
<template>
  <div>
    <!-- v-if logic to determine which auth component to show -->
    <sign-up v-if="isSignUp && !useMagicLink" />
    <sign-in v-else-if="!isSignUp && !useMagicLink" />
    <magic-link v-else />
    <div v-if="!useMagicLink">
      <button v-if="!useMagicLink" @click="isSignUp = !isSignUp">
        {{
          isSignUp
            ? "Already have an account? Sign In"
            : "Don't have an account yet? Sign Up"
        }}
      </button>
      <p>Or</p>
    </div>
    <button @click="toggleMagicLink">
      {{
        useMagicLink
          ? "Sign in with email and password"
          : "Sign in with magic link"
      }}
    </button>
  </div>
</template>

<script>
import { ref } from "vue";
import SignUp from "./SignUp.vue";
import SignIn from "./SignIn.vue";
import MagicLink from "./MagicLink.vue";
export default {
  components: { SignUp, SignIn, MagicLink },
  setup() {
    const isSignUp = ref(true);
    const useMagicLink = ref(false);

    function toggleMagicLink() {
      useMagicLink.value = !useMagicLink.value;
    }

    return {
      isSignUp,
      useMagicLink,

      toggleMagicLink,
    };
  },
};
</script>
```

Если вы введете свой **email** и нажмете "Войти", вы должны получить письмо с волшебной ссылкой. Нажмите на эту ссылку, и вы
будете перенаправлены в приложение как зарегистрированный пользователь, где вы увидите представление **HelloWorld**.

## Заключение

**Supabase** позволяет относительно легко настроить аутентификацию. Они также обеспечивают аутентификацию с помощью
нескольких социальных провайдеров, таких как **Google**, **Apple**, **Github** и многих других.







