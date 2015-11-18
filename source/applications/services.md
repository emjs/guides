`Ember.Service` — долгоживущий объект Ember, который можно сделать доступным в разных частях приложения.

Примеры использования сервисов:

* Регистрация
* Аутентификация пользователя/сессии
* Геолокация
* Сторонние APIs
* Веб-сокеты
* События и уведомления с сервера
* Поддерживаемые сервером вызовы API, которые могут не соответствовать Ember Data

### Определение сервисов

Сервисы можно создать с помощью генератора Ember CLI `service`. Например, следующая команда создаст сервис `ShoppingCart`:

```bash
ember generate service shopping-cart
```

Сервисы должны расширять базовый класс `Ember.Service`:

`app/services/shopping-cart.js`
```js
export default Ember.Service.extend({
});
```

Как и любые объекты Ember, сервис инициализируется и может иметь собственные свойства и методы.

`app/services/shopping-cart.js`
```js
export default Ember.Service.extend({
  items: null,

  init() {
    this._super(...arguments);
    this.set('items', []);
  },

  add(item) {
    this.get('items').pushObject(item);
  },

  remove(item) {
    this.get('items').removeObject(item);
  },

  empty() {
    this.get('items').setObjects([]);
  }
});
```

### Доступ к сервисам

Чтобы получить доступ к сервису, введите его в инициализатор или с `Ember.inject`:

`app/components/cart-contents.js`
```js
export default Ember.Component.extend({
  cart: Ember.inject.service('shopping-cart')
});
```

Так мы вводим сервис корзины покупок в компонент и делаем его доступным в качестве свойства `cart`.

Затем вы можете получить доступ к свойствам и методам сервиса:

`app/components/cart-contents.js`
```js
export default Ember.Component.extend({
  cart: Ember.inject.service('shopping-cart'),

  actions: {
    remove(item) {
      this.get('cart').remove(item);
    }
  }
});
```

`app/templates/components/cart-contents.hbs`
```hbs
<ul>
  {{#each cart.items as |item|}}
    <li>
      {{item.name}}
      <button {{action "remove" item}}>Remove</button>
    </li>
  {{/each}}
</ul>
```

Введенное свойство — отложенное; сервис не реализуется, пока свойство вызвано явно. Оно сохранится, пока существует приложение.

Если для `service()` не предоставлено аргумента, Ember будет использовать версию имени свойства с тире:

`app/components/cart-contents.js`
```js
export default Ember.Component.extend({
  shoppingCart: Ember.inject.service()
});
```

Это также вводит сервис корзины покупок в качестве свойства `shoppingCart`.