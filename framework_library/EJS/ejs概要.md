# EJS概要

## できること

- 例えばindex.htmlで<% %>内にJavaScriptを書ける

- MPAでheadを使いまわす時などに下記のように参照できる

```javascript
<%- include(/*moduleのパス*/) %>
```

- module内にプロパティを指定して、引数を受け取れる<br>
読み込む為のmoduleファイルは_から始めるとbuild時に対象外(partial class)

```html
index.html 

<%- include('../modules/meta/_head.html',{プロパティ:"引数"}) %>
<h1>Index page</h1>
<p>Hello world</p>
```

```html
_module.html

<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title><%= プロパティ %></title>
  </head>

  <body></body>
</html>
```

- 引数を外部ファイルから参照することもできる

```html
index.html <%- include('../modules/meta/_head.html',data) %>
<h1>Index page</h1>
<p>Hello world</p>
```

```html
module.html

<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title><%= title %></title>
  </head>

  <body></body>
</html>
```

```javascript
mainData.js

export const mainData = {
  data: {
    title: 'Index Page',
    desc: 'This is the index page',
    keywords: 'index,home,main'
  }
};
```
```javascript
vite.config.js

plugins: [
    ViteEjsPlugin(mainData, {
      ejs: {
        beautify: true,
      },
    }),
  ]
```