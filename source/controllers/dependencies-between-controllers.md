Иногда, особенно с вложенными ресурсами, нам может понадобиться некий вид связи между двумя контроллерами. Возьмем этот роутер как пример:

`app/router.js`
```js
Router.map(function() {
  this.route('post', { path: '/posts/:post_id' }, function() {
    this.route('comments', { path: '/comments' });
  });
});
```

Если мы посетим URL `/posts/1/comments`, наша модель `Post` будет загружена в модели `PostController`. Это значит, что она не будет напрямую доступна в `CommentsController`. Но, может, мы хотим показать некоторую информацию о ней в шаблоне `comments`.

Чтобы сделать это, мы внедряем `PostController` в `CommentsController` (который имеет желаемую модель `Post`).

`app/controllers/comments.js`
```js
export default Ember.Controller.extend({
  postController: Ember.inject.controller('post')
});
```

Как только comments получает доступ к `PostController`, чтобы считать модель из контроллера, можно использовать псевдонимы, доступные только для чтения. Чтобы получить модель `Post`, мы обращаемся к `postController.model`:

`app/controllers/comments.js`
```js
export default Ember.Controller.extend({
  postController: Ember.inject.controller('post'),
  post: Ember.computed.reads('postController.model')
});
```

`app/templates/comments.hbs`
```js
<h1>Comments for {{post.title}}</h1>

<ul>
  {{#each model as |comment|}}
    <li>{{comment.text}}</li>
  {{/each}}
</ul>
```

Чтобы узнать больше о псевдонимах, смотрите документацию API по [свойствам псевдонимов](http://emberjs.com/api/#method_computed_alias). Если вам нужно больше всестороннего «совместного использования данных» в приложении, то смотрите [страницу служб](http://guides.emberjs.com/v2.1.0/services/), которая в широком масштабе заменяет внедренные контроллеры.