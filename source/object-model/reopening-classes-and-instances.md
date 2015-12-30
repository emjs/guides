Вам не нужно сразу определять все классы. Вы можете повторно открыть класс и установить новые свойства с помощью метода `reopen()`.

```javascript
Person.reopen({
  isPerson: true
});

Person.create().get('isPerson') // true
```

При использовании `reopen()` вы можете переопределить существующие методы и вызвать `this._super`.

```javascript
Person.reopen({
  // override `say` to add an ! at the end
  say(thing) {
    this._super(thing + '!');
  }
});
```

Метод `reopen()` применяют, чтобы добавить методы и свойства экземпляра, которые будут использоваться для всех экземпляров класса. То есть вы не добавляете методы и свойства конкретному экземпляру класса, как в классическом JavaScript (без использования прототипа).

Но если вам нужно добавить статичные методы или свойства только классу, то можете использовать `reopenClass()`.

```javascript
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