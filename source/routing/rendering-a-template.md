Одна из главных задач обработчика маршрута — это отображение нужного шаблона на экране.

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

Если вы хотите отобразить какой-либо шаблон, за исключением исходного, выполните hook [`renderTemplate()`](http://emberjs.com/api/classes/Ember.Route.html#method_renderTemplate):

`app/routes/posts.js`
```js
export default Ember.Route.extend({
  renderTemplate() {
    this.render('favoritePosts');
  }
});
```