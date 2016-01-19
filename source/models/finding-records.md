Хранилище Ember Data предоставляет интерфейс для извлечения записей одного типа.

### Извлечение одной записи

Используйте `store.findRecord()`, чтобы извлекать запись по ее типу и ID. Этот код вернет обещание, которое завершится запрошенной записью:

```js
var post = this.store.findRecord('post', 1); // => GET /posts/1
```

Используйте `store.peekRecord()`, чтобы извлечь запись по ее типу и ID без сетевого запроса. Этот код вернет запись, если она уже находится в хранилище:

```js
var post = this.store.peekRecord('post', 1); // => no network request
```

### Извлечение нескольких записей

Используйте `store.findAll()`, чтобы извлечь все записи для данного типа:

```js
var posts = this.store.findAll('post'); // => GET /posts
```

Используйте `store.peekAll()`, чтобы извлечь без сетевого запроса все записи для данного типа, которые уже загружены в хранилище:

```js
var posts = this.store.peekAll('post'); // => no network request
```

`store.findAll()` возвращает `DS.PromiseArray`, который соответствует `DS.RecordArray`, и `store.peekAll` напрямую возвращает `DS.RecordArray`.

Важно отметить, что `DS.RecordArray` — не массив JavaScript. Это объект, который реализует `Ember.Enumerable`. Это важно, так как если вы, например, хотите извлечь записи по индексу, обозначение `[]` не сработает. Вместо этого вам будет нужно использовать `objectAt(index)`.

### Запрос для нескольких записей

Ember Data дает возможность отправлять запрос для записей, которые соответствуют определенному критерию. Вызов `store.query()` выполнит запрос `GET` с переданным объектом, сериализованным как параметры запросов. Этот метод возвращает `DS.PromiseArray` таким же образом, как `find`.

Например, мы могли бы поискать все модели `person`, которые имеют имя `Peter`:

```js
// GET to /persons?filter[name]=Peter
this.store.query('person', { filter: { name: 'Peter' } }).then(function(peters) {
  // Do something with `peters`
});
```

### Запросы для одной записи

Если вы знаете, что ваш запрос вернет только один результат, на такой случай Ember Data предоставляет удобный метод. Он вернет обещание, которое завершится одной записью. Вызов `store.queryRecord()` выполнит запрос `GET` с переданным объектом, сериализованным как параметры запросов.

Например, если мы знаем, что человека можно определить исключительно по адресу электронной почты, мы могли бы искать модель `person`, которая имеет адрес `tomster@example.com`:

```js
// GET to /persons?filter[email]=tomster@example.com
this.store.queryRecord('person', { filter: { email: 'tomster@example.com' } }).then(function(tomster) {
  // do something with `tomster`
});
```