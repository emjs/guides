Кроме обычного текста, возможно, вам понадобится, чтобы шаблоны содержали элементы HTML, чьи атрибуты связаны с контроллером.

Например, представьте, что ваш контроллер имеет свойство, которое содержит URL на изображение:

```handlebars
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

Если вы используете данные, которые связаны с булевым значением, это добавит или удалит определенный атрибут. Например, есть такой шаблон:

```handlebars
<input type="checkbox" disabled={{isAdministrator}}>
```

Если `isAdministrator` — `true`, Handlebars сформирует следующий элемент HTML:

```html
<input type="checkbox" disabled>
```

Если `isAdministrator` — `false`, Handlebars сформирует следующее:

```html
<input type="checkbox">
```

### Добавление атрибутов данных

По умолчанию, помощники представления (view) не принимают *атрибуты данных*. Например

```handlebars
{{#link-to "photos" data-toggle="dropdown"}}Photos{{/link-to}}

{{input type="text" data-toggle="tooltip" data-placement="bottom" title="Name"}}
```

генерирует следующий код HTML:

```handlebars
<a id="ember239" class="ember-view" href="#/photos">Photos</a>

<input id="ember257" class="ember-view ember-text-field" type="text"
       title="Name">
```

Чтобы разрешить поддержку атрибутов данных, привязка атрибутов должна быть добавлена к компоненту, например, `Ember.LinkComponent` или `Ember.TextField` для определенного атрибута:

```javascript
Ember.LinkComponent.reopen({
  attributeBindings: ['data-toggle']
});

Ember.TextField.reopen({
  attributeBindings: ['data-toggle', 'data-placement']
});
```

Теперь тот же код handlebars, что мы видели выше, отобразит следующий код HTML:

```html
<a id="ember240" class="ember-view" href="#/photos" data-toggle="dropdown">Photos</a>

<input id="ember259" class="ember-view ember-text-field"
       type="text" data-toggle="tooltip" data-placement="bottom" title="Name">
```