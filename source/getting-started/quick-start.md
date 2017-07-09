Это руководство научит вас создавать с нуля простые приложения с помощью Ember.
Мы рассмотрим следующие этапы:

1. Установка Ember.
2. Создание нового приложения.
3. Определение маршрута.
4. Написание компонента пользовательского интерфейса.
5. Компоновка приложения для развертывания в рабочей среде.

## Установка Ember

Ember можно установить с помощью одной команды npm - диспетчере пакетов Node.js или [Yarn](https://yarnpkg.com/lang/en/). Введите в терминале следующее:

С помощью npm
```bash
npm install -g ember-cli
```

У вас нет npm? [Узнайте здесь, как установить Node.js и npm](https://docs.npmjs.com/getting-started/installing-node).

Или с помощью yarn
```bash
yarn add global ember-cli
```

Чтобы ознакомиться с полным списком зависимостей, необходимых для проекта Ember CLI, посмотрите раздел [Установка Ember](http://emjs.ru/v2/getting-started/) в руководстве.

## Создание нового приложения

После установки Ember CLI через npm в терминале вы получите доступ к новой команде `ember`. Можно использовать команду `ember new`, чтобы создать новое приложение.

```bash
ember new ember-quickstart
```

Эта команда создаст новую директорию под названием `ember-quickstart` и установит в нее новое приложение Ember. По умолчанию, у вас будут следующие средства для создания приложения:

* Сервер разработки.
* Средство компиляции шаблонов.
* Средство минификации JavaScript и CSS.
* Возможности ES2015, реализованные посредством Babel.

Ember предоставляет вам все, что необходимо для создания готовых к использованию веб-приложений в интегрированном пакете, что позволяет с легкостью начинать новые проекты.

Убедимся, что все работает правильно. Перейдите в директорию приложения `ember-quickstart` и запустите сервер разработки с помощью следующих команд:

```bash
cd ember-quickstart
ember server
```

Через несколько секунд вы должны увидеть результат примерно в таком виде:

```
Livereload server on http://localhost:49152
Serving on http://localhost:4200/
```

(Чтобы в любой момент остановить сервер, нажмите Ctrl-C в терминале).

Откройте [`http://localhost:4200`](localhost:4200) в любом браузере. Вы должны увидеть страницу приветствия Ember и ничего больше. Поздравляем! Вы только что создали и загрузили свое первое приложение на Ember.

Создадим новый шаблон с помощью команды `ember generate`.

```
ember generate template application
```

Шаблон `application` отображается на экране, пока у пользователя загружено ваше приложение. Откройте в редакторе `app/templates/application.hbs` и добавьте следующее:

`app/templates/application.hbs`
```hbs
<h2>PeopleTracker</h2>

{{outlet}}
```

Учтите, что Ember обнаруживает новый файл и автоматически перезагружает для вас страницу в фоновом режиме. Страница приветствия должна смениться на "PeopleTracker".

## Определение маршрута

Создадим приложение, которое показывает список ученых. Для этого сначала создаем маршрут. Пока что можете рассматривать маршруты как разные страницы, из которых состоит ваше приложение.

В Ember есть *генераторы*, которые автоматизируют написание стереотипного кода для типичных задач. Чтобы сгенерировать маршрут, наберите в терминале следующее:

```
ember generate route scientists
```

Вы увидите примерно такой результат:

```
installing route
  create app/routes/scientists.js
  create app/templates/scientists.hbs
updating router
  add route scientists
installing route-test
  create tests/unit/routes/scientists-test.js
```

Так Ember сообщает вам, что он создал:

1. Шаблон, который отобразится, когда пользователь посетит `/scientists`.
2. Объект `Route` он запрашивает модель, которую использует этот шаблон.
3. Запись в роутере приложения (расположена в `app/router.js`).
4. Модульный тест для этого маршрута.

Откройте только что созданный шаблон в `app/templates/scientists.hbs` и добавьте следующий код HTML:

`app/templates/scientists.hbs`
```hbs
<h2>List of Scientists</h2>
```

В браузере откройте [`http://localhost:4200/scientists`](localhost:4200/scientists). Вы должны увидеть заголовок `<h2>` из шаблона `scientists.hbs` прямо под заголовком `<h2>` из шаблона `application.hbs`.

Теперь, когда у нас отображается шаблон `scientists`, внесем в него какие-нибудь данные. Чтобы сделать это, мы укажем *модель* для нашего маршрута. Чтобы указать модель, мы отредактируем `app/routes/scientists.js`.

Мы возьмем код, который для нас создал генератор, и добавим метод `model()` в `Route`:

`app/routes/scientists.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return ['Marie Curie', 'Mae Jemison', 'Albert Hofmann'];
  }
});
```

(В этом примере используются последние возможности JavaScript, с некоторыми из которых вы, возможно, не знакомы. Подробнее об этом можете узнать в статье [Обзор новейших возможностей JavaScript](https://ponyfoo.com/articles/es6)).

С помощью метода `model()` в маршруте вы возвращаете любые данные, которые хотите увидеть в шаблоне. Если вам нужно запросить данные асинхронно, то метод `model()` поддерживает любую библиотеку, которая использует [Обещания JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

Теперь сообщим Ember, как преобразовать этот массив строк в код HTML. Откройте шаблон `scientists` и добавьте какой-нибудь код Handlebars, чтобы пройти в цикле по массиву и отобразить его:

`app/templates/scientists.hbs`
```hbs
<h2>List of Scientists</h2>

<ul>
  {{#each model as |scientist|}}
    <li>{{scientist}}</li>
  {{/each}}
</ul>
```

Здесь мы используем хелпер `each`, чтобы пройти в цикле каждый элемент массива, который мы получаем из hook `model()`, и помещаем его внутри элемента `<li>`.

## Создание компонента пользовательского интерфейса

По мере расширения приложения вы заметите, что используете одинаковые элементы пользовательского интерфейса на нескольких страницах (или используете их несколько раз на одной странице). Ember позволяет с легкостью перестроить шаблоны в повторно используемые компоненты.

Создадим компонент `people-list`, который можно использовать в нескольких местах, чтобы отображать список людей.

Как обычно, генератор упростит для нас эту задачу. Чтобы создать новый компонент, наберите:

```
ember generate component people-list
```

Скопируйте и вставьте шаблон `scientists` в шаблон компонента `people-list` и отредактируйте его, чтобы он выглядел так:

`app/templates/components/people-list.hbs`
```hbs
<h2>{{title}}</h2>

<ul>
  {{#each people as |person|}}
    <li>{{person}}</li>
  {{/each}}
</ul>
```

Обратите внимание, что мы изменили заголовок из строго закодированной строки (List of Scientists) на динамическое свойство (`{{title}}`). Мы также переименовали `scientist` в более обобщенное понятие person, чтобы ослабить связь компонента с местом, где он используется.

Сохраните этот шаблон и вернитесь к шаблону `scientists`. Замените весь старый код на новую, разбитую на компоненты версию. Компоненты выглядят как теги HTML, но вместо угловых скобок (`<tag>`) в них используются двойные фигурные скобки (`{{component}}`). Мы сообщим нашему компоненту:

1. Какой заголовок использовать, через атрибут `title`.
2. Какой массив людей использовать, через атрибут `people`. Мы используем `model` маршрута как список людей.

`app/templates/scientists.hbs`
```hbs
{{people-list title="List of Scientists" people=model}}
```

Вернитесь в браузер. Пользовательский интерфейс должен выглядеть идентично. Единственная разница в том, что мы преобразовали наш список в более пригодную для повторного использования и более обслуживаемую версию, разбитую на компоненты.

Это можно проверить на практике, если создать новый маршрут, который показывает другой список людей. В качестве упражнения вы можете попробовать создать маршрут `programmers`, который показывает список известных программистов. Если вы повторно используете компонент `people-list`, вам почти не потребуется писать код.

## Компоновка для использования в рабочей среде

Теперь, когда мы написали приложение и проверили его работоспособность на этапе разработки, пришло время подготовить его для развертывания в среде наших пользователей. Чтобы сделать это, запустите следующую команду:

```
ember build --env production
```

Команда `build` упаковывает все ресурсы, из которых состоит ваше приложение: JavaScript, шаблоны, CSS, веб-шрифты, изображения и т. д.

Здесь мы сообщили Ember скомпоновать приложение для рабочей среды с помощью метки `--env`. Так мы создаем оптимизированный пакет, который готов к загрузке на ваш веб-хост. После завершения компоновки вы найдете все объединенные и минифицированные ресурсы в директории приложения `dist/`.

Сообщество Ember ценит сотрудничество и создание общих инструментов, которыми каждый может воспользоваться. Если вы хотите развернуть свое приложение в рабочей среде быстрым и надежным способом, ознакомьтесь с дополнением [Ember CLI Deploy](http://ember-cli-deploy.com/).

Если вы развернули приложение на веб-сервере Apache, сначала создайте новый виртуальный хост для приложения. Убедитесь, что все маршруты обрабатываются index.html. Для этого добавьте следующую директиву в конфигурацию виртуального хоста приложения: `FallbackResource index.html`.