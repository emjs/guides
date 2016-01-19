*Модульное тестирование методов и вычислительных свойств происходит согласно предыдущим паттернам, которые описаны в [Основах модульного тестирования](http://emjs.ru/v2/testing/unit-testing-basics), так как Ember.Controller служит расширением для Ember.Object.*

Модульное тестирование контроллеров осуществляется довольно просто с использованием помощника, который является частью библиотеки ember-qunit.

### Тестирование действий контроллера

У нас есть контроллер `PostsController` с некоторыми вычислительными свойствами и действием `setProps`.

Вы можете сгенерировать собственный контроллер с помощью `ember generate controller posts`.

`app/controllers/posts.js`
```js
export default Ember.Controller.extend({
  propA: 'You need to write tests',
  propB: 'And write one for me too',

  setPropB(str) {
    this.set('propB', str);
  },

  actions: {
    setProps(str) {
      this.set('propA', 'Testing is cool');
      this.setPropB(str);
    }
  }
});
```

`setProps` устанавливает на контроллер свойство и вызывает метод. В нашем сгенерированном тесте ember-cli уже использует помощника `moduleFor`, чтобы установить контейнер теста:

`tests/unit/controllers/posts-test.js`
```js
moduleFor('controller:posts', {
});
```

Затем мы используем `this.subject()`, чтобы получить экземпляр `PostsController` и написать тест для проверки действия. `this.subject()` — метод помощника из библиотеки `ember-qunit`, который возвращает синглтон-экземпляр модуля, установленного с помощью `moduleFor`.

`tests/unit/controllers/posts-test.js`
```js
test('should update A and B on setProps action', function(assert) {
  assert.expect(4);

  // get the controller instance
  const ctrl = this.subject();

  // check the properties before the action is triggered
  assert.equal(ctrl.get('propA'), 'You need to write tests', 'propA initialized');
  assert.equal(ctrl.get('propB'), 'And write one for me too', 'propB initialized');

  // trigger the action on the controller by using the `send` method,
  // passing in any params that our action may be expecting
  ctrl.send('setProps', 'Testing Rocks!');

  // finally we assert that our values have been updated
  // by triggering our action.
  assert.equal(ctrl.get('propA'), 'Testing is cool', 'propA updated');
  assert.equal(ctrl.get('propB'), 'Testing Rocks!', 'propB updated');
});
```

### Тестирование зависимостей контроллера

Иногда контроллеры имеют зависимости от других контроллеров. Это достигается добавлением одного контроллера внутрь другого. Например, здесь у нас два простых контроллера. `CommentsController` использует `PostController` через `inject`.

Вы можете сгенерировать собственные контроллеры с помощью `ember generate controller post` и `ember generate controller comments`.

`app/controllers/post.js`
```js
export default Ember.Controller.extend({
  title: Ember.computed.alias('model.title')
});
```

`app/controllers/comments.js`
```js
export default Ember.Controller.extend({
  post: Ember.inject.controller(),
  title: Ember.computed.alias('post.title')
});
```

В этот раз при установке `moduleFor` нам нужно передать объект параметров в качестве третьего аргумента, который имеют `needs` контроллера.

`tests/unit/controllers/comments-test.js`
```js
moduleFor('controller:comments', 'Comments Controller', {
  needs: ['controller:post']
});
```

Теперь давайте напишем тест. Он устанавливает свойство на модель `post` в `PostController`, который будет доступен в `CommentsController`.

`tests/unit/controllers/comments-test.js`
```js
test('should modify the post model', function(assert) {
  assert.expect(2);

  // grab an instance of `CommentsController` and `PostController`
  const ctrl = this.subject();
  const postCtrl = ctrl.get('post');

  // wrap the test in the run loop because we are dealing with async functions
  Ember.run(function() {

    // set a generic model on the post controller
    postCtrl.set('model', Ember.Object.create({ title: 'foo' }));

    // check the values before we modify the post
    assert.equal(ctrl.get('title'), 'foo', 'title is set');

    // modify the title of the post
    postCtrl.get('model').set('title', 'bar');

    // assert that the controllers title has changed
    assert.equal(ctrl.get('title'), 'bar', 'title is updated');
  });
});
```