## Создание записей

Давайте предположим, что у нас есть модели `post` и `comment`, которые связаны друг с другом следующим образом:

`app/models/post.js`
```js
export default DS.Model.extend({
  comments: DS.hasMany('comment')
});
```

`app/models/comment.js`
```js
export default DS.Model.extend({
  post: DS.belongsTo('post')
});
```

Когда пользователь комментирует пост, нам нужно создать связь между двумя записями. Мы можем просто установить связь `belongsTo` в новом комментарии:

```js
let post = this.store.peekRecord('post', 1);
let comment = this.store.createRecord('comment', {
  post: post
});
comment.save();
```

Так мы создадим новую запись `comment` и сохраним ее на сервере. Ember Data также обновит пост, чтобы включить вновь созданный комментарий в связь `comments`.

Мы могли бы также соединить две записи, обновляя связь поста `hasMany`:

```js
let post = this.store.peekRecord('post', 1);
let comment = this.store.createRecord('comment', {
});
post.get('comments').pushObject(comment);
comment.save();
```

В этом случае связь нового комментария `belongsTo` будет установлена родительскому посту.

## Обновление существующих записей

Иногда нам нужно установить связи на уже существующие записи. Мы можем просто установить связь `belongsTo`:

```js
let post = this.store.peekRecord('post', 1);
let comment = this.store.peekRecord('comment', 1);
comment.set('post', post);
comment.save();
```

Также мы могли бы обновить связь `hasMany`, вставив запись в связь:

```js
let post = this.store.peekRecord('post', 1);
let comment = this.store.peekRecord('comment', 1);
post.get('comments').pushObject(comment);
post.save();
```

## Удаление связей

Чтобы удалить связь `belongsTo`, мы можем просто установить для нее значение `null`, что также удалит ее со стороны `hasMany`:

```js
let comment = this.store.peekRecord('comment', 1);
comment.set('post', null);
comment.save();
```

Еще можно удалить запись из связи `hasMany`:

```js
let post = this.store.peekRecord('post', 1);
let comment = this.store.peekRecord('comment', 1);
post.get('comments').removeObject(comment);
post.save();
```

Как и в ранних примерах связь комментария `belongsTo` будет очищена Ember Data.

## Связи как обещания

Работая со связями, важно помнить, что они возвращают обещания.

Например, если бы мы работали с асинхронными комментариями поста, то должны были ждать завершения обещания:

```js
let post = this.store.peekRecord('post', 1);

post.get('comments').then((comments) => {
  // now we can work with the comments
});
```

То же распространяется и на связи `belongsTo`:

```js
let comment = this.store.peekRecord('comment', 1);

comment.get('post').then((post) => {
  // the post is available here
});
```

Шаблоны Handlebars автоматически обновятся, чтобы отразить разрешенное обещание. Мы можем показать список комментариев в посте так:

```hbs
<ul>
  {{#each post.comments as |comment|}}
    <li>{{comment.id}}</li>
  {{/each}}
</ul>
```

Ember Data запросит у сервера подходящие записи и по-новому отобразит шаблон, когда будут получены данные.