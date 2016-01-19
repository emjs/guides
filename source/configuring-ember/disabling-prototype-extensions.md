По умолчанию, Ember.js расширяет прототипы исходных объектов JavaScript следующим образом:

* `Array` получает возможность реализовывать интерфейсы `Ember.Enumerable`, `Ember.MutableEnumerable`, `Ember.MutableArray` и `Ember.Array`. Это позволяет выполнять методы массива ECMAScript 5 в браузерах, которые их не поддерживают, добавлять удобные методы и свойства во встроенные массивы и отслеживать изменения массива.
* `String` получает возможность добавлять удобные методы вроде `camelize()` и `w()`. Вы можете найти список этих методов в [документации по Ember.String](http://emberjs.com/api/classes/Ember.String.html).
* `Function` расширяется методами для добавления аннотаций к функциям как вычислительным свойствам через метод `property()`, и как наблюдателям через методы `observes()` или `observesBefore()`. В настоящее время поддерживается использование этих методов, но они не охватывались в предыдущих версиях руководства.

Так Ember.js улучшает исходные прототипы. Мы хорошо взвесили все побочные эффекты, которые возникают при изменении этих прототипов и рекомендуем большинству разработчиков Ember использовать их. Эти расширения значительно сокращают количество стереотипного кода, который необходимо набирать.

Но мы понимаем, что бывают случаи, когда приложение Ember должно быть встроено в среду вне вашего контроля. Самые распространенные сценарии: авторская разработка стороннего JavaScript, который встроен непосредственно в другие страницы, или перенос приложения по частям в более современную архитектуру Ember.js.

В таких случаях, когда вы не можете или не хотите изменять исходные прототипы, Ember.js позволяет полностью отключить расширения, описанные выше.

Чтобы сделать это, просто установите флажок `EmberENV.EXTEND_PROTOTYPES` на `false`:

`config/environment.js`
```js
ENV = {
  EmberENV: {
    EXTEND_PROTOTYPES: false
  }
}
```

Вы можете настроить запрет расширений определенных классов. Например, так:

`config/environment.js`
```js
ENV = {
  EmberENV: {
    EXTEND_PROTOTYPES: {
      String: false,
      Array: true
    }
  }
}
```

## Жизнь без расширения прототипа

Чтобы ваше приложение правильно работало, вам нужно будет вручную делать то, что прежде выполнялось исходными объектами.

### Массивы

Исходные массивы больше не будут реализовывать функциональность, необходимую для их отслеживания. Если вы отключите расширение прототипа и попытаетесь использовать исходные массивы с элементами вроде помощника шаблона `{{#each}}`, Ember.js не сможет определить модификации в массиве, и шаблон не обновится при изменении лежащего в основе массива.

Кроме того, если вы попробуете установить модель `Ember.ArrayController` на простой исходный массив, это вызовет исключение, так как он больше не реализует интерфейс `Ember.Array`.

Вы можете вручную преобразовать исходный массив в массив, который реализует требуемые интерфейсы с помощью удобного метода `Ember.A`:

```js
var islands = ['Oahu', 'Kauai'];
islands.contains('Oahu');
//=> TypeError: Object Oahu,Kauai has no method 'contains'

// Convert `islands` to an array that implements the
// Ember enumerable and array interfaces
Ember.A(islands);

islands.contains('Oahu');
//=> true
```

### Строки

Строки больше не будут иметь удобных методов, описанных в справке [Ember.String API](http://emberjs.com/api/classes/Ember.String.html). Вместо этого вы можете использовать одноименные методы объекта `Ember.String` и передавать строку, чтобы использовать в качестве первого параметра:

```js
"my_cool_class".camelize();
//=> TypeError: Object my_cool_class has no method 'camelize'

Ember.String.camelize("my_cool_class");
//=> "myCoolClass"
```

### Функции

Раздел [Объект модели](http://emjs.ru/v2/object-model/) в руководстве рассказывает, как писать вычислительные свойства, наблюдателей и привязки без расширений прототипа. Ниже вы можете изучить, как преобразовать код в формат, который поддерживается в настоящее время.

Чтобы добавить аннотацию к вычислительным свойствам, используйте метод `Ember.computed()` и охватите функцию:

```js
// This won't work:
fullName: function() {
  return `${this.get('firstName')} ${this.get('lastName')}`;
}.property('firstName', 'lastName')


// Instead, do this:
fullName: Ember.computed('firstName', 'lastName', function() {
  return `${this.get('firstName')} ${this.get('lastName')}`;
})
```

Наблюдателям добавляются аннотации с помощью `Ember.observer()`:

```js
// This won't work:
fullNameDidChange: function() {
  console.log('Full name changed');
}.observes('fullName')


// Instead, do this:
fullNameDidChange: Ember.observer('fullName', function() {
  console.log('Full name changed');
})
```

Событийным функциям добавляются аннотации при помощи `Ember.on()`:

```js
// This won't work:
doStuffWhenInserted: function() {
  /* awesome sauce */
}.on('didInsertElement');

// Instead, do this:
doStuffWhenInserted: Ember.on('didInsertElement', function() {
  /* awesome sauce */
});
```