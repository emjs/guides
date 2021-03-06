Чтобы сделать обновление приложения наиболее плавным, Inspector собирает все устаревшие элементы, группирует их и отображает в том виде, который помогает эти элементы корректировать.

Чтобы просмотреть список устаревших элементов в приложении, щелкните по `Deprecations`.

<img src="/static/images/guides/ember-inspector/deprecations-screenshot.png" width="680"/>

Вы можете увидеть общее число устаревших элементов рядом с надписью в меню. Также можно посмотреть частоту использования каждого элемента.

### EMBER CLI и источники устаревших элементов

Если вы используете Ember CLI, и у вас есть карты кода, то вы можете просмотреть список источников для каждого устаревшего элемента. Если вы используете Chrome или Firefox, то щелкнув по source, вы откроете панель с источниками и увидите код, который послужил причиной сообщения об устаревшем элементе.

<img src="/static/images/guides/ember-inspector/deprecations-source.png" />

<img src="/static/images/guides/ember-inspector/deprecations-sources-panel.png" width="550"/>

Вы можете отправить отслеживаемый стек сообщения об устаревшем элементе на консоль, если щелкните на `Trace in the console`.

### Способы перехода

Щелкните по ссылке «Transition Plan», чтобы получить информацию о том, как убрать предупреждение об устаревшем элементе. Вы попадете в полезное руководство по устаревшим элементам на веб-сайте Ember.

<img src="/static/images/guides/ember-inspector/deprecations-transition-plan.png" width="680" />

### Фильтр и чистка

Вы можете отфильтровать устаревшие элементы, введя запрос в окне поиска. Также вы можете убрать текущие устаревшие элементы, щелкнув вверху по иконке очистки.    

<img src="/static/images/guides/ember-inspector/deprecations-toolbar.png" width="300"/>