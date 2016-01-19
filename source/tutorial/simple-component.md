Когда пользователь просматривает список недвижимости, ему могут понадобиться интерактивные опции, которые помогут сделать выбор. Давайте добавим возможность скрывать и показывать изображение недвижимости в каждом пункте списка. Чтобы сделать это, мы используем компонент.

Сначала сгенерируем компонент `rental-listing`, который будет управлять поведением каждого элемента списка. Чтобы избежать конфликта с элементами HTML, в имени каждого компонента нужно использовать тире. Поэтому `rental-listing` подойдет, а `rental` уже нет.

```shell
ember g component rental-listing
```

Ember CLI сгенерирует несколько файлов для компонента:

```shell
installing component
  create app/components/rental-listing.js
  create app/templates/components/rental-listing.hbs
installing component-test
  create tests/integration/components/rental-listing-test.js
```

Компонент состоит из двух частей: шаблона Handlebars, который определяет внешний вид (`app/templates/components/rental-listing.hbs`), и исходного файла JavaScript (`app/components/rental-listing.js`), который определяет поведение.

Наш новый компонент `rental-listing` будет отвечать за то, как пользователь видит и взаимодействует с rental. Чтобы начать, давайте переместим детали отображения недвижимости для одного элемента rental из шаблона `index.hbs` в `rental-listing.hbs`:

`app/templates/components/rental-listing.hbs`
```hbs
<h2>{{rental.title}}</h2>
<p>Owner: {{rental.owner}}</p>
<p>Type: {{rental.type}}</p>
<p>Location: {{rental.city}}</p>
<p>Number of bedrooms: {{rental.bedrooms}}</p>
```

В шаблоне `index.hbs` мы разместим старую разметку HTML в цикле `{{#each}}` с новым компонентом `rental-listing`:

`app/templates/index.hbs`
```hbs
…
{{#each model as |rentalUnit|}}
  {{rental-listing rental=rentalUnit}}
{{/each}}
…
```

Здесь мы вызываем компонент `rental-listing` по наименованию и назначаем каждый `rentalUnit` в качестве атрибута `rental` компонента.

## Скрыть и показать изображения

Теперь мы можем добавить функциональность, которая позволит по запросу пользователя открыть изображение.

Давайте используем помощник `{{#if}}`, чтобы показать текущее изображение недвижимости только в том случае, когда `isImageShowing` будет true. Или же давайте покажем кнопку, которая даст пользователю возможность переключаться:

`app/templates/components/rental-listing.hbs`
```hbs
<h2>{{rental.title}}</h2>
<p>Owner: {{rental.owner}}</p>
<p>Type: {{rental.type}}</p>
<p>Location: {{rental.city}}</p>
<p>Number of bedrooms: {{rental.bedrooms}}</p>
{{#if isImageShowing }}
  <p><img src={{rental.image}} alt={{rental.type}} width="500px"></p>
{{else}}
  <button>Show image</button>
{{/if}}
```

Значение `isImageShowing` приходит от компонента в файле JavaScript. В нашем случае это `rental-listing.js`. Так как нам нужно, чтобы изначально изображение было скрыто, мы установим свойство на `false`:

`app/components/rental-listing.js`
```js
import Ember from 'ember';

export default Ember.Component.extend({
  isImageShowing: false
});
```

Чтобы значение `isImageShowing` сменялось на `true`, когда пользователь щелкает по кнопке, нужно добавить действие. Давайте назовем его `imageShow`.

`app/templates/components/rental-listing.hbs`
```hbs
...
<button {{action "imageShow"}}>Show image</button>
...
```

Нажатие кнопки отправит действие компоненту. Ember рассмотрит hash `actions` и вызовет функцию `imageShow`. Мы создадим функцию `imageShow` и установим `isImageShowing` в компоненте на `true`:

`pp/components/rental-listing.js`
```js
export default Ember.Component.extend({
  isImageShowing: false,
  actions: {
    imageShow() {
      this.set('isImageShowing', true);
    }
  }
});
```

Теперь при нажатии на кнопку в браузере, мы можем видеть изображение.

Нам также следует дать пользователям возможность скрывать картинку. В шаблоне мы добавим кнопку с действием `imageHide`:

`app/templates/components/rental-listing.hbs`
```hbs
<h2>{{rental.title}}</h2>
<p>Owner: {{rental.owner}}</p>
<p>Type: {{rental.type}}</p>
<p>Location: {{rental.city}}</p>
<p>Number of bedrooms: {{rental.bedrooms}}</p>
{{#if isImageShowing }}
  <p><img src={{rental.image}} alt={{rental.type}} width="500px"></p>
  <button {{action "imageHide"}}>Hide image</button>
{{else}}
  <button {{action "imageShow"}}>Show image</button>
{{/if}}
```

Затем мы определим в компоненте обработчик действия `imageHide`, чтобы установить `isImageShowing` на `false`:

`app/components/rental-listing.js`
```js
export default Ember.Component.extend({
  isImageShowing: false,
  actions: {
    imageShow() {
      this.set('isImageShowing', true);
    },
    imageHide() {
      this.set('isImageShowing', false)
    }
  }
});
```

Теперь наши пользователи могут скрыть и показать изображение недвижимости с помощью кнопок "Show image" и "Hide image".