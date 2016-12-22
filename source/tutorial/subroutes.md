К этому моменту мы сгенерировали 4 маршрута верхнего уровня.

* Маршрут `about`, который предоставляет информацию о приложении.
* Маршрут `contact`, который предоставляет контакты для связи с компанией.
* Маршрут `rentals`, где мы даем пользователям возможность просматривать позиции с недвижимостью.
* Маршрут `index`, который при нашей настройке перенаправляет на маршрут rentals.

Маршрут `rentals` выполняет несколько функций. В [приемочных тестах](https://guides.emberjs.com/v2.8.0/tutorial/acceptance-test/) мы показали, что наша задача — дать пользователям возможность искать недвижимость, а также просматривать подробную информацию о ней. Чтобы выполнить это требование, мы воспользуемся [вложенными маршрутами](https://guides.emberjs.com/v2.8.0/routing/defining-your-routes/#toc_nested-routes) Ember.

К концу этого раздела нам нужно создать следующие новые маршруты:

* Маршрут `rentals/index`, который отображает информацию основной страницы и список доступной недвижимости. Вложенный маршрут index отображается по умолчанию, когда пользователь посещает URL `rentals`.
* Маршрут `rentals/show`, который все так же показывает информацию основной страницы и еще данные выбранной недвижимости. Маршрут `show` будет заменяться id отображаемой недвижимости (например, `rentals/grand-old-mansion`).

## Родительский маршрут

В главе [«Маршруты и шаблоны»](http://emjs.ru/v2/tutorial/routes-and-templates/) мы создали маршрут `rentals`.

При открытии шаблона этого маршрута мы видим заполнитель (outlet) под основной информацией страницы маршрута. Внизу шаблона вы заметите хелпер `{{outlet}}`. Именно здесь будет отображаться активный вложенный маршрут.

`app/templates/rentals.hbs`
```hbs
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Welcome!</h2>
  <p>We hope you find exactly what you're looking for in a place to stay.</p>
  {{#link-to 'about' class="button"}}
    About Us
  {{/link-to}}
</div>
{{#list-filter
   filter=(action 'filterByCity')
   as |rentals|}}
  <ul class="results">
    {{#each rentals as |rentalUnit|}}
      <li>{{rental-listing rental=rentalUnit}}</li>
    {{/each}}
  </ul>
{{/list-filter}}
{{outlet}}
```

Если есть родительский маршрут, значит, любые материалы в его шаблоне будут показаны во время просмотра дочерних маршрутов. Это позволяет добавить, например, общие инструкции, навигацию, футеры или боковые панели.

## Генерация вложенного маршрута index

Первым вложенным маршрутом будет index. Вложенный маршрут index работает так же, как базовый маршрут index. Это исходный маршрут, который отображается, когда нет других маршрутов.
Поэтому в нашем случае, когда мы перейдем в `/rentals`, Ember попытается загрузить маршрут index для rentals, как вложенный.

Чтобы создать вложенный маршрут index, запустите следующую команду:

```
ember g route rentals/index
```

Если вы откроете роутер (`app/router`), то увидите, что ничего не обновилось.

`app/router.js`
```js
Router.map(function() {
  this.route('about');
  this.route('contact');
  this.route('rentals');
});
```

Маршрут приложения `index` не добавляется в роутер, поэтому и маршруты `index` в подмаршрутах в нем не появятся. Ember знает, что по умолчанию нужно направить пользователя на маршрут `index`.
Но вы можете добавить маршрут `index`, если хотите настроить его под себя.
Например, вы можете изменить путь маршрута `index`, указав  `this.route('index', { path:
'/custom-path'})`.

В главе [«Использование Ember Data»](https://guides.emberjs.com/v2.8.0/tutorial/ember-data/#toc_updating-the-model-hook) мы добавили запрос всех позиций недвижимости. Реализуем новый сгенерированный маршрут `rentals/index`, переместив вызов `findAll` из родительского маршрута `rentals` в новый подмаршрут.

`app/routes/rentals.js`
```js
export default Ember.Route.extend({
});
```

`app/routes/rentals/index.js`
```js
export default Ember.Route.extend({
  model() {
    return this.store.findAll('rental');
  }
});
```

Теперь мы возвращаем все позиции недвижимости модели вложенного маршрута. Мы также переместим разметку списка недвижимости из основного шаблона маршрута в шаблон вложенного маршрута index. 

`app/templates/rentals.hbs`
```hbs
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Welcome!</h2>
  <p>We hope you find exactly what you're looking for in a place to stay.</p>
  {{#link-to 'about' class="button"}}
    About Us
  {{/link-to}}
</div>
{{outlet}}
```

`app/templates/rentals/index.hbs`
```hbs
{{#list-filter
   filter=(action 'filterByCity')
   as |rentals|}}
  <ul class="results">
    {{#each rentals as |rentalUnit|}}
      <li>{{rental-listing rental=rentalUnit}}</li>
    {{/each}}
  </ul>
{{/list-filter}}
{{outlet}}
```

Наконец, нам нужно сделать контроллер, который отвечает за фильтрацию, доступным в новом вложенном маршруте index.

Начнем с запуска команды `ember g controller rentals/index`, чтобы создать контроллер index для вложенного маршрута.

Вместо копирования всего файла контроллера в `app/controller/rentals/index.js` из `app/controller/rentals.js`, мы используем функцию import/export в JavaScript, чтобы экспортировать контроллер rentals как контроллер rentals/index:

`app/controller/rentals/index.js`
```js
import RentalsController from '../rentals';

export default RentalsController;
```

## Настройка данных для вложенного маршрута с информацией 

Далее, нам нужно создать подмаршрут, который будет показывать информацию по конкретной недвижимости.
Чтобы сделать это, нам нужно обновить несколько файлов. Чтобы найти конкретную недвижимость, нам нужно использовать функцию `findRecord` из Ember Data (смотрите [«Нахождение записей»](https://guides.emberjs.com/v2.8.0/models/finding-records/)). Для работы функции `findRecord` требуется, чтобы мы выполняли поиск по уникальному ключу.

И еще мы хотим, чтобы маршрут `show` показывал дополнительную информацию о конкретной недвижимости.

Чтобы сделать это, нам нужно изменить файл `config.js` в Mirage, который мы добавляли в главе [«Установка дополнений»](http://emjs.ru/v2/tutorial/installing-addons/). Мы добавим обработчик нового маршрута, чтобы возвращать конкретную недвижимость:

`mirage/config.js`
```js
export default function() {
  this.namespace = '/api';

  let rentals = [
    {
      type: 'rentals',
      id: 'grand-old-mansion',
      attributes: {
        title: "Grand Old Mansion",
        owner: "Veruca Salt",
        city: "San Francisco",
        type: "Estate",
        bedrooms: 15,
        image: "https://upload.wikimedia.org/wikipedia/commons/c/cb/Crane_estate_(5).jpg",
        description: "This grand old mansion sits on over 100 acres of rolling hills and dense redwood forests."
      }
    },
    {
      type: 'rentals',
      id: 'urban-living',
      attributes: {
        title: "Urban Living",
        owner: "Mike Teavee",
        city: "Seattle",
        type: "Condo",
        bedrooms: 1,
        image: "https://upload.wikimedia.org/wikipedia/commons/0/0e/Alfonso_13_Highrise_Tegucigalpa.jpg",
        description: "A commuters dream. This rental is within walking distance of 2 bus stops and the Metro."
      }
    },
    {
      type: 'rentals',
      id: 'downtown-charm',
      attributes: {
        title: "Downtown Charm",
        owner: "Violet Beauregarde",
        city: "Portland",
        type: "Apartment",
        bedrooms: 3,
        image: "https://upload.wikimedia.org/wikipedia/commons/f/f7/Wheeldon_Apartment_Building_-_Portland_Oregon.jpg",
        description: "Convenience is at your doorstep with this charming downtown rental. Great restaurants and active night life are within a few feet."
      }
    }
  ];

  this.get('/rentals', function(db, request) {
    if (request.queryParams.city !== undefined) {
      let filteredRentals = rentals.filter(function (i) {
        return i.attributes.city.toLowerCase().indexOf(request.queryParams.city.toLowerCase()) !== -1;
      });
      return { data: filteredRentals };
    } else {
      return { data: rentals };
    }
  });

  // Find and return the provided rental from our rental list above
  this.get('/rentals/:id', function (db, request) {
    return { data: rentals.find((rental) => request.params.id === rental.id) };
  });

};
```

## Генерация вложенного маршрута с информацией

Теперь, когда наш API готов возвращать отдельные позиции недвижимости, мы можем сгенерировать подмаршрут `show`. Как и при генерации маршрута `rentals`, мы будем использовать команду `ember g`, чтобы создать вложенный маршрут.

```
ember g route rentals/show
```

Вы увидите примерно такой результат:

```
installing route
  create app/routes/rentals/show.js
  create app/templates/rentals/show.hbs
updating router
  add route rentals/show
installing route-test
  create tests/unit/routes/rentals/show-test.js
```

Начнем с просмотра изменений в роутере (`app/router.js`).

`app/router.js`
```js
Router.map(function() {
  this.route('about');
  this.route('contact');
  this.route('rentals', function() {
    this.route('show');
  });
});
```

Новый маршрут вложен в маршрут `rentals`. Ember понимает, что это подмаршрут и доступ к нему предоставляется по адресу `localhost:4200/rentals/show`.

Чтобы указать приложению, к какой недвижимости мы хотим получить доступ, нам нужно поменять путь маршрута `show` на ID позиции недвижимости. Еще нужно упростить URL, чтобы он выглядел примерно так: `localhost:4200/rentals/id-for-rental`.

Для этого мы изменим маршрут следующим образом:

`app/router.js`
```js
Router.map(function() {
  this.route('about');
  this.route('contact');
  this.route('rentals', function() {
    this.route('show', { path: '/:rental_id' });
  });
});
```

Теперь `rental_id` будет передан маршруту.

## Нахождение по ID

Далее, нам нужно отредактировать маршрут `show`, чтобы возвращать запрашиваемую недвижимость:

`app/routes/rentals/show.js`
```js
export default Ember.Route.extend({
  model(params) {
    return this.store.findRecord('rental', params.rental_id);
  }
});
```

Так как мы добавили `:rental_id` в путь `show` в роутере, теперь `rental_id` доступен в hook `model`.
Когда мы вызываем `this.store.findRecord('rental', params.rental_id)`, Ember Data обращается к `/rentals/our-id` с помощью запроса HTTP GET ([подробнее об этом можно узнать здесь](https://guides.emberjs.com/v2.8.0/models/)).

## Добавление недвижимости в шаблон

Далее, мы можем обновить шаблон для маршрута show (`app/templates/rentals/show.hbs`) и список информации для недвижимости.

`app/templates/rentals/show.hbs`
```hbs
<div class="jumbo show-listing">
  <h2 class="title">{{model.title}}</h2>
  <div class="right detail-section">
    <div class="detail owner">
      <strong>Owner:</strong> {{model.owner}}
    </div>
    <div class="detail">
      <strong>Type:</strong> {{rental-property-type model.type}} - {{model.type}}
    </div>
    <div class="detail">
      <strong>Location:</strong> {{model.city}}
    </div>
    <div class="detail">
      <strong>Number of bedrooms:</strong> {{model.bedrooms}}
    </div>
    <p>{{model.description}}</p>
  </div>
  <img src="{{model.image}}" class="rental-pic">
</div>
```

Теперь перейдите по адресу `localhost:4200/rentals/grand-old-mansion`, и вы увидите информацию об этой недвижимости.

![subroutes super rentals show](https://guides.emberjs.com/v2.8.0/images/subroutes/subroutes-super-rentals-show.png)

## Создание ссылки на конкретную недвижимость

Теперь, когда мы можем загружать страницы для конкретной недвижимости, мы добавим ссылку (с помощью хелпера `link-to`) в компонент `rental-listing`, чтобы переходить на отдельные страницы. Здесь хелпер `link-to` принимает имя маршрута и объект модели rental как аргументы. Когда вы передаете объект в качестве второго аргумента блочному хелперу `link-to`, он по умолчанию [преобразует](http://emberjs.com/api/classes/Ember.Route.html#method_serialize) объект в ID модели и в URL. Но для ясности вы можете просто передавать `rental.id`.

По нажатию заголовка будет загружаться страница с информацией о конкретной недвижимости.

`app/templates/components/rental-listing.hbs`
```hbs
<article class="listing">
  <a {{action 'toggleImageSize'}} class="image {{if isWide "wide"}}">
    <img src="{{rental.image}}" alt="">
    <small>View Larger</small>
  </a>
  <h3>{{#link-to "rentals.show" rental}}{{rental.title}}{{/link-to}}</h3>
  <div class="detail owner">
    <span>Owner:</span> {{rental.owner}}
  </div>
  <div class="detail type">
    <span>Type:</span> {{rental-property-type rental.type}} - {{rental.type}}
  </div>
  <div class="detail location">
    <span>Location:</span> {{rental.city}}
  </div>
  <div class="detail bedrooms">
    <span>Number of bedrooms:</span> {{rental.bedrooms}}
  </div>
  {{location-map location=rental.city}}
</article>
```

![subroutes super rentals index](https://guides.emberjs.com/v2.8.0/images/subroutes/subroutes-super-rentals-index.png)

## Финальная проверка

Сейчас все тесты должны быть пройдены, включая [список приемочных тестов](https://guides.emberjs.com/v2.8.0/tutorial/acceptance-test/), которые мы создали в качестве начальных требований.

![images/subroutes/all acceptance pass](https://guides.emberjs.com/v2.8.0/images/subroutes/all-acceptance-pass.png)

И теперь вы можете [развернуть](https://guides.emberjs.com/v2.8.0/tutorial/deploying/) приложение Super Rentals и поделиться им с миром или использовать его как базу для освоения других особенностей Ember и дополнений. В любом случае мы надеемся, что это помогло вам понять принцип создания амбициозных приложений с помощью Ember!