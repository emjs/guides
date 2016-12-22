Когда пользователь просматривает список недвижимости, ему могут понадобиться интерактивные опции, которые помогут сделать выбор. Добавим возможность переключать размер изображения недвижимости для каждой позиции. Чтобы сделать это, мы используем компонент.

Сначала сгенерируем компонент `rental-listing`, который будет управлять поведением каждого элемента списка. Чтобы избежать конфликта с элементами HTML, в имени каждого компонента нужно использовать тире. Поэтому `rental-listing` подойдет, а `rental` уже нет.

```
ember g component rental-listing
```

Ember CLI сгенерирует несколько файлов для компонента:

```
installing component
  create app/components/rental-listing.js
  create app/templates/components/rental-listing.hbs
installing component-test
  create tests/integration/components/rental-listing-test.js
```

Мы начнем с реализации заведомо провального теста с таким поведением изображения, которое нам нужно.

Для интеграционного теста мы создадим заглушку rental, которая имеет все свойства модели rental.
Мы сделаем так, что компонент изначально отображается без класса `wide`.
Нажатие изображения будет добавлять класс `wide` элементу, а повторное нажатие — убирать его.
Учтите, что мы находим элемент изображения с помощью CSS-селектора `.image`.

`tests/integration/components/rental-listing-test.js`
```js
import { moduleForComponent, test } from 'ember-qunit';
import hbs from 'htmlbars-inline-precompile';
import Ember from 'ember';

moduleForComponent('rental-listing', 'Integration | Component | rental listing', {
  integration: true
});

test('should toggle wide class on click', function(assert) {
  assert.expect(3);
  let stubRental = Ember.Object.create({
    image: 'fake.png',
    title: 'test-title',
    owner: 'test-owner',
    type: 'test-type',
    city: 'test-city',
    bedrooms: 3
  });
  this.set('rentalObj', stubRental);
  this.render(hbs`{{rental-listing rental=rentalObj}}`);
  assert.equal(this.$('.image.wide').length, 0, 'initially rendered small');
  this.$('.image').click();
  assert.equal(this.$('.image.wide').length, 1, 'rendered wide after click');
  this.$('.image').click();
  assert.equal(this.$('.image.wide').length, 0, 'rendered small after second click');
});
```

Компонент состоит из двух частей:

* Шаблона, который определяет внешний вид (`app/templates/components/rental-listing.hbs`)
* Исходного файла JavaScript (`app/components/rental-listing.js`), который определяет логику работы компонента.

Наш новый компонент `rental-listing` будет отвечать за то, как пользователь видит и взаимодействует с rental. Чтобы начать, переместим детали отображения для одного элемента rental из шаблона `rentals.hbs` в `rental-listing.hbs` и добавим поле с изображением:

`app/templates/components/rental-listing.hbs`
```hbs
<article class="listing">
  <img src="{{rental.image}}" alt="">
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
```

В шаблоне `rentals.hbs` разместим старую разметку HTML в цикле `{{#each}}` с новым компонентом `rental-listing`:

`app/templates/rentals.hbs`
```hbs
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Welcome!</h2>
  <p>
    We hope you find exactly what you're looking for in a place to stay.
  </p>
  {{#link-to 'about' class="button"}}
    About Us
  {{/link-to}}
</div>

{{#each model as |rentalUnit|}}
  {{rental-listing rental=rentalUnit}}
{{/each}}
```

Здесь мы вызываем компонент `rental-listing` по имени и назначаем каждый `rentalUnit` как атрибут `rental` компонента.

## Скрыть и показать изображение

Теперь мы можем добавить функциональность, которая позволит по запросу пользователя показать изображение rental.

Используем хелпер `{{#if}}`, чтобы крупнее показывать текущее изображение rental только в том случае, если `isWide` — true. Для этого установим класс элемента — `wide`. Также мы добавим текст и укажем, что изображение можно нажать, и заключим обе строки в тег привязки, задав ей классовое имя `image`, чтобы тест смог найти ее.

`app/templates/components/rental-listing.hbs`
```hbs
<article class="listing">
  <a class="image {{if isWide "wide"}}">
    <img src="{{rental.image}}" alt="">
    <small>View Larger</small>
  </a>
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
```

Значение `isWide` поступает из файла JavaScript в компоненте. В нашем случае это `rental-listing.js`. Так как нам нужно, чтобы изначально изображение было меньше, мы установим свойство на `false`:

`app/components/rental-listing.js`
```js
import Ember from 'ember';

export default Ember.Component.extend({
  isWide: false
});
```

Чтобы пользователь мог увеличить изображение, нам нужно добавить действие, которое будет переключать значение `isWide`. Назовем это действие `toggleImageSize`.

`app/templates/components/rental-listing.hbs`
```hbs
<article class="listing">
  <a {{action 'toggleImageSize'}} class="image {{if isWide "wide"}}">
    <img src="{{rental.image}}" alt="">
    <small>View Larger</small>
  </a>
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
```

Нажатие ссылки будет посылать действие компоненту.
Затем Ember будет просматривать хеш `actions` и вызывать функцию `toggleImageSize`.
Создадим функцию `toggleImageSize` и переключим свойство `isWide` в компоненте:

`app/components/rental-listing.js`
```js
import Ember from 'ember';

export default Ember.Component.extend({
  isWide: false,
  actions: {
    toggleImageSize() {
      this.toggleProperty('isWide');
    }
  }
});
```

Теперь, когда мы нажимаем изображение или ссылку `View Larger` (увеличить) в браузере, мы видим, как оно становится больше. Когда мы нажимаем увеличенное изображение, мы видим, как оно становится меньше.

![styled rental listings](https://guides.emberjs.com/v2.7.0/images/simple-component/styled-rental-listings.png)