Если вам нужно отобразить все ключи или значения объекта JavaScript в шаблоне, то можно использовать хелпер [`{{#each-in}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_each-in):

`/app/components/store-categories.js`
```js
import Ember from 'ember';

export default Ember.Component.extend({
  willRender() {
    // Set the "categories" property to a JavaScript object
    // with the category name as the key and the value a list
    // of products.
    this.set('categories', {
      'Bourbons': ['Bulleit', 'Four Roses', 'Woodford Reserve'],
      'Ryes': ['WhistlePig', 'High West']
    });
  }
});
```

`/app/templates/components/store-categories.hbs`
```hbs
<ul>
  {{#each-in categories as |category products|}}
    <li>{{category}}
      <ol>
        {{#each products as |product|}}
          <li>{{product}}</li>
        {{/each}}
      </ol>
    </li>
  {{/each-in}}
</ul>
```

Шаблон внутри блока `{{#each-in}}` повторяется один раз для каждого ключа в переданном объекте. Первый блочный параметр (`category` в примере выше) — ключ для этой итерации, а второй блочный параметр (`products`) — фактическое значение этого ключа.

Пример выше покажет подобный список:

```
<ul>
  <li>Bourbons
    <ol>
      <li>Bulleit</li>
      <li>Four Roses</li>
      <li>Woodford Reserve</li>
    </ol>
  </li>
  <li>Ryes
    <ol>
      <li>WhistlePig</li>
      <li>High West</li>
    </ol>
  </li>
</ul>
```

## Повторное отображение

Хелпер [`{{#each-in}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_each-in) **не отслеживает изменения свойства** в переданном ему объекте. В примере выше, если вы добавите ключ к свойству компонента `categories` после того, как отобразится компонент, шаблон автоматически **не** обновится.

`/app/components/store-categories.js`
```js
import Ember from 'ember';

export default Ember.Component.extend({
  willRender() {
    this.set('categories', {
      'Bourbons': ['Bulleit', 'Four Roses', 'Woodford Reserve'],
      'Ryes': ['WhistlePig', 'High West']
    });
  },

  actions: {
    addCategory(category) {
      // This won't work!
      let categories = this.get('categories');
      categories[category] = [];
    }
  }
});
```

Чтобы повторно отобразить компонент после добавления, удаления или изменения свойства из объекта, нужно снова назначить свойство компонента с помощью [`set()`](http://emberjs.com/api/classes/Ember.Component.html#method_set) или вручную запустить повторное отображение через [`rerender()`](http://emberjs.com/api/classes/Ember.Component.html#method_rerender):

`/app/components/store-categories.js`
```js
import Ember from 'ember';

export default Ember.Component.extend({
  willRender() {
    this.set('categories', {
      'Bourbons': ['Bulleit', 'Four Roses', 'Woodford Reserve'],
      'Ryes': ['WhistlePig', 'High West']
    });
  },

  actions: {
    addCategory(category) {
      let categories = this.get('categories');
      categories[category] = [];

      // A manual re-render causes the DOM to be updated
      this.rerender();
    }
  }
});
```

## Упорядочение

Ключи объекта будут представлены в том же порядке, что и в возвращенном массиве после вызова `Object.keys` для этого объекта. Если вам нужен другой порядок, то следует использовать `Object.keys`, чтобы получить массив, отсортировать его с помощью встроенных инструментов JavaScript и использовать хелпер [`{{#each}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_each-in).

## Пустые списки
Хелпер [`{{#each-in}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_each-in) может иметь соответствующий `{{else}}`. Если объект пустой, несуществующий или неопределенный, то будет отображено содержимое этого блока:

```hbs
{{#each-in people as |name person|}}
  Hello, {{name}}! You are {{person.age}} years old.
{{else}}
  Sorry, nobody is here.
{{/each-in}}
```