В отличие от большинства других фреймворков, в которых реализованы своего рода привязки, в Ember.js привязки можно использовать с любым объектом. Тем не менее привязки чаще всего используют в рамках самого фреймворка Ember. C большинством проблем в приложениях Ember разработчики справляются с помощью вычислительных свойств.

Самый простой способ создать двустороннюю привязку — использовать `computed.alias()`, который определяет путь к другому объекту.

```javascript
wife = Ember.Object.create({
  householdIncome: 80000
});

Husband = Ember.Object.extend({
  householdIncome: Ember.computed.alias('wife.householdIncome')
});

husband = Husband.create({
  wife: wife
});

husband.get('householdIncome'); // 80000

// Someone gets raise.
wife.set('householdIncome', 90000);
husband.get('householdIncome'); // 90000
```

Обратите внимание, что привязки не обновляются сразу. Ember ожидает, когда весь код вашего приложения будет выполнен, а затем синхронизирует изменения. Поэтому вы можете менять связанное свойство столько раз, сколько хотите, и не беспокоиться об издержках синхронизации привязок при временных значениях.

### Однонаправленные привязки

Однонаправленная привязка передает изменения только в одном направлении с помощью `computed.oneWay()`. Чаще всего однонаправленные привязки нужны для оптимизации производительности. Но вы без проблем можете использовать и двунаправленную привязку (двунаправленные привязки служат однонаправленными, если вы производите изменения только на одной стороне). Иногда лучше использовать однонаправленные привязки, чтобы получить особое поведение. Скажем, значение по умолчанию, которое соответствует другому свойству, но которое можно переопределить (например, адрес поставки начинается так же, как адрес выставления счета, но позднее его можно изменить).

```javascript
user = Ember.Object.create({
  fullName: 'Kara Gates'
});

UserComponent = Ember.Component.extend({
  userName: Ember.computed.oneWay('user.fullName')
});

userComponent = UserComponent.create({
  user: user
});

// Changing the name of the user object changes
// the value on the view.
user.set('fullName', 'Krang Gates');
// userComponent.userName will become "Krang Gates"

// ...but changes to the view don't make it back to
// the object.
userComponent.set('userName', 'Truckasaurus Gates');
user.get('fullName'); // "Krang Gates"
```