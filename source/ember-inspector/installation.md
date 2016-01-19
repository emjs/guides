Ember Inspector — браузерное дополнение. Его задача: помочь вам понять и отладить приложения Ember.

Вы можете установить Inspector в Google Chrome, Firefox или другие браузеры (через букмарклет), а также на мобильные устройства, руководствуясь следующими шагами.

### GOOGLE CHROME

Вы можете установить Inspector в Google Chrome в качестве нового средства разработчика. Чтобы начать, посетите страницу с расширениями в интернет-магазине [Chrome Web Store](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi).

Нажмите «Установить».

<img src="/static/images/guides/ember-inspector/installation-chrome-store.png" width="680" />

После установки перейдите в приложение Ember, откройте «Инструменты разработки» и нажмите на вкладку `Ember` в дальнем правом углу.

<img src="/static/images/guides/ember-inspector/installation-chrome-panel.png" width="680">

#### Протокол FILE://

Чтобы использовать Inspector с протоколом `file://`, посетите `chrome://extensions` в Chrome и проверьте галочку в поле «Разрешить открывать файлы по ссылкам»:

<img src="/static/images/guides/ember-inspector/installation-chrome-file-urls.png" width="400">

#### Разрешить TOMSTER

Вы можете настроить иконку Tomster. Она будет отображаться в адресной строке браузера при посещении сайта, который использует Ember.

Перейдите в `chrome://extensions`, затем щелкните `Options`.

<img src="/static/images/guides/ember-inspector/installation-chrome-tomster.png" width="400">

Проверьте, что в поле «Display the Tomster» стоит галочка.

<img src="/static/images/guides/ember-inspector/installation-chrome-tomster-checkbox.png" width="400">

### FIREFOX

Посетите страницу с расширениями на сайте [Mozilla Add-ons](https://addons.mozilla.org/en-US/firefox/addon/ember-inspector/).

Щелкните «Установить».

<img src="/static/images/guides/ember-inspector/installation-firefox-store.png" width="680">

После установки перейдите в приложение Ember, откройте «Инструменты разработки» и щелкните по вкладке `Ember`.

<img src="/static/images/guides/ember-inspector/installation-firefox-panel.png" width="680">

#### Разрешить TOMSTER

Чтобы иконка Tomster отображалась в адресной строке браузера при посещении сайта, который использует Ember, перейдите в `about:addons`.

Щелкните `Расширения` -> `Настройки`.

<img src="/static/images/guides/ember-inspector/installation-firefox-preferences.png" width="600">

Затем убедитесь, что в поле «Display the Tomster icon when a site runs Ember.js» стоит галочка.

<img src="/static/images/guides/ember-inspector/installation-firefox-tomster-checkbox.png" width="400">

### Использование через букмарклет

Если вы не используете Chrome или Firefox, то можете применять Inspector через букмарклет.

Добавьте следующую закладку:

<a href="javascript: (function() { var s = document.createElement('script'); s.src = '//ember-extension.s3.amazonaws.com/dist_bookmarklet/load_inspector.js'; document.body.appendChild(s); }());">Bookmark Me</a>

Чтобы открыть Inspector, просто щелкните по новой закладке. Браузер Safari по умолчанию блокирует всплывающие окна, поэтому отключите эту функцию, прежде чем использовать букмарклет.

### Мобильные устройства

Если вы хотите запустить Inspector на мобильном устройстве, то можете использовать дополнение [Ember CLI Remote Inspector](https://github.com/joostdevries/ember-cli-remote-inspector).