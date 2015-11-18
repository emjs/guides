Вы изучаете Ember, поэтому будете встречать код вроде `Ember.Component.extend()` и `DS.Model.extend()`. Здесь вы узнаете о методе `extend()` и о других главных особенностях модели объекта Ember.

### Определение классов

Чтобы определить в Ember новый класс, вызовите метод `extend()` в `Ember.Object`:

```javascript
Person = Ember.Object.extend({
  say(thing) {
    alert(thing);
  }
});
```

Так мы добавляем новый класс `Person` с методом `say()`.

Еще с помощью метода `extend()` можно создать *подкласс* любого из существующих классов. Например, вам нужно создать подкласс встроенного в Ember класса `Ember.Component`:

`app/components/todo-item.js`
```javascript
export default Ember.Component.extend({
  classNameBindings: ['isUrgent'],
  isUrgent: true
});
```

Когда вы определяете подкласс, можете переопределить методы. При этом вы будете иметь доступ к реализации родительского класса, если вызовите специальный метод `_super()`:

```javascript
Person = Ember.Object.extend({
  say(thing) {
    var name = this.get('name');
    alert(`${name} says: ${thing}`);
  }
});

Soldier = Person.extend({
  say(thing) {
    // this will call the method in the parent class (Person#say), appending
    // the string ', sir!' to the variable `thing` passed in
    this._super(thing + ', sir!');
  }
});

var yehuda = Soldier.create({
  name: 'Yehuda Katz'
});

yehuda.say('Yes'); // alerts "Yehuda Katz says: Yes, sir!"
```

### Создание экземпляров

Как только вы определили класс, с помощью метода `create()` можно создать новые *экземпляры* этого класса. Все методы, характеристики и вычислительные свойства, которые вы присвоили классу, будут доступны экземплярам:

```javascript
var person = Person.create();
person.say('Hello'); // alerts " says: Hello"
```

При создании экземпляра вы можете присвоить значения его свойств посредством передачи дополнительного хеша методу `create()`:

```javascript
Person = Ember.Object.extend({
  helloWorld() {
    alert('Hi, my name is ' + this.get('name'));
  }
});

var tom = Person.create({
  name: 'Tom Dale'
});

tom.helloWorld(); // alerts "Hi, my name is Tom Dale"
```

Имейте в виду, что когда вы вызываете `create()`, из соображений производительности вы не сможете повторно определить или назначить экземпляру новые вычислительные свойства или методы. При использовании метода `create()` следует устанавливать только простые свойства. Если нужно определить или переназначить методы или вычислительные свойства, создайте новый подкласс и его экземпляр.

Согласно правилу, свойства или переменные, которые относятся к классам, пишут в стиле PascalCased, а экземпляры нет. Например, переменная `Person` будет относиться к классу, а `person` — к экземпляру (класса `Person`). Вам следует придерживаться тех же правил в своих приложениях.

### Инициализация экземпляров

Когда вы создаете новый экземпляр, автоматически вызывается метод `init`. Это идеальное место для настройки, которая требуется новым экземплярам:

```js
Person = Ember.Object.extend({
  init() {
    var name = this.get('name');
    alert(name + ', reporting for duty!');
  }
});

Person.create({
  name: 'Stefan Penner'
});

// alerts "Stefan Penner, reporting for duty!"
```


Если вы создаете подкласс для класса фреймворка вроде `Ember.Component` и переопределяете метод `init`, то обязательно вызывайте метод `this._super(...arguments)`! Если этого не сделать, то родительский класс не сможет выполнить настройку, и вы увидите странное поведение в приложении.

### Получение доступа к свойствам объекта

Чтобы получить доступ к свойствам объекта, используйте методы доступа `get` и `set`:

```js
var person = Person.create();

var name = person.get('name');
person.set('name', 'Tobias Fünke');
```

Убедитесь, что вы используете именно эти методы, иначе вы не рассчитаете вычислительные свойства, не запустите наблюдателей и не обновите шаблоны.