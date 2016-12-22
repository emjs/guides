Часто требуется, чтобы шаблон отобразил данные из модели. Загрузка соответствующей модели — одна из задач маршрута.

Например, возьмем этот маршрут:

`app/router.js`
```js
Router.map(function() {
  this.route('favorite-posts');
});
```

Чтобы загрузить модель для маршрута `favoritePosts`, вы будете использовать hook [`model()`](http://emberjs.com/api/classes/Ember.Route.html#method_model) в обработчике маршрута `posts`:

`app/routes/favorite-posts.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.get('store').query('post', { favorite: true });
  }
});
```

Обычно hook `model` возвращает запись [Ember Data](http://emjs.ru/v2/models/), но он также может вернуть любой объект-[обещание](https://www.promisejs.org/) (записи Ember Data — это обещания) или простой объект/массив JavaScript. Ember будет ждать, пока данные полностью загрузятся (пока завершится обещание), чтобы отобразить шаблон.

После этого маршрут устанавливает возвращенное значение из hook `model` как свойство `model` контроллера. И вы сможете получить доступ к свойству `model` контроллера в шаблоне:

`app/templates/favorite-post.hbs`
```hbs
<h1>Favorite Posts</h1>
{{#each model as |post|}}
  <p>{{post.body}}</p>
{{/each}}
```

## Динамические модели

Некоторые маршруты всегда показывают одну и ту же модель. Например, маршрут `/photos` всегда показывает один список фотографий, который доступен в приложении. Если пользователь покидает этот маршрут и возвращается к нему позднее, модель не меняется.

Но вы часто будете сталкиваться с маршрутами, где модель меняется в зависимости от действий пользователя. Например, представьте приложение для просмотра фото. Маршрут `/photos` отображает шаблон `photos` со списком фото в качестве модели, которая никогда не меняется. Но когда пользователь нажимает конкретное фото, нам нужно отобразить эту модель с шаблоном `photo`. Если пользователь вернется и нажмет другое фото, нам нужно снова отобразить шаблон `photo`, но в этот раз с другой моделью.

В подобных случаях важно включить в URL некоторую информацию не только о том, какой шаблон показать, но и какую использовать модель.

В Ember это можно сделать, определив маршруты с [динамическими сегментами](http://emjs.ru/v2/routing/defining-your-routes/#toc_dynamic-segments).

Когда вы определили маршрут с динамическим сегментом, Ember будет извлекать для вас значение динамического сегмента из URL и передавать его hook `model` как хеш в качестве первого аргумента.

`app/router.js`
```js
Router.map(function() {
  this.route('photo', { path: '/photos/:photo_id' });
});
```

`app/routes/photo.js`
```js
import Ember from 'ember';

export default Ember.Route.extend({
  model(params) {
    return this.get('store').findRecord('photo', params.photo_id);
  }
});
```

В hook `model` для маршрутов с динамическими сегментами ваша задача преобразовать идентификатор (вроде `47` или `post-slug`) в модель, которую может отобразить шаблон маршрута. В примере выше мы используем идентификатор фотографии (`params.photo_id`) как аргумент для метода Ember Data `findRecord`.
 
Примечание: hook `model` маршрута с динамическим сегментом вызывается только в том случае, если на маршрут переходят через URL. Если на маршрут попадают через переход (например, при использовании хелпера Handlebars [link-to](http://emjs.ru/v2/templates/links/)), и предоставлен контекст модели (второй аргумент для `link-to`), тогда hook не будет выполнен. Если вместо этого предоставлен идентификатор (например, id или slug [короткое имя]), то hook model будет выполнен.

Например, переход на маршрут `photo` таким образом не приведет к выполнению hook `model` (потому что `link-to` была передана модель).

`app/templates/photos.hbs`
```hbs
<h1>Photos</h1>
{{#each model as |photo|}}
  <p>
    {{#link-to 'photo' photo}}
      <img src="{{photo.thumbnailUrl}}" alt="{{photo.title}}" />
    {{/link-to}}
  </p>
{{/each}}
```

А переход таким образом приведет к выполнению hook `model` (потому что `link-to` был передан `photo.id`, то есть идентификатор):

`app/templates/photos.hbs`
```hbs
<h1>Photos</h1>
{{#each model as |photo|}}
  <p>
    {{#link-to 'photo' photo.id}}
      <img src="{{photo.thumbnailUrl}}" alt="{{photo.title}}" />
    {{/link-to}}
  </p>
{{/each}}
```

Маршруты без динамических сегментов всегда будут запускать выполнение hook model.

## Несколько моделей

Несколько моделей можно вернуть через [RSVP.hash](http://emberjs.com/api/classes/RSVP.html#method_hash). `RSVP.hash` принимает параметры, которые возвращают обещания. Когда все обещания параметров будут разрешены, разрешается обещание `RSVP.hash`. Например:

`app/routes/songs.js`
```js
import Ember from 'ember';
import RSVP from 'rsvp';

export default Ember.Route.extend({
  model() {
    return RSVP.hash({
      songs: this.get('store').findAll('song'),
      albums: this.get('store').findAll('album')
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