---
layout: post
title: Не забывайте об автоматической вставке точки с запятой
tags: [JS советы]
comments : False
---

Как вы думаете, эти два фрагмента кода возвращают одно и то же?
Вы можете быть удивлены, попробовав их оба в своей консоли.

```js
𝚏𝚞𝚗𝚌𝚝𝚒𝚘𝚗 𝚐𝚎𝚝𝚂𝚞𝚙𝚎𝚛𝚑𝚎𝚛𝚘() {
    𝚛𝚎𝚝𝚞𝚛𝚗 
    {
        𝚗𝚊𝚖𝚎: '𝚂𝚞𝚙𝚎𝚛𝚖𝚊𝚗'
    }
}
𝚐𝚎𝚝𝚂𝚞𝚙𝚎𝚛𝚑𝚎𝚛𝚘()
```

и

```js
𝚏𝚞𝚗𝚌𝚝𝚒𝚘𝚗 𝚐𝚎𝚝𝚂𝚞𝚙𝚎𝚛𝚑𝚎𝚛𝚘() {
    𝚛𝚎𝚝𝚞𝚛𝚗 {
        𝚗𝚊𝚖𝚎: '𝚂𝚞𝚙𝚎𝚛𝚖𝚊𝚗'
    }
}
𝚐𝚎𝚝𝚂𝚞𝚙𝚎𝚛𝚑𝚎𝚛𝚘()
```

Почему первый возвращает 'undefined'?
Из-за автоматической вставки точки с запятой.

JS предполагает наличие ";" в некоторых местах вашего JS-кода, даже если вы его там не написали.

В первой функции JS добавляет точку с запятой после 'return', поэтому функция становится:

```js
𝚏𝚞𝚗𝚌𝚝𝚒𝚘𝚗 𝚐𝚎𝚝𝚂𝚞𝚙𝚎𝚛𝚑𝚎𝚛𝚘() {
    𝚛𝚎𝚝𝚞𝚛𝚗; // -> 𝚛𝚎𝚝𝚞𝚛𝚗 (𝚞𝚗𝚍𝚎𝚏𝚒𝚗𝚎𝚍);
    {
        𝚗𝚊𝚖𝚎: '𝚂𝚞𝚙𝚎𝚛𝚖𝚊𝚗'
    };
}
```

![Не забывайте об автоматической вставке точки с запятой]({{ site.baseurl }}/assets/img/js-tips/2022-02-07_13-05-08.png)

