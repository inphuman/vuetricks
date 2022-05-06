---
layout: post
title: Система реактивности Vue 3 великолепна! Вот как это работает
tags: [Реактивность, Vue 3]
comments: false
---

## Что понимается под реактивностью?

Быть фронтенд-разработчиком в наши дни означает, что вы ежедневно имеете дело с реактивностью. По сути, это плавное
сопоставление между состоянием приложения и DOM. Любое изменение в состоянии приложения будет немедленно отражено в DOM
без необходимости обрабатывать это вручную, просто измените состояние и позвольте фреймворку сделать остальную работу за
вас.

Проще говоря, фреймворк обрабатывает это как _"О! Цена изменилась, нужно обновить элемент DOM новой ценой и обновить любые другие
переменные, которые также зависят от этой цены"_.

## Реактивность Привет, мир! Какую проблему мы пытаемся решить?

Рассмотрим пример. У нас есть продукт, который имеет `price` и `quantity`. И есть еще одна переменная `totalPrice`,
которая вычисляется из `price` и `quantity`.

```js
let product = {price = 20, quantity: 5}
let totalPrice = product.price * product.quantity

console.log(totalPrice) // Returns 100
```

Теперь, если мы изменим цену продукта, общая цена не обновится.

```js
product.price = 30
console.log(totalPrice) // Still returns 100
```

Нам нужен способ сказать нашему коду: "Эй, `totalPrice` зависит от `price` и `quantity`. Всякий раз, когда какая-либо из переменных
изменяется, необходимо пересчитывать `totalPrice`".

Прежде всего, мы можем обернуть код, который обновляется, `totalPrice` в функцию `updateTotalPrice` и вызывать ее при
необходимости.

```js
function updateTotalPrice() {
    totalPrice = product.price * product.quantity
}

product.price = 30
updateTotalPrice()

console.log(totalPrice) // Now it returns 150
```

Теперь нам нужно вызывать `updateTotalPrice` функцию всякий раз, когда изменяется `price` или `quantity`. Но вызывать его
вручную после каждого обновления непрактично, нам нужен способ автоматически узнать, что значение изменилось, и 
следовательно, вызвать `updateTotalPrice` функцию автоматически. И здесь на помощь приходит **Javascript Proxy**.

## Знакомство с Javascript-прокси

Объект **Proxy** позволяет вам создать прокси для другого объекта, который может перехватывать и переопределять основные
операции для этого объекта.

Конструктор прокси принимает 2 параметра:

- **target**: какой исходный объект нам нужен для прокси.
- **handler**: объект, который определяет перехваченные операции и логику, которая будет выполняться.

Давайте начнем с простого примера: мы собираемся создать прокси с пустым обработчиком объектов, прокси, который ничего
не делает. Он будет вести себя так же, как исходный объект.

```js
let product = {price: 20, quantity: 5}
let proxiedProduct = new Proxy(product, {})

console.log(proxiedProduct.price) // Returns 20

proxiedProduct.price = 50
console.log(product.price) // Returns 50
```

Теперь давайте перехватим функции получения, установки и просто добавим console.log перед получением или установкой
значения. Это делается путем реализации `get` и `set` функций в объекте-обработчике прокси.

**get функция**: это ловушка для операций получения объекта и принимает 3 параметра:

- **target**: исходный объект
- **property**: свойство, значение которого мы пытаемся получить
- **receiver**: объект, который был вызван, обычно сам прокси-объект или любой унаследованный объект (мы все равно не будем его использовать на данный момент)

**set функция**: ловушка для заданных операций над объектом и принимает 4 параметра:

- **target**: исходный объект
- **property**: свойство, которому мы пытаемся установить значение
- **value**: значение, которое нам нужно установить
- **receiver**: То же, что и getфункция, но пока не буду ее использовать

```js
let product = {price: 20, quantity: 5}

let proxiedProduct = new Proxy(product, {
    get(target, property){
        console.log(`Getting value of ${property}`);
        return target[property]
    },

    set(target, property, value){
        console.log(`Setting value of ${property} to ${value}`);
        target[property] = value
        return true
    }
})

console.log(proxiedProduct.price) 
// Prints "Getting value of price"
// Then, Returns 20

proxiedProduct.price = 50  
// Prints "Setting value of price to 50"
// Then, Set value of product.price to 50
```

