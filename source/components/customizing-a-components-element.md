По умолчанию, каждый компонент поддерживается элементом `<div>`. Если бы вы хотели взглянуть на отображенный компонент в инструментах разработчика, то увидели представление DOM, которое выглядело примерно так:

```html
<div id="ember180" class="ember-view">
  <h1>My Component</h1>
</div>
```

Вы можете индивидуально настроить, какой тип элемента Ember сгенерирует для компонента, включая его атрибуты и имена класса, если создадите подкласс `Ember.Component` в JavaScript.

### Настройка элемента

Чтобы использовать какой-либо тег кроме `div`, создайте подкласс `Ember.Component` и назначьте ему свойство `tagName`. Это свойство может быть любым подходящим именем тегов HTML5 в виде строки:

`app/components/navigation-bar.js`
```js
export default Ember.Component.extend({
  tagName: 'nav'
});
```

`app/templates/components/navigation-bar.hbs`
```hbs
<ul>
  <li>{{#link-to "home"}}Home{{/link-to}}</li>
  <li>{{#link-to "about"}}About{{/link-to}}</li>
</ul>
```

### Настройка имен класса

Вы можете также указать, какие имена класса применять к элементу компонента, если назначите его свойству `classNames` массив строк:

`app/components/navigation-bar.js`
```js
export default Ember.Component.extend({
  classNames: ['primary']
});
```

Если вы хотите, чтобы имена класса определялись свойствами компонента, то можете использовать привязки имени класса. Если вы осуществите привязку к логическому свойству, то имя класса будет добавлено или удалено в зависимости от значения:

`app/components/todo-item.js`
```js
export default Ember.Component.extend({
  classNameBindings: ['isUrgent'],
  isUrgent: true
});
```

Этот компонент отобразит следующее:

```html
<div class="ember-view is-urgent"></div>
```

Если `isUrgent` изменится на `false`, тогда имя класса `is-urgent` будет удалено.

По умолчанию, имя логического свойства пишется с тире. Вы можете настроить имя класса, указав его через двоеточие:

`app/components/todo-item.js`
```js
export default Ember.Component.extend({
  classNameBindings: ['isUrgent:urgent'],
  isUrgent: true
});
```

Это отобразит такой код HTML:

```html
<div class="ember-view urgent">
```

Кроме индивидуального имени класса для значения `true`, вы можете указать имя класса, которое будет использоваться при значении `false`:

`app/components/todo-item.js`
```js
export default Ember.Component.extend({
  classNameBindings: ['isEnabled:enabled:disabled'],
  isEnabled: false
});
```

Это покажет следующий код HTML:

```html
<div class="ember-view disabled">
```

Вы также можете указать единственный класс, который будет добавлен, когда свойство имеет значение `false`, если объявите `classNameBindings` таким образом:

`app/components/todo-item.js`
```js
export default Ember.Component.extend({
  classNameBindings: ['isEnabled::disabled'],
  isEnabled: false
});
```

Это отобразит следующий код HTML:

```html
<div class="ember-view disabled">
```

Если свойство `isEnabled` имеет значение `true`, имя класса не добавится:

```html
<div class="ember-view">
```

Если привязанное значение свойства — это строка, то значение будет добавлено в качестве имени класса без изменения:

`app/components/todo-item.js`
```js
export default Ember.Component.extend({
  classNameBindings: ['priority'],
  priority: 'highestPriority'
});
```

Это покажет такой код HTML:

```html
<div class="ember-view highestPriority">
```

### Настройка атрибутов

С помощью `attributeBindings` вы можете привязать атрибуты к элементу DOM, который представляет компонент: 

`app/components/link-item.js`
```js
export default Ember.Component.extend({
  tagName: 'a',
  attributeBindings: ['href'],
  href: "http://emberjs.com"
});
```

Также можно привязать эти атрибуты к по-разному именованным свойствам:

`app/components/link-item.js`
```js
export default Ember.Component.extend({
  tagName: 'a',
  attributeBindings: ['customHref:href'],
  customHref: 'http://emberjs.com'
});
```