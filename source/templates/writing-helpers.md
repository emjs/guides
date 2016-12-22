Хелперы позволяют добавлять шаблонам функциональность, которой у них изначально не было.
Хелперы очень полезны для преобразования исходных значений моделей и компонентов в более подходящий для пользователей формат.

Например, представьте, что у вас есть модель `Invoice` (счет). Она содержит атрибут `totalDue`, который представляет конечную сумму этого счета. Так как мы не хотим, чтобы компания прекратила деятельность в связи со странными ошибками округления в JavaScript, [мы храним это значение в центах вместо использования доллара с плавающей запятой](https://stackoverflow.com/questions/3730019/why-not-use-double-or-float-to-represent-currency/3730040#3730040).

Но если мы будем показывать пользователям стоимость в долларах в виде «100¢» вместо «$1.00», они запутаются. Мы можем написать хелпер, чтобы преобразовать формат этих значений в подходящую удобочитаемую форму.

Создадим хелпер `format-currency`, который берет целое число центов и преобразует его в долларовый формат.

Чтобы использовать хелпер `format-currency`, его нужно вызывать с помощью фигурных скобок в шаблоне:

```hbs
Your total is {{format-currency model.totalDue}}.
```

Теперь реализуем этот хелпер. Хелперы — функции. Они берут одно или несколько входных значений и возвращают единственное выходное значение, которое следует вставить в HTML.

Чтобы добавить новый хелпер, создайте файл с подходящим для вас именем (например, `format-currency.js`) в директории приложения `helpers`. Вы также можете использовать командную строку, чтобы Ember сгенерировал для вас файл:

```
ember generate helper format-currency
```

Этот файл должен экспортировать функцию, которая заключена в [`Ember.Helper.helper()`](http://emberjs.com/api/classes/Ember.Helper.html):

`app/helpers/format-currency.js`
```js
import Ember from 'ember';

export function formatCurrency([value, ...rest]) {
  let dollars = Math.floor(value / 100);
  let cents = value % 100;
  let sign = '$';

  if (cents.toString().length === 1) { cents = '0' + cents; }
  return `${sign}${dollars}.${cents}`;
}

export default Ember.Helper.helper(formatCurrency);
```

В этом примере функция получает сумму в долларах в пересчете на центы как первый параметр (`value`). Затем мы используем стандартный JavaScript, чтобы преобразовать количество центов в отформатированную строку, например `«$5.00»`.

Всякий раз, когда вы используете хелпер в шаблоне, Ember будет вызывать функцию и вставлять в модель DOM то, что вы вернете из хелпера.

Поэтому, если мы хотим отобразить сумму покупки, можно передать в шаблон значение в центах:

```
Your total is {{format-currency 250}}.
```

И Ember использует функцию нового хелпера, чтобы заменить содержимое внутри `{{ }}` на отформатированную сумму:

```
Your total is $2.50.
```

Каждый раз, когда меняются аргументы, которые вы передаете хелперу, Ember автоматически будет повторно вызывать хелпер с новыми значениями и обновлять модель DOM. И неважно, поступают ли аргументы от модели или компонента.

## Имена хелпера

В отличие от имен [компонентов](https://guides.emberjs.com/components/defining-a-component/), где нужно ставить тире для соответствия спецификации Custom Element, имена хелперов могут состоять из одного или нескольких слов. Если имя хелпера состоит из нескольких слов, его следует писать с тире. На этой странице есть примеры.

## Аргументы хелпера

Вы можете передать один или несколько аргументов, которые будут использоваться внутри функции. В примере выше мы передали сумму в центах в качестве первого и единственного аргумента.

Чтобы передать хелперу несколько аргументов, добавьте их в виде разделенного пробелами списка после имени хелпера:

```hbs
{{my-helper "hello" "world"}}
```

Массив этих аргументов передается функции хелпера:

`app/helpers/my-helper.js`
```js
import Ember from 'ember';

export default Ember.Helper.helper(function(params) {
  let [arg1, arg2] = params;

  console.log(arg1); // => "hello"
  console.log(arg2); // => "world"
});
```

Вы можете использовать сокращение деструктурирующего присваивания JavaScript, чтобы упорядочить код.
Этот пример — эквивалент вышестоящего примера (обратите внимание на сигнатуру функции):

`app/helpers/my-helper.js`
```js
import Ember from 'ember';

export default Ember.Helper.helper(function([arg1, arg2]) {
  console.log(arg1); // => "hello"
  console.log(arg2); // => "world"
});
```

## Именованные аргументы

Стандартные аргументы полезны в передаче данных для преобразования в функциях хелпера. Но так как порядок, в котором вы передаете аргументы, имеет значение, будет лучше, чтобы хелперы не принимали больше одного или двух.

При этом иногда вам будет нужно оставить поведение хелперов настраиваемым для разработчиков, которые вызывают их из своих шаблонов. Например, уйдем в сторону от примеров с североамериканской валютой и обновим наш хелпер `format-currency`, чтобы добавить настройку, которая позволит отображать определенный валютный знак.

Хелперы позволяют передавать именованные аргументы в качестве объекта JavaScript, который содержит имя аргумента с соответствующим значением. Порядок, в котором передаются именованные аргументы, не влияет на функциональность.

В этом примере мы можем передать аргумент `sign` нашему хелперу `format-currency`:

```hbs
{{format-currency 350 sign="£"}}
```

Нам нужно, чтобы хелпер выводил фунты стерлингов вместо долларов:

```hbs
£3.50
```

Объект, который содержит именованные аргументы, передается функции хелпера как второй аргумент.
Здесь представлен наш пример выше, в который мы добавили поддержку опционального параметра `sign`:

`app/helpers/format-currency.js`
```js
import Ember from 'ember';

export default Ember.Helper.helper(function([value, ...rest], namedArgs) {
  let dollars = Math.floor(value / 100);
  let cents = value % 100;
  let sign = namedArgs.sign === undefined ? '$' : namedArgs.sign;

  if (cents.toString().length === 1) { cents = '0' + cents; }
  return `${sign}${dollars}.${cents}`;
});
```

Вы можете передавать столько именованных аргументов, сколько захотите. Они будут добавлены к аргументу `namedArgs`, который передается функции:

```hbs
{{my-helper option1="hello" option2="world" option3="goodbye cruel world"}}
```

`app/helpers/my-helper.js`
```js
import Ember from 'ember';

export default Ember.Helper.helper(function(params, namedArgs) {
  console.log(namedArgs.option1); // => "hello"
  console.log(namedArgs.option2); // => "world"
  console.log(namedArgs.option3); // => "goodbye cruel world"
});
```

В этом случае можно использовать сокращение деструктурирующего присваивания JavaScript, чтобы упорядочить код выше:

`app/helpers/my-helper.js`
```js
import Ember from 'ember';

export default Ember.Helper.helper(function(params, { option1, option2, option3 }) {
  console.log(option1); // => "hello"
  console.log(option2); // => "world"
  console.log(option3); // => "goodbye cruel world"
});
```

В итоге аргументы отлично подходят для передачи значений:

```hbs
{{format-date currentDate}}
```

Хеши полезны для настройки поведения хелпера:

```hbs
{{print-current-date format="YYYY MM DD"}}
```

Вы можете использовать и то и другое, сколько захотите, пока параметры идут первыми:

```hbs
{{format-date-and-time date time format="YYYY MM DD h:mm" locale="en"}}
```

Пример выше содержит два аргумента:

* `date`
* `time`

и два именованных аргумента:

* `format="YYY MM DD h:mm"`
* `locale="en"`

## Хелперы на основе классов

По умолчанию хелперы не сохраняют состояние. Им передаются входные данные (параметры и хеш), они выполняют операцию с этими данными и возвращают один результат. У них нет побочных эффектов, и они не сохраняют какую-либо информацию, которая используется при последующих запусках функции.

Но в некоторых случаях вам может потребоваться хелпер, который взаимодействует с остальной частью приложения. Вы можете создавать хелперы на основе классов, которые имеют доступ к службам приложения и могут опционально сохранять состояние. Хотя обычно это необязательно и ненадежно.

Чтобы создать такой хелпер, а не экспортировать простую функцию, вам следует экспортировать подкласс [`Ember.Helper`](http://emberjs.com/api/classes/Ember.Helper.html). Классы хелпера должны содержать метод [`compute`](http://emberjs.com/api/classes/Ember.Helper.html#method_compute). Он ведет себя таким же образом, как и функция, которая передается [`Ember.Helper.helper`](http://emberjs.com/api/classes/Ember.Helper.html#method_helper).
Чтобы получить доступ к службе, вы должны сначала добавить ее в хелпер на основе класса. После добавления вы сможете вызывать методы службы или получить доступ к ее свойствам из метода `compute()`.

Для примера вот наш хелпер `format-currency`, переделанный в хелпер на основе класса:

`app/helpers/format-currency.js`
```js
import Ember from 'ember';

export default Ember.Helper.extend({
  compute([value, ...rest], hash) {
    let dollars = Math.floor(value / 100);
    let cents = value % 100;
    let sign = hash.sign === undefined ? '$' : hash.sign;

    if (cents.toString().length === 1) { cents = '0' + cents; }
    return `${sign}${dollars}.${cents}`;
  }
});
```

Это эквивалент `format-currency` из примера выше. Эту версию функции можно рассматривать как сокращение для более длинной классовой формы, если не требуется внедрение зависимости.

В качестве еще одного примера создадим хелпер, использующий службу аутентификации, которая приветствует пользователей по их имени, когда они входят в систему:

`app/helpers/is-authenticated.js`
```js
import Ember from 'ember';

export default Ember.Helper.extend({
  authentication: Ember.inject.service(),
  compute() {
    let authentication = this.get('authentication');

    if (authentication.get('isAuthenticated')) {
      return 'Welcome back, ' + authentication.get('username');
    } else {
      return 'Not logged in';
    }
  }
});
```

## Изолирование содержимого HTML

Чтобы защитить приложение от межсайтового скриптинга (XSS), Ember автоматически изолирует любое значение, которое вы возвращаете из хелпера. Поэтому браузер не распознает его как HTML.

Например, вот хелпер `make-bold`, который возвращает строку, содержащую HTML:

`app/helpers/make-bold.js`
```js
import Ember from 'ember';

export default Ember.Helper.helper(function([param, ...rest]) {
  return `<b>${param}</b>`;
});
```

Вы можете вызвать его таким образом:

```hbs
{{make-bold "Hello world"}}
```

Ember будет изолировать теги HTML примерно так:

```hbs
&lt;b&gt;Hello world&lt;/b&gt;
```

Пользователь увидит текстовую строку `<b>Hello world</b>`, а не текст с жирным шрифтом, как вы хотели. Мы можем указать Ember, чтобы он не изолировал возвращаемое значение (то, что не несет угрозы) с помощью строкового параметра [`htmlSafe`](http://emberjs.com/api/classes/Ember.String.html#method_htmlSafe):

`app/helpers/make-bold.js`
```js
import Ember from 'ember';

export default Ember.Helper.helper(function([param, ...rest]) {
  return Ember.String.htmlSafe(`<b>${param}</b>`);
});
```

Если вы возвращаете `SafeString` (строка с [`htmlSafe`](http://emberjs.com/api/classes/Ember.String.html#method_htmlSafe)), Ember понимает, что строка не содержит вредоносного HTML, и вы подтверждаете это.

Но обратите внимание, что в вышеуказанном коде мы могли по неосторожности внести уязвимости для XSS в приложение! Если слепо помечать строку как безопасную, взломщик сможет внести собственный код HTML в приложение. Это позволит ему, например, получить доступ к конфиденциальным данным клиентов.

Представьте, что у нас есть приложение обмена сообщениями и мы используем хелпер `make-bold`, чтобы приветствовать новых пользователей на канале:

```hbs
Welcome back! {{make-bold model.firstName}} has joined the channel.
```

Теперь злоумышленнику нужно просто внести свое `firstName` в строку, которая содержит HTML (например, тег `<script>`, который отправляет личные данные клиентов на сервер), и каждый пользователь в этой комнате чата будет скомпрометирован.

В общем вам следует использовать компоненты, если вы заключаете содержимое в HTML. Но если вам нужно включить смесь кода HTML и значений моделей, которые вы возвращаете из хелпера, убедитесь, что вы изолировали любые данные, которые могут поступить от непроверенного пользователя, с помощью параметра `escapeExpression`:

`app/helpers/make-bold.js`
```js
import Ember from 'ember';

export default Ember.Helper.helper(function([param, ...rest]) {
  let value = Ember.Handlebars.Utils.escapeExpression(param);
  return Ember.String.htmlSafe(`<b>${value}</b>`);
});
```

Теперь значение, которое передается хелперу, имеет изолированный HTML, но проверенные теги `<b>`, где нам нужно заключить значение, не затрагиваются. Злоумышленник, который попробует установить `irstName` куда-либо, где есть HTML, увидит только это:

```hbs
Welcome back! <b>&lt;script
type="javascript"&gt;alert('pwned!');&lt;/script&gt;</b> has joined the channel.
```