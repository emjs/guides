Параметры запросов — опциональные пары ключ-значение, которые появляются справа от знака `?` в URL. Например, следующий URL имеет два параметра запросов: `sort` и `page` с соответствующими значениями `ASC` и `2`:

```bash
http://example.com/articles?sort=ASC&page=2
```

Параметры запросов позволяют сериализовать в адрес URL дополнительное состояние приложения, которое нельзя иным способом вставить в путь URL (то есть все, что находится слева от `?`). Параметры запросов в основном используют для представления номера текущей страницы в коллекции, параметрах фильтра или сортировки.

### Указание параметров запросов

Параметры запросов объявляют на контроллерах, которыми управляет маршрут. Например, чтобы сформировать параметры запросов, которые будут активны в пределах маршрута `articles`, их необходимо объявить на `controller:articles`.

Чтобы добавить параметр запросов `category`, который будет отфильтровывать все непопулярные статьи, мы указываем `'category'` в качестве одного из `queryParams` в `controller:article`:

`app/controllers/articles.js`
```js
export default Ember.Controller.extend({
  queryParams: ['category'],
  category: null
});
```

Так мы устанавливаем привязку между параметром запросов `category` в URL и свойством `category` в `controller:articles`. Другими словами, когда пройдут по маршруту `articles`, любые изменения в параметре запросов `category` в URL обновят свойство `category` на `controller:articles`, и наоборот.

Теперь нужно лишь определить вычислительное свойство отфильтрованного по категории массива, который отобразит шаблон `articles`:

