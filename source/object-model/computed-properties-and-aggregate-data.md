Бывают случаи, что у вас есть вычислительные свойства, чьи значения зависят от свойств элементов в массиве. Например, у вас может быть массив элементов списка дел, и вы хотите вычислить, сколько из них остались незавершенными на основе свойства `isDone`.

Чтобы облегчить процесс, Ember предоставляет ключ `@each`, показанный ниже:

`app/components/todos.js`
```js
export default Ember.Component.extend({
  todos: [
    Ember.Object.create({ isDone: true }),
    Ember.Object.create({ isDone: false }),
    Ember.Object.create({ isDone: true })
  ],

  remaining: Ember.computed('todos.@each.isDone', function() {
    var todos = this.get('todos');
    return todos.filterBy('isDone', false).get('length');
  })
});
```

Здесь зависимый ключ `todos.@each.isDone` указывает Ember.js обновить привязки и запустить наблюдателей, когда произойдет любое из следующих событий:

1. Меняется свойство `isDone` любых объектов в массиве `todos`.
2. В массив `todos` добавляется элемент.
3. Из массива `todos` удаляется элемент.
4. Свойство `todos` компонента переходит к другому массиву.

В примере выше подсчет `remaining` составляет `1`:

```js
import TodosComponent from 'app/components/todos';

let todosComponent = TodosComponent.create();
todosComponent.get('remaining');
// 1
```

Если мы изменим свойство `isDone` в списке, свойство `remaining` автоматически обновится:

```js
let todos = todosComponent.get('todos');
let todo = todos.objectAt(1);
todo.set('isDone', true);

todosComponent.get('remaining');
// 0

todo = Ember.Object.create({ isDone: false });
todos.pushObject(todo);

todosComponent.get('remaining');
// 1
```

Обратите внимание, что `@each` действует только в пределах одного уровня. Вы не можете использовать вложенные формы вроде `todos.@each.owner.name` или `todos.@each.owner.@each.name`.

Бывает, что вас не волнует, меняются ли свойства индивидуальных элементов массива. В таком случае используйте ключ `[]` вместо `@each`. Вычислительные свойства, которые зависят от массива и используют ключ `[]`, обновятся только в том случае, если элементы будут добавлены в массив или удалены из него, или если свойство массива будет установлено на другой массив. Например:

`app/components/todos.js`
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

Здесь `indexOfSelectedTodo` зависит от `todos.[]`, поэтому он обновится, если мы добавим элемент к `todos`, но не обновится, если изменится значение `isDone` в `todo`.

Несколько макросов [Ember.computed](http://emberjs.com/api/classes/Ember.computed.html) используют ключ `[]` для реализации обычных случаев использования. Например, чтобы создать вычислительное свойство, которое преобразует свойства из массива, можно использовать [Ember.computed.map](http://emberjs.com/api/classes/Ember.computed.html#method_map) или создать само вычислительное свойство:

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

Для сравнения, использование вычислительного макроса обобщает часть кода:

```js
const Hamster = Ember.Object.extend({
  excitingChores: Ember.computed.map('chores', function(chore, index) {
    return `CHORE ${index}: ${chore.toUpperCase()}!`;
  })
});
```

Вычислительные макросы ожидают, что вы будете использовать массив, поэтому нет нужды в таких случаях применять ключ `[]`. Но при создании собственного вычислительного свойства необходимо указать Ember, что оно будет следить за изменениями в массиве. Вот здесь и пригодится ключ `[]`.