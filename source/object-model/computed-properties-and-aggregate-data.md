Иногда значение вычисляемого свойства зависит от свойств элементов в массиве. Например, у вас есть массив, который представляет собой список дел, и вы хотите вычислить количество незавершенных пунктов по их свойству `isDone`.

Для решения этой задачи Ember предоставляет ключ `@each`, показанный ниже:

`app/components/todo-list.js`
```js
export default Ember.Component.extend({
  todos: [
    Ember.Object.create({ isDone: true }),
    Ember.Object.create({ isDone: false }),
    Ember.Object.create({ isDone: true })
  ],

  incomplete: Ember.computed('todos.@each.isDone', function() {
    var todos = this.get('todos');
    return todos.filterBy('isDone', false);
  })
});
```

Здесь зависимый ключ `todos.@each.isDone` указывает Ember.js обновить привязки и запустить наблюдателя, когда произойдет любое из следующих событий:

1. Меняется свойство `isDone` любых объектов в массиве `todos`.
2. В массив `todos` добавляется элемент.
3. Из массива `todos` удаляется элемент.
4. Свойство `todos` компонента используется в другом массиве.

В Ember есть макрос вычисляемого свойства [`computed.filterBy`](http://emberjs.com/api/classes/Ember.computed.html#method_filterBy), который служит сокращенным вариантом свойства выше:

`app/components/todo-list.js`
```js
export default Ember.Component.extend({
  todos: [
    Ember.Object.create({ isDone: true }),
    Ember.Object.create({ isDone: false }),
    Ember.Object.create({ isDone: true })
  ],

  incomplete: Ember.computed.filterBy('todos', 'isDone', false)
});
```

В обоих примерах выше `incomplete` — массив, содержащий одну незавершенную задачу:

```js
import TodoListComponent from 'app/components/todo-list';

let todoListComponent = TodoListComponent.create();
todoListComponent.get('incomplete.length');
// 1
```

Если мы изменим свойство `isDone` в списке, свойство `incomplete` автоматически обновится:

```js
let todos = todoListComponent.get('todos');
let todo = todos.objectAt(1);
todo.set('isDone', true);

todoListComponent.get('incomplete.length');
// 0

todo = Ember.Object.create({ isDone: false });
todos.pushObject(todo);

todoListComponent.get('incomplete.length');
// 1
```

Обратите внимание, что `@each` действует только в пределах одного уровня. Вы не можете использовать вложенные формы вроде `todos.@each.owner.name` или `todos.@each.owner.@each.name`.

Не всегда нужно отслеживать изменения свойств отдельных элементов массива. В таком случае используйте ключ `[]` вместо `@each`. Вычисляемые свойства, которые зависят от массива, при использовании ключа `[]` обновятся только в том случае, если элементы будут добавлены в массив или удалены из него, или если свойство массива будет использовано в другом массиве. Например:

`app/components/todo-list.js`
```js
export default Ember.Component.extend({
  todos: [
    Ember.Object.create({ isDone: true }),
    Ember.Object.create({ isDone: false }),
    Ember.Object.create({ isDone: true })
  ],

  selectedTodo: null,
  indexOfSelectedTodo: Ember.computed('selectedTodo', 'todos.[]', function() {
    return this.get('todos').indexOf(this.get('selectedTodo'));
  })
});
```

Здесь `indexOfSelectedTodo` зависит от `todos.[]`, поэтому оно обновится, если мы добавим элемент к `todos`, но не обновится, если изменится значение `isDone` в `todo`.

В нескольких макросах [Ember.computed](http://emberjs.com/api/classes/Ember.computed.html) применяется ключ `[]` для реализации частых случаев использования. Например, чтобы создать вычисляемое свойство, которое сопоставляет свойства из массива, можно использовать [Ember.computed.map](http://emberjs.com/api/classes/Ember.computed.html#method_map) или создать само вычисляемое свойство:

```js
const Hamster = Ember.Object.extend({
  excitingChores: Ember.computed('chores.[]', function() {
    return this.get('chores').map(function(chore, index) {
      return `CHORE ${index}: ${chore.toUpperCase()}!`;
    });
  })
});

const hamster = Hamster.create({
  chores: ['clean', 'write more unit tests']
});

hamster.get('excitingChores'); // ['CHORE 1: CLEAN!', 'CHORE 2: WRITE MORE UNIT TESTS!']
hamster.get('chores').pushObject('review code');
hamster.get('excitingChores'); // ['CHORE 1: CLEAN!', 'CHORE 2: WRITE MORE UNIT TESTS!', 'CHORE 3: REVIEW CODE!']
```

Для сравнения, при использовании макроса вычисляемого свойства часть кода убирается:

```js
const Hamster = Ember.Object.extend({
  excitingChores: Ember.computed.map('chores', function(chore, index) {
    return `CHORE ${index}: ${chore.toUpperCase()}!`;
  })
});
```

В макросах вычисляемых свойств учитывается, что вы будете использовать массив, поэтому в таких случаях нет нужды применять ключ `[]`. Но при создании собственного вычисляемого свойства необходимо указать Ember.js, что оно отслеживает изменения в массиве. Вот здесь и пригодится ключ `[]`.