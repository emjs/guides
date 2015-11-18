Хранилище можно воспринимать как кэш всех записей, которые загрузило приложение. Когда маршрут или контроллер в приложении запрашивает запись, хранилище может вернуть ее сразу, если она находится в кэше. В противном случае хранилище должно обратиться к адаптеру, чтобы он загрузил запись. Обычно это отправляет запрос по сети для извлечения записи с сервера.

Вместо того чтобы ожидать, пока приложение запросит запись, вы можете заранее поместить записи в кэш хранилища.

Это полезно, если вы хорошо знаете, какие записи понадобятся пользователю. Когда он щелкает по ссылке, вместо ожидания, пока завершится сетевой запрос, Ember.js может немедленно отобразить новый шаблон. Кажется, что все происходит мгновенно.

Еще это можно использовать, когда приложение имеет потоковое соединение с backend. Тогда, если запись создают или изменяют, интерфейс пользователя сразу обновляется.

### Помещение записей

Чтобы поместить запись в хранилище, вызовите метод хранилища `push()`.

Например, представьте, что вам понадобилось предварительно загрузить некоторые данные в хранилище, когда приложение запускается в первый раз.

Мы можем использовать `route:application`, чтобы сделать это. `route:application` — верхний маршрут в иерархии, и его hook `model` запускается при начальной загрузке приложения.

`app/models/album.js`
```js
export default DS.Model.extend({
  title: DS.attr(),
  artist: DS.attr(),
  songCount: DS.attr()
});
```

`app/routes/application.js`
```js
export default Ember.Route.extend({
  model() {
    this.store.push({
      data: [{
        id: 1,
        type: 'album',
        attributes: {
          title: 'Fewer Moving Parts',
          artist: 'David Bazan',
          songCount: 10
        },
        relationships: {}
      }, {
        id: 2,
        type: 'album',
        attributes: {
          title: 'Calgary b/w I Can\'t Make You Love Me/Nick Of Time',
          artist: 'Bon Iver',
          songCount: 2
        },
        relationships: {}
      }]
    });
  }
});
```

Метод хранилища `push()` — API низкого уровня. Он принимает документ JSON API, который имеет несколько важных отличий от документа JSON API, что принимает JSONAPISerializer. Имя типа в документе JSON API должно в точности соответствовать имени типа модели (в примере выше тип — `album`, так как модель определена в `app/models/album.js`). Имена атрибутов и связей должны соответствовать кешу свойств, которые определены в классе модели.

Если вы хотите, чтобы сериализатор нормировал данные прежде чем поместить их в хранилище, то можете использовать метод `store.pushPayload`.

`app/routes/application.js`
```js
export default Ember.Route.extend({
  model() {
    this.store.pushPayload({
      data: [{
        id: 1,
        type: 'albums',
        attributes: {
          title: 'Fewer Moving Parts',
          artist: 'David Bazan',
          song-count: 10
        },
        relationships: {}
      }, {
        id: 2,
        type: 'albums',
        attributes: {
          title: 'Calgary b/w I Can\'t Make You Love Me/Nick Of Time',
          artist: 'Bon Iver',
          song-count: 2
        },
        relationships: {}
      }]
    });
  }
});
```

Метод `push()` также важен при работе со сложными конечными точками. Вы можете обнаружить, что приложение имеет конечную точку, которая реализует некоторую бизнес-логику при создании нескольких записей. Это не соответствует в точности существующему API `save()` в Ember Data, который организован для сохранения одной записи. Вместо этого вам следует делать собственный запрос AJAX и помещать полученные данные модели в хранилище. Так они будут доступны для других частей приложения.

`app/routes/confirm-payment.js`
```js
export default Ember.Route.extend({
  actions: {
    confirm: function(data) {
      $.ajax({
        data: data,
        method: 'POST',
        url: 'process-payment'
      }).then((digitalInventory) => {
        this.store.pushPayload(digitalInventory);
        this.transitionTo('thank-you');
      });
    }
  }
});
```