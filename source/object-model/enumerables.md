В Ember.js перечислимый объект — это любой объект, который содержит некоторое количество дочерних объектов и позволяет вам работать с ними при помощи [Ember.Enumerable](http://emberjs.com/api/classes/Ember.Enumerable.html) API. Самый распространенный перечислимый объект в большинстве приложений — встроенный массив JavaScript. В Ember.js его расширили, чтобы адаптировать под интерфейс Enumerable.

Ember.js обеспечивает стандартизированный интерфейс для работы с перечислимыми объектами. Он также позволяет вам полностью изменить способ хранения соответствующих данных без необходимости корректировать другие части вашего приложения, которые имеют к ним доступ.

Enumerable API максимально соответствует спецификациям ECMAScript. Это минимизирует несовместимость с другими библиотеками и позволяет Ember.js использовать встроенные реализации браузера в массивах, где это доступно.

## Использование методов наблюдения и свойств

Чтобы Ember отслеживал изменения в перечислимом объекте, вам нужно использовать специальные методы, которые обеспечивает `Ember.Enumerable`. Например, если вы добавите элемент в массив с помощью стандартного метода JavaScript `push()`, то Ember не сможет отследить это изменение. А если вы примените метод Enumerable `pushObject()`, изменение пройдет через ваше приложение.

Ниже приведен список стандартных методов массива JavaScript и их эквиваленты для Enumerable:

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

В остальной части руководства мы рассмотрим несколько самых выдающихся возможностей перечислимых объектов. Если хотите ознакомиться с полным списком, то загляните в [соответствующую документацию Ember.Enumerable API](http://emberjs.com/api/classes/Ember.Enumerable.html).

### Выполнение перебора перечислимого объекта

Чтобы вывести все значения перечислимого объекта, используйте метод `forEach()`:

```javascript
var food = ['Poi', 'Ono', 'Adobo Chicken'];

food.forEach(function(item, index) {
  console.log(`Menu Item ${index+1}: ${item}`);
});

// Menu Item 1: Poi
// Menu Item 2: Ono
// Menu Item 3: Adobo Chicken
```

### Первый и последний объект

Все перечислимые объекты показывают свойства `firstObject` и `lastObject`, которые вы можете связывать.

```javascript
var animals = ['rooster', 'pig'];

animals.get('lastObject');
//=> "pig"

animals.pushObject('peacock');

animals.get('lastObject');
//=> "peacock"
```

### Map

Вы легко можете преобразовать любой элемент в перечислимый объект с помощью метода `map()`. Он создает новый массив с результатами вызова функции по каждому элементу в перечислимом объекте.

```javascript
var words = ['goodbye', 'cruel', 'world'];

var emphaticWords = words.map(function(item) {
  return item + '!';
});
// ["goodbye!", "cruel!", "world!"]
```

Если ваш перечислимый объект состоит из нескольких объектов, то можно применить метод `mapBy()`. Он извлечет названное свойство из каждого объекта по очереди и вернет новый массив:

```javascript
var hawaii = Ember.Object.create({
  capital: 'Honolulu'
});

var california = Ember.Object.create({
  capital: 'Sacramento'
});

var states = [hawaii, california];

states.mapBy('capital');
//=> ["Honolulu", "Sacramento"]
```

### Фильтрация
Еще перечислимые объекты используют в качестве входных данных, чтобы вернуть массив после фильтрации по некоторому критерию.

Для произвольной фильтрации используйте метод `filter()`. Он ожидает обратный вызов, чтобы вернуть значение `true`, если Ember необходимо включить элемент в конечный массив, и `false` или `undefined`, если не нужно.

```javascript
var arr = [1,2,3,4,5];

arr.filter(function(item, index, self) {
  return item < 4;
})

// returns [1,2,3]
```

При работе с коллекцией объектов Ember вы часто будете отфильтровывать набор объектов по значению некоторого свойства. Метод `filterBy()` предоставляет самый короткий путь для этого.

```javascript
Todo = Ember.Object.extend({
  title: null,
  isDone: false
});

todos = [
  Todo.create({ title: 'Write code', isDone: true }),
  Todo.create({ title: 'Go to sleep' })
];

todos.filterBy('isDone', true);

// returns an Array containing only items with `isDone == true`
```

Если вы хотите вернуть только первое подходящее значение, а не массив со всеми подобранными значениями, то можете использовать `find()` и `findBy()`. Они работают также как `filter()` и `filterBy()`, но возвращают только один элемент.

### Сбор информации (всей или частичной)

Если вы хотите выяснить, соответствует ли каждый элемент перечислимого объекта определенному условию, то можете использовать метод `every()`:

```javascript
Person = Ember.Object.extend({
  name: null,
  isHappy: false
});

var people = [
  Person.create({ name: 'Yehuda', isHappy: true }),
  Person.create({ name: 'Majd', isHappy: false })
];

people.every(function(person, index, self) {
  return person.get('isHappy');
});

// returns false
```

Если вам нужно узнать, соответствует ли хотя бы один элемент перечислимого объекта определенным условиям, то можете использовать метод `any()`:

```javascript
people.any(function(person, index, self) {
  return person.get('isHappy');
});

// returns true
```

Как и в случае с методами фильтрации, `every()` и `any()` имеют аналогичные методы `isEvery()` и `isAny()`.

```javascript
people.isEvery('isHappy', true) // false
people.isAny('isHappy', true)  // true
```