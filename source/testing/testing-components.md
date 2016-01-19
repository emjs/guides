Компоненты можно проверить интеграционными тестами благодаря помощнику `moduleForComponent`.

Допустим, у нас есть компонент со свойством `style`, который обновляется, когда меняется значение свойства `name`. Атрибут компонента `style` связан со свойством `style`.

Вы можете сгенерировать собственный компонент с помощью `ember generate component pretty-color`.

`app/components/pretty-color.js`
```js
export default Ember.Component.extend({
  attributeBindings: ['style'],

  style: Ember.computed('name', function() {
    const name = this.get('name');
    return `color: ${name}`;
  })
});
```

`app/templates/components/pretty-color.hbs`
```hbs
Pretty Color: {{name}}
```

Помощник `moduleForComponent` найдет компонент по имени (`pretty-color`) и его шаблону (если он доступен). Убедитесь, что у вас установлено `integration: true`, чтобы была возможность проведения интеграционного теста.

`tests/integration/components/pretty-color-test.js`
```js
moduleForComponent('pretty-color', 'Integration | Component | pretty color', {
  integration: true
});
```

Каждый тест, который следует за вызовом помощника `moduleForComponent`, имеет доступ к функции `render()`. Это позволяет создать новый экземпляр компонента, объявив компонент в синтаксисе шаблона, как мы и хотим сделать в приложении.

Можно протестировать, обновится ли атрибут компонента `style` при изменении свойства компонента `name`. Это отразится в представленном коде HTML:

`tests/integration/components/pretty-color-test.js`
```js
test('should change colors', function(assert) {
  assert.expect(2);

  // set the outer context to red
  this.set('colorValue', 'red');

  this.render(hbs`{{pretty-color name=colorValue}}`);

  assert.equal(this.$('div').attr('style'), 'color: red', 'starts as red');

  this.set('colorValue', 'blue');

  assert.equal(this.$('div').attr('style'), 'color: blue', 'updates to blue');
});
```

Еще мы можем проверить этот компонент и убедиться, что контент его шаблона правильно отображается:

`tests/integration/components/pretty-color-test.js`
```js
test('should be rendered with its color name', function(assert) {
  assert.expect(2);

  this.set('colorValue', 'orange');

  this.render(hbs`{{pretty-color name=colorValue}}`);

  assert.equal(this.$().text().trim(), 'Pretty Color: orange', 'text starts as orange');

  this.set('colorValue', 'green');

  assert.equal(this.$().text().trim(), 'Pretty Color: green', 'text switches to green');

});
```

### Тестирование взаимодействия пользователя

Компоненты позволяют создавать функциональные, интерактивные, самостоятельные и индивидуальные элементы HTML. Важно тестировать методы компонента *и* взаимодействия пользователя с компонентом.

Представьте, что у вас есть следующий компонент, который немного меняется, когда щелкают по кнопке.

Вы можете сгенерировать собственный компонент с помощью `ember generate component magic-title`.

`app/components/magic-title.js`
```js
export default Ember.Component.extend({
  title: 'Hello World',

  actions: {
    updateTitle() {
      this.set('title', 'This is Magic');
    }
  }
});
```

`app/templates/components/magic-title.hbs`
```hbs
<h2>{{title}}</h2>

<button {{action "updateTitle"}}>
  Update Title
</button>
```

Триггеры jQuery можно использовать, чтобы имитировать взаимодействия пользователя и протестировать, будет ли обновляться заголовок при нажатии на кнопку:

`tests/integration/components/magic-title-test.js`
```js
test('should update title on button click', function(assert) {
  assert.expect(2);

  this.render(hbs`{{magic-title}}`);

  assert.equal(this.$('h2').text(), 'Hello World', 'initial text is hello world');

  //Click on the button
  this.$('button').click();

  assert.equal(this.$('h2').text(), 'This is Magic', 'title changes after click');
});
```

### Тестирование действий

Начиная с Ember 2, компоненты используют замкнутые действия. Они позволяют компонентам напрямую вызывать функции, предоставляемые внешними компонентами.
 
Предположим, что у нас есть компонент с формой для комментариев, который вызывает действие `submitComment`, когда форма подтверждается, и передаются ее данные.

Вы можете сгенерировать собственный компонент с помощью `ember generate component comment-form`.

