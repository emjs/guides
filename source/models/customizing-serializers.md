В Ember Data сериализаторы форматируют данные, которые переданы или получены из хранилища backend. По умолчанию, Ember Data сериализует данные по формату [JSON API](http://jsonapi.org/). Если ваш backend использует другой формат, то Ember Data позволяет настроить сериализатор или использовать другой.

Ember Data поставляется с 3 сериализаторами. `JSONAPISerializer` — исходный сериализатор, который работает с backends JSON API. `JSONSerializer` — простой сериализатор для работы с одним объектом json или массивом записей. `RESTSerializer` — более сложный сериализатор, который поддерживает загрузку данных между локальными устройствами. Он был исходным сериализатором до версии 2.0.

## Соглашения JSONAPISERIALIZER

При запросе записи `JSONAPISerializer` ожидает, пока сервер вернет JSON-представление записи, что соответствует следующим соглашениям.

### Документ JSON API

Сериализатор JSONAPI ожидает, пока backend вернет документ JSON API, который следует спецификации JSON API и соглашениям по примерам на сайте http://jsonapi.org/format/. Это значит, что все имена типа должны быть преобразованы во множественное число, а имена атрибутов и связей иметь тире. Например, если вы запрашиваете запись из `/people/123`, ответ будет выглядеть таким образом:

```js
{
  "data": {
    "type": "people",
    "id": "123",
    "attributes": {
      "first-name": "Jeff",
      "last-name": "Atwood"
    }
  }
}
```

Ответ, который включает несколько записей, может иметь массив в свойстве `data`.

```js
{
  "data": [{
    "type": "people",
    "id": "123",
    "attributes": {
      "first-name": "Jeff",
      "last-name": "Atwood"
    }
  }, {
    "type": "people",
    "id": "124",
    "attributes": {
      "first-name": "Yehuda",
      "last-name": "Katz"
    }
  }]
}
```

### Загруженные данные

Данные, которые не являются частью первичного запроса, но включают связанные отношения, должны быть размещены в массиве под ключом `included`. Например, если вы запросили запись из `/people/1`, а backend вернул в придачу какие-либо комментарии, имеющие отношение к этой связи, ответ должен выглядеть так:

```js
{
  "data": [{
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "JSON API paints my bikeshed!"
    },
    "links": {
      "self": "http://example.com/articles/1"
    },
    "relationships": {
      "comments": {
        "data": [
          { "type": "comments", "id": "5" },
          { "type": "comments", "id": "12" }
        ]
      }
    }
  }],
  "included": [{
    "type": "comments",
    "id": "5",
    "attributes": {
      "body": "First!"
    },
    "links": {
      "self": "http://example.com/comments/5"
    }
  }, {
    "type": "comments",
    "id": "12",
    "attributes": {
      "body": "I like XML better"
    },
    "links": {
      "self": "http://example.com/comments/12"
    }
  }]
}
```

## Настройка сериализаторов

Ember Data, по умолчанию, использует `JSONAPISerializer`, но вы можете переопределить исходные настройки, если зададите свой сериализатор. Есть два способа определить индивидуальный сериализатор. Вы можете задать его для всего приложения, если определите сериализатор «application».

`app/serializers/application.js`
```js
import DS from 'ember-data';

export default DS.JSONSerializer.extend({});
```

Еще вы можете определить сериализатора для специфической модели. Например, если у вас есть модель `post`, то вы также можете определить сериализатор `post`:

`app/serializers/post.js`
```js
import DS from 'ember-data';

export default DS.JSONSerializer.extend({});
```

Чтобы изменить формат данных, отправленных в хранилище backend, можно использовать hook `serialize`. Скажем, у нас есть такой ответ JSON API от Ember Data:

```js
{
  "data": {
    "attributes": {
      "id": "1",
      "name": "My Product",
      "amount": 100,
      "currency": "SEK"
    },
    "type": "product"
  }
}
```

Но наш сервер ожидает данные в таком формате:

```js
{
  "data": {
    "attributes": {
      "id": "1",
      "name": "My Product",
      "cost": {
        "amount": 100,
        "currency": "SEK"
      }
    },
    "type": "product"
  }
}
```

Вот как можно изменить данные:

`app/serializers/application.js`
```js
import DS from 'ember-data';

export default DS.JSONSerializer.extend({
  serialize(snapshot, options) {
    var json = this._super(...arguments);

    json.data.attributes.cost = {
      amount: json.data.attributes.amount,
      currency: json.data.attributes.currency
    };

    delete json.data.attributes.amount;
    delete json.data.attributes.currency;

    return json;
  },
});
```

Похожим образом, если хранилище backend предоставляет данные в отличном от JSON API формате, можно использовать hook `normalizeResponse`. Возьмем тот же пример выше. Если сервер предоставляет данные, которые выглядят так:

```js
{
  "data": {
    "attributes": {
      "id": "1",
      "name": "My Product",
      "cost": {
        "amount": 100,
        "currency": "SEK"
      }
    },
    "type": "product"
  }
}
```

И нам нужно изменить их таким образом:

```js
{
  "data": {
    "attributes": {
      "id": "1",
      "name": "My Product",
      "amount": 100,
      "currency": "SEK"
    },
    "type": "product"
  }
}
```

То вот как можно это сделать:

`app/serializers/application.js`
```js
import DS from 'ember-data';

export default DS.JSONSerializer.extend({
  normalizeResponse(store, primaryModelClass, payload, id, requestType) {
    payload.data.attributes.amount = payload.data.attributes.cost.amount;
    payload.data.attributes.amount = payload.data.attributes.cost.currency;

    delete payload.data.attributes.cost;

    return this._super(...arguments);
  },
});
```

Чтобы нормализовать только одну модель, можно точно также использовать hook `normalize`.

Больше информации о hooks для настройки сериализатора вы найдете в [документации API по сериализатору в Ember Data](http://emberjs.com/api/data/classes/DS.JSONAPISerializer.html#index).

### IDS

Чтобы отслеживать уникальные записи в хранилище, Ember Data ожидает, что каждая запись будет иметь свойство `id` в полезной нагрузке. Идентификаторы должны быть уникальными для каждой отдельной записи специфического типа. Если backend применил другой ключ, который не является `id`, то вы можете использовать свойство сериализатора `primaryKey`, чтобы корректно переводить свойство идентификатора в `id` при сериализации и десериализации данных.

`app/serializers/application.js`
```js
export default DS.JSONSerializer.extend({
  primaryKey: '_id'
});
```

### Имена атрибутов

В Ember Data принято соглашение использовать camelize с именами атрибутов в модели. Например:

`app/models/person.js`
```js
export default DS.Model.extend({
  firstName: DS.attr('string'),
  lastName:  DS.attr('string'),

  isPersonOfTheYear: DS.attr('boolean')
});
```

Но `JSONAPISerializer` ожидает, что атрибуты в документе полезной нагрузки, которую возвращает сервер, будут иметь тире:

```js
{
  "data": {
    "id": "44",
    "type": "people",
    "attributes": {
      "first-name": "Barack",
      "last-name": "Obama",
      "is-person-of-the-year": true
    }
  }
}
```

Если возвращенные сервером атрибуты используют другое соглашение, вы можете применить метод сериализатора `keyForAttribute`. Так вы преобразуете имя атрибута в модели в ключ в полезной нагрузке JSON. Например, если backend вернул атрибуты, которые `under_scored` вместо `dash-cased`, то вы могли бы переопределить метод `keyForAttribute` вот так:

`app/serializers/application.js`
```js
export default DS.JSONAPISerializer.extend({
  keyForAttribute: function(attr) {
    return Ember.String.underscore(attr);
  }
});
```

Нестандартные ключи может преобразовать индивидуальный сериализатор. Объект `attrs` можно использовать, чтобы объявить простое преобразование между именами свойства в записях DS.Model и ключами полезной нагрузки в сериализованном объекте JSON, который представляет запись. Объект с ключом свойства также может использоваться, чтобы объявлять ключ атрибута в полезной нагрузке ответа.

Если JSON для `person` имеет ключ `lastNameOfPerson`, а желательное имя атрибута — просто `lastName`, то создайте индивидуального сериализатора для модели и переопределите свойство `attrs`.

`app/models/person.js`
```js
export default DS.Model.extend({
  lastName: DS.attr('string')
});
```

`app/serializers/person.js`
```js
export default DS.JSONAPISerializer.extend({
  attrs: {
    lastName: 'lastNameOfPerson',
  }
});
```

### Связи

Обращение к другим записям должно производиться по ID. Например, если у вас есть модель со связью `hasMany`:

`app/models/post.js`
```js
export default DS.Model.extend({
  comments: DS.hasMany('comment', { async: true })
});
```

JSON должен кодировать связь как массив типов и идентификаторов:

```js
{
  "data": {
    "type": "posts",
    "id": "1",
    "relationships": {
      "comments": {
        "data": [
          { "type": "comments", "id": "5" },
          { "type": "comments", "id": "12" }
        ]
      }
    }
  }
}
```

`post.get('comments')` может загрузить `comments` для `post`. Адаптер JSON API пошлет 3 запроса `GET` в `/comments/1/`, `/comments/2/` и `/comments/3/`.

Любые связи `belongsTo` в представлении JSON должны быть версиями имени свойства с тире. Например, если у вас есть модель:

`app/models/comment.js`
```js
export default DS.Model.extend({
  originalPost: DS.belongsTo('post')
});
```

JSON должен кодировать связь как ID к еще одной записи:

```js
{
  "data": {
    "type": "comment",
    "id": "1",
    "relationships": {
      "original-post": {
        "data": { "type": "post", "id": "5" },
      }
    }
  }
}
```

Если необходимо, эти соглашения по наименованиям можно переписать с помощью метода `keyForRelationship`.

`app/serializers/application.js`
```js
export default DS.JSONAPISerializer.extend({
  keyForRelationship: function(key, relationship) {
    return key + 'Ids';
  }
});
```

## Создание индивидуальных преобразований

Иногда встроенные типы атрибутов (`string`, `number`, `boolean` и `date`) могут не соответствовать требованиям. Например, сервер может вернуть дату в нестандартном формате.

Ember Data может иметь новые преобразования JSON, отмеченные для использования в качестве атрибутов:

`app/transforms/coordinate-point.js `
```js
export default DS.Transform.extend({
  serialize: function(value) {
    return [value.get('x'), value.get('y')];
  },
  deserialize: function(value) {
    return Ember.create({ x: value[0], y: value[1] });
  }
});
```

`app/models/cursor.js`
```js
export default DS.Model.extend({
  position: DS.attr('coordinate-point')
});
```

Когда `coordinatePoint` получен из API, ожидается, что он будет массивом:

```js
{
  cursor: {
    position: [4,9]
  }
}
```

Но загруженный в экземпляре модели, он будет вести себя как объект:

```js
var cursor = store.findRecord('cursor', 1);
cursor.get('position.x'); //=> 4
cursor.get('position.y'); //=> 9
```

Если `position` меняется и сохраняется, он преобразуется через функцию `serialize` и снова представляется как массив в JSON.

## JSONSERIALIZER

Не все APIs следуют соглашениям, которые `JSONAPISerializer` использует с пространством имен данных и загруженными записями об отношениях. Некоторые существующие APIs могут возвращать простую полезную нагрузку JSON, которая является только запросом на выделение ресурса или массивом сериализованных записей. `JSONSerializer` — сериализатор, который поставляется с Ember Data. Его можно использовать вместе с `RESTAdapter`, чтобы сериализовать эти упрощенные APIs.

Чтобы использовать его в приложении, вам будет нужно определить `adapter:application`, который расширяет `JSONSerializer`.

`app/serializer/application.js`
```js
export default DS.JSONSerializer.extend({
  // ...
});
```

Для запросов, которые возвращают только одну запись (например, `store.findRecord('post', 1))`, `JSONSerializer` ожидает получить в ответ объект JSON, похожий на это:

```js
{
    "id": "1",
    "title": "Rails is omakase",
    "tag": "rails",
    "comments": ["1", "2"]
}
```

Для запросов, которые возвращают 0 или более записей (например, `store.findAll('post')`или `store.query('post', { filter: { status: 'draft' } }))`, `JSONSerializer` ожидает получить в ответ массив JSON. Он выглядит примерно так:

```js
[{
  "id": "1",
  "title": "Rails is omakase",
  "tag": "rails",
  "comments": ["1", "2"]
}, {
  "id": "2",
  "title": "I'm Running to Reform the W3C's Tag",
  "tag": "w3c",
  "comments": ["3"]
}]
```

JSONAPISerializer построен на JSONSerializer, поэтому они совместно используют многие hooks для настройки поведения процесса сериализации. Проверьте [документацию по API](http://emberjs.com/api/data/classes/DS.JSONSerializer.html), чтобы ознакомиться с полным списком методов и свойств.

## EMBEDDEDRECORDMIXIN

Хотя Ember Data поощряет загрузку связей, при работе с существующими APIs вы можете столкнуться с JSON, который содержит связи, вложенные внутри других записей. `EmbeddedRecordsMixin` помогает справиться с этой проблемой.

Чтобы установить вложенные записи, включите миксины во время расширения сериализатора, затем определите и сконфигурируйте вложенные связи.

Например, если бы модель `post` содержала вложенную запись `author`, вы определили связь так:

```js
{
    "id": "1",
    "title": "Rails is omakase",
    "tag": "rails",
    "authors": [
        {
            "id": "2",
            "name": "Steve"
        }
    ]
}
```

Вы могли бы определить связь так:

`app/serializers/post.js`
```js
export default DS.JSONSerializer.extend(DS.EmbeddedRecordsMixin, {
  attrs: {
    author: {
      serialize: 'records',
      deserialize: 'records'
    }
  }
});
```

Если вам понадобится сериализовать и десериализовать вложенную связь, вы можете использовать сокращенную форму — `{ embedded: 'always' }`. Следующий пример равноценен примеру выше:

`app/serializers/post.js`
```js
export default DS.JSONSerializer.extend(DS.EmbeddedRecordsMixin, {
  attrs: {
    author: { embedded: 'always' }
  }
});
```

Ключи `serialize` и `deserialize` поддерживают 3 выбора: `records` используется для подачи сигнала, что ожидается вся запись; `ids` используется для подачи сигнала, что ожидается только идентификатор записи; false используется для подачи сигнала, что запись не ожидается.

Например, вы захотели прочитать вложенную запись при извлечении полезной нагрузки JSON, но при сериализации записи включить только идентификатор связи. Это возможно с помощью выбора `serialize: 'ids'`. Вы можете также отказаться от сериализации связи, если установите `serialize: false`.

`app/serializers/post.js`
```js
export default DS.JSONSerializer.extend(DS.EmbeddedRecordsMixin, {
  attrs: {
    author: {
      serialize: false,
      deserialize: 'records'
    },
    comments: {
      deserialize: 'records',
      serialize: 'ids'
    }
  }
});
```

### Исходные установки EMBEDDEDRECORDSMIXIN

Если вы не переписываете `attrs` для специфической связи, `EmbeddedRecordsMixin` будет вести себя следующим образом:

BelongsTo: `{ serialize: 'id', deserialize: 'id' }`
HasMany: `{ serialize: false, deserialize: 'ids' }`

Есть вариант не вкладывать JSON в сериализованную полезную нагрузку с помощью `serialize: 'ids'`. Если вы совсем не хотите посылать связь, то можете использовать `serialize: false`.

## Авторские сериализаторы

Если вы хотите создать индивидуальний сериализатор, то вам рекомендуется начать с `JSONAPISerializer` или `JSONSerializer` и расширить один из них, чтобы он соответствовал вашим нуждам. Но если ваша полезная нагрузка сильно отличается от одного из этих сериализаторов, то вы можете создать собственный, если расширите базовый класс `DS.Serializer`.

* [normalizeResponse](http://emberjs.com/api/data/classes/DS.Serializer.html#method_normalizeResponse)
* [serialize](http://emberjs.com/api/data/classes/DS.Serializer.html#method_serialize)
* [normalize](http://emberjs.com/api/data/classes/DS.Serializer.html#method_normalize)

Также важно знать о форме JSON `normalized`, которую Ember Data ожидает увидеть в качестве аргумента к `store.push()`.

`store.push` принимает документ JSON API. Но, в отличие от store.push в JSONAPISerializer, он не совершает каких-либо преобразований с именем типа записи или атрибутами. Важно убедиться, что имя типа соответствует имени файла, где оно точно определено. Также имена атрибута и связи в документе JSON API должны соответствовать стилю имени атрибута и свойствам связи модели.

Например, учитывая модель `post`:

`app/models/post.js`
```js
export default DS.Model.extend({
  title: DS.attr('string'),
  tag: DS.attr('string'),
  comments: hasMany('comment', { async: false }),
  relatedPosts: hasMany('post')
});
```

`store.push` будет принимать объект, который выглядит примерно так:

```js
{
  data: {
    id: "1",
    type: 'post',
    attributes: {
      title: "Rails is omakase",
      tag: "rails",
    },
    relationships: {
      comments: {
        data: [{ id: "1", type: 'comment' },
               { id: "2", type: 'comment' }],
      },
      relatedPosts: {
        data: {
          related: "/api/v1/posts/1/related-posts/"
        }
      }
    }
}
```

Каждая сериализованная запись должна следовать этому формату, чтобы корректно конвертироваться в запись Ember Data.

Свойства, которые определены в модели, но не включены в объект документа нормализованного JSON API, не будут обновляться. Свойства, которые включены в объект документа нормализованного JSON API, но не определены в модели, будут игнорироваться.

## Сериализаторы сообщества

Если ни один из встроенных в Ember Data сериализаторов не работает с backend, проверьте сообщества, которые поддерживают адаптеры и сериализаторы Ember Data. Среди них можно отметить:

* [Ember Observer](http://emberobserver.com/categories/data)
* [GitHub](https://github.com/search?q=ember+data+serializers&ref=cmdform)
* [Bower](http://bower.io/search/?q=ember-data-)