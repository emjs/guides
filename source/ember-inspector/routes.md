Вкладка Routes (маршруты) показывает список маршрутов вашего приложения.

Для следующего кода:

```js
this.route('posts', function() {
  this.route('new');
});
```

Inspector покажет эти маршруты:

<img src="/static/images/guides/ember-inspector/routes-screenshot.png" width="680"/>

Как вы видите, Inspector показывает маршруты, которые определили вы, и которые автоматически сгенерировал Ember.

### Обзор текущего маршрута

Inspector выделяет текущие активные маршруты. Но если приложение становится слишком громоздким, а эта особенность бесполезной, вы можете использовать поле `Current Route Only`, чтобы скрыть все маршруты кроме текущих активных.

<img src="/static/images/guides/ember-inspector/routes-current-route.png" width="680"/>