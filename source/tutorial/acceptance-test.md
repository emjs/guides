Давайте продумаем, что должно быть на главной странице нашего приложения Super Rentals.

Нам нужно, чтобы там были:

* список доступной недвижимости, сдаваемой в аренду;
* ссылка на информацию о компании;
* ссылка на контактную информацию;
* фильтр списка недвижимости по городу.

Мы можем представить эти цели как [приемочные тесты Ember](https://guides.emberjs.com/v2.8.0/testing/acceptance/). Приемочные тесты взаимодействуют с приложением как реальный пользователь, но их можно автоматизировать и таким образом гарантировать, что работа приложения не нарушится в будущем.

Сначала используем Ember CLI, чтобы сгенерировать новый приемочный тест:
```
ember g acceptance-test list-rentals
```

После запуска команды вы получите результат ниже. Здесь показано, что создан один файл под названием `list-rentals-test`.

```
installing acceptance-test
  create tests/acceptance/list-rentals-test.js
```

Если открыть новый файл теста, то будет выполнен некоторый стереотипный код, который попытается перейти по маршруту `list-rentals` и подтвердить, что этот маршрут загружен. Этот стереотипный код представлен здесь, чтобы познакомить вас с первым рабочим приемочным тестом. Так как мы тестируем маршрут index, который представлен `/`, мы заменим случаи употребления `/list-rentals` на `/`.

`/tests/acceptance/list-rentals-test.js`
```js
import { test } from 'qunit';
import moduleForAcceptance from 'super-rentals/tests/helpers/module-for-acceptance';

moduleForAcceptance('Acceptance | list-rentals');

test('visiting /', function(assert) {
  visit('/');

  andThen(function() {
    assert.equal(currentURL(), '/');
  });
});
```

Теперь в новом окне запустите набор тестов из командной строки с помощью `ember test --server`, и вы увидите успешно пройденный приемочный тест (вместе с группой тестов JSHint).

Как упоминалось ранее, этот шаблон теста необходим просто для проверки среды, поэтому теперь заменим этот тест на наш список целей.

`/tests/acceptance/list-rentals-test.js`
```js
import { test } from 'qunit';
import moduleForAcceptance from 'super-rentals/tests/helpers/module-for-acceptance';

moduleForAcceptance('Acceptance | list-rentals');

test('should redirect to rentals route', function (assert) {
});

test('should list available rentals.', function (assert) {
});

test('should link to information about the company.', function (assert) {
});

test('should link to contact information.', function (assert) {
});

test('should filter the list of rentals by city.', function (assert) {
});

test('should show details for a specific rental', function (assert) {
});
```

Эти тесты Ember провалятся, так как мы не проверяем ничего (что называется `утверждением`). У нас есть представление того, как должно выглядеть наше приложение, поэтому мы можем добавить некоторые подробности в тесты.

В Ember есть хелперы тестов, которые можно использовать, чтобы выполнять распространенные задачи в приемочных тестах, например, посещать маршруты, заполнять поля, нажимать элементы и ожидать отображения страниц.

Нам нужно, чтобы основное внимание на сайте уделялось выбору недвижимости, поэтому мы планируем направить трафик c корневого URL `/` на маршрут `rentals`. С помощью хелперов мы можем создать простой тест, чтобы это проверить:

`/tests/acceptance/list-rentals-test.js`
```js
test('should redirect to rentals route', function (assert) {
  visit('/');
  andThen(function() {
    assert.equal(currentURL(), '/rentals', 'should redirect automatically');
  });
});
```

В этом тесте действуют несколько хелперов:

* Хелпер [`visit`](http://emberjs.com/api/classes/Ember.Test.html#method_visit) загружает маршрут, указанный для данного URL.
* Хелпер [`andThen`](https://guides.emberjs.com/v2.8.0/testing/acceptance/#toc_wait-helpers) ожидает, пока все ранее вызванные хелперы тестов завершат работу, чтобы выполнить назначенную вами функцию. В нашем случае нам нужно подождать, пока страница загрузится после `visit`. Тогда мы можем подтвердить, что позиции с недвижимостью отображены.
* [`currentURL`](http://emberjs.com/api/classes/Ember.Test.html#method_currentURL) возвращает URL, который сейчас посещает тест приложения.

Чтобы протестировать список недвижимости, мы сначала посетим маршрут index и проверим, чтобы в результатах оказалось 3 позиции:

`/tests/acceptance/list-rentals-test.js`
```js
test('should list available rentals.', function (assert) {
  visit('/');
  andThen(function () {
    assert.equal(find('.listing').length, 3, 'should see 3 listings');
  });
});
```

В тесте предполагается, что каждый элемент недвижимости будет иметь класс `listing`.

В следующих двух тестах нам нужно подтвердить, что по нажатию ссылок «О компании» и «Контакты» успешно загружаются правильные URL. Мы будем использовать хелпер [`click`](http://emberjs.com/api/classes/Ember.Test.html#method_click), чтобы имитировать нажатие ссылок. После загрузки нового экрана мы просто проверим, что новый URL соответствует нашим ожиданиям с помощью хелпера [`currentURL`](http://emberjs.com/api/classes/Ember.Test.html#method_click).

`/tests/acceptance/list-rentals-test.js`
```js
test('should link to information about the company.', function (assert) {
  visit('/');
  click('a:contains("About")');
  andThen(function () {
    assert.equal(currentURL(), '/about', 'should navigate to about');
  });
});

test('should link to contact information', function (assert) {
  visit('/');
  click('a:contains("Contact")');
  andThen(function () {
    assert.equal(currentURL(), '/contact', 'should navigate to contact');
  });
});
```

Учтите, что мы можем вызывать два [асинхронных хелпера тестов](https://guides.emberjs.com/v2.8.0/testing/acceptance/#toc_asynchronous-helpers) подряд без необходимости использовать `andThen` или обещание. Каждый асинхронный хелпер теста создан таким образом, что он будет ждать, пока другие хелперы выполнят свою задачу.

После тестирования URL мы подробно рассмотрим главную страницу, чтобы проверить фильтрацию списка по городу в качестве поискового критерия. Представим, что у нас есть поле ввода в контейнере с классом `list-filter`. Мы введем в поле слово «Сиэтл» в качестве поискового параметра и отправим событие, привязанное к клавише, чтобы запустить фильтрацию. Так как мы управляем своими данными, нам известно, что есть только одна позиция недвижимости с городом «Сиэтл». Поэтому мы утверждаем, что количество позиций равно 1 и что недвижимость находится в Сиэтле.

`/tests/acceptance/list-rentals-test.js`
```js
test('should filter the list of rentals by city.', function (assert) {
  visit('/');
  fillIn('.list-filter input', 'seattle');
  keyEvent('.list-filter input', 'keyup', 69);
  andThen(function () {
    assert.equal(find('.listing').length, 1, 'should show 1 listing');
    assert.equal(find('.listing .location:contains("Seattle")').length, 1, 'should contain 1 listing with location Seattle');
  });
});
```

Наконец, нам нужно проверить, что можно нажать конкретную недвижимость и загрузить детальное представление. Мы нажмем название и проверим, что отображается расширенное описание недвижимости.

`/tests/acceptance/list-rentals-test.js`
```js
test('should show details for a specific rental', function (assert) {
  visit('/rentals');
  click('a:contains("Grand Old Mansion")');
  andThen(function() {
    assert.equal(currentURL(), '/rentals/grand-old-mansion', "should navigate to show route");
    assert.equal(find('.show-listing h2').text(), "Grand Old Mansion", 'should list rental title');
    assert.equal(find('.description').length, 1, 'should list a description of the property');
  });
});
```

Так как мы еще не реализовали эту функциональность, тесты провалятся. В результатах вы должны увидеть все провалившиеся тесты, которые послужат пунктами в дальнейших планах изучения.

![failed acceptance tests](https://guides.emberjs.com/v2.8.0/images/acceptance-test/failed-acceptance-tests.png)

По мере прохождения руководства, мы будем использовать приемочные тесты как проверочный список функциональности. Если все пункты зеленые, значит, мы достигли главных целей!