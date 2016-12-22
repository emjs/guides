При поиске недвижимости у пользователей должна быть возможность сузить круг до конкретного города.
Создадим компонент, который позволит им фильтровать недвижимость по определенному городу.

Для начала сгенерируем новый компонент. Мы назовем этот компонент `list-filter`, так как хотим, чтобы он фильтровал список недвижимости на основе текста в поле ввода.

```
ember g component list-filter
```

В итоге мы получим шаблон Handlebars (`app/templates/components/list-filter.hbs`), файл JavaScript (`app/components/list-filter.js`) и интеграционный тест компонента (`tests/integration/components/list-filter-test.js`).

Начнем с написания тестов, которые помогут продумать план действий. Компонент для фильтрации должен предоставить список отсортированных элементов и отобразить его в своем внутреннем блоке шаблона.
Нам нужно, чтобы компонент вызывал два действия: предоставлял список всех элементов, когда нет фильтрации, и искал позиции по городу.

Для изначального теста мы проверим, чтобы все предоставленные нами города отображались и объект списка был доступен из шаблона.

Так как мы планируем использовать Ember Data в качестве хранилища модели, нам нужно, чтобы при вызовах действий данные запрашивались асинхронно. Поэтому мы будем возвращать обещания. Так как доступ к сохраненным данным обычно осуществляется асинхронно, нам нужно добавить хелпер wait в конец теста. Он будет ждать разрешения всех обещаний, прежде чем завершить тест.

