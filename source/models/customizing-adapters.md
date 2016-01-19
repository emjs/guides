В Ember Data Адаптер определяет, как сохраняются данные в хранилище backend, вроде формата URL и заголовков для REST API. (Формат данных определяет [сериализатор](http://guides.emberjs.com/v2.1.0/models/customizing-serializers/)). По умолчанию, Адаптер в Ember Data имеет некоторые встроенные положения относительно того, как [должен выглядеть REST API](http://jsonapi.org/). Если соглашения backend отличаются от этих положений, Ember Data позволяет легко изменить свою функциональность с помощью выгрузки или расширения исходного адаптера.

Причины для индивидуальной настройки адаптера включают: использование `underscores_case` в URL, использование среды (помимо REST) для установления связи с API backend или даже использование [локального backend](https://github.com/locks/ember-localstorage-adapter).

Расширение адаптеров — естественный процесс в Ember Data. Ember занимает следующую позицию: вместо добавления флага вам следует расширять адаптер, чтобы добавить другую функциональность. В результате код легче тестировать, он становится проще для понимания и уменьшает волокиту для людей, которые захотят создать подкласс вашего адаптера.

Если backend имеет некоторые устойчивые правила, вы можете определить `adapter:application`. `adapter:application` получит приоритет перед исходным адаптером, но все еще будет замещаться адаптерами со специфической моделью.

`app/adapters/application.js`
```js
export default DS.JSONAPIAdapter.extend({
  // Application specific overrides go here
});
```

Если у вас есть одна модель, которая в отличие от других имеет особые правила для установления связи с backend, вы можете создать адаптер со специфической моделью с помощью команды `ember generate adapter adapter-name`. Например, запуск `ember generate adapter post` создаст следующий файл:

`app/adapters/post.js`
```js
export default DS.JSONAPIAdapter.extend({
  namespace: 'api/v1'
});
```

По умолчанию, Ember Data имеет несколько встроенных адаптеров. Свободно используйте эти адаптеры в качестве отправной точки для создания собственного адаптера.

* [DS.Adapter](http://emberjs.com/api/data/classes/DS.Adapter.html) — базовый адаптер, который не имеет функциональности. Это хороший исходный вариант, если вы хотите создать адаптер, который радикально отличается от других адаптеров Ember.
* [DS.JSONAPIAdapter](http://emberjs.com/api/data/classes/DS.JSONAPIAdapter.html). `JSONAdapter` — исходный адаптер, который следует соглашениям JSON API по установлению связи с сервером HTTP при передаче JSON через XHR.
* [DS.RESTAdapter](http://emberjs.com/api/data/classes/DS.RESTAdapter.html). `RESTAdapter` позволяет хранилищу устанавливать связь с сервером HTTP при передаче JSON через XHR. До выхода Ember Data 2.0 этот адаптер использовался по умолчанию.

## Настройка JSONAPIADAPTER

[DS.JSONAPIAdapter](http://emberjs.com/api/data/classes/DS.JSONAdapter.html) имеет небольшое количество hooks, которые обычно используются, чтобы расширить его для работы с нестандартными backends.

### Соглашения URL

`JSONAPIAdapter` достаточно разумный, чтобы на основе имени модели определить URLs, с которыми он устанавливает связь. Например, если вы запросите `Post` по ID:

```js
store.find('post', 1).then(function(post) {
});
```

Адаптер JSON API автоматически отправит запрос `GET` в `/posts/1`.

В адаптере JSON API вы можете применить следующие действия к записи, которые соответствуют URLs:

<table>
  <thead>
    <tr><th>Действие</th><th>Команда HTTP</th><th>URL</th></tr>
  </thead>
  <tbody>
    <tr><th>Найти</th><td>GET</td><td>/posts/123</td></tr>
    <tr><th>Найти все</th><td>GET</td><td>/posts</td></tr>
    <tr><th>Обновить</th><td>PATCH</td><td>/posts/123</td></tr>
    <tr><th>Создать</th><td>POST</td><td>/posts</td></tr>
    <tr><th>Удалить</th><td>DELETE</td><td>/posts/123</td></tr>
  </tbody>
</table>

#### Настройка образования множественного числа

Чтобы способствовать образованию множественного числа имен модели при генерации маршрута URLs, Ember Data объединяется с [Ember Inflector](https://github.com/stefanpenner/ember-inflector). Это совместимая библиотека ActiveSupport::Inflector, которая предназначена для преобразования слов во множественную и единственную формы. Неправильные или неисчисляемые формы множественного числа можно указать через `Ember.Inflector.inflector`:

```js
var inflector = Ember.Inflector.inflector;

inflector.irregular('formula', 'formulae');
inflector.uncountable('advice');
```

Так вы сообщите адаптеру JSON API, что запросы для `formula` должны следовать в `/formulae/1` вместо `/formulas/1`.

#### Настройка пути к конечной точке

Свойство `namespace` можно использовать, чтобы ставить в начале запросов специфический URL из пространства имен.

`app/adapters/application.js`
```js
export default DS.JSONAPIAdapter.extend({
  namespace: 'api/1'
});
```

Запросы для `person` теперь будут направляться на `http://emberjs.com/api/1/people/1`.

#### Настройка хоста

По умолчанию, адаптер будет нацелен на текущий домен. Если вы хотите указать новый домен, то для этого можете установить на адаптер свойство `host`:

`app/adapters/application.js`
```js
export default DS.JSONAPIAdapter.extend({
  host: 'https://api.example.com'
});
```

Запросы для `person` теперь будут направляться на `https://api.example.com/people/1`.

#### Настройка пути

По умолчанию, `JSONAPIAdapter` пытается образовать множественное число и установить тире в имени модели, чтобы сгенерировать имя пути. Если это соглашение не соответствует backend, вы можете переопределить метод `pathForType`.

Например, если вы не хотели образовывать множественное число имен модели, и вам нужно использовать underscore_case вместо camelCase, то можно переопределить метод `pathForType` таким образом:

`app/adapters/application.js`
```js
export default DS.JSONAPIAdapter.extend({
  pathForType: function(type) {
    return Ember.String.underscore(type);
  }
});
```

Запросы для `person` теперь будут направлены на `/person/1`. Запросы для `user-profile` — на `/user_profile/1`.

#### Настройка заголовков

Некоторые APIs требуют заголовков HTTP, например, чтобы предоставить ключ API. На объекте `headers` в `JSONAPIAdapter` можно установить случайные заголовки в качестве пар ключ/значение. Ember Data будет отправлять их вместе с каждым запросом ajax.

`app/adapters/application.js`
```js
export default DS.JSONAPIAdapter.extend({
  headers: {
    'API_KEY': 'secret key',
    'ANOTHER_HEADER': 'Some header value'
  }
});
```

`headers` также можно использовать как вычислительное свойство, чтобы поддерживать динамические заголовки. В примере ниже контейнер Ember вводит в адаптер объект `session`.

`app/adapters/application.js`
```js
export default DS.JSONAPIAdapter.extend({
  session: Ember.inject.service('session'),
  headers: Ember.computed('session.authToken', function() {
    return {
      'API_KEY': this.get('session.authToken'),
      'ANOTHER_HEADER': 'Some header value'
    };
  })
});
```

В некоторых случаях для динамических заголовков требуются данные из объектов вне системы наблюдения Ember (например, `document.cookie`). Вы можете использовать функцию [volatile](http://guides.emberjs.com/api/classes/Ember.ComputedProperty.html#method_volatile), чтобы установить свойство в режим без кэширования. В результате заголовки будут вычисляться по-новому с каждым запросом.

`app/adapters/application.js`
```js
export default DS.JSONAPIAdapter.extend({
  headers: Ember.computed(function() {
    return {
      'API_KEY': Ember.get(document.cookie.match(/apiKey\=([^;]*)/), '1'),
      'ANOTHER_HEADER': 'Some header value'
    };
  }).volatile()
});
```

#### Авторские адаптеры

Свойство `defaultSerializer` можно применить, чтобы указать сериализатору, что будет использовать этот адаптер. Оно применяется только в тех случаях, если сериализатор со специфической моделью или `serializer:application` не определен.

В приложении зачастую проще указать `serializer:application`. Но если вы разработчик адаптера для сообщества, то важно не забыть установить это свойство. Это гарантирует, что Ember сделает все правильно, если пользователь вашего адаптера не укажет `serializer:application`.

`app/adapters/my-custom-adapter.js`
```js
export default DS.JSONAPIAdapter.extend({
  defaultSerializer: '-default'
});
```

## Адаптеры сообщества

Если ни один из встроенных адаптеров Ember Data не работает с вашим backend, то проверьте некоторые из адаптеров, которые поддерживает сообщество Ember. Адаптеры можно найти в этих местах:

* [Ember Observer](http://emberobserver.com/categories/data)
* [GitHub](https://github.com/search?q=ember+data+adapter&ref=cmdform)
* [Bower](http://bower.io/search/?q=ember-data-)