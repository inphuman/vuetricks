---
layout: post
title: Не используйте необязательный вызов функции
tags: [JS советы]
comments : False
---

Не используйте необязательный вызов функции если значение свойства может быть равно null/undefined, конкретный 
«необязательный вызов функции» обманчиво выглядит как «вызов функции, если она вызывается», но это НЕ так!

Попробуйте вызвать функцию, если значение не является null.

Есть куча ненулевых значений, которые не являются функциями, поэтому условие в итоге будет неверным.

Чтобы проверить, можно ли что-то вызвать, используйте следующее:

{% highlight js %}
if (typeof callback === 'function') { 
    callback(); 
}
{% endhighlight %}

ИЛИ

{% highlight js %}
if (callback instanceof Function) { 
    callback(); 
}
{% endhighlight %}

![Не используйте необязательный вызов функции]({{ site.baseurl }}/assets/img/js-tips/2022-02-02-javascript-tip.png)

