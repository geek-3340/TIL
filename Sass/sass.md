# SASS概要

## できること

- ネストできる（以下参照）

  ```scss
  .button {
    background-color: red;
    &:hover {
      background-color: #0f0;
    }
    .text {
      color: yellow;
    }
  }
  ```

- インポート機能

  読み込む為のmoduleファイルは\_から始めるとbuild時に対象外(partial class)

  ```scss
  _module.scss
   
    .button {
    background-color: red;
    &:hover {
      background-color: #0f0;
    }
    .text {
      color: yellow;
    }
  }
  ```

  ```scss
  main.scss

  @use 'module';
  ```

  ※SASSのパスでは\_や拡張子は省略可

- 変数を使える
  1. 変数用ファイルを作成し変数宣言
  ```scss
  _const.scss

  $color-main: #214a4a;
  $color-sub: #289B8F;
  $color-sub-dark: #248b80;
  $color-core: #212732;
  $color-accent: #EE7D2B;
  $color-accent-dark: #d67026;
  $color-base: #F4F4F4;
  $color-strong: #0DD48D;
  $color-stream: #313948;
  $color-img-bg: #C5DDDD;
  $color-gray: #ededed;

  $site-width: 1440px;
  $z-index-max: 100;
  $bp-lg: 1280px;
  $bp-sm: 600px;
  ```

  2. 読み込み
  ```scss
  _base.scss

  @use "./const" as cn;

  body {
    font-family: "Noto Sans JP", "Meiryo", "Arial", 
    background: cn.$color-base;
    letter-spacing: .05em;
    -webkit-tap-highlight-color: cn.$color-base;
  }

  ```