Одна из задач обработчика маршрута — это отображение нужного шаблона на экране.

По умолчанию, обработчик маршрута отобразит шаблон с тем же именем, что имеет маршрут. Возьмем такой маршрут:

`app/router.js`
```js
Router.map(function() {
  this.route('posts', function() {
    this.route('new');
  });
});
```

Здесь маршрут `posts` отобразит шаблон `posts.hbs`, и маршрут `posts.new` отобразит `posts/new.hbs`.

Каждый шаблон будет отображаться в `{{outlet}}` шаблона родительского маршрута. Например, маршрут `posts.new` отобразит свой шаблон в `{{outlet}}` `post.hbs`, а маршрут `posts` отобразит шаблон в `{{outlet}}` `application.hbs`.

Если вы хотите отобразить шаблон отличный от исходного, установите свойство [`templateName`](http://emberjs.com/api/classes/Ember.Route.html#property_templateName) маршрута с именем шаблона, который хотите отобразить вместо исходного.

`app/routes/posts.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  templateName: 'posts/favorite-posts'
});
```

Вы можете переопределить hook [`renderTemplate()`](http://emberjs.com/api/classes/Ember.Route.html#method_renderTemplate), если хотите получить больший контроль над отображением шаблона. Помимо прочего, это позволяет вам выбрать контроллер для настройки шаблона и особый заполнитель, в котором он будет отображен.