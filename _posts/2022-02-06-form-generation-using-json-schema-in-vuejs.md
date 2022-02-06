---
layout: post
title: Простая и быстрая генерация форм с использованием схемы JSON в VueJS
tags: [VueJS, JSON]
comments: false
---

Разработчики, всегда находят способы сделать все быстро и эффективно. Например, клиент потребовал интегрировать форму и
настроить ее в проекте с помощью vuetify-jsonschema-form. Поиск подходящего информативного руководства по этому вопросу
был настоящим испытанием. Поэтому, чтобы уменьшить трудности и упростить процесс была написана эта статья по генерации формы с
использованием JSON-схемы в VueJS, которая покажет без лишней суеты технику реализации vuetify-jsonschema-form для
генерации и настройки формы.

Вот некоторые преимущества внедрения vuetify-jsonschema-form:

- Поддерживает все основные типы данных.
- Позволяет реализовывать вложенные объекты и вложенные массивы.
- Поддерживает различные варианты отображения.
- Поддерживает валидацию по предоставленной схеме.
- Позволяет выводить содержимое с помощью слотов.
- Обеспечивает согласованность и возможность повторного использования.

Существует множество пакетов, поддерживающих jsonSchema. Но в этом руководстве мы рассмотрим
@koumoul/vuetify-jsonschema-form и реализуем несколько расширенных возможностей.

## Начальная установка

Для первоначальной настройки используйте перечисленные ниже команды.

```
vue create schema-app
cd schema-app
```

## Установка зависимостей

```
vue add vuetify
npm i --save @koumoul/vuetify-jsonschema-form
```

Поскольку это небольшое демонстрационное приложение, структура папок будет довольно простой. Вот основная структура.

![Поскольку это небольшое демонстрационное приложение, структура папок будет довольно простой.]({{ site.baseurl }}/assets/img/posts/Install-Dependencies-min.png)

И main.js будет выглядеть так:

{% highlight js %}
import Vue from 'vue'
import App from './App.vue'
import vuetify from './plugins/vuetify'

Vue.config.productionTip = false

new Vue({
  vuetify,
  render: h => h(App)
}).$mount('#app')
{% endhighlight %}

## Настройка SchemaForm.vue

SchemaForm.vue будет нашим основным компонентом, который будет содержать логику и пользовательский интерфейс демо-версии. Импортируйте необходимые зависимости и CSS-файл в компонент SchemaForm.vue, как показано ниже.

`// SchemaForm.vue`

{% highlight js %}
import Vue from 'vue'
import Vuetify from 'vuetify'
import 'vuetify/dist/vuetify.min.css'
import Draggable from 'vuedraggable'
import Swatches from 'vue-swatches'
import 'vue-swatches/dist/vue-swatches.min.css'
import VJsonschemaForm from '@koumoul/vuetify-jsonschema-form'
import '@koumoul/vuetify-jsonschema-form/dist/main.css'
import { Sketch } from 'vue-color'

Vue.use(Vuetify)

Vue.component('swatches', Swatches)
Vue.component('draggable', Draggable)
Vue.component('color-picker', Sketch)

export default {
    name: 'SchemaForm',
    components: {
      VJsonschemaForm,
    },
}
{% endhighlight %}

Передача значений в реквизиты

После импорта пакета настало время его использовать. Пакет предоставляет реквизиты, которым вы можете передавать значения.

```vue
<template>
  <v-container>
    <h1>Create Form using json-schema</h1>
    <v-jsonschema-form
      v-if="schemaData && Object.keys(schemaData).length > 0"
      :schema="schemaData"
      :model="modelData"
      :options="schemaOptions"
    />
    <v-btn color="light-blue" @click="saveJson">Save</v-btn>
  </v-container>
</template>
```

**Объяснение**:

- v-if будет проверять, есть ли свойства в schemaData или нет.
- Schema prop библиотека понимает как структуру полей вашей формы. Здесь schema prop принимает объект schemaData в
  качестве значения, и таким образом пользовательский интерфейс отображает данные поля.
