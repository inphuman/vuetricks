---
layout: post
title: Плавная прокрутка вверх на JS
tags: [JS советы]
comments : False
---

Простой способ реализовать плавную прокрутку в верхнюю часть страницы.

```js
window.scrollTo({
  top: 0,
  behavior: 'smooth',
});
```

![Console.dir для регистрации HTML-элементов]({{ site.baseurl }}/assets/img/js-tips/2022-02-11_11-45-21.png)

