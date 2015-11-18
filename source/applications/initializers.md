Инициализаторы предоставляют возможность сконфигурировать приложение при загрузке.

Есть два типа инициализаторов: инициализаторы приложения и инициализаторы экземпляра приложения.

Инициализаторы приложения запускаются при загрузке приложения и обеспечивают исходные способы для настройки [введений зависимости](http://guides.emberjs.com/v2.1.0/applications/initializers/dependency-injection) в приложении.

Инициализаторы экземпляра приложения запускаются, когда загружается экземпляр приложения. Они предоставляют способ настроить исходное состояние приложения и установить введения зависимости, которые являются локальными для экземпляра приложения (например, настройки A/B тестирования).

Выполняемые в инициализаторах операции должны быть упрощены, насколько это возможно, чтобы минимизировать задержки в загрузке приложения. Хотя существуют продвинутые техники для допущения асинхронности в инициализаторах приложения (то есть `deferReadiness` и `advanceReadiness`), этих техник стоит избегать. Любые асинхронные состояния загрузки (например, авторизация пользователя) почти всегда лучше обрабатываются в hooks маршрута приложения. Hooks допускают взаимодействие с DOM, пока состояния разрешаются.

## Инициализаторы приложения

Инициализаторы приложения можно создать через генератор Ember CLI `initializer`:

```bash
ember generate initializer shopping-cart
```

Давайте настроим инициализатор `shopping-cart`, чтобы ввести свойство `cart` во все маршруты приложения:

`app/initializers/shopping-cart.js`
```js
export function initialize(application) {
  application.inject('route', 'cart', 'service:shopping-cart');
};

export default {
  name: 'shopping-cart',
  initialize: initialize
};
```

## Инициализаторы экземпляра приложения

Инициализаторы экземпляра приложения можно создать через генератор Ember CLI `instance-initializer`:

```bash
ember generate instance-initializer logger
```

Давайте добавим простую регистрацию, чтобы отметить, что экземпляр загружен:

`app/instance-initializers/logger.js`
```js
export function initialize(applicationInstance) {
  var logger = applicationInstance.lookup('logger:main');
  logger.log('Hello from the instance initializer!');
}

export default {
  name: 'logger',
  initialize: initialize
};
```

## Указание порядка инициализаторов

Если вы хотите контролировать порядок, в котором инициализаторы запускаются, то можете использовать опции `before` и/или `after`:

`app/initializers/config-reader.js`
```js
export function initialize(application) {
  // ... your code ...
};

export default {
  name: 'configReader',
  before: 'websocketInit',
  initialize: initialize
};
```

`app/initializers/websocket-init.js`
```js
export function initialize(application) {
  // ... your code ...
};

export default {
  name: 'websocketInit',
  after: 'configReader',
  initialize: initialize
};
```

Учтите, что порядок применяется только к инициализаторам одного типа (то есть приложения или экземпляра приложения). Инициализаторы приложения всегда будут запускаться до инициализаторов экземпляра приложения.