Чтобы продемонстрировать основную настройку и обработку приложения Ember, в этом разделе мы создадим приложение для сайта с арендой недвижимости. Назовем его Super Rentals. Изначально приложение будет состоять из главной страницы, а также страниц about (о компании) и contact (контакты). Прежде чем начать, взглянем на приложение с точки зрения пользователя.

![style super rentals maps](https://guides.emberjs.com/v2.8.0/images/service/style-super-rentals-maps.png)

Мы заходим на главную страницу и видим список сдаваемой в аренду недвижимости. Отсюда мы можем перейти на страницу about или contact.

Теперь начнем создавать страницу "about". Помните, что при загрузке URL-пути `/about` роутер будет сопоставлять URL с обработчиком маршрута по тому же имени, то есть *about.js*. Затем обработчик маршрута загружает шаблон.

## Маршрут about

Если мы запустим команду `ember help generate`, то сможем увидеть разнообразие инструментов Ember для автоматической генерации файлов под различные ресурсы Ember. Используем генератор маршрутов, чтобы создать маршрут `about`.

```
ember generate route about
```

или более коротко:

```
ember g route about
```

И мы увидим, какие действия выполнил генератор:

```
installing route
  create app/routes/about.js
  create app/templates/about.hbs
updating router
  add route about
installing route-test
  create tests/unit/routes/about-test.js
```

В результате создано три новых файла: один для обработчика маршрута, один для шаблона, который будет отображать обработчик маршрута, и тестовый файл. Четвертый файл, который здесь затрагивается, это роутер.

Если мы откроем роутер, то увидим, что генератор прописал в нем новый маршрут *about*.
Этот маршрут будет загружать обработчик маршрута `about`.

`app/router.js`
```js
import Ember from 'ember';
import config from './config/environment';

const Router = Ember.Router.extend({
  location: config.locationType,
  rootURL: config.rootURL
});

Router.map(function() {
  this.route('about');
});

export default Router;
```

По умолчанию, обработчик маршрута `about` загружает шаблон `about.hbs`. То есть нам не нужно ничего менять в новом файле `app/routes/about.js` для шаблона `about.hbs`, чтобы отобразить то, что мы хотим.

Обо всей маршрутизации позаботился генератор, и мы можем приступить к программированию шаблона.
На страницу `about` мы добавим код HTML, в котором содержится немного информации о сайте:

`app/templates/about.hbs`
```hbs
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>About Super Rentals</h2>
  <p>
    The Super Rentals website is a delightful project created to explore Ember.
    By building a property rental site, we can simultaneously imagine traveling
    AND building Ember applications.
  </p>
</div>
```

Выполните команду `ember serve` (или короткий вариант `ember s`) из оболочки, чтобы запустить сервер разработки Ember, затем пройдите по адресу [`http://localhost:4200/about`](http://localhost:4200/about), и вы увидите новое приложение в действии.

## Маршрут contact

Создадим еще один маршрут с информацией о контактах компании. И снова мы начнем с генерации маршрута, обработчика маршрута и шаблона.

```
ember g route contact
```

Как мы видим, генератор создал маршрут `contact` в файле `app/router.js` и соответствующий обработчик маршрута в `app/routes/contact.js`. Так как мы будем использовать шаблон `contact`, в маршрут `contact` не нужно вносить дополнительных изменений.

В `contact.hbs` мы можем добавить информацию для связи с главным офисом Super Rentals:

`app/templates/contact.hbs`
```hbs
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Contact Us</h2>
  <p>Super Rentals Representatives would love to help you<br>choose a destination or answer
    any questions you may have.</p>
  <p>
    Super Rentals HQ
    <address>
      1212 Test Address Avenue<br>
      Testington, OR 97233
    </address>
    <a href="tel:503.555.1212">+1 (503) 555-1212</a><br>
    <a href="mailto:superrentalsrep@emberjs.com">superrentalsrep@emberjs.com</a>
  </p>
</div>
```

Мы создали второй маршрут. Теперь, если мы перейдем по URL [`http://localhost:4200/contact`](http://localhost:4200/contact), то попадем на страницу с контактами.

## Навигация с помощью хелпера {{link-to}} и ссылок

Нам не нужно, чтобы для перехода по страницам пользователи были вынуждены вписывать адреса URL. Поэтому внизу каждой страницы мы добавим ссылки. Сделаем ссылку contact на странице about и ссылку about на странице contact.

В Ember есть встроенные **хелперы**, которые обеспечивают определенную функциональность, например, установку ссылок на другие маршруты. Здесь мы используем в коде хелпер `{{link-to}}`, чтобы установить ссылку между маршрутами:

`app/templates/about.hbs`
```hbs
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>About Super Rentals</h2>
  <p>
    The Super Rentals website is a delightful project created to explore Ember.
    By building a property rental site, we can simultaneously imagine traveling
    AND building Ember applications.
  </p>
  {{#link-to 'contact' class="button"}}
    Get Started!
  {{/link-to}}
</div>
```

Хелпер `{{link-to}}` берет аргумент с именем маршрута, на который нужно сделать ссылку. В нашем случае: contact. Теперь, когда мы перейдем на страницу about по адресу [`http://localhost:4200/about`](http://localhost:4200/about), мы увидим рабочую ссылку на страницу contact.

![ember super rentals about](https://guides.emberjs.com/v2.8.0/images/routes-and-templates/ember-super-rentals-about.png)

Далее, мы добавим ссылку на страницу contact, чтобы свободно перемещаться между разделами `about` и `contact`.

`app/templates/contact.hbs`
```hbs
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Contact Us</h2>
  <p>Super Rentals Representatives would love to help you<br>choose a destination or answer
    any questions you may have.</p>
  <p>
    Super Rentals HQ
    <address>
      1212 Test Address Avenue<br>
      Testington, OR 97233
    </address>
    <a href="tel:503.555.1212">+1 (503) 555-1212</a><br>
    <a href="mailto:superrentalsrep@emberjs.com">superrentalsrep@emberjs.com</a>
  </p>
  {{#link-to 'about' class="button"}}
    About
  {{/link-to}}
</div>
```

## Маршрут rentals

Нам нужно, чтобы приложение отображало список недвижимости, который пользователи могут просматривать. Чтобы сделать это, мы добавим третий маршрут и назовем его `rentals`.

```
ember g route rentals
```

Обновим сгенерированный `rentals.hbs` базовой разметкой, чтобы создать фундамент страницы со списком недвижимости. Позднее вы вернемся к этой странице, чтобы добавить параметры недвижимости.

`app/templates/rentals.hbs`
```hbs
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Welcome!</h2>
  <p>We hope you find exactly what you're looking for in a place to stay.</p>
  {{#link-to 'about' class="button"}}
    About Us
  {{/link-to}}
</div>
```

## Маршрут index

У нас есть две статичные страницы. Теперь мы готовы добавить главную страницу, которая будет приветствовать пользователей на сайте. На данный момент наша главная страница в приложении — страница rentals, для которой мы уже создали маршрут. Поэтому нам нужно, чтобы маршрут index просто перенаправлял на уже созданный маршрут `rentals`.

С помощью того же процесса, что мы делали ранее для страниц about и contact, мы генерируем новый маршрут `index`.

```
ember g route index
```
Мы видим знакомый результат работы генератора маршрутов:

```
installing route
  create app/routes/index.js
  create app/templates/index.hbs
installing route-test
  create tests/unit/routes/index-test.js
```
  
В отличие от других обработчиков маршрутов, что мы создавали ранее, у маршрута `index` есть свои особенности: для него НЕ требуется запись в роутере. В разделе о [вложенных маршрутах](https://guides.emberjs.com/v2.8.0/tutorial/subroutes/) мы подробнее рассмотрим, почему запись не требуется. 

Мы можем начать с реализации модульного теста для index. Так как мы хотим просто выполнить переход на `rentals`, наш модульный тест будет проверять, чтобы метод [`replaceWith`](http://emberjs.com/api/classes/Ember.Route.html#method_replaceWith) вызывался с нужным маршрутом. `replaceWith` похож на функцию `transitionTo`. Разница в том, что `replaceWith` заменит текущий URL в истории браузера, а `transitionTo` добавит его в историю. Нам нужно, чтобы маршрут `rentals` служил домашней страницей. Поэтому мы используем функцию `replaceWith`. Чтобы проверить это, мы подставим заглушку метода `replaceWith` для маршрута и утвердим, что при его вызове передается маршрут `rentals`.

`tests/unit/routes/index-test.js`
```js
import { moduleFor, test } from 'ember-qunit';

moduleFor('route:index', 'Unit | Route | index');

test('should transition to rentals route', function(assert) {
  let route = this.subject({
    replaceWith(routeName) {
      assert.equal(routeName, 'rentals', 'replace with route name rentals');
    }
  });
  route.beforeModel();
});
```

В маршрут index мы просто добавим вызов `replaceWith`.

`app/routes/index.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  beforeModel() {
    this._super(...arguments);
    this.replaceWith('rentals');
  }
});
```

Теперь при посещении корневого маршрута `/`, будет загружаться URL `/rentals`.

## Добавление «шапки» с навигацией

В каждом маршруте нашего приложения мы создали ссылки в стиле кнопок. Теперь мы добавим общую «шапку», чтобы отобразить название нашего приложения и его основные страницы.

Сначала наберите команду `ember g template application`, чтобы создать шаблон приложения.

```
installing template
  create app/templates/application.hbs
```

Все, что вы указываете в шаблоне `application.hbs`, будет показано на каждой странице приложения.
Теперь добавьте такую разметку навигации «шапки»:

`app/templates/application.hbs`
```hbs
<div class="container">
  <div class="menu">
    {{#link-to 'index'}}
      <h1 class="left">
        <em>SuperRentals</em>
      </h1>
    {{/link-to}}
    <div class="left links">
      {{#link-to 'about'}}
        About
      {{/link-to}}
      {{#link-to 'contact'}}
        Contact
      {{/link-to}}
    </div>
  </div>
  <div class="body">
    {{outlet}}
  </div>
</div>
```

Обратите внимание на включение заполнителя `{{outlet}}` в элемент `div` с классом body. Заполнитель [`{{outlet}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_outlet) подчиняется роутеру, который будет отображать на его месте разметку текущего маршрута. Это значит, что разные маршруты, которые мы создавали для приложения, будут отображаться там.

Теперь, когда мы добавили маршруты и связи между ними, три приемочных теста, которые мы создали для проверки навигации по нашим маршрутам, будут успешно пройдены:

![passing navigation tests](https://guides.emberjs.com/v2.8.0/images/routes-and-templates/passing-navigation-tests.png)