`app/components/comment-form.js`
```js
export default Ember.Component.extend({
  comment: '',

  actions: {
    submitComment() {
      this.attrs.submitComment({ comment: this.get('comment') });
    }
  }
});
```

`app/templates/components/comment-form.hbs`
```hbs
<form {{action "submitComment" on="submit"}}>
  <label>Comment:</label>
  {{textarea value=comment}}

  <input type="submit" value="Submit"/>
</form>
```

Здесь представлен пример теста, который подтверждает, что при запуске внутреннего действия компонента `submitComment` вызывается заданная функция `externalAction`. В этом примере используется тест копии (фиктивная функция):

`tests/integration/components/comment-form-test.js`
```js
test('should trigger external action on form submit', function(assert) {

  // test double for the external action
  this.set('externalAction', (actual) => {
    let expected = { comment: 'You are not a wizard!' };
    assert.deepEqual(actual, expected, 'submitted value is passed to external action');
  });

  this.render(hbs`{{comment-form submitComment=(action externalAction)}}`);

  // fill out the form and force an onchange
  this.$('textarea').val('You are not a wizard!');
  this.$('textarea').change();

  // click the button to submit the form
  this.$('input').click();
});
```

### Заглушки под сервисы

В тех случаях, где компоненты зависят от сервисов Ember, для интеграционных тестов можно определить эти зависимости в качестве заглушек. Создать и зарегистрировать в исходном месте заглушки под сервисы Ember можно с помощью встроенной функции регистрации.

Представим, что у нас есть компонент, который использует сервис определения местоположения, чтобы показывать город и страну, где вы сейчас находитесь.

Можно создать собственный компонент с помощью `ember generate component location-indicator`.

`app/components/location-indicator.js`
```js
export default Ember.Component.extend({
  locationService: Ember.inject.service('location-service'),

  //when the coordinates change, call the location service to evaluate what the city and country would be
  city: Ember.computed('locationService.currentLocation', function () {
    return this.get('locationService').getCurrentCity();
  }),

  country: Ember.computed('locationService.currentLocation', function () {
    return this.get('locationService').getCurrentCountry();
  })
});
```

`app/templates/components/location-indicator.hbs`
```hbs
You currently are located in {{city}}, {{country}}
```

Чтобы сделать сервис-заглушку для теста, создайте локальный объект-заглушку, который расширяет `Ember.Service`. Затем зарегистрируйте заглушку в функции beforeEach как сервис, необходимый для тестов. В нашем случае мы изначально установили Нью-Йорк как текущее местоположение:

`tests/integration/components/location-indicator-test.js`
```js
import { moduleForComponent, test } from 'ember-qunit';
import hbs from 'htmlbars-inline-precompile';
import Ember from 'ember';

//Stub location service
const locationStub = Ember.Service.extend({
  city: 'New York',
  country: 'USA',
  currentLocation: {
    x: 1234,
    y: 5678
  },

  getCurrentCity() {
    return this.get('city');
  },
  getCurrentCountry() {
    return this.get('country');
  }
});

moduleForComponent('location-indicator', 'Integration | Component | location indicator', {
  integration: true,

  beforeEach: function () {
    this.register('service:location-service', locationStub);
    this.inject.service('location-service', { as: 'location' });
  }
});
```

Когда сервис-заглушка зарегистрирован, тесту остается лишь проверить, что данные заглушки, возвращенные от сервиса, отражаются в выводе компонента.

`tests/integration/components/location-indicator-test.js`
```js
test('should reveal current location', function(assert) {
  this.render(hbs`{{location-indicator}}`);
  assert.equal(this.$().text().trim(), 'You currently are located in New York, USA');
});
```

В следующем примере мы добавим еще один тест, который подтверждает, что отображение меняется, когда мы корректируем значения в сервисе.

`tests/integration/components/location-indicator-test.js`
```js
test('should change displayed location when current location changes', function (assert) {
  this.render(hbs`{{location-indicator}}`);
  assert.equal(this.$().text().trim(), 'You currently are located in New York, USA', 'origin location should display');
  this.set('location.city', 'Beijing');
  this.set('location.country', 'China');
  this.set('location.currentLocation', { x: 11111, y: 222222 });
  assert.equal(this.$().text().trim(), 'You currently are located in Beijing, China', 'location display should change');
});
```