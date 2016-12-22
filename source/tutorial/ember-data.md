Сейчас, чтобы установить модель, приложение использует в обработчике маршрута `rentals` строго закодированные данные для объектов *rentals*. По мере расширения приложения нам понадобится возможность создавать новые позиции с недвижимостью, обновлять их, удалять и сохранять эти изменения на сервере. Для решения этой проблемы мы используем библиотеку управления данными Ember Data.

Сгенерируем первую модель Ember Data с именем `rental`:

```
ember g model rental
```

Такой результат мы получим при создании файла модели и тестового файла:

```
installing model
  create app/models/rental.js
installing model-test
  create tests/unit/models/rental-test.js
```

Открыв файл модели, мы увидим следующее:

`app/models/rental.js`
```js
import DS from 'ember-data';

export default DS.Model.extend({

});
```

Добавим для rental те же атрибуты, что мы использовали в строго закодированном массиве объектов JavaScript, а именно *title*, *owner*, *city*, *type*, *image*, *bedrooms*:

`app/models/rental.js`
```
import DS from 'ember-data';

export default DS.Model.extend({
  title: DS.attr(),
  owner: DS.attr(),
  city: DS.attr(),
  type: DS.attr(),
  image: DS.attr(),
  bedrooms: DS.attr()
});
```

Теперь у нас есть модель в хранилище Ember Data.

## Обновление hook model

Чтобы использовать новое хранилище данных, нам нужно обновить hook `model` в обработчике маршрута.

`app/routes/rentals.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.get('store').findAll('rental');
  }
});
```

Когда мы вызываем `this.get('store').findAll('rental')`, Ember Data делает GET запрос к `/rentals`.
Подробнее о библиотеке Ember Data можно узнать в разделе [«Модели»](http://emjs.ru/v2/models/).

Так как мы используем Mirage в среде разработки, она вернет данные, которые мы предоставили.
Когда мы развернем приложение на рабочем сервере, нам понадобится предоставить для Ember Data backend, с которым мы будем взаимодействовать.