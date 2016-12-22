Ember поддерживает отслеживание любых свойств, включая вычисляемые.

Наблюдатели должны содержать поведение, которое реагирует на изменения в другом свойстве. Наблюдатели особенно полезны, когда нужно осуществить определенное поведение после того, как синхронизируется привязка.

Новички в разработке приложений Ember часто злоупотребляют наблюдателями. Их интенсивно используют в рамках самого фреймворка Ember. Но для большинства проблем, с которыми сталкиваются разработчики, подходящим решением служат вычисляемые свойства.

Вы можете настроить наблюдатель объекта с помощью `Ember.observer`:

```js
Person = Ember.Object.extend({
  // these will be supplied by `create`
  firstName: null,
  lastName: null,

  fullName: Ember.computed('firstName', 'lastName', function() {
    return `${this.get('firstName')} ${this.get('lastName')}`;
  }),

  fullNameChanged: Ember.observer('fullName', function() {
    // deal with the change
    console.log(`fullName changed to: ${this.get('fullName')}`);
  })
});

var person = Person.create({
  firstName: 'Yehuda',
  lastName: 'Katz'
});

// observer won't fire until `fullName` is consumed first
person.get('fullName'); // "Yehuda Katz"
person.set('firstName', 'Brohuda'); // fullName changed to: Brohuda Katz
```

Так как вычисляемое свойство `fullName` зависит от `firstName`, обновление `firstName` запустит наблюдатель `fullName`.

## Наблюдатели и асинхронность

Сейчас наблюдатели в Ember синхронные. То есть они запустятся, как только изменится одно из свойств, которое они отслеживают. Это может легко привести к ошибкам, если свойства еще не синхронизировались:

```js
Person.reopen({
  lastNameChanged: Ember.observer('lastName', function() {
    // The observer depends on lastName and so does fullName. Because observers
    // are synchronous, when this function is called the value of fullName is
    // not updated yet so this will log the old value of fullName
    console.log(this.get('fullName'));
  })
});
```

Также синхронное поведение может привести к тому, что наблюдатели будут запускаться много раз при отслеживании нескольких свойств:

```js
Person.reopen({
  partOfNameChanged: Ember.observer('firstName', 'lastName', function() {
    // Because both firstName and lastName were set, this observer will fire twice.
  })
});

person.set('firstName', 'John');
person.set('lastName', 'Smith');
```

Чтобы обойти эти проблемы, следует использовать [`Ember.run.once()`](http://emberjs.com/api/classes/Ember.run.html#method_once). Это гарантирует, что любая необходимая обработка данных произойдет один раз и в следующем цикле исполнения, когда все привязки синхронизированы:

```js
Person.reopen({
  partOfNameChanged: Ember.observer('firstName', 'lastName', function() {
    Ember.run.once(this, 'processFullName');
  }),

  processFullName() {
    // This will only fire once if you set two properties at the same time, and
    // will also happen in the next run loop once all properties are synchronized
    console.log(this.get('fullName'));
  }
});

person.set('firstName', 'John');
person.set('lastName', 'Smith');
```

## Наблюдатели и инициализация объекта

Наблюдатели никогда не запускаются до завершения инициализации объекта.

Если вам нужно запустить наблюдатель, как часть процесса инициализации, то нельзя полагаться на побочный эффект `set`. Вместо этого укажите с помощью [`Ember.on()`](http://emberjs.com/api/classes/Ember.html#method_on), чтобы наблюдатель запускался после `init`:

```js
Person = Ember.Object.extend({
  init() {
    this.set('salutation', 'Mr/Ms');
  },

  salutationDidChange: Ember.on('init', Ember.observer('salutation', function() {
    // some side effect of salutation changing
  }))
});
```

## Неиспользуемые вычисляемые свойства не запускают наблюдатели

Если вы никогда не использовали метод `get()` с вычисляемым свойством, его наблюдатели не запустятся, даже если изменятся зависимые ключи.

Представьте такую картину: значение меняется с одного неизвестного значения на другое. Обычно это никак не влияет на код приложения, так как вычисляемые свойства почти всегда отслеживаются, когда их запрашивают. Например, вы получаете значение вычисляемого свойства и вносите его в модель DOM (или отображаете с помощью D3). Затем вы отслеживаете его, чтобы обновить DOM при изменении свойства.

Если вам нужно отследить вычисляемое свойство, которое вы сейчас не запрашиваете, просто сделайте это через метод `init()`.

## Без определений класса

Вы можете добавлять наблюдатели объекта без определения класса с помощью [`addObserver()`](http://emberjs.com/api/classes/Ember.Object.html#method_addObserver):

```js
person.addObserver('fullName', function() {
  // deal with the change
});
```