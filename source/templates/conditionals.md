Операторы [`if`](emberjs.com/api/classes/Ember.Templates.helpers.html#method_if) и [`unless`](emberjs.com/api/classes/Ember.Templates.helpers.html#method_unless) реализованы как встроенные хелперы. Хелперы можно вызвать тремя способами, каждый из которых проиллюстрирован ниже с условными конструкциями.

Первый стиль вызова — **строковый**. Это похоже на отображение свойства, но хелперы принимают аргументы. Например:

```hbs
<div>
  {{if isFast "zoooom" "putt-putt-putt"}}
</div>
```

[`{{if}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_if) в этом случае возвращает `"zoooom"`, когда `isFast` имеет значение true, и `"putt-putt-putt"`, когда `isFast` равен false. Хелперы, вызванные как строковые выражения, отображают одно значение, как и в случае со свойствами.

Строковые хелперы не нужно использовать внутри тегов HTML. Их можно использовать внутри значений атрибутов:

```hbs
<div class="is-car {{if isFast "zoooom" "putt-putt-putt"}}">
</div>
```

**Вложенный вызов** — еще один способ использовать хелпер. Как и строковые хелперы, вложенные генерируют и возвращают одно значение. Например, этот шаблон отображает `"zoooom"`, если `isFast` и `isFueled` имеют значение true:

```hbs
<div>
  {{if isFast (if isFueled "zoooom")}}
</div>
```

Сначала вызывается вложенный хелпер, который возвращает только `"zoooom"`, если `isFueled` - true.
Затем вызывается строковое выражение, которое отображает значение вложенного хелпера (`"zoooom"`), только если `isFast` - true.

Третья форма использования хелпера — **блочный вызов**. При использовании блочных хелперов отображается только часть шаблона. Блочный вызов хелпера можно распознать по знаку `#` перед именем хелпера и закрывающим двойным фигурным скобкам `{{/` в конце вызова.

Например, этот шаблон условно показывает свойства `person`, только если объект представлен:

```hbs
{{#if person}}
  Welcome back, <b>{{person.firstName}} {{person.lastName}}</b>!
{{/if}}
```

[`{{if}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_if) проверяет истину, то есть все значения кроме `false`, `undefined`, `null`, `''`, `0` или `[]` (иными словами, любых ложных значений JavaScript или пустого массива).

Если значение, переданное `{{#if}}`, определяется как ложное, то отображается блок `{{else}}` этого вызова:

```hbs
{{#if person}}
  Welcome back, <b>{{person.firstName}} {{person.lastName}}</b>!
{{else}}
  Please log in.
{{/if}}
```

`{{else}}` может связать в цепочку вызов помощника. Чаще всего для этого используется `{{else if}}`:

```hbs
{{#if isAtWork}}
  Ship that code!
{{else if isReading}}
  You can finish War and Peace eventually...
{{/if}}
```

Противоположность `{{if}}` — [`{{unless}}`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_unless), который можно использовать в тех же трех стилях вызова. Например, этот шаблон показывает сумму, когда пользователь не заплатил:

```hbs
{{#unless hasPaid}}
  You owe: ${{total}}
{{/unless}}
```