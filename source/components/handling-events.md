В компоненте через обработчик событий вы можете реагировать на действия пользователя вроде двойного нажатия мышкой, наведения курсора и нажатия на клавиши. Просто реализуйте имя события, на которое хотите среагировать, в качестве метода в компоненте.

Например, представьте, что у нас есть такой шаблон:

```hbs
{{#double-clickable}}
  This is a double clickable area!
{{/double-clickable}}
```

Давайте реализуем `double-clickable` так, чтобы при нажатии отобразилось предупреждение:

`app/components/double-clickable.js`
```js
export default Ember.Component.extend({
  doubleClick: function() {
    alert("DoubleClickableComponent was clicked!");
  }
});
```

События браузера могут распространиться по модели DOM. Они последовательно нацеливаются на родительские компоненты. Чтобы разрешить распространение, верните `true` из метода обработчика события в компоненте.

`app/components/double-clickable.js`
```js
export default Ember.Component.extend({
  doubleClick: function() {
    Ember.Logger.info("DoubleClickableComponent was clicked!");
    return true;
  }
});
```

Просмотрите список имен событий в конце страницы. Любое событие можно определить как обработчика событий в компоненте.

## Отправка действий

В некоторых случаях компоненту нужно определить обработчик события, чтобы поддерживать различное поведение перетаскивания. Например, компоненту нужно отправить `id`, когда он получает событие перетаскивания:

```hbs
{{drop-target action="didDrop"}}
```

Вы можете определить обработчик события компонента, чтобы управлять событием перетаскивания. И если вам нужно, то можете также остановить распространение события с помощью `return false;`.

`app/components/drop-target.js`
```js
export default Ember.Component.extend({
  attributeBindings: ['draggable'],
  draggable: 'true',

  dragOver: function() {
    return false;
  },

  drop: function(event) {
    let id = event.dataTransfer.getData('text/data');
    this.sendAction('action', id);
  }
});
```

## Имена событий

Описанные выше примеры обработки событий реагируют на один набор событий. Имена встроенных событий перечислены ниже. Индивидуальные события можно зарегистрировать с помощью [Ember.Application.customEvents](http://emberjs.com/api/classes/Ember.Application.html#property_customEvents).

События касания:

* `touchStart`
* `touchMove`
* `touchEnd`
* `touchCancel`

События клавиатуры:

* `keyDown`
* `keyUp`
* `keyPress`

События мыши:

* `mouseDown`
* `mouseUp`
* `contextMenu`
* `click`
* `doubleClick`
* `mouseMove`
* `focusIn`
* `focusOut`
* `mouseEnter`
* `mouseLeave`

События формы:

* `submit`
* `change`
* `focusIn`
* `focusOut`
* `input`

События перетаскивания в HTML5:

* `dragStart`
* `drag`
* `dragEnter`
* `dragLeave`
* `dragOver`
* `dragEnd`
* `drop`