Сейчас наше приложение напрямую показывает данные пользователя из моделей Ember Data. По мере расширения приложения, вам захочется воздействовать на данные до их отображения пользователям. И для этого Ember предоставляет хелперы шаблонов Handlebars. Они позволяют оформлять данные в шаблонах. Давайте используем хелпер handlebars, чтобы дать пользователю возможность бегло посмотреть, стоит ли свойство "Standalone" или "Community".

Для начала давайте сгенерируем хелпер для `rental-property-type`:

```bash
ember g helper rental-property-type
```

Так мы создадим хелпер и его тест:

```bash
installing helper
  create app/helpers/rental-property-type.js
installing helper-test
  create tests/unit/helpers/rental-property-type-test.js
```

Наш новый хелпер состоит из шаблонного кода, который выдал генератор:

`app/helpers/rental-property-type.js`
```js
import Ember from 'ember';

export function rentalPropertyType(params/*, hash*/) {
  return params;
}

export default Ember.Helper.helper(rentalPropertyType);
```

Давайте обновим шаблон компонента `rental-listing`, чтобы использовать новый хелпер и передавать `rental.type`:

`app/templates/components/rental-listing.hbs`
```hbs
<h2>{{rental.title}}</h2>
<p>Owner: {{rental.owner}}</p>
<p>Type: {{rental-property-type rental.type}} - {{rental.type}}</p>
<p>Location: {{rental.city}}</p>
<p>Number of bedrooms: {{rental.bedrooms}}</p>
{{#if isImageShowing }}
  <p><img src={{rental.image}} alt={{rental.type}} width="500px"></p>
  <button {{action "imageHide"}}>Hide image</button>
{{else}}
  <button {{action "imageShow"}}>Show image</button>
{{/if}}
```

В идеале мы увидим "Type: Standalone - Estate" для первой арендуемой недвижимости. Наш исходный хелпер шаблона возвращает значения `rental.type`. Давайте обновим хелпер так, чтобы проверять, есть ли свойство в массиве `communityPropertyTypes`, и если оно есть, то вернется `'Community'` или `'Standalone'`:

`app/helpers/rental-property-type.js`
```js
import Ember from 'ember';

const communityPropertyTypes = [
  'Condo',
  'Townhouse',
  'Apartment'
];

export function rentalPropertyType([type]/*, hash*/) {
  if (communityPropertyTypes.contains(type)) {
    return 'Community';
  }

  return 'Standalone';
}

export default Ember.Helper.helper(rentalPropertyType);
```

Handlebars передает хелперу массив аргументов из шаблона. Мы используем структурирование ES2015, чтобы получить первый элемент массива и назначить ему `type`. Затем мы можем проверить, существует ли `type` в массиве `communityPropertyTypes`.

Теперь в нашем браузере мы должны увидеть, что первая недвижимость значится как "Standalone", а другие две как "Community".