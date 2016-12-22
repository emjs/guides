Сейчас наше приложение напрямую показывает данные пользователя из моделей Ember Data. По мере расширения приложения, вам захочется манипулировать данными до их отображения пользователям. Для этого Ember предоставляет хелперы шаблонов Handlebars. Они позволяют оформлять данные в шаблонах.
Используем хелпер handlebars, чтобы дать пользователю возможность быстро посмотреть, является ли недвижимость Standalone (обособленной) или Community (частью жилого района).

Для начала сгенерируем хелпер для `rental-property-type`:

```
ember g helper rental-property-type
```

Так мы создадим два файла, хелпер и его тест:

```
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

Обновим шаблон компонента `rental-listing`, чтобы использовать новый хелпер и передавать `rental.type`:

`app/templates/components/rental-listing.hbs`
```hbs
<article class="listing">
  <a {{action 'toggleImageSize'}} class="image {{if isWide "wide"}}">
    <img src="{{rental.image}}" alt="">
    <small>View Larger</small>
  </a>
  <h3>{{rental.title}}</h3>
  <div class="detail owner">
    <span>Owner:</span> {{rental.owner}}
  </div>
  <div class="detail type">
    <span>Type:</span> {{rental-property-type rental.type}} - {{rental.type}}
  </div>
  <div class="detail location">
    <span>Location:</span> {{rental.city}}
  </div>
  <div class="detail bedrooms">
    <span>Number of bedrooms:</span> {{rental.bedrooms}}
  </div>
</article>
```

В идеале мы увидим Type: Standalone - Estate (Тип: Обособленный — Участок) для первой арендуемой недвижимости. Вместо этого наш исходный хелпер шаблона возвращает значения `rental.type`. Обновим хелпер, чтобы посмотреть, есть ли свойство в массиве `communityPropertyTypes`, и если есть, мы вернем `Community` или `Standalone`:

`app/helpers/rental-property-type.js`
```js
import Ember from 'ember';

const communityPropertyTypes = [
  'Condo',
  'Townhouse',
  'Apartment'
];

export function rentalPropertyType([type]/*, hash*/) {
  if (communityPropertyTypes.includes(type)) {
    return 'Community';
  }

  return 'Standalone';
}

export default Ember.Helper.helper(rentalPropertyType);
```

Handlebars передает хелперу массив аргументов из шаблона. Мы используем деструктурирование ES2015, чтобы получить первый элемент массива и назначить ему `type`. Затем мы можем проверить, существует ли `type` в массиве `communityPropertyTypes`.

Теперь в нашем браузере мы должны увидеть, что первая недвижимость отображается как Standalone, а другие две — как Community.