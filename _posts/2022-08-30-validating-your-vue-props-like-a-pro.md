---
layout: post
title: Как сохранять и получать данные с помощью локального хранилища и Vue 3
tags: [Vue, local storage, Composition API]
comments: false
---

Итак, представьте, что у нас есть данные, которые должны оставаться в компоненте при перезагрузке страницы. Эти
данные должны оставаться, поскольку пользователю необходимо взаимодействовать с ними или изменять их по какой-то
причине. В Vue 2 мы использовали Vuex store, а затем пакет, который сохранял состояние в LocalStorage или
SessionStorage. В результате Vuex хранит данные в хранилище, а затем при обновлении сохраняет их в хранилище, при
перезагрузке страницы получает данные из хранилища и возвращает их в хранилище.

На момент написания этой статьи Vue 3 не работает с Vue
persist, и нужно разработать новый способ обеспечения постоянного доступа к данным. Как мы это сделаем? Ну, мы
сделаем это, убрав посредника, которым является Vuex, и просто обратимся непосредственно к локальному хранилищу
браузера.

В качестве примера возьмем приведенный ниже код:

```js
// Sets an item with a Key to local storage
const saveStorage = function (key, data) {
    localStorage.setItem(key, JSON.stringify(data));
};
```

Как мы видим, у нас есть простая функция, которая принимает два параметра. Первый - ключ, это имя нашего элемента
хранения, что-то вроде `user` или `token`, а затем мы принимаем некоторые данные. Теперь локальное хранилище или
хранилище сеансов работает только со строками, поэтому перед тем, как передать ему данные, мы должны превратить их в
строку, что и делает для нас JSON.stringify.

Как получить данные? C помощью функции:

```js
// Looks for a local storage item and returns if present
const getStorage = function(key, item) {
    if( localStorage.getItem(key) && item) {
        const data = JSON.parse(localStorage.getItem(key))
        return data[item]
    }
    else if(localStorage.getItem(key)) {
       return localStorage.getItem(key)
    }
};
```

Итак, мы работаем с двумя сценариями:

1. Нам нужна часть некоторых данных. Например, у нас может быть список из 20 пользователей, и нам нужен пользователь 19.
   Поэтому ключом будет `users`, а элементом - `19`. Затем мы разберем возвращаемые данные, чтобы мы могли использовать
   объект.

2. Мы хотим получить все данные обратно и просто передаем ключ в функцию. Что-то вроде `token` или `user`.

И, наконец, нам нужен способ очистки данных, которые нам больше не нужны или которые мы уничтожим при переходе с сайта:

```js
// Clear a single item or the whole local storage
const clearStorage = function(key='false') {
    if(key) {
        localStorage.removeItem(key);
    } else {
        localStorage.clear();
    }
}
```

И снова у нас есть два сценария:

1. Мы хотим указать ключ и просто удалить определенный фрагмент данных.

2. Мы хотим очистить все хранилище.

И это действительно все, это работает очень хорошо и делает работу с данными в Vue SPA действительно простой, и все
компоненты имеют доступ к единому источнику данных!

Весь файл должен выглядеть следующим образом и посмотрите как это
работает:

```js
// Sets an item with a Key to local storage
const saveStorage = function(key, data) {
    localStorage.setItem(key, JSON.stringify(data));
};

// Looks for a local storage item and returns if present
const getStorage = function(key, item) {
    if( localStorage.getItem(key) && item) {
        const data = JSON.parse(localStorage.getItem(key))
        return data[item]
    }
    else if(localStorage.getItem(key)) {
       return localStorage.getItem(key)
    }
};

// Clear a single item or the whole local storage
const clearStorage = function(key='false') {
    if(key) {
        localStorage.removeItem(key);
    } else {
        localStorage.clear();
    }
}

// Exports all avaliable functions on the script
export {getStorage, saveStorage, clearStorage}
```