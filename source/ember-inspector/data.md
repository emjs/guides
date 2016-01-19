Вы можете просматривать модели, если щелкните по меню `Data`. Посмотрите секцию [Создание индивидуального адаптера данных](http://emjs.com/v2/ember-inspector/data/#toc_building-a-data-custom-adapter), если вы поддерживаете собственную библиотеку для обеспечения сохранности данных.

Когда вы откроете вкладку Data, вы увидите список типов моделей, определенных в приложении, вместе с числом загруженных записей. Inspector показывает загруженные записи, когда вы щелкаете по типу модели.

<img src="/static/images/guides/ember-inspector/data-screenshot.png" width="680"/>

### Просмотр записей

Каждый ряд в списке соответствует одной записи. Первые четыре атрибута модели показаны в виде списка. Нажатие на запись откроет Object Inspector для этой записи и отобразит ее атрибуты.

<img src="/static/images/guides/ember-inspector/data-object-inspector.png" width="680"/>

### Состояния записи и фильтр

Вкладка Data поддерживает синхронизацию с данными, загруженными в приложении. Любые добавления, удаления или изменения записи сразу отражаются. Если у вас есть несохраненные записи, они будут показаны зеленым цветом по щелчку на вкладку New.

<img src="/static/images/guides/ember-inspector/data-new-records.png" width="680"/>

Щелкните по вкладке Modified, чтобы отобразить изменения несохраненных записей.

<img src="/static/images/guides/ember-inspector/data-modified-records.png" width="680"/>

### Создание индивидуального адаптера данных

Вы можете использовать с Inspector собственную библиотеку для обеспечения сохранности данных. Создайте [адаптер данных](https://github.com/emberjs/ember.js/blob/3ac2fdb0b7373cbe9f3100bdb9035dd87a849f64/packages/ember-extension-support/lib/data_adapter.js), и вы сможете просматривать модели с помощью вкладки Data. Используйте [адаптер данных Ember Data](https://github.com/emberjs/data/blob/d7988679590bff63f4d92c4b5ecab173bd624ebb/packages/ember-data/lib/system/debug/debug_adapter.js) как пример для создания собственного.