- Model prop обозначает значения, которые мы заполняем в форме и которые мы получим в объекте modelData. Таким образом,
  все данные, введенные пользователем, будут сохранены в modelData и далее отправлены в model prop.
- Options prop обозначает дополнительные опции, которые мы хотим предоставить.

Пример:

```js
schemaOptions: {
    debug: false,
    disableAll: false,
    autoFoldObjects: true
}
```

Здесь представлены исходные данные и методы, определенные в демо-версии.

```js
data: () => ({
  schemaData: require("./schema"),
  modelData: {},
  schemaOptions: {
    "accordionMode": "normal"
  }
}),
```

Здесь schemaData будет получать поля формы, которые мы определили в файле schema.json. Таким образом, структура нашей
формы определяется на основе полей и типов, которые мы определили в файле JSON.

Давайте посмотрим на наш файл schema.json.

`schema.json`

```js
{
  "type": "object",
  "title": "",
  "required": ["Name"],
  "properties": {
    "Name": {
      "type": "string",
      "title": "Name "
    },
     "Gender": {
      "type": "string",
      "title": "Gender ",
      "enum": ["Male", "Female", "Other"],
      "attrs": {
        "placeholder": "Select gender",
        "title": "Please select gender"
      }
    },
    "FavColor": {
      "title": "Select favourite color using color picker",
      "description": "In hex format",
      "type": "string",
      "format": "hexcolor",
      "x-display": "color-picker"
    },
    "Active": {
      "type": "boolean",
      "title": "Profile Active?",
      "description": "",
      "default": true,
      "attrs": {
        "type": "switch"
      }
    },
    "Hobby": {
      "type": "array",
      "title": "Hobbies",
      "items": {
        "$ref": "#/definitions/hobby"
      }
    },
    "PaymentInfo": {
      "type": "object",
      "title": "Payment Info",
      "oneOf": [
        {
          "$ref": "#/definitions/creditCard"
        },
        {
          "$ref": "#/definitions/upi"
        }
      ]
    }
  },
  "definitions": {
    "hobby": {
      "type": "object",
      "properties": {
        "Hobby": {
          "type": "string"
        },
        "isCertified": {
          "title": "Is certified?",
          "type": "boolean"
        }
      }
    },
    "creditCard": {
      "title": "Main infos",
      "properties": {
        "address": {
          "type": "string",
          "maxLength": 2000
        },
        "credit_card": {
          "type": "number"
        }
      },
      "dependencies": {
        "credit_card": {
          "properties": {
            "billing_address": {
              "type": "string"
            }
          },
          "required": ["billing_address"]
        }
      }
    },
    "upi": {
      "title": "UPI",
      "properties": {
        "upiId": {
          "type": "string"
        }
      }
    }
  }
}
```

Для этой схемы формы мы можем непосредственно определить тип, если это необходимо, не заботясь о его HTML. Например,
дата-время, выпадающий список, выбор цвета, выбор диапазона, переключатель и т.д.

Здесь поле **Hobby** представляет собой массив объектов, который будет определен позже в определениях. Поле **PaymentInfo**
позволяет нам выбрать одну из ссылок (CreditCard/UPI). Оно также поддерживает зависимые поля, если мы хотим отображать
другие поля на условной основе, например, адрес счета будет отображаться только в том случае, если введен номер
кредитной карты. Это также экономит время разработчиков Frontend и Backend на сохранение и предварительное заполнение
форм.

Итак, речь шла о генерации форм с использованием схемы JSON в VueJS. Существует еще много дополнительных функций и
свойств, предлагаемых vuetify-jsonschema-form. Это была лишь базовая реализация. Не стесняйтесь изучать больше и
генерировать формы.

## Репозиторий Github: Генерация форм с использованием схемы JSON в VueJS

Вы можете скачать [демо-приложение](https://github.com/riddhi-adhvaryu-bacancy/vue-schema) в репозитории Github и поиграться с кодом или выполнить описанные выше шаги для
разработки приложения.