`tests/integration/components/list-filter-test.js`
```js
import { moduleForComponent, test } from 'ember-qunit';
import hbs from 'htmlbars-inline-precompile';
import wait from 'ember-test-helpers/wait';
import RSVP from 'rsvp';

moduleForComponent('list-filter', 'Integration | Component | filter listing', {
  integration: true
});

const ITEMS = [{city: 'San Francisco'}, {city: 'Portland'}, {city: 'Seattle'}];
const FILTERED_ITEMS = [{city: 'San Francisco'}];

test('should initially load all listings', function (assert) {
  // we want our actions to return promises, since they are potentially fetching data asynchronously
  this.on('filterByCity', (val) => {
    if (val === '') {
      return RSVP.resolve(ITEMS);
    } else {
      return RSVP.resolve(FILTERED_ITEMS);
    }
  });

  // with an integration test, you can set up and use your component in the same way your application 
  // will use it.
  this.render(hbs`
    {{#list-filter filter=(action 'filterByCity') as |results|}}
      <ul>
      {{#each results as |item|}}
        <li class="city">
          {{item.city}}
        </li>
      {{/each}}
      </ul>
    {{/list-filter}}
  `);

  // the wait function will return a promise that will wait for all promises 
  // and xhr requests to resolve before running the contents of the then block.
  return wait().then(() => {
    assert.equal(this.$('.city').length, 3);
    assert.equal(this.$('.city').first().text().trim(), 'San Francisco');
  });
});
```

Во втором тесте мы проверим, чтобы введенный в поле текст запускал фильтрацию и обновлял списки.

Чтобы запустить действие, мы сгенерируем событие `keyUp` для поля ввода, а затем удостоверимся, что отображен только один элемент.

`tests/integration/components/list-filter-test.js`
```js
test('should update with matching listings', function (assert) {
  this.on('filterByCity', (val) => {
    if (val === '') {
      return RSVP.resolve(ITEMS);
    } else {
      return RSVP.resolve(FILTERED_ITEMS);
    }
  });

  this.render(hbs`
    {{#list-filter filter=(action 'filterByCity') as |results|}}
      <ul>
      {{#each results as |item|}}
        <li class="city">
          {{item.city}}
        </li>
      {{/each}}
      </ul>
    {{/list-filter}}
  `);

  // The keyup event here should invoke an action that will cause the list to be filtered
  this.$('.list-filter input').val('San').keyup();

  return wait().then(() => {
    assert.equal(this.$('.city').length, 1);
    assert.equal(this.$('.city').text().trim(), 'San Francisco');
  });
});
```

Далее, в файл `app/templates/rentals.hbs` мы добавим новый компонент `list-filter`; так же, как делали в нашем тесте. Вместо того чтобы просто показывать город, мы будем использовать компонент `rental-listing`, чтобы отобразить информацию о недвижимости.

`app/templates/rentals.hbs`
```hbs
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Welcome!</h2>
  <p>
    We hope you find exactly what you're looking for in a place to stay.
    <br>Browse our listings, or use the search box above to narrow your search.
  </p>
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
```

Теперь, когда у нас есть заведомо провальные тесты и понимание того, каким должен быть компонент, начнем его реализацию. Нам нужно, чтобы в компоненте было поле ввода и он предоставлял список с результатами в своем блоке. Поэтому шаблон будет простым:

`app/templates/components/list-filter.hbs`
```hbs
{{input value=value key-up=(action 'handleFilterEntry') class="light" placeholder="Filter By City"}}
{{yield results}}
```

Шаблон содержит хелпер [`{{input}}`](http://emjs.ru/v2/templates/input-helpers/), который отображается как текстовое поле. В нем пользователь может набрать текстовый шаблон и отфильтровать по нему недвижимость. Свойство `value` в `input` будет связано со свойством `value` в компоненте.
Свойство `key-up` будет связано с действием `handleFilterEntry`.

Так выглядит JavaScript компонента:

`app/components/list-filter.js`
```js
import Ember from 'ember';

export default Ember.Component.extend({
  classNames: ['list-filter'],
  value: '',

  init() {
    this._super(...arguments);
    this.get('filter')('').then((results) => this.set('results', results));
  },

  actions: {
    handleFilterEntry() {
      let filterInputValue = this.get('value');
      let filterAction = this.get('filter');
      filterAction(filterInputValue).then((filterResults) => this.set('results', filterResults));
    }
  }

});
```

Мы используем hook `init`, чтобы отобразить изначальные позиции при вызове действия `filter` с пустым значением. Действие `handleFilterEntry` запускает фильтрацию на основе атрибута `value`, заданного хелпером input.

Вызывающий объект [передает](http://emjs.ru/v2/components/triggering-changes-with-actions/) действие `filter`. Такая схема называется *замкнутым действием*.

Чтобы реализовать эти действия, мы создадим контроллер `rentals` для приложения. Контроллер index будет задействован, когда пользователь перейдет по исходному маршруту (index) приложения.

Сгенерируйте контроллер для страницы `rentals`, запустив следующее:

```
ember g controller rentals
```

Теперь определяем новый контроллер так:

`app/controllers/rentals.js`
```js
import Ember from 'ember';

export default Ember.Controller.extend({
  actions: {
    filterByCity(param) {
      if (param !== '') {
        return this.get('store').query('rental', { city: param });
      } else {
        return this.get('store').findAll('rental');
      }
    }
  }
});
```

Когда пользователь набирает текст в поле компонента, в контроллере вызывается действие `filterByCity`. Это действие берет свойство `value` и фильтрует данные `rental` в хранилище в поисках записей, которые соответствуют тому, что ввел пользователь. Результат запроса возвращается инициатору вызова.

Чтобы это действие работало, нам нужно изменить файл Mirage `config.js`, чтобы он смог отвечать на запросы.

`mirage/config.js`
```js
export default function() {
  this.namespace = '/api';

  let rentals = [{
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
    }];

  this.get('/rentals', function(db, request) {
    if(request.queryParams.city !== undefined) {
      let filteredRentals = rentals.filter(function(i) {
        return i.attributes.city.toLowerCase().indexOf(request.queryParams.city.toLowerCase()) !== -1;
      });
      return { data: filteredRentals };
    } else {
      return { data: rentals };
    }
  });
}
```

После обновления настроек Mirage тесты будут пройдены, а мы увидим простой фильтр на главной странице, который обновляет список недвижимости при введении текста в поле:

![styled super rentals filter](https://guides.emberjs.com/v2.7.0/images/autocomplete-component/styled-super-rentals-filter.png)
![passing acceptance tests](https://guides.emberjs.com/v2.7.0/images/autocomplete-component/passing-acceptance-tests.png)