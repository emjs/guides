Иногда вам будет нужно определить компонент, который охватывает контент, обеспеченный другими шаблонами.

Например, представьте, что вы создаете компонент `blog-post`, который можно использовать в приложении, чтобы отобразить пост блога:

`app/templates/components/blog-post.hbs`
```hbs
<h1>{{title}}</h1>
<div class="body">{{body}}</div>
```

Теперь мы можем использовать компонент `{{blog-post}}` и передать ему свойства еще одного шаблона:

```hbs
{{blog-post title=title body=body}}
```

(Смотрите раздел [Передача свойств компоненту](http://emjs.ru/v2.0.0/components/passing-properties-to-a-component/))

В этом случае контент, который мы хотели показать, брался из модели. Но что если мы хотим, чтобы разработчик, использующий наш компонент, мог предоставить индивидуальный контент HTML?

В дополнение к простой форме, которую мы уже изучили, компоненты могут использоваться в **блочной форме**. В этой форме компоненты можно передавать шаблону Handlebars, который отображается внутри шаблона компонента, где появляется выражение `{{yield}}`.

Чтобы использовать блочную форму, добавьте знак `#` в начало имени компонента и не забудьте добавить закрывающий тег. (Смотрите документацию Handlebars по [блочным выражениям](http://handlebarsjs.com/#block-expressions)).

В таком случае мы можем использовать компонент `{{blog-post}}` в **блочной форме** и указать Ember, где следует отобразить блочный контент с использованием помощника `{{yield}}`. Чтобы обновить пример выше, мы сначала изменим шаблон компонента:

`app/templates/components/blog-post.hbs`
```hbs
<h1>{{title}}</h1>
<div class="body">{{yield}}</div>
```

Вы видите, что мы заменили `{{body}}` на `{{yield}}`. Это сообщает Ember, что контент будет предоставлен при использовании компонента.

Далее мы обновим шаблон с помощью компонента, чтобы использовать блочную форму:

`app/templates/index.hbs`
```hbs
{{#blog-post title=title}}
  <p class="author">by {{author}}</p>
  {{body}}
{{/blog-post}}
```

Важно отметить, что область действия шаблона внутри компонентного блока такая же, как снаружи. Если свойство доступно в шаблоне вне компонента, оно будет доступно и внутри компонентного блока.