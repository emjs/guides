Вам не нужно сразу определять все классы. Вы можете повторно открыть класс и определить новые свойства с помощью метода [`reopen()`](http://emberjs.com/api/classes/Ember.Object.html#method_reopenClass).

```js
Person.reopen({
  isPerson: true
});

Person.create().get('isPerson'); // true
```

При использовании `reopen()` вы можете переопределить существующие методы и вызвать `this._super`.

```js
Person.reopen({
  // override `say` to add an ! at the end
  say(thing) {
    this._super(thing + '!');
  }
});
```

Метод `reopen()` применяют, чтобы добавить методы и свойства экземпляра, которые будут доступны всем экземплярам класса. То есть вы не добавляете методы и свойства конкретному экземпляру класса, как в классическом JavaScript (без использования прототипа).

Но если вам нужно добавить статичные методы или свойства только классу, то можете использовать [`reopenClass()`](http://emberjs.com/api/classes/Ember.Object.html#method_reopenClass).
```js
// add static property to class
Person.reopenClass({
  isPerson: false
});
// override property of Person instance
Person.reopen({
  isPerson: true
});

Person.isPerson; // false - because it is static property created by `reopenClass`
Person.create().get('isPerson'); // true
```