## Улучшение 1: updateTotalPrice функция вызова внутри сеттера

В прошлый раз нам нужен был способ сказать "Эй, всякий раз, когда `price` или `quantity` изменяются то необходимо вызывать 
`updateTotalPrice` функцию". Это казалось каким-то волшебством, которое нам нужно, чтобы произошло. Теперь у нас
есть способ автоматически определять изменение свойства. Мы можем просто вызвать **updateTotalPrice функцию** внутри нашего сеттера.

```js
let proxiedProduct = new Proxy(product, {
    get(target, property){
        // Let's keep the getter empty for now
    },

    set(target, property, value){
        target[property] = value;

        if(property === "price") updateTotalPrice();
        if(property === "quantity") updateTotalPrice();

        return true;    
    }
})
```

Теперь у нас есть то, что нам нужно. Если мы обновили цену продукта, `totalPrice` будет обновлена соответственно.

```js
console.log(totalPrice) // Returns 100

proxiedProduct.price = 30
console.log(totalPrice) //Returns 150 🎉
```

Нам не нужны избыточные вызовы функции обновления. Если значение,
`price` например, было 20, и мы также устанавливаем новое значение 20. Пересчитывать нет смысла, `totalPrice` не
изменился. Поэтому немного изменим код:

```js
let proxiedProduct = new Proxy(product, {
    get(target, property){
        // Let's keep the getter empty for now
    },

    set(target, property, value){
        let oldValue = target[property];
        target[property] = value;

        // Call update function only if the value changed
        if( oldValue !== value ) {
            if(property === "price") updateTotalPrice();
            if(property === "quantity") updateTotalPrice();
        }

        return true;    
    }
})
```

## Улучшение 2: Создание хранилища зависимостей depsMap

В другом сценарии у нас может быть функция `updatePriceAfterDiscount`, которая зависит только от `price` и должна вызываться
при `price` изменении. Здесь наше предыдущее решение не работает, нам нужно изменить наш сеттер, чтобы справиться
с этим сценарием.

```js
if(property === "price"){
    updateTotalPrice();
    updatePriceAfterDiscount();
}

if(property === "quantity") updateTotalPrice();
```

Что, если у нас есть огромное количество функций, зависящих от каких-то свойств? Нам не нужно трогать сеттер и геттер
настолько, насколько это возможно, и позволить ему выполнять свою работу автоматически. Поэтому мы собираемся выделить
свойства и соответствующие им функции в отдельное место.

Прежде всего, давайте определим **некоторые термины**, которые использует vue, чтобы лучше понять, что мы строим.

- Функция `updateTotalPrice` называется **эффектом**, так как она изменяет состояние программы.
- Свойства `price` и `quantity` называются зависимостями `updateTotalPrice` эффекта. Так как от них зависит эффект.
- `updateTotalPrice` эффект является подписчиком его зависимостей, т.е. `updateTotalPrice` является подписчиком `price` и `quantity`

Теперь, когда мы знаем термины, мы создадим карту `depsMap` которая сопоставляет каждую зависимость с соответствующим
списком эффектов, которые должны выполняться при изменении зависимости.

Например, на предыдущем изображении мы видим, что `price` свойство является зависимостью для эффектов `updateTotalPrice` и для `upadtePriceAfterDiscount`.
При каждом `price` изменении нам нужно перезапускать все эффекты.

Теперь нам нужно немного изменить сеттер `depsMap`, чтобы извлечь выгоду из созданного файла. Мы собираемся получить
список эффектов определенной зависимости и зациклить их, выполнив их все сразу.

```js
let proxiedProduct = new Proxy(product, {
    get(target, property){
        // Let's keep the getter empty for now
    },

    set(target, property, value){
        let oldValue = target[property];
        target[property] = value;

        // Call update function only if the value changed
        if( oldValue != value ) {
            // Get list of effects of dependency
            let dep = depsMap.get(property)

            // Run all the effects of this dependency
            dep.forEach(effect => {
                effect()
            })
        }

        return true;
    }
})
```

Теперь все, что нам нужно сделать, это постоянно обновлять `depsMap`, все зависимости и все эффекты, которые должны выполняться при изменении.

