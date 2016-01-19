Приложения Ember используют шаблон разработки [«введение зависимости»](https://en.wikipedia.org/wiki/Dependency_injection) (DI), чтобы объявлять и создавать экземпляры классов объектов, а также зависимости между ними. Приложения и их экземпляры исполняют свою роль в реализации DI в Ember.

`Ember.Application` служит в качестве «системного реестра» для объявлений зависимости. Фабрики (то есть классы) регистрируются приложением, как и правила «введения» зависимостей. Правила применяются, когда объекты реализованы.

`Ember.ApplicationInstance` служит в качестве «контейнера» для объектов, которые реализованы из зарегистрированных фабрик. Экземпляры приложения предоставляют способы «находить» (то есть реализовывать и/или извлекать) объекты.

*Примечание: хотя `Application` служит как исходный системный реестр для приложения, каждый `ApplicationInstance` также может быть реестром. Регистрирование на уровне экземпляра полезно для обеспечения пользовательских правок на уровне экземпляра, таких как A/B тестирование функции.*

## Регистрирование фабрик

Фабрика может представлять любую часть приложения вроде *маршрута*, *шаблона* или пользовательского класса. Каждая фабрика регистрируется со специфическим ключом. Например, шаблон индекса регистрируется с ключом `template:index`, а приложение маршрута регистрируется с ключом `route:application`.

Ключи регистрации имеют два сегмента, разделенных двоеточием (`:`). Первый сегмент — тип фабрики фреймворка, второй — имя конкретной фабрики. Поэтому шаблон `index` имеет ключ `template:index`. Ember имеет несколько встроенных типов фабрик, таких как `service`, `route`, `template` и `component`.

Вы можете создать собственный тип фабрики, если просто зарегистрируете фабрику с новым типом. Например, чтобы создать тип `user`, вам нужно зарегистрировать фабрику с `application.register('user:user-to-register')`.

Регистрации фабрики должны выполняться в приложении или инициализаторах экземпляра приложения (которые имеют много общего с первым).

Например, инициализатор приложения может зарегистрировать фабрику `Logger` с ключом `logger:main`:

`app/initializers/logger.js`
```js
export function initialize(application) {
  var Logger = Ember.Object.extend({
    log(m) {
      console.log(m);
    }
  });

  application.register('logger:main', Logger);
}

export default {
  name: 'logger',
  initialize: initialize
};
```

### Регистрирование уже реализованных объектов

По умолчанию, Ember попытается реализовать зарегистрированную фабрику, когда отыщет ее. При регистрировании уже реализованного объекта вместо класса, используйте опцию `instantiate: false`, чтобы избежать попыток его повторной реализации во время поисков.

В следующем примере `logger` — простой объект JavaScript, который при нахождении должен быть возвращен «как есть»:

`app/initializers/logger.js`
```js
export function initialize(application) {
  var logger = {
    log(m) {
      console.log(m);
    }
  };

  application.register('logger:main', logger, { instantiate: false });
}

export default {
  name: 'logger',
  initialize: initialize
};
```

### Регистрирование синглтонов и несинглтонов

По умолчанию, регистрации рассматриваются как «синглтоны». Это значит, что экземпляр будет создан при первом нахождении, и этот же экземпляр будет сохранен в кэш и возвращен из последующих поисков.

Если вы хотите создать новые объекты для каждого поиска, регистрируйте ваши фабрики как несинглтоны с помощью опции `singleton: false`.

В следующем примере класс `Message` зарегистрирован как несинглтон:

`app/initializers/logger.js`
```js
export function initialize(application) {
  var Message = Ember.Object.extend({
    text: ''
  });

  application.register('notification:message', Message, { singleton: false });
}

export default {
  name: 'logger',
  initialize: initialize
};
```

## Введение фабрики

Если фабрика зарегистрирована, ее можно «ввести», когда это необходимо.

Можно вводить фабрики всех «типов» с помощью *введения типа*. Например:

`app/initializers/logger.js`
```js
export function initialize(application) {
  var Logger = Ember.Object.extend({
    log(m) {
      console.log(m);
    }
  });

  application.register('logger:main', Logger);
  application.inject('route', 'logger', 'logger:main');
}

export default {
  name: 'logger',
  initialize: initialize
};
```

В результате этого введения типа все фабрики типа `route` будут реализованы с введенным свойством `logger`. Значение `logger` будет исходить из фабрики под именем `logger:main`.

Маршруты в этом примере приложения теперь могут получить доступ к введенному регистратору:

`app/routes/index.js`
```js
export default Ember.Route.extend({
  activate() {
    // The logger property is injected into all routes
    this.get('logger').log('Entered the index route!');
  }
});
```

Введения также можно произвести для специфической фабрики с помощью ее полного ключа:

```js
application.inject('route:index', 'logger', 'logger:main');
```

В этом случае регистратор будет введен только на маршруте индекса.

Введения можно сделать и для любого класса, который требует реализации. Сюда можно включить все основные классы фреймворка Ember, такие как компоненты, помощники, маршруты и роутер.

### Специализированные введения

Введения зависимости также можно объявить напрямую в классах Ember с помощью `Ember.inject`. Сейчас `Ember.inject` поддерживает введение контроллеров (через `Ember.inject.controller`) и сервисов (через `Ember.inject.service`).

Следующий код вводит сервис `shopping-cart` в компонент `cart-contents` в качестве свойства `cart`:

`app/components/cart-contents.js`
```js
export default Ember.Component.extend({
  cart: Ember.inject.service('shopping-cart')
});
```

Если вы хотите ввести сервис с тем же именем в качестве свойства, просто оставьте имя сервиса (будет использоваться версия имени с тире):

`app/components/cart-contents.js`
```js
export default Ember.Component.extend({
  shoppingCart: Ember.inject.service()
});
```

## Поиски фабрики

Значительная часть регистраций и поисков Ember выполняется неявным образом.

В редких случаях, когда вы хотите выполнить точный поиск зарегистрированной фабрики, можно сделать это на экземпляре приложения в инициализаторе, который связан с экземпляром. Например:

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