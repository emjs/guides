По мере изучения Ember вы будете встречать в коде `Ember.Component.extend()` и `DS.Model.extend()`.
Здесь вы узнаете о методе `extend()` и о других главных особенностях модели объекта Ember.

## Определение классов

Чтобы определить в Ember новый класс, вызовите метод [`extend()`](http://emberjs.com/api/classes/Ember.Object.html#method_extend) с [`Ember.Object`](http://emberjs.com/api/classes/Ember.Object.html):

```js
const Person = Ember.Object.extend({
  say(thing) {
    alert(thing);
  }
});
```

Так мы определяем новый класс `Person` с методом `say()`.

Еще с помощью метода `extend()` можно создать *подкласс* любого из существующих классов. Например, вам нужно создать подкласс встроенного в Ember класса [`Ember.Component`](http://emberjs.com/api/classes/Ember.Component.html):

`app/components/todo-item.js`
```js
export default Ember.Component.extend({
  classNameBindings: ['isUrgent'],
  isUrgent: true
});
```

## Переопределение методов родительского класса

При создании подкласса вы можете переопределить методы. При этом вы будете иметь доступ к реализации родительского класса, если вызовите специальный метод  `_super()`:

```js
const Person = Ember.Object.extend({
  say(thing) {
    alert(`${this.get('name')} says: ${thing}`);
  }
});

const Soldier = Person.extend({
  say(thing) {
    // this will call the method in the parent class (Person#say), appending
    // the string ', sir!' to the variable `thing` passed in
    this._super(`${thing}, sir!`);
  }
});

let yehuda = Soldier.create({
  name: 'Yehuda Katz'
});

yehuda.say('Yes'); // alerts "Yehuda Katz says: Yes, sir!"
```

В некоторых случаях вам будет нужно передать аргументы методу `_super()` до или после переопределения.

Это позволяет исходному методу и дальше функционировать в нормальном режиме.

В пример можно привести переопределение hook'а [`normalizeResponse()`](http://emberjs.com/api/data/classes/DS.JSONAPISerializer.html#method_normalizeResponse) в одном из сериализаторов Ember Data.

Здесь можно использовать «оператор расширения», например, `...arguments`, в качестве полезного сокращения:

```js
normalizeResponse(store, primaryModelClass, payload, id, requestType)  {
  // Customize my JSON payload for Ember-Data
  return this._super(...arguments);
}
```

Пример выше возвращает исходные аргументы (после ваших настроек) родительскому классу, и класс продолжает функционировать в обычном режиме.

## Создание экземпляров

Как только вы определили класс, с помощью метода [`create()`](http://emberjs.com/api/classes/Ember.Object.html#method_create) можно создать новые экземпляры этого класса. Все методы, свойства и вычисляемые свойства, которые вы определили для класса, будут доступны экземплярам:

```js
const Person = Ember.Object.extend({
  say(thing) {
    alert(`${this.get('name')} says: ${thing}`);
  }
});

let person = Person.create();

person.say('Hello'); // alerts " says: Hello"
```

При создании экземпляра вы можете присвоить значения его свойств посредством передачи дополнительного хеша методу `create()`:

```js
const Person = Ember.Object.extend({
  helloWorld() {
    alert(`Hi, my name is ${this.get('name')}`);
  }
});

let tom = Person.create({
  name: 'Tom Dale'
});

tom.helloWorld(); // alerts "Hi, my name is Tom Dale"
```

Имейте в виду, что когда вы вызываете `create()`, из соображений производительности вы не можете переназначать вычисляемые свойства экземпляра и не должны переназначать существующие или определять новые методы. При использовании метода `create()` следует устанавливать только простые свойства.
Если нужно определить или переназначить методы или вычисляемые свойства, создайте новый подкласс и его экземпляр.

Согласно правилу, свойства или переменные, которые относятся к классам, пишут в стиле PascalCased, а экземпляры — нет. Например, переменная `Person` будет относиться к классу, а `person` — к экземпляру (класса `Person`). Вам следует придерживаться тех же правил наименования в своих приложениях.

## Инициализация экземпляров

Когда вы создаете новый экземпляр, автоматически вызывается метод [`init()`](http://emberjs.com/api/classes/Ember.Object.html#method_init). Это идеальное место для настройки новых экземпляров:

```js
const Person = Ember.Object.extend({
  init() {
    alert(`${this.get('name')}, reporting for duty!`);
  }
});

Person.create({
  name: 'Stefan Penner'
});

// alerts "Stefan Penner, reporting for duty!"
```

Если вы создаете подкласс для класса фреймворка, например, `Ember.Component`, и переопределяете метод `init()`, обязательно вызывайте метод `this._super(...arguments)`! Если этого не сделать, родительский класс не сможет выполнить необходимую настройку, и вы увидите странное поведение в приложении.

Массивы и объекты, которые определены напрямую для любого `Ember.Object`, доступны во всех экземплярах этого объекта.

```js
const Person = Ember.Object.extend({
  shoppingList: ['eggs', 'cheese']
});

Person.create({
  name: 'Stefan Penner',
  addItem() {
    this.get('shoppingList').pushObject('bacon');
  }
});

Person.create({
  name: 'Robert Jackson',
  addItem() {
    this.get('shoppingList').pushObject('sausage');
  }
});

// Stefan and Robert both trigger their addItem.
// They both end up with: ['eggs', 'cheese', 'bacon', 'sausage']
```

Чтобы избежать такого поведения, рекомендуется инициализировать эти свойства массивов и объекта во время `init()`. Так можно гарантировать, что каждый экземпляр будет уникальным.

```js
const Person = Ember.Object.extend({
  init() {
    this.set('shoppingList', ['eggs', 'cheese']);
  }
});

Person.create({
  name: 'Stefan Penner',
  addItem() {
    this.get('shoppingList').pushObject('bacon');
  }
});

Person.create({
  name: 'Robert Jackson',
  addItem() {
    this.get('shoppingList').pushObject('sausage');
  }
});

// Stefan ['eggs', 'cheese', 'bacon']
// Robert ['eggs', 'cheese', 'sausage']
```

## Получение доступа к свойствам объекта

Чтобы получить доступ к свойствам объекта, используйте методы доступа [`get()`](http://emberjs.com/api/classes/Ember.Object.html#method_get) и [`set()`](http://emberjs.com/api/classes/Ember.Object.html#method_set):

```js
const Person = Ember.Object.extend({
  name: 'Robert Jackson'
});

let person = Person.create();

person.get('name'); // 'Robert Jackson'
person.set('name', 'Tobias Fünke');
person.get('name'); // 'Tobias Fünke'
```

Убедитесь, что вы используете именно эти методы, иначе вычисляемые свойства не будут заново рассчитаны, наблюдатели не запустятся, и шаблоны не обновятся.