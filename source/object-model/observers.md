Ember поддерживает отслеживание любого свойства, включая вычислительные.

Наблюдатели должны содержать поведение, которое реагирует на изменения в другом свойстве.
Наблюдатели особенно полезны, когда нужно осуществить определенное поведение после того, как синхронизируется привязка.

Новички в разработке приложений Ember часто злоупотребляют наблюдателями. Их интенсивно используют в рамках самого фреймворка Ember. Но для большинства проблем, с которыми сталкиваются разработчики, подходящим решением служат вычислительные свойства.

Вы можете настроить наблюдателя объекта с помощью `Ember.observer`:

```javascript
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

Так как вычислительное свойство `fullName` зависит от `firstName`, обновление `firstName` соответственно запустит наблюдателей `fullName`.

### Наблюдатели и асинхронность

В настоящий момент наблюдатели в Ember синхронные. То есть они запустятся, как только изменится одно из свойств, которое они отслеживают. Это может легко привести к ошибкам, если свойства еще не синхронизировались:

```javascript
Person.reopen({
  lastNameChanged: Ember.observer('lastName', function() {
    // The observer depends on lastName and so does fullName. Because observers
    // are synchronous, when this function is called the value of fullName is
    // not updated yet so this will log the old value of fullName
    console.log(this.get('fullName'));
  })
});
```

Еще синхронное поведение может привести к тому, что наблюдатели будут запускаться много раз при отслеживании множественных свойств:

```javascript
Person.reopen({
  partOfNameChanged: Ember.observer('firstName', 'lastName', function() {
    // Because both firstName and lastName were set, this observer will fire twice.
  })
});

person.set('firstName', 'John');
person.set('lastName', 'Smith');
```

Чтобы обойти эти проблемы, следует использовать `Ember.run.once()`. Это позволит убедиться, что любая необходимая обработка данных произойдет лишь один раз. И в следующем цикле все привязки синхронизируются:

```javascript
Person.reopen({
  partOfNameChanged: Ember.observer('firstName', 'lastName', function() {
    Ember.run.once(this, 'processFullName');
  }),

  processFullName: Ember.observer('fullName', function() {
    // This will only fire once if you set two properties at the same time, and
    // will also happen in the next run loop once all properties are synchronized
    console.log(this.get('fullName'));
  })
});

person.set('firstName', 'John');
person.set('lastName', 'Smith');
```

### Наблюдатели и инициализация объекта

Наблюдатели никогда не запускаются до завершения инициализации объекта.

Если вам нужно запустить наблюдателя, как часть процесса инициализации, то вы не можете полагаться на побочный эффект `set`. Вместо этого укажите, чтобы наблюдатель запускался после `init` с помощью `Ember.on()`:

```javascript
Person = Ember.Object.extend({
  init() {
    this.set('salutation', 'Mr/Ms');
  },

  salutationDidChange: Ember.on('init', Ember.observer('salutation', function() {
    // some side effect of salutation changing
  }))
});
```

### Неиспользуемые вычислительные свойства не запускают наблюдателей

Если вы никогда не использовали метод `get()` с вычислительным свойством, его наблюдатели не запустятся, даже если изменить зависимые ключи. Представьте такую картину: значение меняется с одного неизвестного значения на другое.

Обычно это никак не влияет на код приложения, так как вычислительные свойства почти всегда отслеживаются, когда их извлекают. Например, вы получаете значение вычислительного свойства и вносите его в модель DOM (или отображаете с помощью D3). И затем вы отслеживаете его, чтобы обновить DOM при изменении свойства.

Если вам нужно отследить вычислительное свойство, которое вы сейчас не извлекаете, просто сделайте это через метод `init()`.

### Вне определений класса

Вы можете добавить наблюдателей объекта вне определения класса с помощью `addObserver()`:

```javascript
person.addObserver('fullName', function() {
  // deal with the change
});
```