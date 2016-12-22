## Хелпер `{{link-to}}`

Ссылку на маршрут можно создать с помощью хелпера [`{{link-to}}`](emberjs.com/api/classes/Ember.Templates.helpers.html#method_link-to).

`app/router.js`
```js
Router.map(function() {
  this.route('photos', function(){
    this.route('edit', { path: '/:photo_id' });
  });
});
```

`app/templates/photos.hbs`
```hbs
<ul>
  {{#each photos as |photo|}}
    <li>{{#link-to "photos.edit" photo}}{{photo.title}}{{/link-to}}</li>
  {{/each}}
</ul>
```

Если модель шаблона `photos` представляет собой список из трех фото, отображенный код HTML будет выглядеть примерно так:

```
<ul>
  <li><a href="/photos/1">Happy Kittens</a></li>
  <li><a href="/photos/2">Puppy Running</a></li>
  <li><a href="/photos/3">Mountain Landscape</a></li>
</ul>
```

Хелпер `{{link-to}}` принимает один или два аргумента:

* Имя маршрута. В этом примере, это будет `index`, `photos` или `photos.edit`.
* Чаще всего одну модель для каждого [динамического сегмента](https://guides.emberjs.com/v2.8.0/routing/defining-your-routes/#toc_dynamic-segments). По умолчанию, Ember.js заменит каждый сегмент значением свойства `id` соответствующего объекта. В примере выше второй аргумент — каждый объект `photo`, а свойство `id` используется, чтобы заполнить динамический сегмент `1`, `2` или `3`. Если нет модели, чтобы передать хелперу, вместо нее можно использовать заданное значение:

`app/templates/photos.hbs`
```hbs
{{#link-to "photos.edit" 1}}
  First Photo Ever
{{/link-to}}
```

Когда представленная ссылка соответствует текущему маршруту, и тот же экземпляр объекта передается хелперу, тогда ссылке назначается `class="active"`. Например, если вы прошли по URL `/photos/2`, первый пример выше будет выглядеть так:

```
<ul>
  <li><a href="/photos/1">Happy Kittens</a></li>
  <li><a href="/photos/2" class="active">Puppy Running</a></li>
  <li><a href="/photos/3">Mountain Landscape</a></li>
</ul>
```

## Пример для нескольких сегментов

Если маршрут вложенный, то можно предоставить модель или идентификатор для каждого динамического сегмента.

`app/router.js`
```js
Router.map(function() {
  this.route('photos', function(){
    this.route('photo', { path: '/:photo_id' }, function(){
      this.route('comments');
      this.route('comment', { path: '/comments/:comment_id' });
    });
  });
});
```

`app/templates/photo/index.hbs`
```hbs
<div class="photo">
  {{body}}
</div>

<p>{{#link-to "photos.photo.comment" primaryComment}}Main Comment{{/link-to}}</p>
```

Если вы указываете только одну модель, она представит внутренний динамический сегмент `:comment_id`. Сегмент `:photo_id`  будет использовать текущее фото.

В качестве альтернативы, вы могли бы передать хелперу ID фото и комментарий:

`app/templates/photo/index.hbs`
```hbs
<p>
  {{#link-to 'photo.comment' 5 primaryComment}}
    Main Comment for the Next Photo
  {{/link-to}}
</p>
```

В примере выше hook модели для `PhotoRoute` запустится с `params.photo_id = 5`.
Hook `model` для `CommentRoute` не запустится, пока вы предоставляете объект модели для сегмента `comment`. Идентификатор комментария заполнит URL в соответствии с hook `serialize` в `CommentRoute`.

## Настройка query-params

Хелпер `query-params` можно использовать, чтобы установить параметры запроса для ссылки:

```hbs
// Explicitly set target query params
{{#link-to "posts" (query-params direction="asc")}}Sort{{/link-to}}

// Binding is also supported
{{#link-to "posts" (query-params direction=otherDirection)}}Sort{{/link-to}}
```

## Использование link-to как строкового хелпера

Кроме использования в качестве блочного выражения, хелпер [`link-to`](emberjs.com/api/classes/Ember.Templates.helpers.html#method_link-to) можно применять в строковой форме. Для этого нужно указать текст ссылки как первый аргумент для хелпера:

```hbs
A link in {{#link-to "index"}}Block Expression Form{{/link-to}},
and a link in {{link-to "Inline Form" "index"}}.
```

Из примера выше мы получим:

```html
A link in <a href="/">Block Expression Form</a>,
and a link in <a href="/">Inline Form</a>.
```

## Добавление атрибутов для ссылки

При генерации ссылки, вам может понадобится установить для нее дополнительные атрибуты. Это можно сделать с помощью дополнительных аргументов для хелпера `link-to`:

```hbs
<p>
  {{link-to "Edit this photo" "photo.edit" photo class="btn btn-primary"}}
</p>
```

Многие из часто используемых свойств HTML, например, `class` и `rel`, будут при этом работать.
Когда вы добавите имена класса, Ember также применит стандарт `ember-view` и, возможно, имена класса `active`.

## Замена записей в истории

Изначально заданное поведение для [`link-to`](emberjs.com/api/classes/Ember.Templates.helpers.html#method_link-to) — добавление записей в историю браузера при переходе между маршрутами. Но чтобы заменить текущую запись в истории браузера, можно использовать опцию `replace=true`:

```hbs
<p>
  {{#link-to "photo.comment" 5 primaryComment replace=true}}
    Main Comment for the Next Photo
  {{/link-to}}
</p>
```