В приложение Super Rentals нам нужно добавить карту, чтобы показывать расположение каждой недвижимости. Для реализации этой функции мы используем несколько особенностей Ember:

1. Компонент, чтобы показывать карту для каждой недвижимости.
2. Службу, чтобы кэшировать показанные карты и использовать их в разных местах приложения.
3. Функцию утилиты, чтобы создать карту с помощью API Google Maps.

Мы начнем с отображения карты и проработаем наш способ использования API Google Maps.

## Компонент для отображения карт

Сначала мы добавим компонент, который показывает на карте город, где находится недвижимость.

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
  {{location-map location=rental.city}}
</article>
```

Затем генерируем компонент карты с помощью Ember CLI.
```
ember g component location-map
```

Запустив эту команду, мы получаем три файла: файл JavaScript для компонента, шаблон и тестовый файл.
Чтобы продумать реализацию компонента, сначала мы создадим тест.

В данном случае мы планируем, чтобы наша служба Google Maps обрабатывала отображение карты. Задача компонента — получить результаты от службы Maps (то есть элемент карты) и добавить их к элементу в шаблоне компонента.

Чтобы проверять в тесте только это поведение, мы используем API registration (регистрации), чтобы получить заглушку службы Maps. Заглушка стоит на месте реального объекта в приложении и имитирует его поведение. В заглушке службы определите метод `getMapElement`, который будет запрашивать карту на основе местоположения объекта.

`tests/integration/components/location-map-test.js`
```js
import { moduleForComponent, test } from 'ember-qunit';
import hbs from 'htmlbars-inline-precompile';
import Ember from 'ember';

let StubMapsService = Ember.Service.extend({
  getMapElement(location) {
    this.set('calledWithLocation', location);
    // We create a div here to simulate our maps service,
    // which will create and then cache the map element
    return document.createElement('div');
  }
});

moduleForComponent('location-map', 'Integration | Component | location map', {
  integration: true,
  beforeEach() {
    this.register('service:maps', StubMapsService);
    this.inject.service('maps', { as: 'mapsService' });
  }
});

test('should append map element to container element', function(assert) {
  this.set('myLocation', 'New York');
  this.render(hbs`{{location-map location=myLocation}}`);
  assert.equal(this.$('.map-container').children().length, 1, 'the map element should be put onscreen');
  assert.equal(this.get('mapsService.calledWithLocation'), 'New York', 'a map of New York should be requested');
});
```

В функции `beforeEach`, которая запускается перед каждым тестом, мы используем неявную функцию `this.register`, чтобы зарегистрировать заглушку службы на месте службы Maps. Регистрация позволяет сделать объект доступным для приложения Ember, и в нашем случае оно сможет загружать компоненты из шаблонов и внедрять службы.

При вызове функции `this.inject.service` только что зарегистрированная нами служба вставляется в контекст тестов. Поэтому каждый тест может получить доступ к ней через `this.get('mapsService')`.
В этом примере мы утверждаем, что `calledWithLocation` в заглушке назначается местоположению, которое мы передали компоненту.

Чтобы тест был пройден, добавьте элемент-контейнер в шаблон компонента.

`app/templates/components/location-map.hbs`
```hbs
<div class="map-container"></div>
```

Затем обновите компонент, чтобы добавить выходные данные карты в его внутренний элемент-контейнер. Мы внедрим службу Maps и вызовем функцию `getMapElement` с предоставленным местоположением.

Затем мы добавляем элемент карты, который вернули от службы, реализуя `didInsertElement`. Это hook [жизненного цикла компонента](https://guides.emberjs.com/v2.8.0/components/the-component-lifecycle/#toc_integrating-with-third-party-libraries-with-code-didinsertelement-code). Эта функция выполняется во время отображения, после того как разметка компонента вставляется в DOM.

`app/components/location-map.js`
```js
import Ember from 'ember';

export default Ember.Component.extend({
  maps: Ember.inject.service(),

  didInsertElement() {
    this._super(...arguments);
    let location = this.get('location');
    let mapElement = this.get('maps').getMapElement(location);
    this.$('.map-container').append(mapElement);
  }
});
```

## Получение карт с помощью службы

Сейчас мы бы могли пройти интеграционный тест компонента, но приемочный тест провалится, так как не сможет найти службу Maps. Помимо провального приемочного теста мы не увидим ни одной карты на веб-странице. Чтобы генерировать карты, мы реализуем службу Maps.

Обращение к API maps через [службу](http://emjs.ru/v2/applications/services/) даст нам несколько преимуществ:

* Она внедряется с помощью метода [«локатор служб»](https://en.wikipedia.org/wiki/Service_locator_pattern). Это значит, что она получит API maps из кода, который его использует, что упрощает реорганизацию и сопровождение кода.
* Это служба с отложенной загрузкой, то есть она не будет инициализирована, пока ее не вызовут один раз.
* В некоторых случаях это может снизить нагрузку приложения на процессор и потребление памяти.
* Это синглтон, который позволит нам кэшировать данные карты.
* У нее есть жизненный цикл. Это значит, что у нас есть hook`и, которые выполняют код очистки, когда служба прекращает работу. Это предотвращает утечки данных и ненужную обработку.

