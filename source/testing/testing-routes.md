*Модульное тестирование методов и вычислительных свойств происходит согласно предыдущим паттернам, которые описаны в [Основах модульного тестирования](http://emjs.ru/v2/testing/unit-testing-basics), так как Ember.Route служит расширением для Ember.Object.*

Тестирование маршрутов можно выполнить через приемочные или модульные тесты. Приемочные тесты обеспечивают для маршрутов лучший охват, так как маршруты в основном используются для выполнения переходов и загрузки данных. А оба этих процесса проще тестировать в контексте, чем изолированно.

Но иногда важно проводить модульное тестирование маршрутов. Например, нам нужно предупреждение, которое можно запустить с любого места в пределах нашего приложения. Функцию предупреждения `displayAlert` следует вставить в `ApplicationRoute`, так как все действия и события распространяются до него от контроллеров и нижних маршрутов.

По умолчанию, Ember CLI не генерирует файл для маршрута приложения. Чтобы расширить поведение маршрута приложения, мы запустим команду `ember generate route application`. Но при этом Ember CLI сгенерирует шаблон приложения. Поэтому, когда нас спросят, хотим ли мы переписать `app/templates/application.hbs`, мы ответим 'n'.

`app/routes/application.js`
```js
export default Ember.Route.extend({
  actions: {
    displayAlert(text) {
      this._displayAlert(text);
    }
  },

  _displayAlert(text) {
    alert(text);
  }
});
```

В этом маршруте мы [разделили наши понятия](http://en.wikipedia.org/wiki/Separation_of_concerns): действие `displayAlert` содержит код, который вызывается, когда получено действие, а закрытая функция `_displayAlert` выполняет задачу. В этом случае разделение кода на небольшие части (или «понятия») необязательно из-за малого размера функций. Но это позволяет легко изолировать части кода для тестирования, что, в свою очередь, упрощает поиск ошибок.

Вот пример модульного теста для этого маршрута:

`tests/unit/routes/application-test.js`
```js
import { moduleFor, test } from 'ember-qunit';

let originalAlert;

moduleFor('route:application', 'Unit | Route | application', {
  beforeEach() {
    originalAlert = window.alert; // store a reference to window.alert
  },

  afterEach() {
    window.alert = originalAlert; // restore window.alert
  }
});

test('should display an alert', function(assert) {
  assert.expect(2);

  // with moduleFor, the subject returns an instance of the route
  let route = this.subject();

  // stub window.alert to perform a qunit test
  const expectedTextFoo = 'foo';
  window.alert = (text) => {
    assert.equal(text, expectedTextFoo, `expect alert to display ${expectedTextFoo}`);
  };

  // call the _displayAlert function which triggers the qunit test above
  route._displayAlert(expectedTextFoo);

  // set up a second stub to perform a test with the actual action
  const expectedTextBar = 'bar';
  window.alert = (text) => {
    assert.equal(text, expectedTextBar, `expected alert to display ${expectedTextBar}`);
  };

  // Now use the routes send method to test the actual action
  route.send('displayAlert', expectedTextBar);
});
```