# ディレクトリ構造・説明
```
├── node_modules (npm modules)
├── public (静的ファイル画像・動画など)
├── src (ソースコード)
│   ├── data (EJSの引数に入るデータ)
│   │   └── mainData.js
│   ├── modules (HTML・EJSのmodule)
│   │   └── meta (マークアップのmeta module)
│   │       ├── _foot.html
│   │       └── _head.html
│   └── pages (画面描画関連のファイル)
│       ├── about
│       │   └── index.html
│       ├── assets (cssやjsファイル)
│       │   ├── scripts (jsファイル)
│       │   │   ├── main.js
│       │   │   └── modules (js module)
│       │   │       └── button.js
│       │   └── styles (cssファイル)
│       │       ├── _base.scss
│       │       ├── main.scss
│       │       └── modules(css module)
│       │           └── button.scss
│       └── index.html
├── .eslintrc (eslint設定ファイル)
├── .gitignore (githubリポジトリに追加させないファイルを指定)
├── .prettierignore (prettier設定ファイル)
├── .prettierrc (prettier対象外)
├── package-lock.json (installされた依存関係のバージョンを固定)
├── package.json (npm packageの設定ファイル)
├── readme.md
└── vite.config.js (Vite設定ファイル)
```