Кроме обычного текста, возможно, вам понадобится, чтобы шаблоны содержали элементы HTML, чьи атрибуты связаны с контроллером.

Например, представьте, что ваш контроллер имеет свойство, которое содержит URL изображения:

```hbs
<div id="logo">
  <img src={{logoUrl}} alt="Logo">
</div>
```

Это генерирует следующий код HTML:

```html
<div id="logo">
  <img src="http://www.example.com/images/logo.png" alt="Logo">
</div>
```

Если вы используете данные, которые связаны с булевым значением, это добавит или удалит определенный атрибут. Например, возьмем такой шаблон:

```hbs
<input type="checkbox" disabled={{isAdministrator}}>
```

Если `isAdministrator` = `true`, Handlebars представит следующий элемент HTML:

```html
<input type="checkbox" disabled>
```

Если `isAdministrator` = `false`, Handlebars представит следующее:

```html
<input type="checkbox">
```

## Добавление атрибутов данных

По умолчанию хелперы и компоненты не принимают атрибуты данных. Например,

```hbs
{{#link-to "photos" data-toggle="dropdown"}}Photos{{/link-to}}

{{input type="text" data-toggle="tooltip" data-placement="bottom" title="Name"}}
```

генерирует следующий код HTML:

```html
<a id="ember239" class="ember-view" href="#/photos">Photos</a>

<input id="ember257" class="ember-view ember-text-field" type="text"
       title="Name">
```

Чтобы разрешить поддержку атрибутов данных, нужно добавить компоненту привязку к атрибуту, например, [`Ember.LinkComponent`](http://emberjs.com/api/classes/Ember.LinkComponent.html) или [`Ember.TextField`](http://emberjs.com/api/classes/Ember.TextField.html) для определенного атрибута:

```js
Ember.LinkComponent.reopen({
  attributeBindings: ['data-toggle']
});

Ember.TextField.reopen({
  attributeBindings: ['data-toggle', 'data-placement']
});
```

Теперь тот же шаблон, что мы видели выше, отобразит следующий код HTML:

```html
<a id="ember240" class="ember-view" href="#/photos" data-toggle="dropdown">Photos</a>

<input id="ember259" class="ember-view ember-text-field"
       type="text" data-toggle="tooltip" data-placement="bottom" title="Name">
```