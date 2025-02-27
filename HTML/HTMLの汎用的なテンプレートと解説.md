# HTMLの汎用的な初期設定テンプレート

## HTML5の基本テンプレート

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>Document</title>
  <link rel="stylesheet" href="styles.css"> <!-- 外部CSSファイル -->
  <script src="script.js" defer></script> <!-- 外部JavaScriptファイル -->
</head>
<body>
  <header>
    <h1>サイトタイトル</h1>
    <nav>
      <ul>
        <li><a href="#">ホーム</a></li>
        <li><a href="#">サービス</a></li>
        <li><a href="#">お問い合わせ</a></li>
      </ul>
    </nav>
  </header>
  <main>
    <section>
      <h2>セクションタイトル</h2>
      <p>ここに本文が入ります。</p>
    </section>
  </main>
  <footer>
    <p>&copy; 2024 あなたのサイト名</p>
  </footer>
</body>
</html>
```

---

## よく使われる初期設定ポイント

### 1. **`<meta>`タグ**
- **`charset="UTF-8"`**: 文字コードをUTF-8に設定。
- **`viewport`**: レスポンシブデザイン対応のために設定。
- **`X-UA-Compatible`**: 古いIEブラウザでの互換モードを防ぐ。

```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

---

### 2. **CSSリセットの利用**
ブラウザ間で異なるデフォルトスタイルを統一するためにCSSリセットを使用する。
- Normalize.cssの例:
```html
<link rel="stylesheet" href="https://necolas.github.io/normalize.css/8.0.1/normalize.css">
```

---

### 3. **外部リソースの読み込み**
- **CSSファイル**: `<link>` タグを使用。
- **JavaScriptファイル**: `<script>` タグで `defer` 属性を付けると、HTML解析後にスクリプトが実行される。

---

### 4. **`lang`属性**
HTMLの`<html>`タグに`lang`属性を指定することで、検索エンジンや支援技術が正しく解釈できるようにする。
```html
<html lang="ja">
```

---

### 5. **コメントと構造化**
HTMLをセクションごとに分け、コメントで補足する。
```html
<!-- ヘッダー -->
<header>
  ...
</header>
```

---

## 開発効率を上げるための工夫

### 1. **ライブリロードの活用**
- **Live Server**（VSCode拡張機能）を利用して、コードの変更をリアルタイムでブラウザに反映。

### 2. **BEM命名規則**
クラス名を管理しやすくする方法。
```html
<div class="block__element--modifier"></div>
```

### 3. **ダミーコンテンツ**
プロトタイプ作成時に[Lorem Ipsum](https://lipsum.com/)を活用。

---

## 最終的なHTML構成イメージ

- **`<header>`**: サイトタイトルやナビゲーションバーを配置。
- **`<main>`**: メインコンテンツを配置。
- **`<footer>`**: 著作権情報やフッターリンクを配置。

---

これを基にプロジェクトに応じてカスタマイズしてください！
