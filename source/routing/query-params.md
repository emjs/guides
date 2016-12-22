Параметры запроса — опциональные пары ключ-значение, которые появляются справа от знака `?` в URL. Например, следующий URL имеет два параметра запросов: `sort` и `page` с соответствующими значениями `ASC` и `2`:

```bash
http://example.com/articles?sort=ASC&page=2
```

Параметры запросов позволяют преобразовать в адрес URL дополнительное состояние приложения, которое нельзя иным способом вставить в путь URL (то есть все, что находится слева от `?`). Параметры запросов в основном используют для представления номера текущей страницы в коллекции, параметрах фильтра или сортировки.

### Указание параметров запроса

Параметры запроса объявляют в контроллерах, которыми управляет маршрут. Например, чтобы сформировать параметры запросов, которые будут активны в пределах маршрута `articles`, их необходимо объявить на `controller:articles`.

Чтобы добавить параметр запроса `category`, который будет фильтровать все статьи, которые не входят в категорию popular (популярные), мы указываем `'category'` в качестве одного из `queryParams` в `controller:article`:

`app/controllers/articles.js`
```js
import Ember from 'ember';

export default Ember.Controller.extend({
  queryParams: ['category'],
  category: null
});
```

Так мы устанавливаем привязку между параметром запросов `category` в URL и свойством `category` в `controller:articles`. Когда пользователь перейдет по маршруту `articles`, любые изменения в параметре запросов `category` в URL обновят свойство `category` на `controller:articles`, и наоборот. Учтите, что нельзя привязать `queryParams` к вычисляемым свойствам; они должны быть значениями.

Теперь нужно лишь определить вычислительное свойство отфильтрованного по категории массива, который отобразит шаблон `articles`:

`app/controllers/articles.js`
```js
import Ember from 'ember';

export default Ember.Controller.extend({
  queryParams: ['category'],
  category: null,

  filteredArticles: Ember.computed('category', 'model', function() {
    var category = this.get('category');
    var articles = this.get('model');

    if (category) {
      return articles.filterBy('category', category);
    } else {
      return articles;
    }
  })
});
```

С помощью этого кода мы задали следующее поведение:

