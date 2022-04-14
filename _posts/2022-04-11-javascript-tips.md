---
layout: post
title: Проверка нескольких критериев в условии
tags: [JS советы]
comments : False
---

Если вам нужно проверить много условий в вашем **if**,
вместо того, чтобы дополнять это заявление более **"||"**,
перепишите условие с помощью **Array.includes**.

Альтернативой является использование **Set** и его метода **'has'**.

Короче и декларативнее!

![Проверка нескольких критериев в условии в JS]({{ site.baseurl }}/assets/img/js-tips/2022-04-11_11-58-43.png)