Начнем с создания службы путем генерации через Ember CLI. Так мы создадим файл службы, а также модульный тест для нее.

```
ember g service maps
```

Служба будет хранить кэш элементов карты на основе местоположения. Если элемент карты есть в кэше, служба вернет его, или же создаст новый и добавит его в кэш.

Чтобы протестировать службу, нам нужно удостовериться, что ранее загруженные местоположения извлекаются из кэша, а новые создаются с помощью утилиты.

`tests/unit/services/maps-test.js`
```js
import { moduleFor, test } from 'ember-qunit';
import Ember from 'ember';

const DUMMY_ELEMENT = {};

let MapUtilStub = Ember.Object.extend({
  createMap(element, location) {
    this.assert.ok(element, 'createMap called with element');
    this.assert.ok(location, 'createMap called with location');
    return DUMMY_ELEMENT;
  }
});

moduleFor('service:maps', 'Unit | Service | maps', {
  needs: ['util:google-maps']
});

test('should create a new map if one isnt cached for location', function (assert) {
  assert.expect(4);
  let stubMapUtil = MapUtilStub.create({ assert });
  let mapService = this.subject({ mapUtil: stubMapUtil });
  let element = mapService.getMapElement('San Francisco');
  assert.ok(element, 'element exists');
  assert.equal(element.className, 'map', 'element has class name of map');
});

test('should use existing map if one is cached for location', function (assert) {
  assert.expect(1);
  let stubCachedMaps = Ember.Object.create({
    sanFrancisco: DUMMY_ELEMENT
  });
  let mapService = this.subject({ cachedMaps: stubCachedMaps });
  let element = mapService.getMapElement('San Francisco');
  assert.equal(element, DUMMY_ELEMENT, 'element fetched from cache');
});
```

Учтите, что тест использует объект-пустышку в качестве возвращаемого элемента карты.
Это может быть любой объект, потому что он используется только для подтверждения доступа к кэшу.
Также учтите, что местоположение в объекте кэша `camelized` (имеет регистр Camel), поэтому может быть использовано как ключ.

Теперь реализуем службу. Обратите внимание, что мы проверяем, существует ли для данного местоположения карта, и используем ее, или же вызываем утилиту Google Maps, чтобы создать новую.
Мы взаимодействуем с API maps в обход утилиты Ember, поэтому можем протестировать службу без сетевых запросов к Google.

`app/services/maps.js`
```js
import Ember from 'ember';
import MapUtil from '../utils/google-maps';

export default Ember.Service.extend({

  init() {
    if (!this.get('cachedMaps')) {
      this.set('cachedMaps', Ember.Object.create());
    }
    if (!this.get('mapUtil')) {
      this.set('mapUtil', MapUtil.create());
    }
  },

  getMapElement(location) {
    let camelizedLocation = location.camelize();
    let element = this.get(`cachedMaps.${camelizedLocation}`);
    if (!element) {
      element = this.createMapElement();
      this.get('mapUtil').createMap(element, location);
      this.set(`cachedMaps.${camelizedLocation}`, element);
    }
    return element;
  },

  createMapElement() {
    let element = document.createElement('div');
    element.className = 'map';
    return element;
  }

});
```

## Доступ к Google Maps

