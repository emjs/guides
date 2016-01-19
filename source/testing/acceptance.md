Чтобы создать приемочный тест, запустите `ember generate acceptance-test <name>`. Например:

```bash
ember g acceptance-test login
```

Так мы сгенерируем следующий файл:

`tests/acceptance/login-test.js`
```js
import Ember from 'ember';
import { module, test } from 'qunit';
import startApp from 'testing-guide-examples/tests/helpers/start-app';

let application;

module('Acceptance | login', {
  beforeEach() {
    application = startApp();
  },

  afterEach() {
    Ember.run(application, 'destroy');
  }
});

test('visiting /login', function(assert) {
  visit('/login');
  andThen(() => assert.equal(currentURL(), '/login'));
});
```

Большая часть из этого — стандартный текст для установок теста и освобождения ресурсов. Последние несколько строк в пределах функции `test` содержат пример теста.

Почти каждый тест имеет паттерн посещения маршрута, взаимодействия со страницей (с использованием помощников) и проверки ожидаемых изменений в DOM.

Например:

`tests/acceptance/new-post-appears-first-test.js`
```js
test('should add new post', function(assert) {
  visit('/posts/new');
  fillIn('input.title', 'My new post');
  click('button.submit');
  andThen(() => assert.equal(find('ul.posts li:first').text(), 'My new post'));
});
```

## Помощники тестирования

Одна из основных проблем в тестировании веб-приложений заключается в том, что весь код — событийно-управляемый, и поэтому он может быть асинхронным (то есть вывод будет происходить вне последовательности относительно ввода). В результате этот код может выполняться в любом порядке.

Здесь поможет пример: скажем, пользователь нажимает по очереди на две кнопки, и обе загружают данные с различных серверов. Им нужно разное время для ответа.

При написании тестов вам необходимо хорошо осознавать один факт: вы не можете быть уверены, что после запросов ответ вернется немедленно. Поэтому код утверждения («тестер») должен ждать, пока проверяемый объект («тестируемый») перейдет в синхронизированное состояние. В примере выше так и произошло бы, когда оба сервера ответили, и код теста смог приступить к проверке данных (будь то фиктивные данные или реальные).

Поэтому все помощники тестирования в Ember заключаются в код. Это гарантирует, что Ember вернется в синхронизированное состояние, когда будет проверять свои утверждения. Еще это избавляет от необходимости писать этот код, и упрощает чтение тестов, так как они содержат меньше текста.

В Ember есть несколько помощников для содействия приемочным тестам. Они делятся на два типа: **асинхронные** и **синхронные**.

## Асинхронные помощники

Асинхронные помощники «знают» (и выжидают) об асинхронном поведении в приложении, что упрощает написание детерминированных тестов.

Также эти помощники регистрируются в том порядке, а котором вы их вызываете. Они будут запускаться по цепочке: каждый вызывается, только когда предыдущий завершится. Вы можете быть уверены, что порядок их вызова будет соответствовать порядку их выполнения, и предыдущий помощник завершится прежде, чем запустится следующий.

* `click(selector)`
 - Щелкает по элементу и запускает любые действия, охваченные событием `click` в элементе, возвращает обещание, которое выполняется при завершении всего полученного в результате асинхронного поведения.
* `fillIn(selector, value)`
 - Вставляет в выбранное поле ввода определенное значение и возвращает обещание. Оно выполняется по завершении всего полученного в результате асинхронного поведения. Работает с элементами `<select>` и `<input>`. Помните, что с элементами `<select>` `value` должно быть установлено к значению тега `<option>`, а не его контенту (например, `true`, а не `"Yes"`).
* `keyEvent(selector, type, keyCode)`
 - Имитирует тип клавиатурного события (например, `keypress`, `keydown`, `keyup`) с нужным `keyCode` на элементе, который находит селектор.
* `triggerEvent(selector, type, options)`
 - Запускает определенное событие (например, `blur`, `dblclick`) на элементе, который идентифицируется предоставленным селектором.
* `visit(url)`
 - Посещает определенный маршрут и возвращает обещание, которое выполняется по завершении всего полученного в результате асинхронного поведения.

## Синхронные помощники

Синхронные помощники выполняются сразу при запуске.

* `currentPath()`
 - Возвращает текущий путь.
* `currentRouteName()`
 - Возвращает имя текущего активного маршрута.
* `currentURL()`
 - Возвращает текущий URL.
