## Создание записей

Вы можете создавать записи в хранилище с помощью вызова метода `createRecord`.

```js
store.createRecord('post', {
  title: 'Rails is Omakase',
  body: 'Lorem ipsum'
});
```

Объект хранилища доступен в контроллерах и маршрутах через `this.store`.

Хотя `createRecord` довольно простой, стоит обратить особое внимание на одну вещь: теперь вы не можете назначить обещание как связь.

Например, вы хотите установить свойство `author` для поста. Это **не** сработает, если `user` с ID еще не загружен в хранилище:

```js
var store = this.store;

store.createRecord('post', {
  title: 'Rails is Omakase',
  body: 'Lorem ipsum',
  author: store.findRecord('user', 1)
});
```

Но вы можете легко установить связь после завершения обещания:

```js
var store = this.store;

var post = store.createRecord('post', {
  title: 'Rails is Omakase',
  body: 'Lorem ipsum'
});

store.findRecord('user', 1).then(function(user) {
  post.set('author', user);
});
```

## Обновление записей

Чтобы вносить изменения в записи Ember Data, можно просто установить атрибут, который вы хотите изменить:

```js
this.store.findRecord('person', 1).then(function(tyrion) {
  // ...after the record has loaded
  tyrion.set('firstName', "Yollo");
});
```

Все удобства Ember.js доступны для изменения атрибутов. Например, вы можете использовать помощника `incrementProperty` в `Ember.Object`:

```js
person.incrementProperty('age'); // Happy birthday!
```

## Сохранение записей

Записи в Ember Data сохраняются на основе экземпляра. Вызовите `save()` на любом экземпляре `DS.Model`, и он сделает сетевой запрос.

Ember Data отслеживает для вас состояние каждой записи. Это позволяет Ember Data при сохранении обрабатывать только что созданные записи иначе, чем существующие записи.

По умолчанию, Ember Data будет отправлять `POST` запрос для новых записей на URL в зависимости от их типа.

```js
var post = store.createRecord('post', {
  title: 'Rails is Omakase',
  body: 'Lorem ipsum'
});

post.save(); // => POST to '/posts'
```

Записи, которые уже существуют в backend, обновляются с помощью команды HTTP `PATCH`.

```js
store.findRecord('post', 1).then(function(post) {
  post.get('title'); // => "Rails is Omakase"

  post.set('title', 'A new post');

  post.save(); // => PUT to '/posts/1'
});
```

Если запись имеет важные изменения, которые еще не сохранены, вы можете сказать об этом, проверив ее свойство `hasDirtyAttributes`. Вы также можете увидеть, были ли изменены части записи, и использовало ли исходное значение функцию `changedAttributes`. `changedAttributes` возвращает объект, чьи ключи — измененные свойства, а значение — массив значений `[oldValue, newValue]`.

```js
person.get('isAdmin');            //=> false
person.get('hasDirtyAttributes'); //=> false
person.set('isAdmin', true);
person.get('hasDirtyAttributes'); //=> true
person.changedAttributes();       //=> { isAdmin: [false, true] }
```

Сейчас вы можете сохранить изменения через `save()` или откатить их назад. Вызов `rollbackAttributes()` возвращает все `changedAttributes` к их исходному значению.

```js
person.get('hasDirtyAttributes'); //=> true
person.changedAttributes();       //=> { isAdmin: [false, true] }

person.rollbackAttributes();

person.get('hasDirtyAttributes'); //=> false
person.get('isAdmin');            //=> false
person.changedAttributes();       //=> {}
```

## Обещания

`save()` возвращает обещание, которое упрощает асинхронную обработку сценариев неудачи и успеха. Вот общий паттерн:

```js
var post = store.createRecord('post', {
  title: 'Rails is Omakase',
  body: 'Lorem ipsum'
});

var self = this;

function transitionToPost(post) {
  self.transitionToRoute('posts.show', post);
}

function failure(reason) {
  // handle the error
}

post.save().then(transitionToPost).catch(failure);

// => POST to '/posts'
// => transitioning to posts.show route
```

## Удаление записей

Удаление записей происходит так же просто, как их создание. Вызовите `deleteRecord()` на любом экземпляре `DS.Model`. Это отмечает запись как `isDeleted`. Удаление затем можно сохранить с помощью `save()`. В качестве альтернативы вы можете использовать метод `destroyRecord`, чтобы удалять и сохранять в одно и то же время.

```js
store.findRecord('post', 1).then(function(post) {
  post.deleteRecord();
  post.get('isDeleted'); // => true
  post.save(); // => DELETE to /posts/1
});

// OR
store.findRecord('post', 2).then(function(post) {
  post.destroyRecord(); // => DELETE to /posts/2
});
```

Удаленные записи будут показываться в RecordArrays, которые возвращены связями `store.peekAll` и `hasMany`, пока они успешно не сохранятся.