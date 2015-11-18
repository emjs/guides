## Помощники ввода данных

Помощники `{{input}}` и `{{textarea}}` в Ember.js позволяют самым простым путем создавать элементы управления стандартной формой. Помощник `{{input}}` охватывает встроенные представления [Ember.TextField](http://emberjs.com/api/classes/Ember.TextField.html) и [Ember.Checkbox](http://emberjs.com/api/classes/Ember.Checkbox.html), а `{{textarea}}` — [Ember.TextArea](http://emberjs.com/api/classes/Ember.TextArea.html). С их помощью можно создавать представления, которые объявляются почти идентично созданию стандартных элементов `<input>` или `<textarea>`.

### Текстовые поля

```handlebars
{{input value="http://www.facebook.com"}}
```

Станет:

```html
<input type="text" value="http://www.facebook.com"/>
```

Вы можете передавать следующие стандартные атрибуты `<input>` в рамках помощника ввода данных:

<table>
  <tr><td>`readonly`</td><td>`required`</td><td>`autofocus`</td></tr>
  <tr><td>`value`</td><td>`placeholder`</td><td>`disabled`</td></tr>
  <tr><td>`size`</td><td>`tabindex`</td><td>`maxlength`</td></tr>
  <tr><td>`name`</td><td>`min`</td><td>`max`</td></tr>
  <tr><td>`pattern`</td><td>`accept`</td><td>`autocomplete`</td></tr>
  <tr><td>`autosave`</td><td>`formaction`</td><td>`formenctype`</td></tr>
  <tr><td>`formmethod`</td><td>`formnovalidate`</td><td>`formtarget`</td></tr>
  <tr><td>`height`</td><td>`inputmode`</td><td>`multiple`</td></tr>
  <tr><td>`step`</td><td>`width`</td><td>`form`</td></tr>
  <tr><td>`selectionDirection`</td><td>`spellcheck`</td><td>&nbsp;</td></tr>
</table>

Если эти атрибуты стоят в строке в кавычках, то их значения будут напрямую установлены элементу, как в предыдущем примере. Однако, без кавычек эти значения будут привязаны к свойству в текущем отображенном контексте шаблона. Например:

```handlebars
{{input type="text" value=firstName disabled=entryNotAllowed size="50"}}
```

привяжет атрибут `disabled` к значению `entryNotAllowed` в текущем контексте.

### Действия

Чтобы назначить действие для специфических событий вроде `enter` или `key-press`, используйте следующее:

```handlebars
{{input value=firstName key-press="updateFirstName"}}
```

[Имена события](http://emberjs.com/api/classes/Ember.View.html#toc_event-names) нужно записывать с тире.

### Чекбоксы

Вы также можете использовать помощника `{{input}}`, чтобы создать чекбокс, если установите его `type`:

```handlebars
{{input type="checkbox" name="isAdmin" checked=isAdmin}}
```

Чекбоксы поддерживают следующие свойства:

* `checked`
* `disabled`
* `tabindex`
* `indeterminate`
* `name`
* `autofocus`
* `form`

В предыдущих разделах описано, как их можно привязать или назначить.

### Текстовые области

```handlebars
{{textarea value=name cols="80" rows="6"}}
```

Свяжет значение текстовой области с `name` в текущем контексте.

`{{textarea}}` поддерживает привязку и/или установку следующих свойств:

* `value`
* `name`
* `rows`
* `cols`
* `placeholder`
* `disabled`
* `maxlength`
* `tabindex`
* `selectionEnd`
* `selectionStart`
* `selectionDirection`
* `wrap`
* `readonly`
* `autofocus`
* `form`
* `spellcheck`
* `required`