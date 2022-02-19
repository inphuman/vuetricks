---
layout: post
title: Применение обязательных аргументов
tags: [JS советы]
comments : False
---

С помощью вспомогательной функции `isRequired` вы можете воспользоваться функцией назначения JS по умолчанию, чтобы
выдавать ошибку, если аргумент не передается.

```js
const isRequired = (argumentName = 'Argument') => {
    throw Error(`${argumentName} is required`);
}

const getFullName = (
    firstName = isRequired('firstName'),
    lastName = isRequired('lastName')
) => {
    return `${firstName} ${lastName}`;
}
```

![Console.dir для регистрации HTML-элементов]({{ site.baseurl }}/assets/img/js-tips/2022-02-19_13-18-54.png)

