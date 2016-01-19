## Что такое вычислительные свойства?

Если коротко, то вычислительные свойства позволяют объявлять функции в качестве свойств. Вы создаете их, когда определяете вычислительное свойство как функцию, которую Ember автоматически вызовет при запросе этого свойства. Вы можете использовать их так же, как обычное, статичное свойство.

Крайне удобно брать одно или несколько обычных свойств и преобразовывать их или управлять их данными, чтобы создать новое значение.

### Вычислительные свойства в действии

Начнем с простого примера:

```javascript
Person = Ember.Object.extend({
  // these will be supplied by `create`
  firstName: null,
  lastName: null,

  fullName: Ember.computed('firstName', 'lastName', function() {
    return `${this.get('firstName')} ${this.get('lastName')}`;
  })
});

var ironMan = Person.create({
  firstName: 'Tony',
  lastName:  'Stark'
});

ironMan.get('fullName'); // "Tony Stark"
```

Здесь мы объявляем, что функция является вычислительным свойством. А аргументы указывают Ember, что она зависит от атрибутов `firstName` и `lastName`.

Каждый раз, когда вы получаете доступ к свойству `fullName`, вызывается эта функция. Она вернет значение функции в виде `firstName` + `lastName`.

### Связывание вычислительных свойств

Вы можете использовать вычислительные свойства в качестве значений, чтобы создать новые вычислительные свойства. Давайте добавим к предыдущему примеру вычислительное свойство `description`, используем существующее свойство `fullName` и добавим еще несколько других:

```javascript
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

var captainAmerica = Person.create({
  firstName: 'Steve',
  lastName: 'Rogers',
  age: 80,
  country: 'USA'
});

captainAmerica.get('description'); // "Steve Rogers; Age: 80; Country: USA"
```

### Динамическое обновление

По умолчанию, вычислительные свойства отслеживают любые изменения, которые сделаны в родительских свойствах, и динамически обновляются при вызове. Давайте посмотрим, как это работает.

```javascript
captainAmerica.set('firstName', 'William');

captainAmerica.get('description'); // "William Rogers; Age: 80; Country: USA"
```

Итак, вычислительное свойство `fullName` обнаруживает изменения в `firstName`. В свою очередь, за `fullName` наблюдает свойство `description`.

Изменение любого зависимого свойства повлияет на всю цепочку вычислительных свойств, которую вы создали.

### Установление вычислительных свойств

Вы можете указать Ember, что следует делать при назначении вычислительного свойства. Если вы попробуете установить вычислительное свойство, оно будет вызвано с ключом (именем свойства), значением, которое вы хотите присвоить, и предыдущим значением.

```javascript
Person = Ember.Object.extend({
  firstName: null,
  lastName: null,

  fullName: Ember.computed('firstName', 'lastName', {
    get(key) {
      return `${this.get('firstName')} ${this.get('lastName')}`;
    },
    set(key, value) {
      var [firstName, lastName] = value.split(/\s+/);
      this.set('firstName', firstName);
      this.set('lastName',  lastName);
      return value;
    }
  })
});


var captainAmerica = Person.create();
captainAmerica.set('fullName', 'William Burnside');
captainAmerica.get('firstName'); // William
captainAmerica.get('lastName'); // Burnside
```   