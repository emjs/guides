Во время перехода на маршрут роутер Ember передает объект перехода различным hooks на маршрутах, которые участвуют в переходе. Любой hook, который имеет доступ к этому объекту перехода, может сразу сбросить переход с помощью вызова `transition.abort()`. Если объект перехода сохранен, можно повторить попытку позднее при помощи вызова `transition.retry()`.

### Предотвращение переходов через `willTranslation`

Когда происходит попытка перехода через `{{link-to}}`, `transitionTo` или смену URL, на текущих активных маршрутах запускается действие `willTransition`. Это дает каждому активному маршруту, начиная с конечного, возможность решить, произойдет ли переход или нет.

Представьте, что ваше приложение находится на маршруте, который показывает пользователю сложную форму. Пользователь ее заполняет, но случайно переходит обратно. Если не прервать переход, пользователь может потерять все заполненные в форме данные, что довольно неприятно.

Ниже приведен вариант, как можно обработать эту ситуацию:

`app/routes/form.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  actions: {
    willTransition(transition) {
      if (this.controller.get('userHasEnteredData') &&
          !confirm('Are you sure you want to abandon progress?')) {
        transition.abort();
      } else {
        // Bubble the `willTransition` action so that
        // parent routes can decide whether or not to abort.
        return true;
      }
    }
  }
});
```

Когда пользователь нажимает хелпер `{{link-to}}`, или когда приложение инициирует переход с помощью `transitionTo`, переход будет сброшен, и URL не изменится. Но если в браузере используется кнопка возврата, чтобы выйти с `route:form`, или если пользователь вручную меняет URL, приложение переместит пользователя на новый URL, прежде чем будет вызвано действие `willTransition`. В результате браузер отобразит новый URL, даже если `willTransition` вызовет `transition.abort()`.

### Сброс переходов в `model`, `beforeModel`, `afterModel`

Hooks `model`, `beforeModel` и `afterModel`, описанные в разделе [Асинхронная маршрутизация](http://emjs.ru/v2/routing/asynchronous-routing/), вызываются объектом перехода. Это позволяет маршрутам назначения сбрасывать неудавшиеся переходы.

`app/routes/disco.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  beforeModel(transition) {
    if (new Date() > new Date('January 1, 1980')) {
      alert('Sorry, you need a time machine to enter this route.');
      transition.abort();
    }
  }
});
```

### Сохранение и повторная попытка перехода

Сброшенные переходы можно повторно осуществить позднее. Это часто используют в маршрутах аутентификации, которые перенаправляют пользователя на страницу авторизации, а после авторизации отправляют его обратно.

`app/routes/some-authenticated.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  beforeModel(transition) {
    if (!this.controllerFor('auth').get('userIsLoggedIn')) {
      var loginController = this.controllerFor('login');
      loginController.set('previousTransition', transition);
      this.transitionTo('login');
    }
  }
});
```

`app/controllers/login.js`
```js
import Ember from 'ember';

export default Ember.Controller.extend({
  actions: {
    login() {
      // Log the user in, then reattempt previous transition if it exists.
      var previousTransition = this.get('previousTransition');
      if (previousTransition) {
        this.set('previousTransition', null);
        previousTransition.retry();
      } else {
        // Default back to homepage
        this.transitionToRoute('index');
      }
    }
  }
});
```