Хелперы [`{{input}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_input) и [`{{textarea}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_textarea) в Ember.js позволяют с легкостью создавать элементы управления стандартной формой. Хелпер `{{input}}` охватывает встроенные представления [Ember.TextField](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_textarea) и [Ember.Checkbox](http://emberjs.com/api/classes/Ember.Checkbox.html), а `{{textarea}}` — [Ember.TextArea](http://emberjs.com/api/classes/Ember.Checkbox.html). С их помощью можно создавать представления, которые объявляются почти идентично созданию стандартных элементов `<input>` или `<textarea>`.

## Текстовые поля

```hbs
{{input value="http://www.facebook.com"}}
```

станет:

```
<input type="text" value="http://www.facebook.com"/>
```

Вы можете передавать следующие стандартные атрибуты `<input>` в рамках хелпера input:

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
  <tr><td>`selectionDirection`</td><td>`spellcheck`</td><td>`type`</td></tr>
</table>

Если эти атрибуты стоят в строке в кавычках, их значения будут напрямую назначены элементу, как в предыдущем примере. Но без кавычек эти значения будут привязаны к свойству в текущем отображенном контексте шаблона. Например:

```hbs
{{input type="text" value=firstName disabled=entryNotAllowed size="50"}}
```

привяжет атрибут `disabled` к значению `entryNotAllowed` в текущем контексте.

## Действия

Чтобы назначить действие для специфических событий, например `enter` или `key-press`, используйте следующее:

```hbs
{{input value=firstName key-press="updateFirstName"}}
```

[Имена события](emberjs.com/api/classes/Ember.View.html#toc_event-names) нужно записывать с тире.

## Чекбоксы

Вы также можете использовать хелпер [`{{input}}`](emberjs.com/api/classes/Ember.Templates.helpers.html#method_input), чтобы создать чекбокс, если установите его `type`:

```hbs
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

## Текстовые области

```hbs
{{textarea value=name cols="80" rows="6"}}
```

свяжет значение текстовой области с `name` текущего контекста.

[`{{textarea}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_textarea) поддерживает привязку и/или назначение следующих свойств:

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

## Привязка динамического атрибута

В некоторых случаях вам будет нужно динамически привязывать свойство к входным данным. Например, когда вы создаете свободную форму. Чтобы добиться этого, вам нужно использовать [`{{get}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_get) и [`{{mut}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_mut) в связке, как показано в следующем примере:

```hbs
{{input value=(mut (get person field))}}
```

Хелпер `{{get}}` позволяет динамически указывать, какое свойство привязывать. А хелпер `{{mut}}` позволяет обновлять привязку в соответствии с входными данными. Чтобы получить больше информации, смотрите соответствующую документацию о хелперах.