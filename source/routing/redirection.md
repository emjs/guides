Вызов [`transitionTo()`](http://emberjs.com/api/classes/Ember.Route.html#method_transitionTo) из маршрута или [`transitionToRoute()`](http://emberjs.com/api/classes/Ember.Controller.html#method_transitionToRoute) из контроллера останавливает любой текущий переход и начинает новый, то есть действует как перенаправление. `transitionTo()` ведет себя точно так же, как хелпер [link-to](http://emjs.ru/v2/templates/links).

Если новый маршрут содержит динамические сегменты, вам нужно передать либо *модель*, либо *идентификатор* для каждого сегмента. Передача модели проигнорирует hook `model()` сегмента (так как модель уже загружена).

## Переход пока модель неизвестна

Если вы хотите выполнить перенаправление с одного маршрута на другой, то можете выполнить переход в hook [`beforeModel()`](http://emberjs.com/api/classes/Ember.Route.html#method_beforeModel) вашего обработчика маршрута.

`app/router.js`
```js
Router.map(function() {
  this.route('posts');
});
```

`app/routes/index.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  beforeModel() {
    this.transitionTo('posts');
  }
});
```

Если вам нужно проверить какое-либо состояние приложения, чтобы понять, куда перенаправлять, можете использовать [службу](http://emjs.ru/v2/applications/services/).

## Переход когда модель известна

Если вам нужна информация о текущей модели, чтобы принять решение о перенаправлении, вам следует использовать hook [`afterModel()`](http://emberjs.com/api/classes/Ember.Route.html#method_afterModel). Он получают разрешенную модель в качестве первого параметра и переход — в качестве второго. Например:

`app/router.js`
```js
Router.map(function() {
  this.route('posts');
  this.route('post', { path: '/post/:post_id' });
})
```

`app/routes/posts.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  afterModel(model, transition) {
    if (model.get('length') === 1) {
      this.transitionTo('post', model.get('firstObject'));
    }
  }
});
```

Если при переходе на маршрут `posts` окажется, что публикация всего одна, текущий переход прервется, и приложение перенаправит на `PostRoute` с объектом единственной публикации, который выступает в качестве его модели.

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

Если мы перенаправляем на `posts.post` в hook `afterModel`, `afterModel` признает недействительной текущую попытку пройти по маршруту. Поэтому hook  `beforeModel`, `model` и `afterModel` маршрута `posts` запустятся снова в рамках нового перенаправленного перехода.
Это неэффективно, так как они запускаются до перенаправления.

Вместо этого можно использовать метод [`redirect()`](http://emberjs.com/api/classes/Ember.Route.html#method_redirect), который признает исходный переход действительным и не станет причиной повторного запуска hooks родительского маршрута:

`app/routes/posts.js` 
```js
import Ember from 'ember';

export default Ember.Route.extend({
  redirect(model, transition) {
    if (model.get('length') === 1) {
      this.transitionTo('posts.post', model.get('firstObject'));
    }
  }
});
```