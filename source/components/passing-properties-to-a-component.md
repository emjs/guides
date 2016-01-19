Компоненты изолированы от среды, поэтому любые данные, которые нужны компоненту, должны быть ему переданы.

Например, представьте, что у вас есть компонент `blog-post`, который используется, чтобы отобразить пост блога:

`app/templates/components/blog-post.hbs `
```hbs
<article class="blog-post">
  <h1>{{title}}</h1>
  <p>{{body}}</p>
</article>
```

Теперь представьте, что у нас есть следующие шаблон и маршрут:

`app/routes/index.js`
```js
export default Ember.Route.extend({
  model() {
    return this.store.findAll('post');
  }
});
```

Если бы мы попытались использовать компонент таким образом:

`app/templates/index.hbs`
```hbs
{{#each model as |post|}}
  {{blog-post}}
{{/each}}
```

Был бы отображен следующий код HTML: 

```html
<article class="blog-post">
  <h1></h1>
  <p></p>
</article>
```

Чтобы свойство стало доступно компоненту, вы должны передать его так:

`app/templates/index.hbs`
```hbs
{{#each model as |post|}}
  {{blog-post title=post.title body=post.body}}
{{/each}}
```

Важно отметить, что эти свойства остаются синхронизированными (технически это называется "bound"). То есть, если значение `componentProperty` в компоненте меняется, то `outerProperty` будет обновлен, чтобы отразить это изменение. Возможна и обратная ситуация.

## Позиционные параметры

Помимо передачи параметров по имени, вы можете передавать их по позиции. Другими словами, вы можете вызвать компонент выше так: 

`app/templates/index.hbs`
```hbs
{{#each model as |post|}}
  {{blog-post post.title post.body}}
{{/each}}
```

Чтобы компонент получил параметры таким образом, нужно установить атрибут `positionalParams` в классе компонента.

`app/components/blog-post.js`
```js
const BlogPostComponent = Ember.Component.extend({});

BlogPostComponent.reopenClass({
  positionalParams: ['title', 'body']
});

export default BlogPostComponent;
```

Затем вы можете использовать атрибуты в компоненте, как если бы они были переданы в виде `{{blog-post title=post.title body=post.body}}`.

Обратите внимание, что свойство `positionalParams` добавлено к классу в качестве статической переменной через `reopenClass`. Позиционные параметры всегда объявляются в классе компонента, и их нельзя изменить пока приложение запускается.
  
В качестве альтернативы вы можете принять произвольное число параметров, если назначить `positionalParams` строку, например: `positionalParams: 'params'`. Это позволит получить доступ к этим параметрам как к массиву:

`app/components/blog-post.js`
```js
const BlogPostComponent = Ember.Component.extend({
  title: Ember.computed('params.[]', function(){
    return this.get('params')[0];
  }),
  body: Ember.computed('params.[]', function(){
    return this.get('params')[1];
  })
});

BlogPostComponent.reopenClass({
  positionalParams: 'params'
});

export default BlogPostComponent;
```