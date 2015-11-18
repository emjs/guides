Вызов `transitionTo` из маршрута или `transitionToRoute` из контроллера остановит любой текущий переход и начнет новый, который сработает как переадресация. `transitionTo` ведет себя точно так же, как помощник [link-to](http://emjs.ru/v2/templates/links).

Если новый маршрут содержит динамические сегменты, вам нужно передать либо модель, либо идентификатор для каждого сегмента. Передача модели проигнорирует hook `model` сегмента (так как модель уже загружена).

## Переход пока модель неизвестна

Если вы хотите выполнить переадресацию с одного маршрута на другой, то можете выполнить переход в hook `beforeModel` вашего обработчика маршрута.

`app/router.js`
```js
Router.map(function() {
  this.route('posts');
});
```

`app/routes/index.js`
```js
export default Ember.Route.extend({
  beforeModel() {
    this.transitionTo('posts');
  }
});
```

Если вам нужно проверить некоторое состояние приложения, чтобы выяснить, куда перенаправлять, то можете использовать [сервис](http://guides.emberjs.com/v2.1.0/applications/services/).

## Переход когда модель известна

Если вам нужна информация о текущей модели, чтобы принять решение о переадресации, вам следует использовать hook `afterModel`. Он получают разрешенную модель в качестве первого параметра и переход — в качестве второго. Например:

`app/router.js`
```js
Router.map(function() {
  this.route('posts');
  this.route('post', { path: '/post/:post_id' });
})
```

`app/routes/posts.js`
```js
export default Ember.Route.extend({
  afterModel(model, transition) {
    if (model.get('length') === 1) {
      this.transitionTo('post', model.get('firstObject'));
    }
  }
});
```

Если при переходе к маршруту `posts` оказывается, что есть только один пост, текущий переход будет прерван для переадресации к `PostRoute` с объектом единичного поста, который выступает в качестве его модели.

### Дочерние маршруты

Давайте изменим роутер выше, чтобы использовать вложенный маршрут:

`app/router.js`
```js
Router.map(function() {
  this.route('posts', function() {
    this.route('post', { path: ':post_id' });
  });
});
```

Если мы выполняем переадресацию на `posts.post` в hook `afterModel`, `afterModel` признает недействительной текущую попытку пройти по маршруту. Поэтому hooks `beforeModel`, `model`, и `afterModel` маршрута `posts` запустятся снова в пределах нового перенаправленного перехода. Это неэффективно, так как они запускаются только до переадресации.

Вместо этого можно использовать hook `redirect`, который оставит исходный переход действительным и не послужит причиной повторного запуска hooks родительского маршрута:

`app/routes/posts.js`
```js
export default Ember.Route.extend({
  redirect(model, transition) {
    if (model.get('length') === 1) {
      this.transitionTo('posts.post', model.get('firstObject'));
    }
  }
});
```