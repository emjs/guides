В Ember.js перечислимый объект — это любой объект, который содержит несколько дочерних объектов и позволяет вам работать с ними при помощи [Ember.Enumerable](http://emberjs.com/api/classes/Ember.Enumerable.html) API. Самый распространенный перечислимый объект в большинстве приложений — встроенный массив JavaScript. В Ember.js его расширили, чтобы адаптировать под интерфейс Enumerable.

Ember.js предоставляет стандартизированный интерфейс для работы с перечислимыми объектами. Он также позволяет вам полностью изменить способ хранения соответствующих данных без необходимости корректировать другие части вашего приложения, которые имеют к ним доступ.

Enumerable API максимально соответствует спецификациям ECMAScript. Это минимизирует несовместимость с другими библиотеками и позволяет Ember.js использовать встроенные реализации браузера в массивах, где это доступно.

## Использование методов и свойств наблюдения

Чтобы Ember отслеживал изменения в перечислимом объекте, вам нужно использовать специальные методы, которые обеспечивает `Ember.Enumerable`. Например, если вы добавите элемент в массив с помощью стандартного метода JavaScript `push()`, то Ember не сможет отследить это изменение. А если вы примените метод Enumerable `pushObject()`, изменение пройдет через ваше приложение.

Ниже приведен список стандартных методов массива JavaScript и их эквиваленты для отслеживаемых перечисляемых объектов:

<table>
  <thead>
    <tr><th>Стандартный метод</th><th>Метод наблюдения</th></tr>
  </thead>
  <tbody>
    <tr><td>pop</td><td>popObject</td></tr>
    <tr><td>push</td><td>pushObject</td></tr>
    <tr><td>reverse</td><td>reverseObjects</td></tr>
    <tr><td>shift</td><td>shiftObject</td></tr>
    <tr><td>unshift</td><td>unshiftObject</td></tr>
  </tbody>
</table>

Кроме того, чтобы извлечь первый и последний объекты массива в отслеживаемой форме, следует использовать `myArray.get('firstObject')` и `myArray.get('lastObject')`. 


## Обзор API

В остальной части руководства мы рассмотрим несколько самых распространенных возможностей перечислимых объектов. Если хотите ознакомиться с полным списком, то загляните в [соответствующую документацию Ember.Enumerable API](http://emberjs.com/api/classes/Ember.Enumerable.html).

### Перебор элементов перечислимого объекта

Чтобы вывести все значения перечислимого объекта, используйте метод [`forEach()`](http://emberjs.com/api/classes/Ember.Enumerable.html#method_forEach):

```javascript
let food = ['Poi', 'Ono', 'Adobo Chicken'];

food.forEach((item, index) => {
  console.log(`Menu Item ${index+1}: ${item}`);
});

// Menu Item 1: Poi
// Menu Item 2: Ono
// Menu Item 3: Adobo Chicken
```

### Первый и последний объект

У всех перечислимых объектов есть свойства [`firstObject`](http://emberjs.com/api/classes/Ember.Enumerable.html#property_firstObject) и [`lastObject`](http://emberjs.com/api/classes/Ember.Enumerable.html#property_lastObject), к которым можно устанавливать привязки.

```javascript
let animals = ['rooster', 'pig'];

animals.get('lastObject');
//=> "pig"

animals.pushObject('peacock');

animals.get('lastObject');
//=> "peacock"
```

### Map

Вы легко можете преобразовать любой элемент в перечислимый объект с помощью метода [`map()`](http://emberjs.com/api/classes/Ember.Enumerable.html#method_map). Он создает новый массив с результатами вызова функции по каждому элементу в перечислимом объекте.

```javascript
let words = ['goodbye', 'cruel', 'world'];

let emphaticWords = words.map(item => `${item}!`);
//=> ["goodbye!", "cruel!", "world!"]
```

Если перечислимый объект состоит из нескольких объектов, то можно применить метод [`mapBy()`](http://emberjs.com/api/classes/Ember.Enumerable.html#method_mapBy). Он извлечет указанное свойство из каждого объекта по очереди и вернет новый массив:

```javascript
let hawaii = Ember.Object.create({
  capital: 'Honolulu'
});

let california = Ember.Object.create({
  capital: 'Sacramento'
});

let states = [hawaii, california];

states.mapBy('capital');
//=> ["Honolulu", "Sacramento"]
```

### Фильтрация
Перечислимые объекты можно использовать в качестве входных данных и возвращать массив после фильтрации по некоторому критерию.

Для произвольной фильтрации используйте метод [`filter()`](http://emberjs.com/api/classes/Ember.Enumerable.html#method_filter). Он ожидает обратный вызов, чтобы вернуть значение `true`, если Ember необходимо включить элемент в конечный массив, и `false` или `undefined`, если не нужно.

```javascript
let arr = [1, 2, 3, 4, 5];

arr.filter((item, index, self) => item < 4);

//=> [1, 2, 3]
```

При работе с коллекцией объектов Ember вы часто будете фильтровать набор объектов по значению некоторого свойства. Метод [`filterBy()`](http://emberjs.com/api/classes/Ember.Enumerable.html#method_filterBy) предоставляет самый короткий путь для этого.

```javascript
Todo = Ember.Object.extend({
  title: null,
  isDone: false
});

let todos = [
  Todo.create({ title: 'Write code', isDone: true }),
  Todo.create({ title: 'Go to sleep' })
];

todos.filterBy('isDone', true);

// returns an Array containing only items with `isDone == true`
```

Если вы хотите вернуть только первое подходящее значение, а не массив со всеми подобранными значениями, то можете использовать [`find()`](http://emberjs.com/api/classes/Ember.Enumerable.html#method_find) и [`findBy()`](http://emberjs.com/api/classes/Ember.Enumerable.html#method_findBy). Они работают также как `filter()` и `filterBy()`, но возвращают только один элемент.

### Сбор информации (всей или частичной)

Если вы хотите выяснить, соответствует ли каждый элемент перечислимого объекта определенному условию, то можете использовать метод [`every()`](http://emberjs.com/api/classes/Ember.Enumerable.html#method_every): 

```javascript
Person = Ember.Object.extend({
  name: null,
  isHappy: false
});

let people = [
  Person.create({ name: 'Yehuda', isHappy: true }),
  Person.create({ name: 'Majd', isHappy: false })
];

people.every((person, index, self) => person.get('isHappy'));

//=> false
```

Если вам нужно узнать, соответствует ли хотя бы один элемент перечислимого объекта определенному условию, то можете использовать метод [`any()`](http://emberjs.com/api/classes/Ember.Enumerable.html#method_any):

```javascript
people.any((person, index, self) => person.get('isHappy'));

//=> true
```

Как и в случае с методами фильтрации, `every()` и `any()` имеют аналогичные методы `isEvery()` и `isAny()`.

```javascript
people.isEvery('isHappy', true) // false
people.isAny('isHappy', true)  // true
```