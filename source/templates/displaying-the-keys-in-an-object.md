Если вам нужно отобразить все ключи или значения объекта JavaScript в шаблоне, то можно использовать помощника [`{{#each-in}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_each-in):

`/app/components/store-categories.js`
```javascript
import Component from "ember-component";

export Component.extend({
  willRender() {
    // Set the "categories" property to a JavaScript object
    // with the category name as the key and the value a list
    // of products.
    this.set('categories', {
      "Bourbons": ["Bulleit", "Four Roses", "Woodford Reserve"],
      "Ryes": ["WhistlePig", "High West"]
    });
  }
});
```

`/app/templates/components/store-categories.hbs`
```handlebars
<ul>
  {{#each-in categories as |category, products|}}
    <li>{{category}}
      <ol>
        {{#each products key="@item" as |product|}}
          <li>{{product}}</li>
        {{/each}}
      </ol>
    </li>
  {{/each}}
</ul>
```

Шаблон внутри блока `{{#each-in}}` повторяется для каждого ключа в переданном объекте. Первый блок с параметром (`category` в примере выше) — ключ для этой итерации, а второй блок с параметром (`product`) — фактическое значение этого ключа.

Пример выше покажет подобный список:

```html
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

### Повторное отображение

Помощник [`{{#each-in}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_each-in) **не отслеживает изменения свойства** в объекте, который ему передается. В примере выше, если вы добавите ключ к свойству компонента `categories` после того, как отобразится компонент, шаблон автоматически **не** обновится.

`/app/components/store-categories.js`
```javascript
export Component.extend({
  willRender() {
    this.set('categories', {
      "Bourbons": ["Bulleit", "Four Roses", "Woodford Reserve"],
      "Ryes": ["WhistlePig", "High West"]
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

Чтобы компонент отобразился по-новому после добавления, удаления или изменения свойства из объекта, нужно выполнить [`set()`](http://emberjs.com/api/classes/Ember.Component.html#method_set) свойство компонента еще раз или вручную запустить повторное отображение через [`rerender()`](http://emberjs.com/api/classes/Ember.Component.html#method_rerender):

`/app/components/store-categories.js`
```javascript
export Component.extend({
  willRender() {
    this.set('categories', {
      "Bourbons": ["Bulleit", "Four Roses", "Woodford Reserve"],
      "Ryes": ["WhistlePig", "High West"]
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

### Упорядочение

Ключи объекта будут представлены в том же порядке, что и в возвращенном массиве после вызова `Object.keys` на этом объекте. Если вам нужен другой порядок, то следует использовать `Object.keys`, чтобы получить массив, отсортировать его с помощью встроенных инструментов JavaScript и использовать помощника [`{{#each}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_each-in).

### Пустые списки

Помощник [`{{#each-in}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_each-in) может иметь сопоставление `{{else}}`. Если объект пустой, несуществующий или неопределенный, то будет отображено содержимое этого блока:

```handlebars
{{#each-in people as |name, person|}}
  Hello, {{name}}! You are {{person.age}} years old.
{{else}}
  Sorry, nobody is here.
{{/each}}
```