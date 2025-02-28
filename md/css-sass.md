# CSS / SASS

## プロパティ概要

- ### box-shadow

  ボックス要素の影を設定

  **構文**

  ```scss
  // box-shadow: x y ぼかし 広がり rgba(r,g,b,a);
  box-shadow: 10px -10px 25px 5px rgba(0, 0, 0, 0.15);
  ```

- ### transform-origin

  transformの起点を設定（デフォルトは中心）

  **構文**

  ```scss
  // transform-origin: 水平位置　垂直位置;
  transform-origin: 0% 50%;
  ```

- ### ease-in-out
  transitionの設定値
  最初と最後はゆっくりで中間は早くなる
- ### white-space: nowrap;
  要素内のコンテンツがコンテナの幅を超えても改行されない
  
  カルーセルなどに便利
- ### display: inline-block;
  ブロック要素を横並びにできる

- ### 1em
  現在のフォントサイズに対する相対サイズ
- ### letter-spacing
  文字の横感覚

- ### aspect-ratio
  サイズに関係なく画像の比率を固定できる

  **構文**

  ```scss
  //画像を横352px,縦240pxで固定
    img{
      aspect-ratio: 352/240px;
    }
  ```

## modifier class

使いまわす用のスタイルを設定した疑似クラス

```scss
// クラスを適用したい要素のクラスに以下を設定

// modifier class
.link-nav {
  padding: 16px 32px;
  border-radius: 74px;
  font-size: 14px;
}
```

## 疑似要素

CSS / SASSで要素を追加することができる

### 構文

```scss
.class {
  padding: 80px;

  // before:親要素の前に疑似要素を追加
  &::before {
    content: "手前"; // コンテンツ指定必須、空も可
    background: blue;
  }

  // after:親要素の後に疑似要素を追加
  &::after {
    content: "後ろ";
    background: green;
  }
}
```

<br>

### 疑似要素で親要素に動的アンダーラインを作る例

```scss
.nav-link {
  padding: 10px 20px;
  text-decoration: none;
  color: cn.$color-main;
  display: inline-block;
  position: relative;
  font-size: 16px;
  font-weight: bold;

//親要素の後に疑似要素を追加
  &::after {
    content: ""; //空要素
    display: block;
    width: 100%;
    height: 2px;
    background: cn.$color-main;
    position: absolute;
    left: 0;
    bottom: 0;
    box-shadow: 5px 5px 10px 5px rgba(100, 100, 100, 0.25);
    transform: scaleX(0);
    transform-origin: 0% 50%;
    transition: transform ease-in-out 0.15s;
  }
// ホバーしたら
  &:hover {
    &::after {
      transform: scaleX(1); // 等倍にして描画
    }
  }
}
```

<br>

## 操作不可オートスライドカルーセル実装

```scss
.hero-images {
  width: 100%;
  position: absolute;
  z-index: 2;
  bottom: -1vw;
  left: 0;
}
.hero-images-list {
  white-space: nowrap;
  animation: infiniteCarousel 60s linear infinite;
}
@keyframes infiniteCarousel {
  0% {
    transform: translateX(0);
  }
  100% {
    transform: translateX(-50%);
  }
}
.hero-images-item {
  height: 100%;
  display: inline-block;
  margin: 0 5px;
  > img {
    width: auto;
    height: 286px;
  }
}
```