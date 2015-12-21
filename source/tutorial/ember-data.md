Сейчас, чтобы установить модель, наше приложение использует строго кодированные данные для *rentals* в обработчике маршрута `index`. По мере увеличения размеров приложения нам понадобится возможность создавать новые пункты с недвижимостью, обновлять их, удалять и сохранять эти изменения на сервере базы данных. Для решения этой проблемы мы используем библиотеку управления данными под названием Ember Data.

Давайте сгенерируем нашу первую модель Ember Data с именем `rental`:

```shell
ember g model rental
```

Такой результат мы получим при создании файла модели и тестового файла:

```shell
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

Давайте добавим для rental те же атрибуты, что мы использовали в строго закодированном массиве объектов JavaScript, а именно *owner*, *city*, *type*, *image*, *bedrooms*:

`app/models/rental.js`
```js
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

## Использование Mirage с Ember Data

Ember Data можно настроить так, чтобы сохранять данные в самых разных местах. Чаще всего библиотека запрограммирована на работу с API сервера базы данных. Здесь мы используем [Mirage](http://www.ember-cli-mirage.com/). Это позволит нам создать ложные данные для работы, параллельно разрабатывать приложение и имитировать работу сервера базы данных.

Давайте начнем с установки Mirage:

```shell
ember install ember-cli-mirage
```

Настроим Mirage, чтобы возвращать rentals, которые мы определили выше. Для этого обновим `app/mirage/config.js`:

`app/mirage/config.js`
```js
export default function() {
  this.get('/rentals', function() {
    return {
      data: [{
        type: 'rentals',
        id: 1,
        attributes: {
          title: 'Grand Old Mansion',
          owner: 'Veruca Salt',
          city: 'San Francisco',
          type: 'Estate',
          bedrooms: 15,
          image: 'https://upload.wikimedia.org/wikipedia/commons/c/cb/Crane_estate_(5).jpg'
        }
      }, {
        type: 'rentals',
        id: 2,
        attributes: {
          title: 'Urban Living',
          owner: 'Mike Teavee',
          city: 'Seattle',
          type: 'Condo',
          bedrooms: 1,
          image: 'https://upload.wikimedia.org/wikipedia/commons/0/0e/Alfonso_13_Highrise_Tegucigalpa.jpg'
        }
      }, {
        type: 'rentals',
        id: 3,
        attributes: {
          title: 'Downtown Charm',
          owner: 'Violet Beauregarde',
          city: 'Portland',
          type: 'Apartment',
          bedrooms: 3,
          image: 'https://upload.wikimedia.org/wikipedia/commons/f/f7/Wheeldon_Apartment_Building_-_Portland_Oregon.jpg'
        }
      }]
    }
  });
}
```

Теперь, когда бы Ember Data ни послала запрос GET к `/rentals`, Mirage будет возвращать этот объект JavaScript как JSON.

### Обновление Hook Model

Чтобы использовать новое хранилище данных, нам нужно обновить hook `model` в обработчике маршрута.

`app/routes/index.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.store.findAll('rental');
  },
});
```

Когда мы вызываем `this.store.findAll('rental')`, Ember Data делает запрос GET к `/rentals`. Так как мы используем Mirage в среде разработки, Mirage вернет данные, которые мы предоставили. Когда мы развернем наше приложение на рабочем сервере, то нам понадобится предоставить backend для Ember Data, чтобы поддерживать связь.