`app/controllers/articles.js`
```js
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

С этим кодом мы установили следующее поведение:

1. Если пользователь переходит в `/articles`, `category` примет `null`, поэтому статьи не будут отфильтрованы.
2. Если пользователь переходит в `/articles?category=recent`, `category` примет `"recent"`, и статьи будут отфильтрованы.
3. Внутри маршрута `articles` любые изменения свойства `category` на `controller:articles` приведут к тому, что URL обновит параметр запросов. По умолчанию, изменение свойства параметра запросов не служит причиной для полного перехода роутера (то есть он не обращается к hooks `model`, `setupController` и т. д.). Он только обновит URL.

### Помощник LINK-TO

Помощник `link-to` поддерживает определение параметров запросов посредством подвыражения помощника `query-params`.

```hbs
// Explicitly set target query params
{{#link-to "posts" (query-params direction="asc")}}Sort{{/link-to}}

// Binding is also supported
{{#link-to "posts" (query-params direction=otherDirection)}}Sort{{/link-to}}
```

В примерах выше `direction`, предположительно, служит свойством параметра запросов на `controller:post`. Но оно может относиться и к свойству `direction` на любом из контроллеров, которые связаны с иерархией маршрута `posts`, который сопоставляет конечный контроллер с предоставленным именем свойства.

Помощник `link-to` распознает параметры запросов, когда определяет их «активное» состояние и в соответствии с ним устанавливает класс. Чтобы определить активное состояние, он вычисляет, сохраняют ли параметры запросов то же окончание после нажатия на ссылку. Чтобы это было так, вам не нужно предоставлять все текущие активные параметры запросов.

### TRANSITIONTO

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

Аргументы, предоставленные `transitionTo` или `link-to`, согласовывают изменение только в значениях параметров запросов, а не в иерархии маршрута. Это не считается полным переходом, то есть hooks вроде `model` и `setupController` не запустятся по умолчанию. Только URL и свойства контроллера обновятся в соответствии с новыми значениями параметров запросов.

Но некоторые параметры запросов меняют необходимые загрузочные данные с сервера, и в таком случае желательно разрешить полный переход. Чтобы это сделать, когда свойство параметра запросов контроллера меняется, используйте опциональный хеш конфигурации `queryParams` на `Route`, который связан с этим контроллером, и установите значение `true` для свойства конфигурации параметра запросов `refreshModel`:

`app/routes/articles.js`
```js
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
    // which we can just forward to the server.
    return this.store.query('articles', params);
  }
});
```

`app/controllers/articles.js`
```js
export default Ember.Controller.extend({
  queryParams: ['category'],
  category: null
});
```

### Обновление URL с помощью `replaceState`
 
По умолчанию, Ember будет использовать `pushState`, чтобы обновить URL в адресной строке в ответ на изменение свойства параметра запросов контроллера. Но если вместо этого вы хотите использовать `replaceState` (который предотвращает добавление вторичного элемента в историю браузера), можно указать это в хеше конфигурации `queryParams` в `Route`. Взглянем сюда (продолжение примера выше):

`app/routes/articles.js`
```js
export default Ember.Route.extend({
  queryParams: {
    category: {
      replace: true
    }
  }
});
```

Обратите внимание, что имя этого свойства конфигурации и его исходное значение `false` похожи на помощника `link-to`, который тоже позволяет разрешить переход `replaceState` через `replace=true`.

### Установление соответствия свойства контроллера другому ключу параметра запросов

По умолчанию, определение `foo` в качестве свойства параметра запросов контроллера установит привязку к параметру запросов, чей ключ — `foo` (например, `?foo=123`). Можно установить соответствие свойства контроллера другому ключу-значению параметра запросов контроллера с помощью следующей формы синтаксиса:

`app/controllers/articles.js`
```js
export default Ember.Controller.extend({
  queryParams: {
    category: 'articles_category'
  },
  category: null
});
```

Это изменит свойство `category` в `controller:articles`, чтобы обновлять параметр запросов `articles_category`, и наоборот.

Обратите внимание, что параметры запросов, которые требуют дополнительной настройки, могут быть предоставлены вместе со строками в массиве `queryParams`.

`app/controllers/articles.js`
```js
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

В следующем примере предполагается, что свойство параметра запросов контроллера `page` имеет исходное значение `1`.

`app/controllers/articles.js`
```js
export default Ember.Controller.extend({
  queryParams: 'page',
  page: 1
});
```

Это влияет на поведение параметра запросов следующим образом:

1. Значения параметра запросов приводятся к тому же типу данных, что и исходное значение. Например, изменение URL с `/?page=3` на `/?page=2` установит свойство `page` в `controller:articles` в виде цифры `2`, а не строки `"2"`. То же самое применимо к исходным булевым значениям.
2. Когда у свойства параметра запросов контроллера уже установлено исходное значение, оно не будет сериализовано в URL. Поэтому в примере выше, если `page` — `1`, URL может выглядеть как `/articles`, но если кто-либо установит `2` в качестве значения для `page` контроллера, URL станет `/articles?page=2`.

### «Залипающие» значения параметра запросов

По умолчанию, значения параметра запросов в Ember «залипающие». То есть если вы изменили параметр запросов, затем вышли и снова вернулись к маршруту, новое значение этого параметра запросов будет сохранено (вместо сброса к исходному варианту). Это особенно удобно для сохранения параметров сортировки/фильтрации при возвращении и дальнейшем перемещении между маршрутами.

«Залипающие» значения параметра запросов запоминаются/сохраняются в соответствии с моделью, которую загружают в маршрут. У нас есть маршрут `team` с динамическим сегментом `/:team_name` и параметром запросов контроллера «filter». Вы перемещаетесь в `/badgers` и фильтруете по `"rookies"`, затем переходите в `/bears` и сортируете по `"best"`, далее идете в `/potatoes` и фильтруете по `"lamest"`. Теперь, учитывая следующие ссылки навигационной панели:

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

Это показывает, что когда вы меняете параметр запросов, он сохраняется и привязывается к модели, которую загружают в маршрут.

Если нужно сбросить параметр запросов, есть две опции:

1. напрямую перейти к исходному значению для этого параметра запросов через `link-to` или `transitionTo`;
2. использовать hook `Route.resetController`, чтобы вернуть значения параметра запросов к исходным перед выходом с маршрута или изменением модели маршрута.

В следующем примере параметр запросов контроллера `page` сброшен к `1`, *но он все еще находится в области действия модели пред-перехода `ArticlesRoute`*. В результате все ссылки, которые указывают на покинутый маршрут, будут использовать новое значение `1` в качестве значения для параметра запросов `page`.

`app/routes/articles.js`
```js
export default Ember.Route.extend({
  resetController(controller, isExiting, transition) {
    if (isExiting) {
      // isExiting would be false if only the route's model was changing
      controller.set('page', 1);
    }
  }
});
```

Бывают случаи, когда вы не хотите, чтобы «залипающие» значения параметра запросов находились в области действия модели маршрута, но по новой использовали значение параметра запросов даже после изменения модели маршрута. Это можно осуществить с помощью установки опции `scope` на `"controller"` в хеше конфигурации контроллера `queryParams`:

`app/controllers/articles.js`
```js
export default Ember.Controller.extend({
  queryParams: [{
    showMagnifyingGlass: {
      scope: 'controller'
    }
  }]
});
```

Ниже показано, как можно переопределить область действия и ключ URL параметра запросов в единичном свойстве параметра запросов контроллера:

`app/controllers/articles.js`
```js
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