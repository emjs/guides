## Что такое вычисляемые свойства?

Если коротко, то вычисляемые свойства позволяют объявлять функции в качестве свойств. Вы создаете их, когда определяете вычисляемое свойство как функцию, которую Ember автоматически вызовет при запросе этого свойства. Вы можете использовать их и как обычное, статическое свойство.

Крайне удобно брать одно или несколько обычных свойств и преобразовывать их или управлять их данными, чтобы создать новое значение.

### Вычисляемые свойства на практике

Начнем с простого примера:

```js
Person = Ember.Object.extend({
  // these will be supplied by `create`
  firstName: null,
  lastName: null,

  fullName: Ember.computed('firstName', 'lastName', function() {
    return `${this.get('firstName')} ${this.get('lastName')}`;
  })
});

let ironMan = Person.create({
  firstName: 'Tony',
  lastName:  'Stark'
});

ironMan.get('fullName'); // "Tony Stark"
```

Здесь мы объявляем `fullName` вычисляемым свойством с `firstName` и `lastName` в качестве свойств, от которых оно зависит. Когда вы в первый раз получаете доступ к свойству `fullName`, функция, которая определена для вычисляемого свойства (то есть последний аргумент), будет запущена, а результаты помещены в кэш. При последующем доступе к `fullName` результаты будут считываться из кэша без вызова функции. Изменение любых свойств-зависимостей делает кэш недействительным, поэтому вычисляемая функция запускается снова при следующем доступе.

Если вы хотите установить зависимость от свойства, которое принадлежит объекту, можете установить несколько ключей-зависимостей с помощью раскрытия скобок:

```js
let obj = Ember.Object.extend({
  baz: {foo: 'BLAMMO', bar: 'BLAZORZ'},

  something: Ember.computed('baz.{foo,bar}', function() {
    return this.get('baz.foo') + ' ' + this.get('baz.bar');
  })
});
```

Это позволяет вам отслеживать `foo` и `bar` в `baz` без лишнего дублирования/избыточности, когда ваши ключи-зависимости в целом похожи.

### Связывание вычисляемых свойств

Вы можете использовать вычисляемые свойства в качестве значений, чтобы создавать новые вычисляемые свойства. Добавим к предыдущему примеру вычисляемое свойство `description`, используем существующее свойство `fullName` и добавим еще несколько других:

```js
Person = Ember.Object.extend({
  firstName: null,
  lastName: null,
  age: null,
  country: null,

  fullName: Ember.computed('firstName', 'lastName', function() {
    return `${this.get('firstName')} ${this.get('lastName')}`;
  }),

  description: Ember.computed('fullName', 'age', 'country', function() {
    return `${this.get('fullName')}; Age: ${this.get('age')}; Country: ${this.get('country')}`;
  })
});

let captainAmerica = Person.create({
  firstName: 'Steve',
  lastName: 'Rogers',
  age: 80,
  country: 'USA'
});

captainAmerica.get('description'); // "Steve Rogers; Age: 80; Country: USA"
```

### Динамическое обновление

По умолчанию, вычисляемые свойства отслеживают любые изменения в свойствах, от которых они зависят, и динамически обновляются при вызове. Посмотрим, как это работает.

```js
captainAmerica.set('firstName', 'William');

captainAmerica.get('description'); // "William Rogers; Age: 80; Country: USA"
```

Итак, вычисляемое свойство `fullName` обнаружило изменение в `firstName`. В свою очередь, за `fullName` наблюдает свойство `description`.

Изменение любого свойства-зависимости повлияет на всю созданную вами цепочку вычисляемых свойств.

### Определение вычисляемых свойств

Вы можете указать Ember, что следует делать при определении вычисляемого свойства. Если вы попробуете определить вычисляемое свойство, оно будет вызвано с ключом (именем свойства) и значением, которое вы хотите ему присвоить. Вы должны вернуть новое значение вычисляемого свойства из задающей функции:

```js
Person = Ember.Object.extend({
  firstName: null,
  lastName: null,

  fullName: Ember.computed('firstName', 'lastName', {
    get(key) {
      return `${this.get('firstName')} ${this.get('lastName')}`;
    },
    set(key, value) {
      let [firstName, lastName] = value.split(/\s+/);
      this.set('firstName', firstName);
      this.set('lastName',  lastName);
      return value;
    }
  })
});


let captainAmerica = Person.create();
captainAmerica.set('fullName', 'William Burnside');
captainAmerica.get('firstName'); // William
captainAmerica.get('lastName'); // Burnside
```

### Макросы вычисляемых свойств

Некоторые типы вычисляемых свойств встречаются часто. Ember предоставляет несколько макросов, которые служат сокращенным вариантом выражения определенных типов вычисляемых свойств.

В этом примере два вычисляемых свойства эквивалентны:

```js
Person = Ember.Object.extend({
  fullName: 'Tony Stark',

  isIronManLongWay: Ember.computed('fullName', function() {
    return this.get('fullName') === 'Tony Stark';
  }),

  isIronManShortWay: Ember.computed.equal('fullName', 'Tony Stark')
});
```

Полный список макросов можно посмотреть в [документации API](http://emberjs.com/api/classes/Ember.computed.html).