До реализации утилиты карты нам нужно открыть доступ приложению Ember к стороннему API map. 
Есть несколько способов включить в Ember сторонние библиотеки. Для начала посмотрите раздел руководства об [управлении зависимостями](https://guides.emberjs.com/v2.8.0/addons-and-dependencies/managing-dependencies/).

Так как Google предоставляет свой API map как удаленный скрипт, мы используем curl, чтобы скачать его в директорию vendor нашего проекта.

Из корневой директории проекта запустите команду ниже, чтобы добавить скрипт Google Maps в папку vendor проекта как `gmaps.js`.

`Curl` — команда UNIX. Поэтому, если вы работаете на Windows, вам нужно изучить информацию о [поддержке bash на Windows](https://msdn.microsoft.com/en-us/commandline/wsl/about) или использовать альтернативный метод, чтобы скачать скрипт в директорию vendor.

```
curl -o vendor/gmaps.js https://maps.googleapis.com/maps/api/js?v=3.22
```

Когда скрипт находится в директории vendor, его можно встроить в приложение.
Нам нужно только сообщить Ember CLI, чтобы он импортировал его с помощью файла сборки:

`ember-cli-build.js`
```js
/*jshint node:true*/
/* global require, module */
var EmberApp = require('ember-cli/lib/broccoli/ember-app');

module.exports = function(defaults) {
  var app = new EmberApp(defaults, {
    // Add options here
  });

  // Use `app.import` to add additional libraries to the generated
  // output files.
  //
  // If you need to use different assets in different
  // environments, specify an object as the first parameter. That
  // object's keys should be the environment name and the values
  // should be the asset to use in that environment.
  //
  // If the library that you are including contains AMD or ES6
  // modules that you would like to import into your application
  // please specify an object with the list of modules as keys
  // along with the exports of each module as its value.
  app.import('vendor/gmaps.js');

  return app.toTree();
};
```

## Получение доступа к API Google Maps

Теперь API maps доступен для приложения, и мы можем создать утилиту карты. Файлы утилиты можно сгенерировать через Ember CLI.

```
ember g util google-maps
```

С помощью команды CLI `generate util` мы создадим файл утилиты и модульный тест. Мы удалим модульный тест, так как нам не нужно проверять код Google. Нашему приложению нужна одна функция, `createMapElement`. Она использует `google.maps.Map`, чтобы создать элемент карты, `google.maps.Geocoder`, чтобы найти координаты местоположения, и `google.maps.Marker`, чтобы поставить на карте метку на основе приведенного местоположения.

`app/utils/google-maps.js`
```js
import Ember from 'ember';

const google = window.google;

export default Ember.Object.extend({

  init() {
    this.set('geocoder', new google.maps.Geocoder());
  },

  createMap(element, location) {
    let map = new google.maps.Map(element, { scrollwheel: false, zoom: 10 });
    this.pinLocation(location, map);
    return map;
  },

  pinLocation(location, map) {
    this.get('geocoder').geocode({address: location}, (result, status) => {
      if (status === google.maps.GeocoderStatus.OK) {
        let geometry = result[0].geometry.location;
        let position = { lat: geometry.lat(), lng: geometry.lng() };
        map.setCenter(position);
        new google.maps.Marker({ position, map, title: location });
      }
    });
  }

});
```

После перезапуска сервера на главной странице мы должны увидеть карты у каждой позиции с недвижимостью!

![style super rentals maps](https://guides.emberjs.com/v2.7.0/images/service/style-super-rentals-maps.png)

## Заглушки служб в приемочных тестах

Наконец, нам нужно обновить приемочные тесты, чтобы учесть новую службу.
Хотя было бы здорово подтвердить, что карта отображается, не хотелось бы затрагивать API Google Maps при каждом запуске приемочного теста. В этом руководстве мы будем полагаться на интеграционные тесты компонента и проверять, чтобы карта из DOM была привязана к нашему экрану. Чтобы не достичь лимита запросов к Maps, мы сделаем заглушку для службы Maps в приемочных тестах.

Очень часто службы связываются со сторонними API, которые не стоит включать в автоматизированные тесты. Чтобы сделать заглушки для этих служб, мы просто должны зарегистрировать для службы заглушку. Она реализует тот же API, но не имеет зависимостей, которые могут стать проблемой для тестов.

Добавьте следующий код после imports (импортов) в приемочный тест:

`/tests/acceptance/list-rentals-test.js`
```js
import Ember from 'ember';

let StubMapsService = Ember.Service.extend({
  getMapElement() {
    return document.createElement('div');
  }
});

moduleForAcceptance('Acceptance | list rentals', {
  beforeEach() {
    this.application.register('service:stubMaps', StubMapsService);
    this.application.inject('component:location-map', 'maps', 'service:stubMaps');
  }
});
```

Здесь мы добавляем собственную заглушку службы Maps, которая просто создает пустой div. Затем мы добавляем ее в [реестр](https://guides.emberjs.com/v2.8.0/applications/dependency-injection/#toc_factory-registrations) Ember и внедряем в компонент `location-map`, который ее использует.
Таким образом, каждый раз при создании этого компонента наша заглушка будет использоваться вместо службы Google maps. Теперь, когда мы запустим приемочные тесты, вы заметите, что карты не отображаются.

![acceptance without maps](https://guides.emberjs.com/v2.8.0/images/service/acceptance-without-maps.png)