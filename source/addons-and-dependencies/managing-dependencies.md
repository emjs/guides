Когда вы разрабатываете приложение Ember, вы будете сталкиваться с распространенными сценариями, которые не относятся к самому Ember, например, аутентификация или использование SASS для вашей таблицы стилей. Чтобы решить эти проблемы, Ember CLI предоставляет общий формат под названием [Дополнения Ember](http://emjs.ru/v2/addons-and-dependencies/managing-dependencies/#toc_addons) для распространения повторно используемых библиотек. Кроме того, вы могли бы использовать front-end зависимости вроде фреймворка CSS или элемента выбора даты JavaScript, которые не специфичны для приложений Ember. Ember CLI поддерживает установку этих пакетов через стандартный [диспетчер пакетов Bower](http://guides.emberjs.com/v2.2.0/addons-and-dependencies/managing-dependencies/#toc_bower).

## Дополнения

Дополнения Ember устанавливаются с помощью NPM (например, `npm install --save-dev ember-cli-sass`). Дополнения могут вносить другие зависимости, автоматически изменяя файл `bower.json` в вашем проекте.

Вы можете найти списки дополнений в [Ember Observer](http://emberobserver.com/).

## Bower

Ember CLI использует диспетчер пакетов [Bower](http://bower.io/), что упрощает поддержание актуальности front-end зависимостей. Файл конфигурации Bower `bower.json` расположен в корне проекта Ember CLI и содержит перечень зависимостей проекта. Выполнение `bower install` в один шаг установит все перечисленные в `bower.json` зависимости.

Ember CLI отслеживает изменения `bower.json`. Таким образом, перезагружает приложение, если вы устанавливаете новые зависимости через `bower install <dependencies> --save`.

## Другие ресурсы 
Ресурсы, недоступные в качестве дополнения или пакета Bower, следует помещать в папку `vendor` вашего проекта.

## Компиляция ресурсов

Когда вы используете зависимости, которые не включены в дополнение, вы будете должны сообщить Ember CLI, чтобы она включила ваши ресурсы в сборку. Это делается с помощью файла манифеста ресурса `ember-cli-build.js`. Вам нужно лишь попробовать импортировать ресурсы, расположенные в папках `bower_components` и `vendor`.

### Глобальные объекты, предоставленные ресурсами Javascript

Глобальные объекты, предоставленные некоторыми ресурсами (вроде `moment` в примере ниже), могут использоваться в приложении без необходимости их импортировать. Предоставьте путь ресурса в качестве первого и единственного аргумента.

`ember-cli-build.js`
```js
app.import('bower_components/moment/moment.js');
```

Вам будет нужно добавить `"moment": true` к секции `predef` в `.jshintrc`, чтобы не допустить ошибок JSHint, связанных с использованием неопределенных переменных.

### Модули Javascript AMD

Предоставьте путь ресурса в качестве первого аргумента и список модулей и экспорта в качестве второго.

`ember-cli-build.js`
```js
app.import('bower_components/ic-ajax/dist/named-amd/main.js', {
  exports: {
    'ic-ajax': [
      'default',
      'defineFixture',
      'lookupFixture',
      'raw',
      'request'
    ]
  }
});
```

Теперь вы можете импортировать их в приложение. Например, `import { raw as icAjaxRaw } from 'ic-ajax';`.

### Среда специфических ресурсов

Если вам нужно использовать разные ресурсы в разных средах, укажите объект в качестве первого параметра. Этот ключ объекта должен быть именем среды, а значение — ресурсом для использования в этой среде.

`ember-cli-build.js`
```js
app.import({
  development: 'bower_components/ember/ember.js',
  production:  'bower_components/ember/ember.prod.js'
});
```

Если вам нужно импортировать ресурс только в одну среду, вы можете заключить `app.import` в оператор `if`. Для ресурсов, которые необходимы во время тестирования, вам также следует использовать опцию `{type: 'test'}`. Это позволит убедиться, что они будут доступны в режиме тестирования. 

`ember-cli-build.js`
```js
if (app.env === 'development') {
  // Only import when in development mode
  app.import('vendor/ember-renderspeed/ember-renderspeed.js');
}
if (app.env === 'test') {
  // Only import in test mode and place in test-supoprt.js
  app.import(app.bowerDirectory + '/sinonjs/sinon.js', { type: 'test' });
  app.import(app.bowerDirectory + '/sinon-qunit/lib/sinon-qunit.js', { type: 'test' });
}
```

### CSS

Предоставьте путь ресурса в качестве первого аргумента:

`ember-cli-build.js`
```js
app.import('bower_components/foundation/css/foundation.css');
```

Все стили ресурсов, добавленные таким образом, будут объединены и выведены как `/assets/vendor.css`.

### Другие ресурсы

Все прочие ресурсы вроде изображений или шрифтов могут быть добавлены через `import()`. По умолчанию, они будут скопированы в `dist/`, как есть.

`ember-cli-build.js`
```js
app.import('bower_components/font-awesome/fonts/fontawesome-webfont.ttf');
```

Этот пример создаст файл шрифтов в `dist/font-awesome/fonts/fontawesome-webfont.ttf`.

Вы также можете опционально указать, чтобы `import()` разместил файл по другому пути. Следующий пример скопирует файл в `dist/assets/fontawesome-webfont.ttf`.

`ember-cli-build.js`
```js
app.import('bower_components/font-awesome/fonts/fontawesome-webfont.ttf', {
  destDir: 'assets'
});
```

Если вам нужно загрузить определенные зависимости раньше других, вы можете установить свойство `prepend`, эквивалентное `true`, как второй аргумент `import()`. Это поставит зависимость в начало в файле сторонних ресурсов вместо простого добавления, которое выполняется по умолчанию.

`ember-cli-build.js`
```js
app.import('bower_components/es5-shim/es5-shim.js', {
  type: 'vendor',
  prepend: true
});
```