*Модульное тестирование методов и вычислительных свойств происходит согласно предыдущим паттернам, которые описаны в [Основах модульного тестирования](http://emjs.ru/v2/testing/unit-testing-basics), так как Ember.Model служит расширением для Ember.Object.*

Модели [Ember Data](https://github.com/emberjs/data) можно протестировать с использованием помощника `moduleForModel`.

Давайте предположим, что у нас есть модель `Player`, которая имеет атрибуты `level` и `levelName`. Мы хотим вызвать `levelUp()`, чтобы увеличить `level` и назначить новый `levelName`, когда игрок достигнет уровня 5.

Вы можете сгенерировать собственную модель с помощью `ember generate model player`.

`app/models/player.js`
```js
export default DS.Model.extend({
  level:     DS.attr('number', { defaultValue: 0 }),
  levelName: DS.attr('string', { defaultValue: 'Noob' }),

  levelUp() {
    var newLevel = this.incrementProperty('level');
    if (newLevel === 5) {
      this.set('levelName', 'Professional');
    }
  }
});
```

Теперь давайте создадим тест, который будет вызывать `levelUp` для игрока, когда он имеет уровень 4, чтобы подтвердить изменения `levelName`. Мы используем `moduleForModel`:

`tests/unit/models/player-test.js`
```js
import { moduleForModel, test } from 'ember-qunit';
import Ember from 'ember';

moduleForModel('player', 'Unit | Model | player', {
  // Specify the other units that are required for this test.
  needs: []
});

test('should increment level when told to', function(assert) {
  // this.subject aliases the createRecord method on the model
  const player = this.subject({ level: 4 });

  // wrap asynchronous call in run loop
  Ember.run(() => player.levelUp());

  assert.equal(player.get('level'), 5, 'level gets incremented');
  assert.equal(player.get('levelName'), 'Professional', 'new level is called professional');
});
```

## Тестирование связей

В отношении связей, скорее всего, вы хотели бы протестировать только один момент: то, что объявления связей установлены правильно.

Предположим, что `User` имеет `Profile`.

Вы можете сгенерировать собственные модели user и profile с помощью `ember generate model user` и `ember generate model profile`.

`app/models/profile.js`
```js
export default DS.Model.extend({
});
```

`app/models/user.js`
```js
export default DS.Model.extend({
  profile: DS.belongsTo('profile')
});
```

Затем с помощью этого теста можно проверить, что связи установлены правильно:

`tests/unit/models/user-test.js`
```js
import { moduleForModel, test } from 'ember-qunit';
import Ember from 'ember';

moduleForModel('user', 'Unit | Model | user', {
  // Specify the other units that are required for this test.
  needs: ['model:profile']
});

test('should own a profile', function(assert) {
  const User = this.store().modelFor('user');
  const relationship = Ember.get(User, 'relationshipsByName').get('profile');

  assert.equal(relationship.key, 'profile', 'has relationship with profile');
  assert.equal(relationship.kind, 'belongsTo', 'kind of relationship is belongsTo');
});
```

*Ember Data содержит всесторонние тесты для проверки функционирования связей, поэтому нет нужды дублировать эти тесты. Если нужно, вы можете просмотреть [тесты Ember Data](https://github.com/emberjs/data/tree/master/tests) в качестве примеров более глубокого тестирования связей.*