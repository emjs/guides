У Ember есть богатый набор дополнений, которые можно легко добавлять в проекты.
Дополнения предоставляют широкий спектр функций, часто экономят время и позволяют сосредоточиться на самом проекте.

Чтобы посмотреть дополнения, посетите сайт [Ember Observer](https://emberobserver.com/).
На этом сайте дополнения, которые были опубликованы в NPM, рассортированы по каталогам и категориям. Также им присвоены оценки на основе ряда критериев.

Для Super Rentals мы используем два дополнения: [ember-cli-tutorial-style](https://github.com/toddjordan/ember-cli-tutorial-style) и [ember-cli-mirage](http://www.ember-cli-mirage.com/).

## ember-cli-tutorial-style

Вместо того чтобы копировать/вставлять код в CSS для стилизации Super Rentals, мы создали дополнение [ember-cli-tutorial-style](https://github.com/ember-learn/ember-cli-tutorial-style), которое сразу добавляет CSS в руководство. Дополнение работает следующим образом: оно создает файл `ember-tutorial.css` и вкладывает его в директорию `vendor` super-rentals. При запуске Ember CLI берет файл CSS `ember-tutorial` и вкладывает его в `vendor.css` (который ссылается на `/app/index.html`). Мы можем вносить дополнительные корректировки в стиль в `/vendor/ember-tutorial.css`. Изменения вступят в силу после перезапуска приложения.

Запустите следующую команду, чтобы установить дополнение:

```
ember install ember-cli-tutorial-style
```

Так как дополнения — npm-пакеты, команда `ember install` устанавливает их в директорию `node_modules` и создает запись в `package.json`. Не забудьте перезапустить сервер после установки дополнения.
После перезапуска сервера новый CSS будет включен в проект. Обновите окно браузера и увидите следующую картину:

![styled super rentals basic](https://guides.emberjs.com/v2.7.0/images/installing-addons/styled-super-rentals-basic.png)

## ember-cli-mirage

[Mirage](http://www.ember-cli-mirage.com/) — библиотека HTTP-заглушек, которая часто используется в Ember для приемочного тестирования со стороны клиента. В нашем руководстве мы будем использовать mirage как источник данных. Mirage позволит нам создавать фиктивные данные, которые мы будем использовать во время разработки приложения, и имитировать работу внутреннего сервера.

Установите дополнение Mirage следующим образом:

```
ember install ember-cli-mirage
```

Если вы запускали `ember serve` в другой оболочке, перезапустите сервер, чтобы добавить Mirage в сборку.

Теперь настроим Mirage, чтобы возвращать объекты rental, которые мы определили выше. Для этого обновляем `/mirage/config.js`:

`mirage/config.js`
```js
export default function() {
  this.namespace = '/api';

  this.get('/rentals', function() {
    return {
      data: [{
        type: 'rentals',
        id: 'grand-old-mansion',
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
        id: 'urban-living',
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
        id: 'downtown-charm',
        attributes: {
          title: 'Downtown Charm',
          owner: 'Violet Beauregarde',
          city: 'Portland',
          type: 'Apartment',
          bedrooms: 3,
          image: 'https://upload.wikimedia.org/wikipedia/commons/f/f7/Wheeldon_Apartment_Building_-_Portland_Oregon.jpg'
        }
      }]
    };
  });
}
```

Такая настройка Mirage предполагает, что когда Ember Data будет делать запрос GET к `/api/rentals`, Mirage вернет этот объект JavaScript как JSON. Чтобы это работало, наше приложение, по умолчанию, должно делать запросы в пространстве имен `/api`. Без этого изменения переход в приложении в `/rentals` будет конфликтовать с Mirage.

Чтобы внести изменение, нам нужно сгенерировать адаптер приложения.

```
ember generate adapter application
```

Этот адаптер расширит базовый класс [`JSONAPIAdapter`](http://emberjs.com/api/data/classes/DS.JSONAPIAdapter.html) из Ember Data:

`app/adapters/application.js`
```js
import DS from 'ember-data';

export default DS.JSONAPIAdapter.extend({
  namespace: 'api'
});
```