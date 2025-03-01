# Vite + Prettier + ESLint 環境構築ガイド

## 1. Prettier と ESLint の違い

### **Prettier（コードフォーマッター）**
**目的**: コードのフォーマット（整形）を自動化し、統一する。  

#### **主な特徴**
- コードの**見た目を統一**する（インデント、クォート、改行 など）
- 設定がシンプルで、開発チームのスタイルを統一できる
- `prettier --write` でコードを自動整形できる

#### **例**
**適用前**
```js
function helloWorld() { console.log( "Hello, world!" ); }
```
**適用後**
```js
function helloWorld() {
  console.log("Hello, world!");
}
```

---

### **ESLint（静的解析ツール）**
**目的**: コードの品質向上やバグ防止のため、文法的な誤りやベストプラクティス違反をチェックする。

#### **主な特徴**
- コードの**構文エラーや品質**をチェック
- 例: 未使用の変数、`===` の強制、バグの防止 など
- `eslint --fix` で一部の修正が可能

#### **例**
```js
let unusedVar = 42; // ← 未使用の変数として警告
console.log(  "Hello, world!") // ← 不要なスペースやセミコロンを指摘
```

---

## 2. Prettier と ESLint の違いまとめ

| ツール | 目的 | 例 |
|--------|------|----|
| **Prettier** | **コードのフォーマットを統一** | インデント、改行、クォートの統一 |
| **ESLint** | **コードの品質チェック** | 未使用の変数、`===` の強制、バグの防止 |

---

## 3. Vite 環境構築

### **プロジェクトの作成**
```sh
npm create vite@latest my-project
cd my-project
npm install
```

---

## 4. Vite + Prettier + ESLint の環境構築

### **1. ESLint の導入**
```sh
npm install --save-dev eslint
npx eslint --init
```
`.eslintrc.cjs` の設定:
```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  extends: [
    "eslint:recommended",
    "plugin:prettier/recommended"
  ],
  parserOptions: {
    ecmaVersion: "latest",
    sourceType: "module",
  },
  rules: {
    "no-unused-vars": "warn",
    "prettier/prettier": "error"
  },
};
```

---

### **2. Prettier の導入**
```sh
npm install --save-dev prettier eslint-config-prettier eslint-plugin-prettier
```
`.prettierrc` の設定:
```json
{
  "singleQuote": true,
  "semi": false,
  "trailingComma": "es5"
}
```

`.prettierignore` の設定:
```txt
node_modules
dist
```

---

### **3. VSCode での自動フォーマット設定**
`.vscode/settings.json`:
```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "eslint.validate": ["javascript", "typescript", "vue", "react"]
}
```

---

### **4. Lint & Format の実行**
```sh
npx eslint --fix .
npx prettier --write .
```

---

### **5. Husky + lint-staged（コミット前にチェック）**
```sh
npm install --save-dev husky lint-staged
```
`package.json` に追加:
```json
"husky": {
  "hooks": {
    "pre-commit": "lint-staged"
  }
},
"lint-staged": {
  "**/*.{js,jsx,ts,tsx,vue}": ["eslint --fix", "prettier --write"]
}
```
設定後、husky を有効化:
```sh
npx husky install
```

---

## **まとめ**
| ツール | 役割 |
|--------|------|
| **Vite** | 高速なフロントエンド開発環境 |
| **ESLint** | コードの品質チェック |
| **Prettier** | コードのフォーマット統一 |
| **husky + lint-staged** | コミット前に ESLint & Prettier を適用 |

この環境を構築することで、**Vite の開発速度を活かしながら、ESLint で品質を管理し、Prettier でコードフォーマットを統一** できます。
