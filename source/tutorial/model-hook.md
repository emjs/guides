А сейчас добавим список доступной недвижимости в шаблон index. Мы знаем, что позиции в списке не будут статичными, так как у пользователей будет возможность их добавлять, обновлять и удалять. Поэтому нам нужно, чтобы модель *rentals* сохраняла информацию о недвижимости.
Чтобы не усложнять обучение, на первых порах мы используем строго закодированный массив объектов JavaScript. Позднее мы будем использовать библиотеку Ember Data, для надежного управления данными в приложении.

Вот так будет выглядеть наша главная страница, когда мы все сделаем:

![super rentals index with list](https://guides.emberjs.com/v2.7.0/images/models/super-rentals-index-with-list.png)

В Ember обработчики маршрута отвечают за загрузку данных модели. Откроем `app/routes/rentals.js` и добавим наши строго закодированные данные в качестве возвращаемого значения hook `model`:

`app/routes/rentals.js`
```js
import Ember from 'ember';

let rentals = [{
  id: 1,
  title: 'Grand Old Mansion',
  owner: 'Veruca Salt',
  city: 'San Francisco',
  type: 'Estate',
  bedrooms: 15,
  image: 'https://upload.wikimedia.org/wikipedia/commons/c/cb/Crane_estate_(5).jpg'
}, {
  id: 2,
  title: 'Urban Living',
  owner: 'Mike TV',
  city: 'Seattle',
  type: 'Condo',
  bedrooms: 1,
  image: 'https://upload.wikimedia.org/wikipedia/commons/0/0e/Alfonso_13_Highrise_Tegucigalpa.jpg'
}, {
  id: 3,
  title: 'Downtown Charm',
  owner: 'Violet Beauregarde',
  city: 'Portland',
  type: 'Apartment',
  bedrooms: 3,
  image: 'https://upload.wikimedia.org/wikipedia/commons/f/f7/Wheeldon_Apartment_Building_-_Portland_Oregon.jpg'
}];

export default Ember.Route.extend({
  model() {
    return rentals;
  }
});
```

Здесь мы используем сокращенный синтаксис ES6 для определения метода: `model()` — то же самое, что и `model: function()`.

Функция `model` работает как **hook**. То есть Ember будет вызывать ее в приложении в различное время. Hook model, который мы добавили в обработчик маршрута `rentals`, будет вызван, когда пользователь пройдет по маршруту `rentals`.

Hook `model` возвращает наш массив *rentals* и передает его шаблону `rentals` как свойство `model`.

Теперь переключимся на шаблон. Мы можем использовать данные модели, чтобы отобразить список недвижимости. Здесь мы будем использовать еще один распространенный хелпер Handlebars под названием `{{each}}`. Этот хелпер позволит нам пройти в цикле каждый из объектов в модели:

`app/templates/rentals.hbs`
```hbs
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Welcome!</h2>
  <p>
    We hope you find exactly what you're looking for in a place to stay.
    <br>Browse our listings, or use the search box below to narrow your search.
  </p>
  {{#link-to 'about' class="button"}}
    About Us
  {{/link-to}}
</div>

{{#each model as |rental|}}
  <article class="listing">
    <h3>{{rental.title}}</h3>
    <div class="detail owner">
      <span>Owner:</span> {{rental.owner}}
    </div>
    <div class="detail type">
      <span>Type:</span> {{rental.type}}
    </div>
    <div class="detail location">
      <span>Location:</span> {{rental.city}}
    </div>
    <div class="detail bedrooms">
      <span>Number of bedrooms:</span> {{rental.bedrooms}}
    </div>
  </article>
{{/each}}
```

В этом шаблоне мы проходим каждый объект модели и называем его *rental*. Для каждого объекта rental мы в дальнейшем создадим список с информацией о недвижимости.

Теперь, когда мы сформировали список недвижимости, наш приемочный тест проверяющий вывод недвижимости, должен быть пройден:

![passing list rentals tests](https://guides.emberjs.com/v2.7.0/images/model-hook/passing-list-rentals-tests.png)