* `find(selector, context)`
 - Находит элемент в пределах корневого элемента приложения и контекста (опционально). Определение границ корневого элемента позволяет избежать конфликтов с генератором отчетов по тестам во фреймворке. Это делается по умолчанию, если не указан контекст.

## Помощники ожидания

Прежде чем пойти дальше, помощник `andThen` будет ждать завершения всех предыдущих асинхронных помощников. Давайте взглянем на следующий пример:

`tests/acceptance/new-post-appears-first-test.js`
```js
test('should add new post', function(assert) {
  visit('/posts/new');
  fillIn('input.title', 'My new post');
  click('button.submit');
  andThen(() => assert.equal(find('ul.posts li:first').text(), 'My new post'));
});
```

Сначала мы посещаем новые посты по URL «/posts/new», вводим текст «My new post» в элемент управления вводом с классом CSS «title» и щелкаем по кнопке с классом «submit».

Затем мы вызываем помощника `andThen`, который будет ждать завершения предыдущих асинхронных помощников тестирования (а именно `andThen` будет вызван только **после** того, как мы посетим новые посты, заполним текст и щелкнем по кнопке «submit», **и** браузер освободится от выполнения требуемых для этих задач действий). Обратите внимание, что `andThen` имеет один аргумент функции, который содержит код. Он будет выполнен после завершения других помощников.
 
В помощнике `andThen` мы, наконец, вызываем `assert.equal`. Он проверяет утверждение, что текст в первом элементе `li` списка `ul` с классом «posts» эквивалентен «My new post».

### Индивидуальные помощники тестирования

Для создания собственного помощника тестирования запустите `ember generate test-helper <helper-name>`. Ниже представлен результат запуска `ember g test-helper shouldHaveElementWithCount`:

`tests/helpers/should-have-element-with-count.js`
```js
export default Ember.Test.registerAsyncHelper(
    'shouldHaveElementWithCount', function(app) {
});
```

`Ember.Test.registerAsyncHelper` и `Ember.Test.registerHelper` используются, чтобы регистрировать помощников тестирования, которые будут вводиться при вызове `startApp`. Разница между `Ember.Test.registerHelper` и `Ember.Test.registerAsyncHelper` в том, что последний не запускается, пока не завершится любой предыдущий асинхронный помощник, и любой последующий асинхронный помощник до запуска будет ждать завершения `Ember.Test.registerAsyncHelper`.

Метод помощника всегда будет вызываться текущим приложением в качестве первого параметра. Другие параметры должны предоставляться при вызове помощника. Помощники должны быть зарегистрированы до вызова `startApp`. Но об этом позаботится ember-cli.

А вот пример не асинхронного помощника:

`tests/helpers/should-have-element-with-count.js`
```js
export default Ember.Test.registerHelper('shouldHaveElementWithCount', function(app, assert, selector, n, context) {
  const el = findWithAssert(selector, context);
  const count = el.length;
  assert.equal(n, count, `found ${count} times`);
});
// shouldHaveElementWithCount(assert, 'ul li', 3);
```

Здесь пример асинхронного помощника:

`tests/helpers/dblclick.js`
```js
export default Ember.Test.registerAsyncHelper('dblclick',
  function(app, assert, selector, context) {
    let $el = findWithAssert(selector, context);
    Ember.run(() => $el.dblclick());
  }
);

// dblclick('#person-1')
```

Асинхронные помощники также полезны, если вы хотите сгруппировать взаимодействия в одном помощнике. Например:

`tests/helpers/add-contact.js`
```js
export default Ember.Test.registerAsyncHelper('addContact',
  function(app, assert, name) {
    fillIn('#name', name);
    click('button.create');
  }
);

// addContact('Bob');
// addContact('Dan');
```

Наконец, не забудьте добавить помощников в `tests/.jshintrc` и `tests/helpers/start-app.js`. Чтобы тесты jshint не провалились, в `tests/.jshintrc` вам нужно добавить их в раздел `predef`:

`tests/.jshintc`
```js
{
  "predef": [
    "document",
    "window",
    "location",
    ...
    "shouldHaveElementWithCount",
    "dblclick",
    "addContact"
  ],
  ...
}
```

В `tests/helpers/start-app.js` вам нужно лишь импортировать файл помощника: тогда он будет зарегистрирован.

`tests/helpers/start-app.js`
```js
import Ember from 'ember';
import Application from '../../app';
import Router from '../../router';
import config from '../../config/environment';
import './should-have-element-with-count';
import './dblclick';
import './add-contact';
```