# HTML


# EJS
## for文を用いてHTML要素をコピー
### 構文例
```javascript
  <% for (let i = 0; i < 5; i++) { %>  
    <li class="hero-images-item">
    <img src="https://placehold.jp/438x292.png" alt="">
    </li>    
  <% } %>
```

### 解説
1. EJSのタグ<% %>で囲った中にJavaScriptでfor文を記述<br>
※EJSタグ内にHTMLは書けないので、for文の閉括弧だけ別で囲む
2. for文内にコピーしたい要素を記述

**補足：**上記コードでいう 5 の部分にコピーしたい数を記載
