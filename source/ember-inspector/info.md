Чтобы просмотреть список используемых в приложении библиотек, щелкните по `Info`. Это представление отобразит библиотеки вместе с их версиями.

<img src="/static/images/guides/ember-inspector/info-screenshot.png" width="680"/>

### Регистрация библиотек

Если вы хотите добавить в список собственное приложение или библиотеку, то можете зарегистрировать их с помощью:

```js
Ember.libraries.register(libraryName, libraryVersion);
```

#### EMBER CLI

Если вы используете дополнение [ember-cli-app-version](https://github.com/embersherpa/ember-cli-app-version), имя и версия вашего приложения будут автоматически добавлены в список.