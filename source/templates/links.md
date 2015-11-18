## Помощник `{{link-to}}`

Ссылку на маршрут можно создать с использованием помощника `{{link-to}}`.

`app/router.js`
```javascript
Router.map(function() {
  this.route("photos", function(){
    this.route("edit", { path: "/:photo_id" });
  });
});
```

`app/templates/photos.hbs`
```handlebars
<ul>
  {{#each photos as |photo|}}
    <li>{{#link-to 'photos.edit' photo}}{{photo.title}}{{/link-to}}</li>
  {{/each}}
</ul>
```

Если модель для шаблона `photos` представляет собой список из трех фотографий, отображенный код HTML будет выглядеть примерно так:

```html
<ul>
  <li><a href="/photos/1">Happy Kittens</a></li>
  <li><a href="/photos/2">Puppy Running</a></li>
  <li><a href="/photos/3">Mountain Landscape</a></li>
</ul>
```

Помощник `{{link-to}}` принимает один или два аргумента:

* Имя маршрута. В этом примере, это будет `index`, `photos` или `photos.edit`.
* И чаще всего одну модель для каждого [динамического сегмента](http://guides.emberjs.com/v2.0.0/routing/defining-your-routes/#toc_dynamic-segments). По умолчанию, Ember.js заменит каждый сегмент значением свойства `id` соответствующего объекта. В примере выше второй аргумент — это каждый объект `photo`, а свойство `id` используется, чтобы заполнить динамический сегмент `1`, `2` или `3`. Если нет модели, чтобы передать помощнику, вместо нее можно использовать заданное значение:

`app/templates/photos.hbs`
```handlebars
{{#link-to 'photos.photo.edit' 1}}
  First Photo Ever
{{/link-to}}
```

Когда представленная ссылка соответствует текущему маршруту, и тот же экземпляр объекта передается помощнику, тогда ссылке назначается `class="active"`. Например, если вы прошли по URL `/photos/2`, первый пример выше будет выглядеть так:

```html
<ul>
  <li><a href="/photos/1">Happy Kittens</a></li>
  <li><a href="/photos/2" class="active">Puppy Running</a></li>
  <li><a href="/photos/3">Mountain Landscape</a></li>
</ul>
```

### Пример для множественных сегментов

Если маршрут вложенный, то можно предоставить модель или идентификатор для каждого динамического сегмента.

`app/router.js`
```javascript
Router.map(function() {
  this.route("photos", function(){
    this.route("photo", { path: "/:photo_id" }, function(){
      this.route("comments");
      this.route("comment", { path: "/comments/:comment_id" });
    });
  });
});
```

`app/templates/photo/index.hbs`
```handlebars
<div class="photo">
  {{body}}
</div>

<p>{{#link-to 'photos.photo.comment' primaryComment}}Main Comment{{/link-to}}</p>
```

Если вы указываете только одну модель, она представит самый внутренний динамический сегмент `:comment_id`. Сегмент `:photo_id`  будет использовать текущее фото.

В качестве альтернативы, вы могли бы передать помощнику и ID фото и комментарий:

`app/templates/photo/index.hbs`
```handlebars
<p>
  {{#link-to 'photo.comment' 5 primaryComment}}
    Main Comment for the Next Photo
  {{/link-to}}
</p>
```

В примере выше hook модели для `PhotoRoute` запустится с `params.photo_id = 5`. Hook `model` для `CommentRoute` не запустится, пока вы предоставляете объект модели для сегмента `comment`. Идентификатор комментария заполнит URL в соответствии с hook `serialize` `CommentRoute`.

### Использование LINK-TO как строкового помощника

Кроме использования в качестве блочного выражения, помощник `link-to` можно применять в строковой форме. Для этого нужно определить текст ссылки как первый аргумент для помощника:

```handlebars
A link in {{#link-to 'index'}}Block Expression Form{{/link-to}},
and a link in {{link-to 'Inline Form' 'index'}}.
```

Из примера выше мы получим:

```html
A link in <a href='/'>Block Expression Form</a>,
and a link in <a href='/'>Inline Form</a>.
```

### Добавление атрибутов для ссылки

При генерации ссылки, возможно, вам понадобится установить для нее дополнительные атрибуты. Это можно сделать с помощью дополнительных аргументов для `link-to` помощника:

```handlebars
<p>
  {{link-to 'Edit this photo' 'photo.edit' photo class="btn btn-primary"}}
</p>
```

Многие из часто используемых свойств HTML, например, `class` и `rel`, будут работать. Когда вы добавите имена класса, Ember также применит стандарт `ember-view` и, возможно, имена класса `active`.

### Замена записей в истории

Изначально заданное поведение для `link-to` — это добавление записей в историю браузера при переходе между маршрутами. Но чтобы заменить текущую запись в истории браузера, можно использовать опцию `replace=true`:

```handlebars
<p>
  {{#link-to 'photo.comment' 5 primaryComment replace=true}}
    Main Comment for the Next Photo
  {{/link-to}}
</p>
```