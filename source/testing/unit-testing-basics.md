Модульные тесты обычно используются для проверки небольших частей кода. Они позволяют убедиться, что этот код выполняет то, для чего предназначен. В отличие от приемочных тестов их область действия уже, и для работы им не требуется, чтобы было запущено приложение Ember.

Так как в Ember существует основной тип объекта, возможность проверять простой `Ember.Object` дает основу для тестирования более специфических частей приложения Ember вроде контроллеров, компонентов и т.д. Тестирование `Ember.Object` представляет собой создание экземпляра объекта, установку его состояния и проверку утверждений на объекте. В качестве примера давайте посмотрим на несколько общих случаев.

### Тестирование вычислительных свойств

Давайте начнем с создания объекта, который имеет вычислительное свойство `computedFoo` на основе свойства `foo`.

`app/models/some-thing.js`
```js
export default Ember.Object.extend({
  foo: 'bar',

  computedFoo: Ember.computed('foo', function() {
    const foo = this.get('foo');
    return `computed ${foo}`;
  })
});
```

В тесте для этого объекта мы создадим экземпляр, обновим свойство `foo` (которое должно задействовать вычислительное свойство) и подтвердим, что логика в вычислительном свойстве работает правильно.

`tests/unit/models/some-thing-test.js`
```js
import { moduleFor, test } from 'ember-qunit';

moduleFor('model:some-thing', 'Unit | some thing', {
  unit: true
});

test('should correctly concat foo', function(assert) {
  const someThing = this.subject();
  someThing.set('foo', 'baz');
  assert.equal(someThing.get('computedFoo'), 'computed baz');
});
```

Здесь мы использовали `moduleFor`. Это один из нескольких помощников модульного тестирования, которые предоставляет Ember-Qunit. Помощники тестирования дают возможность воспользоваться некоторыми удобствами вроде функции, которая оперирует поиском и созданием экземпляра для тестируемого объекта. Обратите внимание, что в модульном тесте вы можете настроить создание экземпляра тестируемого объекта, если передадите функции объект, который содержит нужные вам переменные экземпляра. Например, чтобы установить свойство 'foo' для тестируемого объекта, мы могли бы вызвать `this.subject({ foo: 'bar' });`.

### Тестирования методов объекта

Теперь давайте взглянем на логику тестирования при работе с методом объекта. В нашем случае метод `testMethod` меняет внутреннее состояние объекта (обновляя свойство `foo`).

`app/models/some-thing.js`
```js
export default Ember.Object.extend({
  foo: 'bar',
  testMethod() {
    this.set('foo', 'baz');
  }
});
```

Чтобы протестировать его, мы создаем экземпляр класса `SomeThing`, который определен выше, вызываем метод `testMethod` и подтверждаем, что в результате вызова метода внутреннее состояние будет верным.

`tests/unit/models/some-thing-test.js`
```js
test('should update foo on testMethod', function(assert) {
  const someThing = this.subject();
  someThing.testMethod();
  assert.equal(someThing.get('foo'), 'baz');
});
```

В событии, когда метод объекта возвращает значение, вы можете просто подтвердить, что возвращенное значение вычислено верно. Предположим, что наш объект имеет метод `calc`, который возвращает значение на основе некоторого внутреннего состояния.  

`app/models/some-thing.js`
```js
export default Ember.Object.extend({
  count: 0,
  calc() {
    this.incrementProperty('count');
    let count = this.get('count');
    return `count: ${count}`;
  }
});
```

Тест вызовет метод `calc` и подтвердит, что возвращено верное значение.

`tests/unit/models/some-thing-test.js`
```js
test('should return incremented count on calc', function(assert) {
  const someThing = this.subject();
  assert.equal(someThing.calc(), 'count: 1');
  assert.equal(someThing.calc(), 'count: 2');
});
```

### Тестирование наблюдателей

Предположим, что у нас есть объект, который имеет свойство и метод, отслеживающий это свойство.

`app/models/some-thing.js`
```js
export default Ember.Object.extend({
  foo: 'bar',
  other: 'no',
  doSomething: Ember.observer('foo', function() {
    this.set('other', 'yes');
  })
});
```

Чтобы протестировать метод `doSomething`, мы создаем экземпляр `SomeThing`, обновляем отслеживаемое свойство (`foo`) и подтверждаем, что ожидаемые эффекты на месте.

`tests/unit/models/some-thing-test.js`
```js
test('should set other prop to yes when foo changes', function(assert) {
  const someThing = this.subject();
  someThing.set('foo', 'baz');
  assert.equal(someThing.get('other'), 'yes');
});
```