Модель — класс, который определяет свойства и поведение предоставляемых пользователю данных. Все, что пользователь ожидает увидеть, если он покинет приложение и вернется (или если обновит страницу), должно быть представлено моделью.

Если вы хотите получить новую модель для приложения, вам нужно создать в папке моделей новый файл, который расширяет `DS.Model`. Гораздо удобнее использовать одну из генерирующих команд Ember CLI. Например, давайте создадим модель `person`:

```bash
ember generate model person
```

Это сгенерирует следующий файл:

`app/models/person.js`
```js
export default DS.Model.extend({
});
```

Когда вы определили класс модели, можно начинать [поиск](http://guides.emberjs.com/v2.1.0/models/finding-records) и [работать с записями](http://guides.emberjs.com/v2.1.0/models/creating-updating-and-deleting-records) этого типа.

## Определение атрибутов

Модель `person`, которую создали ранее, не имела каких-либо атрибутов. Давайте добавим ей имя, фамилию и дату рождения с помощью `DS.attr`:

`app/models/person.js`
```js
export default DS.Model.extend({
  firstName: DS.attr(),
  lastName: DS.attr(),
  birthday: DS.attr()
});
```

Атрибуты используются, когда возвращенная с сервера полезная нагрузка JSON превращается в запись, и когда запись преобразуется в последовательную форму для сохранения на сервере после ее изменения.

Вы можете использовать атрибуты, как и любое другое свойство, в том числе как часть вычислительного свойства. Зачастую нужно определять вычислительные свойства, которые комбинируют или преобразуют примитивные атрибуты.

`app/models/person.js`
```js
export default DS.Model.extend({
  firstName: DS.attr(),
  lastName: DS.attr(),

  fullName: Ember.computed('firstName', 'lastName', function() {
    return `${this.get('firstName')} ${this.get('lastName')}`;
  })
});
```

Про добавление классам вычислительных свойств, можно почитать в разделе [Вычислительные свойства](http://emjs.ru/v2.0.0/object-model/computed-properties/).

### Преобразования

Вы можете обнаружить, что тип атрибута, который вернул сервер, не соответствует типу, который вы хотели бы видеть в коде JavaScript. Ember Data позволяет определять для типов атрибутов простые методы сериализации и десериализации, которые называют преобразованиями. Если вы хотите запустить преобразование для атрибута, то предоставьте имя преобразования в качестве первого аргумента для метода `DS.attr`.

Например, если вы хотите преобразовать строку [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) в объект данных JavaScript, то вам следует определить атрибут таким образом:

`app/models/person.js`
```js
export default DS.Model.extend({
  birthday: DS.attr('date')
});
```

Ember Data поддерживает следующие типы атрибута: `string`, `number`, `boolean` и `date`. Они приводят значение к типу JavaScript, который соответствует их имени.

Преобразования необязательны. Если вы не указываете имя преобразования, Ember Data не выполнит дополнительную обработку значения.

### Параметры

`DS.attr` также может принимать хеш параметров в качестве второго аргумента. Сейчас единственный доступный параметр — `defaultValue`. Он может использовать строку или функцию, чтобы установить исходное значение атрибута, если оно не предоставлено.

В следующем примере мы определяем, что `verified` имеет исходное значение `false`, а исходные значения `createdAt` присваиваются текущей дате в момент создания модели:

`app/models/user.js`
```js
export default DS.Model.extend({
  username: DS.attr('string'),
  email: DS.attr('string'),
  verified: DS.attr('boolean', { defaultValue: false }),
  createdAt: DS.attr('string', {
    defaultValue() { return new Date(); }
  })
});
```

## ОПРЕДЕЛЕНИЕ СВЯЗЕЙ

Ember Data включает несколько встроенных типов связей, чтобы помочь вам определить, в каких отношениях находятся модели.

### ОДИН К ОДНОМУ

Чтобы объявить связь «один к одному» между двумя моделями, используйте `DS.belongsTo`:

`app/models/user.js`
```js
export default DS.Model.extend({
  profile: DS.belongsTo('profile')
});
```

`app/models/profile.js`
```js
export default DS.Model.extend({
  user: DS.belongsTo('user')
});
```

### ОДИН КО МНОГИМ

Чтобы объявить связь «один ко многим» между двумя моделями, используйте `DS.belongsTo` в сочетании с `DS.hasMany`. Примерно так:

`app/models/post.js`
```js
export default DS.Model.extend({
  comments: DS.hasMany('comment')
});
```

`app/models/comment.js`
```js
export default DS.Model.extend({
  post: DS.belongsTo('post')
});
```

### МНОГИЕ КО МНОГИМ

Чтобы объявить связь «многие ко многим» между двумя моделями, используйте `DS.hasMany`:

`app/models/post.js`
```js
export default DS.Model.extend({
  tags: DS.hasMany('tag')
});
```

`app/models/tag.js`
```js
export default DS.Model.extend({
  posts: DS.hasMany('post')
});
```

### Явные инверсии

Ember Data сделает все возможное, чтобы обнаружить, какие связи соответствуют друг другу. В вышеуказанном коде «один ко многим», например, Ember Data может выяснить, что изменение связи `comments` должно обновить связь `post` на обратной стороне, так как `post` — единственная связь для этой модели.

Но иногда имеются многочисленные `belongsTo`/`hasMany` для одного типа. Вы можете указать, какое свойство на соответствующей модели служит инверсией, с помощью параметра `inverse` для `DS.belongsTo` или `DS.hasMany`. Связи без инверсии также можно отметить, если включить `{ inverse: null }`.

`app/models/comment.js`
```js
export default DS.Model.extend({
  onePost: DS.belongsTo('post', { inverse: null }),
  twoPost: DS.belongsTo('post'),
  redPost: DS.belongsTo('post'),
  bluePost: DS.belongsTo('post')
});
```

`app/models/post.js`
```js
export default DS.Model.extend({
  comments: DS.hasMany('comment', {
    inverse: 'redPost'
  })
});
```

### Обратное отношение

Если вы хотите определить обратное отношение (модель связана сама с собой), то должны явно определить обратную связь. Если обратной связи нет, то вы можете установить `null` для инверсии.

Здесь пример обратного отношения один ко многим:

`app/models/folder.js`
```js
export default DS.Model.extend({
  children: DS.hasMany('folder', { inverse: 'parent' }),
  parent: DS.belongsTo('folder', { inverse: 'children' })
});
```

Здесь пример обратного отношения один к одному:

`app/models/user.js`
```js
export default DS.Model.extend({
  name: DS.attr('string'),
  bestFriend: DS.belongsTo('user', { inverse: 'bestFriend' }),
});
```

Вы также можете определить обратное отношение без инверсии:

`app/models/folder.js`
```js
export default DS.Model.extend({
  parent: DS.belongsTo('folder', { inverse: null })
});
```

### Вложенные данные только для чтения

Некоторые модели могут иметь свойства, которые вложены глубоко в объекты данных, предназначенных только для чтения. Исходным решением будет определить модели для каждого вложенного объекта и использовать `hasMany` и `belongsTo`, чтобы заново создать вложенную связь. Но так как данные только для чтения никогда не понадобится обновлять и сохранять, это зачастую приводит к созданию большого количества кода с очень малой пользой. Альтернативный подход: определить эти связи с помощью атрибута без преобразования (`DS.attr()`). Так проще получить доступ к предназначенным только для чтения значениям в вычислительных свойствах и шаблонах без издержек на определение посторонних моделей.