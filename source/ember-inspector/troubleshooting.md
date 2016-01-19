Ниже приведены распространенные проблемы, с которыми вы можете столкнуться при использовании Inspector, а также необходимые действия для их исправления. Если ваша проблема здесь не указана, то отправьте ее на рассмотрение в архив [GitHub](https://github.com/emberjs/ember-inspector).

### Приложение Ember не обнаружено

Если Inspector не может обнаружить приложение Ember, то вы увидите следующее сообщение:

<img src="/static/images/guides/ember-inspector/troubleshooting-application-not-detected.png" width="350">

Причинами могут быть:

* Это не приложение Ember.
* Вы используете старую версию Ember (< 1.0).
* Вы используете протокол отличный от http или https. Для протокола file:// следуйте [этим рекомендациям](http://emjs.ru/v2/ember-inspector/installation/#toc_file-protocol).
* Приложение Ember находится внутри встроенного фрейма, помещенного в «песочницу», без адреса url (если вы используете JS Bin, следуйте [этим рекомендациям](http://emjs.ru/v2/ember-inspector/troubleshooting/#toc_using-the-inspector-with-js-bin)).

### Использование INSPECTOR с JS BIN

Учитывая, как JS Bin использует встроенные фреймы, Inspector не работает с режимом редактирования. Чтобы использовать Inspector с JS Bin, переключитесь на режим «предварительного просмотра в реальном времени», щелкнув по стрелке в кружке.

<img src="/static/images/guides/ember-inspector/troubleshooting-jsbin.png" width="350">

### Приложение не обнаруживается без перезагрузки

Если вы всегда должны перезагружать приложение после открытия Inspector, это может значить, что загрузочное состояние приложение некорректно. Такое происходит, если вы вызываете `advanceReadiness` или `deferReadiness`, когда приложение уже загружено.

### Адаптер данных не обнаружен

Это когда вы щелкаете по вкладке Data и видите такое сообщение:

<img src="/static/images/guides/ember-inspector/troubleshooting-data-adapter.png" width="350">

Это значит, что библиотека для обеспечения сохранности данных, которую вы используете, не поддерживается Inspector. Если вы автор библиотеки, то посмотрите в [этом разделе](http://emjs.ru/v2/ember-inspector/data/#toc_building-a-data-custom-adapter), как добавить в Inspector поддержку вашей библиотеки.

### Обещания не обнаружены

Вы щелкаете по вкладке Promises и видите такое сообщение:

<img src="/static/images/guides/ember-inspector/troubleshooting-promises-not-detected.png" width="350">

Это происходит, если вы используете версию Ember < 1.3.

#### Отсутствие обещаний

Если вкладка Promises работает, но есть обещания, которые вы не можете найти, это значит, что пропавшие обещания были созданы до того, как Inspector был активирован. Чтобы обнаружить обещания в момент загрузки приложения, щелкните по кнопке `Reload` ниже:

<img src="/static/images/guides/ember-inspector/troubleshooting-promises-toolbar.png" width="350">

#### Устаревшая версия INSPECTOR в FIREFOX

Дополнения Firefox должны проходить процесс рассмотрения перед каждым обновлением, поэтому Inspector обычно на одну версию позади.

К сожалению, мы не контролируем процесс рассмотрения в Firefox, и если вам нужна последняя версия Inspector, то загрузите и установите ее вручную с [GitHub](https://github.com/emberjs/ember-inspector).