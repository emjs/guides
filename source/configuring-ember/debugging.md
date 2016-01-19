Ember предоставляет несколько опций настройки, которые помогут вам отлаживать проблемы в приложении.

## Маршрутизация

#### Журнал переходов роутера

`app/app.js`
```js
export default Ember.Application.extend({
  // Basic logging, e.g. "Transitioned into 'post'"
  LOG_TRANSITIONS: true,

  // Extremely detailed logging, highlighting every internal
  // step made while transitioning into a route, including
  // `beforeModel`, `model`, and `afterModel` hooks, and
  // information about redirects and aborted transitions
  LOG_TRANSITIONS_INTERNAL: true
});
```

## Представления/Шаблоны

#### Журнал подстановок представления

`config/environment.js`
```js
ENV.APP.LOG_VIEW_LOOKUPS = true;
```

```js
Ember.keys(Ember.TEMPLATES)
```

## Контроллеры

#### Журнал генерации контроллера

`config/environment.js `
```js
ENV.APP.LOG_ACTIVE_GENERATION = true;
```

## Наблюдатели/Привязки

#### Просмотри всех наблюдателей для объекта, ключа

```js
Ember.observersFor(comments, keyName);
```

#### Журнал привязок объекта

`config/environments.js`
```js
ENV.APP.LOG_BINDINGS = true
```

## Прочее

#### Включение ведения записей разрешений резолвера

Эта опция записывает все просмотры, которые делаются на консоли. Объекты, которые вы сами создали, имеют отметку, а объекты, сгенерированные Ember, нет.

Это полезно для понимания, какие объекты Ember находит во время просмотра, и какие генерируются для вас автоматически.

`app/app.js`
```js
export default Ember.Application.extend({
  LOG_RESOLVER: true
});
```

#### Решение проблем с устаревшими элементами

```js
Ember.ENV.RAISE_ON_DEPRECATION = true
Ember.ENV.LOG_STACKTRACE_ON_DEPRECATION = true
```

#### Реализация hook Ember.onerror для ведения записей обо всех ошибках на этапе производства

```js
Ember.onerror = function(error) {
  Ember.$.ajax('/error-notification', {
    type: 'POST',
    data: {
      stack: error.stack,
      otherInformation: 'exception message'
    }
  });
}
```

#### Импорт консоли

Если вы используете в Ember импорты, то убедитесь, что импортировали консоль:

```js
Ember = {
  imports: {
    Handlebars: Handlebars,
    jQuery: $,
    console: window.console
  }
};
```

#### Ошибки в RSVP.Promise

При работе с обещаниями бывают случаи, когда кажется, что любые ошибки «проглатываются» и должным образом не отмечаются. Поэтому довольно сложно отслеживать, откуда исходит конкретная проблема. К счастью, `RSVP` имеет встроенное решение.

Вы можете реализовать функцию `onerror`, которая будет вызываться с деталями ошибки, если таковая возникнет в обещании. Эта функция отличается от обычной практики вызова `console.assert` для вывода ошибки на консоль.

`app/app.js`
```js
Ember.RSVP.on('error', function(error) {
  Ember.Logger.assert(false, error);
});
```

#### Ошибки в Еmber.run.later ([BACKBURNER.JS](https://github.com/ebryn/backburner.js))

Backburner поддерживает объединение отслеживаемых стеков, так что вы можете понять, откуда идут ошибки с `Ember.run.later`. К сожалению, это довольно медленный процесс, и он не подходит для этапа производства или даже нормальной разработки.

Чтобы активировать режим, вам нужно установить:

```js
Ember.run.backburner.DEBUG = true;
```