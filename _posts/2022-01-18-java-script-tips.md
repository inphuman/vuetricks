---
layout: post
title: Разговорная функция (Speaking Function)
tags: [JS советы]
comments : False
---

Вам нужно озвучить свое приложение? Вот решение.

Попробуйте это в консоли:

{% highlight js %}
function speak(message, language = 'en-US') {
    const msg = new SpeechSynthesisUtterance(message);
    msg.lang = language;
    speechSynthesis.speak(msg);
}

speak('May the Force be with you');
speak('Ciao, io sono Daniele', 'it-IT');
{% endhighlight %}

![Разговорная функция (Speaking Function)]({{ site.baseurl }}/assets/img/js-tips/2021-01-18-java-script-tip.png)

