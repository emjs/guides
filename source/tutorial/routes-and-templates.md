Чтобы продемонстрировать начальную установку и обработку приложения Ember, мы рассмотрим в этом разделе создание приложения для сайта с арендой недвижимости под названием Super Rentals. Мы начнем с домашней страницы, разделов about (о компании) и contact (контакты). Но прежде чем начать, давайте взглянем на приложение с точки зрения пользователя.

![super rentals homepage screenshot](/static/images/guides/routes-and-templates/ember-super-rentals-index.png)

Мы видим домашнюю страницу, на которой размещен список с различной недвижимостью. Отсюда мы можем перейти в раздел about или contact.

Для начала убедимся, что у нас создана новая программа под названием `super-rentals`. Для этого в Ember CLI наберите:

```bash
ember new super-rentals
```

Прежде чем начать создание трех страниц для приложения, мы удалим содержимое файла `app/templates/application.hbs` и оставим вместо него только `{{outlet}}`. Мы более подробно поговорим о роли файла `application.hbs`, когда у сайта будет несколько маршрутов.

Теперь начнем создание раздела "about". Помните, что когда загружается путь URL `/about`, роутер будет сопоставлять URL с обработчиком маршрута с тем же именем, то есть *about.js*. Затем обработчик маршрута загрузит шаблон.

## Маршрут About

Если мы запустим `ember help generate`, то сможем увидеть разнообразие инструментов, которые предоставляет Ember для автоматически генерируемых файлов для разных средств Ember. Давайте используем генератор маршрутов, чтобы создать маршрут `about`.

```shell
ember generate route about
```

А вот упрощенная версия:

```shell
ember g route about
```

И мы увидим, какие действия выполнил генератор:

```shell
installing route
  create app/routes/about.js
  create app/templates/about.hbs
updating router
  add route about
installing route-test
  create tests/unit/routes/about-test.js
```

В итоге создаются три новых файла: один для обработчика маршрута, один для шаблона, который отображается обработчиком маршрута, и тестовый файл. Четвертый файл, который здесь затрагивается, это роутер.

Если мы откроем роутер, то увидим, что генератор отобразил для нас новый маршрут *about*. Этот маршрут будет загружать обработчик маршрута `about`.

`app/router.js`
```js
import Ember from 'ember';
import config from './config/environment';

const Router = Ember.Router.extend({
  location: config.locationType
});

Router.map(function() {
  this.route('about');
});

export default Router;
```

По умолчанию, обработчик маршрута `about` загружает шаблон `about.hbs`. То есть нам не нужно ничего менять в новом файле `app/routes/about.js` для шаблона `about.hbs`, чтобы отобразить то, что мы хотим.

Со всей маршрутизацией, которую мы получили из генератора, мы можем приступить к программированию шаблона. Для страницы `about` мы добавим код HTML, который содержит немного информации о сайте:

`app/templates/about.hbs`
```hbs
<h2>About Super Rentals</h2>

<p>The Super Rentals website is a delightful project created to explore Ember.
By building a property rental site, we can simultaneously imagine traveling
AND building Ember applications.</p>
```

Выполните `ember serve` (или короткий вариант `ember s`) из оболочки, чтобы запустить сервер разработки Ember, и затем пройдите по адресу `localhost:4200`, чтобы увидеть новое приложение в действии!

## Маршрут Contact

Давайте создадим еще один маршрут с контактами компании. И снова мы начнем с генерации маршрута, обработчика маршрута и шаблона.

```shell
ember g route contact
```

Как мы видим, генератор создал маршрут `contact` в файле `app/router.js` и соответствующий обработчик маршрута в `app/routes/contact.js`. Так как мы будем использовать шаблон `contact`, маршрут `contact` не нуждается в дополнительных изменениях.

В `contact.hbs` мы можем добавить контакты для связи с главным офисом Super Rentals:

`app/templates/contact.hbs`
```hbs
<p>Super Rentals Representatives would love to help you choose a destination or answer
any questions you may have.</p>

<p>Contact us today:</p>

<p>
  Super Rentals HQ<br>
  1212 Test Address Avenue<br>
  Testington, OR 97233
</p>

<p>(503)555-1212</p>

<p>superrentalsrep@superrentals.com</p>
```

Мы сделали второй маршрут. Теперь, если мы перейдем по URL `localhost:4200/contact`, то попадем на страницу с контактами.

## Навигация с помощником {{link-to}} и ссылками

Вряд ли вы хотите, чтобы для перехода по страницам пользователи были вынуждены знать полные адреса URL. Поэтому внизу каждой страницы мы добавим ссылки. Давайте сделаем ссылку contact на странице about и ссылку about на странице contact.

В Ember есть встроенные **помощники**, которые предоставляют функциональность вроде установления ссылок на другие маршруты. Здесь мы используем помощник `{{link-to}}`, чтобы связать два маршрута:

`app/templates/about.hbs`
```hbs
<h2>About Super Rentals</h2>

<p>The Super Rentals website is a delightful project created to explore Ember.<br>
  By building a property rental site, we can simultaneously imagine traveling<br>
  AND building Ember applications simultaneously.</p>

{{#link-to "contact"}}Click here to contact us.{{/link-to}}
```

Помощник `{{link-to}}` берет аргумент с именем маршрута, на который нужно сделать ссылку. В нашем случае это `contact`. Теперь, когда мы посмотрим на страницу about, мы увидим рабочую ссылку на раздел contact.

![super rentals about page screenshot](/static/images/guides/routes-and-templates/ember-super-rentals-about.png)

Далее, мы добавим ссылку на страницу about, чтобы перемещаться обратно и между разделами `about` и `contact`.

`app/templates/contact.hbs`
```hbs
<p>Super Rentals Representatives would love to help you <br>
  choose a destination or answer any questions you may have.</p>

<p>Contact us today:</p>

<p>
  Super Rentals HQ<br>
  1212 Test Address Avenue<br>
  Testington, OR 97233
</p>

<p>(503)555-1212</p>

<p>superrentalsrep@superrentals.com</p>

{{#link-to "about"}}About{{/link-to}}
```

## Маршрут Index

У нас есть две статичные страницы. Теперь мы готовы добавить домашнюю страницу, которая будет приветствовать пользователей на сайте. С помощью того же процесса, что мы делали ранее, мы сначала генерируем маршрут `index`. 

```shell
ember g route index
```

Мы видим знакомый результат работы генератора маршрутов

```shell
installing route
  create app/routes/index.js
  create app/templates/index.hbs
installing route-test
  create tests/unit/routes/index-test.js
```

В отличие от других обработчиков маршрутов `index` имеет свои особенности: для него НЕ требуется элемент в отображении роутера. Позднее в разделе про вложенные маршруты мы подробнее рассмотрим эту особенность.

Давайте обновим `index.hbs` и добавим код HTML для домашней страницы и ссылки на другие маршруты приложения:

`app/templates/index.hbs`
```shell
<h1>Welcome to Super Rentals</h1>

<p>We hope you find exactly what you're looking for in a place to stay.</p>

{{#link-to "about"}}About{{/link-to}}
{{#link-to "contact"}}Click here to contact us.{{/link-to}}
```