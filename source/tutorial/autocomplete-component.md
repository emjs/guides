При поиске недвижимости пользователи явно хотели бы сузить круг, указав конкретный город. Давайте создадим компонент, который позволит им искать недвижимость в пределах определенного города и будет предлагать им варианты по мере набора текста.

Для начала сгенерируем новый компонент. Мы назовем его `filter-listing`.

```bash
ember g component filter-listing
```

В итоге мы получим шаблон Handlebars (`app/templates/components/filter-listing.hbs`) и файл JavaScript (`app/components/filter-listing.js`).

Шаблон Handlebars выглядит следующим образом:

`app/templates/components/filter-listing.hbs`
```hbs
City: {{input value=filter key-up=(action 'autoComplete' filter)}} 
<button {{action 'search'}}>Search</button>

<ul>
{{#each filteredList as |item|}}
  <li {{action 'choose' item.city}}>{{item.city}}</li>
{{/each}}
</ul>
```

Он содержит хелпер [`{{input}}`](/v2/templates/input-helpers), который отображается как текстовое поле. В нем пользователь может набирать текст для поиска недвижимости в конкретном городе. Свойство `value` у `input` будет связано со свойством `filter` в нашем компоненте. Свойство `key-up` будет связано с действием `autoComplete` в базовом объекте и передавать свойство `filter` как параметр.

Шаблон также содержит кнопку, чей параметр `action` связан с действием `search` в нашем компоненте.

Наконец, он содержит неупорядоченный список, который использует свойство `filteredList` для данных и отображает свойство `city` каждого элемента в списке. Нажатие на элемент списка запустит действие `choose`, которое заполнит поле `input`наименованием `city` (города) выбранного элемента.

Вот как выглядит JavaScript компонента:

`app/components/filter-listing.js`
```js
export default Ember.Component.extend({
  filter: null,
  filteredList: null,
  actions: {
    autoComplete() {
      this.get('autoComplete')(this.get('filter'));
    },
    search() {
      this.get('search')(this.get('filter'));
    },
    choose(city) {
      this.set('filter', city);
    }
  }
});
```

Как описано выше, для каждого `filter` и `filteredList` есть свойство и действия.  Любопытно, что только действие `choose` определяется компонентом. Фактическая логика каждого действия `autoComplete` и `search` извлекается из свойств компонента. Это значит, что эти действия [передаются](/v2/components/triggering-changes-with-actions/#toc_passing-the-action-to-the-component) при вызове объекта. Эта модель известна как *замкнутые действия*.

Чтобы посмотреть, как это работает, изменим шаблон `index.hbs` таким образом:

`app/templates/index.hbs`
```hbs
<h1>Welcome to Super Rentals</h1>

We hope you find exactly what you're looking for in a place to stay.
<br /><br />
{{filter-listing filteredList=filteredList 
autoComplete=(action 'autoComplete') search=(action 'search')}}
{{#each model as |rentalUnit|}}
  {{rental-listing rental=rentalUnit}}
{{/each}}

{{#link-to 'about'}}About{{/link-to}}
{{#link-to 'contact'}}Click here to contact us.{{/link-to}}
```

Мы добавляем компонент `filter-listing` к шаблону `index.hbs`. Затем мы передаем функции и свойства, которые будет использовать компонент `filter-listing`, чтобы страница `index` могла определять некоторую часть поведения компонента, и чтобы компонент мог использовать специфические функции и свойства.

Чтобы все это работало, нам нужно ввести в приложение контроллер. Генерируем контроллер для страницы `index`, запустив следующее:

```bash
ember g controller index
```

Теперь определяем наш новый контроллер так:

`app/controllers/index.js`
```js
export default Ember.Controller.extend({
  filteredList: null,
  actions: {
    autoComplete(param) {
      if(param !== "") {
        this.store.query('rental', {city: param}).then((result) => {
          this.set('filteredList',result);
        });
      } else {
        this.set('filteredList').clear();
      }
    },
    search(param) {
      if(param !== "") {
        this.store.query('rental', {city: param}).then((result) => {
          this.set('model',result);
        });
      } else {
        this.set('model').clear();
      }
    }
  }
});
```

Как вы можете видеть, мы определяем свойство в контроллере под названием `filteredList`, к которому обращаются из действия `autoComplete`. Это действие вызывается, когда пользователь набирает текст в поле компонента. Оно фильтрует данные `rental`, чтобы найти в них записи, которые соответствуют тому, что пользователь уже набрал в поле. Когда действие выполняется, результат запроса помещается в свойство `filteredList`, которое используется, чтобы заполнить список автозавершения компонента.

Мы также определяем здесь действие `search`, которое передается компоненту и вызывается, когда нажимают на кнопку поиска. Разница в том, что результат запроса используется, чтобы обновить модель маршрута `index`, и это меняет весь список недвижимости на странице.

Чтобы эти действия работали, нам нужно подкорректировать файл Mirage `config.js`. Так он сможет отвечать на запросы.

`app/mirage/config.js`
```js
export default function() {
  this.get('/rentals', function(db, request) {
    let rentals = [{
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
      }];

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

С помощью этих изменений пользователи смогут искать недвижимость в пределах конкретного города, а в поле ввода будут появляться варианты по мере набора текста.  