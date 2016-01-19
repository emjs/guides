Теперь давайте добавим список доступной недвижимости в шаблон index. Сами пункты в списке не должны быть статичными, так как пользователи со временем будут их добавлять, обновлять и удалять. Поэтому нам нужно, чтобы модель *rentals* сохраняла информацию о недвижимости. Для простоты сначала мы используем строго закодированный массив объектов JavaScript. А потом мы переключимся на Ember Data, библиотеку для управления данными в приложении.

Вот так будет выглядеть наша страница, когда мы все сделаем:

![super rentals homepage with rentals list](/static/images/guides/tutorial/super-rentals-index-with-list.png)

В Ember загрузкой данных модели занимаются обработчики маршрута. Давайте откроем `app/routes/index.js` и добавим наши строго закодированные данные в качестве возвращаемого значения hook `model`:

`app/routes/index.js`
```js
import Ember from 'ember';

var rentals = [{
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
  },
});
```

Здесь мы используем сокращенные варианты синтаксиса ES6: `model()` — то же самое, что и `model: function()`.

Функция `model` работает как **hook**. То есть Ember вызовет ее для нас в определенное  время. Hook model, который мы добавили в обработчик маршрута `index`, будет вызываться, когда пользователь пройдет по маршруту `index`.

Hook `model` возвращает наш массив *rentals* и передает его шаблону `index` как свойство `model`.

Теперь давайте переключимся на шаблон. Мы можем использовать данные модели, чтобы отобразить список недвижимости. Здесь мы используем еще один распространенный помощник Handlebars под названием `{{each}}`. Этот помощник позволит нам пройти в цикле по каждому из объектов в модели:

`app/templates/index.hbs`
```hbs
<h1> Welcome to Super Rentals </h1>

We hope you find exactly what you're looking for in a place to stay.

{{#each model as |rental|}}
  <h2>{{rental.title}}</h2>
  <p>Owner: {{rental.owner}}</p>
  <p>Type: {{rental.type}}</p>
  <p>Location: {{rental.city}}</p>
  <p>Number of bedrooms: {{rental.bedrooms}}</p>
{{/each}}
```

В этом шаблоне мы проходим по каждому объекту модели и называем его *rental*. Для каждого rental мы в дальнейшем создадим перечень информации о недвижимости.