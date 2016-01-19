Вы можете использовать View Tree (дерево видимых элементов), чтобы просматривать текущее состояние приложения. View Tree показывает вам текущие отображенные шаблоны, модели, контроллеры и компоненты в формате «дерева». Щелкните на View Tree в меню слева, чтобы увидеть это.

<img src="/static/images/guides/ember-inspector/view-tree-screenshot.png" width="680">

Используйте подсказки, описанные в [Object Inspector](http://emjs.ru/v2/ember-inspector/object-inspector), чтобы просматривать модели и контроллеры. Ниже смотрите шаблоны и компоненты.

### Просмотр шаблонов

Чтобы увидеть, как Ember отображает шаблон, щелкните по шаблону в View Tree. Если вы используете Chrome или Firefox, то вы попадете на панель элементов с выбранным элементом DOM.

<img src="/static/images/guides/ember-inspector/view-tree-template.png" width="350">

<img src="/static/images/guides/ember-inspector/view-tree-elements-panel.png" width="450">

### Компоненты и строковые представления

По умолчанию, View Tree игнорирует компоненты и строковые представления. Чтобы загрузить их в View Tree, проверьте галочки на полях `Components` и `All Views`.

<img src="/static/images/guides/ember-inspector/view-tree-components.png" width="600">

Затем вы можете просматривать компоненты с помощью Object Inspector.

### Выделение шаблонов

#### Наведение курсора на VIEW TREE

Когда вы наводите курсор на элементы в View Tree, в приложении выделяются соответствующие шаблоны. У каждого выделенного шаблона вы можете увидеть имя и связанные объекты.

<img src="/static/images/guides/ember-inspector/view-tree-highlight.png" width="680">

#### Наведение курсора на приложение

Если вы хотите выделить шаблон или компонент напрямую в приложении, то щелкните на иконку лупы в Inspector, затем наведите курсор на приложение. Когда курсор мыши будет проходить через него, соответствующий шаблон или компонент будет выделен.

<img src="/static/images/guides/ember-inspector/view-tree-magnifying-glass.png" width="500">

Если вы щелкните по выделенному шаблону или компоненту, Inspector выберет его. Вы можете щелкнуть на базовые объекты, чтобы отправить их в object inspector.

<img src="/static/images/guides/ember-inspector/view-tree-inspect.png">

Нажмите на кнопку `X`, чтобы отменить выбор шаблона.

### Продолжительность

Колонка Duration показывает время отображения для данного шаблона, включая его потомков.

<img src="/static/images/guides/ember-inspector/view-tree-duration.png" width="500">

Измеряя время отображения, Inspector добавляет небольшую задержку в процесс отображения. Поэтому показатели в колонке не служат точным представлением ожидаемого времени отображения. Время отображения полезно для сравнения показателей как абсолютной меры производительности.