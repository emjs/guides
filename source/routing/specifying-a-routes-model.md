Вам нужно, чтобы шаблон отобразил данные из модели. Загрузка соответствующей модели — одна из задач маршрута.

Например, возьмем этот маршрут:

`app/router.js`
```js
Router.map(function() {
  this.route('favoritePosts');
});
```

Чтобы загрузить модель для маршрута `favoritePosts`, вы будете использовать hook [`model()`](http://emberjs.com/api/classes/Ember.Route.html#method_model) в обработчике маршрута `posts`:

`app/routes/favorite-posts.js`
```js
export default Ember.Route.extend({
  model() {
    return this.store.query('post', { favorite: true });
  }
});
```

Обычно hook `model` возвращает запись [Ember Data](http://emjs.ru/v2/models/), но он также может вернуть любой объект [обещания](https://www.promisejs.org/) (записи Ember Data — это обещания) или простой объект/массив JavaScript. Ember будет ждать, пока данные закончат загружаться (пока завершится обещание), чтобы отобразить шаблон.

Тогда возвращенное значение из hook `model` становится доступно в шаблоне и контроллере со свойством `model`:

`app/templates/favorite-post.hbs`
```hbs
<h1>Favorite Posts</h1>
{{#each model as |post|}}
  <p>{{post.body}}</p>
{{/each}}
```

## Динамические модели

Некоторые маршруты всегда показывают одну и ту же модель. Например, маршрут `/photos` всегда показывает один список фотографий, который доступен в приложении. Если пользователь оставляет этот маршрут и возвращается к нему позднее, модель не меняется.

Но зачастую в ваших маршрутах модель будет меняться в зависимости от действий пользователя. Например, представьте приложение для просмотра фотографий. Маршрут `/photos` отобразит шаблон `photos` со списком фотографий в качестве модели, которая никогда не меняется. Но когда пользователь щелкает на конкретное фото, нам нужно, чтобы отобразилась эта модель с шаблоном `photo`. Если пользователь вернется и щелкнет на другую фотографию, нам нужно, чтобы снова отобразился шаблон `photo`, но в этот раз с другой моделью.

В подобных случаях важно включить в URL некоторую информацию не только о том, какой шаблон показать, но и какую использовать модель.

В Ember это можно осуществить, если определить маршруты с [динамическими сегментами](http://emjs.ru/v2/routing/defining-your-routes/).

Когда вы определили маршрут с динамическим сегментом, Ember будет извлекать для вас значение динамического сегмента из URL и передавать его hook `model` как хеш в качестве первого аргумента.

`app/router.js`
```js
Router.map(function() {
  this.route('photo', { path: '/photos/:photo_id' });
});
```

`app/routes/photo.js`
```js
export default Ember.Route.extend({
  model(params) {
    return this.store.findRecord('photo', params.photo_id);
  }
});
```

В hook `model` для маршрутов с динамическими сегментами ваша задача переводить идентификатор (вроде `47` или `post-slug`) в модель, которую может отобразить шаблон маршрута. В примере выше мы используем идентификатор фотографии (`params.photo_id`) как аргумент для метода Ember Data `findRecord`.
 
Примечание: hook `model` в маршруте с динамическим сегментом вызывается только в случае, если маршрут подается через URL. Если маршрут подается через переход (например, при использовании помощника Handlebars [link-to](http://emjs.ru/v2/templates/links/)), тогда контекст модели уже предоставлен, и hook не активируется. Маршруты без динамических сегментов всегда будут реализовать hook модели.

## Несколько моделей

Несколько моделей можно вернуть через [Ember.RSVP.hash](http://emberjs.com/api/classes/RSVP.html#method_hash). `Ember.RSVP.hash` принимает параметры, которые возвращают обещания. Обещание `Ember.RSVP.hash` завершается, когда разрешаются все параметры обещаний. Например:

`app/routes/songs.js`
```js
export default Ember.Route.extend({
  model() {
    return Ember.RSVP.hash({
      songs: this.store.findAll('song'),
      albums: this.store.findAll('album')
    });
  }
});
```

В шаблоне `songs` мы можем указать обе модели и использовать помощника `{{#each}}`, чтобы отобразить каждую запись в модели song и модели album:

`app/templates/songs.hbs`
```hbs
<h1>Playlist</h1>

<ul>
  {{#each model.songs as |song|}}
    <li>{{song.name}} by {{song.artist}}</li>
  {{/each}}
</ul>

<h1>Albums</h1>

<ul>
  {{#each model.albums as |album|}}
    <li>{{album.title}} by {{album.artist}}</li>
  {{/each}}
</ul>
```