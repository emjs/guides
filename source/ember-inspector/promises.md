Inspector предоставляет способ просматривать все обещания, созданные в приложении. Щелкните по строке `Promises` в меню, чтобы начать просмотр.

<img src="/static/images/guides/ember-inspector/promises-screenshot.png" width="680" />

Вы увидите иерархический список обещаний с отметками, которые описывают каждое из них, их состояние, закрепленное значение и время, необходимое для завершения.

### Состояния обещаний и фильтрация

Состояния обещаний отмечаются разными цветами.

<img src="/static/images/guides/ember-inspector/promises-fulfilled.png" width="300"/>

<img src="/static/images/guides/ember-inspector/promises-pending.png" width="300"/>

<img src="/static/images/guides/ember-inspector/promises-rejected.png" width="300"/>

Вы можете отфильтровать их, щелкнув по следующим меткам: `Rejected`, `Pending`, `Fulfilled`.   

<img src="/static/images/guides/ember-inspector/promises-toolbar.png" width="600"/>

Вы также можете искать обещания, введя запрос в поле поиска.

Чтобы очистить текущие записи обещаний, щелкните по иконке очистки вверху слева на вкладке.

### Просмотр закрепленных значений

Если значение исполнения обещания — это объект или массив Ember, то вы можете щелкнуть по этому объекту, чтобы открыть его в Object Inspector. 

<img src="/static/images/guides/ember-inspector/promises-object-inspector.png" width="400"/>

Если значение отклонения — это объект `Error`, то вы можете отправить отслеживание стека на консоль.

<img src="/static/images/guides/ember-inspector/promises-error.png" width="400"/>

Еще вы можете щелкнуть по кнопке `$E`, чтобы послать значение на консоль.

### Отслеживание

Inspector предоставляет возможность просмотреть отслеживание стека обещания. Отслеживание обещаний по умолчанию отключено из соображений производительности. Чтобы активировать его, поставьте галочку в поле `Trace promise`. Вам стоит выполнить перезагрузку, чтобы отслеживать существующие обещания.

<img src="/static/images/guides/ember-inspector/promises-trace-checkbox.png" width="200"/>

Чтобы отслеживать обещание, щелкните по кнопке `Trace`, которая находится за отметкой. Так вы отправите отслеживание стека на консоль.

<img src="/static/images/guides/ember-inspector/promises-trace.png" width="300"/>

### Маркирование обещаний

Обещания, сгенерированные Ember, маркируются по умолчанию. Вы также можете пометить собственные обещания RSVP, чтобы найти их во вкладке обещаний в Inspector. Все методы RSVP могут принимать отметку в качестве конечного аргумента. 

```js
var label = 'Find Posts'

new RSVP.Promise(method, label);

RSVP.Promise.resolve(value, label);

RSVP.Promise.reject(reason, label);

RSVP.Promise.all(array, label);

RSVP.Promise.hash(hash, label);

promise.then(success, failure, label);

promise.catch(callback, label);

promise.finally(callback, label);

```