1. Если пользователь переходит в `/articles`, `category` примет `null`, поэтому статьи не будут отфильтрованы.
2. Если пользователь переходит в `/articles?category=recent`, `category` примет `"recent"`, и статьи будут отфильтрованы.
3. Внутри маршрута `articles` любые изменения свойства `category` на `controller:articles` приведут к тому, что URL обновит параметр запроса. По умолчанию изменение свойства параметра запроса не приводит к полному переходу (то есть не запускает hook'и `model`, `setupController` и т. д.). Изменение только обновит URL.

### Хелпер link-to

Хелпер `link-to` поддерживает определение параметров запросов посредством подвыражения `query-params`.

```hbs
// Explicitly set target query params
{{#link-to "posts" (query-params direction="asc")}}Sort{{/link-to}}

// Binding is also supported
{{#link-to "posts" (query-params direction=otherDirection)}}Sort{{/link-to}}
```

В примерах выше `direction` служит свойством параметра запроса в контроллере `post`. Но `direction` может относиться и к свойству в любом из контроллеров, которые связаны с иерархией маршрута `posts`, вплоть до крайнего контроллера с предоставленным именем свойства.

Хелпер `link-to` распознает параметры запросов, когда определяет их «активное» состояние и в соответствии с ним устанавливает класс. Чтобы определить активное состояние, он вычисляет, сохраняют ли параметры запросов то же окончание после нажатия на ссылку. Чтобы это было так, вам не нужно предоставлять все текущие активные параметры запросов.

### transitionTo

`Route#transitionTo` и `Controller#transitionToRoute` принимают конечный аргумент, который является объектом с ключом `queryParams`.

`app/routes/some-route.js`
```js
this.transitionTo('post', object, { queryParams: { showDetails: true }});
this.transitionTo('posts', { queryParams: { sort: 'title' }});

// if you just want to transition the query parameters without changing the route
this.transitionTo({ queryParams: { direction: 'asc' }});
```

Можно еще добавить параметры запросов к переходам URL:

`app/routes/some-route.js`
```js
this.transitionTo('/posts/1?sort=date&showDetails=true');
```

### Разрешение на полный переход

Аргументы, предоставленные `transitionTo` или `link-to`, соответствуют только изменению в значениях параметров запроса, а не в иерархии маршрута. Это не считается полным переходом, то есть hooks вроде `model` и `setupController` не запустятся по умолчанию. Только URL и свойства контроллера обновятся в соответствии с новыми значениями параметров запросов.

Но некоторые изменения параметров запроса приводят к загрузке данных с сервера. В таком случае желательно разрешить полный переход. Чтобы это сделать, когда свойство параметра запроса в контроллере меняется, используйте опциональный хеш конфигурации `queryParams` в `Route`, который связан с этим контроллером, и присвойте `true` свойству конфигурации `refreshModel` параметра запроса:

`app/routes/articles.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  queryParams: {
    category: {
      refreshModel: true
    }
  },
  model(params) {
    // This gets called upon entering 'articles' route
    // for the first time, and we opt into refiring it upon
    // query param changes by setting `refreshModel:true` above.

    // params has format of { category: "someValueOrJustNull" },
    // which we can forward to the server.
    return this.get('store').query('article', params);
  }
});
```

`app/controllers/articles.js`
```js
import Ember from 'ember';

export default Ember.Controller.extend({
  queryParams: ['category'],
  category: null
});
```

### Обновление URL с помощью `replaceState`
 
По умолчанию Ember использует `pushState`, чтобы обновить URL в адресной строке в ответ на изменение свойства параметра запроса в контроллере. Но если вместо него вы хотите использовать `replaceState` (который предотвращает появление дополнительного элемента в истории браузера), в хеше конфигурации `queryParams` в `Route` можно указать следующее (продолжение примера выше):

`app/routes/articles.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  queryParams: {
    category: {
      replace: true
    }
  }
});
```

Обратите внимание, что имя этого свойства конфигурации и его исходное значение `false` похожи на хелпер `link-to`, который тоже позволяет разрешить переход `replaceState` через `replace=true`.

### Сопоставление свойства контроллера с другим ключом параметра запроса

По умолчанию определение `foo` в качестве свойства параметра запроса в контроллере установит привязку к параметру запроса, чей ключ — `foo` (например, `?foo=123`). Можно также сопоставить свойства контроллера с другим ключом параметра запроса с помощью следующего синтаксиса конфигурации:

`app/controllers/articles.js`
```js
import Ember from 'ember';

export default Ember.Controller.extend({
  queryParams: {
    category: 'articles_category'
  },
  category: null
});
```

Это приведет к тому, что изменения свойства `category` в `controller:articles` будут обновлять параметр запроса `articles_category` и наоборот.

Обратите внимание, что параметры запроса, которые требуют дополнительной настройки, можно указать вместе со строками в массиве `queryParams`.

`app/controllers/articles.js`
```js
import Ember from 'ember';

export default Ember.Controller.extend({
  queryParams: ['page', 'filter', {
    category: 'articles_category'
  }],
  category: null,
  page: 1,
  filter: 'recent'
});
```

### Исходные значения и десериализация

В следующем примере предполагается, что свойство `page` параметра запроса в контроллере имеет исходное значение `1`.

`app/controllers/articles.js`
```js
import Ember from 'ember';

export default Ember.Controller.extend({
  queryParams: 'page',
  page: 1
});
```

Это влияет на поведение параметра запроса следующим образом:

1. Значения параметра запроса приводятся к тому же типу данных, что и исходное значение. Например, изменение URL с `/?page=3` на `/?page=2` установит свойство `page` в `controller:articles` в виде цифры `2`, а не строки `"2"`. То же самое применимо к исходным булевым значениям.
2. Когда у свойства параметра запросов контроллера уже установлено исходное значение, оно не будет сериализовано в URL. Поэтому в примере выше, если `page` — `1`, URL может выглядеть как `/articles`, но если кто-либо установит `2` в качестве значения для `page` контроллера, URL станет `/articles?page=2`.

### «Залипающие» значения параметра запроса

По умолчанию, значения параметра запроса в Ember «залипающие». То есть если вы изменили параметр запроса, затем вышли и снова вернулись к маршруту, новое значение этого параметра сохранится (вместо сброса к исходному варианту). Это особенно удобно для сохранения параметров сортировки/фильтрации при перемещении между маршрутами.

«Залипающие» значения параметра запроса запоминаются/сохраняются в соответствии с моделью, которую загружает в маршрут. У нас есть маршрут `team` с динамическим сегментом `/:team_name` и параметром запроса «filter» в контроллере. Вы перемещаетесь в `/badgers` и фильтруете по `"rookies"`, затем переходите в `/bears` и сортируете по `"best"`, далее идете в `/potatoes` и фильтруете по `"lamest"`. Теперь, учитывая следующие ссылки навигационной панели:

```hbs
{{#link-to "team" "badgers"}}Badgers{{/link-to}}
{{#link-to "team" "bears"}}Bears{{/link-to}}
{{#link-to "team" "potatoes"}}Potatoes{{/link-to}}
```

сгенерированные ссылки будут такими:

```html
<a href="/badgers?filter=rookies">Badgers</a>
<a href="/bears?filter=best">Bears</a>
<a href="/potatoes?filter=lamest">Potatoes</a>
```

Это показывает, что когда вы меняете параметр запроса, он сохраняется и привязывается к модели, которую загружает маршрут.

Сбросить параметр запроса можно двумя способами:

1. напрямую перейти к исходному значению для этого параметра запроса через `link-to` или `transitionTo`;
2. использовать hook `Route.resetController`, чтобы вернуть значения параметра запроса к исходным до выхода с маршрута или изменением модели маршрута.

В следующем примере параметр запроса `page` в контроллере сброшен к 1, *но он все еще находится в области действия пред-перехода модели `ArticlesRoute`*. В результате все ссылки, которые вели на покинутый маршрут, будут использовать новое значение `1` в качестве значения для параметра запроса `page`.

`app/routes/articles.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  resetController(controller, isExiting, transition) {
    if (isExiting) {
      // isExiting would be false if only the route's model was changing
      controller.set('page', 1);
    }
  }
});
```

В некоторых случаях вам необходимо, чтобы «залипающие» значения параметра запроса не находились в области действия модели маршрута, а повторно использовали значение параметра запроса даже при изменении модели маршрута. Это можно осуществить, установив опцию `scope` `"controller"` в хеш конфигурации `queryParams` контроллера:

`app/controllers/articles.js`
```js
import Ember from 'ember';

export default Ember.Controller.extend({
  queryParams: [{
    showMagnifyingGlass: {
      scope: 'controller'
    }
  }]
});
```

Ниже показано, как можно переопределить область действия и ключ URL единичного свойства параметра запроса в контроллере:

`app/controllers/articles.js`
```js
import Ember from 'ember';

export default Ember.Controller.extend({
  queryParams: ['page', 'filter',
    {
      showMagnifyingGlass: {
        scope: 'controller',
        as: 'glass'
      }
    }
  ]
});
```