До сих пор это прекрасно работало, пока у нас не появилось несколько реактивных объектов. Если мы еще раз взглянем на
объект `depsMap`, то увидим, что он содержит только свойства `product` объекта. В реальности все сложнее. Мы имеем дело не с
одним объектом, а с несколькими объектами, и нам нужно, чтобы все они были реактивными. Рассмотрим случай, когда у нас
есть другой объект, `user` и мы хотим, чтобы он тоже был реактивным. Нам нужно будет создать новый `depsMap` для `user` объекта,
отличного от объекта `product`. Чтобы решить эту проблему, мы собираемся ввести новый слой `targetMap`.

## Улучшение 3: Создание targetMap

Мы собираемся построить `depsMap` для каждого реактивного объекта. Теперь нам нужен способ сопоставления между реактивным
объектом и его `depsMap`. Итак, мы собираемся создать новую карту, `targetMap` где ключ - это сам реактивный объект, а его значение - это значение `depsMap` этого объекта.

Нам нужно, чтобы ключ был в `targetMap` самим реактивным объектом, а не просто строкой.

Поэтому мы не можем использовать обычную карту для построения `targetMap`, вместо этого мы будем использовать
javascript **WeakMap**. WeakMaps - это просто пары ключ/значение (обычный объект), где ключи должны быть объектами, а
значение может быть любым допустимым типом javascript. Например:

```js
const product = {price: 20, quantity: 5};
const wm = new WeakMap(),;
wm.set(product, "this is the value");

wm.get(product) // Returns "this is the value"
```

Теперь, когда мы знаем, как построить файл `targetMap` нам также нужно обновить наш сеттер, так как теперь мы имеем дело
с несколькими `depsMaps` и нам нужно получить правильный `depsMap`. Так как в сеттере мы можем получить целевой объект. Мы
можем использовать наш `targetMap`, чтобы получить следующее:

```js
let proxiedProduct = new Proxy(product, {
    get(target, property){
        // Let's keep the getter empty for now
    },

    set(target, property, value){
        let oldValue = target[property];
        target[property] = value;

        // Call update function only if the value changed
        if( oldValue != value ) {
            // We get the correct depsMap using the target (reactive object)
            let depsMap = targetMap.get(target)
            if(!depsMap) return

            // Get list of effects of dependency
            let dep = depsMap.get(property)
            if(!dep) return

            // Run all the effects of this dependency
            dep.forEach(effect => {
                effect()
            })
        }

        return true;
    }
})
```

## Подведение итогов

До сих пор мы создали только функцию установки в прокси. Эта функция отвечает за запуск всех эффектов при изменении
свойства. Поэтому эта функция вызывается `triggerVue`.

Давайте завершим все улучшения и мы добавим новую
реактивную переменную `priceAfterDiscount`, которая зависит только от `price` свойства.

1. Во-первых, давайте определим наши реактивные переменные и их соответствующие эффекты.

```js
let product = { price: 20, quantity: 5 };
let totalPrice = product.price * product.quantity;
let priceAfterDiscount = product.price * 0.9;

// Effects
function updateTotalPrice() {
 totalPrice = product.price * product.quantity;
}

function updatePriceAfterDiscount() {
 priceAfterDiscount = product.price * 0.9;
}
```

2. Во-вторых, давайте определим и заполним `targetMap` и `depMap`

```js
const targetMap = new WeakMap();
const productDepsMap = new Map();

targetMap.set(product, productDepsMap);

productDepsMap.set('price', [updateTotalPrice, updatePriceAfterDiscount]);
productDepsMap.set('quantity', [updateTotalPrice]);
```

3. Наконец, нам нужно создать прокси для `product` объекта.

```js
// Creating the Proxied Product
let proxiedProduct = new Proxy(product, {
 get(target, property) {
   // Let's keep the getter empty for now
 },

 set(target, property, value) {
   // ... same as we've defined it in the previous section
 },
});
```

4. Вуаля, наш `product` объект теперь реактивный

```js
console.log(totalPrice); // Returns 100
console.log(priceAfterDiscount); // Returns 18

// totalPrice and priceAfterDiscount should be updated on updating price
proxiedProduct.price = 30;

console.log('✅ After Updating Price');
console.log(totalPrice); // Returns 150
console.log(priceAfterDiscount); // Returns 27

// only totalPrice should be updated on updating quantity
proxiedProduct.quantity = 10;

console.log('✅ After Updating Quantity');
console.log(totalPrice); // Returns 300
console.log(priceAfterDiscount); // Still Returns 27
```