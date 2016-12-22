Роутер Ember позволяет обеспечить обратную связь при загрузке данных на маршруте и при возникновении ошибки во время загрузки.

## Подсостояния `loading`

Во время выполнения hooks `beforeModel`, `model`, `afterModel` данным может потребоваться время для загрузки. Технически роутер приостанавливает переход, пока не будут разрешены обещания, возвращенные из каждого hook.

Рассмотрим следующий пример:

`app/router.js`
```js
Router.map(function() {
  this.route('slow-model');
});
```

`app/routes/slow-model.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.get('store').findAll('slow-model');
  }
});
```

Если вы переходите на `slow-model`, в hook `model`, на выполнение запроса может уйти много времени.
При этом интерфейс пользователя не предоставляет вам какой-либо информации о происходящем. Если вы переходите на этот маршрут после полного обновления страницы, интерфейс пользователя будет абсолютно пустым, так как вы не выполнили полный переход на какой-либо маршрут и еще не отобразили шаблоны.
Если вы переходите на `slow-model` с другого маршрута, то будете видеть шаблоны из предыдущего маршрута, пока не загрузится модель, а затем резко отобразятся все шаблоны `slow-model`.

Как же визуально отобразить происходящее во время перехода?

Просто определите шаблон под названием `loading` (и опционально соответствующий маршрут), к которому Ember будет переходить. Промежуточный переход в подсостояние loading (загрузки) происходит сразу (синхронно); URL не обновляется, и, в отличие от других переходов, текущий активный переход не сбрасывается. 

Когда основной переход на `slow-model` выполнен, приложение покидает маршрут `loading`, и продолжается переход на `slow-model`.

Для вложенных маршрутов, это выглядит примерно так:

`app/router.js`
```js
Router.map(function() {
  this.route('foo', function() {
    this.route('bar', function() {
      this.route('slow-model');
    });
  });
});
```

При получении доступа к маршруту `foo.bar.slow-model` Ember будет поочередно искать шаблон `routeName-loading` (имя маршрута-loading) или `loading` в этой иерархии, начиная с `foo.bar.slow-model-loading`:

1. `foo.bar.slow-model-loading`
2. `foo.bar.loading` или `foo.bar-loading`
3. `foo.loading` или `foo-loading`
4. `loading` или `application-loading`
  
Важно отметить, что для самого маршрута `slow-model` Ember не будет искать шаблон `slow-model.loading`, но для остальной части иерархии такой синтаксис допустим. Это полезно для создания собственного загрузочного экрана для крайнего маршрута, например, `slow-model`.

При получении доступа к маршруту `foo.bar` Ember будет искать:

1. `foo.bar-loading`
2. `foo.loading` или `foo-loading`
3. `loading` или `application-loading`

Стоит заметить, что `foo.bar.loading` здесь не рассматривается.

### Событие `loading`

Если вы возвращаете обещание из hooks `beforeModel`/`model`/`afterModel`, и оно не разрешается сразу, на этом маршруте будет запущено событие [`loading`](http://emberjs.com/api/classes/Ember.Route.html#event_error).

`app/routes/foo-slow-model.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.get('store').findAll('slow-model');
  },
  actions: {
    loading(transition, originRoute) {
      let controller = this.controllerFor('foo');
      controller.set('currentlyLoading', true);
    }
  }
});
```

Если обработчик `loading` не определен на конкретном маршруте, событие продолжит распространяться выше родительского маршрута перехода, предоставляя маршруту `application` возможность обработать его.

При использовании обработчика `loading` мы можем применять обещание перехода, чтобы знать, когда завершится событие loading:

`app/routes/foo-slow-model.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  ...
  actions: {
    loading(transition, originRoute) {
      let controller = this.controllerFor('foo');
      controller.set('currentlyLoading', true);
      transition.promise.finally(function() {
          controller.set('currentlyLoading', false);
      });
    }
  }
});
```

## Подсостояния `error`

В случае возникновения ошибок во время перехода Ember предоставляет подход, который аналогичен подсостояниям `loading`.

Как и исходные обработчики события `loading`, обработчики `error` отобразят соответствующее подсостояние ошибки, если она будет обнаружена.

`app/router.js`
```js
Router.map(function() {
  this.route('articles', function() {
    this.route('overview');
  });
});
```

Как и в случае с подсостоянием `loading`, при возвращении ошибки или неразрешенного обещания из hook `model` маршрута `articles.overview` (или `beforeModel`/`afterModel`), Ember будет искать шаблон ошибки или маршрут в следующем порядке:

1. `articles.overview-error`
2. `articles.error` или `articles-error`
3. `error` или `application-error`

Если что-то из вышеперечисленного найдено, роутер сразу перейдет в это подсостояние (без обновления URL). «Причина» ошибки (то есть вызванное исключение или значение обещания с ошибкой) будет передана этому состоянию error в качестве его `model`.

Hook model (`beforeModel`,`model` и `afterModel`) подсостояния error не запускаются. Вызывается только метод `setupController` подсостояния error вместе с `error` в качестве модели. Смотрите пример ниже:

```js
setupController: function(controller, error) {
  Ember.Logger.debug(error.message);
  this._super(...arguments);
}
```

Если подходящих подсостояний error не найдено, сообщение об ошибке регистрируется в журнале.

### Событие `error`
 
Если hook `model` маршрута `articles.overview` возвращает обещание, которое завершено с ошибкой (например, сервер вернул ошибку, что пользователь не авторизовался, и т. д.), сработает событие [`error`](http://emberjs.com/api/classes/Ember.Route.html#event_error), которое распространится дальше. Это событие `error` можно обработать и использовать, чтобы отобразить сообщение об ошибке, перенаправить на страницу авторизации и т. д.

`app/routes/articles-overview.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  model(params) {
    return this.get('store').findAll('problematic-model');
  },
  actions: {
    error(error, transition) {
      if (error) {
        return this.transitionTo('error-page');
      }
    }
  }
});
```

По аналогии с событием `loading` вы можете управлять событием `error` на уровне приложения, чтобы не писать один и тот же код для нескольких маршрутов.